---
title: VUE中使用pros传递参数的一些细节
date: 2023-02-08 19:42:01
tags:
  - vue
  - props
---

## 使用一个对象绑定多个 prop
如果你想要将一个对象的所有属性都当作 props 传入，你可以使用没有参数的 v-bind，即只使用 v-bind 而非 :prop-name。例如，这里有一个 post 对象：

```js
export default {
  data() {
    return {
      post: {
        id: 1,
        title: 'My Journey with Vue'
      }
    }
  }
}
```
以及下面的模板：

```template
<BlogPost v-bind="post" />
```
而这实际上等价于：

```template
<BlogPost :id="post.id" :title="post.title" />
```

## 单向数据流
所有的 props 都遵循着单向绑定原则，props 因父组件的更新而变化，自然地将新的状态向下流往子组件，而不会逆向传递。这避免了子组件意外修改父组件的状态的情况，不然应用的数据流将很容易变得混乱而难以理解。

另外，每次父组件更新后，所有的子组件中的 props 都会被更新到最新值，这意味着你不应该在子组件中去更改一个 prop。若你这么做了，Vue 会在控制台上向你抛出警告：

```js
export default {
  props: ['foo'],
  created() {
    // ❌ 警告！prop 是只读的！
    this.foo = 'bar'
  }
}
```
导致你想要更改一个 prop 的需求通常来源于以下两种场景：

1. prop 被用于传入初始值；而子组件想在之后将其作为一个局部数据属性。在这种情况下，最好是新定义一个局部数据属性，从 props 上获取初始值即可：

```js
export default {
  props: ['initialCounter'],
  data() {
    return {
      // 计数器只是将 this.initialCounter 作为初始值
      // 像下面这样做就使 prop 和后续更新无关了
      counter: this.initialCounter
    }
  }
}
```

2. 需要对传入的 prop 值做进一步的转换。在这种情况中，最好是基于该 prop 值定义一个计算属性：

```js
export default {
  props: ['size'],
  computed: {
    // 该 prop 变更时计算属性也会自动更新
    normalizedSize() {
      return this.size.trim().toLowerCase()
    }
  }
}
```

## 更改对象 / 数组类型的 props
当对象或数组作为 props 被传入时，虽然子组件无法更改 props 绑定，但仍然可以更改对象或数组内部的值。这是因为 JavaScript 的对象和数组是按引用传递，而对 Vue 来说，禁止这样的改动虽然可能，但有很大的性能损耗，比较得不偿失。

这种更改的主要缺陷是它允许了子组件以某种不明显的方式影响父组件的状态，可能会使数据流在将来变得更难以理解。在最佳实践中，你应该尽可能避免这样的更改，除非父子组件在设计上本来就需要紧密耦合。在大多数场景下，子组件应该抛出一个事件来通知父组件做出改变。