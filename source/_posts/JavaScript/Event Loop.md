---
title: Event Loop
date: 2020-01-29 17:22:52
categories:
  - - 前端
tags:
  - FX
---

浏览器的 Event Loop 有宏任务和微任务两个阶段，而 NodeJS 中的则更加复杂一些，包括 timer、poll 和 check 等阶段。

<!-- more -->

# 浏览器中的 Event Loop

浏览器中的 Event Loop 有 2 个阶段：**宏任务** 和 **微任务**

1.  **第一个宏任务**：整段脚本。执行过程中的 **同步代码** 直接执行，**宏任务** 进入宏任务队列，**微任务** 进入微任务队列
2. 当前 **宏任务** 执行完出队，检查 **微任务** 队列，若有则依次执行，直至微任务队列为空
3. 微任务执行完成之后，执行浏览器 **UI 线程** 的渲染工作
4. 检查是否有 **Web worker** 任务，有则执行
5. 执行队首新的 **宏任务**，回到 2 循环至宏任务队列为空

宏任务有：

- 用户交互事件（点击、滚动、鼠标移动等）
- 网络请求完成
- 计时器
- 页面生命周期
- 浏览器自身的事件（浏览器窗口大小变化、浏览器关闭事件）
- JavaScript 代码执行（如果任务是通过函数调用产生的，那么函数本身也会作为宏任务）

微任务：

- [[Promise]]

注意：

1. 任务执行过程中，**浏览器不会渲染**（如果执行时间太长，有的浏览器会提示说“页面无响应”）
2. 微任务会在其他所有的宏任务执行之前执行，包括用户事件。这保证了浏览器环境在各个微任务执行的时候保持一致（鼠标指针不会发生变化、没有新的网络数据）
3. 可以通过 `queueMicrotask` 手动将一些任务移动到微任务队列中。

## Examples

### 下列代码的打印顺序

```js
async function async1() {
  console.log('async1 start')  // 2
  await async2() // -> 执行 async2 并等待返回结果
  console.log('async1 end') // 6 -> 有的浏览器会先执行 then 的，而不是按入栈出栈的先后顺序
}
async function async2() {
  console.log('async2')  // 3
}
console.log('script start')  // 1
setTimeout(function() {
  console.log('setTimeout')  // 8
})
async1()
new Promise(function(resolve) {
  console.log('promise1') // 4
  resolve()
}).then(function() {
  console.log('promise2') // 7
})
console.log('script end') // 5
```

### 拆分复杂的计算任务

比如语法高亮就是 CPU 密集型的复杂计算，需要分析代码、创建高亮元素、加进 document。下面用一个遍历来模拟这个计算过程：

```js
let i = 0;
let start = Date.now();
function count() {
  // do a heavy job
  for (let j = 0; j < 1e9; j++) {
    i++;
  }
  alert("Done in " + (Date.now() - start) + 'ms');
}
count();
```

可以把他用 `setTimeout` 拆分到下次执行，让别的任务（比如 `onclick` 有时间进行）：

```js
let i = 0;
let start = Date.now();
function count() {
  // do a piece of the heavy job (*)
  do {
    i++;
  } while (i % 1e6 != 0);
  if (i == 1e9) {
    alert("Done in " + (Date.now() - start) + 'ms');
  } else {
    setTimeout(count); // schedule the new call (**)
  }
}
count();
```

###  进度条

如果我们按照下面这样写代码，是看不到进度条变化的，因为他会在所有的代码执行完成之后才会更新 DOM：

```html
<div id="progress"></div>
<script>
  function count() {
    for (let i = 0; i < 1e6; i++) {
      i++;
      progress.innerHTML = i;
    }
  }
  count();
</script>
```

得这样写才行：

```html
<div id="progress"></div>
<script>
  let i = 0;
  function count() {
    // do a piece of the heavy job (*)
    do {
      i++;
      progress.innerHTML = i;
    } while (i % 1e3 != 0);
    if (i < 1e7) {
      setTimeout(count);
    }
  }
  count();
</script>
```

### 延迟用户事件到冒泡完成之后

可以通过 `setTimout` 延迟发送一个自定义事件，这样他就会在本轮事件冒泡结束之后进行了。

```js
menu.onclick = function() {
  // ...
  // create a custom event with the clicked menu item data
  let customEvent = new CustomEvent("menu-open", {
    bubbles: true
  });
  // dispatch the custom event asynchronously
  setTimeout(() => menu.dispatchEvent(customEvent));
};
```


# Node.js 中的 Event Loop

```
   ┌───────────────────────────┐
┌─>│           timers          │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │
│  └─────────────┬─────────────┘      ┌───────────────┐
│  ┌─────────────┴─────────────┐      │   incoming:   │
│  │           poll            │<─────┤  connections, │
│  └─────────────┬─────────────┘      │   data, etc.  │
│  ┌─────────────┴─────────────┐      └───────────────┘
│  │           check           │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │
   └───────────────────────────┘
```

## 三大关键阶段

1. **timer**：执行定时器回调阶段，检查定时器（`setTimeout` `setInterval`），如果到了时间就执行回调。
2. **poll**：轮询阶段，当文件 I/O、网络 I/O 等一步操作执行完之后，通过 `data` `connect` 等事件来通知 JS 主线程，使得时间循环到达 poll 阶段：
   - 如果当前已经存在定时器，且定时器到时间了，拿出来执行，Event Loop 回到 timer 阶段
   - 如果没有定时器，就去看回调函数队列
     - 如果队列不为空，拿出队列中的方法依次执行
     - 如果队列为空，检查是否有 `setImmediate`
       - 有则前往 check 阶段
       - 没有则继续等待，等待 callback 加入队列，加入后会立即执行；一定时间后自动进入 check 阶段
3. **check**：直接执行 `setImmediate`

   - `setTimeout` -> timers
   - `setImmediate` -> check
   - `nextTick` -> 当前阶段的后面
   - `promise.then` -> 看他是通过什么实现的，如果是用 `nextTick` 实现的，就当 `nextTick` 看，当 `resolve` 的时候放在当前阶段的后面

## Examples

注意：有可能先执行 JS 代码，也有可能先开启 Event Loop，因为进程开启都需要时间。

比如当执行：

```js
setTimeout(fn, 100)
setImmediate(fn2)
```

**`fn` 放到一个 timers 的一个数组中**

V

**进入 poll 阶段进行等待，同时看时间**

V

**进入 check 阶段**
允许在 poll 阶段空闲的时候立即执行一些函数，主要是 `setImmediate()` 里面的，
比如如果有 `setImmediate(fn2)`，则在 poll 中就不等了，直接进入 check，在 check 中执行 `fn2`

V

**进入 timers 阶段，执行 `fn`**
如果因为之前有 `setImmediate()` 导致提前进入了下一个循环，而此时 `setTimeout` 的计时还没到，则此时不执行 `fn`

而有种情况比较特殊：

```js
setTimeout(fn, 0)
setImmediate(fn2)
```

注意：

- 如果 Event Loop 开启得很快，则此时 timers 中没有 `fn`；
- 如果 Event Loop 开启得慢，则此时 timers 中有 `fn`，而如果此时 timers 中有 `fn`，而且计时已经结束，就马上执行了 `fn` 再往下走

也就是说因为 Event Loop 有时候开启得快，有时候开启得慢，所以如果一开始就执行上面两句代码，他们的执行先后是不确定的！

