---
title: React Fiber
date: 2023-06-23 16:41:41
categories:
  - - 前端
tags:
  - FX
---

# Fiber

> `Fiber` 就是 Component 上的需要被执行的、或已经执行完成的工作，每个 Component 可能有多个 `Fiber`。
>
> ——React 源码中的注释

`Fiber` 可以暂时先理解为虚拟节点，他有不同的类型，比如 `FunctionComponent` `ClassComponent` `HostRoot` `HostText` `Fragment` 等。也就是说，一个 `Fiber` 节点可以对应某个 React 组件或某个 DOM 节点等。

React 中的很多过程都是使用深度优先遍历（DFS）来实现的：比如生成 FiberTree（暂时可以理解为 VirtualDOMTree）、将 FiberTree 的内容提交到宿主环境 UI 中、遍历 FiberTree 执行 Effects 等等。具体可参考 [[React Reconciliation#Data Structures and Algorithms|React Reconciliation 的数据结构与算法章节]]。

需要注意的是：**如果使用递归实现 DFS 的过程，那么 React 将无法控制调用栈的随时中断与继续。** 因此 React 选择借助 FiberTree 并使用迭代的方式实现 DFS，并在中间的某些环节设置了可以提前中断的手段。同时 `Fiber.updateQueue` 等属性提供了记忆并恢复工作状态的能力，使得继续之前被中断的任务成为可能。

> 不过，由于 React 源码使用了 Flow，其实可能某个 `Fiber` 类型并不需要某些属性，但 React 并没有将他们拆分开来，而是都直接散开放在了 `Fiber` 这一个数据类型中。

```typescript
export type Fiber = {
  /* Instance 相关的属性 */
  // Fiber 的类型（FunctionComponent、ClassComponent、HostRoot、HostComponent 等）
  tag: WorkTag
  // 唯一标识符
  key: null | string
  // element.type，用于在 Reconciliation 过程中保持该 Fiber 的特征
  elementType: any
  // 对于函数组件来说，就是函数本身；对于类组件来说，是类本身
  type: any
  // HostRoot 将会指向 FiberRoot，HostComponent 会指向 DOM Element
  stateNode: any
  
  /* Fiber 相关的属性 */
  // 对于 Instance 来说，return 就是 parent
  // 但 Fiber 除了 Instance 之外，还可以作为工作单元，这时相当于是模拟调用栈，return 存放了该 Fiber 执行完之后需要返回的地方
  return: Fiber | null
  // 单链表树形结构
  child: Fiber | null
  sibling: Fiber | null
  index: number
  
  ref: any
  refCleanup: null | (() => void)
  
  /* 用于处理 Fiber 得到输出的属性 */
  // 用于处理该 Fiber 的外部输入（Props、Arguments）
  pendingProps: any   // 当次传入的输入
  memorizedProps: any // 上次的输入
  // 内部状态更新和回调的列表，不同类型的 Fiber 存的东西也不一样
  updateQueue: unknown
  // 上一次的内部状态
  memorizedState: any
  
  /* 副作用相关的属性 */
  // 标记此次 render 都需要执行哪些副作用，比如往 DOM 添加节点、删除节点、执行 useEffect 等
  flags: Flags
  // 子结点都有什么副作用
  subtreeFlags: Flags
  // 用于标记哪些节点需要删除
  deletions: Fiber[] | null
  
  // 双缓存机制，指向另一棵 Fiber 树的指针
  alternate: Fiber | null
}
```

对于最开始的例子，首次渲染后最终将会出现这样的 FiberTree：

```text
Fiber (HostRoot) -> Fiber (<App />) -> Fiber (<h1>Hello World!</h1>)
```

如果之后比如 `<h1>` 触发了更新，其中的文字发生了变化，React 不会直接在原来的 FiberTree 上执行更改，而是会建立一棵新的 FiberTree，将其称为  `workInProgress` ；而原来的树对应着屏幕上当下显示的内容（被称为 `current`）。他们节点的 `alternate` 属性相互指着对方。

```text
HostRootFiber (Current) -> Fiber (<App />) -> Fiber (<h1>Hello World!</h1>)
HostRootFiber (WIP)     -> Fiber (<App />) -> Fiber (<h1>NEW World!</h1>)
```

当这棵新的树创建完毕、内容更新到 DOM 中之后，两棵树的地位随即发生互换。当下的 `WIP` 将会成为新的 `current`；而 `current` 则成为 `WIP`，等待着下一次更新。

```text
HostRootFiber (WIP)     -> Fiber (<App />) -> Fiber (<h1>Hello World!</h1>)
HostRootFiber (Current) -> Fiber (<App />) -> Fiber (<h1>NEW World!</h1>)
```

这种机制被称为 **双缓存机制**，在显卡中也有所使用，可以确保屏幕上不会出现只渲染了一半的结果。

# FiberTree

FiberTree 不是一种额外定义的类型，他只是由众多 `Fiber` 节点组成的单链表树形结构。`Fiber` 中的 `child` 和 `sibling` 就构成了这个数据类型，`return` 在其中也起到了一定的辅助作用。

`Fiber` 中的 `child` 指针指向子节点，`sibling` 指向下一个兄弟节点，`return` 指向任务执行完需要返回的地方，在这里可以理解为父节点。

以这个 JSX 的例子来做说明：

```jsx
const El = <MyComponent>
  <div id="div">
    <ul id="ul">
      <li id="li0"></li>
      <li id="li1"></li>
      <li id="li2"></li>
    </ul>
    <span id="span">Hello!</span>
  </div>
</MyComponent>
```

| Fiber       | Child  | Sibling | Return      |
| ----------- | ------ | ------- | ----------- |
| MyComponent | div    | null    | null*       |
| div         | ul     | null    | MyComponent |
| ul          | li0    | span    | div         |
| li0         | null   | li1     | ul          |
| li1         | null   | li2     | ul          |
| li2         | null   | null    | ul          |
| span        | null** | null    | div         |

*需要具体情况具体分析，可能是别的 `Fiber`

**React 中，为了简化，没有其他兄弟节点的文本节点不能单独成为一个 `Fiber`，所以 `span` 没有子节点，而是在 `memorizedProps` 等属性中记录文字内容

# FiberRoot

```text
FiberRoot -> HostRootFiber (Current) -> Fiber (<App />) -> Fibers..
                                                        -> Fibers..
             HostRootFiber (WIP)     -> Fiber (<App />) -> Fibers..
                                                        -> Fibers..
```

`createRoot` 的一个重要任务就是创建 `FiberRoot`，`FiberRoot` 是整个 [[React Reconciliation|Reconciliation]] 中最上面的对象，是 FiberTree 的根结点的父结点。

`FiberRoot` 中有一个 `current` 指针，指向当前屏幕上的 UI 对应的 `HostRootFiber`。`WIP` 和 `current`  FiberTree 地位的互换就是指的 `FiberRoot.current` 指向的改变。

```typescript
export type FiberRootNode = {
  // FiberRoot 的类型，分为 ConcurrentRoot 和 LegacyRoot，对应新版的 Concurrent 更新和之前的 Sync 更新
  // 本文中提到的都是 ConcurrentRoot
  tag: RootTag
  // Container Element
  containerInfo: Container
  // 指向当前屏幕上的 UI 对应的 HostRootFiber
  current: Fiber
  // 已经完成的 WIP HostRootFiber，准备 commit
  finishedWork: Fiber
  
  // 当次更新需要处理的优先级集合
  pendingLanes: Lanes
  // 已经处理完的优先级集合
  finishedLanes: Lanes
  
  // 将所有被调度了需要执行任务的 FiberRoot 用链表存起来
  next: FiberRoot | null
}
```
