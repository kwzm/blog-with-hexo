---
title: redux、immutablejs和mobx性能对比（一）
date: 2018-11-01 10:51:38
categories: [技术, 前端]
tags: [redux, immutablejs, mobx]
---

一、前言

关于 react 的性能优化，有很多方面可以去做，而其中避免重复渲染又是比较重要的一点，那么为什么 react 会重复渲染，我们又该如何避免重复渲染呢，关于这方面官方其实早有说明：[避免重复渲染](https://react.docschina.org/docs/optimizing-performance.html#%E9%81%BF%E5%85%8D%E9%87%8D%E5%A4%8D%E6%B8%B2%E6%9F%93)，这里我就不赘述了。这次我主要是想对目前应用比较多的两种解决方案进行一次性能对比，分别是 immutablejs 和 mobx，作为参考我把没有任何优化的 redux 也加入进来，对这三者在页面首次加载速度、用户点击执行一个操作的响应速度进行一系列的测试，最终根据测试结果得出结论。

<!-- more -->

二、采集数据

1、测试对象

测试对象很简单就是一个 todoList：

{% img /images/redux-immutable-mobx1/view.png %}

2、测试工具

（一）chrome 自带的 performance 工具，如下图所示：

{% img /images/redux-immutable-mobx1/performance.png %}
（二）[performance API](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/performance)

3、测试方法

（一）前提说明：

- 一共测试三组数据：1000 条、10000 条和 100000 条
- 分别测试开发环境和生产环境下的数据
- 分别记录这三组数据页面首次加载时间和点击完成某个 todo 页面渲染的时间，记录数量为 10 条
- 页面都运行在 chrome 的无痕模式下，避免插件的影响
- 每次测试前都要先清空缓存并硬性重新加载，避免缓存的影响
- 每次点击的 todo 条目保持一致（倒数第二条）
- chrome 的 performance 工具会影响到页面的渲染速度，实际使用的速度会快于测试的速度
- 虽然实际的处理函数式绑定在 click 事件上的，但是因为 focus 的时候内部的脚本也会执行，所以我们就统- 一从 focus 事件开始计算时间
- 因为 mobx 在设置了 propTypes 后 dev 环境和 prod 环境表现不一致（可以查看这个 issue），所以我们- 的代码统一取消了 propTypes 验证

（二）页面首次渲染时间测试：

- 执行清空缓存并硬性重新加载
- 在控制台上输入如下命令得到页面加载时间

  {% img /images/redux-immutable-mobx1/performance-api.png %}

（三）点击完成某个 todo 页面渲染时间测试：

- 执行清空缓存并硬性重新加载
- 点击 record 开启记录
- 点击完成某个 todo

  {% img /images/redux-immutable-mobx1/test-click-step1.png %}

- 点击 stop 结束记录

  {% img /images/redux-immutable-mobx1/test-click-step2.png %}

- 截取从 focus 事件开始一直到页面渲染完成这段时间

  {% img /images/redux-immutable-mobx1/test-click-step3.png %}

- 查看 summary 面板，用总时间减去 Idle（空闲时间）得到全部的渲染时间

  {% img /images/redux-immutable-mobx1/test-click-step4.png %}

通过以上方法反复测试就得到了测试数据，那么在第二篇里我奖对获得到的数据进行分析。

此乃作者原创作品，如需转载，请在标题标明【转载】并附上原文链接，谢谢。
