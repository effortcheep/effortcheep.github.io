---
title: 小程序
date: 2023-02-23 21:49:22
tags:
  - 小程序
  - 前端
---

## 微信小程序的双线程模型

小程序的双线程指的就是渲染线程和逻辑线程，这两个线程分别承担UI的渲染和执行 JavaScript 代码的工作。

渲染层的界面使用了WebView 进行渲染；逻辑层采用 JsCore 线程运行 JS 脚本。一个小程序存在多个界面，所以渲染层存在多个 WebView 线程，这两个线程的通信会经由微信客户端做中转，逻辑层发送网络请求也经由 Native 转发

为什么不采用 Web Worker 的方式 主要是性能问题  Web Worker 通信方式 要比 Native 转发更加 耗费性能

另一个 shadow Dom 兼容性很差

小程序 事件驱动的通信方式

渲染成与逻辑层并不是直接传递数据和事件的，而是由 Native 作为中间媒介进行转发

UI -> event -> jscore -> tranform data(逻辑、接口调用、数据请求) -> setData(render) -> UI

setData 主要做了两件事

+ 改变逻辑层中目标属性的值

+ 通过微信客户端(Native)通知到渲染层将 目标属性的值 更新到视图

往下延伸

也就是 不在 UI 层展示的数据 完全可以不用 setData 方法去更新值

减少多次 setData 调用


[具体查看](https://developers.weixin.qq.com/miniprogram/dev/framework/MINA.html)

[具体查看](https://developers.weixin.qq.com/miniprogram/dev/framework/quickstart/framework.html#%E6%B8%B2%E6%9F%93%E5%B1%82%E5%92%8C%E9%80%BB%E8%BE%91%E5%B1%82)


## 小程序用户认证体系 OAuth2.0

[OAuth2.0](https://zhuanlan.zhihu.com/p/509212673)


## 小程序优化

###

### 图片资源

大图资源可放在云服务上优化

对于小图可采用 svg 格式 矢量图 保证图片质量的同时，减少资源大小

### setData

减少 setData 的调用

setData 在这里主要做了两件事情：

+ 改变了逻辑层中这个目标属性的值

+ 通过微信客户端发起通信，通知到渲染层

在 setData 使用时 需要注意两件事情：

+ 不要把页面渲染无用的数据也放置在里面，夹带无用的内容是浪费资源的表现；

+ 不要应用在一些过大数据中，几个大数据叠加在一起可能超过了一次通信的上限，导致失败


## 小程序的冷启动、热启动概念

[具体查看](https://developers.weixin.qq.com/miniprogram/dev/framework/runtime/operating-mechanism.html)

## 小程序生命周期

### APP 整个小程序

每个小程序都需要在 app.js 中调用 App 方法注册小程序实例

App 接受一个 Object 参数，其指定小程序的生命周期回调等

+ onLaunch(Object object)
小程序初始化完成时触发，全局只触发一次

+ onShow(Object object)
小程序启动，或从后台进入前台显示时触发

+ onHide()
小程序从前台进入后台时触发

### Page 小程序每个页面
对于小程序中的每个页面，都需要在页面对应的 js 文件中进行注册，指定页面的初始数据、生命周期回调、事件处理函数等

简单的页面可以使用 Page() 进行构造

+ onLoad(Object query)
页面加载时触发。一个页面只会调用一次，可以在 onLoad 的参数中获取打开当前页面路径中的参数

+ onShow()
页面显示/切入前台时触发

+ onReady()
页面初次渲染完成时触发。一个页面只会调用一次，代表页面已经准备妥当，可以和视图层进行交互

+ onHide()
页面隐藏/切入后台时触发

+ onUnload()
页面卸载时触发


## 全局 App 对象

getAPP 函数用来获取到小程序全局唯一的 App 实例

可以用于页面传递参数

不要在定义于 App() 内的函数中，或调用 App 前调用 getApp() ，使用 this 就可以拿到 app 实例。
