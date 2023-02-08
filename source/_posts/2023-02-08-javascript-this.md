---
title: this
date: 2023-02-08 21:54:56
tags:
  - js
  - this
---


## this
当函数被调用执行时，变量对象会生成，这个时候，this 的指向会确定。一个非常重要的结论，当前函数的 this 是在函数被调用执行的时候才确定的。如果当前的执行上下文处于函数调用栈的栈顶，那么这个时候变量对象会变成活动对象，同时this的指向确认。

在全局对象中，this 指向它本身。

在一个函数的执行上下文中，this 由该函数的调用者提供，由调用函数的方式来决定其指向。

如果调用者被某个对象所持有，那么在调用该函数时，内部的 this 指向该对象。
如果调用者函数独立调用，那么该函数内部的 this 指向 undefined，在非严格模式下，当 this 为 undefined 是，他会自动指向全局对象。

```js
// demo1
var a = 20

var obj = {
  a: 40
}

function fn() {
  console.log('fn this: ', this)

  function foo() {
    console.log(this.a)
  }

  foo()
}

fn.call(obj)
fn()

// fn this: Object { a: 40 }
// 20

// fn this: Window { }
// 20
```

这个例子中 fn 最终的调用方式不同，因此在 fn 的环境中，this 会有所变化。
但是无论 fn 如何调用，在 fn 执行时，foo 始终都是独立调用。
因此 foo 内部的 this 都是指向 undefined 的，但是由于这是非严格模式，因此自动转向 window。


```js
// demo2

var a = 20
var foo = {
  a: 10,
  getA: function() {
    return a
  }
}

console.log(foo.getA()) // 10

var test = foo.getA()
console.log(test()) // 20
```

foo.getA() 中，getA 为调用者，被 foo 所拥有，当 getA 执行时，this 指向 foo
test()执行是，test 为调用者，它是独立调用，当 getA 执行时候，this 指向 undefined ，自动转向 window



## call/apply/bind 显示修改 this

+ 当函数调用 call/apply 时，则表示会执行该函数，并且内部函数内部 this 指向 call/apply 的第一个参数

+ call/apply 的不同之处在于
  + call 的第一个参数是为函数内部指定 this 指向，后续的参数则是函数执行时所需要的参数，一个个传递
  + apply 的第一个参数是为函数内部指定 this 指向，而函数的参数，则以数组的形式传递，作为 apply 的第二个参数

+ 当函数调用 bind 是，函数不会立即执行，而是返回一个新的函数，这个新的函数与原函数有共同的函数体，但它并非原函数，并且新函数的参数与 this 指向都已经被绑定，参数为 bind 的后续参数

```js
function fn(num1, num2) {
  return this.a + num1 + num2
}

var a = 20
var object = { a: 40 }

var _fn = fn.bind(object, 1, 2)
console.log(_fn === fn) // false

_fn() // 43

_fn(1, 4) // 43  因为参数已经绑定，因此重新传入参数是无效的
```


## 箭头函数 this

一般是全局对象

如果被普通函数包裹，则是上一层 定义时的 this 不会随调用发生改变