---
title: react+mobx实现兼容IE9的CMS系统过程中的经验总结
date: 2019-07-01 18:16:41
categories: [技术, 前端]
tags: [mobx]
---

## 前言

因为想要尝试用 mobx 并且也想自己试着搭建脚手架，所以在开发此项目时并没有使用比较成熟的脚手架如：ant-design-pro，所以在开发过程中遇到了很多问题，在此记录，希望能帮助到其它开发者。

<!-- more -->

## 框架

经过此项目我将这个项目的脚手架开源，欢迎大家使用和提建议。  
[react-mobx-react-router-boilerplate](https://github.com/kwzm/react-mobx-react-router-boilerplate)

## 技术选型

- create-react-app@2.1.1
- react@16.6.0
- mobx@4.5.2（5.x 版本不支持 IE9）
- mobx-react@5.3.6
- react-router-dom@4.3.1
- antd 由@2.x 到@3.18.1
- superagent@4.0.0-beta.5（axios 最新版本不支持 IE9）

## 遇到的问题

### 浏览器兼容性

- IE9 不支持 flex 布局
- create-react-app 提供的 react-app-polyfill 只是对某些 ES6 的 API 做了兼容，所以用 @babel/polyfill 替代了 react-app-polyfill
- dev 环境不支持 IE9：在 IE9 上打开 dev 环境发现了很多问题，不仅有 webpack-dev-server 存在的兼容性问题，还有一些其它第三方库存在的兼容性问题，主要有以下问题（因为问题太多所以最后我放弃了 IE9 下的 dev 环境）：
  - 首先是 bundle.js 文件里报 const map 未定义是因为 webpack-dev-server 从 2.7.1 后不支持 IE9
  - 如果是 vendor.js 文件里报 setPrototypeOf 未定义是因为 create-react-app 引入的 react-dev-utils/webpackHotDevClient 文件里面有一个 chalk 的包使用了这个 ES6 的方法，而 @babel/polyfill 的编译是基于**proto**，而 IE9 并不支持**proto**，所 以@babel/polyfill 无效
- 调试的时候保证是 IE9 的标准模式
- 线上的项目无法在 IE9 上打开的原因：IE9 默认采用的文档模式是 IE7。解决方案：加入`<meta http-equiv="X-UA-Compatible" content="IE=edge">`，注意顺序：
  > The X-UA-Compatible header is not case sensitive; however, it must appear in the header of the webpage (the HEAD section) before all other elements except for the title element and other meta elements.
- IE9 Spin 元素包裹的内容无法点击 原因：Spin 设置的 css 属性 pointer-events 不兼容 IE9。解决方案：升级 antd
- IE9 设置 img height 为百分比并不能像 chrome 浏览器里一样按比例缩放，只能设置 width
- IE9 上 css 样式没有浏览器前缀导致样式错误，原因 autoprefixer 对应的 browserslist 没写对没能将 IE9 包含进去
- 默认使用 webkit 内核：`<meta name="renderer" content="webkit" />`

### mobx

- mobx-react observer 对传入的 props 做了浅比较，所以传递相同的值会有问题
- 为了在生产环境不把 mock 文件打包进来，我是这么做的
  - 首先添加了一个 npm script: `"start:no-mock": "cross-env REACT_APP_MOCK=none npm start"`
  - 其次在 index.js 文件里面：
  ```js
  if (
    process.env.NODE_ENV === "development" &&
    process.env.REACT_APP_MOCK !== "none"
  ) {
    import("../mock");
  }
  ```
- HMR 导致修改之后 mobx 的数据都变成初始数据了：react-hot-loader 遇到更新的时候会 rerender 组件，之前的 hot 挂载在 App 上，App 里面用了 provider 去传递数据，每次 rerender 都会重新传数据，所以数据就变成初始的了，hot 方法包裹的组件应该不包括它

### http

- ajax 无法截获 302 状态码
- 本地开发时的 cookie 校验问题：因为登录是用的服务器上现成的登录页面，权限校验是通过 cookie 来实现的，所以我需要设法让我的本地域名下也能保存到后台返回的 cookie，这样本地开发时才能校验成功，所以我通过 http-proxy-middle 设置了 /login 页面的代理，代理到测试服务器上的登录页面，这样登录成功后我本地域名下也有了 cookie。

  ```js
  const proxy = require("http-proxy-middleware");

  module.exports = function(app) {
    // 代理api请求
    app.use(
      proxy("/api", {
        target: process.env.REACT_APP_DEV_SERVER,
        pathRewrite: { "^/api": "" },
        changeOrigin: true
      })
    );
    // 代理登录、退出页面
    app.use(
      proxy(
        [process.env.REACT_APP_LOGIN_URL, "/public", "/auth/check", "/logout"],
        {
          target: process.env.REACT_APP_DEV_SERVER
        }
      )
    );
  };
  ```

### antd

- treeselect 组件，指定 disableCheckbox 会有 bug，详情见[https://github.com/ant-design/ant-design/issues/10360](https://github.com/ant-design/ant-design/issues/10360)，解决方案：给需要禁用复选框的元素添加 class，然后用 css 来实现禁用效果

### git

- git revert 后合并导致代码丢失的问题，这个我专门写了一篇[博客](https://www.cnblogs.com/kwzm/p/10288684.html)

### webpack

- 每次修改一个 chunk 如果和第三方库有关的代码，则会导致很多 chunk 打包的 hash 变化，这个是正常的，因为 webpack 的 splickChunk 结合 import() 的原因导致的
- hard-source-webpack-plugin 可以提高再次编译速度

### 其它

- textarea 里面输入带换行的文本在详情页里面没有体现出来，解决方案：1、替换'/r/n'为`<br/>`，2、定义`white-space: pre-wrap`
- 生产环境下出现样式覆盖的问题，因为动态加载导致，比如先访问 a 页面，加载 a.css，然后访问 b 页面，引入 b.css，b.css 后引入所以如果有重名的 class，就会覆盖 a.css 里面的样式，解决方案通过修改 css 来解决
- 新的页面会记录之前页面的滚动位置，因为滚动的样式定义在 route 外，所以 route 对应的组件即便卸载页不会影响外面的滚动条的位置，解决方案：[https://react-router.docschina.org/web/guides/scroll-restoration](https://react-router.docschina.org/web/guides/scroll-restoration)
- hash 路由无法用 hash 锚点，可以用 scrollIntoView 方法，但是在某些场景下会有问题，比如你设置了 header，你想让要显示的元素在 header 下方显示，但是由于 scrollIntoView 是根据浏览器视窗去滚动的所以它会忽略 header 这时你滚动的元素的一部分就可能被 header 遮挡，所以我自己封装了一个 react 组件[react-anchor-without-hasn](https://github.com/kwzm/react-anchor-without-hash)，可以定义滚动的 offset
- 搜索优化：一般我们对于用户输入搜索都会采用 debounce 进行优化减少服务端请求，但是假如用户输入的比较慢两次输入的字符在你设置的延迟时间之外，比如延迟间隔 500 你输入第一个字符 'k' 后隔了 500 再输入 'w' 这时会发送两条请求，我们知道 http 请求的响应时长不是固定的，如果第一次请求的响应时长比第二次请求的响应时长长就有可能发送第一次搜索结果覆盖第二次的搜索结果，这样就会给用户造成困扰。解决方法就是设置一个数组，存储每次请求的 cancel 函数(这块需要根据自己采用的 ajax 库)，在每次请求之前先判断数组是否为空，如果不为空则先依次调用数组里面的 cancel 方法终止之前的请求，再发送请求
