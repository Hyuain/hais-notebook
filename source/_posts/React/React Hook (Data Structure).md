---
title: React Hook (Data Structure)
date: 2023-06-23 16:41:41
categories:
  - - 前端
tags:
  - FX
---

> 本文介绍的是 React 源码中使用的数据结构 `Hook`，他是 React Hooks 特性在源码中实现所依赖的数据结构基础，每次我们在开发中使用 Hook 的时候，都会用到这个数据结构。
> 
> 关于 React Hooks 特性在普通前端开发中的介绍与使用请参阅 [[React Hooks|这篇文章]]。
> 
> 关于 `Hook` 数据结构是怎样使用的，请参阅 [[React Reconciliation|Reconciliation]]。

React 的 `FunctionComponentFiber` 中的 `memorizedState` 属性指向的即为该函数组件的第一个 `Hook`，一个函数组件内部的 Hooks 会组成一个链表，他们由 `.next` 链接起来。

**注意当前 React 源码中 `ReactFiberHooks.js` 中单独定义了 `Hook` 的 `Update` 和 `UpdateQueue`，这跟 [[React Fiber|Fiber]] 中的 [[React Update#Update|Update]] 和 [[React Update#UpdateQueue|UpdateQueue]] 思路类似但结构有所不同！**  

```typescript
export type Hook = {
  memorizedState: any
  baseState: any
  baseQueue: Update<any, any> | null
  queue: any
  next: Hook | null
}

export type Update<S, A> = {
  lane: Lane
  // Fiber 中 Update 的更新信息主要由 payload 携带，而这里主要是 action
  action: A
  next: Update<S, A>
}

export type UpdateQueue<S, A> = {
  pending: Update<S, A> | null
  lanes: Lanes
  dispatch: (A => any) | null
  lastRenderedReducer: ((S, A) => S) | null
  lastRenderedState: S | null
}
```
