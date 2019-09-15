---
title: 理解webpack4.splitChunks之chunks
date: 2019-01-24 15:38:10
categories: [技术, 前端]
tags: [webpack]
---

上回说到按照默认的 splitChunks 配置，入口里面的第三方依赖没有打包出来，这个是因为 chunks 属性的原因，下面我们就介绍 chunks 属性的意义和用法。

<!-- more -->

chunks 的含义是拆分模块的范围，它有三个值 async、initial 和 all。

async 表示只从异步加载得模块（动态加载 import()）里面进行拆分
initial 表示只从入口模块进行拆分
all 表示以上两者都包括
我们回顾下上一篇文章里面我们说的 webpack splitChunks 默认配置：

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

我们发现 chunks 的默认配置是 async，也就是只从动态加载得模块里面进行拆分，怪不得能够把 page1 引入的第三方模块拆分出来，但是因为 entry1.js 属于入口 chunk 所以它引入的第三方库 react-dom 就没能拆分出来。
现在让我们来将 chunks 设置为 all 看看能否修复这个问题，下图就是 chunks 设置 all 拆分出来的代码：
{% img /images/splitChunks-chunks/1.png %}
这回我们发现入口模块的第三方依赖已经被成功拆分出来了（第一个），接下来让我来考考你，如果我将 chunks 设置为 initial，打包的结果会是什么呢？
{% img /images/splitChunks-chunks/2.png %}
上面就是 chunks 设置为 initial 的结果，不知道你猜对了吗，也就是将 page1.js 这个动态加载的模块所引入的第三方模块去掉了，没有拆分出来，因为 initial 只会对入口模块进行拆分。
