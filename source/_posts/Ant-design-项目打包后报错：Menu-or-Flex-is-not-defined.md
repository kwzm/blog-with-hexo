---
title: Ant design 项目打包后报错：Menu(or Flex) is not defined
date: 2018-05-18 11:17:11
categories: [技术, 前端]
tags: [ant-design]
---

我的项目使用了 ant-design 和 ant-design-mobile，在测试环境上没问题，但是打包发布之后控制台报错

<!-- more -->

Menu is not defined

Flex is not defined
经过一番查找，终于发现问题的原因：

我在代码中使用 Menu 和 Flex 组件的方式是这样的：

```javascript
<Menu.Item>xxxx</Menu.Item>
<Flex.Item>xxxx</Flex.Item>
```

打包的时候使用了 [babel-react-optimize](https://github.com/jamiebuilds/babel-react-optimize)

这个库包含四个子库，会对 react 代码进行优化，可能是因为其中某个子库对带点的标签如：`<Menu.Item>`无法识别导致的，具体是哪个我还没找到，因为官方提供的文档也没有说明。

解决方案：

```javascript
const MenuItem = Menu.Item
const FlexItem = Flex.Item

<MenuItem>xxx</MenuItem>
<FlexItem>xxx</FlexItem>
```
