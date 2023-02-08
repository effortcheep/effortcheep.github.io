---
title: 作用域和作用域链
date: 2023-02-08 21:43:50
tags:
  - js
---

## 作用域

常见的作用域有两种，全局作用域和函数作用域

ES6之后有块级作用域('{}')

## 作用域链

作用域链是由当前环境与上层执行环境的一系列变量对象组成的，它保证了当前执行环境对符合访问权限的变量和函数的有序访问。

```js
var a = 20

function test() {
  var b = a + 10

  function innerTest() {
    var c = 10
    return b + c
  }
  return innerTest()
}

test()
```
上面这个例子中，innerTest 的作用域链包含了三个变量对象。

```js
// innerTest 的执行上下文

innerTestEC = {
  VO: {...}, // 变量对象
  scopeChain: [VO(innerTest), VO(test), VO(global)],
  this: {},
}
```
可以用一个数组来表示作用域链的有序性。数组的第一项 scopeChain[0] 为作用域链的最前端，而数组的最后一项则为作用域链的最末端。所有作用域链的最末端都是全局变量对象。