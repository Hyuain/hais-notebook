---
title: React Committing
date: 2023-06-23 16:41:41
categories:
  - - 前端
tags:
  - FX
---

Commit 阶段会将刚刚渲染中标记的各种副作用 `flags` 提交到 UI 上。与 [[React Rendering|Render]] 阶段不同，Commit 阶段无法暂停或终止，他会在一次同步的代码中执行完成对整个 `FiberRoot` 的 Commit。

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

# 子阶段执行流程

三个子阶段均按照 [[Binary Tree#Deep First Traversal|DFS]] 对节点进行处理，且大部分操作都是在 **自下而上的后序过程中** 处理的。在目前 React 的源码中，`commitBeforeMutationEffects` 是采用的迭代写法，而 `commitMutationEffects` 和 `commitLayoutEffects` 中均采用递归写法，他们会先找到最下层的有该副作用的节点（该节点的 `subtreeFlags` 不再包含需要处理的副作用），然后再自下而上地处理（也有部分副作用是自上而下的前序过程中处理的）。

## 迭代写法

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

## 递归写法

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

# BeforeMutation

BeforeMutation 阶段主要有两个作用：

- 执行 `ClassComponenent` 的 `getSnapshotBeforeUpdate` 方法
- 清空 `HostRoot` 挂载的内容

# Mutation

对于 HostComponent 来说，Mutation 阶段会进行 DOM 元素的增、删、改：

- **删除元素：** 在向下遍历的过程中会前序执行删除操作，该操作会将 Render 阶段 `beginWOrk` 中往 `fiber.deletions` 里添加的元素删除掉，删除会有以下过程
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

# Layout

在 DFS 中会前序执行 `OffscreenComponent` 的显隐逻辑，后序执行 `useLayoutEffect` 回调。
