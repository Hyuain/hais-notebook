---
title: FAQs：DOM
date: 2020-02-13 16:23:07
tags:
  - 入门
  - 收集
  - FAQ
categories:
  - [前端, 其他]
---

关于 DOM 的一些问题与答案。

<!-- more -->

# 事件委托

```js
ul.addEventListener('click', (e) => {
  if(e.target.tagName.toLowerCase() === 'li') {
    console.log('点击了 li')
  }
})
```

```js
const bindEvent = (element, eventType, selector, fn) => {
  element.addEventListener(eventType, e => {
    let el = e.target
    while (!el.matches(selector)) {
      if (el === element) {
        el = null
        break
      }
      el = el.parentNode
    }
    el && fn.call(el, e)
  })
}
```

# 用 mouse 事件写一个可拖拽的 div

```js
let dragging = false
let position = null

div.addEventListener('mousedown', (e) => {
  dragging = true
  position = [e.clientX, e.clientY]
})

document.addEventListener('mousemove', (e) => {
  if (!dragging) return
  const x = e.clientX
  const y = e.clientY
  const deltaX = x - position[0]
  const deltaY = y - position[1]
  const left = parseInt(div.style.left || 0)
  const top = parseInt(div.style.top || 0)
  div.style.left = left + deltaX + 'px'
  div.style.top = top + deltaY + 'px'
  position = [x, y]
})

document.addEventListener('mouseup', (e) => {
  dragging = false
})
```