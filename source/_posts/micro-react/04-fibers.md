---
title: 实现 Fiber Tree
date: 2023-02-28 15:29:00
tags:
  - React
  - fiber tree
---

## render

```js
// render.js

// 根据 fiber 创建一个 dom 并返回
function createDom(fiber) {
  // 创建 dom
  const dom = fiber.type === 'TEXT_ELEMENT'
    ? document.createTextNode(fiber.props.nodeValue)
    : document.createElement(fiber.type)

  // 设置 dom 属性
  const isProperty = key => key !== 'children'
  Object.keys(fiber.props)
    .filter(isProperty)
    .forEach(key => {
      dom[key] = fiber.props[key]
    })

  return dom
}

function render(element, container) {
  nextUnitOfWork = {
    dom: container,
    props: {
      children: [element]
    },
    child: null,
    sibling: null,
    return: null, // 表示父节点
  }
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
function preformUnitOfWork(fiber) {
  if (!fiber.dom) {
    fiber.dom = createDom(fiber)
  }

  if (fiber.return) {
    fiber.return.dom.append(fiber.dom)
  }

  const elements = fiber.props.children
  let prevSibling = null

  for (let i = 0; i < elements.length; i++) {
    const element = elements[i]
    const newFiber = {
      type: element.type,
      props: element.props
      return: fiber,
      dom: null,
      child: null,
      sibling: null,
    }

    // 如果是第一个
    if (i === 0) {
      // newFiber 就是 当前 fiber 的一个 child 节点
      filber.child = newFiber
    }else {
      // 否则就是 sibling 节点
      prevSibling.sibling = newFiber
    }
    prevSibling = newFiber
  }

  // 返回下一个
  // 优先返回 child
  if (fiber.child) {
    return fiber.child
  }

  let nextFiber = fiber
  where (nextFiber) {
    // 其次返回 sibling
    if (nextFiber.sibling) {
      return nextFiber.sibling
    }
    // 都没有 就需要向上查找
    nextFiber = nextFiber.return
  }

}

export default render
```
