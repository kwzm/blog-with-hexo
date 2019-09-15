---
title: redux、immutablejs和mobx性能对比（三）
date: 2018-11-01 18:44:24
categories: [技术, 前端]
tags: [redux, immutablejs, mobx]
---

四、我的结论

通过第三部分的数据数据分析，我觉得我们可以得到以下结论：

<!-- more -->

1. 无论是在开发环境还是测试环下页面的首次加载速度结果都是：redux>immutablejs>mobx，但是他们之间的差距并不是很大。
   10000 条-100000 条数据的页面加载时间的增量明显也高于 10000-1000 条数据的页面加载时间增量。

2. 无论是在开发环境还是生产环境下点击完成某个 todo 所需的页面渲染速度结果都是：mobx>immutablejs>redux，正好和页面的首次加载时间相反，但是它们之间的差距还是蛮大的，具体表现在：
   mobx 的渲染速度一骑绝尘大幅度领先其它两者，尤其是在开发环境下，而且数据量越多越明显。

3. immutablejs 和 redux 的差距在 1000 条和 10000 条数据时并不明显，但是在 100000 条数据的时候有了明显差距。
   mobx 在 1000 条到 100000 条数据的增量并不明显幅度很小，尤其是开发环境下，与此同时 redux 的增量要大于 immutablejs，immutablejs 大概处于它们俩之间。

从第三部分的统计图上我目前能看到的信息就是这么多，聪明的你没准能够得出更多有用的信息，所以我把我的测试数据以及统计原图的 git 仓库打在下面，你们可以尽情的下载查看分析：

https://github.com/kwzm/redux-immutablejs-mobx-performance-test

五、深入的思考

其实如果读到这里，我相信你和我一样虽然通过纸面的数据得到了想要的结论，但是脑海里还是有各种疑问：

为什么 mobx 的用户操作渲染速度会那么快
影响它们三者渲染速度的罪魁祸首又是谁
下面来让我通过一组图片来为你们解开这其中的缘由：

我选取了生产环境下 10000 条数据时，chrome 的 performance 面板上的 Summary 饼状图来作为实例

从上往下依次为 redux、immutablejs 和 mobx：

{% img /images/redux-immutable-mobx3/1.png %}
{% img /images/redux-immutable-mobx3/2.png %}
{% img /images/redux-immutable-mobx3/3.png %}

我先介绍一下饼状图里面各项的含义：

- 黄色(Scripting)：JavaScript 执行
- 紫色(Rendering)：样式计算和布局，即重排
- 绿色(Painting)：重绘
- 灰色(other)：其它事件花费的时间
- 白色(Idle)：空闲时间

从中我们可以看到虽然出了 Scripting 其它四项也有差异，但是差异并不大，最大的差异点还是 Scripting，也就是说 js 代码的执行时间才是影响三者性能的主要原因，那么为什么三者的 js 执行时间会有如此大的差异呢？为了解释这个问题，我在每个组件的 render 方法中添加了 console.log 语句，让我看看当点击操作发生后控制台输出了什么。

在此之前，我先简单介绍一下 todoList 的组件结构：

- App
  - TodoHeader
  - TodoList
  - TodoItem
  - TodoItem
  - ......
  - TodoFooter

从左往右依次是 redux、immutablejs 和 mobx：

从 console 面板我们可以看出 mobx 为什么 js 执行时间最短，因为它只有两个组件执行了 render 方法，两个必要的组件，而纵观其它两者都有些不必要的 render，虽然 react 的 diff 算法已经很快了，但是当数据量达到一定规模的时候，这种不必要的 render 会越积越多，造成了内存和 cpu 的性能浪费。

至于为什么它们三者对于同一个事件执行的 render 方法会如此不同，这个并不在本文的探讨范围之内，这个涉及到这三者的原理，请大家自行百度吧。

六、参考资料

1. 源码
   redux: https://github.com/kwzm/react-redux-todo  
   immutablejs：https://github.com/kwzm/react-redux-immutablejs-todo  
   mobx：https://github.com/kwzm/react-mobx-todo
2. 测试数据及图表:
   https://github.com/kwzm/redux-immutablejs-mobx-performance-test
3. 性能测试:  
   [全新 Chrome Devtools Performance 使用指南](https://segmentfault.com/a/1190000011516068)  
   [初探 performance-监控网页于程序性能](https://www.cnblogs.com/zhuyang/p/4789020.html)

七、最后的话

其实我之前也从网上想找些现成的资料，但是无奈没有找到，所以借由目前正在学习 immutablejs 和 mobx 之机，随便做下性能的测试，但是其实这方面我完全是小白，包括如何进行性能测试，如何分析数据，我都是第一次做，所以如果有哪块不正确还请您指出，我们共同学习，谢谢！
