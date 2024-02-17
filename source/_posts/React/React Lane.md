---
title: React Lane
date: 2023-06-23 16:41:41
categories:
  - - 前端
tags:
  - FX
---

React 中用 `Lane` 表示优先级这一概念。`Lane` 实际上就是二进制数字类型，每种优先级用不同的位为 `1` 来表示，用二进制来表示优先级就可以很方便地进行二进制运算来表达 “集合” 或者说 “批” 的概念。因为除去符号位之后一共还可以有 31 位，所以一共有 31 个 `Lane`，比如：

```typescript
export const NoLane: Lane = 0b0000000000000000000000000000000
export const SyncLane: Lane = 0b0000000000000000000000000000010
export const InputContinuousLane: Lane = 0b0000000000000000000000000001000
export const DefaultLane: Lane =  0b0000000000000000000000000100000
```

一般来讲，除了 `NoLane` 以外，数字越小代表优先级越高。

除了 `Lane` 以外，React 中还有 `Lanes`，实际上就是一群 `Lane` 做按位或运算的结果，表示 `Lane` 的集合。

# requestUpdateLane

该函数位于 `react-reconciler/src/ReactFiberWorkLoop.js`，用于获取当前更新对应的优先级。

```typescript
export function requestUpdateLane(fiber: Fiber): Lane {
  // 特殊情况处理
  if ((fiber.mode & ConcurrentMode) === NoMode) {
    // Sync 模式
    return SyncLane
  } else if (
    // 在 render 阶段收到的更新
    (excutionContext & RenderContext) !== NoContext &&
    workInProgressRootRenderLanes !== NoLanes
  ) {
    return pickArbitaryLane(workInProgressRootRenderLanes)
  }
  
  // ...省略 transition 相关代码
  
  // 更新由特定的 React 内部函数触发，比如 flushSync，通过全局变量获取到当前上下文
  const updateLane: Lane = getCurrentUpdatePriority()
  if (updateLane !== NoLane) {
    return updateLane
  }
  
  // 更新由 React 外部触发，从宿主环境获取优先级
  // 对于浏览器环境来说，该函数会获取 window.event，然后根据事件类型来获取优先级
  // 首次渲染中，event 是 DOMContentLoaded，优先级为 DefaultLane
  const eventLane: Lane = getCurrentEventPriority()
  return eventLane
}
```

React 还提供了一些操作 `Lane` 和 `Lanes` 的方法，他们底层都是通过位运算实现的，具体请参考 [[React Reconciliation#Data Structures and Algorithms|React Reconciliation 的算法与数据结构章节]]。
