---
title: 理解webpack4.splitChunks之maxInitialRequests
date: 2019-01-24 18:41:10
categories: [技术, 前端]
tags: [webpack]
---

maxInitialRequests 是 splitChunks 里面比较难以理解的点之一，它表示允许入口并行加载的最大请求数，之所以有这个配置也是为了对拆分数量进行限制，不至于拆分出太多模块导致请求数量过多而得不偿失。

<!-- more -->

这里需要注意几点：

入口文件本身算一个请求
如果入口里面有动态加载得模块这个不算在内
通过 runtimeChunk 拆分出的 runtime 不算在内
只算 js 文件的请求，css 不算在内
如果同时又两个模块满足 cacheGroup 的规则要进行拆分，但是 maxInitialRequests 的值只能允许再拆分一个模块，那尺寸更大的模块会被拆分出来
是不是感觉还是太抽象了难以理解，我下面会通过一个例子来实际演示一下：

我在 webpack 里面配置三个入口文件 entry1.js、entry2.js 和 entry3.js：
{% img /images/splitChunks-maxInitialRequests/1.png %}
splitChunks 的配置如下基本就是默认配置（只不过把 chunks 的范围改为了 all）：

```javascript
splitChunks: {
  chunks: 'all',
  minSize: 30000,
  minChunks: 1,
  maxAsyncRequests: 5,
  maxInitialRequests: 3,
  automaticNameDelimiter: '~',
  name: true,
  cacheGroups: {
    vendors: {
      test: /[\\/]node_modules[\\/]/,
      priority: -10
    },
    default: {
      minChunks: 2,
      priority: -20,
      reuseExistingChunk: true
    }
  }
}
```

entry1.js：

```javascript
import React from "react";
import ReactDOM from "react-dom";
import $ from "./assets/jquery";
import OrgChart from "./assets/orgchart";

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

打包结果：
{% img /images/splitChunks-maxInitialRequests/2.png %}
让我们来分析下这个 demo：

1. entry1.js 和 entry2.js 两个入口会被拆分成两个文件就是最下面这两个，这个没有问题

2. 因为 entry1.js 和 entry2.js 都引入了第三方库 react、react-dom 所以会被 cacheGroups 中的 vendors 缓存组进行拆分成第二个包 vendors~entry1~entry2

3. 因为 entry1.js 和 entry2.js 都引入了共同的模块 jquery.js（注意 default 不是从 node_modules 里面引入的，是我下载到本地的），所以满足 cacheGroups 的 default 缓存组，所以被拆成了第二个包 default~entry1~entry2

4. page1 因为是在 entry1.js 里面动态引入的所以被拆分出来

5. vendors~page1 就是 page1 里面引入的第三方库 lodash

6. 剩下的三个文件都是入口 chunk

但是这里我们发现我们在 entry1.js 和 entry3.js 里面共同引入的 orgchart.js 没有被拆分出来，这个文件是完全满足 cacheGroups 中的 default 这个缓存组的，但是却没有被拆分出来，查看打包的分析页面：
{% img /images/splitChunks-maxInitialRequests/3.png %}
{% img /images/splitChunks-maxInitialRequests/4.png %}
我们发现 orgchart.js 确实都还在入口文件里面面，那为什么会导致这个问题呢，这就是因为咱们今天的主角 maxInitialRequests，因为它的默认值为 3，所以每个入口的并发请求数就不能大于 3，我们看下 entry1 的并发请求数目前有哪些：

1. entry1.js 本身是一个对应的就是 entry1.cc7c4ca4.js 这个文件

2. vendors~entry1~entry2~entry3.chunkhash.chunk.js（hash 太长了就不写了= =）

3. default~entry1~entry2.chunkhash.chunk.js

所以目前已经达到了最大的请求数 3，这就是为什么不会吧 orgchart.js 再拆分出来的原因，那么如果我把 maxInitialRequests 改为 4 呢？

打包之后的结果如下：
{% img /images/splitChunks-maxInitialRequests/5.png %}
{% img /images/splitChunks-maxInitialRequests/6.png %}
可以看到多打出来的包正是我们希望的 orgchart.js 文件，这也证明了 maxInitialRequests 的作用。
注意：你有没有发现第一次打包的时候为什么是 jquery 被拆分了而 orgchart.js 没有被拆分，为啥不是倒过来呢？

这就是我在文章开头所说的当同时又两个模块满足拆分条件的时候更大的包会先被拆分
{% img /images/splitChunks-maxInitialRequests/7.png %}
可以看到 jquery.js 的大小明显大于 orgchart.js 的大小，所以它被先拆分了。
