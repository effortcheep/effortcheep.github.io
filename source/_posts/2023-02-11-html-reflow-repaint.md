---
title: 浏览器回流和重绘
date: 2023-02-11 12:57:16
tags:
  - html
  - 回流
  - 重绘
---

## 回流(Reflow)
当Render Tree中部分或全部元素的尺寸、结构、或某些属性发生改变时，浏览器重新渲染部分或全部文档的过程称为回流。

会导致回流的操作：
+ 页面首次渲染
+ 浏览器窗口大小发生改变
+ 元素尺寸或位置发生改变
+ 元素内容变化（文字数量或图片大小等等）
+ 元素字体大小变化
+ 添加或者删除可见的DOM元素
+ 激活CSS伪类（例如：:hover）
+ 查询某些属性或调用某些方法

一些常用且会导致回流的属性和方法：
+ clientWidth、clientHeight、clientTop、clientLeft
+ offsetWidth、offsetHeight、offsetTop、offsetLeft
+ scrollWidth、scrollHeight、scrollTop、scrollLeft
+ scrollIntoView()、scrollIntoViewIfNeeded()
+ getComputedStyle()
+ getBoundingClientRect()
+ scrollTo()

## 重绘(Repaint)
当页面中元素样式的改变并不影响它在文档流中的位置时（例如：color、background-color、visibility等），浏览器会将新样式赋予给元素并重新绘制它，这个过程称为重绘。


## 当输入网址后，到网页显示，其间发生了什么
+ 浏览器解析 URL
  http://web服务器/目录/../文件
  + http 访问数据的协议
  + // 后面 服务器名称
  + 资源路径
+ 对 URL 进行解析之后，浏览器确定了 Web 服务器和文件名，根据这些信息来生成 HTTP 请求消息了

+ 通过 DNS 服务器获取 真实 IP 地址

+ 通过 DNS 获取到 IP 后，就可以把 HTTP 的传输工作交给操作系统中的协议栈，应用程序(浏览器)通过调用 Socket 库

+ 建立链接  TCP 三次握手

+ 服务器 层层解析 把相应数据装在 HTTP 响应报 文里

+ 客户端收到后 交给浏览器去渲染页面

+ 最后，客户端要离开了，向服务器发起了 TCP 四次挥手，至此双方的连接就断开了