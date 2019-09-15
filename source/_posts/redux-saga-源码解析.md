---
title: redux-saga 源码解析
date: 2019-06-26 10:57:55
categories: [技术, 前端]
tags: [源码解读]
---

本篇解读是将 api 作为入口进行解读的，因为 redux-saga 的 api 过多尤其是 effect 创建器，所以这里只是挑了常用的 api 做解读。

<!-- more -->

## 版本

1.0.3

## 电子书

[https://lfesc.github.io/redux-saga](https://lfesc.github.io/redux-saga)

## github

[https://github.com/LFESC/redux-saga](https://github.com/LFESC/redux-saga)

## 概览

- [Middleware API](https://lfesc.github.io/redux-saga/middleware.html)
- [Effect 创建器](https://lfesc.github.io/redux-saga/effectCreators.html)
- [Effect 组合器](https://lfesc.github.io/redux-saga/effectCreators.html#race)
- [接口](https://lfesc.github.io/redux-saga/interfaces.html)
- [外部 API](https://lfesc.github.io/redux-saga/runSaga.html)

## API

- Middleware API
  - [createSagaMiddleware](https://lfesc.github.io/redux-saga/middleware.html#createsagamiddleware)
  - [middleware.run](https://lfesc.github.io/redux-saga/middleware.html#middleware-run)
- Effect 创建器
  - [call](https://lfesc.github.io/redux-saga/effectCreators.html#call)
  - [fork](https://lfesc.github.io/redux-saga/effectCreators.html#fork)
  - [put](https://lfesc.github.io/redux-saga/effectCreators.html#put)
  - [take](https://lfesc.github.io/redux-saga/effectCreators.html#take)
  - [cancel](https://lfesc.github.io/redux-saga/effectCreators.html#cancel)
  - [cancelled](https://lfesc.github.io/redux-saga/effectCreators.html#cancelled)
  - [delay](https://lfesc.github.io/redux-saga/effectCreators.html#delay)
- Effect 组合器
  - [race](https://lfesc.github.io/redux-saga/effectCreators.html#race)
  - [all](https://lfesc.github.io/redux-saga/effectCreators.html#all)
- 接口
  - [Task](https://lfesc.github.io/redux-saga/task.html)
  - [Channel](https://lfesc.github.io/redux-saga/channel.html)
  - [Buffer](https://lfesc.github.io/redux-saga/buffers.html)
  - [SagaMonitor](https://lfesc.github.io/redux-saga/sagaMonitor.html)
- 外部 API
  - [runSaga](https://lfesc.github.io/redux-saga/runSaga.html)
- 工具
  - [channel([buffer])](https://lfesc.github.io/redux-saga/channel.html)
  - [eventChannel(subscribe, [buffer])](https://lfesc.github.io/redux-saga/channel.html#eventchannel)
  - [buffers](https://lfesc.github.io/redux-saga/buffers.html)

## 参考资料

- [redux-saga 中文文档](https://redux-saga-in-chinese.js.org/)
- [redux-saga 英文文档](https://redux-saga.js.org/)
