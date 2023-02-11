---
title: KVO
date: 2020-06-18 13:28
tags:
  - iOS
  - KVO
---

## **KVO**的本质

**KVO**--- key-value Observing 键值监听，可以用于监听某个对象属性的改变
**isa** NSKVONotifying_Class  是  Class 的一个子类
**Runtime**动态创建一个类
int key
xx = int
double key
xx = double
重写 set方法 内部调用 **Foundation** _NSSet**xx**ValueAndNotify()

还会重写
重写 -(CLass)class,  返回 的是 Class 而不是 NSKVONotifying_Class
-(void)dealloc,
-(BOOL)_isKOVA

**_NSSetIntValueAndNotify()** 相当于
[self willChangeValueForKey:@"key"]

[super setKey:key];

[self didChangeValueForKey:@"key"]

didChangeValueForKey 方法内部会通知监听器
observe observeValueForKeyPath: ofObject: change: content:



利用**RuntimeAPI**动态生成一个子类， 并且让 实例对象的 isa 指向这个全新的子类

当修改 实例对象的属性时，会调用 Foundation 的 _NSSet**XX**ValueAndNotify() 函数

willChangeValueForKey:
父类原来的setter
didChangeValueForKey
内部会出发监听器(Oberser)的监听方法(observe observeValueForKeyPath: ofObject: change: content:)




## 如何手动触发**KVO**
调用
willChangeValueForKey:
didChangeValueForKey: