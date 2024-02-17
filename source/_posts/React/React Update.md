---
title: React Update
date: 2023-06-23 16:41:41
categories:
  - - 前端
tags:
  - FX
---

# Update

`Update` 是 [[React Fiber]] 用来承载 *改变 State 的更新任务* 的数据类型，React 中通过 `createUpdate` 来创建 `Update`。

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

# UpdateQueue

`UpdateQueue` 是 [[React Fiber]] 用来存放和消费 `Update` 的数据类型，`initializeUpdateQueue` 负责初始化 `Fiber` 上的 `updateQueue`，`enqueueUpdate` 用于将 `Update` 加入 `UpdateQueue` 中，`processUpdateQueue` 则用于消费 `Update`。

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

## enqueueUpdate

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

## processUpdateQueue

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
