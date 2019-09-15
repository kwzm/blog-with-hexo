---
title: 理解webpack4.splitChunks
date: 2019-01-24 15:09:16
categories: [技术, 前端]
tags: [webpack]
---

## 前言

之前一直也没有研究过 webpack4 是基于怎样的规则去拆分模块的，现在正好有时间打算好好了解一下，看了官方文档也陆陆续续的看了看网上别人写的文章，感觉大部分都是将官方文档翻译了一遍，很多问题都没有解释清楚，无奈只好自己写 demo 去通过实际编译结果来理解，经过一天多的不断调试和百度，基本弄清楚了 splitChuns 的运行规则了，特此记录下来。

<!-- more -->

## webpack 中的三个概念 module、chunk 和 bundle

在研究 splitChunks 之前，我们必须先弄明白这三个名词是什么意思，主要是 chunk 的含义，要不然你就不知道 splitChunks 是在什么的基础上进行拆分。

从官网上貌似没找太多的解释，去网上搜了搜基本上都在转述这位老哥的回答[what are module,chunk and bundle in webpack](https://stackoverflow.com/questions/42523436/what-are-module-chunk-and-bundle-in-webpack)，我根据自己的理解给出我个人的看法：

module：就是 js 的模块化 webpack 支持 commonJS、ES6 等模块化规范，简单来说就是你通过 import 语句引入的代码。
chunk: chunk 是 webpack 根据功能拆分出来的，包含三种情况：
　　　　 1. 你的项目入口（entry）

2. 通过 import()动态引入的代码

3. 通过 splitChunks 拆分出来的代码

chunk 包含着 module，可能是一对多也可能是一对一。

bundle：bundle 是 webpack 打包之后的各个文件，一般就是和 chunk 是一对一的关系，bundle 就是对 chunk 进行编译压缩打包等处理之后的产出。

## splitChunks

下面进入正题讲解 splitChunks，splitChunks 就算你什么配置都不做它也是生效的，源于 webpack 有一个默认配置，这也符合 webpack4 的开箱即用的特性，它的默认配置如下：

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

我们现在用一个简单的 react 项目来测试下打包之后的效果如何，我的这个项目有两个页面 entry1.js 和 page1.js，entry1.js 是入口文件，entry1.js 里面动态引入了 page1.js。

entry1.js

```javascript
 1 import React from 'react'
 2 import ReactDOM from 'react-dom'
 4
 5 const App = () => {
 6   let Page1 = null
 7
 8   import(/* webpackChunkName: "page1" */'./routes/page1').then(comp => {
 9     Page1 = comp
10   })
11   console.log($)
12   return (
13     <div>
14       <div>App</div>
15       <Page1 />
16     </div>
17   )
18 }
19
20 ReactDOM.render(<App />, document.getElementById('root'))
```

page1.js

```javascript
 1 import React from 'react'
 2 import _ from 'lodash'
 4
 5 const Page1 = () => {
 6   console.log($)
 7
 8   return (
 9     <div>
10       <div>Page1</div>
11     </div>
12   )
13 }
14
15 export default Page1
```

让我们想一想打包之后的代码是怎样的呢？

以上就是打包之后的代码，是否如你所想呢，让我们分析一下：

1. 第一个 main 文件就是打包之后的入口文件，这个我们上面说了 webpack 会把入口文件单独拆成一个 chunk，这个没有问题

2. 第三个 page1 文件，我们上面也说过动态加载得文件 webpack 会将其拆分为一个 chunk，这个也没有问题

3. 第二个 vendor~page1 文件，这个是对 page1 文件里面引入的第三方库进行打包，具体就是 lodash 那个第三方库了，这个涉及到 cacheGroup，我们在下面的系列文章里面会详细讲述

以上就是所有被拆分出来的包，但是我们发现有一个文件没有拆分出来，那就是 entry1 里面引入的第三方库 react-dom，这个是为什么呢，这个就要涉及到我们接下来讲到的 chunks 属性的配置。

注意：这里提个小问题为什么 react-dom 这个第三方库只在 entry1.js 里面引入了一次就被拆分出来了？这个答案我将在第三篇文章《理解 webpack4.splitChunks 之 cacheGroups》里面进行解释。

为了方便阅读我将整个系列分为了若干小部分，大家可以各取所需：

《理解 webpack4.splitChunks 之 chunks》

《理解 webpack4.splitChunks 之 cacheGroups》

《理解 webpack4.splitChunks 之 maxInitialRequests》

《理解 webpack4.splitChunks 之 maxAsyncRequests》

《理解 webpack4.splitChunks 之其余要点》

文章中所用到的源码仓库地址是 [webpack-splitChunks-demo](https://github.com/kwzm/webpack-splitChunks-demo)
