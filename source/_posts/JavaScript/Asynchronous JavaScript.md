---
title: Asynchronous JavaScript
date: 2020-01-29 17:22:52
categories:
  - - 前端
tags:
  - FX
---

JavaScript 中的异步操作主要发生在使用 `XMLHttpRequest` 等与外界交互时，以及内部的定时器 `setTimeout` 时。此时需要运用轮询或回调的方式，取得异步操作的结果。

其中 [[Promise]] 在异步编程中尤为重要。

<!-- more -->

# AJAX

> AJAX 即 Asynchronous JavaScript and XML（异步的 JavaScript 与 XML 技术），其实就是一套综合了多项技术的浏览器端网页开发技术。Google 在它多个著名的交互应用程序中使用了这套技术，如 Google 讨论组、Google 地图、Google 搜索建议、Gmail 等，这使得人们看到了前端领域新的可能性。
>
> 传统的 Web 应用允许用户端填写表单（form），当提交表单时就向网页服务器发送一个请求；服务器接收并处理传来的表单，然后送回一个**新的网页**。
>
> 但这个做法浪费了许多带宽，因为在前后两个页面中的大部分 [[HTML]] 往往是相同的。由于每次应用的沟通都需要向服务器发送请求，应用的回应时间依赖于服务器的回应时间。这导致了用户界面的回应比本机应用慢得多。
>
> 与此不同，AJAX 应用可以仅向服务器发送并取回必须的数据，并在客户端采用 JavaScript 处理来自服务器的回应。因为在服务器和浏览器之间交换的数据大量减少，服务器回应更快了。同时，很多的处理工作可以在发出请求的客户端机器上完成，因此 Web 服务器的负荷也减少了。

简单来讲：

1. AJAX 是浏览器上的功能，浏览器可以使用 AJAX 发请求、收响应；
2. 浏览器在 `window` 上面加了一个 `XMLHttpRequest`，方便开发者通过 JS 来发请求、收响应，这是一个构造函数

## 加载 CSS / JS / HTML / XML

如果想要使用 `XMLHttpRequest` 让浏览器来帮我们加载 CSS / JS / HTML / XML，一共需要进行四步：

1. 创建 `XMLHttpResquest` 对象
2. 调用他的 `open()` 方法
3. 使用 `onreadystatechange` 监听他的成功和失败事件
4. 调用他的 `send()` 方法

```js
const request = new XMLHttpRequest()
request.open('GET', '/{url}')
request.onreadystatechange = () => {
  if (request.readyState == 4) {
    if (request.status >= 200 && request.status < 300) {
      alert('success')
    } else {
      alert('error')
    }
  }     
}
request.send()
```

HTTP 里面可以装 HTML / CSS / JS / XML 等，但得知道怎么解析他们：

- 拿到 CSS 之后生成 `<style>` 标签
- 拿到 JS 之后生成 `<script>` 标签
- 拿到 HTML 之后使用 `innerHTML` 和 DOM API，可以新建一个 `div`
- 拿到 XML 之后的 `request.responseXML` 实际上已经是一个 DOM 节点了

此外我们还可以用 AJAX 来请求 [[JSON]] 格式的数据，经由 JavaScript 解析之后使用。

# 异步与同步

- **同步**： **能直接拿到结果，不拿到结果就不离开**，就像医院挂号一样，可能需要花上几分钟，但是 **拿到号之后才会离开窗口**
- **异步**： **不能直接拿到结果**，比如在餐厅门口等位，拿到号之后我们可以去逛街，那什么时候真正吃上饭呢？我们可以：
  - 每 10 分钟去餐厅问一下（**轮询，Polling**）
  - 也可以扫码用微信接收通知（**回调，Callback**）

AJAX 举例：当 `request.send()` 之后，并不能直接得到 `response`，必须要等待 `readyState` 变成 `4` 之后，浏览器回头调用 `request.onreadystatechange` 函数

# 异步与回调
## 回调

- 写给自己的函数，不是 **回调**
- 写给别人用的函数，放在一个地方，让他来调用，就是回调
- `request.onreadystatechange`，是写给浏览器调用的，意思是让浏览器回头调一下这个函数

举例：

```js
function f1(){}
function f2(fn){
  fn()
}
f2(f1)
// 我调用了 f2，因此 f2 不是回调
// 我把 f1 传给了 f2
// 我没有调用，而是 f2 调用了 f1
// 所以 f1 是回调
```

## 关联

异步任务需要在得到结果时通知 JS 来拿结果，怎么通知呢？

1. 可以让 JS 留一个函数地址（电话号码）给浏览器
2. 异步任务完成之后，浏览器调用该函数的地址（拨打电话）
3. 同时 **把结果作为参数** 传给该函数（电话里说可以来吃了）

这个函数是我写给浏览器调用的，所以是回调函数

## 区别

异步任务需要用回调函数来通知结果

- 但异步也可以用 **轮询**
- 回调也不一定用在异步里面，比如 `array.forEach( n ⇒ console.log(n) )`，这就是 **同步回调**

# 异步函数

[MDN Async Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/async_function)

如果一个函数的返回值处于以下这三个内部，即为异步函数：

- `setTimeout`
- AJAX（即 `XMLHttpRequest`）
- `AddEventListener`

也可以设置同步的 AJAX，`request.open("get", "/5.json", false)`，但是这样会使页面在请求期间卡住。[[Promise]] 是 JavaScript 给出的一种处理异步函数的方案。

举例：

```js
function 摇骰子(){
  setTimeout( () => {
    return parseInt(Math.random() * 6) + 1
  }, 1000)
  // return undefined
}

// 如果是按上面这样写，将拿不到 1 ~ 6 的随机数，而是 undefined
const n = 摇骰子()
console.log(n) // undefined

// 通过回调来拿到异步的结果
function 摇骰子(fn){
  setTimeout( () => {
    fn(parseInt(Math.random() * 6) + 1)
  }, 1000)
}
function f1(x){ console.log(x) }
摇骰子(f1)

// 可以简化代码，因为 f1 只用了一次
function 摇骰子(fn){
  setTimeout( () => {
    fn(parseInt(Math.random() * 6) +1)
  }, 1000)
}
摇骰子(x => {
  console.log(x)
})

// 可以再简化为
摇骰子(console.log) // 但是如果参数个数不一致就不能这样简化
```

可以再看一下下面这道题：

```js
const array = ['1', '2', '3'].map(parseInt)
  console.log(array) // [1, NaN, NaN]
    
  // 正确的写法，i 和 arr 可以省略
  const array = ['1', '2', '3'].map((item, i, arr) => {
    return paseInt(item)
  })
    
  // 最开始的写法相当于
  const array = ['1', '2', '3'].map((item, i, arr) => {
    return paseInt(item, i, arr)
    // parseInt('1', 0, arr)，0 为无效参数，忽略
    // parseInt('2', 1, arr)，相当于把 '2' 作为 1 进制来进行解析 => NaN
    // parseInt('3', 2, arr)，相当于把 '3' 作为 2 进制来进行解析 => NaN
  })
```


