---
title: 实现并行渲染调度函数
date: 2023-02-28 15:28:00
tags:
  - React
  - Concurrent Mode
---

## render

```js
// render.js
function render(element, container) {

  // 创建 dom
  const dom = element.type === 'TEXT_ELEMENT'
    ? document.createTextNode(element.nodeValue)
    : document.createElement(element.type)

  // 设置 dom 属性
  const isProperty = key => key !== 'children'
  Object.keys(element.props)
    .filter(isProperty)
    .forEach(key => {
      dom[key] = element.props[key]
    })

  // 遍历创建子元素
  element.props.children.forEach(child => {
    render(child, dom)
  })

  // 添加 dom 元素 到 父节点
  container.append(dom)
}


let nextUnitOfWork = null

// 调度函数
function workLoop(deadLine) {

  // 应该终止
  let shouldYield = false
  where (nextUnitOfWork && !shouldYield) {
    // 做工作
    nextUnitOfWork = preformUnitOfWork(
      nextUnitOfWork
    )
    // 判断后续是否还有空闲时间
    shouldYield = deadLine.timeRemaining() < 1
  }

  // 空闲时间不足  会在浏览器下一次空闲时候执行
  requestIdleCallback(workLoop)
}

// 首次调用
requestIdleCallback(workLoop)

//
function preformUnitOfWork() {

}

export default render
```
