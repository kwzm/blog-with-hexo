---
title: redux、immutablejs和mobx性能对比（二）
date: 2018-11-01 15:14:07
categories: [技术, 前端]
tags: [redux, immutablejs, mobx]
---

三、分析数据

1、前提说明

我对测试出的 10 个数据摘除最大值与最小值，然后求平均值
根据平均值我绘制了一个曲线图一个柱状图
曲线图用于查看 1000-100000 的性能趋势
柱状图用于比较在相同条数下 redux、immutablejs 和 mobx 三者的性能差异

<!-- more -->

2、图表展示

（一）页面加载速度

开发环境页面首次加载时间柱状图
{% img /images/redux-immutable-mobx2/1.png %}
开发环境页面首次加载时间曲线图
{% img /images/redux-immutable-mobx2/2.png %}
生产环境页面首次加载时间柱状图
{% img /images/redux-immutable-mobx2/3.png %}
生产环境页面首次加载时间曲线图
{% img /images/redux-immutable-mobx2/4.png %}

（二）点击某个 todo 页面渲染速度

开发环境点击完成某个 todo 页面渲染时间柱状图
{% img /images/redux-immutable-mobx2/5.png %}
开发环境点击完成某个 todo 页面渲染时间曲线图
{% img /images/redux-immutable-mobx2/6.png %}
生产环境点击完成某个 todo 页面渲染时间柱状图
{% img /images/redux-immutable-mobx2/7.png %}
生产环境点击完成某个 todo 页面渲染时间曲线图
{% img /images/redux-immutable-mobx2/8.png %}

此乃作者原创作品，如需转载，请在标题标明【转载】并附上原文链接，谢谢。
