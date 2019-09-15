---
title: "gulp 打包报错：Error: libsass bindings not found. Try reinstalling node-sass"
date: 2018-11-23 16:58:07
categories: [技术, 前端]
tags: [gulp]
---

看了网上很多帖子

- 有说切换 node 版本的
- 有说卸载重新装 gulp-sass 的
- 有说删除 node_modules 重新 install 的
  但是我测试了下在我们的电脑环境下都不行，后来找到一个可以打包不报错的机子，然后按照它的环境装了一下，就可以了，

  <!-- more -->

  我把成功的环境贴一下，以供日后参考：

- node@4.6.0
- gulp@3.9.1
- gulp-sass@3.1.0
  之前的 gulp-sass 是 1.3.3 版本，所以怎么都不行，不知道是不是版本的原因，先记录一下以后有时间再研究吧。
