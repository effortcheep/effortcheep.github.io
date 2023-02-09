---
title: 原型链
date: 2023-02-09 20:56:41
tags:
---


## prototype

每一个函数都有一个 prototype 属性，指向了实例的原型（\_\_proto\_\_）


## \_\_proto\_\_

每一个 js 对象（除了 null ）都有一个属性 \_\_proto\_\_，指向对象的原型，即函数的 prototype

每个原型都有一个 constructor 属性指向关联的构造函数


## 如何模拟 new 的创建

1. 创建一个新的、空的实力对象 obj

2. 将 obj.\_\_proto\_\_ 指向构造函数的 prototype 链接

3. 将构造函数内部的 this，修改为实例

4. 返回实例 obj

如果使用 PPrototype 指代原型对象，那么构造函数 Person、原型 PPrototype、实例 p1，p2 之间的关系如下

// -> 表示指向
Person.prototype -> PPrototype
p1.\_\_proto\_\_ -> PPrototype
p2.\_\_proto\_\_ -> PPrototype
PPrototype.constructor -> Person


## Function

+ Function 是最顶层的构造器

+ Function 是 JavaScript 里最顶层的构造器，它构造了系统中的所有对象，包括用户自定义对象，系统内置对象，甚至包括它自己

+ Function.\_\_proto\_\_ === Function.prototype


## Object 是最顶层的对象

+ 所有对象都继承 Object 的原型

+ Object 是 Function 构造出来

+ Object.\_\_proto\_\_ === Function.prototype

+ Object.prototype.\_\_proto\_\_ === null


## 函数的 \_\_proto\_\_

函数也是对象，也有 \_\_proto\_\_ 指向 Function.prototype

## 函数的 prototype.\_\_proto\_\_

函数.prototype，本质上是和 var obj = {} 是一样的，都是对象，是由 new Object 创建的

即 函数.prototype.\_\_proto\_\_ === Object.prototype
