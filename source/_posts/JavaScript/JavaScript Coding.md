---
title: JavaScript Coding
date: 2021-03-23 16:33:44
categories:
  - [前端]
---

JavaScript 的一些 Coding 问题。

<!-- more -->

## Promise

## 使用 Promise 控制请求并发数

```typescript
const createRequestPool = (limit: number) => {
  const queue = []
  const runningTasks = new Set()
  
  const run = () => {
    if (runningTasks.size >= limit) { return }
    const wantage = limit - runningTasks.size
    queue.splice(0, wantage).forEach((task) => {
      runningTasks.add(task)
      task().finally(() => {
        runningTasks.delete(task)
        run()
      })
    })
  }
  
  return function(fn) {
    return new Promise((resolve, reject) => {
      queue.push(() => {
        fn().then((res) => {
          resolve(res)
          return res
        }).catch(reject)
      })
      run()
    })
  }
}
```

可以自己写一些假的请求尝试一下：

```typescript
const request = (x) => {
  return new Promise((resolve) => {
   	setTimeout(() => {
      resolve(x)
    }, Math.random() * 3000)
  })
}
```

然后这样使用：

```js
const limitRequest = createRequestPool(3)
limitRequest(request(1)).then(console.log)
limitRequest(request(2)).then(console.log)
limitRequest(request(3)).then(console.log)
```

