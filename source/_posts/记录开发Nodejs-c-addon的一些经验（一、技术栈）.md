---
title: 记录开发Nodejs c++ addon的一些经验（一、技术栈）
date: 2017-09-25 16:32:31
categories: [技术, 桌面端]
tags: [nodejs, electron]
---

Nodejs c++ addon 是用 c++去编写 Nodejs 的插件

<!-- more -->

技术栈：

1、[node-gyp](https://github.com/nodejs/node-gyp)

一个用于把 c++文件编译成 node 可执行文件的库

2、[v8](https://v8.paulfryzel.com/docs/master/index.html)

google v8 引擎 用于处理 c++的数据类型和 node 的数据类型的转换

3、[nan](https://github.com/nodejs/nan)

相当于对 v8 做了一层封装，去处理 v8 不同版本兼容的问题

4、[c++](http://www.cplusplus.com/reference/fstream/ifstream/)

因为插件是用 c++编写，所以掌握 c++的知识
