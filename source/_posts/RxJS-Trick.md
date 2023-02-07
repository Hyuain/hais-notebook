---
title: RxJS 小技巧
date: 2021-05-14 18:51:12
categories:
  - [前端]
---

利用 share 防止重复请求。

<!-- more -->

# 防止重复请求

参考了 [这篇文章](https://stackoverflow.com/questions/50864978/angular-rxjs-6-how-to-prevent-duplicate-http-requests)

```typescript
// request.ts
let dataObservable: Observable<any>

const request = (): Observerble<any> => {
  if (dataObservable) { return dataObservable }
  dataObservable = new Observable<any>((subscriber) => {
    console.log("mock Run API")
    setTimeout(() => {
      console.log("mock API Callback")
      subscriber.next(Math.random())
      subscriber.complete()
      // 可以根据需求自行确定这个 observable 销毁的时机
      dataObservable = undefined
    }, 500)
  }).pipe(share())
  return dataObservable
}
```
```typescript
// somewhere.ts
request().subscribe((res) => {
  console.log(1, res)
})
request().subscribe((res) => {
  console.log(2, res)
})
setTimeout(() => {
  request().subscribe((res) => {
    console.log(3, res)
  })
})

// Logs:
// mock Run API
// mock API Callback
// 1 0.403....
// 2 0.403....       跟 1 打出的值一样
// mock Run API      上面的 500 ms 之后
// mock API Callback
// 3 0.838....       新的值
```
