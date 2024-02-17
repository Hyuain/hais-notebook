---
title: React Reconciliation
date: 2023-06-23 16:41:41
categories:
  - - 前端
tags:
  - FX
---

在 Render 阶段，React 会使用 DFS 构建 `workInProgress` [[React Fiber|FiberTree]]，**整个 Render 的过程可以分为两部分**：

| beginWork                                                    | completeWork                                                 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 前序执行<br />在遍历子树之前执行<br />向下遍历的过程中执行<br />“递” 的阶段执行 | 后序执行<br />在遍历子树之后执行<br />向下遍历的过程中执行<br />“归” 的阶段执行 |
| 根据当前 `Fiber` 创建 `child`<br />标记 `Placement` `ChildDeletion` | 构建 DOMTree<br />`flags` 和 `lanes` 冒泡                    |

> 关于 DFS 的详细说明请看 [[Binary Tree#Deep First Traversal|二叉树的深度优先遍历]] 和 [[Backtracing|回溯算法]]，以及下面的 **遍历 FiberTree 执行任务** 节

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

# 渲染流程控制

渲染流程由 ``performSyncWorkOnRoot`` 或 `performConcurrentWorkOnRoot` 控制，这两个函数有些许不同，但大体流程是基本一致的：

1. **获取优先级：** 获取本次渲染要处理的 `lanes`
2. **Render：** 进行 Sync 或 Concurrent 渲染
3. **[[React Committing|Commit]]：** 根据渲染结果进入 Commit 阶段
4. **[[React Scheduling|Schedule]]：** 调用 `eunsureRootIsScheduled` 再次调度

## Sync 渲染流程控制

如果该渲染的优先级是 `SyncLane` 等 Sync 优先级，将会进入 `performSyncWorkOnRoot` 开启的更新流程

1. **获取优先级：** 通过 `getNextLanes` 获取本次更新要执行的优先级
2. **Render：** 调用 `renderRootSync` 开始 Sync 渲染
3. **Commit：** 根据渲染任务的终止状态判断是否可以调用 `commitRoot` 进入 Commit 阶段
4. **Schedule：** 调用 `ensureRootIsScheduled` 再次调度

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

## Concurrent 渲染流程控制

首次渲染过程的优先级是 `DefaultLane`，将进入 `performConcurrentWorkOnRoot` 函数，他会：

1. **获取优先级：** 通过 `getNextLanes` 获取本次更新要执行的优先级
2. **Render：**
   1. 通过优先级和 `didTimeout` 得到 `shouldTimeSlice`，判断使用 Sync 渲染 `renderRootSync` 还是 Concurrent 渲染 `renderRootConcurrent`
   2. 调用 `renderRootSync` 或 `renderRootConcurrent` 进行渲染，并获得渲染任务的终止状态 `exitStatus` 
3. **Commit：** 根据渲染任务的终止状态判断是否可以调用 `commitRoot` 进入 Commit 阶段
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

# 渲染

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

## 初始化渲染所需变量

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

## workLoop

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

# 遍历 FiberTree 执行任务

不管是 Sync 还是 Concurrent 渲染，他们在 `workLoop` 中都将调用 `performUnitOfWork` 来执行具体的每一个 `Fiber` 上的任务：

1. 渲染的过程实际上就是遍历 FiberTree 的过程，React 采用深度优先（DFS）的方式对 FiberTree 进行遍历
2. `performUnitOfWork` 接受一个 `Fiber` 作为工作单元 `unitOfWork`
3. `unitOfWork` 参数实际上就是全局的 `workInProgress`，每次 `performUnitOfWork` 中都将更新这个变量，以便 `workLoop` 的下次循环中调用时使用新的 `workInProgress`
4. 每次 `performUnitOfWork` 的执行过程是无法中断的，Concurrent 任务中断的时机是在 `workLoop` 中每次对下一个 `workInProgress` 进行处理的时候

## 遍历流程

React 使用 DFS 中来遍历 FiberTree。渲染过程实际上分为 **“递”** 和 **“归”** 两个部分：

- **在 “递” 的阶段，会执行 `beginWork` 方法，并沿着 `.child` 单链表向下遍历。** `beginWork` 方法中，会创建新 `Fiber` 或对旧的 `Fiber` 进行复用及更新。
- **在 “归” 的阶段，会执行 `completeWork` 方法，并先沿着 `.siblings` 遍历所有兄弟节点。如果有兄弟节点，将会沿着兄弟节点的 `.child` 向下遍历，即进入 “递” 阶段；如果没有兄弟节点，则会沿着 `.return` 向上遍历。** `completeWork` 方法中会根据标记创建或修改离屏 DOM。

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

## 简化版代码

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

# beginWork

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

1. **计算新的子节点：** 处理 `UpdateQueue`、执行函数组件本体、执行类组件的渲染函数等，计算得到新的内部状态 `memorizedState` 和新的子节点
2. **创建子节点并增加标记：** 使用 `reconcileChildren` 来处理新的子节点，为子节点或父节点 `flags` 添加标记，使得更新在 `commit` 阶段能得到合适的处理并应用到最终的宿主环境 UI 中

`flags` 与 `lanes` 类似，也是用二进制位来表示的，因此可以用按位或来打上标记，比如为节点打上 ”新增节点“ 标记 `Placement` ，可以使用 `Fiber.flags |= Placement`。

## 不同类型的 beginWork

### HostRoot

`HostRoot` 会进入 `updateHostRoot` 处理：

1. **计算新的子节点：** 调用 `processUpdateQueue` 计算更新，得到新的 `element`
2. **创建子节点并增加标记：** 调用 `reconcileChildren` 并传入 `element` 更新子节点

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

### IndeterminateComponent

函数组件对应的 `Fiber` 在创建时不会被归纳为 `FunctionComponent`，而是会被归纳为 `IndeterminateComponent`（未被确定类型的组件），并在 `begeinWork` 中进入 `mountIndeterminateComponent` 函数进行处理：

1. **计算新的子节点：** 调用 `renderWithHooks` 调用函数组件本体进行更新并得到返回值 `value`
2. **创建子节点并增加标记：** 调用 `reconcileChildren` 并传入 `value` 更新子节点

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

### HostComponent

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

## reconcileChildren

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

### CreateChildReconciler

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

### ChildReconciliation

#### reconcileSingleElement

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

### TrackSideEffects

#### placeSingleChild

`placeSingleChild` 用来给 `Fiber` 加上 `Placement` 标记：

```typescript
function placeSingleChild(newFiber: Fiber): Fiber {
  if (shouldTrackSideEffects && newFiber.alternate === null) {
    return newFiber
  }
}
```

#### deleteChild

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


#### deleteRemainingChildren

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

# completeWork

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

## 构造离屏 DOM

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

## Lanes 和 Flags 冒泡

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
