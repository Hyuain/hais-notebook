---
title: React - Deep Dive
date: 2023-06-23 16:41:41
categories:
  - [前端]
---

与另一篇 React 笔记不同，本文主要是深入讨论 React 的源码与设计理念。

<!-- more -->

Clone React 项目之后，在 README 中找到调试 React 的方法，具体可参照 [How to Contribute](https://legacy.reactjs.org/docs/how-to-contribute.html)。然后按照文中安装依赖并 Build 之后，将 `fixture/packaging/babel-standalone/dev.html` 作为我们的 Playground。

React 从状态更新到 UI 变化中间经历的过程，被称为 **Reconciliation**，完成这个过程的工具就是 **Reconciler**，他的核心代码都在 `react-reconciler` 这个包中。

# 首次渲染过程

React 的更新流程分为 **Schedule、Render 和 Commit** 三个阶段：

- **Schedule 阶段** 会挑出一批优先级，并为不同优先级准备不同的调度策略；
- **Render 阶段** 会执行 Fiber 节点的重新计算，并标记好需要执行的副作用；
- **Commit 阶段** 分为 **BeforeMutation**、**Mutation** 和 **Layout** 三个子阶段，在 **Mutation 阶段** 会完成 UI 相关的副作用，并且会完成 `workInProgress` 与 `current` `FiberTree` 的切换。 

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

```text
ReactDOM.createRoot             // 创建 ReactDOMRoot
  createContainer               // 创建 Container，Container 实际上就是创建 FiberRoot
    createFiberRoot             // 创建 FiberRoot
      createHostRootFiber       // 创建 HostRootFiber（tag 为 HostRoot 的 Fiber）
        createFiber             // 创建 Fiber
  listenToAllSupportedEvents    // 在容器 DOM 元素上绑定事件
React.createElement             // 暴露给外部，将 JSX 转换的结果再转换为 ReactElement

ReactDOMRoot.render             // 传入 ReactElement，开始首次渲染
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
2. 为该 `FiberRoot` 创建了一个 `HostRootFiber`（一个 `tag` 为 `HostRoot` 的 `Fiber`）。该 `HostRootFiber` 即为后续 `FiberTree` 的根节点
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
  // 获取优先级（lane），后面会提到
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

1. 给当前的 `root` 标记选出的 `lane`
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

## 数据结构

要完成上文的过程，已经需要不少的数据结构进行支撑了，包括 `ReactDOMRoot`  `Fiber `  `FiberRoot`  `Update`  `UpdateQueue`  `Lane` 等，了解这些数据结构对了解 React Reconcile 的

# Schedule

