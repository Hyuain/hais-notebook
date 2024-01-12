---
title: Regular Expression
date: 2020-01-29 17:22:52
categories:
  - - 前端
tags:
  - FX
---

# 用正则实现 `trim()`

```js
const trim = (string) => {
  return string.replace(/^\s+|\s+$/g, '')
}
```
