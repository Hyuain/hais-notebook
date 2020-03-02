---
title: FAQs：原生 JavaScript
date: 2020-02-04 10:30:50
tags:
  - 入门
  - 收集
  - FAQ
categories:
  - [前端, 其他]
---

关于原生 JavaScript 的一些问题与答案。

<!-- more -->

# Promise

## Promise 如何消灭回调地狱

回调地狱：
1. 多层嵌套
2. 无法方便地进行错误处理

解决办法：
1. 回调函数延迟绑定，回调函数是通过后面的 then 方法传入的
2. 返回值穿透，根据 then 中回调函数的传入值创建不同类型的 Promise，再把返回的 Promise 穿透到外层，以供后续使用。以上两点实现了链式调用，解决多层嵌套问题
3. 错误冒泡，前面产生的错误会一直向后传递，被 catch 接收到，就不用频繁地检查错误了

## Promise 怎样实现链式调用的

```js
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

function MyPromise(executor) {
  // 缓存当前 Promise 实例
  let self = this
  self.value = null
  self.error = null
  self.status = PENDING
  // 回调函数有可能是个数组
  self.onFulfilledCallbacks = []
  self.onRejectedCallbacks = []

  const resolve = value => {
    if (self.status !== PENDING) return
    setTimeout(() => {
      self.status = FULFILLED
      self.value = value
      self.onFulfilledCallbacks.forEach(callback => callback(self.value))
    })
  }
  
  const reject = error => {
    if (self.status !== PENDING) return
    setTimeout(() => {
      self.status = REJECTED
      self.error = error
      self.onRejectedCallbacks.forEach(callback => callback(self.error))
    })
  }

  executor(resolve, reject)
}

MyPromise.prototype.then = function(onFulfilled, onRejected) {
  const self = this
  let bridgePromise
  onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value
  onRejected = typeof onRejected === 'function' ? onRejected : error => { throw error }

  function resolvePromise(bridgePromise, x, resolve, reject) {
    if (x instanceof MyPromise) {
      // 拆解这个 promise，直到返回值不为 promise 为止
      if (x.status === PENDING) {
        x.then(y => {
          resolvePromise(bridgePromise, y, resolve, reject)
        }, error => {
          reject(error)
        })
      }
    } else {
      resolve(x)
    }
  }

  if (self.status === PENDING) {
    return bridgePromise =  new MyPromise((resolve, reject) => {
      self.onFulfilledCallbacks.push(value => {
        // 要拿到 then 中回调返回的结果
        try {
          let x = onFulfilled(value)
          resolvePromise(bridgePromise, x, resolve, reject)
        } catch (e) {
          reject(e)
        }
      })
      self.onRejectedCallbacks.push(error => {
        try {
          reject(onRejected(error))
        } catch (e) {
          reject(e)
        }
      })
    })
  } else if (self.status === FULFILLED) {
    return bridgePromise = new MyPromise((resolve, reject) => {
      setTimeout(() => {
        try {
          let x = onFulfilled(self.value)
          resolvePromise(bridgePromise, x, resolve, reject)
        } catch (e) {
          reject(e)
        }
      })
    })
  } else {
    return bridgePromise = new MyPromise((resolve, reject) => {
      setTimeout(() => {
        try {
          let x = onRejected(self.error)
          resolvePromise(bridgePromise, x, resolve, reject)
        } catch (e) {
          reject(e)
        }
      })
    })
  }
}

MyPromise.prototype.catch = function (onRejected) {
  return this.then(null, onRejected)
}
```


# 前端路由

> 路由就是根据不同的 URL 来展示不同的内容或页面

用户每次提交表单就要重新刷新页面 -> 催生了 AJAX；
用户在多页面之间跳转体验很差 -> 催生了单页应用（SPA, Single-Page Application）
单页应用页面本身的 URL 没有变化，导致其无法记住用户的操作，对 SEO 也不友好 -> 催生了前端路由

> 前端路由就是在 **保证只有一个 HTML 页面**，且与用户交互时不刷新和跳转页面的同时，为 SPA 中的每个视图展示形式匹配一个 **特殊的 URL**
> 在刷新、前进、后退和 SEO 时均通过这个特殊的 URL 来实现
> 并且改变 URL 时不向服务器发送请求

## Hash 与 History

### Hash

由于 hash 值的变化不会导致浏览器像服务器发送请求，而且 hash 的改变会触发 `hashchange` 事件，浏览器的前进后退也能对其进行控制，所以在 H5 的 history 模式出现之前，基本都是使用 hash 模式来实现前端路由。

#### 基本 API

```js
window.location.hash = 'string' // 用于设置 hash 值
let hash = window.location.hash // 获取当前 hash 值

// 监听 hash 变化，点击浏览器的前进后退会触发：
window.addEventListener('hashchange', function(event) {
  let newURL = event.newURL // hash 改变后的新 URL
  let oldURL = event.oldURL // hash 改变前的旧 URL
})

// 也可以简写成
window.onhashchange((event) => {
  // do something
})
```

#### 实现一个路由对象

```js
class hashRouter {
  constructor() {
    // 存储不同的 hash 值对应的回调函数
    this.routers = {}
    // 通过 hashchange 来监听 hash 变化
    window.addEventListener('hashchange', this.load.bind(this))
  }
  // 注册每个视图（注册每个 hash 值对应的回调函数）
  register(hash, callback = function() {}) {
    this.routers[hash] = callback
  }
  // 注册首页的回调（没有 hash 值时默认为首页）
  registerIndex(callback = function() {}) {
    this.routers['index'] = callback
  }
  // 处理视图未找到的情况
  registerNotFound(callback = function() {}) {
    this.routers['404'] = callback
  }
  // 处理异常情况
  registerError(callback = function() {}) {
    this.routers['error'] = callback
  }
  load() {
    let hash = location.hash.slice(1)
    let handler
      // 没有 hash 默认首页
    if (!hash) {
      handler = this.routers.index
      // 没找到对应的 hash 值
    } else if (!this.routers.hasOwnProperty(hash)) {
      handler = this.routers['404'] || function() {}
      // 其他普通情况
    } else {
      handler = this.routers[hash]
    }
    // 执行注册的回调函数
    try {
      handler.call(this)
    } catch (e) {
      console.error(e)
      (this.routers['error'] || function() {}).call(this, e)
    }
  }
}
```

然后再这样使用：

```html
<body>
  <div id="nav">
    <a href="#/page1">page1</a>
    <a href="#/page2">page2</a>
    <a href="#/page3">page3</a>
    <a href="#/page4">page4</a>
    <a href="#/page5">page5</a>
  </div>
  <div id="container"></div>
</body>
```

```js
let router = new HashRouter()
let container = document.getElementById('container')

// 注册首页回调函数
router.registerIndex(() => container.innerHTML = '我是首页')

// 注册其他视图回调函数
router.register('/page1', () => container.innerHTML = '我是 page1')
router.register('/page2', () => container.innerHTML = '我是 page2')
router.register('/page3', () => container.innerHTML = '我是 page3')
router.register('/page4', () => {throw new Error('抛出一个异常')})

// 注册未找到对应 hash 值时的回调
router.regierNotFound(() => container.innerHTML = '页面未找到')
// 注册出现异常时的回调
router.registerError((e) => container.innerHTML = '页面异常，错误信息：<br>' + e.message)
// 加载视图
router.load()
```

### History

在 HTML5 之前，浏览器就已经有了 history 对象，但早期的只有 `go` `forward` `back` 这些方法，用于多页面之间的跳转，HTML 5 中新加入了：

```js
history.pushState() // 添加新的状态到历史状态栈
history.replaceState() // 用新的状态代替当前状态
history.state // 返回当前状态对象
```

与 hash 不同，history 的改变并不会触发任何事件，所以我们无法直接监听 history 的改变而做出相应的改变

而对于单页应用的 history 模式而言，URL 的改变只有可能是以下四种情况：

1. 点击浏览器的而前进或后退按钮
2. 点击 a 标签
3. 在 JS 代码中触发 `history.pushState` 函数
4. 在 JS 代码中触发 `history.replaceState` 函数

#### 实现一个路由对象

```js
class HistoryRouter {
  constructor() {
    // 存储不同 path 值对应的回调函数
    this.routers = {}
    this.listenPopState()
    this.listenLink()
  }
  // 监听 popstate 事件，处理前进和后退时应该如何调用
  listenPopState() {
    window.addEventListener('popstate', (e) => {
      let state = e.state || {}
      let path = state.path || ''
      this.dealPathHandler(path)
    })
  }
  // 监听 a 标签，阻止他的默认事件，并且调用 pushState 方法
  listenLink() {
    window.addEventListener('click', (e) => {
      let dom = e.target
      if ((dom.tagName.toLowerCase === 'a') && dom.getAttribute('href')) {
        e.preventDefault()
        this.assign(dom.getAttribute('href'))
      }
    })
  }
  // 首次进入页面时调用
  load() {
    let path = location.pathname
    this.dealPathHandler(path)
  }
  register(path, callback = function() {}) {
    this.routers[path] = callback
  }
  registerIndex(callback = function() {}) {
    this.routers['/'] = callback
  }
  registerNotFound(callback = function() {}) {
    this.routers['404'] = callback
  }
  registerError(callback = function() {}) {
    this.routers['error'] = callback
  }
  // 触发 pushState
  assign(path) {
    history.pushState({path}, null, path)
    this.dealPathHandler(path)
  }
  // 触发 replaceState
  replace(path) {
    history.replaceState({path}, null, path)
    this.dealPathHandler(path)
  }
  dealPathHandler(path) {
    let handler
    if(!this.routers.hasOwnProperty(path)) {
      handler = this.routers['404'] || function() {}
    } else {
      handler = this.routers[path]
    }
    try {
      handler.call(this)
    } catch (e) {
      console.error(e)
      (this.routers['error'] || function() {}).call(this, e)
    }
  }
}
```

同样的使用方法：

```html
<body>
  <div id="nav">
    <a href="#/page1">page1</a>
    <a href="#/page2">page2</a>
    <a href="#/page3">page3</a>
    <a href="#/page4">page4</a>
    <a href="#/page5">page5</a>
    <button id="btn">page2</button>
  </div>
  <div id="container"></div>
</body>
```

```js
let router = new HashRouter()
let container = document.getElementById('container')

// 注册首页回调函数
router.registerIndex(() => container.innerHTML = '我是首页')

// 注册其他视图回调函数
router.register('/page1', () => container.innerHTML = '我是 page1')
router.register('/page2', () => container.innerHTML = '我是 page2')
router.register('/page3', () => container.innerHTML = '我是 page3')
router.register('/page4', () => {throw new Error('抛出一个异常')})

btn.onclick = () => router.assign('/page2')

// 注册未找到对应 path 值时的回调
router.regierNotFound(() => container.innerHTML = '页面未找到')
// 注册出现异常时的回调
router.registerError((e) => container.innerHTML = '页面异常，错误信息：<br>' + e.message)
// 加载视图
router.load()
```

## 各自的优缺点及使用场景

### 为什么 hash 模式需要服务器端配合？

我们通过 history 来修改 URL 后，页面不会刷新；如果我们手动刷新页面，或者通过 URL 直接进入应用时，服务端是无法识别这个 URL 的——因为我们其实只有一个 html 文件

因此，如果要使用 history 模式，就需要在服务端增加一个覆盖所有情况的候选资源，如果 URL 匹配不到任何静态资源，则应返回单页应用的 html 文件

### 各自的优缺点

|     | hash | history |
| --- | --- | --- |
| 观赏性 | 丑 | 美 |
| 兼容性 | >IE8 | >IE10 |
| 其他 | 锚点功能失效，相同 hash 值不触发动作 | 需要服务端配合，相同 path 值也可以通过 pushState 触发动作 |

# 函数防抖与函数节流

## 函数防抖

等了一段时间，没有新的人上车了再发车：任务频繁触发的情况下，只有任务触发的间隔超过指定间隔的时候，任务才会执行

```js
const debounce = (fn, delay = 300) => {
  let timer = null
  return function() {
    clearTimeout(timer)
    timer = setTimeout(() => {
      fn.apply(this, arguments)
    }, delay)
  }
}
```

## 函数节流

冷却时间：指定时间间隔内只会执行一次任务

```js
const throttle = (fn, interval = 300) => {
  let canRun = true
  return function() {
    if (!canRun) return
    canRun = false
    setTimeout(() => {
      fn.apply(this, arguments)
      canRun = true
    }, interval)
  }
}
```

# 如何用正则实现 `trim()`

```js
const trim = (string) => {
  return string.replace(/^\s+|\s+$/g, '')
}
```

# 对象和数组的深拷贝

```js
const obj = {
  a: 100,
  b: [10, 20, 30],
  c: {
    x: 10
  },
  d: /^\d+$/
}
```

## 直接赋值

```js
const obj2 = obj1
```

当改变 `obj1` 的时候，`obj2` 会跟着改变，这是直接赋值，不属于深/浅克隆

## 浅克隆

> 只克隆第一层

### 手动实现

```js
const obj2 = {}
for(let key in obj) {
  if(!obj.hasOwnProperty(key)) continue // 这里也可以用 break，因为到这一步基本就已经到原型的属性了，可以直接跳出循环
  obj2[key] = obj[key]
}
```

### Object.assign

他拷贝的是对象属性的引用，而不是对象本身

```js
let obj2 = Object.assign({}, obj)
```

### 展开运算符

```js
const obj2 = { ...obj }
```

### concat

```js
let arr = [1, 2, 3]
let newArr = arr.concat()
```

### slice

```js
let arr = [1, 2, 3]
let newArr = arr.slice()
```

## 深克隆

> 每一层都克隆

### 简易版的深拷贝

最最最简单的深拷贝可以这样写：

```js
const obj2 = JSON.parse(JSON.stringify(obj))
```

但是在有些时候是会出现问题的：
1. 循环引用
2. 无法处理 RegExp、Date、Set、Map、Function 等

也可以用 `lodash` 之类的库，当然也可以手写一个简易版的

```js
function deepClone(target) {
  if (typeof target === 'object' && target !== null) {
    const cloneTarget = Array.isArray(target) ? [] : {}
    for (let prop in target) {
      if (target.hasOwnProperty(prop)) {
        cloneTarget[prop] = deepClone(target[prop])
      }
    }
    return cloneTarget
  } else {
    return target
  }
}
```

### 解决循环引用问题

思路：创建一个 Map，将已经拷贝过的对象记录下来，如果发现已经拷贝过，就直接返回他

```js
const isObject = (target) => (typeof target === 'object' || typeof target === 'function') && target !== null

const deepClone = (target, map = new weakMap()) => {
  if (map.get(target)) {
    return target
  }
  
  if (isObject(target)) {
    map.set(target, true)
    const cloneTarget = Array.isArray(target) ? [] : {}
    for (let prop in target) {
      if (target.hasOwnProperty(prop)) {
        cloneTarget[prop] = deepClone(target[prop], map)
      }
    }
    return cloneTarget
  } else {
    return target
  }
}
```

如果使用 Map，那么 map 的 key 将会与 map 形成强引用，如果强引用一直存在，那么对象将无法被回收；使用 WeakMap 可以构造一种弱引用（一个对象若只被弱引用所引用，则被认为是不可访问（或弱可访问）的，并因此可能在任何时刻被回收），WeakMap 的 Key 必须是对象，而值可以是任意的。

### 拷贝特殊对象

#### 可继续遍历

比如 Map、Set、Array、Object、Arguments：

```js
const getType = Object.prototype.toString.call
const isObject = (target) => (typeof target === 'object' || typeof target === 'function') && target !== null

const canTraverse = {
  '[object Map]': true,
  '[object Set]': true,
  '[object Array]': true,
  '[object Object]': true,
  '[object Arguments]': true
}

const deepClone = (target, map = new WeakMap()) => {
  if (!isObject(target)) return target
  let type = getType(target)
  let cloneTarget
  if (!canTraverse[type]) {
    // 处理不可遍历对象
  } else {
    cloneTarget = new target.constructor()
  }
  
  if (map.get(target)) return target
  map.set(target, true)

  if (type === '[object Map]') {
    target.forEach((item, key) => {
      cloneTarget.set(deepClone(key), deepClone(item))
    })
  }

  if (type === '[object Set]') {
    target.forEach(item => {
      target.add(deepClone(item))
    })
  }


  // 处理 Object 和 Array
  for (let prop in target) {
    if (target.hasOwnProperty(prop)) {
      cloneTarget[prop] = deepClone(target[prop])
    }
  }
  return cloneTarget
}
```

#### 不可遍历

```js
const boolTag = '[object Boolean]';
const numberTag = '[object Number]';
const stringTag = '[object String]';
const dateTag = '[object Date]';
const errorTag = '[object Error]';
const regExpTag = '[object RegExp]';
const funcTag = '[object Function]';

const handleRegExp = (target) => {
  const {source, flags} = target
  return new target.constructor(source, flags)
}

// 需要区分处理箭头函数和普通函数，因为普通函数是 Function 的实例，而箭头函数不是任何类的实例
const handleFunc = (func) => {
  // 箭头函数返回自身
  if (!func.prototype) return func
  const bodyReg = /(?<={)(.|\n)(?=})/m
  const paramReg = /(?<=\().+(?=\)\s*{)/
  const funcString = func.toString()
  const param = paramReg.exec(funcString)
  const body = bodyReg.exec(funcString)
  if (!body) return null
  if (param) {
    const paramArr = param[0].slice(',')
    return new Function(...param, body[0])
  } else {
    return new Function(body[0])
  }
}

const handleNotTraverse = (target, tag) => {
  const constructor = target.constructor
  switch (tag) {
    case boolTag:
      // 因为用 new Boolean 创造出来的对象有问题
      return new Object(Boolean.prototype.valueOf.call(target))
    case numberTag:
      return new Object(Number.prototype.valueOf.call(target))
    case stringTag:
      return new Object(String.prototype.valueOf.call(target))
    case errorTag:
    case dateTag:
      return new constructor(target)
    case regExpTag:
      return handleRegExp(target)
    case funcTag:
      return handleFunc(target)
    default:
      return new constructor(target)
  }
}
```

### 完整代码

```js
const getType = target => Object.prototype.toString.call(target).match(/\[object (.*?)\]/)[1].toLowerCase()

const isObject = target => (typeof target === 'object' || typeof target === 'function') && target !== null

const canTraverse = {
  'map': true,
  'set': true,
  'array': true,
  'object': true,
  'arguments': true
}

const handleRegExp = target => {
  const {source,flags} = target
  return new target.constructor(source, flags)
}

const handleFunc = func => {
  if (!func.prototype) return func
  const bodyReg = /(?<={)(.|\n)(?=})/m
  const paramReg = /(?<=\().+(?=\)\s+{)/
  const funcString = func.toString()
  const param = paramReg.exec(funcString)
  const body = bodyReg.exec(funcString)
  if (!body) return null
  if (param) {
    const paramArr = param[0].split(',')
    return new Function(...paramArr, body[0])
  } else {
    return new Function(body[0])
  }
}

const handleNotTraverse = (target, type) => {
  const constructor = target.constructor
  switch (type) {
    case 'boolean':
      return new Object(Boolean.prototype.valueOf.call(target))
    case 'number':
      return new Object(Number.prototype.valueOf.call(target))
    case 'string':
      return new Object(String.prototype.valueOf.call(target))
    case 'symbol':
      return new Object(Symbol.prototype.valueOf.call(target))
    case 'error':
    case 'date':
      return new constructor(target)
    case 'regexp':
      return handleRegExp(target)
    case 'function':
      return handleFunc(target)
    default:
      return new constructor(target)
  }
}

const deepClone = (target, map = new WeakMap()) => {
  if (!isObject(target)) {
    return target
  }
  const type = getType(target)
  let cloneTarget
  if (!canTraverse[type]) {
    return handleNotTraverse(target, type)
  } else {
    cloneTarget = new target.constructor()
  }

  if (map.get(target)) return target
  map.set(target, true)

  if (type === 'map') {
    target.forEach((item, key) => {
      cloneTarget.set(deepClone(key, map), deepClone(item, map))
    })
  }
  
  if (type === 'set') {
    target.forEach(item => {
      cloneTarget.add(deepClone(item, map))
    })
  }

  for (let prop in target) {
    if (target.hasOwnProperty(prop)) {
      cloneTarget[prop] = deepClone(target[prop], map)
    }
  }
  return cloneTarget
}
```

# 关于堆栈内存和闭包

```js
// example 1
let a = {},
    b = '0',
    c = 0
a[b]= 'Harvey'
a[c] = 'Zhang'
console.log(a[b]) // => 'Zhang'

// example 2
let a = {},
    b = Symbol('1'),
    c = Symbol('1')
a[b] = 'Harvey'
a[c] = 'Zhang'
console.log(a[b]) // => 'Harvey'

// example 3
let a = {},
    b = { n: '1' },
    c = { m: '2' }
a[b] = 'Harvey'
a[c] = 'Zhang'
console.log(a[b]) // => 'Zhang'
// 所有的引用类型存的时候都会被转换成字符串，而 object 转换成字符串的结果都是 '[object Object]'
```

{% note warning %}
堆：存储引用类型值的空间；
栈：存储基本类型值和执行代码的环境；浏览器加载页面就会形成栈内存，当执行函数的时候，都会形成一个新的执行上下文（Execution Context Stack），压入栈中执行
{% endnote %}

闭包的作用 1：保存——保存私有变量，不被销毁

```js
var test = (function (i) {
  return function () {
    alert(i *= 2)
  }
})(2) // => var test = AAAFFF111
test(5) // => '4' 
```

```text
┌─────────────────────────────────────────────────
│ 堆内存：地址 AAAFFF111
│ 作为函数，存储代码（字符串）
│ 作为对象，存储键值对（prototype、length、name ...）
└─────────────────────────────────────────────────

┌─────────────────────────────────────────────────
│ 栈内存：执行上下文 1，不销毁，因为 i 被占着
│ i = 2
│ return AAAFFF111
└─────────────────────────────────────────────────
    ↑
    | 上级作用域
    |
┌─────────────────────────────────────────────────
│ 栈内存：执行上下文 2，执行完之后销毁
│ alert(i *= 2)
└─────────────────────────────────────────────────
```

闭包的作用 2：保护——修改自己的私有变量，不会影响外面的东西

```js
var a = 0,
    b = 0
function A(a) { // 全局 A = AAAFFF000
  A = function (b) {
    alert(a + b++)
  }
  alert(a++)
}
A(1) // => '1', a = 2
A(2) // => '4', b = 3
```

```text
┌─────────────────────────────────────────────────
│ Global Object
| a = 0
| b = 0
| A = AAAFFF000
└─────────────────────────────────────────────────

┌─────────────────────────────────────────────────
│ 堆内存：AAAFFF000
│ `A = function(b) { alert(a + b++) }; alert(a++)`
└─────────────────────────────────────────────────
┌─────────────────────────────────────────────────
│ 堆内存：BBBFFF000
│ `alert(a + b++)`
└─────────────────────────────────────────────────

┌─────────────────────────────────────────────────
│ 栈内存：A(1) ECStack，因为 BBBFFF000 被全局 A 占用了，所以不销毁
│ a = 1
| A = BBBFFF000
| alert(a++)
└─────────────────────────────────────────────────
┌─────────────────────────────────────────────────
│ 栈内存：A(2) ECStack，没有被占用的，所以销毁
│ b = 2
| alert(a + b++)
└─────────────────────────────────────────────────
```

# 面向对象

```js
function Foo() { // 存在变量提升
  getName = function() {
    console.log(1)
  }
  return this
}
Foo.getName = function() {
  console.log(2)
}
Foo.prototype.getName = function() {
  console.log(3)
}
var getName = function() { // 存在变量提升
  console.log(4)
}
function getName() { // 存在变量提升
  console.log(5)
}

Foo.getName() // 2
getName() // 4
Foo().getName() // Foo() 作为普通函数执行，改变了全局的 getName() 为 1，返回 this（为 window）
getName() // 1
new Foo.getName() // 前面是无参数 new（优先级18），后面是成员访问（优先级19），先执行成员访问，获取 Foo.getName()，为 2，再 new（相当于普通函数执行）
new Foo().getName() // 前面是有参数 new（优先级19），后面是成员访问（优先级19），先 new（创建实例），getName 是原型上的 getName，3
new new Foo().getName() // 先 new（创建实例 xxx），变为 new xxx.getName()，再算成员访问（原型上的方法），为 3，再 new
```

```text
┌─────────────────────────────────────────────────
│ 堆内存：AAAFFF000
| 代码字符串
| getName: fun -> 2
| prototype: BBBFFF000
└─────────────────────────────────────────────────
┌─────────────────────────────────────────────────
│ 堆内存：BBBFFF000
│ constructor: Foo
| getName: func -> 3
└─────────────────────────────────────────────────

┌─────────────────────────────────────────────────
│ 变量提升阶段
| Foo = AAAFFF000，声明并定义
│ getName = func -> 5，var 声明，之后在 function 赋值
└─────────────────────────────────────────────────
┌─────────────────────────────────────────────────
│ 代码执行阶段
| getName = func -> 4，在 var 处赋值
└─────────────────────────────────────────────────
```

# parseUrl 函数

可以用 `new URL` API 或者创建一个 `<a>` 标签的，并获取他的属性：

new URL 在 IE 上兼容性不是很好，但是 Node 也支持这个 API。

```js
const parseUrl = source => {
  const url = new URL(source)
  const query = {}
  const params = url.search.replace(/^\?/, '').split('&')
  params.forEach(item => {
    const param = item.split('=')
    query[param[0]] = param[1]
  })
  return {
    protocol: url.protocol.replace(':', ''),
    host: url.host,
    path: url.pathname,
    query: query,
    hash: url.hash.replace('#', '')
  }
}

console.log(parseUrl("http://www.xiyanghui.com/product/list?id=123456&sort=discount#title"))
```

```js
const parseUrl = source => {
  const a = document.createElement('a')
  a.href = source
  // 下面的内容是一样的
  const query = {}
  const params = a.search.replace(/^\?/,'').split('&');
  params.forEach(item => {
    const param = item.split('=')
    query[param[0]] = param[1]
  })
  return {
    protocol: a.protocol.replace(':',''),
    host: a.host,
    path: a.pathname,
    query: query,
    hash: a.hash.replace('#','')
  }
};

console.log(parseUrl("http://www.xiyanghui.com/product/list?id=123456&sort=discount#title"))
```

# EventLoop

## 浏览器中的 EventLoop

浏览器中的 EventLoop 有 2 个阶段：**宏任务** 和 **微任务**

1. 第一个 **宏任务**：整段脚本。执行过程中的 **同步代码** 直接执行，**宏任务** 进入宏任务队列，**微任务** 进入微任务队列
2. 当前 **宏任务** 执行完出队，检查 **微任务** 队列，若有则依次执行，直至微任务队列为空
3. 执行浏览器 **UI 线程** 的渲染工作
4. 检查是否有 **Web worker** 任务，有则执行
5. 执行队首新的 **宏任务**，回到 2 循环至宏任务队列为空

### 一些例子

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

## Node.js 中的 EventLoop

![](https://user-gold-cdn.xitu.io/2019/11/23/16e96b8587ad911d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 三大关键阶段

1. **timer**：执行定时器回调阶段，检查定时器（`setTimeout` `setInterval`），如果到了时间就执行回调。
2. **poll**：轮询阶段，当文件 I/O、网络 I/O 等一步操作执行完之后，通过 `data` `connect` 等事件来通知 JS 主线程，使得时间循环到达 poll 阶段：
    - 如果当前已经存在定时器，且定时器到时间了，拿出来执行，eventLoop 回到 timer 阶段
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

### 一些例子

注意：有可能先执行 JS 代码，也有可能先开启 EventLoop，因为进程开启都需要时间。

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
如果 EventLoop 开启得很快，则此时 timers 中没有 `fn`；
如果 EventLoop 开启得慢，则此时 timers 中有 `fn`，而如果此时 timers 中有 `fn`，而且计时已经结束，就马上执行了 `fn` 再往下走

也就是说因为 EventLoop 有时候开启得快，有时候开启得慢，所以如果一开始就执行上面两句代码，他们的执行先后是不确定的！

# JS 垃圾回收

## 内存的生命周期

JS 环境中的内存声明周期由以下三部分组成：

1. 内存分配：当我们声明变量、函数、对象的时候，系统会自动分配内存
2. 内存使用：使用变量、函数等
3. 内存回收

## 垃圾回收

### 什么是垃圾

- 没有被引用的，所谓引用就是一个对象拥有访问另一个对象的权限（隐式或显式）
- 引用的几个对象互相组成环

### 垃圾回收算法

### 引用计数法

每次生成一个新东西，就将被引用的对象的计数 +1，每次删除一个东西，就将被引用的对象计数 -1
但是若出现循环引用，则无法回收

```js
var div = document.createElement('div')
div.onclick = function() {
  console.log('click')
}
```

此处的 div 引用了事件处理函数，事件处理函数也引用了 div，因为 div 可以在事件处理函数中被访问到。

### 标记清除法

1. 标记所有变量
2. 从根部清除所有能触及的对象的标记
3. 删除掉还有标记的变量

解决除了循环引用的问题，因为循环引用的对象无法从全局对象出发再获取到他们的引用。

## 内存泄漏

经验法则是，如果连续五次垃圾回收之后，内存占用一次比一次大，就有内存泄漏

### 常见的内存泄露

#### 意外的全局变量

```js
function foo() {
  bar1 = 'some text' // 实际上声明了全局变量 window.bar1
  this.bar2 = 'some text' // 全局变量 window.bar2
}
```

#### 被遗忘的计时器和回调函数

```js
var serverData = loadData()
setInterval(function() {
  var renderer = document.getElementById('renderer')
  if(renderer) {
    renderer.innerHTML = JSON.stringify(serverData)
  }
}, 5000)
```

如果后续移除了 renderer，那么其实计时器已经没用了，但是没有回收计时器的话，计时器仍然后效，他依赖的 serverData 也依然有效

#### DOM 引用

前端除了 JS 进程以外，还有 DOM 进程：

- 只使用 `div.remove` 只是将 `div` 从页面中删掉，但在内存中还在
- 只使用 `div = null`，而没有 `remove` 的话，`div` 还在 DOM 中，他就不会被回收

此外还有一些特殊情况，比如如果我们引用了一个表格中的 td 元素，一旦在 DOM 中删除了整个表格，我们直观的觉得内存回收应该回收除了被引用的 td 外的其他元素。但是事实上，这个 td 元素是整个表格的一个子元素，并保留对于其父元素的引用。这就会导致对于整个表格，都无法进行内存回收。

# 与后端进行数据交互的方法

## Form 表单

- 只支持 GET 和 POST 类型
- 有问无答
- 提交后页面会刷新，用户体验不佳

## AJAX

- 支持多种请求方法
- XMLHttpRequest
- Fetch
- POST 请求编码方式
  - multipart/formData
```js
// 可以用
let formData = new FormData()
formData.append('','')
// 也可以直接将 Form 表单的 DOM 对象直接放进去
let formData = new FormData($form)
```

## WebSocket

- 服务器可以主动发起请求

# 关于时间

- ISO 8601
- Moment.js
- Day.js

# 代码题

```js
var a = {name: 'x'}
a.x = a = {}
console.log(a.x) // undefined

// 代码执行分为两步： 1. 确定 a 是什么；2. 执行（从右向左）
// 执行到 a.x 的时候，a 还是原来的地址，但是他右边的 a 已经是新的地址了
```