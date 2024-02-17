---
title: React Reconciliation
date: 2023-06-23 16:41:41
categories:
  - - 前端
tags:
  - FX
---

现代前端框架的原理基本都可以用 `UI = f(State)`  来简单概括。其中 React 从状态更新到 UI 变化中间经历的过程，被称为 **Reconciliation**，完成这个过程的工具就是 **Reconciler**，他的核心代码都在 `react-reconciler` 这个包中。这个包不会耦合具体的宿主环境的代码，他只提供对状态更新流程的实现与封装，不同的宿主环境会通过不同的包来调用 Reconciler。

比如对于浏览器环境，React 提供了 `react-dom` 包，里面实现了操作 DOM 的方法。

除此之外，React 还提供了 `react` 包，他提供了诸如 [[React.createElement]] 等通用的方法，以及暴露给开发者的 React 上层特性，比如 [[React Hooks|Hooks]] 等。

<!-- more -->
# Overview

Reconciliation 大致可以看做四步：

1. Triggering a render
2. [[React Scheduling|Scheduling]] the tasks
3. [[React Rendering|Rendering]] the component
4. [[React Committing|Committing]] to the DOM

[尽管 React 文档中](https://react.dev/learn/render-and-commit) 并没有提及 Scheduling，但其在 React 源码中是有体现的，他对于 React 的并发更新尤为重要。

## Step 1: Trigger a render

两个原因：

1. 组件首次渲染：通过 `createRoot` 和 `render` 方法。

   ```jsx
   import { createRoot } from 'react-dom/client'
   const root = createRoot(document.getElementById('root'))
   root.render(<Image />)
   ```

2. 组件或他的某个祖先 State 改变（通过 set 方法）。

## Step 2: React schedules the updates

React Scheduler 会根据更新触发的原因，来赋予更新不同的优先级，并在安排他们在接下来的微任务或宏任务中进行。

## Step 3: React renders your component

所谓 [[React Rendering|渲染（Rendering）]] 指的就是 React 调用 [[React Elements & Components|组件（函数）]]：

- 首次渲染，React 将会调用 root 组件；
- 后续渲染，React 会调用触发此次渲染的组件函数。

这个过程是递归的：如果更新的组件返回了别的组件，React 会在下次渲染那个组件；如果那个组件也返回了别的组件，React 又会在下一次渲染别的组件……

以这段代码举例：

```jsx
export default function Gallery() {
  return (
    <section>
      <h1>Inspiring Sculptures</h1>
      <Image />
      <Image />
      <Image />
    </section>
  );
}

function Image() {
  return (
    <img
      src="https://i.imgur.com/ZF6s192.jpg"
      alt="'Floralis Genérica' by Eduardo Catalano: a gigantic metallic flower sculpture with reflective petals"
    />
  );
}
```

- 首次渲染中，React 会为 `<section>` `<h1> ` `<img>` 创建 DOM 节点；
- 重新渲染时，React 会计算他们的属性，看看谁的属性自上次渲染之后发生了变化，**但是他在进入下一个阶段前，都不会根据这个信息做任何事情**。

注意，对于同样的输入，一个组件应该永远有同样的输出。这可以通过 `<Sctrict Mode>` 来进行检查，在严格模式下， React 会调用一个组件两次来帮助发现问题，严格模式在生产环境下不生效。

## Step 4: React commits changes to the DOM

渲染完成后，React 会修改 DOM：

- 首次渲染时，React 会使用 `appendChild()` 将所有创建的 DOM 节点放到页面上；
- 重新渲染时，React 只会执行在渲染阶段计算出来的少量变动到 DOM 上。

# Data Structure Definitions

要完成 React Reconciliation 过程，需要定义数量众多的新数据结构，这包括：

- [[ReactDOMRoot]]
- [[React Fiber|Fiber、FiberTree 和 FiberRoot]]
- [[React Lane|Lane]]
- [[React Update|Update 和 UpdateQueue]]
- [[React Hook (Data Structure)|Hook]]

初次阅读时可以跳过枯燥的定义部分，后面遇到的时候再返回来查阅。

# Data Structures and Algorithms

React 的源码中使用了几个非常典型的数据结构和算法，包括各种链表的变体、深度优先遍历、位运算等。

## 链表

[[Linked List|链表]]是 React 中使用较多的数据结构之一，其中有两种链表较为特殊：**一种是以链表为基础的树形结构，一种是环形链表。**

### 单链表树

FiberTree 就是单链表[[Tree|树]]，他跟传统的树有些不同，他是由一条条链表组成的，更像是一个鱼骨形的结构。

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

React 中大量使用 [[Binary Tree#Deep First Traversal|DFS]] 来处理节点，基本都按照下列代码的顺序执行：

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

可以看到这个写法的遍历顺序与[[Backtracing|回溯算法]]其实非常像。

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

Scheduler 中使用[[Heap#Priority Queue & Heap|优先级队列]]来为任务进行排序，并保证每次出队的任务均是优先级最高的任务，优先级队列有多种实现方式，React 的 Scheduler 是采用常用的[[Heap#Priority Queue & Heap|小顶堆]]来实现的。

# Initial Render

React 的更新流程分为 **[[React Scheduling|Schedule]]、[[React Rendering|Render]] 和 [[React Committing|Commit]]** 三个阶段：

- **Schedule 阶段** 会挑出一批优先级，并为不同优先级准备不同的调度策略；
- **Render 阶段** 会执行 Fiber 节点的重新计算，并标记好需要执行的副作用；
- **Commit 阶段** 分为 **BeforeMutation**、**Mutation** 和 **Layout** 三个子阶段，在 **Mutation 阶段** 会完成 UI 相关的副作用，并且会完成 `workInProgress` 与 `current` FiberTree 的切换。 

## Before Scheduling

首次渲染中，Schedule 阶段之前，会有创建 `root` 的过程，然后才会进入正式的 Reconciliation 流程。

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

1. **createRoot：** 来自 `react-dom` 包的 `createRoot` 方法获取 Container Element 相关的信息，并调用 `react-reconciler` 中的方法来初始化 [[React Fiber|FiberRoot]] 和 `HostRootFiber`；
2. **createElement：** 来自 `react` 包的 `createElement` 将 JSX 转换结果再次转换为 `ReactElement`；
3. **render：** 调用 `createRoot` 返回对象的 `render` 方法，并传入 `ReactElement`，该方法会调用 `react-reconciler` 中的方法进入 Reconciliation 流程。

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

# Events in React

React 在首次渲染的过程中会给 `container` 绑定所有支持事件的监听，以事件委托的形式处理所有在 `container` 内触发的事件。当有事件触发时，React 会：

1. 通过 `nativeEvent.target` 上的属性找到这个元素对应的 `Fiber`（React 中的 `HostComponent.sateNode` 里面存了两个属性：`[internalInstanceKey]` 指向 `HostComponentFiber` 、 `[internalPropsKey]` 指向他的 `props`）
2. 遍历 `Fiber` 的所有祖先节点，并且通过 `Fiber.stateNode[internalPropsKey]` 找到 `props` 中对应事件的监听函数，收集到起来
3. 构造一个通用的 `SyntheticEvent` 对象，然后顺着事件监听函数的顺序调用事件处理函数

# Hooks

在代码结构上讲，[[React Hooks|Hooks]] 有几点比较特别的地方：

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

[[useState]]

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

[[React Effects]]

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