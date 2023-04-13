---
title: 实现createElement函数
date: 2023-02-28 15:25:00
tags:
  - React
  - createElement
---

## CreateElement

```js
// createElement.js
function createElement(type, props, ...children) {
  return {
    type,
    props: {
      ...props,
      children: children.map(child => {
        return (typeof child === 'object') ? child : createTextElement(child)
      })
    }
  }
}

// text 文本节点
function createTextElement(text) {
  return {
    type: 'TEXT_ELEMENT',
    props: {
      nodeValue: text,
      children: [],
    },
  }
}

export default createElement
```


```js
// main.js
import { createElement, render } from 'micro-react'

// 创建 dom  节点
const element = createElement({
  type: 'h1',
  { class: 'custom', id: 1, style: 'background: yellow'},
  'Hello Word',
  createElement('h2', null, 'Effort')
})

// 渲染 dom
const container = document.querySelector('#root')
render(element, container)
```