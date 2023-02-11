---
title: RunLoop
date: 2020-05-24 21:28
tags:
  - iOS
  - RunLoop
---

## 什么是**RunLoop**
+ 运行循环
+ 在程序运行过程中循环做一些事情
    + 某种 mode 下的 Source0/Source1/Timer/Observer
+ 保持程序的持续运行
+ 节省CPU资源，提高程序性能：该做事时做事，该休息时休息
+ 应用范围
    + 定时器(Timer)、PerformSelector
    + GCD Async Main Queue
    + 事件响应、手势识别、界面刷新
    + 网络请求
    + AutoreleasePool

## **RunLoop**对象
+ iOS中有2套API来访问和使用RunLoop
    + Foundation: NSRunLoop
    + Core Foundation: CFRunLoopRef
+ NSRunLoop 和 CFRunLoopRef 都代表着RunLoop对象
    + NSRunLoop是基于CFRunLoopRef的一层OC包装

## **RunLoop**与线程
+ 每条线程都有唯一的一个与之对应的**RunLoop**对象
+ **RunLoop**保存在一个全局的Dictionary里，线程作为key，RunLoop作为value
+ 线程刚开始创建时并没有RunLoop对象，RunLoop会在第一次获取它时创建([NSRunLoop currentRunLoop])
+ RunLoop会在线程结束时候销毁
+ 主线程的RunLoop已经自动获取(创建)，**子线程默认没有开启RunLoop**

## RunLoop相关的类
+ Core Foundation 中关于RunLoop的5个类
+ CFRunLoopRef
+ CFRunLoopModeRef
+ CFRunLoopSourceRef    事务
+ CFRunLoopTimerRef     定时器
+ CFRunLoopObServerRef  监听器

```objective-c
// CFRunLoopRef
typedef struct __CFRunLoop * CFRunLoopRef;
struct __CFRunLoop {
    pthread_t _pthread;
    CFMutableSetRef _commonModes;
    CFMutableSetRef _commonModeItems;
    CFRunLoopModeRef _currentMode;  // 当前 mode
    CFMutableSetRef _modes;   // 存放 CFRunLoopModeRef 的集合
}

// CFRunLoopModeRef
typedef struct __CFRunLoopModeRef * CFRunLoopModeRef;
struct __CFRunLoopModeRef {
    CFStringRef _name;
    CFMutableSetRef _sources0;   // 存放 CFRunLoopSourceRef 的集合
    CFMutableSetRef _sources1;   // 存放 CFRunLoopSourceRef 的集合
    CFMutableArrayRef _obervers; // 存放 CFRunLoopObServerRef 的集合
    CFMutableArrayRef _timers;   // 存放 CFRunLoopTimerRef 的集合
}
```
### CFRunLoopModeRef
+ CFRunLoopModeRef代表RunLoop的运行模式
+ 一个RunLoop包含若干个Mode，每个Mode又包含若干个Source0/Source1/Timer/Observer
+ RunLoop启动时只能选择其中一个Mode，作为currentMode
+ 如果需要切换Mode，只能退出当前Loop，再重新选择一个Mode进入
    + 不同组的Source0/Source1/Timer/Observer能分隔开来，互不影响(普通模式和滚动模式)
+ 如果Mode里没有Source0/Source1/Timer/Observer，RunLoop就会立马退出
+ 常见的2中Mode
    + KCFRunLoopDefaultMode(NSDefaultRunLoopMode)：App的默认Mode，通常主线程是在这个Mode下运行
    + UITrackingRunLoopMode：界面跟踪Mode，用户ScrollView追中触摸滑动，保证界面滑动时不受其他Mode影响
+ kCFRunLoopCommonModes 默认包括 KCFRunLoopDefaultMode 和 UITrackingRunLoopMode

### CFRunLoopSourceRef
#### Source0
+ 触摸事件处理(__CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE0_PERFORM_FUNCTION__)
+ performSelector:onThread:

#### Source1
+ 基于Port(端口)的线程通信(多线程通信)
+ 系统事件捕捉(事件通过Source1捕捉，分发到Source0处理)

### CFRunLoopTimerRef
+ NSTimer
+ performSelector:withObject:afterDelay:

### CFRunLoopObServerRef
+ 用于监听RunLoop的状态
+ UI刷新(BeforeWaiting)
+ autorelease pool(BeforeWaiting)

## RunLoop 状态
+ kCFRunLoopEntry             即将进入Loop
+ kCFRunLoopBeforeTimers      即将处理Timer
+ kCFRunLoopBeforeSources     即将处理Sources
+ kCFRunLoopBeforewaiting     即将进入休眠
+ kCFRunLoopAfterwaiting      刚从休眠中唤醒
+ kCFRunLoopExit              即将退出Loop


## RunLoop的运行逻辑
1. 通知Observers：进入Loop
2. 通知Observers：即将处理Timers
3. 通知Observers：即将处理Sources
4. 处理Blocks
5. 处理Source0
6. 如果存在Source1，跳转到第8步
7. 通知Observers：开始休眠(等待消息唤醒)    // 线程阻塞 通过**match_msg** 内核层面**API**
8. 通知Observers：结束休眠(被某个消息唤醒)
    8.1 处理Timer
    8.2 处理GCD Async To Main Queue
    8.3 处理Source1
9.  处理Blocks
10. 根据前面的执行结果，决定如果操作
     10.1 有可能回到第2步
     10.2 退出Loop
11. 通知Observers：退出Loop







