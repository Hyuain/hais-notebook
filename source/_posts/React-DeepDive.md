---
title: React - Deep Dive
date: 2023-06-23 16:41:41
categories:
  - [前端]
---

与另一篇 React 笔记不同，本文主要是深入讨论 React 的源码与设计理念。

<!-- more -->

Clone React 项目之后，在 README 中找到调试 React 的方法，具体可参照 [How to Contribute](https://legacy.reactjs.org/docs/how-to-contribute.html)。然后按照文中安装依赖并 Build 之后，将 `fixture/packaging/babel-standalone/dev.html` 作为我们的 Playground。

现代前端框架的原理基本都可以用 `UI = f(State)`  来简单概括。其中 React 从状态更新到 UI 变化中间经历的过程，被称为 **Reconciliation**，完成这个过程的工具就是 **Reconciler**，他的核心代码都在 `react-reconciler` 这个包中。这个包不会耦合具体的宿主环境的代码，他只提供对状态更新流程的实现与封装，不同的宿主环境会通过不同的包来调用 Reconciler。

比如对于浏览器环境，React 提供了 `react-dom` 包，里面实现了操作 DOM 的方法。

除此之外，React 还提供了 `react` 包，他提供了诸如 `createElement` 等通用的方法，以及暴露给开发者的React 上层特性，比如 Hooks 等。

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

1. 来自 `react-dom` 包的 `createRoot` 方法获取 Container Element 相关的信息，并调用 `react-reconciler` 中的方法进行初始化；
2. 来自 `react` 包的 `createElement` 将 JSX 转换结果再次转换为 `ReactElement`；
3. 调用 `createRoot` 返回对象的 `render` 方法，并传入 `ReactElement`，该方法会调用 `react-reconciler` 中的方法进入 Reconciliation 流程。

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

首先是 `createRoot` 函数，该函数位于 `react-dom/ReactDOMRoot.js`，主要用于：

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

`createContainer` 位于 `reac-reconciler/src/ReactFiberReconciler.js` 中，他直接调用了 `createFiberRoot`

```typescript
function createContainer(...args) {
  return createFiberRoot(...)
}
```

`createFiberRoot` 位于 `react-reconciler/src/ReactFiberRoot.js` 中，他：

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
  
  // 3. 初始化 memorizedState，可以看做是用来记录某些值的
  const initialState: RootState = {
    // 首次渲染中 initialChildren 为 null
    element: initialChildren,
  }
  uninitializedFiber.memorizedState = initialState
  
  // 3. 初始化 updateQueue，稍后进入渲染流程后将会提到，用于存取 Update
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

1. 获取优先级（lane）
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
  // 获取当前更新的优先级（lane）
  const lane = requestUpdateLane(container.current)
  // 创建 Update，他的 payload 就是传入的 ReactElement
  const update = createUpdate(lane)
  update.payload = {
    element,
  }
  // 将该更新入队，并且会返回当前 Fiber 对应的 FiberRoot
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

### 数据结构

要完成上文的过程，已经需要不少的数据结构进行支撑了，包括 `ReactDOMRoot`  `Fiber `  `FiberRoot`  `Update`  `UpdateQueue`  `Lane` 等，了解这些数据结构对了解 React Reconciliation 至关重要。

#### ReactDOMRoot

ReactDOMRoot 实际上就只是暴露了两个方法给我们使用，一个是 `render`，一个是 `unmount`。通过 `render` 方法，我们可以传入 `ReactElement` 然后开启首次渲染。

#### Fiber

> Fiber 就是 Component 上的需要被执行的、或已经执行完成的工作，每个 Component 可能有多个 Fiber。
>
> ——React 源码中的注释

`Fiber` 可以暂时先理解为虚拟节点，他有不同的类型，比如 `FunctionComponent` `ClassComponent` `HostRoot` `HostText` `Fragment` 等，这标记着该 `Fiber` 的类型。也就是说，一个 Fiber 节点可以对应某个 React 组件或某个 DOM 节点等。

> 此外，由于 React 源码使用了 Flow。因此其实可能某个 Fiber 类型并不需要某些属性，但 React 并没有将他们拆分开来，而是都直接散开放在了 `Fiber` 这一个数据结构中。

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
  // 内部 State 更新和回调的列表
  updateQueue: unknown
  // 上一次用来创建输出的 State
  memorizedState: any
  
  /* Effect（副作用）相关的属性 */
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

如果之后比如 `<h1>` 触发了更新，其中的文字发生了变化，React 不会直接在原来的 FiberTree 上执行更改，而是会建立一棵新的 FiberTree，将其作为 “工作中”  `WorkInProgress` ；原来的树对应着屏幕上当下显示的内容（被称为 `current`）。他们节点的 `alternate` 属性相互指着对方。

```text
HostRootFiber (Current) -> Fiber (<App />) -> Fiber (<h1>Hello World!</h1>)
HostRootFiber (WIP)     -> Fiber (<App />) -> Fiber (<h1>NEW World!</h1>)
```

当这棵新的树创建完毕、内容更新到 DOM 中之后，两棵树的地位随即发生互换。当下的 `WIP` 将会成为新的 `current`，而 `current` 则成为 `WIP`。

```text
HostRootFiber (WIP)     -> Fiber (<App />) -> Fiber (<h1>Hello World!</h1>)
HostRootFiber (Current) -> Fiber (<App />) -> Fiber (<h1>NEW World!</h1>)
```

这种机制被称为 **双缓存机制**，可以确保屏幕上不会出现只渲染了一半的结果。

##### FiberTree

FiberTree 不是一种额外定义的类，他只是由众多 `Fiber` 节点组成的单链表树形结构。`Fiber` 中的 `child` 和 `sibling` 就构成了这个数据结构，`return` 在其中也起到了一定的辅助作用。

`Fiber` 中的 `child` 指针指向子结点，`sibling` 指向下一个兄弟节点，`return` 指向任务执行完需要返回的地方，在这里可以理解为父结点。

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

| Fiber       | Child | Sibling | Return      |
| ----------- | ----- | ------- | ----------- |
| MyComponent | div   | null    | null*       |
| div         | ul    | null    | MyComponent |
| ul          | li0   | span    | div         |
| li0         | null  | li1     | ul          |
| li1         | null  | li2     | ul          |
| li2         | null  | null    | ul          |
| span        | null* | null    | div         |

*需要具体情况具体分析，可能是别的 Fiber

**React 中，为了简化，没有其他兄弟节点的文本节点不能单独成为一个 `Fiber`，所以 `span` 没有子节点

#### FiberRoot

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
  // FiberRoot 的类型，分为 ConcurrentRoot 和 LegacyRoot，对应新版的并发更新和之前的同步更新
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

#### Lane

React 中用 `Lane` 表示优先级这一概念。`Lane` 实际上就是二进制数字类型，每种优先级用不同的位为 `1` 来表示，用二进制来表示优先级就可以很方便地进行二进制运算来表达 “集合” 或者说 “批” 的概念。因为除去符号位之后一共还可以有 31 位，所以一共有 31 个 `Lane`，比如：

```typescript
export const NoLane: Lane = 0b0000000000000000000000000000000
export const SyncLane: Lane = 0b0000000000000000000000000000010
export const InputContinuousLane: Lane = 0b0000000000000000000000000001000
export const DefaultLane: Lane =  0b0000000000000000000000000100000
```

一般来讲，除了 `NoLane` 以外，数字越小代表优先级越高。

除了 `Lane` 以外，React 中还有 `Lanes`，实际上就是一群 `Lane` 做按位或运算的结果，表示 `Lane` 的集合。

##### requestUpdateLane

该函数位于 `react-reconciler/src/ReactFiberWorkLoop.js`，用于获取当前更新对应的优先级。

```typescript
export function requestUpdateLane(fiber: Fiber): Lane {
  // 特殊情况处理
  if ((fiber.mode & ConcurrentMode) === NoMode) {
    // 同步模式
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

##### 其他位运算

React 还使用了一些关于 `Lane` 和 `Lanes` 的位运算：

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

#### Update

`Update` 是 React 中用来承载 *改变 State 的更新任务* 的数据结构，React 中通过 `createUpdate` 来创建 `Update`。

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

#### UpdateQueue

`UpdateQueue` 是 React 中用来存放和消费 `Update` 的数据结构，`initializeUpdateQueue` 负责初始化 `Fiber` 上的 `updateQueue`，`enqueueUpdate` 用于将 `Update` 加入 `UpdateQueue` 中，`processUpdateQueue` 则用于消费 `Update`。

```typescript
export type UpdateQueue<State> = {
  baseState: State
  shared: SharedQueue<State>
  callbacks: Array<() => any> | null
}

export type SharedQueue<State> = {
  pending: Update<State> | null
  lanes: Lanes
  hiddenCallbacks: Array<() => any> | null
}
```

```typescript
export function enqueueUpdate
```

// TBC

## Schedule

React 的任务调度分为 **宏任务调度** 和 **微任务调度** 两种，React 会根据任务的优先级采取不同的调度策略，这个选择调度策略以及进行任务调度的阶段，即为 Schedule 阶段。

Schedule 阶段主要用到 `react-reconciler` 和 `scheduler` 。

`react-reconciler` 主要负责在 React 的工作流中进行任务调度，而涉及到具体的调度的具体实现（比如任务队列的维护，具体使用什么 API 让宿主环境在什么时候执行什么优先级的任务等）则会根据是宏任务还是微任务而有所不同：

React 中所有的更新工作都会被调度，或是在微任务中执行、或是在宏任务重执行**，因此 React 中并没有严格意义上的同步更新（我们后文提到的“同步”，如果没有特殊说明，都是指的在微任务中执行的、React 所定义的同步）**，这也是 `setState` 并不会同步生效的原因。

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
  
  // 每次 root 收到更新的时候，都标记可能有同步任务要做
  // 如果这个标记为 false，flushSync 将会跳过执行同步任务的过程
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

#### 选择优先级并进行调度

这个 `scheduleTaskForRootDuringMicrotask` 函数会获取到本轮调度应该执行的优先级 `nextLanes`，并进行调度：

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
    // 在本次调度任务的微任务最后，总是会 flush 一遍同步任务，因此不需要额外调度
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

#### 执行同步任务

首次渲染过程，优先级为 `DefaultLane`，不是同步任务，该函数根据 `mightHavePendingSyncWork === false` 将跳过具体内容的执行。

```typescript
function flushSyncWorkAcrossRoots() {
  if (isFlushing) { return }
  if (!mightHavePendingSyncWork) { return }
  // ...
}
```

## Render

React Render 阶段也由 `react-reconciler` 包进行处理，他有两个入口函数，一个是同步优先级使用的 `performSyncWorkOnRoot`，一个是其他优先级（并发更新）使用的 `performConcurrentWorkOnRoot`。

### 开始渲染过程

渲染过程由 ``performSyncWorkOnRoot`` 或 `performConcurrentWorkOnRoot` 开启，这两个些许不同，但大体流程是基本一致的：

1. 执行 Passive Effects（需要执行的 `useEffect` 回调）
2. 获取本次渲染要执行的优先级
3. 进行同步或并发渲染
4. 根据渲染结果进入 Commit 阶段
5. 调用 `eunsureRootIsScheduled` 再次调度

#### 开始同步渲染过程

如果该渲染的优先级是 `SyncLane` 等同步优先级，将会进入同步 `performSyncWorkOnRoot` 开启的更新流程：

1. 通过 `flushPassiveEffects` 执行 Passive Effects
2. 通过 `getNextLanes` 获取本次更新要执行的优先级
3. 调用 `renderRootSync` 开始同步渲染
4. 根据渲染任务的终止状态判断是否可以调用 `commitRoot` 进入 Commit 阶段
5. 调用 `ensureRootIsScheduled` 再次调度

```typescript
export function performSyncWorkOnRoot(root: FiberRoot): null {
  if ((executionContext & (RenderContext | CommitContext)) !== NoContext) {
    throw new Error('Should not already be working')
  }
  // 1. 执行 PassiveEffects，因为他可能会产生新的更新 
  flushPassiveEffects()
  // 2. 获取本次更新的优先级
  const lanes = getNextLanes(root, NoLanes)
  if (!includesSyncLane(lanes)) {
    ensureRootIsScheduled(root)
    return null
  }
  // 3. 开始渲染
  let exitStatus = renderRootSync(root, lanes)
  // ... 判断 exitStatus
  // 4. 进入 Commit 阶段
  const finishedWork: Fiber = root.current.alternate
  const finishedLanes: Lanes = lanes
  commitRoot(root)
  // 5. 再次调度，直到没有任务可以调度
  ensureRootIsScheduled(root)
  return null
}
```

#### 开始并发渲染过程

首次渲染过程的优先级是 `DefaultLane`，将进入 `performConcurrentWorkOnRoot` 函数，他会：

1. 通过 `flushPassiveEffects` 执行 Passive Effects，并检查他们是否产生了新的更高优先级的任务
2. 通过 `getNextLanes` 获取本次更新要执行的优先级
3. 通过优先级和 `didTimeout` 得到 `shouldTimeSlice`，判断使用同步渲染 `renderRootSync` 还是并发渲染 `renderRootConcurrent`
4. 调用 `renderRootSync` 或 `renderRootConcurrent` 进行渲染，并获得渲染任务的终止状态 `exitStatus` 
5. 根据渲染任务的终止状态判断是否可以调用 `commitRoot` 进入 Commit 阶段
6. 调用 `ensureRootIsScheduled` 再次调度
7. 确定该函数返回值，从而告诉 Scheduler 怎样继续调度

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
  // 1. 执行 PassiveEffects，因为他可能会产生新的更新
  const didFlushPassiveEffect = flushPassiveEffects()
  
  if (didFlushPassiveEffect) {
    // PassiveEffect 中的某些高优先级的任务可能中断了本次任务，先检查一下本次任务有没有被取消掉
    if (root.callbackNode !== originalCallbackNode) {
      return null
    }
  }
  
  // 2. 获取本次更新要执行的优先级，可以暂时理解为 getHighestPriorityLane(root.pendingLanes)
  const lanes = getNextLanes(
    root,
    root === workInProgressRoot ? workInProgressRootRenderLanes: NoLanes,
  )
  if (lanes === NoLane) { return null }
  
  // 3. 需要同步执行的 Lane、过期的 Lane、或者该任务由于 Scheduler 传给的参数导致该渲染需要同步进行
  // 在开启了默认使用同步更新（Concurrent）的情况下，includesBlockingLane 将永远返回 false
  // 否则 DefaultLane 会认为是 BlockingLane，首次渲染时会返回 true
  const shouldTimeSlice =
    !includesBlockingLane(root, lanes) &&
    !includesExpiredLane(root, lanes) &&
    !didTimeout
  
  // 4. 开始执行 render 的具体任务
  let exitStatus = shouldTimeSlice
    ? renderRootConcurrent(root lanes)
    : renderRootSync(root, lanes)
  
  // ... 判断 exitStatus
  
  if (exitStatus === RootCompleted) {
    // 5. 进入 Commit 阶段（省略 commitRootWhenReady 中间过程）
    root.finishedWork: Fiber = root.current.alternate
    root.finishedLanes: Lanes = lanes
    commitRoot(root)
  }
  
  // 6. 再次调度，直至没有任务需要调度
  ensureRootIsScheduled(root)
  // 7. 按照 Scheduler 的约定，该函数的返回值将会被 Scheduler 继续调度
  // 因此通过 getContinuationForRoot 看需要继续调度什么函数
  return getContinuationForRoot(root, originalCallbackNode)
}
```

### 渲染

#### 同步渲染

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

`workLoop` 会循环地将当前的 `workInProgress` `Fiber` 送入 `performUnitOfWork` 函数来执行具体的渲染工作，与 `performXXXWorkOnRoot` 和 `renderXXXRoot` 一样，他也有同步和并发两种版本。

两种版本唯一的区别在于同步版本的 `workSyncConcurrent` 会一直循环执行直到 `workInProgress` 为空，而并发版本则会受到来自 Scheduler 的 `shouldYield` 函数控制，使得他可以提前中断。

Render 阶段的同步和并发的区别到此就结束了，他们后面的流程完全一样。

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

不管是同步还是并发渲染，他们在 `workLoop` 中都将调用 `performUnitOfWork` 来执行具体的每一个 `Fiber` 上的任务。

渲染的过程实际上就是遍历 FiberTree 的过程，React 采用深度优先（DFS）的方式对 FiberTree 进行遍历

该函数接受一个 `Fiber` 作为工作单元 `unitOfWork`，事实上他每次收到的参数就是全局的 `workInProgress` 变量存的 `Fiber`。每次

