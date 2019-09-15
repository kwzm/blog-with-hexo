---
title: 记录前端单元测试和e2e测试踩过的坑【持续更新】
date: 2019-08-19 18:15:29
categories: [技术, 前端]
tags: [前端测试]
---

# 方案

- 单元测试：jest
- react 组件测试：enzyme
- e2e 测试： puppeteer

<!-- more -->

# 问题

1. **安装 puppeteer 需要设置 chromium 国内镜像**  
   https://github.com/cnpm/cnpmjs.org/issues/1246
2. **jest 和 puppeteer 结合使用的时候如何保持之前的 JSDOM 环境还能使用**  
   当设置 jest.config.js 为如下配置之后之前的 JSDOM 环境就没有了，取而代之的是 nodejs 执行环境，如果想要 e2e 测试和 单元测试共存的话，我的方案是设置了两个 jest 配置文件，一个用于配置单元测试，一个用于配置 e2e 测试。
   `javascript module.exports = { preset: 'jest-puppeteer', testMatch: ["**/__tests__/**/*.e2e.[jt]s?(x)"], }`
3. **如何在启动 devserver 之后启动 e2e 测试**
   我的 e2e 测试需要依赖本地启动一个 devserver，所以我需要等本地 server 启动之后再测试，找了半天终于在 github 上找到一个库 [start-server-and-test](https://github.com/bahmutov/start-server-and-test) 专门解决这个问题。
4. **如何在 puppeteer 里面是用 DOM 操作**
   其实这个不算是个坑，只不过新人对 puppeteer 可能不熟悉，其实 puppeteer 是提供了一套 api 专门去做 DOM 的操作的。  
   https://pptr.dev/#?product=Puppeteer&version=v1.19.0&show=api-pageselector
   https://stackoverflow.com/questions/55702626/why-i-got-document-is-not-defined-error-in-puppeteer
5. **如何依次执行单元测试和 e2e 测试**  
   可以试试这个库 [npm-run-all](https://github.com/mysticatea/npm-run-all) 它可以控制你的多个 npm script 任务并行或是串行执行。
