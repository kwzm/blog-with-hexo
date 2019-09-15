---
title: React SPA 应用 hash 路由如何使用锚点
date: 2019-08-13 21:58:56
categories: [技术, 前端]
tags: [hash]
---

当我们在做 SPA 应用的时候，为了兼容老的浏览器（如 IE9）我们不得不放弃 HTML5 browser history api 而只能采用 hash 路由的这种形式来实现前端路由，但是因为 hash 被路由占据了，导致本来不是问题的锚点功能却成了一个不大不小的问题。

<!-- more -->

经过我自己的搜索目前有两种方式能够解决这个问题，为了能在 react 生态下面简单优雅的使用，我专门封装了一个锚点组件 [react-anchor-without-hash](https://github.com/kwzm/react-anchor-without-hash)，它使用了类似原生 a 标签的写法，并且可以支持滚动的距离和指定滚动的元素，尽可能的满足业务的需求。

不过在介绍这个组件之前，还是得先说一下两种基本的解决方案。

## 两种解决方案

### [scrollIntoView](https://developer.mozilla.org/en-US/docs/Web/API/Element/scrollIntoView)

scrollIntoView 方法可以让当前的元素滚动到浏览器窗口的可视区域内。  
它的使用方法如下：

```javascript
var element = document.getElementById("box");

element.scrollIntoView();
```

这个 api 兼容 IE8 及以上的浏览器，所以可以放心使用。

> 注：IE10 之前的 IE 浏览器部分支持，具体请查看[Can I Use](https://caniuse.com/#search=scrollIntoView)。

### [scrollTop](https://developer.mozilla.org/en-US/docs/Web/API/Element/scrollTop)

另一个方法是使用 scrollTop 这个 api，这个方法的兼容性也是比较好的，这个方法相比于 `scrollIntoView` 来说需要你自己定义要滚动的元素和要滚动的高度，虽然看起来麻烦一些，但是好处是自由度比较高，试想一下下面的场景：

- 你有一个 A 元素在 Content 里面，Content 设置了滚动，你想让 A 元素滚动到可视区域内。

![](https://user-gold-cdn.xitu.io/2019/8/7/16c6c4e9fe48ab52?w=1008&h=581&f=png&s=10173)

- 如果用 `scrollIntoView` 会变成下面这样，A 元素显示到整个浏览器视口的最上面，这样就会和 Header 重合，被遮挡住一部分。

![](https://user-gold-cdn.xitu.io/2019/8/7/16c6c4f25ceb07bb?w=1042&h=570&f=png&s=11575)

- 所以这时候需要使用 `scrollTop` 去设置 content 滚动距离，比如说是 60px，最后的效果就变成了我们想要的结果。

![](https://user-gold-cdn.xitu.io/2019/8/7/16c6c52765e274ca?w=1130&h=602&f=png&s=15439)

使用方式如下：

```javascript
const cont = document.querySelector("#container");
const a = document.querySelector("#a");

cont.scrollTop = a.offsetTop + 60;
```

## [react-anchor-without-hash](https://github.com/kwzm/react-anchor-without-hash)

以上两种方式如果想方便的在项目里面使用多少都需要封装一下，而且使用起来和原生的 a 标签形式也相差甚远。

但是如果是在 react 技术栈下，我们可以利用组件来封装一个类似 a 标签的锚点，这样在使用形式上，我们就能更接近于原生的方式，降低使用成本。

于是我写了这个 react 组件，兼容以上两种方案，让你能够非常简单的实现锚点，如果使用了该组件的话，上面的实现方式就会变成下面这样：

```javascript
import Anchor from 'react-anchor-without-hash'

// ......

const anchorProps = {
  type: 'scrollTop',
  container: '#container',
  interval: 60
}

<div id="container" style={{position: 'relative', overflow: 'scroll'}}>
  <Anchor name="a" {...anchorProps}>
    <div>
      <h2>This is a</h2>
      <div>There are some text...</div>
    </div>
  </Anchor>
</div>
```

这时候你只需要在页面的地址栏输入: `http://somehost/path/#hash?_to=a` 页面就会让 A 滚动到你想要的位置啦！

github：https://github.com/kwzm/react-anchor-without-hash

demo：https://kwzm.github.io/react-anchor-without-hash/

codesandbox: https://codesandbox.io/embed/react-anchor-without-hash-2xq2h

欢迎讨论和 Star！！！
