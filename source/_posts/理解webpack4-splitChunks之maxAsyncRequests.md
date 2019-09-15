---
title: 理解webpack4.splitChunks之maxAsyncRequests
date: 2019-01-24 19:50:21
categories: [技术, 前端]
tags: [webpack]
---

maxAsyncRequests 和 maxInitialRequests 有相似之处，它俩都是用来限制拆分数量的，maxInitialRequests 是用来限制入口的拆分数量而 maxAsyncRequests 是用来限制异步模块内部的并行最大请求数的，说白了你可以理解为是每个 import()它里面的最大并行请求数量。

<!-- more -->

这其中要注意以下几点：

1. import()文件本身算一个请求

2. 并不算 js 以外的公共资源请求比如 css

3. 如果同时有两个模块满足 cacheGroup 的规则要进行拆分，但是 maxInitialRequests 的值只能允许再拆分一个模块，那尺寸更大的模块会被拆分出来

我们还是通过一个例子来解释一下，我定义三个入口文件 entry1.js、entry2.js 和 entry3，entry1.js 里面动态加载 page1.js

webpack 的配置如下：

因为默认的 maxAsyncRequests 为 5 太大了，不方便测试，所以改为了 3
{% img /images/splitChunks-maxAsyncRequests/1.png %}
entry1.js：

```javascript
import React from "react";
import ReactDOM from "react-dom";

const App = () => {
  let Page1 = null;

  import(/* webpackChunkName: "page1" */ "./routes/page1").then(comp => {
    Page1 = comp;
  });

  return (
    <div>
      <div>App</div>
      <Page1 />
    </div>
  );
};

ReactDOM.render(<App />, document.getElementById("root"));
```

entry2.js

```javascript
import React from "react";
import ReactDOM from "react-dom";
import $ from "./assets/jquery";
const App = () => {
  console.log($);
  return (
    <div>
      <div>entry2</div>
    </div>
  );
};

ReactDOM.render(<App />, document.getElementById("root"));
```

entry3.js

```javascript
import React from "react";
import ReactDOM from "react-dom";
import OrgChart from "./assets/orgchart";

const App = () => {
  return (
    <div>
      <div>App</div>
    </div>
  );
};

ReactDOM.render(<App />, document.getElementById("root"));
```

page1.js

```javascript
import React from "react";
import _ from "lodash";
import $ from "../assets/jquery";
import OrgChart from "../assets/orgchart";

const Page1 = () => {
  return (
    <div>
      <div>Page1</div>
    </div>
  );
};

export default Page1;
```

打包结果：
{% img /images/splitChunks-maxAsyncRequests/2.png %}
让我们分享一下打包结果，主要是看和 page1 有关的，因为 page1 是通过 import()动态引入的

1. vendors~page1.chunkhash.chunk.js 是 page1 里面引入的第三方库 lodash，这个是根据 cacheGroups 进行拆分的，如果不明白可以去看我之前的文章《理解 webpack4.splitChunks 之 cacheGroups》

2. page1.chunkhash.chunk.js 是 page1.js 文件本身，这个也没问题，如果不明白为啥 import()的文件会被拆分可以去看我的第一篇文章《理解 webpack.splitChunks》

3. default\~entry2~page1.chunkhash.js 这个拆分的 entry2 和 page1 的共用文件 jquery.js，这个是根据 cacheGroups 进行拆分的，如果不明白可以去看我之前的文章《理解 webpack4.splitChunks 之 cacheGroups》

那么 page1 这个异步模块的并发请求数正好是设置的最大值 3，符合 maxAsyncRequests

这里我们发现除了 jquery.js 之外 page1.js 和 entry3.js 还共同引入了 orgchart.js 文件，但是却没有被拆分出来，这就是因为 maxAsyncRequests 的限制，如果我们把值改为 4 呢？

改为 4 后进行打包，打包的结果如下：
{% img /images/splitChunks-maxAsyncRequests/3.png %}
{% img /images/splitChunks-maxAsyncRequests/4.png %}
我们发现 orgchart.js 被打包出来了，这个时候 page1.js 的最大请求数量也变成了 4 个。

注意： 这里有一个小问题，为啥是 jquery.js 被拆分了而不是 orgchart.js？

仔细看文章的小伙伴应该会发现我在文章开头提到的需要注意的几个点中最后一点，在匹配 maxAsyncRequests 这个条件进行拆分的时候尺寸大的包会先被拆分
{% img /images/splitChunks-maxAsyncRequests/5.png %}
由上图我们知道 jquery.js 的尺寸远远大于 orgchart.js 的尺寸，所以它被先拆分了。

PS：至于 vendors~page1.chunkhash.chunk.js 为啥没有把 react 拆出来可以去看我的最后一篇博客《理解 webpack 之其余要点》
