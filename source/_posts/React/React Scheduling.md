---
title: React - Deep Dive
date: 2023-06-23 16:41:41
categories:
  - - 前端
tags:
  - FX
---

React 的任务调度分为 **宏任务调度** 和 **微任务调度** 两种，React 会根据任务的优先级采取不同的调度策略，这个选择调度策略以及进行任务调度的阶段，即为 Schedule 阶段。

Schedule 阶段主要用到 `react-reconciler` 和 `scheduler` 。

`react-reconciler` 主要负责在 React 的工作流中进行任务调度，而涉及到具体的调度的具体实现（比如任务队列的维护，具体使用什么 API 让宿主环境在什么时候执行什么优先级的任务等）则会根据是 [[Event Loop|宏任务]] 还是 [[Event Loop|微任务]] 而有所不同：

React 中所有的更新工作都会被调度，或是在微任务中执行、或是在宏任务重执行，**因此 React 中并没有严格意义上的同步更新**，这也是 `setState` 并不会同步生效的原因。

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

# Scheduler

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

`timerQueue` 和 `taskQueue` 均采用了 [[Heap#Priority Queue & Heap|优先级队列]] 这一数据结构，优先级队列中排序的依据就是过期时间 `expirtionTime`。

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

# React 中的优先级

React 中有两种优先级表示方法，一种是 `Lane`，一种是事件优先级 `EventPriority`，`EventPriority` 可以看做是特殊的 `Lane`。

具体来讲，不同的事件会产生不同的 `EventPriority`，比如 `DiscreteEventPriority` 代表 `click` `input` `focus` 等离散事件； `ContinousEventPriority` 代表 `drag` `mouseover` 等连续事件。这些 `EventPriority` 都有着对应的 `Lane`，他们数值相同，不需要函数来转换。

但是 `Lanes` 转换为 `EventPriority` 时，由于是多对一转换，就需要 `lanesToEventPriority` 来帮忙。

**不过，上面提到的 `Scheduler` 还有一套自己的优先级系统，React 需要再将优先级转换为 `SchedulerPriority`，才能调用 `Scheduler` 的 API。** 其实也很简单，用 `switch-case` 映射一下就好了。

# 调度流程

## 计算并处理调度

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

## 在微任务中选择优先级并进行调度

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
