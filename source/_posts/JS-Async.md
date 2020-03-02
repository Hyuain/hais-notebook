---
title: JavaScript 异步
date: 2020-01-30 08:49:54
tags:
  - 入门
  - 饥人谷
categories:
  - [前端, JavaScript, 原生 JavaScript]
---

介绍 AJAX 与异步等。

<!-- more -->

# AJAX

> AJAX 即 Asynchronous JavaScript and XML（异步的 JavaScript 与 XML 技术），其实就是一套综合了多项技术的浏览器端网页开发技术。Google 在它多个著名的交互应用程序中使用了这套技术，如 Google 讨论组、Google 地图、Google 搜索建议、Gmail 等，这使得人们看到了前端领域新的可能性。
> 
> 传统的 Web 应用允许用户端填写表单（form），当提交表单时就向网页服务器发送一个请求；服务器接收并处理传来的表单，然后送回一个**新的网页**。
> 
> 但这个做法浪费了许多带宽，因为在前后两个页面中的大部分 HTML 码往往是相同的。由于每次应用的沟通都需要向服务器发送请求，应用的回应时间依赖于服务器的回应时间。这导致了用户界面的回应比本机应用慢得多。
> 
> 与此不同，AJAX 应用可以仅向服务器发送并取回必须的数据，并在客户端采用 JavaScript 处理来自服务器的回应。因为在服务器和浏览器之间交换的数据大量减少，服务器回应更快了。同时，很多的处理工作可以在发出请求的客户端机器上完成，因此Web服务器的负荷也减少了。

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

## JSON

> JavaScript Object Natation，JSON 不是对象，而是一种标记语言（就像 HTML、XML、Markdown），用来展示数据，[JSON 中文官网](http://json.org/json-zh.html)

### JSON 的数据类型

- string（但是只支持双引号）
- number（支持科学计数法）
- bool
- null
- object
- array

{% note warning %}
JSON 中，不支持函数和变量
{% endnote %}

### JSON.parse()

> 反序列化

- 将符合 JSON 语法的字符串转换为 JS 对应类型的数据
- JSON 字符串 ⇒ JS 数据
- 由于 JSON 只有六种类型，所以转换出的数据也只有六种
- 如果不符合 JSON 语法，则会抛出 Error，一般用 try catch 捕获错误，比如

```js
let object
try {
	object = JSON.parse(`{'name':'harvey'}`)
} catch (error) {
	console.log('出错了，错误详情是：')
	console.log(error)
	object = {'name': 'no name'}
}
console.log(object)
```

### JSON.stringify()

> 序列化

- 是 JSON.parse 的逆运算
- JS 数据 ⇒ JSON 字符串
- 由于 JS 的数据类型比 JSON 多，所以不一定能够成功
- 如果失败，则会抛出一个 Error 对象

# 异步

## 异步与同步

- **同步**： **能直接拿到结果，不拿到结果就不离开**，就像医院挂号一样，可能需要花上几分钟，但是 **拿到号之后才会离开窗口**
- **异步**： **不能直接拿到结果**，比如在餐厅门口等位，拿到号之后我们可以去逛街，那什么时候真正吃上饭呢？我们可以：
    - 每 10 分钟去餐厅问一下（**轮询**）
    - 也可以扫码用微信接收通知（**回调**）

AJAX 举例：当 `request.send()` 之后，并不能直接得到 `response`，必须要等待 `readyState` 变成 `4` 之后，浏览器回头调用 `request.onreadystatechange` 函数

## 回调（callback）

- 写给自己的函数，不是回调
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

## 异步与回调

### 关联

异步任务需要在得到结果时通知 JS 来拿结果，怎么通知呢？

1. 可以让 JS 留一个函数地址（电话号码）给浏览器
2. 异步任务完成之后，浏览器调用该函数的地址（拨打电话）
3. 同时 **把结果作为参数** 传给该函数（电话里说可以来吃了）

这个函数是我写给浏览器调用的，所以是回调函数

### 区别

异步任务需要用回调函数来通知结果

- 但异步也可以用 **轮询**
- 回调也不一定用在异步里面，比如 `array.forEach( n ⇒ console.log(n) )`，这就是 **同步回调**

## 异步函数

[MDN Async Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/async_function)

如果一个函数的返回值处于以下这三个内部，即为异步函数：

- `setTimeout`
- AJAX（即 `XMLHttpRequest`）
- `AddEventListener`

也可以设置同步的 AJAX，`request.open("get", "/5.json", false)`，但是这样会使页面在请求期间卡住

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

## Promise

> 如果异步任务有两个结果：成功或失败，怎么做？

方法一：回调接受两个参数

```js
fs.readFile('./1.txt', (error, data) => {
  if(error){ console.log('失败'); return }
    console.log(data.toString())
  }
)
```

方法二：使用两个回调

```js
ajax('get', '/1.json', data=>{}, error=>{})
ajax('get', '/1.json', {
  success: ()=>{}, fail: ()=>{}
})
```

但是这两个方法都有问题：

1. 不规范，名称五花八门，有人用 `success + error`，有人用 `success + fail`，有人用 `done + fail`
2. 容易出现回调地狱，代码看不懂
3. 很难进行错误处理

### 回调地狱

```js
getUser( user => {
  getGroups( user, groups => {
    groups.forEach( (g) => {
      g.filter( x => x.owenerId === user.id)
        .forEach( x => console.log(x))
    })
  })
})
```

### Promise

#### 基本用法

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

#### Promise 是如何解决回调地狱的

回调地狱：
1. 多层嵌套
2. 无法方便地进行错误处理

解决办法：
1. 回调函数延迟绑定，回调函数是通过后面的 then 方法传入的
2. 返回值穿透，根据 then 中回调函数的传入值创建不同类型的 Promise，再把返回的 Promise 穿透到外层，以供后续使用。以上两点实现了链式调用，解决多层嵌套问题
3. 错误冒泡，前面产生的错误会一直向后传递，被 catch 接收到，就不用频繁地检查错误了

#### Promise.all()

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

#### Promise.race()

可以传入多个 promise，谁快，Promise.race() 就会跟谁一样，如果传的迭代是空的，则返回的 promise 将永远等待。