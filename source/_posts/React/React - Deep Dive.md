---
title: React - Deep Dive
date: 2023-06-23 16:41:41
categories:
  - - 前端
---

与另一篇 React 笔记不同，本文主要是深入讨论 React 的源码与设计理念。

<!-- more -->

Clone React 项目之后，在 README 中找到调试 React 的方法，具体可参照 [How to Contribute](https://legacy.reactjs.org/docs/how-to-contribute.html)。然后按照文中安装依赖并 Build 之后，将 `fixture/packaging/babel-standalone/dev.html` 作为我们的 Playground。

现代前端框架的原理基本都可以用 `UI = f(State)`  来简单概括。其中 React 从状态更新到 UI 变化中间经历的过程，被称为 **Reconciliation**，完成这个过程的工具就是 **Reconciler**，他的核心代码都在 `react-reconciler` 这个包中。这个包不会耦合具体的宿主环境的代码，他只提供对状态更新流程的实现与封装，不同的宿主环境会通过不同的包来调用 Reconciler。

比如对于浏览器环境，React 提供了 `react-dom` 包，里面实现了操作 DOM 的方法。

除此之外，React 还提供了 `react` 包，他提供了诸如 `createElement` 等通用的方法，以及暴露给开发者的React 上层特性，比如 Hooks 等。

# 抽象数据类型

要完成 React Reconciliation 过程，需要数量众多的抽象数据类型进行支撑，包括 `ReactDOMRoot`  `Fiber `  `FiberRoot`  `Update`  `UpdateQueue`  `Lane` 等，本篇章将逐一介绍这些数据类型、对他们的操作以及他们的用途。初次阅读时可以跳过，后面遇到的时候再返回来查阅。

## ReactDOMRoot

ReactDOMRoot 是 `ReactDOM.createRoot` 的返回值，他事实上只是暴露了两个方法给我们使用，一个是 `render`，一个是 `unmount`。我们可以给 `render` 方法传入 `ReactElement`，然后开启首次渲染。

## Fiber

> `Fiber` 就是 Component 上的需要被执行的、或已经执行完成的工作，每个 Component 可能有多个 `Fiber`。
>
> ——React 源码中的注释

`Fiber` 可以暂时先理解为虚拟节点，他有不同的类型，比如 `FunctionComponent` `ClassComponent` `HostRoot` `HostText` `Fragment` 等。也就是说，一个 `Fiber` 节点可以对应某个 React 组件或某个 DOM 节点等。

React 中的很多过程都是使用深度优先遍历（DFS）来实现的：比如生成 FiberTree（暂时可以理解为 VirtualDOMTree）、将 FiberTree 的内容提交到宿主环境 UI 中、遍历 FiberTree 执行 Effects 等等。具体可参考本文的 **数据结构与算法** 章节。

需要注意的是：**如果使用递归实现 DFS 的过程，那么 React 将无法控制调用栈的随时中断与继续。**因此 React 选择借助 FiberTree 并使用迭代的方式实现 DFS，并在中间的某些环节设置了可以提前中断的手段。同时 `Fiber.updateQueue` 等属性提供了记忆并恢复工作状态的能力，使得继续之前被中断的任务成为可能。

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

## FiberTree

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

## FiberRoot

```text
FiberRoot -> HostRootFiber (Current) -> Fiber (<App />) -> Fibers..
                                                        -> Fibers..
             HostRootFiber (WIP)     -> Fiber (<App />) -> Fibers..
                                                        -> Fibers..
```

`createRoot` 的一个重要任务就是创建 `FiberRoot`，`FiberRoot` 是整个 Rconciliation 中最上面的对象，是 FiberTree 的根结点的父结点。

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

## Lane

React 中用 `Lane` 表示优先级这一概念。`Lane` 实际上就是二进制数字类型，每种优先级用不同的位为 `1` 来表示，用二进制来表示优先级就可以很方便地进行二进制运算来表达 “集合” 或者说 “批” 的概念。因为除去符号位之后一共还可以有 31 位，所以一共有 31 个 `Lane`，比如：

```typescript
export const NoLane: Lane = 0b0000000000000000000000000000000
export const SyncLane: Lane = 0b0000000000000000000000000000010
export const InputContinuousLane: Lane = 0b0000000000000000000000000001000
export const DefaultLane: Lane =  0b0000000000000000000000000100000
```

一般来讲，除了 `NoLane` 以外，数字越小代表优先级越高。

除了 `Lane` 以外，React 中还有 `Lanes`，实际上就是一群 `Lane` 做按位或运算的结果，表示 `Lane` 的集合。

### requestUpdateLane

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

React 还提供了一些操作 `Lane` 和 `Lanes` 的方法，他们底层都是通过位运算实现的，具体请参考 **算法与数据结构** 章节。

## Update

`Update` 是 React 中用来承载 *改变 State 的更新任务* 的数据类型，React 中通过 `createUpdate` 来创建 `Update`。

```typescript
export type Update<State> = {
  // 当前 Update 的优先级
  lane: Lane
  // Update 的类型，通过 createUpdate 创建的 tag 均为 UpdateState
  tag: UpdateState | ReplaceState | ForceUpdate | CaptureUpdate
  // 本次 Update 携带的内容
  payload: any
  callback:(() => any) | null
  // 指向下一个需要执行的 Update
  next: Update<State> | null
}
```

## UpdateQueue

`UpdateQueue` 是 React 中用来存放和消费 `Update` 的数据类型，`initializeUpdateQueue` 负责初始化 `Fiber` 上的 `updateQueue`，`enqueueUpdate` 用于将 `Update` 加入 `UpdateQueue` 中，`processUpdateQueue` 则用于消费 `Update`。

```typescript
export type UpdateQueue<State> = {
  baseState: State
  firstBaseUpdate: Update<State> | null
  lastBaseUpdate: Update<State> | null
  shared: SharedQueue<State>
  callbacks: Array<() => any> | null
}

export type SharedQueue<State> = {
  pending: Update<State> | null
  lanes: Lanes
  hiddenCallbacks: Array<() => any> | null
}
```

### enqueueUpdate

```typescript
export function enqueueUpdate<State>(
  fiber: Fiber,
  update: Update<State>,
  lane: Lane,
): FiberRoot | null {
  const updateQueue = fiber.updateQueue
  // 只有在 fiber unmount 的时候发生
  if (updateQueue === null) {
    return null
  }
  
  const sharedQueue: SharedQueue<State> = updateQueue.shared
  
  // TODO: 待实验验证
  if (isUnsafeClassRenderPhaseUpdate(fiber)) {
    // ... Render 阶段收到的更新
  } else {
    return enqueueConcurrentClassUpdate(fiber, sharedQueue, update, lane)
  }
}
```

```typescript
// 如果一个 render 正在进行中时收到一个并发的事件
// 我们会先将他们先暂存在这个数组中，然后等到当前的 render 结束（完成或者被中断）之后
// 再将这个数组的内容加到 fiber.updateQueue 上去
const concurrentQueues = []
let concurrentQueuesIndex = 0
let concurrentlyUpdatedLanes = NoLanes

// 将 update 暂存在 concurrentQueues 中，并返回 FiberRoot
export function enqueueConcurrentClassUpdate<State>(
  fiber: Fiber,
  queue: SharedQueue<State>,
  update: Update<State>,
  lane: Lane,
): FiberRoot | null {
  // 注意这个 enqueueUpdate 跟上面的不是一个函数，他会将 update 暂存在 concurrentQueues 数组中
  enqueueUpdate(fiber, queue, update, lane)
  // 找到该 Fiber 的 FiberRoot
  return getRootForUpdatedFiber(fiber)
}

// 将 update 暂存在 concurrentQueues 中
function enqueueUpdate(
  fiber: Fiber,
  queue: SharedQueue<State>,
  update: Update<State>,
  lane: Lane,
) {
  concurrentQueues[concurrentQueuesIndex++] = fiber
  concurrentQueues[concurrentQueuesIndex++] = queue
  concurrentQueues[concurrentQueuesIndex++] = update
  concurrentQueues[concurrentQueuesIndex++] = lane
  concurrentlyUpdatedLanes = mergeLanes(concurrentlyUpdatedLanes, lane)
  
  // fiber.lanes 在有的地方会用来检查工作是否被调度了，来判断执行紧急 bailout
  // 因此需要立即更新，而不是放在数组中等下次再更新
  fiber.lanes = mergeLanes(fiber.lanes, lane)
  const alternate = fiber.alternate
  if (alternate !== null) {
    alternate.lanes = mergeLanes(alternate.lanes, lane)
  }
}

// 将 concurrentQueues 中的 update 取出来放在 fiber 上
export function finishQueueingConcurrentUpdates() {
  const endIndex = concurrentQueuesIndex
  concurrentQueuesIndex = 0
  concurrentlyUpdatedLanes = NoLanes
  
  let i = 0
  while (i < endIndex) {
    const fiber = concurrentQueues[i]
    concurrentQueues[i++] = null
    const queue = concurrentQueues[i]
    concurrentQueues[i++] = null
    const update = concurrentQueues[i]
    concurrentQueues[i++] = null
    const lane = concurrentQueues[i]
    concurrentQueues[i++] = null
    
    if (queue !== null && update !== null) {
      const pending = queue.pending
      if (pending === null) {
        // 第一个 update，自己指向自己，创建环状链表
        update.next = update
      } else {
        // 后续的 update，将他插到链表的第一个（同时也是最后一个）
        update.next = pending.next
        pending.next = update
      }
      queue.pending = update
    }
    
    if (lane !== NoLane) {
      // TBC lane 的冒泡
      markUpdateLaneFromFiberToRoot(fiber, update, lane)
    }
  }
}
```

### processUpdateQueue

```typescript
function processUpdateQueue(
  workInProgress: Fiber,
  props: any,
  instance: any,
  renderLanes: Lanes,
) {
  // 对于 ClassComponent 和 HostRoot，一定非空
	const queue = UpdateQueue<State> = workInProgress.updateQueue
  
  hasForceUpdate = false
  
  let firstBaseUpdate = queue.firstBaseUpdate
  let lastBaseUpdate = queue.lastBaseUpdate
  
  // 将 pendingUpdates 合并到 baseQueue 中（放到最后面）
  let pendingQueue = queue.share.pending
  if (pendingQueue !== null) {
    queue.share.pending = null
    // pendingQueue 是一个环状链表，现在拿到他的首末节点，并解环
    const lastPendingUpdate = pendingQueue
    const firstPendingUpdate = lastPendingUpdate.next
    lastPendingUpdate.next = null
    if (lastBaseUpdate === null) {
      firstBaseUpdate = firstPendingUpdate
    } else {
      lastBaseUpdate.next = firstPendingUpdate
    }
    lastBaseUpdate = lastPendingUpdate
    // 将 baseQueue 合并到 current 上去
    // 使得在 commit 之前，不管有多少高优先级的更新打断了该 wip 的 Render，使得该 wip 重新构造，之前的 Updates 依然保留
    const current = workInProgress.alternate
    if (current !== null) {
      const currentQueue = current.updateQueue
      const currentLastBaseUpdate = currentQueue.lastBaseUpdate
      if (currentLastBaseUpdate !== lastBaseUpdate) {
        if (currentLastBaseUpdate === null) {
          currentQueue.firstBaseUpdate = firstPendingUpdate
        } else {
          currentLastBaseUpdate.next = firstPendingUpdate
        }
        currentQueue.lastBaseUpdate = lastPendingUpdate
      }
    }
  }

  // 如果在上述合并之后，baseQueue 里面有 Update
  if (firstBaseUpdate !== null) {
    // 遍历 Updates 链表，计算结果
    let newState = queue.baseState
    let newLanes = NoLanes
    
    let newBaseState = null
    let newFirstBaseUpdate = null
    let newLastBaseUpdate = null
    
    let update = firstUpdate
    do {
      const updateLane = update.lane
      // 通过该 Update 的 lane 是否是 renderLanes 的子集，来判断本次更新中是否需要处理该 Update
      const shouldSkipUpdate = !isSubsetOfLanes(renderLanes, updateLane)
      if (shouldSkipUpdate) {
        // 该 Update 优先级不足，跳过该 Update，将该 Update 放到 newBaseQueue 中供下次使用
        const clone: Update<State> = {
          lane: updateLane,
          tag: update.tag,
          payload: update.payload,
          callback: update.callback,
          next: null,
        }
        if (newLastBaseUpdate === null) {
          // 如果这是第一个被跳过的 Update，则：
          // newBaseState 就固定为当前的 newState（上次 Update 计算后的结果）
          // newFirstBaseUpdate 和 newLastBaseUpdate 都是该 Update
          newFirstBaseUpdate = newLastBaseUpdate = clone
          newBaseState = newState
        } else {
          // 如果不是第一个被跳过的 Update，则将其加在链表尾节点即可
          newLastBaseUpdate = newLastBaseUpdate.next = clone
        }
        // 将跳过的 Lane 记录在 newLanes 里面
        newLanes = mergeLanes(newLanes, updateLane)
      } else {
        // 该 Update 有足够的优先级，本次应该得到处理
        if (newLastBaseUpdate !== null) {
          // 前面有 Update 被跳过，该 Update 也要加入 baseQueue，否则计算会出错
          const clone: Update<State> = {
            // 该 Update 实际上会被考虑到结果的计算之中，并被 commit
            // 我们不希望该 Update 因为优先级的问题一直留在 baseQueue 中（尽管他已经 commit 了）
            // 因此把他的 lane 置为 0，因为 0 是所有的子集，这样在上一步的检查中就永远不会被跳过了
            lane: NoLane,
            tag: update.tag,
            payload: update.payload,
            // 该 Update rebased 了，他的回调不应该再被调用一遍
            callback: null,
            next: null
          }
          newLastBaseUpdate = newLastBaseUpdate.next = clone
        }
        
        // 处理该 Update
        newState = getStateFromUpdate(
          workInProgress, queue, update, newState, props, instance,
        )
        
        // 如果有 callback 需要执行，给该 wipFiber 打上标记，并保存在待执行的 callbacks 列表中
        const callback = update.callback
        if (callback !== null) {
          workInProgress.flags |= Callback
          const callbacks = queue.callbacks
          if (callbacks === null) {
            queue.callbacks = [callback]
          } else {
            callbacks.push(callback)
          }
        }
        
        update = update.next
        
        if (update === null) {
          pendingQueue = queue.shared.pending
          if (pendingQueue === null) {
            // 更新处理完毕
            break
          } else {
            // 边界情况：如果在处理更新的时候有一个更新被插入，继续处理
            const lastPendingUpdate = pendingQueue
            const fistPendingUpdate = lastPendingUpdate.next
            lastPendingUpdate.next = null
            update = firstPendingUpdate
            queue.lastBaseUpdate = lastPendingUpdate
            queue.shared.pending = null
          }
        }
      } while (true)
      
      if (newLastBaseUpdate === null) {
        newBaseState = newState
      }
      
      queue.baseState = newBaseState
      queue.firstBaseUpdate = newFirstBaseUpdate
      queue.lastBaseUpdate = newLastBaseUpdate
      
      // TBC //
      markSkippedUpdateLanes(newLanes)
      workInProgress.lanes = newLanes
      workInProgress.memorizedState = newState
    }
  }
  currentlyProcessingQueue = null
}
```

## Hook

React 的 `FunctionComponentFiber` 中的 `memorizedState` 属性指向的即为该函数组件的第一个 `Hook`，一个函数组件内部的 Hooks 会组成一个链表，他们由 `.next` 链接起来。

**注意当前 React 源码中 `ReactFiberHooks.js` 中单独定义了 `Hook` 的 `Update` 和 `UpdateQueue`，这跟上文提到的 `Fiber` 中的 `Update` 和 `UpdateQueue` 思路类似但结构有所不同！**  

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

# 数据结构与算法

React 的源码中使用了几个非常典型的数据结构和算法，包括各种链表的变体、深度优先遍历、位运算等。

## 链表

链表是 React 中使用较多的数据结构之一，其中有两种链表较为特殊：**一种是以链表为基础的树形结构，一种是环形链表。**

### 单链表树

FiberTree 就是单链表树，他跟传统的树有些不同，他是由一条条链表组成的，更像是一个鱼骨形的结构。

FiberTree 有一条明显的主干链表，即 `root.child.child...` 这一条链表。

然后每一个节点都有 `sibling` 属性，该属性也指向了一个链表，这个链表将其兄弟节点串了起来。

```text
FiberRoot
   |
 Fiber -> Fiber -> Fiber
   |        |
   |      Fiber -> Fiber -> Fiber
   |
 Fiber -> Fiber
   |
 Fiber -> Fiber
            |
          Fiber
```

### 环形链表

React 中的更新队列是采用环形链表存储的，准确的说是 `UpdateQueue.shared.pending` 属性，该属性指向一个由 `Update` 构成的环形链表，链表中的每一项用 `next` 指向下一项，且永远指向最后加入的 `Update`。

假设我们按照 `U1` `U2` `U3` 的顺序插入 `UpdateQueue.shared.pending`，那么每次插入后的结果将是：

```typescript
// append U1
U1.next = U1
UpdateQueue.shared.pending = U1
// U1 -> U1

// append U2
U2.next = U1
U1.next = U2
UpdateQueue.shared.pending = U2
// U2 -> U1 -> U2

// append U3
U3.next = U1
U2.next = U3
UpdateQueue.shared.pending = U3
// U3 -> U1 -> U2 -> U3
```

除了 `UpdateQueue` 以外，React 中的 Hooks 等也是用类似的数据结构保存的。

## 深度优先遍历

React 中大量使用 DFS 来处理节点，基本都按照下列代码的顺序执行：

1. 先沿着 `child` 一路遍历到底
2. 再从最后一层倒着开始检查每一层的 `sibling`，再对每一个 `sibling` 进行 DFS，遍历他的 `child`

```typescript
function dfs(node: Fiber) {
  doPreWork(node)
  let child = node.child
  while (child !== null) {
    dfs(child)
    doPostWork(child)
    child = child.sibling
  }
}
```

可以看到这个写法的遍历顺序与回溯算法其实非常像，有关回溯可参考 vvv Algorithm %} 这篇笔记。

这样写遍历的缺点在前面的 `Fiber` 节也提到过，**由于是借助 JavaScript 调用栈的递归写法，我们无法控制他的终止**。这个时候 `return` 属性的意义就出来了，我们可以借助 `return` 代替栈来记录每个子任务执行完成之后需要返回的地方。

于是就有了下面在 React 中的典型代码：

```typescript
function processTree(root: Fiber) {
  let node = root
  function proecessNode() {
    // 向下遍历
    doPreWork(node)
    if (node.child !== null) {
      node = node.child
      return
    }
    // 向上遍历
    while (node !== null) {
      doPostWork(node)
      if (node.sibling !== null) {
        node = node.sibling
        return
    	}
    	node = node.return
    }
  }
  while (node !== null && shouldYield()) {
	  processNode()
  }
}
```

可以看到，在迭代的写法中，如果 `shouldYield()` 在某次循环时变成了 `false`，那么循环将不再继续，遍历过程被终止了。

递归的写法胜在方便，因此在 React 中仍有部分应用，比如在完全同步执行、不能中断的 Commit 阶段，就会使用到递归的写法。

在两种写法中，都有调用 `doPreWork()` 和 `doPostWork()`，分别起到前序执行和后序遍历的效果。React 中有很多工作都是在这两个时机进行处理的，比如 Render 阶段的 `beginWork` 即为 `doPreWork` 、`completeWork` 即为 `doPostWork`。

## 位运算

React 中广泛使用二进制位来标记状态，比如 `Fiber` 中的 `flag` 和 `lane` 均使用了二进制位。`flag` 中每一位代表一种需要执行的副作用，而 `lane` 中不同的位则表示不同的优先级。

通过位运算，React 可以很方便地实现集合的交、并、差等运算，下面是 `lane` 相关操作的实现，`flag` 也类似：

```typescript
export function includesSomeLane(a: Lanes | Lane, b: Lanes | Lane): boolean {
  return (a & b) !== NoLanes;
}

export function isSubsetOfLanes(set: Lanes, subset: Lanes | Lane): boolean {
  return (set & subset) === subset;
}

export function mergeLanes(a: Lanes | Lane, b: Lanes | Lane): Lanes {
  return a | b;
}

export function removeLanes(set: Lanes, subset: Lanes | Lane): Lanes {
  return set & ~subset;
}

export function intersectLanes(a: Lanes | Lane, b: Lanes | Lane): Lanes {
  return a & b;
}

export function getHighestPriorityLane(lanes: Lanes): Lane {
  // 取最低位的 1
  return lanes & -lanes;
}
```

## 优先级队列

Scheduler 中使用优先级队列来为任务进行排序，并保证每次出队的任务均是优先级最高的任务，优先级队列有多种实现方式，React 的 Scheduler 是采用常用的小顶堆来实现的。关于 *优先级队列和堆* 的更多内容可以参考 vvv Algorithm %} 这篇笔记。

# 首次渲染过程

React 的更新流程分为 **Schedule、Render 和 Commit** 三个阶段：

- **Schedule 阶段** 会挑出一批优先级，并为不同优先级准备不同的调度策略；
- **Render 阶段** 会执行 Fiber 节点的重新计算，并标记好需要执行的副作用；
- **Commit 阶段** 分为 **BeforeMutation**、**Mutation** 和 **Layout** 三个子阶段，在 **Mutation 阶段** 会完成 UI 相关的副作用，并且会完成 `workInProgress` 与 `current` FiberTree 的切换。 

## Schedule 开始之前

写一个简单的例子，然后通过 debug 可以找到 React 的入口：

```jsx
const App = () => {
  return <h1 onClick={() => console.log('click!')}>Hello World!</h1>
}
ReactDOM.createRoot(
  document.getElementById('container')
).render(
  <App />,
);
```

在上述例子中，总的来说会经历这样的过程：

1. **createRoot：**来自 `react-dom` 包的 `createRoot` 方法获取 Container Element 相关的信息，并调用 `react-reconciler` 中的方法来初始化 `FiberRoot` 和 `HostRootFiber`；
2. **createElement：**来自 `react` 包的 `createElement` 将 JSX 转换结果再次转换为 `ReactElement`；
3. **render：**调用 `createRoot` 返回对象的 `render` 方法，并传入 `ReactElement`，该方法会调用 `react-reconciler` 中的方法进入 Reconciliation 流程。

```text
1. ReactDOM.createRoot             // 创建 ReactDOMRoot
     createContainer               // 创建 Container，Container 实际上就是创建 FiberRoot
       createFiberRoot             // 创建 FiberRoot
         createHostRootFiber       // 创建 HostRootFiber（tag 为 HostRoot 的 Fiber）
           createFiber             // 创建 Fiber
     listenToAllSupportedEvents    // 在容器 DOM 元素上绑定事件

2. React.createElement             // 暴露给外部，将 JSX 转换的结果再转换为 ReactElement

3. ReactDOMRoot.render             // 传入 ReactElement，开始首次渲染
     updateContainer               // 首次渲染的入口，创建 Update 并入队，准备进入 Schedule 阶段
```

### createRoot

首先是 `createRoot` 函数，该函数位于 `react-dom`，主要用于：

1. 调用 `createContainer` 创建 `FiberRoot`
2. 调用 `listenToAllSupportedEvents` 为 `container` 绑定事件，React 的事件是通过 `container` 做事件代理来进行监听的
3. 调用 `new ReactDOMRoot` 创建 `ReactDOMRoot`，并返回这个对象，其原型链上有 `render` 和 `unmount` 方法

```typescript
export function createRoot(
  container: Element | Document | DocumentFragment,
  options?: CreateRootOptions,
): RootType {
  // ...
  // createContainer 根据 container 和初始信息创建一个 FiberRoot（FiberRoot 相关的详细内容见对应章节）
  const root = createContainer(container)
  container.__reactContainer$RandomKey = root
  // ...
  const rootContainerElement = container.nodeType === COMMENT_NODE
    ? container.parentNode
    : container;
  // listenToAllSupportedEvents 会在 contaienr 上添加 eventListenr 来监听支持的事件
  listenToAllSupportedEvents(rootContainerElement);
	// ReactDOMRoot.prototype 上定义了两个方法：render 和 unmount
  return new ReactDOMRoot(root);
}
```

#### createContainer

`createContainer` 位于 `reac-reconciler` 中，他直接调用了 `createFiberRoot`

```typescript
function createContainer(...args) {
  return createFiberRoot(...)
}
```

`createFiberRoot` 也位于 `react-reconciler` 中，他：

1. 创建了一个 `FiberRoot`
2. 为该 `FiberRoot` 创建了一个 `HostRootFiber`（一个 `tag` 为 `HostRoot` 的 `Fiber`）。该 `HostRootFiber` 即为后续 FiberTree 的根节点
3. 初始化 `HostRootFiber` 的 `memorizedState` 和 `updateQueue`

```typescript
function createFiberRoot() {
  // 1. 创建 FiberRoot
  const root = new FiberRootNode()
  
  // 2. 创建 tag 为 HostRoot 的 Fiber
  const uninitializedFiber = createHostRootFiber() 
  root.current = uninitializedFiber
  uninitializedFiber.stateNode = root
  
  // 3.1. 初始化 memorizedState，可以看做是用来记录某些值的
  const initialState: RootState = {
    // 首次渲染中 initialChildren 为 null
    element: initialChildren,
  }
  uninitializedFiber.memorizedState = initialState
  
  // 3.2. 初始化 updateQueue，稍后进入渲染流程后将会提到，用于存取 Update
  initializeUpdateQueue(uninitializedFiber)
}
```

### createElement

Babel 会将 JSX 静态编译为 JavaScript 代码，可以在这个 [Babel Playground](https://babeljs.io/repl) 看到即时编译的效果，比如

```jsx
<div>123</div>
```

会被编译为：

```javascript
/*#__PURE__*/React.createElement("div", null, "123");
```

或者

```jsx
import { jsx as _jsx } from "react/jsx-runtime";
/*#__PURE__*/_jsx("div", {
  children: "123"
});
```

`jsx` 和 `createElement` 都定义在 `react/src/ReactElementValidator.js` 中了，他们都会在经过一系列校验之后调用 `react/src/ReactElement.js` 中对应的函数。二者的不同在 [这篇文章](https://github.com/reactjs/rfcs/blob/createlement-rfc/text/0000-create-element-changes.md)，（TBC）

`jsx` 和 `createElement` 最终返回值都是相同的结构，这里以 `createElement` 举例：

```typescript
function createElement(type, config, children) {
  // ...
  // 对于函数组件，这里的 type 会传入这个函数本身
  // 从 config 中提取并生成 key, ref, props 等传给下一层
  return ReactElement(type, key, ref, soource, ReactCurrentOwner.current, props)
}
```

`ReactElement` 是一个十分简单的函数，他构造了一个 `element` 对象：

```typescript
function ReactElement(type, key, ref, self, source, owner, props) {
  const element = {
    // 标记该对象为 ReactElement 类型
    $$typeof: REACT_ELEMENT_TYPE,
    // 一些内置的属性
    type, key, ref, props,
    // 记录创建该元素的组件
    _owner: owner,
  }
  return element
}
```

最终，这个对象被传给了 `ReactDOMRoot` 的 `render` 函数，首屏渲染开始。

### render

调用 `createRoot` 返回的对象的 `render` 方法，他会调用 `updateContainer` 并传入一个由 jsx 转换来的 `ReactElement` 和刚刚创建好的 `FiberRoot`，来进入更新流程：

```typescript
ReactDOMRoot.prototype.render = function(children: ReactNodeList) {
  // ...
  updateContainer(children, root)
}
```

`updateContainer` 位于 `react-reconciler/src/ReactFiberReconciler.js` 中，他会：

1. 获取优先级（`lane`）
2. 创建 `Update`，并将该更新入队
3. 调用 `scheduleUpdateOnFiber`，准备进入 Schedule 阶段

```typescript
// 对于首屏渲染，element 是 <App />，container 是 FiberRoot，剩下两个为 null
function updateContainer(
  element: ReactElement,
  container: FiberRoot,
  parentComponent,
  callback
) {
  const current = container.current

  // 获取当前更新的优先级（lane）
  const lane = requestUpdateLane(current)
  // 创建 Update，他的 payload 就是传入的 ReactElement
  const update = createUpdate(lane)
  update.payload = {
    element,
  }
  // 将该更新入队，并且会返回当前 Fiber 对应的 FiberRoot
  // current.lanes = mergeLanes(current.lanes, lane)
  const root = enqueueUpdate(current, update, lane)
  // 准备进入 Schedule 阶段
  if (root !== null) {
    scheduleUpdateOnFiber(root, current, lane)
  }
  return lane
}
```

`scheduleUpdateOnFiber` 位于 `react-reconciler/src/ReactFiberWorkLoop.js` 中，他主要用于：

1. 把当前更新的的 `lane` 记录在 `root` 中
2. 进入调度阶段

```typescript
function scheduleUpdateOnFiber(
  root: FiberRoot,
  fiber: Fiber,
  lane: Lane
) {
  // root.pendingLanes |= lane
  markRootUpdated(root, lane)
  // 进入 Schedule 阶段
  ensureRootIsScheduled(root)
}
```



## Schedule

React 的任务调度分为 **宏任务调度** 和 **微任务调度** 两种，React 会根据任务的优先级采取不同的调度策略，这个选择调度策略以及进行任务调度的阶段，即为 Schedule 阶段。

Schedule 阶段主要用到 `react-reconciler` 和 `scheduler` 。

`react-reconciler` 主要负责在 React 的工作流中进行任务调度，而涉及到具体的调度的具体实现（比如任务队列的维护，具体使用什么 API 让宿主环境在什么时候执行什么优先级的任务等）则会根据是宏任务还是微任务而有所不同：

React 中所有的更新工作都会被调度，或是在微任务中执行、或是在宏任务重执行**，因此 React 中并没有严格意义上的同步更新**，这也是 `setState` 并不会同步生效的原因。

**为了避免混淆，本文中将 React 规定的同步用英文代指，称为“Sync”；而 JavaScript 中的同步仍然称为“同步”。同时，与“Sync”相对应的术语我们称为“Concurrent”。**

**微任务调度** 可以看做是 React 中最快任务，他们会在下一个宏任务之前进行，因此会快于所有宏任务调度的工作。`react-reconciler` 中主要使用 `scheduleMicrotask` 函数来将任务调度到微任务中来执行。该函数的实现与宿主环境有关，比如在浏览器和 ReactNoopRenderer 下，他将优先使用 `queueMicrotask` API 来进行调度：

```typescript
const scheduleMicrotask =
  typeof queueMicrotask === 'function'
    ? queueMicrotask
    : typeof Promise !== 'undefined'
    ? callback =>
        Promise.resolve(null)
          .then(callback)
          .catch(error => {
            setTimeout(() => {
              throw error;
            });
          })
    : setTimeout
```

**宏任务调度** 则会使用 `scheduler` 包中的 API，`react-reconciler` 只需要传给传给 `scheduler` *需要执行的任务* 及 *对应的优先级* 即可。

### Scheduler

正如上文所说，Scheduler 提供了一些核心 API 供 `react-reconciler` 包使用：

```typescript
// 使用某种优先级调度 task
scheduleCallback(task, priority)

// 提示时间是否用尽，需要中断循环
shouldYield()

// 取消调度 task
cancelCallback(task)
```

Scheduler 实际上就是采用了一堆计时器，每个 `task` 会有 `expirationTime` 属性代表了他的过期时间，`expirationTime` 越小就意味着优先级越高，这在一定程度上也能解决饥饿问题。

```typescript
function unstable_scheduleCallback(
  priorityLevel: PriorityLevel,
  callback: Callback,
  options?: {delay: number},
): Task {
  const currentTime = getCurrentTime();

  // 如果传入了 delay 参数，则会延迟执行
  let startTime;
  if (typeof options === 'object' && options !== null) {
    var delay = options.delay;
    if (typeof delay === 'number' && delay > 0) {
      startTime = currentTime + delay;
    } else {
      startTime = currentTime;
    }
  } else {
    startTime = currentTime;
  }

  // 根据传入的优先级确定 timeout，从而确定 expirationTime
  let timeout;
  switch (priorityLevel) {
    case ImmediatePriority:
      timeout = IMMEDIATE_PRIORITY_TIMEOUT;      // -1
      break;
    case UserBlockingPriority:
      timeout = USER_BLOCKING_PRIORITY_TIMEOUT;  // 250
      break;
    // 永远不会到时间
    case IdlePriority:
      timeout = IDLE_PRIORITY_TIMEOUT;           // maxSigned31BitInt
      break;
    case LowPriority:
      timeout = LOW_PRIORITY_TIMEOUT;            // 10000
      break;
    case NormalPriority:
    default:
      timeout = NORMAL_PRIORITY_TIMEOUT;         // 5000
      break;
  }

  const expirationTime = startTime + timeout;

  const newTask: Task = {
    id: taskIdCounter++,
    callback,
    priorityLevel,
    startTime,
    expirationTime,
    sortIndex: -1,
  };
  if (enableProfiling) {
    newTask.isQueued = false;
  }

  if (startTime > currentTime) {
    // 延迟任务，进入 timerQueue；延迟时间到了之后进入 taskQueue
    newTask.sortIndex = startTime;
    push(timerQueue, newTask);
    if (peek(taskQueue) === null && newTask === peek(timerQueue)) {
      // All tasks are delayed, and this is the task with the earliest delay.
      if (isHostTimeoutScheduled) {
        // Cancel an existing timeout.
        cancelHostTimeout();
      } else {
        isHostTimeoutScheduled = true;
      }
      // Schedule a timeout.
      requestHostTimeout(handleTimeout, startTime - currentTime);
    }
  } else {
    // 非延迟任务，进入 taskQueue
    newTask.sortIndex = expirationTime;
    push(taskQueue, newTask);
    if (enableProfiling) {
      markTaskStart(newTask, currentTime);
      newTask.isQueued = true;
    }
    // Schedule a host callback, if needed. If we're already performing work,
    // wait until the next time we yield.
    if (!isHostCallbackScheduled && !isPerformingWork) {
      isHostCallbackScheduled = true;
      requestHostCallback(flushWork);
    }
  }

  return newTask;
}
```

可以看到 Scheduler 中维护了两个队列： `timerQueue` 和 `taskQueue`。

- 如果调用 `scheduleCallback` 的时候传入了 `delay` 参数，则会进入 `timerQueue` 并延迟开始计时的时间；
- 否则会直接进入 `taskQueue` 开始计时，然后调用 `requestHostCallback` 来开启 **宏任务** 执行 `taskQueue` 中的任务。

`timerQueue` 和 `taskQueue` 均采用了 **优先级队列** 这一数据结构，优先级队列中排序的依据就是过期时间 `expirtionTime`。

`requestHostCallback` 等函数会 **开启一个新的宏任务** 来循环执行 `taskQueue`，目前 Scheduler 中是这样判断使用什么宏任务的：

```typescript
let schedulePerformWorkUnitlDeadline
if (typeof localSetImmediate === 'function') {
  // Node.js 和老 IE，setImmediate 不像 MessageChannel 一样会阻止 Node.js 进程退出，而且执行时机更早
  schedulePerformWorkUnitlDeadline = () => {
    localSetImmediate(performWorkUnitlDeadline)
  }
} else if (typeof MessgeChannel !== 'undefined') {
  // DOM 和 Worker
  const channel = new MessageChannel()
  const port = channel.port2
  channel.port1.onmessage = performWorkUnitlDeadline
  schedulePerformWorkUnitlDeadline = () => {
    port.postMessage(null)
  }
} else {
  // setTimeout 会有 4ms 延迟
  schedulePerformWorkUnitlDeadline = () => {
    localSetTimeout(performWorkUntilDeadline, 0)
  }
}
```

需要注意的是，`requestIdleCallback` 由于兼容性、以及他执行频率不稳定的问题没有被 React 选用。另外 `requestIdleCallback` 是在空闲时期执行低优先级工作，这与 Scheduler 需要执行多种优先级任务的需求相悖。

### React 中的优先级

React 中有两种优先级表示方法，一种是 `Lane`，一种是事件优先级 `EventPriority`，`EventPriority` 可以看做是特殊的 `Lane`。

具体来讲，不同的事件会产生不同的 `EventPriority`，比如 `DiscreteEventPriority` 代表 `click` `input` `focus` 等离散事件； `ContinousEventPriority` 代表 `drag` `mouseover` 等连续事件。这些 `EventPriority` 都有着对应的 `Lane`，他们数值相同，不需要函数来转换。

但是 `Lanes` 转换为 `EventPriority` 时，由于是多对一转换，就需要 `lanesToEventPriority` 来帮忙。

**不过，上面提到的 `Scheduler` 还有一套自己的优先级系统，React 需要再将优先级转换为 `SchedulerPriority`，才能调用 `Scheduler` 的 API。** 其实也很简单，用 `switch-case` 映射一下就好了。

### 计算并处理调度

React 中，“计算并调度更新”的过程本身也是需要在微任务中执行的，`ensureRootIsScheduled` 是整个 Schedule 阶段的入口，每次 `FiberRoot` 收到更新的时候都会调用该函数，他会：

1. 确保 `FiberRoot` 已经被调度了
2. 确保有一个微任务在处理调度这个 `FiberRoot` 

```typescript
// 防止有重复的微任务
let didScheduleMicrotask = false

// 用于存放当前正在执行任务的 root 链表的头尾指针
let firstScheduledRoot = null
let lastScheduledRoot = null

export function ensureRootIsScheduled(root: FiberRoot) {
  // 1. 确保 root 已经被调度了
  // 将 root 加入调度
  if (root === lastScheduledRoot || root.next !== null) {
    // root 已经被调度了，不需要再加入调度
  } else {
    if (lastScheduledRoot === null) {
      firstScheduledRoot = lastScheduledRoot = root
    } else {
      lastScheduledRoot.next = root
      lastScheduledRoot = root
    }
  }
  
  // 每次 root 收到更新的时候，都标记可能有 Sync 任务要做
  // 如果这个标记为 false，flushSync 将会跳过执行 Sync 任务的过程
  mightHavePendingSyncWork = true
  
  // 2. 确保有一个微任务在处理调度这个 root
  // 在微任务中对 root 进行调度
  if (!didScheduleMicrotask) {
    didScheduleMicrotask = true
    scheduleImmediateTask(processRootScheduleInMicrotask)
  }
}

function scheduleImmediateTask(cb: () => any) {
  // 将任务放在微任务中执行
  scheduleMicrotask(() => {
    // ... 省略对 Safari BUG 的兼容代码
    cb()
  })
}
```

`processRootScheduleInMicrotask` 是处理调度任务的核心函数，他会由 `scheduleMicrotask` 调度、使得微任务中进行：

1. 遍历 `root` 链表
2. 通过函数 `scheduleTaskForRootDuringMicrotask` 选择我们想要执行的 `lanes`，并进行调度
3. 通过函数 `flushSyncWorkOnAllRoots` 处理所有的调度进微任务中的任务

```typescript
function processRootScheduleInMicrotask() {
  didScheduleMicrotask = false
  
  // 重新计算 mightHavePendingSyncWork 标记
  mightHavePendingSyncWork = false
  
  // 从 Scheduler 中取到的当前时间
  const currentTime = now()
  
  // 1. 遍历 root 链表
  let prev = null
  let root = firstScheduledRoot
  while (root !== null) {
    const next = root.next
    // 2. 选出此次我们想要执行的 lanes，并进行调度
    const nextLanes = scheduleTaskForRootDuringMicrotask(root, currentTime)
    if (nextLanes === NoLane) {
      // 这个 root 没有更多的任务需要执行，将他从 Schedule 中移除，简单的链表操作
      // 尽管有很多地方可以将 root 加入 Schedule，但是只有这个地方移除
      root.next = null
      if (prev === null) {
        firstScheduledRoot = next
      } else {
        prev.next = next
      }
      if (next == null) {
        lastScheduledRoot = prev
      }
    } else {
      // 这个 root 依然有任务要做
      prev = root
      // lanes 中包含 SyncLane | SyncHydrationLane
      if (includeSyncLane(nextLanes)) {
        mightHavePendingSyncWork = true
      }
    }
    root = next
  }
  
  // 3. 处理所有在调度进微任务执行的任务
  flushSyncWorkOnAllRoots()
}
```

### 在微任务中选择优先级并进行调度

这个 `scheduleTaskForRootDuringMicrotask` 函数只能在微任务中调用，他会获取到本轮调度应该执行的优先级 `nextLanes`，并进行调度：

1. 通过 `getNextLanes` 选择优先级
2. 根据优先级把任务以回调函数的形式传给 `scheduleCallback` 交给 Scheduler 进行调度

```typescript
function scheduleTaskForRootDuringMicrotask(root, currentTime) {
  // 检查是否有 lane 处于饥饿状态（低优先级任务由于高优先级的任务疯狂插队导致迟迟无法完成，称为饥饿）
  // 将他们标记为过期
  // root.expirtaionTimes 里面存有 lanes 的过期时间，通过这个与 currentTime 来判断是否已经过期
  markStarvedLanesAsExpired(root, currentTime)
  
  // 获取 wipRoot，实际上就是全局存了一个 wipRoot 变量，对于首次渲染，应该为 null
  const workInProgressRoot = getWorkInProgressRoot()
  // 跟 wipRoot 逻辑类似，对于首次渲染，应该为 NoLane
  const workInProgressRootRenderLanes = getWorkInProgressRootRenderLanes()
  // 获取本次更新要执行的优先级，可以暂时理解为 getHighestPriorityLane(root.pendingLanes)
  const nextLanes = getNextLanes(
    root,
    root === workInProgressRoot ? workInProgressRootRenderLanes : NoLanes,
  )
  // 当前正在执行的回调
  const existingCallbackNode = root.callbackNode
  
  if (
    // 没有工作进行
    nextLanes === NoLanes ||
    // 当前 root Suspended，并且正在等待数据，不要调度
    // Suspended render phase
    (root === workInProgressRoot && isWorkLoopSuspendedOnData()) ||
    //Suspended commit phase
    root.cancelPendingCommit !== null
  ) {
    // 取消掉之前的回调
    if (existingCallbackNode !== null) {
      cancelCallback(existingCallbackNode)
    }
    root.callbackNode = null
    root.callbackPriority = NoLane
    return NoLane
  }
  
  if (includesSyncLane(nextLanes)) {
    // 在本次调度任务的微任务最后，总是会 flush 一遍 Sync 任务，因此不需要额外调度
    if (existingCallbackNode !== null) {
      cancelCallback(existingCallbackNode)
    }
    root.callbackNode = null
    root.callbackPriority = SyncLane
    return SyncLane
  }
  
  // 获取最高优先级的任务来代表回调的优先级，其实就是取 nextLanes 的最后一个 1
  const exisitingCallbackPriority = root.callbackPriority
  const newCallbackPriority = getHighestPriorityLane(nextLanes)
  
  // 新任务的最高优先级等于正在执行的任务的优先级，优先级没有发生变化
  // 可以重复利用当前的任务，但可能会使得新的任务得不到执行
  if (newCallbackPriority === exisitingCallbackPriority) {
    return newCallbackPriority
  } else {
    cancelCallback(existingCallbackNode)
  }
  
  // 将 lanes 转换为 Scheduler 使用的优先级
  // 源码中实际上是 lanes 先转换为 EventPriority，再转换为 SchedulerPriority，这里做了简化
  let schedulerPriorityLevel = lanesToSchedulerPriority(lanes)
  
  // 调用 Scheduler.scheduleCallback，传入优先级和任务，在宏任务中调度
  const newCallbackNode = scheduleCallback(
    schedulerPriorityLevel,
    performConcurrentWorkOnRoot.bind(null, root)
  )
  
  root.callbackPriority = newCallbackPriority
  root.callbackNode = newCallbackNode
  return newCallbackPriority
}
```

## Render

在 Render 阶段，React 会使用 DFS 构建 `workInProgress` FiberTree，**整个 Render 的过程可以分为两部分**：

| beginWork                                                    | completeWork                                                 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 前序执行<br />在遍历子树之前执行<br />向下遍历的过程中执行<br />“递” 的阶段执行 | 后序执行<br />在遍历子树之后执行<br />向下遍历的过程中执行<br />“归” 的阶段执行 |
| 根据当前 `Fiber` 创建 `child`<br />标记 `Placement` `ChildDeletion` | 构建 DOMTree<br />`flags` 和 `lanes` 冒泡                    |

> 关于 DFS 的详细说明请看 **数据结构与算法 - 深度优先遍历** 节，以及下面的 **遍历 FiberTree 执行任务** 节

Render 阶段也由 `react-reconciler` 包进行处理，他有两个入口函数：

- Sync 优先级使用 `performSyncWorkOnRoot`
- 其他优先级（Concurrent）使用 `performConcurrentWorkOnRoot`

源码中函数的调用流程如下所示：

```text
performSync/ConcurrentWorkOnRoot     // 渲染流程控制（获取优先级、Render、Commit、Schedule）
  getNextLanes                       // 获取优先级
  renderRootSync/Concurrent          // Render（初始化渲染所需变量、workLoop）
    prepareFreshStack                // 初始化渲染所需变量
    workLoopSync/Concurrent          // 循环遍历 FiberTree
      performUnitOfWork              // 处理每一个 Fiber
        beginWork                    
        completeWork                 
  commitRoot                         // Commit
  ensureRootIsScheduled              // Schedule
```

### 渲染流程控制

渲染流程由 ``performSyncWorkOnRoot`` 或 `performConcurrentWorkOnRoot` 控制，这两个函数有些许不同，但大体流程是基本一致的：

1. **获取优先级：**获取本次渲染要处理的 `lanes`
2. **Render：**进行 Sync 或 Concurrent 渲染
3. **Commit：**根据渲染结果进入 Commit 阶段
4. **Shedule：**调用 `eunsureRootIsScheduled` 再次调度

#### Sync 渲染流程控制

如果该渲染的优先级是 `SyncLane` 等 Sync 优先级，将会进入 `performSyncWorkOnRoot` 开启的更新流程

1. **获取优先级：**通过 `getNextLanes` 获取本次更新要执行的优先级
2. **Render：**调用 `renderRootSync` 开始 Sync 渲染
3. **Commit：**根据渲染任务的终止状态判断是否可以调用 `commitRoot` 进入 Commit 阶段
4. **Shedule：**调用 `ensureRootIsScheduled` 再次调度

```typescript
export function performSyncWorkOnRoot(root: FiberRoot): null {
  if ((executionContext & (RenderContext | CommitContext)) !== NoContext) {
    throw new Error('Should not already be working')
  }
  // 1. 获取本次更新的优先级
  const lanes = getNextLanes(root, NoLanes)
  if (!includesSyncLane(lanes)) {
    ensureRootIsScheduled(root)
    return null
  }
  // 2. 开始渲染
  let exitStatus = renderRootSync(root, lanes)
  // ... 判断 exitStatus
  // 3. 进入 Commit 阶段
  const finishedWork: Fiber = root.current.alternate
  const finishedLanes: Lanes = lanes
  commitRoot(root)
  // 4. 再次调度，直到没有任务可以调度
  ensureRootIsScheduled(root)
  return null
}
```

#### Concurrent 渲染流程控制

首次渲染过程的优先级是 `DefaultLane`，将进入 `performConcurrentWorkOnRoot` 函数，他会：

1. **获取优先级：**通过 `getNextLanes` 获取本次更新要执行的优先级
2. **Render：**
   1. 通过优先级和 `didTimeout` 得到 `shouldTimeSlice`，判断使用 Sync 渲染 `renderRootSync` 还是 Concurrent 渲染 `renderRootConcurrent`
   2. 调用 `renderRootSync` 或 `renderRootConcurrent` 进行渲染，并获得渲染任务的终止状态 `exitStatus` 
3. **Commit：**根据渲染任务的终止状态判断是否可以调用 `commitRoot` 进入 Commit 阶段
4. **Schedule：**
   1. 调用 `ensureRootIsScheduled` 再次调度
   2. 确定该函数返回值，从而告诉 Scheduler 怎样继续调度

```typescript
export function performConcurrentWorkOnRoot(
  root: FiberRoot,
   // Scheduler 执行该函数的时候，会传入第二个参数 didTimeout 指示该任务是否已过期
  didTimeout: boolean,
) {
  if ((executionContext & (RenderContext | CommitContext)) !== NoContext) {
    throw new Error('Should not already be working')
  }
  
  const originalCallbackNode = root.callbackNode
  
  // 1. 获取本次更新要执行的优先级，可以暂时理解为 getHighestPriorityLane(root.pendingLanes)
  const lanes = getNextLanes(
    root,
    root === workInProgressRoot ? workInProgressRootRenderLanes: NoLanes,
  )
  if (lanes === NoLane) { return null }
  
  // 2.1. 需要 Sync 执行的 Lane、过期的 Lane、或者该任务由于 Scheduler 传给的参数导致该渲染需要 Sync 进行
  // 在开启了默认使用并发更新（Concurrent）的情况下，includesBlockingLane 将永远返回 false
  // 否则 DefaultLane 会认为是 BlockingLane，首次渲染时会返回 true
  const shouldTimeSlice =
    !includesBlockingLane(root, lanes) &&
    !includesExpiredLane(root, lanes) &&
    !didTimeout
  
  // 2.2. 开始执行 render 的具体任务
  let exitStatus = shouldTimeSlice
    ? renderRootConcurrent(root lanes)
    : renderRootSync(root, lanes)
  
  // ... 判断 exitStatus
  
  if (exitStatus === RootCompleted) {
    // 3. 进入 Commit 阶段（省略 commitRootWhenReady 中间过程）
    root.finishedWork: Fiber = root.current.alternate
    root.finishedLanes: Lanes = lanes
    commitRoot(root)
  }
  
  // 4.1. 再次调度，直至没有任务需要调度
  ensureRootIsScheduled(root)
  // 4.2. 按照 Scheduler 的约定，该函数的返回值将会被 Scheduler 继续调度
  // 因此通过 getContinuationForRoot 看需要继续调度什么函数
  return getContinuationForRoot(root, originalCallbackNode)
}
```

### 渲染

`renderRootSync` 和 `renderRootConcurrent` 就是用于进入渲染过程的函数，除了准备渲染过程所需的变量和做一些清理工作以外，主要的渲染过程都是调用 `workLoop` 完成的。

```typescript
function renderRootSync(root: FiberRoot, lanes: Lanes) {
  const prevExecutionContext = excutionContext
  excutionContext |= RenderContext
  
  // 若传入的待执行的 root 或 lanes 与当前 wipRoot 不同，则需要重新准备一个 wipRoot 并初始化全局变量
  // 否则沿用之前的
  if (workInProgressRoot !== root || workInProgressRootRenderLanes !== lanes) {
    prepareFreshStack(root, lanes)
  }
  
  // 进入 workLoop
  outer: do {
    try {
      // ...
      workLoopSync()
    } catch(thrownValue) {
      handleThrow(root, thrownValue)
    }
  } while (true)
  
  if (workInProgress !== null) {
    // Sync 渲染结束 workInProgress 一定是空的，否则就是哪里出问题了
    throw new Error('...')
  }
  
  workInProgressRoot = null
  workInProgressRootRenderLanes = NoLanes
  
  // 将 render 阶段收到的更新入队
  finishQueueingConcurrentUpdates()
  // completeWork 正常结束应该返回 RootCompleted
  return workInProgressRootExitStatus
}
```

#### 初始化渲染所需变量

开始一轮新的渲染之前，需要清空一些挂在 `root` 上的、和环境中的变量：

```typescript
function prepareFreshStack(root: FiberRoot, lanes: Lanes): Fiber {
  root.finishedWork = null
  root.finishedLanes = NoLanes
 	// ...
  workInProgressRoot = root
  // 根据 root.current 创建一个镜像
  const rootWorkInProgress = createWorkInProgress(root.current, null)
  workInProgress = rootWorkInProgress
  workInProgressRootRenderLanes = renderLanes = lanes
  // ...
  return rootWorkInProgress
}
```

#### workLoop

`workLoop` 会循环地将当前的 `workInProgress` 送入 `performUnitOfWork` 函数来执行具体的渲染工作，与 `performXXXWorkOnRoot` 和 `renderXXXRoot` 一样，他也有 Sync 和 Concurrent 两种版本。

两种版本唯一的区别在于 Sync 版本的 `workLoopSync` 会一直循环执行直到 `workInProgress` 为空，而 Concurrent 版本则会受到来自 Scheduler 的 `shouldYield` 函数控制，使得他可以提前中断。

Render 阶段的 Sync 和 Concurrent 的区别到此就结束了，他们后面的流程完全一样。

```typescript
function workLoopSync() {
  while (workInProgress !== null) {
    pewrformUnitOfWork(workInProgress)
  }
}

function workLoopConcurrent() {
  while (workInProgress !== null && !shouldYield()) {
    pewrformUnitOfWork(workInProgress)
  }
}
```

### 遍历 FiberTree 执行任务

不管是 Sync 还是 Concurrent 渲染，他们在 `workLoop` 中都将调用 `performUnitOfWork` 来执行具体的每一个 `Fiber` 上的任务：

1. 渲染的过程实际上就是遍历 FiberTree 的过程，React 采用深度优先（DFS）的方式对 FiberTree 进行遍历
2. `performUnitOfWork` 接受一个 `Fiber` 作为工作单元 `unitOfWork`
3. `unitOfWork` 参数实际上就是全局的 `workInProgress`，每次 `performUnitOfWork` 中都将更新这个变量，以便 `workLoop` 的下次循环中调用时使用新的 `workInProgress`
4. 每次 `performUnitOfWork` 的执行过程是无法中断的，Concurrent 任务中断的时机是在 `workLoop` 中每次对下一个 `workInProgress` 进行处理的时候

#### 遍历流程

React 使用 DFS 中来遍历 FiberTree。渲染过程实际上分为 **“递”** 和 **“归”** 两个部分：

- **在 “递” 的阶段，会执行 `beginWork` 方法，并沿着 `.child` 单链表向下遍历。** `beginWork` 方法中，会创建新 `Fiber` 或对旧的 `Fiber` 进行复用及更新。
- **在 “归” 的阶段，会执行 `completeWork` 方法，并先沿着 `.siblings` 遍历所有兄弟节点。如果有兄弟节点，将会沿着兄弟节点的 `.child` 向下遍历，即进入 “递” 阶段；如果没有兄弟节点，则会沿着 `.return` 向上遍历。 ** `completeWork` 方法中会根据标记创建或修改离屏 DOM。

下面仍然以这个例子进行说明：

```jsx
const El = <MyComp>         /* 1. beginWork(MyComp)  */  /* 14.completeWork(MyComp)  */
  <div id="div">            /* 2. beginWork(div)     */  /* 13.completeWork(div)     */
    <ul id="ul">            /* 3. beginWork(ul)      */  /* 10.completeWork(ul)      */
      <li id="li0"></li>    /* 4. beginWork(li0)     */  /* 5. completeWork(li0)     */
      <li id="li1"></li>    /* 6. beginWork(li1)     */  /* 7. completeWork(li1)     */
      <li id="li2"></li>    /* 8. beginWork(li1)     */  /* 9. completeWork(li1)     */
    </ul>
    <span id="span">        /* 11.beginWork(span)    */  /* 12.completeWork(li1)     */
      Hello!
    </span>
  </div>
</MyComp>
```

#### 简化版代码

```typescript
function performUnitOfWork(unitOfWork: Fiber) {
  // current 就是当前屏幕上的 UI 所对应的内容
  const current = unitOfWork.alternate
  // 沿着 child 向下遍历
  // beginWork 会返回 unitOfWork 的 child；首次渲染中，beginWork 会创建 child 并返回他
  const next = beginWork(current, unitOfWork, renderLanes)
  unitOfWork.memorizedProps = unitOfWork.pendingProps
  if (next === null) {
    // 当前遍历到叶节点了，进入 compelteWork 流程，里面会更新 workInProgress 的指向
    completeUnitOfWork(unitOfWork)
  } else {
    // 当前还没有遍历到叶节点，继续向下遍历
    workInProgress = next
  }
}

// 该函数相当于也是一个遍历函数，会遍历 siblings 和 return 并更新 workInProgress 指向
function completeUnitOfWork(unitOfWork: Fiber) {
  let completedWork: Fiber = unitOfWork
  // 沿着 return 向上遍历
  do {
    const current = completedWork.alternate
    const returnFiber = completedWork.return
    let next
    // commpleteWork 会返回 completedWork 的 sibling
    next = completeWork(current, completedWork, renderLanes)
    // 如果 completeWork 返回有新工作，退出本次遍历，再次进入向下遍历流程
    if (next !== null) {
      workInProgress = next
      return
    }
    // 如果有 sibling，退出本次向上遍历，再次进入向下遍历流程
    const siblingFiber = completedWork.sibling
    if (siblingFiber !== null) {
      workInProgreess = siblingFiber
      return
    }
    completedWork = returnFiber
    workInProgress = completedWork
  } while (completedWork !== null)
  // 向上遍历已经完成
  if (workInProgressRootExitStatus === RootInProgress) {
    workInProgressRootExitStatus = RootCompleted
  }
}
```

### beginWork

`begeinWork` 会根据 `Fiber` 的类型使用不同的函数进行处理：

```typescript
export function beginWork(current, workInProgress, renderLanes) {
  // 假设本次将会清空该 fiber 的 lanes
  workInProgress.lanes = NoLanes
  
  // 根据类型进入不同的处理函数
  switch (workInProgres.tag) {
    case HostRoot:
      return updateHostRoot(current, workInProgress, renderLanes)
    case IndeterminateComponent:
      return mountIndeterminateComponent(current, workInProgress, workInProgress.type, renderLanes)
    case 
  }
}
```

这些处理函数的大致流程可概括为：

1. **计算新的子节点：**处理 `UpdateQueue`、执行函数组件本体、执行类组件的渲染函数等，计算得到新的内部状态 `memorizedState` 和新的子节点
2. **创建子节点并增加标记：**使用 `reconcileChildren` 来处理新的子节点，为子节点或父节点 `flags` 添加标记，使得更新在 `commit` 阶段能得到合适的处理并应用到最终的宿主环境 UI 中

`flags` 与 `lanes` 类似，也是用二进制位来表示的，因此可以用按位或来打上标记，比如为节点打上 ”新增节点“ 标记 `Placement` ，可以使用 `Fiber.flags |= Placement`。

#### 不同类型的 beginWork

##### HostRoot

`HostRoot` 会进入 `updateHostRoot` 处理：

1. **计算新的子节点：**调用 `processUpdateQueue` 计算更新，得到新的 `element`
2. **创建子节点并增加标记**：调用 `reconcileChildren` 并传入 `element` 更新子节点

```typescript
function updateHostRoot(
  current: Fiber | null,
  workInProgress: Fiber,
  renderLanes: Lanes,
): Fiber | null {
  // 首次渲染为 null
  const nextProps = workInProgress.pendingProps
  // 首次渲染为 { element: null }
  const prevState = workInProgress.memorizedState
  // 首次渲染为 null
  const prevChildren = prevState.element
  
  // 若 current 和 workInProgress 中的 updateQueue 是同一个引用，为 workInProgress 创建一个新的 updateQueue
  cloneUpdateQueue(current, workInProgress)
  // 首次渲染 renderLane 为 DefaultLane
  // 且 updateQueue 中只有一个 Update 为 { payload: { element: ReactElement }, lane: DefaultLane }
  processUpdateQueue(current, workInProgress)
  // 在 processUpdateQueue 中已经计算完成，memorizedState 是最新的结算结果
  // 首次渲染中，计算结果为 Update.payload，即 { element: ReactElement }
  const nextState = workInProgress.memorizedState
  
  const nextChildren = nextState.element
  // 处理子节点（创建或更新）
  reconcileChildren(current, workInProgress, nextChildren, renderLanes)
  // 返回子节点，这个子节点将会在 performUnitOfWork 中成为下一个 workInProgress 继续向下遍历
  // 首次渲染中会返回新创建的子节点
  return workInProgress.child
}
```

##### IndeterminateComponent

函数组件对应的 `Fiber` 在创建时不会被归纳为 `FunctionComponent`，而是会被归纳为 `IndeterminateComponent`（未被确定类型的组件），并在 `begeinWork` 中进入 `mountIndeterminateComponent` 函数进行处理：

1. **计算新的子节点：**调用 `renderWithHooks` 调用函数组件本体进行更新并得到返回值 `value`
2. **创建子节点并增加标记**：调用 `reconcileChildren` 并传入 `value` 更新子节点

```typescript
function mountIndeterminateComponent(
  _current: null | Fiber,
  workInProgress: Fiber,
  Component: any,
  renderLanes: Lanes,
): Fiber {
  const props = workInProgress.pendingProps
  const value = renderWithHooks(
    null,
    workInProgress,
    Component,
    props,
    context, // ... 省略 context 获取
    renderLanes,
  )
  workInProgress.flags |= PerformedWork
  workInProgress.tag = FunctionComponent
  reconcileChildren(null, workInProgress, value, renderLanes)
  return workInProgress.child
}

function renderWithHooks<Props, SecondArg>(
  current: Fiber | null,
  workInProgress: Fiber,
  Component: (p: Props, arg: SecondArg) => any,
  props: Props,
  secondArg: SecondArg,
  nextRenderLanes: Lanes,
) {
  renderLanes = nextRenderLanes
  currentlyRenderingFiber = workInProgress
  workInProgress.memorizedState = null
  workInProgress.updateQueue = null
  workInProgress.lanes = NoLanes
  const children = Component(props, secondArg)
  return children
}
```

##### HostComponent

`HostComponent` 会进入 `updateHostComponent` 进行处理，诸如 `<div>` 等 DOM 元素对应的 `Fiber` 被称为 `HostComponent`。

1. **创建子节点并增加标记**：调用 `reconcileChildren` 并传入 `pendingProps.children` 更新子节点

```typescript
function updateHostComponent(
  current: Fiber | null,
  workInProgress: Fiber,
  renderLanes: Lanes,
) {
  // "div"
  const type = workInProgress.type
  const nextProps = workInProgress.pendingProps
  const prevProps = current !== null ? current.memorizedProps: null
  
  let nextChildren = nextProps.children
  // 子节点是否是文本节点
  const isDirectTextChild = shouldSetTextContent(type, nextProps)
  
  if (isDirectTextChild) {
    nextChildren = null
  } else if (prevProps !== null && shouldSetTextContent(type, nextProps)) {
    workInProgress.flgas |= ContentReset
  }
    
  // ...

  reconcileChildren(current, workInProgress, nextChildren, renderLanes)
}
```

#### reconcileChildren

在计算完当前 `Fiber` 上的 `updateQueue` 之后，会对他的子节点进行 Reconciliation，在这个过程中会创建或更新子结点，并根据 Reconciliation 的结果对 `workInProgress` 的 `child` 属性进行赋值。

```typescript
export function reconcileChildren(
  current: Fiber | null,
  workInProgress: Fiber,
  nextChildren: any,
  renderLanes: Lanes,
) {
  if (current === null) {
    // 这是一个新的、未被渲染过的 Component
    workInProgress.child = mountChildFibers(workInProgreess, null, nextChildren, renderLanes)
  } else {
    // 更新当前 Component
    workInProgress.child = reconcileChildFibers(workInProgress, current.child, nextChildren, renderLanes)
  }
}

// 这两个函数实际上都由 createChildReconciler 创建，区别仅在于“是否需要跟踪副作用”
export const reconcileChildFibers = createChildReconciler(true)
export const mountChildFibers = createChildReconciler(false)
```

什么叫 **是否跟踪副作用** 呢？

如果需要跟踪副作用，那么 `Fiber.flags` 上面就会增加各种各样的标记，比如 `Placement` 等。在 `commit` 阶段，这些副作用就会被 **提交** 到宿主环境的 UI 中。

反之如果不需要跟踪副作用，则不会加上这些标记，也就不会被提交到 UI 中。

首次渲染时有个小优化：只有 `HostRootFiber` 有对应的 `current`。也就是说，首次渲染中，只有在处理 `HostRootFiber`，生成他的子节点的时候会打上标记 ，而所有其他子节点将不会被打上标记，在 `commit` 阶段不会被处理。只有 `HostRootFiber` 会被提交。因此，只会进行一次与宿主环境 UI 的交互，不会有频繁的 DOM 操作。

##### CreateChildReconciler

`createChildReconciler` 接受参数 `shouldTrackEffects` 表示 **是否跟踪副作用**，这个值以闭包的形式保存起来，然后返回一个函数 `reconcileChildFibers` ，这个函数就是上面的 `reconcileChildren` 中使用的函数。

如果需要跟踪副作用，那么我们需要给 `Fiber.flags` 加上 `Placement` 等标记。 

返回的这个 `reconcileChildFibers` 函数会根据 `newChild` 的类型调用不同的辅助函数来帮助进行 Reconciliation：

| 类型            | 方法                                             |
| --------------- | ------------------------------------------------ |
| ReactElement    | `placeSingleChild(reconcileSingleElement(...))`  |
| Array           | `reconcileChildrenArray(...)`                    |
| String / Number | `placeSingleChild(reconcileSingleTextNode(...))` |

```typescript
function createChildReconciler(shouldTrackEffects: boolean) {
  return function reconcileChildFibers(
    returnFiber: Fiber,
    currentFirstChild: Fiber | null,
    newChild: any,
    lanes: Lanes,
  ): Fiber | null {
    // ...省略 Fragement 相关判断，如果是 Fragement，将其中的 children 取出来赋值给 newChild
    if (typeof newChild === 'object' && newChild !== null) {
      switch (newChild.$$typeof) {
        //...
        case REACT_ELEMENT_TYPE:
          return placeSingleChild(
            reconcileSingleElement(
              returnFiber,
              currentFirstChild,
              newChild,
              lanes,
            )
          )
      }
      if (isArray(newChild)) {
        return reconcileChildrenArray(
          returnFiber,
          currentFirstChild,
          newChild,
          lanes
        )
      }
    }
    if ((tyepof newChild === 'string' && newChild !== '') && typeof newChild === 'number') {
      return placeSingleChild(
        reconcileSingleTextNode(
          returnFiber,
          currentFirstChild,
          '' + newChild,
          lanes,
        )
      )
    }
    // 剩下的全部视为空并删除
    return deleteRemainingChildren(returnFiber, currentFirstChild)
  }
}
```

可以看到其中调用的函数大致分为两种类型：

- 一种是计算子节点变更的 `reconcileSingleElement` 等函数
- 一种是跟踪副作用的 `TrackSideEffects` 等函数，该函数通常只有在传入 `shouldTrackSideEffects` 为 `true` 的情况下才会执行，并为 `Fiber` 打上各种标记

##### ChildReconciliation

###### reconcileSingleElement

```typescript
function reconcileSingleElement(
  returnFiber: Fiber,
  currentFirstChild: Fiber | null,
  element: ReactElement,
  lanes: Lanes,
) {
  const key = element.key
  let child = currentFirstChild
  while (child !== null) {
    //... 首次渲染 child 为 null
  }
  const created = createFiberFromElement(element, returnFiber.mode, lanes)
  created.return = returnFiber
  return created
}
```

##### TrackSideEffects

###### placeSingleChild

`placeSingleChild` 用来给 `Fiber` 加上 `Placement` 标记：

```typescript
function placeSingleChild(newFiber: Fiber): Fiber {
  if (shouldTrackSideEffects && newFiber.alternate === null) {
    return newFiber
  }
}
```

###### deleteChild

`deleteChild` 会在父节点上标记 `ChildDeletion`，并用 `deletions` 数组将需要删除的节点存在父节点上。

```typescript
function deleteChild(
  returnFiber: Fiber,
  childToDelete: Fiber,
) {
  const deletions = returnFiber.deletions
  if (deletions === null) {
    returnFiber.deletions = [childToDelete]
    returnFiber.flags |= ChildDeletion
  } else {
    deletions.push(childToDelete)
  }
}
```



###### deleteRemainingChildren

`deleteRemainingChildren` 遍历传入节点的兄弟节点并全部调用 `deleteChild` 方法来删除：

```typescript
function deleteRemainingChildren(
  returnFiber: Fiber,
  currentFirstChild: Fiber
) {
  if (!shouldTrackEffects) {
    return null
  }
  let childToDelete = currentFirstChild
  while (childToDelete !== null) {
    deleteChild(returnFiber, childToDelete)
    childToDelete = childToDelete.sibling
  }
  return null
}
```

### completeWork

`completeWork` 也会根据不同的 `Fiber` 类型应用不同的处理方法，但基本都包括三个步骤：

1. **标记节点更新**
2. **构造离屏 DOM**
3. **Lanes 和 Flags 冒泡**

```typescript
function completeWork(
  current: Fiber | null,
  workInProgress: Fiber,
  renderLanes: Lanes,
): Fiber | null {
  const newProps = workInProgress.pendingProps
  switch (workInProgress.tag) {
    case FunctionComponent:
      // ...
      bubbleProperties(workInProgress)
    case HostRoot:
      // ...
      bubbleProperties(workInProgress)
    case HostComponent:
      const type = workInProgress.type
      if (current !== null && workInProgress.stateNode != null) {
        // ... 更新原有组件
      } else {
        // ... 构造离屏 DOM
      }
      bubbleProperties(workInProgress)
      return null
  }
}
```

#### 构造离屏 DOM

这里我们都是用浏览器作为宿主环境来举例子，所以为了方便起见就直接用 DOM 来描述宿主环境的 UI 结构。

在 `completeWork` 的过程中，`HostComponent` 会调用 DOM API 创建 DOM Element，并将子节点插入这个 Element 中，然后将其放在 `Fiber.stateNode` 上：

```typescript
// 获取 rootElement 的 DOMElement
const rootContainerInstance = getRootHostContainer()
// 创建该 HostComponent 的 DOMElement
const instance = createInstance(
  type, newProps, rootContainerInstance, currentHostContext, workInProgress
)
// 将子节点插入 instance 中
appendAllchildren(instance, workInProgress)
// 将 DOMElement 放在 stateNode 上
workInProgress.stateNode = instance
```

#### Lanes 和 Flags 冒泡

```typescript
function bubbleProperties(completeWork: Fiber) {
  let newChildLanes = NoLanes
  let subtreeFlags = NoFlags
  let child = completedWork.child
  while (child !== null) {
    newChildLanes = mergeLanes(newChildLanes, mergeLanes(child.lanes, child.childLanes))
    subtreeFlags |= child.subtreeFlags
    subtreeFlags |= child.flags
    child = child.sibling
  }
  completedWork.subtreeFlags |= subtreeFlags
  completedWork.childLanes = newChildLanes
}
```

## Commit

Commit 阶段会将刚刚渲染中标记的各种副作用 `flags` 提交到 UI 上。与 Render 阶段不同，Commit 阶段无法暂停或终止，他会在一次同步的代码中执行完成对整个 `FiberRoot` 的 Commit。

**Commit 阶段分为 三个子阶段，不同的副作用会在不同的子阶段得到处理：**

- **BeforeMutation**
- **Mutation：执行完成之后进行 `wip` 和 `current` 的切换**
- **Layout**

Commit 阶段的入口是 `commitRoot` 函数，不同的掩码指示了在 Commit 的不同子阶段是否有任务需要执行，比如 `MutationMask = Placement | Update | ...`，表示如果该 `Fiber` 有 `Placement` 或 `Update` 等副作用，那么他就需要在 Mutation 子阶段得到处理。

```typescript
function commitRoot(
  root: FiberRoot,
  recoverableErrors: any,
  transitions: Array<Transition>,
) {
  // ...
  const finishedWork = root.finishedWork
  const lanes = root.finishedLanes
  
  root.finishedWork = null
  root.finishedLanes = NoLanes
    
  // Commit 会同步执行完毕，所以他不再需要这些属性，可以先把他们置空，方便后续更新被调度
  root.callbackNode = null
  root.callbackPriority = NoLane
  
  let remainingLanes = mergeLanes(finishedWork.lanes, finishedWOrk.childLanes)
  
  // root.pendingLanes = remainingLanes
  markRootFinished(root, remainingLanes)
  
  const subtreeHasEffect =
    (finishedWork.subtreeFlags &
      (BeforeMutationMask | MutationMask | LayoutMask)) !== NoFlags
  const rootHasEffect =
    (finishedWork.flags &
      (BeforeMutationMask | MutationMask | LayoutMask)) !== NoFlags
  if (subtreeHasEffect || rootHasEffect) {
    commitBeforeMutationEffects(root, finishedWork)
    commitMutationEffects(root, finishedWork, lanes)
    root.current = finishedWork
    commitLayoutEffects(finishedWork, root, lanes)
  } else {
    root.current = finishedWork
  }
  ensureRootIsScheduled(root)
}
```

### 子阶段执行流程

三个子阶段均按照 DFS 对节点进行处理，且大部分操作都是在 **自下而上的后序过程中** 处理的。在目前 React 的源码中，`commitBeforeMutationEffects` 是采用的迭代写法，而 `commitMutationEffects` 和 `commitLayoutEffects` 中均采用递归写法，他们会先找到最下层的有该副作用的节点（该节点的 `subtreeFlags` 不再包含需要处理的副作用），然后再自下而上地处理（也有部分副作用是自上而下的前序过程中处理的）。

#### 迭代写法

`commitBeforeMutationEffects` 中采用迭代写法，其中 `commitBeforeMutationEffectsOnFiber` 用于处理节点。

```typescript
export function commitBeforeMutationEffects(
  root: FiberRoot,
  firstChild: Fiber,
) {
  //...
  commitBeforeMutationEffects_begin()
  //...
}

function commitBeforeMutationEffects_begin() {
  while (nextFeect !== null) {
    const fiber = nextEffect
    const child = fiber.child
    // 向下遍历找到有该副作用的最下层的子节点
    if ((fiber.subtreeFlags & BeforeMutationMask) !== NoFlags && child !== null) {
      child.return = fiber
      nextEffect = child
    } else {
      // 向上遍历
      commitBeforeMutationEffects_complete()
    }
  }
}

function commitBeforeMutationEffects_complete() {
  while (nextEffect !== null) {
    const fiber = nextEffect
    try {
      // 处理 Fiber 上的副作用（即 doPostWork）
      commitBeforeMutationEffectsOnFiber(fiber)
    }
    const sibling = fiber.sibling
    if (sibling !== null) {
      sibling.return = fiber.return
      nextEffect = sibling
      return
    }
    nextEffect = fiber.return
  }
}
```

如果将 `commitBeforeMutationEffects_complete` 函数合在 `commitBeforeMutationEffects_begin` 中，再稍微改写一下，就可以看出来跟之前抽象出来的迭代写法非常像了：

```typescript
function commitBeforeMutationEffects_begin() {
  let nextEffcet = root
  function procsee() {
    const fiber = nextEffect
    const child = fiber.child
    // 向下遍历找到有该副作用的最下层的子节点
    if ((fiber.subtreeFlags & BeforeMutationMask) !== NoFlags && child !== null) {
      child.return = fiber
      nextEffect = child
      return
    }
    // 向上遍历
    while (nextEffect !== null) {
      const fiber = nextEffect
      try {
        // doPostWork
        commitBeforeMutationEffectsOnFiber(fiber)
      }
      const sibling = fiber.sibling
      if (sibling !== null) {
        sibling.return = fiber.return
        nextEffect = sibling
        return
      }
      nextEffect = fiber.return
    }
  }
  while (nextFeect !== null) {
    process()
  }
}
```

#### 递归写法

`commitMutationEffects` 和 `commitLayoutEffects` 比较类似，均采用递归写法，下面用前者举例：

```typescript
export function commitMutationEffects(
  root: FiberRoot,
  finishedWork: Fiber,
  committedLanes: Lanes,
) {
  //...
  commitMutationEffectsOnFiber(finishedWork, root, committedLanes)
  //...
}

function commitMutationEffectsOnFiber(
  finishedWork: Fiber,
  root: FiberRoot,
  lanes: Lanes,
) {
  switch (finishedWork.tag) {
    case FunctionComponent:
      recursivelyTraverseMutationEffects(root, finishedWork, lanes)
      // 处理 Fiber 上的副作用（即 doPostWork）
      commitReconcilationEffects(finishedWork)
  }
  // ...
}

function recursivelyTraverseMutationEffects(
  root: FiberRoot,
  parentFiber: Fiber,
  lanes: Lanes,
) {
  // 处理 deletions，先序处理，即 doPreWork
  const deletions = parentFiber.deletions
  if (deletions !== null) {
    for (let i = 0; i < deletions.length; i++) {
      commitDeletionEffects(root, parentFiber, deletions[i])
    }
  }
  if (parentFiber.subtreeFlags & MutationMask) {
    let child = parentFiber.child
    while (child !== null) {
      commitMutationEffectsOnFiber(child, root, lanes)
      child = child.sibling
    }
  }
}
```

我们将 `recursivelyTraverseMutationEffects` 与 `commitMutationEffectsOnFiber` 合并之后就可以看到与原来的递归模板代码也很像：

```typescript
function recursivelyTraverseMutationEffects(
  root: FiberRoot,
  parentFiber: Fiber,
  lanes: Lanes,
) {
  // doPreWork
  const deletions = parentFiber.deletions
  if (deletions !== null) {
    for (let i = 0; i < deletions.length; i++) {
      commitDeletionEffects(root, parentFiber, deletions[i])
    }
  }
  if (parentFiber.subtreeFlags & MutationMask) {
    let child = parentFiber.child
    while (child !== null) {
      switch (child.tag) {
        case FunctionComponent:
          // doPostWork
          recursivelyTraverseMutationEffects(root, child, lanes)
          commitReconcilationEffects(child)
      }
      child = child.sibling
    }
  }
}
```

### BeforeMutation

BeforeMutation 阶段主要有两个作用：

- 执行 `ClassComponenent` 的 `getSnapshotBeforeUpdate` 方法
- 清空 `HostRoot` 挂载的内容

### Mutation

对于 HostComponent 来说，Mutation 阶段会进行 DOM 元素的增、删、改：

- **删除元素：**在向下遍历的过程中会前序执行删除操作，该操作会将 Render 阶段 `beginWOrk` 中往 `fiber.deletions` 里添加的元素删除掉，删除会有以下过程
  - 找到最近的祖先 Host
  - DFS 删除子树的每一个节点
- **插入、移动元素：**
  - 找到最近的祖先 Host
  - 找到 `parentNode.insertBefore` 所需的元素，注意这个找到的元素本身不能被标记为 `Placement`
  - 执行 `parentNode.insertBefore` 或 `parentNode.appendChild`

**Mutation** 执行完之后将进行 FiberTree 的切换：

```typescript
root.current = finishedWork
```

### Layout

在 DFS 中会前序执行 `OffscreenComponent` 的显隐逻辑，后序执行 `useLayoutEffect` 回调。

# React 中的事件系统

React 在首次渲染的过程中会给 `container` 绑定所有支持事件的监听，以事件委托的形式处理所有在 `container` 内触发的事件。当有事件触发时，React 会：

1. 通过 `nativeEvent.target` 上的属性找到这个元素对应的 `Fiber`（React 中的 `HostComponent.sateNode` 里面存了两个属性：`[internalInstanceKey]` 指向 `HostComponentFiber` 、 `[internalPropsKey]` 指向他的 `props`）
2. 遍历 `Fiber` 的所有祖先节点，并且通过 `Fiber.stateNode[internalPropsKey]` 找到 `props` 中对应事件的监听函数，收集到起来
3. 构造一个通用的 `SyntheticEvent` 对象，然后顺着事件监听函数的顺序调用事件处理函数

# Hooks

在代码结构上讲，Hooks 有几点比较特别的地方：

1. Hooks 都是定义在 `react-reconciler` 包的（这个包最终会被合并到 `react-dom` 中），但是我们引入 Hooks 却是在 `react` 包引入的
2. 很多 Hooks 在组件第一次渲染和后续渲染时的表现都不同，比如 `useEffect` 第一次渲染不论如何都会执行回调、`useState` 第一次渲染会使用初始值等

事实上，在 `react` 包里有一个全局变量 `__SECRET_INTERNALS_DO_NOT_USE_OR_YOU_WILL_BE_FIRED`，里面有个 `ReactCurrentDispatcher` 属性，该属性就存放了当前的 `useState` `useEffect` 等，因此 `react` 包可以使用到这些 Hooks。 

此外，React 中原生的 `useXXX` 其实都有两套实现，分别对应 `mount` 和 `update` 两种情况，`reac-reconciler` 中 `renderWithHooks` 执行的时候，会选择根据是首次渲染还是后续更新（通过 `current` 是否有值来判断）合适的 `useXXX` 函数，将其挂载到 `ReactCurrentDispatcher` 上。

## Hooks 通用流程

在 Hooks 运行的过程中，React 会维护两个全局变量：

- `workInProgressHook`：当前正在处理的 `Hook`
- `currentHook`：current `Fiber` 中的 `Hook`，可以理解为上一次渲染的 `Hook`

### workInProgressHook

#### mountWorkInProgressHook

在组件首次渲染时，每个原生 `useXXX` 都会执行 `mountWorkInProgressHook` 来创建一个新的 `Hook`，如果当前 `Hook` 是该组件的第一个 `Hook`，则会将 `currentlyRenderingFiber.memorizedState` 指向他：

```typescript
function mountWorkInProgressHook() {
  const hook: Hook = {
    memorizedState: null,
    baseState: null,
    baseQueue: null,
    queue: null,
    next: null
  }
  if (workInProgressHook === null) {
    currentlyRenderingFiber.memorizedState = workInProgressHook = hook
  } else {
    workInProgressHook = workInProgressHook.next = hook
  }
  return workInProgressHook
}
```

#### updateWorkInProgressHook

```typescript
function updateWorkInProgressHook() {
  let nextCurrentHook
  if (currentHook === null) {
    // 当前是再次渲染时遇到的第一个 Hook
    // 从 curret.memorizedState 中取到上一次的 Hook
    const current = currentlyRenderingFiber.alternate
    if (current !== null) {
      nextCurrentHook = current.memorizedState
    } else {
      nextCurrentHook = null
    }
    // 当前不是第一个 Hook，直接取链表的下一项
  } else {
    nextCurrentHook = currentHook.next
  }
  // 从 WIP 中捡上一次已经不要了的 Hook，看能不能复用
  let nextWorkInProgressHook
  if (workInProgressHook === null) {
    nextWorkInProgressHook = currentlyRenderingFiber.memorizedState
  } else {
    nextWorkInProgressHook = nextWorkInProgressHook.next
  }
  if (nextWorkInProgressHook !== null) {
    // 捡垃圾复用一把
    workInProgressHook = nextWorkInProgressHook
    nextWorkInProgressHook = nextWorkInProgressHook.next
    currentHook = nextCurrentHook
  } else {
    // 只能重新建一个 Hook
    if (nextCurrentHook === null) {
      throw new Error('要么现在不在 Render 阶段（currentFiber === null），要么这次 Render 的 Hook 比上一次多')
    }
    currentHook = nextCurrentHook
    const newHook = {
      memorizedState: currentHook.memorizedHook,
      baseState: currentHook.baseState,
      baseQueue: currentHook.baseQueue,
      queue: currentHook.queue,
      next: null
    }
    if (workInProgressHook === null) {
      currentlyRenderingFiber.memorizedState = workInProgressHook = newHook
    } else {
      workInProgressHook = workInProgressHook.next = newHook
    }
  }
  return workInProgressHook
}
```



## useState

```typescript
function basicStateReducer<S>(state: S, action: BasicStateAction<S>): S {
  return typeof action === 'function' ? action(state) : action
}
```



```typescript
function mountState(initialState: (() => S) | S) {
  // 创建一个新的 Hook
  const hook = mountWorkInProgressHook()
  if (typeof initialState === 'function') {
    initialState = initialState()
  }
  hook.memorizedState = hook.baseState = initialState
  const queue: Update<S, BasicStateAction<S>> = {
    pending: null,
    lanes: NoLanes,
    dispatch: null,
    lastRenderedReducer: basicStateReducer,
    lastRenderedState: initialState,
  }
  hook.queue = queue
  const dipatch = dispatchSetState.bind(null, currentlyRenderingFiber, queue)
  return [hoo.memorizedState, dispatch]
}
```

```typescript
function dispatchSetState<S, A>(
  fiber: Fiber,
  queue: Update<S, A>,
  action: A,
) {
  const lane = requestUpdateLane(fiber)
  const update = {
    lane,
    reverLane: NoLane,
    action,
    next: null,
  }
  const alternate = fiber.alternate
  // 调用 enqueueUpdate 把 Queue 推入 Fiber 的 UpdateQueue 中
  const root = enqueueConcurrentHookUpdate(fiber, queue, update, lane)
  if (root !== null) {
    scheduleUpdateOnFiber(root, fiber, lane)
  }
}
```

```typescript
function updateState(initialState) {
  return updateReducer(basicStateReducer, initialState)
}

function updateReducer(reducer) {
  const hook = updateWorkInProgressHook()
  const queue = hook.queue
  // 计算新 state 的算法与 UpdateQueue 基本一致，只是变量不同
  // ...
  return [hook.memorizedState, queue.dispatch]
}
```

## useEffect

与 Effect 相关的 Hooks 一共又三个，包括：

- `useEffect`：`commit` 完成之后异步执行
- `useLayoutEffect`：在 Commit 的 Layout 子阶段同步执行
- `useInsertionEffect`，在 Commit 的 Mutation 子阶段同步执行，此时无法访问对 DOM 的引用

React 为这些 Hooks 准备了一套抽象数据结构 `Effect`：

```typescript
export type Effect {
  tag: Flags
  // Effect 回调函数
  create: EffectCallback
  // Effect 回调函数的返回值
  destory: EffectCallback
  // 依赖项
  deps: EffectDeps
  // 与当前 FunctionComponent 的其他 Effect 组成环状链表
  next: Effect | null
}
```

### 声明阶段

在对函数组件执行 `renderWithHooks` 时，会分别执行 `mountEffect` 和 `updateEffect` 方法，他们会：

1. 创建 Effect 环状链表，并放到 `fiber.updateQueue` 上
2. `updateEffect` 会浅比较依赖项
3. 在 `tag` 中标记本次渲染之后是否需要执行该 Effect

### 调度阶段

由于 `useEffect` 的回调会在 Commit 完成之后异步执行，因此会在 Commit 阶段的三个子阶段之前进行调度：

```typescript
if (
  (finishedWork.subtreeFlags & PassiveMask) !== NoFlags ||
  (finishedWork.flags & PassiveMask) !== NoFlags
) {
  // ...
  scheduleCallback(NormalSchedulerPriority, () => {
    flushPassiveEffects()
    return null
  })
}
```

而另外两种 Effect 由于是同步执行的，因此没有调度阶段。

### 执行阶段

1. 使用后序遍历 FiberTree，遍历每个 Fiber 的 Effect 链表，依次执行 `effect.detroy` 方法
2. 所有需要执行的 `effect.destory` 执行完成之后，再次后序遍历 FiberTree、每个 Fiber 的 Effect 链表，执行 `effect.create`  方法

