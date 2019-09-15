---
title: 几个css的小知识点
date: 2016-02-03 15:14:05
categories: [技术, 前端]
tags: [css]
---

1、对于不能确定宽度的 div 让它水平居中。

```html
<div class="father">
  <div class="son">居中</div>
</div>
```

son 的宽度不确定，要让它在 father 里面水平居中

<!-- more -->

方法：

```css
.father {
  text-align: center;
}
.son {
  display: inline-block;
}
```

2、box-sizing 的认识

（1）值为 content-box（默认）
在宽度和高度之外绘制元素的内边距和边框。
（2）值为 border-box
为元素指定的任何内边距和边框都将在已设定的宽度和高度内进行绘制。

```css
.content-box {
  -moz-box-sizing: content-box;
  -webkit-box-sizing: content-box;
  -o-box-sizing: content-box;
  -ms-box-sizing: content-box;
  box-sizing: content-box;
  width: 200px;
  height: 200px;
  padding: 10px;
  margin: 20px;
  border: 1px solid red;
  text-align: center;
  line-height: 200px;
}

.border-box {
  -moz-box-sizing: border-box;
  -webkit-box-sizing: border-box;
  -o-box-sizing: border-box;
  -ms-box-sizing: border-box;
  box-sizing: border-box;
  width: 200px;
  height: 200px;
  padding: 10px;
  margin: 20px;
  border: 1px solid blue;
  text-align: center;
  line-height: 200px;
}
```

以上两个元素的宽和高都设置为 200px，可以看出两者的区别。
