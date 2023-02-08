---
title: 变量提升
date: 2023-02-08 20:43:21
tags:
  - js
---

## 变量提升
> 变量提升（Hoisting）被认为是，Javascript中执行上下文（特别是创建和执行阶段）工作方式的一种认识。

变量提升是指在 JavaScript 代码执行过程中，JavaScript 引擎把变量的声明部分和函数的声明部分提升到代码开头的行为。变量被提升后，会给变量设置默认值为 undefined。

ES6 支持新的变量生命方式 let/const, 规则与 var 完全不同，它们是在上下文的执行阶段开始执行的，避免了变量提升带来的一系列问题。

```js
console.log(a)
var a = 10
```
相当于

```js
// 创建阶段
var a = undefined

// 执行阶段
console.log(a)
a = 10
```

在变量对象的创建过程中，函数声明的执行优先级会比变量声明的优先级更高一点，而且同名的函数会覆盖函数与变量，但是同名的变量并不会覆盖函数。

但是在执行上下文阶段，同名的函数会被变量重新赋值。

```js
var a = 20
function fn() { console.log('fn') }
function fn() { console.log('cover fn') }
function a() { console.log('cover a') }

console.log(a)
fn()

var fn = 'I want cover function named fn'
console.log(fn)

// 20
// cover fn
// I want cover function named fn
```

上面例子的执行顺序其实为：
```js
// 创建阶段
function fn() { console.log('fn') }
function fn() { console.log('cover fn') }
function a() { console.log('cover a') }
var a = undefined
var fn = undefined

// 执行阶段
a = 20
console.log(a) // 20
fn() // cover fn

fn = 'I want cover function named fn'
console.log(fn) // I want cover function named fn
```


在 ES6 之前，ECMAScript 没有块级作用域
```js
var array = [1, 2, 3, 4, 5, 6]

for (var i = 0; i < array.length; i++) {
  console.log(i)
}
console.log(i)// i === 6

```