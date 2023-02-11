---
title: 宏队列，微队列
date: 2023-02-11 12:53:51
tags:
  - js
  - 宏任务
  - 微任务
---

## 宏队列
+ dom事件回调
+ ajax请求回调
+ 定时器回调
+ close时间
+ IO时间

## 微队列
+ Promise 回调
+ proccess.nextTick
+ queueMicrotask
+ mutation 回调

## 任务执行顺序
同步任务 > 微队列 > 宏队列
+ js 引擎首先必须执行所有的初始化同步任务代码
+ 每次准备取出第一个`宏任务`执行前，都要将所有的`微任务`一个一个取出来执行

