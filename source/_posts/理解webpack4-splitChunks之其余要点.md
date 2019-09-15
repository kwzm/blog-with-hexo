---
title: 理解webpack4.splitChunks之其余要点
date: 2019-01-29 13:01:43
categories: [技术, 前端]
tags: [webpack]
---

1. splitChunks 除了之前文章提到的规则外，还有一些要点或是叫疑惑因为没有找到官方文档的明确说明，所以是通过我自己测试总结出来的，只代表我自己的测试结果，不一定正确。

<!-- more -->

2. splitChunks.cacheGroup 必须同时满足各个条件才能生效，这个之前我理解错误，我以为比如 minSize 或是 minChunks 等条件只要满足一条就可以拆分，但是实际上必须同时满足才行

3. splitChunks 的配置项都是作用于 cacheGroup 上的，如果将 cacheGroup 的默认两个分组 vendor 和 default 设置为 false，则 splitChunks 就不会起作用
   minChunks、maxAsyncRequests、maxInitialRequests 的值必须设置为大于等于 1 的数

4. 当 chunk 没有名字时，通过 splitChunks 分出的模块的名字用 id 替代，当然你也可以通过 name 属性自定义

5. 当父 chunk 和子 chunk 同时引入相同的 module 时，并不会将其分割出来而是删除掉子 chunk 里面共同的 module，保留父 chunk 的 module，这个是因为 optimization.removeAvaliableModules 默认是 true

6. 当两个 cacheGroup.priority 相同时，先定义的会先命中

7. 除了 js，splitChunks 也适用于 css
