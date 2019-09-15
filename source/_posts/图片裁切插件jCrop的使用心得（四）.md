---
title: 图片裁切插件jCrop的使用心得（四）
date: 2016-02-03 16:37:34
categories: [技术, 前端]
tags: [图片裁切]
---

在本篇中我来介绍一下 jcrop 如何实时展现用户裁切的效果图以及在项目中使用该插件注意的问题。

<!-- more -->

首先，你们在创建头像时，都可以在旁边实时的看到我裁切后的效果图，就如博客园。

这个是如何实现的呢，其实并不难。

还记得我们在 Jcrop 插件初始化的时候配置了一个这个参数吗。

这个参数就是当裁切框不断变化的时候执行的方法。

```javascript
//简单的事件处理程序，响应自onChange,onSelect事件，按照上面的Jcrop调用
function showPreview(coords) {
  if (parseInt(coords.w) > 0) {
    //计算预览区域图片缩放的比例，通过计算显示区域的宽度(与高度)与剪裁的宽度(与高度)之比得到
    var rx = $previewBox.width() / coords.w;
    var ry = $previewBox.height() / coords.h;
    //通过比例值控制图片的样式与显示
    $("#crop_preview").css({
      width: Math.round(rx * $originalImg.width()) + "px", //预览图片宽度为计算比例值与原图片宽度的乘积
      height: Math.round(rx * $originalImg.height()) + "px", //预览图片高度为计算比例值与原图片高度的乘积
      marginLeft: "-" + Math.round(rx * coords.x) + "px",
      marginTop: "-" + Math.round(ry * coords.y) + "px"
    });
  }
}
```

要实现该效果还需要对应的 html 结构，很简单。

```html
<div id="preview_box">
  <img id="crop_preview" src="" alt="" />
</div>
```

这里面的图片路径就是用户上传后的图片路径，我们通过对这个预览图片来做一些 css 样式的变化来实现预览效果。
要注意的一些问题：

1、在用户觉得图片不好删除掉他上传的图片后，同时也要将 jcrop 插件销毁掉。

```javascript
if (jcropObj) {
  jcropObj.destroy();
}
```

2、要考虑用户上传图片如果超过了外面的展示窗口怎么办。

这里可以自己去看下博客园的做法，其实它是把图片在后台进行了缩放。让图片能够不超出这个展示框。

那么到此我想介绍的一些关于 jCrop 插件我使用的心得就已经分享完了，咱们下篇再见。
