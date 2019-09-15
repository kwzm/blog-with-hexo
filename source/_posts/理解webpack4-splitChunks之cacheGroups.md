---
title: 理解webpack4.splitChunks之cacheGroups
date: 2019-01-24 16:12:31
categories: [技术, 前端]
tags: [webpack]
---

cacheGroups 其实是 splitChunks 里面最核心的配置，一开始我还认为 cacheGroups 是可有可无的，这是完全错误的，splitChunks 就是根据 cacheGroups 去拆分模块的，包括之前说的 chunks 属性和之后要介绍的种种属性其实都是对缓存组进行配置的。splitChunks 默认有两个缓存组：vender 和 default，可以再来回顾一下 splitChunks 的默认配置：

<!-- more -->

```javascript
module.exports = {
  //...
  optimization: {
    splitChunks: {
      chunks: "async",
      minSize: 30000,
      minChunks: 1,
      maxAsyncRequests: 5,
      maxInitialRequests: 3,
      automaticNameDelimiter: "~",
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
  }
};
```

想想第一篇文章里为什么默认的打包能够将第三方库拆分出来，就是因为 cacheGroups 里面定义了 vendors 这个缓存组，它的 test 设置为 `/[\\/]node_modules[\\/]/` 表示只筛选从 node_modules 文件夹下引入的模块，所以所有第三方模块才会被拆分出来。除此之外还有一个 default 缓存组，它会将至少有两个 chunk 引入的模块进行拆分，它的权重小于 vendors，下面我们通过 demo 来测试一下。

我修改 webpack 的 entry，新加入一个 entry2.js，代码如下：

webpack 配置：
{% img /images/splitChunks-cacheGroups/1.png %}

entry1.js

```javascript
import React from "react";
import ReactDOM from "react-dom";
import $ from "./assets/jquery";

const App = () => {
  return (
    <div>
      <div>entry1</div>
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
  return (
    <div>
      <div>entry2</div>
    </div>
  );
};

ReactDOM.render(<App />, document.getElementById("root"));
```

打包之后的结果如下：
{% img /images/splitChunks-cacheGroups/2.png 300 200 %}
{% img /images/splitChunks-cacheGroups/3.png 300 200 %}

entry1.js 和 entry2.js 都引入了 jquery.js，所以 jquery.js 的引用次数为 2，minChunks 的默认值为 2，所以正好满足要求于是第二个包被拆分出来了。

你当然也可以自己定义缓存组来根据你的项目实际情况进行耕细粒度的拆分。

另外还需要说明一下，cacheGroups 之外设置的约束条件比如说默认配置里面的 chunks、minSize、minChunks 等等都会作用于 cacheGroups，除了 test, priority and reuseExistingChunk，这三个是只能定义在 cacheGroup 这一层的，这也就解释了第一篇文章里面为什么 entry 里面引入的第三方库 react-dom 只被 entry1.js 引入了一次就会被打包出来，因为默认的 minChunks=1，这个属性会作用于所有的 cacheGroups，但是 cacheGroups 也可以将上面的所有属性都重新定义，就会覆盖外面的默认属性，比如 default 这个缓存组就设置了 minChunks=2，他会覆盖掉默认值 1。

注意：这块再提一个思考，为什么 entry1.js 和 entry2.js 里面都引入了 react-dom 这个第三方库，它完全满足 default 这个 cacheGroup 的条件但是为什么没有被包含在 default~entry1~entry2 这个 chunk 中而是被纳入了 vendor~entry1~entry2 这个 chunk 里面了呢？

其实这是因为 priority 这个属性起了作用，它的含义是权重，如果有一个模块满足了多个缓存组的条件就会去按照权重划分，谁的权重高就优先按照谁的规则处理，default 的 priority 是-20 明显小于 vendors 的-10，所以会优先按照 vendors 这个缓存组拆分。
