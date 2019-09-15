---
title: HTML邮件制作规范
date: 2016-05-05 10:47:29
categories: [技术, 前端]
tags: [邮件]
---

以下内容有些是别人总结的，有些是自己在工作中总结的。

<!-- more -->

模板最佳尺寸：显示宽度 550px-750px，模板高度控制在一屏以内。

1、 用 table+css 方式构建模板

Div+css 布局不完全被邮件客户端支持，所以无法使用 div+css 布局。

2、 可以使用 editplues，或者 UltraEdit 等工具制作 html，但务必使用可视化工具检查嵌套情况推荐 Dreamweaver

3、 插入的图片要定义宽度，高度，针对（editplues，UltraEdit 等工具更要注意这一点）

4、 严禁使用背景图片

在 outlook2007 中，背景图片将无法显示，其他邮件客户端可正确显示背景图片。

Gmail 也不支持 css 里面的背景图，

5、 严禁使用 map 标记形式

造成后期可视化统计困难

6、 不使用 word 转换的 html 文件作为模板

7、 不要用外部链接的 css 文件

```html
<link
  rel="stylesheet"
  rev="stylesheet"
  type="text/css"
  href="/css/new/common.css"
  media="all"
/>
```

这样是抓取不到 css 的，要写在 html，head 里面
8、网站或者论坛客户自己有服务器的涉及上传模板文件的按照这个格式

http://www.abc.com/file/0902_tr/edm.html

http://www.abc.com/file/0902_tr/edm_online.html

这样的格式系统无法抓取

http://www.abc.com/file/0902_tr/

9、模板中的图片请使用绝对路径,完整的 URl

```html
<img src="http://www.abc/123.jpg" width="140" height="123" border="0" ></img>

<img src="http://www.abc/456.jpg" width="140" height="123" border="0" ></img>
```

10、一个 td 里面不要放多个图片，请放在不同 td 里面，

```html
<table width="136" border="0" cellspacing="0" cellpadding="0">

<tr>

<td><img src="http://www.abc/123.jpg" width="140" height="123" border="0" ></img></td>

<td><img src="http://www.abc/456.jpg" width="140" height="123" border="0" ></img></td>

</tr>

</table>
```

11、`<img src="http://444/edm1_03.jpg" width="570" height="52" border="0" />`

12、不规范的换行会让图片丢失

=========================以下是我自己总结的===========================

1、 在 outlook 里面很多 css 属性不支持（比如：margin，overflow，text-overflow 等）

这是查看各种邮箱属性支持情况的网站

https://www.campaignmonitor.com/css/

2、在 outlook 上不能用背景图片，不能用网络图片加载。

3、在 outlook 上图片设置的宽高是不管用的

4、在 outlook 上实现 dom 结构居中要用 align=center

5、outlook 会自动加大行距间距

6、页面元素之间有间距可能是 html 里面混入了\$nbsp;(也就是空格字符)

7、align=center 在不同的浏览器的不同的邮箱解读代码是不一样的，纠错能力也不一样，

8、 邮件里文字的居中就用 text-align，dom 的居中就用 align=center

9、Foxmail 里面要想实现超出的文本不折行，超出部分显示省略号，建议不要直接把文本放到 td 下面，而是在 td 下面创建 a 标签，把内容放到 a 标签下，然后相应样式写在 a 标签里。

10、Foxmail 不支持 https 的图片路径

11、有些邮箱上页面结构显示不正确的问题如果和以上内容无关的话建议采用别的 dom 结构来试试。

12、outlook 邮箱中，多个连续的” ”或\&nbsp;符号不受宽度样式的限制，会一直往后延伸。

目前前端没有找到解决办法。

PS:

在浏览器里面浏览正常的模板，不一定正常！！！，要用发送系统抓取模板新建任务，发送到邮箱用浏览或者用客户端查看，邮件客户端软件和邮箱服务商的 html 解析水平基本停留在 table 布局阶段，而且出于安全考虑也有很多禁忌，请使用 table+css 布局，用 Dreamweaver 修改模板后还要查看 html 代码部分，空连接，怪异的或者过多的 alt,title 值（建议不超过 30 个汉字），没有宽高的图片都会造成邮件显示错误.

建议的测试环境，操作系统 xp，win7，邮件客户端 outlook2007，outlookexpre，foxmail6.0 以上

Ie 下的 qq 邮箱，126 邮箱，hotmail 邮箱，搜狐邮箱，新浪邮箱等等

有精力的话可以再在火狐下面再用以上邮箱测试测试

很多人用 outlook2007，所以要着重测试.

参考资料

1、http://blog.csdn.net/sykent/article/details/8584637
