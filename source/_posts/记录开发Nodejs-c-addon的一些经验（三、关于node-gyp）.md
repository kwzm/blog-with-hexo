---
title: 记录开发Nodejs c++ addon的一些经验（三、关于node-gyp）
date: 2017-09-26 15:20:08
categories: [技术, 桌面端]
tags: [nodejs, electron]
---

关于 node-gyp 如何进行编译，我想它的官网已经说的很详细了：

https://github.com/nodejs/node-gyp

但是我感觉关于 binding.gyp 文件的语法规则还是说的不明确，大概是因为它本身也不属于 node-gyp 的范畴吧

<!-- more -->

gyp 文件的语法规则的官方文档应该是这个：https://code.google.com/p/gyp/wiki/GypLanguageSpecification

但是由于墙的原因，所以无法打开，国内我从网上找的比较好的文档是这个：

http://www.cnblogs.com/x_wukong/p/4829598.html

大部分配置都能找到。

在这里我主要说下以下两个问题：

一、引入第三方头文件和库文件

我以引入 boost 这个库为例，头文件的引入通过设置 include_dirs 而库的引入通过设置 link_settings。

配置文件如下：

```json
{
  "variables": {
    "boost_dir": "D:/boost_1_65_1",
    "boost_lib": "D:/boost_1_65_1/stage/vc14-x64/lib/"
  },
  "targets": [
    {
      "target_name": "addon",
      "sources": ["addon_boost.cpp"],
      "include_dirs": ["<!(node -e \"require('nan')\")", "<(boost_dir)"],
      "link_settings": {
        "libraries": [
          "<(boost_lib)/libboost_filesystem-vc140-mt-s-1_65_1.lib",
          "<(boost_lib)/libboost_system-vc140-mt-s-1_65_1.lib"
        ]
      }
    }
  ]
}
```

二、关于编译的架构（也就是系统位数）问题
node-gyp 提供 --arch 参数去支持编译成不同位数的 addon, ia32 表示 32 位，x64 表示 64 位。

在编译的时候需要注意你引入的第三方库是 32 位的还是 64 位的，我之前引入的 boost 库是 32 位的，然后我用 node-gyp 去按照 64 位去编译结果就报错了。

除此之外，还需要你的 node 位数和你的 addon 位数保持一致，否则无法用 node 执行该文件。

总结一下：就是需要 c++的源文件里引入的第三方库 和 node-gyp 和 node 这三个位数保持一致。

最后附上我的项目地址：

https://github.com/kwzm/node-iofile-addon
