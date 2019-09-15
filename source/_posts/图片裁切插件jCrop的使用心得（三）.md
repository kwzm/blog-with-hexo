---
title: 图片裁切插件jCrop的使用心得（三）
date: 2016-02-03 16:13:04
categories: [技术, 前端]
tags: [图片裁切]
---

在这一篇里，我来具体讲讲代码该如何写。

<!-- more -->

下面是 jCrop 的初始化代码

```javascript
//图片裁剪插件Jcrop初始化
function initJcrop() {
  // 图片加载完成
  document.getElementById("originalImg").onload = function() {
    var imgWidth = $("#originalImg").width();
    var imgHeight = $("#originalImg").height();
    //同时启动裁剪操作，触发裁剪框显示，让用户选择图片区域
    jcropObj = $.Jcrop("#originalImg");
    jcropObj.setOptions({
      //bgColor: 'black',
      //bgOpacity: .4,
      aspectRatio: imgScale,
      boxWidth: imgWidth,
      boxHeight: imgHeight,
      onChange: showPreview, //当裁剪框变动时执行的函数
      onSelect: saveData //当选择完成时执行的函数
    });
    //创建选框
    var selectWidth = imgWidth / 2;
    var selectHeight = selectWidth / imgScale;
    jcropObj.setSelect([0, 0, selectWidth, selectHeight]);
  };
}
```

注意：

因为图片并不是一开始就有的，而是要用户先上传而后才能显示的，所以我在这里用了 onload 事件，当图片加载成功时再去初始化 jCrop 插件。

另外就是为了用户更好的体验，我们先把选框（就是虚线勾勒的裁切框）创建好，这里因为我的图片裁切比例是不固定的，所以用 setSelect 方法

来生成，裁切框的起点坐标是 0,0.

下面是如何获取用户裁切好的参数。

```javascript
//保存图片裁剪的参数
function saveData(coords) {
  //保存图片的参数
  var scaleData = {
    url: $originalImg.attr("src"),
    x1: coords.x,
    y1: coords.y,
    width: coords.w,
    height: coords.h
  };
  $box.data("sacleData", scaleData);
}
```

这个方法是在 onSelect 事件触发时执行，即当选择完成时执行的函数。

这个回调函数会传过来一个 coords 参数，里面包含了裁切的起始点的坐标和裁切框的宽度和高度。

将这四个数值传递给后台即可，剩下的事情就是后台同事来处理了。

http://www.cnblogs.com/kissdodog/archive/2012/12/21/2827867.html

这里有个哥们用.net 后台来实现的图片裁切，如果需要的话可以看看，里面也有 demo 下载。

那么基本上的 jCrop 的使用就介绍的差不多了，再下一篇里我将介绍一些这些插件的扩展和遇到的问题。
