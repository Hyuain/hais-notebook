---
title: Promise
date: 2020-01-29 17:22:52
categories:
  - - 前端
tags:
  - FX
---

用 Promise 可以较为优雅地进行 [[Asynchronous JavaScript|异步编程]]，解决回调地狱的问题。

也可以尝试[[JavaScript Polyfill#Promise|实现一个 Promise]]。

<!-- more -->

# 基本用法

```js
ajax = (method, url, options) => {
  const {success, fail} = options
  const request = new XMLHttpRequest()
  request.open(method, url)
  request.onreadystatechange = () => {
    if(request.readyState === 4) {
      if(request.status < 400) {
       success.call(null, request.response)
      } else if(request.status >= 400) {
        fail.call(null, request, request.status)
      }
    }
  }
  request.send()
}

ajax('get', '/xxx', {
  success(response){}, fail:(request, status) => {}
})  // 左边是 function 的缩写，右边是箭头函数
```

↑ 上面是普通的写法，↓ 下面是 Promise 的写法

```js
ajax = (method, url) => {
  return new Promise( (resolve, reject) => {  // return new Promise((resolve, reject)=>{..})
    const request = new XMLHttpRequest()
    request.open(method, url)
    request.onreadystatechange = () => {
      if(request.readyState === 4) {
        if(request.status < 400) {
          resolve.call(null, request.response)  // 成功调用 resolve(result)，他再回调用第一个函数
        } else if(request.status >= 400) {
          reject.call(null, request)  // 失败调用 reject(error)，他再会调用第二个函数
        }
      }
    }
    request.send()
  })
}
    
ajax('get', '/xxx')
  .then( (response) => {}, (request) => {} )   // Promise 的回调（成功/失败）只能接受一个参数
```

# Promise 是如何解决回调地狱的

回调地狱：

1. 多层嵌套
2. 无法方便地进行错误处理

解决办法：

1. 回调函数延迟绑定，回调函数是通过后面的 then 方法传入的
2. 返回值穿透，根据 then 中回调函数的传入值创建不同类型的 Promise，再把返回的 Promise 穿透到外层，以供后续使用。以上两点实现了链式调用，解决多层嵌套问题
3. 错误冒泡，前面产生的错误会一直向后传递，被 catch 接收到，就不用频繁地检查错误了


# Promise.all()

- 如果传入的参数是一个空的可迭代对象，比如空数组，则返回一个已完成（already resolved）状态的 Promise。
- 如果传入的参数不包含任何 promise，则返回一个异步完成（asynchronously resolved） Promise。
- 其它情况下返回一个处理中（pending）的Promise。

返回的 promise 之后会在所有的 promise 都完成或有一个 promise 失败时异步地变为完成或失败。（如果都成功，则成功；有一个失败，则失败）

返回值将会按照参数内的 promise 顺序排列，而不是由调用 promise 的完成顺序决定。

```js
var p1 = Promise.resolve(3);
var p2 = 1337;
var p3 = new Promise((resolve, reject) => {
  setTimeout(resolve, 100, 'foo');
}); 

Promise.all([p1, p2, p3]).then(values => { 
  console.log(values); // [3, 1337, "foo"] 
});
```

# Promise.race()

可以传入多个 promise，谁快，Promise.race() 就会跟谁一样，如果传的迭代是空的，则返回的 promise 将永远等待。

# Examples

```js
Promise.reject('error')
  .then( ()=>{console.log('success1')}, ()=>{console.log('error1')} )
  .then( ()=>{console.log('success2')}, ()=>{console.log('error2')} )
// error1 -> success2
```
