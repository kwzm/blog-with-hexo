---
title: 记录开发Nodejs c++ addon的一些经验（四、electron）
date: 2017-09-29 11:43:06
categories: [技术, 桌面端]
tags: [nodejs, electron]
---

如果我们要在 electron 里使用我们开发的 addon，那么直接使用是不行的。

官方的解释是：Electron 同样也支持原生模块，但由于和官方的 Node 相比使用了不同的 V8 引擎，如果你想编译原生模块，则需要手动设置 Electron 的 headers 的位置。

<!-- more -->

官方给出了几种解决办法，适用于不用场景：

https://github.com/electron/electron/blob/master/docs-translations/zh-CN/tutorial/using-native-node-modules.md

写的已经足够清楚了，就不赘述了。

因为我们是使用自己开发的 addon，所以采用了这种方式重新编译：

编译好之后，我把它放在了 node_modules/addon 里，另外新建了一个 addon.js

```javascript
const { wirteFile, readFile } = require("./build/Release/addon.node");

module.exports = {
  writeFile: writeFile,
  readFile: readFile
};
```

配置 package.json 的'main':addon.js"

结合 webpack 这样我就可以在其它页面里通过 require('addon')来使用它了，经过测试没有问题。

但是当我打包的时候却报了这样一个错误，大概是这样

```bash
Module parse failed: src/node_modules/webpack/package.json Line 2: Unexpected token :
You may need an appropriate loader to handle this file type.
```

这是因为 webpack 里面没有适合的 loader 去解析 node 文件，所以：

1、npm install --save node-loader

2、在 webpack 配置文件里添加相应的 rule

```javascript
module: {
  rules: [
      text: /.node$/,
      use: 'node-loader'
    ]
}
```

这样就 OK 了。
