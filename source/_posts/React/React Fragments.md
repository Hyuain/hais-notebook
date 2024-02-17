---
title: React Fragments
date: 2020-02-02 16:25:06
categories:
  - - 前端
tags:
  - FX
---

类似于 `<template>`，使用 Fragments 可以创建类似的一个包裹器，而不会在 DOM 中添加额外的节点：

```jsx harmony
render() {
  return (
    <React.Fragment>
      <ChildA/>
      <ChildB/>
      <ChildC/>
    </React.Fragment>
  )
}
```

还有一个简写版本，简写版不支持使用 key 或其他属性：

```jsx harmony
render() {
  return (
    <>
      <ChildA/>
      <ChildB/>
      <ChildC/>
    </>
  )
}
```
