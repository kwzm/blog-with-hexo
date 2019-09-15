---
title: 记录开发Nodejs c++ addon的一些经验（二、数据类型的转换(尤其是Buffer)）
date: 2017-09-25 17:10:27
categories: [技术, 桌面端]
tags: [nodejs, electron]
---

常见的数据类型的转换基本比较容易，结合 nan 应该不是一件难事

<!-- more -->

参考链接：

http://blog.jobbole.com/109598/

http://deadhorse.me/nodejs/2012/10/09/c_addon_in_nodejs_object.html

这里主要说一下 Buffer 类型的转换：

Buffer 是 nodejs 里面的类型，但是 c++里面是没有的，那么怎么实现它们之间的转换呢？

1、c++ -> nodejs

c++里面没有专门的 buffer 类型，但是有类似的 char \*[]，所以我们可以把它当场 buffer，那么怎么把它转换成 Nodejs 的 Buffer 呢

用 Nan::NewBuffer 就可以实现：

```c++
 Nan::NewBuffer(char* data, uint32_t size)
```

另外 nan 还提供了其它参数和 buffer 的工具方法，请移步：
https://github.com/nodejs/nan/blob/master/doc/buffers.md#api_nan_new_buffer

2、nodejs -> c++

node 的 buffer 模块提供了一些方法来做这件事

你需要引入 node_buffer.h 头文件，但是便利的是 nan.h 其实都帮我们引入了这些必要的头文件，所以你只需引入 nan.h 就可以了

通过如下代码你就可以获取到 bufferData 和长度：

```c++
Local<Object> bufferObj = args[0]->ToObject();
char* bufferData = node::Buffer::Data(bufferObj);
size_t bufferLength = node::Buffer::Length(bufferObj);
```

参考链接：

http://www.jianshu.com/p/68d849df6e5e

http://deadhorse.me/nodejs/2012/10/10/c_addon_in_nodejs_buffer.html
