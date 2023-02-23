---
title: 浏览器上的优化
date: 2023-02-23 21:32:58
tags:
  - chrome
  - performance
---

## 浏览器渲染原理

浏览器在渲染页面前需要先构建出 DOM 树 与 CSSDOM 树

DOM 数据即文档对象模型，是由一个节点和对象组成的集合

CSSDOM 树即层叠样式对象模型，里面是 CSS 的对象集合

想要选渲染出页面，就需要将 DOM 树 与 CSSDOM 树结在一起，组个成渲染树

### 渲染阻塞

HTML、CSS、JavaScript都是会对渲染产生阻塞的资源，HTML是必需的（没有DOM还谈何渲染），但可以从CSS与JavaScript着手优化，尽可能地减少阻塞的产生。

#### 优化 CSS
如果可以让CSS资源只在特定条件下使用，这样这些资源就可以在首次加载时先不进行构建CSSOM树，只有在符合特定条件时，才会让浏览器进行阻塞渲染然后构建CSSOM树。
CSS的媒体查询正是用来实现这个功能的，它由媒体类型以及零个或多个检查特定媒体特征状况的表达式组成。

```html
<!-- 没有使用媒体查询，这个css资源会阻塞渲染  -->
<link href="style.css" rel="stylesheet">
<!-- all是默认类型，它和不设置媒体查询的效果是一样的 -->
<link href="style.css" rel="stylesheet" media="all">
<!-- 动态媒体查询， 将在网页加载时计算。
根据网页加载时设备的方向，portrait.css 可能阻塞渲染，也可能不阻塞渲染。-->
<link href="portrait.css" rel="stylesheet" media="orientation:portrait">
<!-- 只在打印网页时应用，因此网页首次在浏览器中加载时，它不会阻塞渲染。 -->
<link href="print.css" rel="stylesheet" media="print">
```

使用媒体查询可以让CSS资源不在首次加载中阻塞渲染，但不管是哪种CSS资源它们的下载请求都不会被忽略，浏览器仍然会先下载CSS文件


#### 优化 JS
当浏览器的 HTML 解析器遇到一个 script 标签时候会暂停构建 DOM

chrome 引擎会开始执行 JavaScript 脚本，直到执行结束后，浏览器才会从之前中断的地方恢复，继续构建 DOM

使用async可以通知浏览器该脚本不需要在引用位置执行，这样浏览器就可以继续构建DOM，JavaScript脚本会在就绪后开始执行，这样将显著提升页面首次加载的性能（async只可以在src标签中使用也就是外部引用的JavaScript文件）

## 使用浏览器 Performance 进行分析
[具体使用点击查看](https://juejin.cn/post/6844904148085768200#heading-19)
