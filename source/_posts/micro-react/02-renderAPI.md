---
title: 实现render函数
date: 2023-02-28 15:26:00
tags:
  - React
  - render
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

export default render
```
