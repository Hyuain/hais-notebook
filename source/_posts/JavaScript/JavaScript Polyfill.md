---
title: JavaScript Polyfill
date: 2020-02-23 16:33:44
categories:
  - - 前端
tags:
  - FX
---

实现一些常见的 API。

<!-- more -->

# Array

我们可以手动实现一些[[JavaScript Arrays|数组]]的简单方法，比如 `map`、`reduce` 等

## map

```js
Array.prototype.myMap = function(callback, thisArg) {
  if (this === null || this === undefined) {
    throw new TypeError('Cannot read property "map" of null or undefined')
  }
  if (Object.prototype.toString.call(callback) !== '[object Function]') {
    throw new TypeError(callback + 'is not a function')
  }

  // 草案中提到要先转换为对象
  let O = Object(this)
  let T = thisArg
  
  // 右移 0 位，可以把前面的空位用 0 填充，实际上是为了保证 len 为数字且为整数，因为位运算会丢弃小数
  let len = O.length >>> 0
  let A = new Array(len)
  for (let k = 0; k < len; k++) {
    if (k in O) {
      let kValue = O[k]
      A[k] = callback.call(T, kValue, k, O)
    }
  }
  return A
}
```

## reduce

```js
Array.prototype.myReduce = function(callback, initialValue) {
  if (this === null || this === undefined) {
    throw new TypeError('Cannot read property "reduce" of null or undefined')
  }
  if (Object.prototype.toString.call(callback) !== '[object Function]') {
    throw new TypeError(callback + 'is not a function')
  }
  
  let O = Object(this)
  let len = O.length >>> 0
  let k = 0
  let accumulator = initialValue
  if (accumulator === undefined) {
    for (; k < len; k++) {
      if (k in O) {
        accumulator = O[k]
        k++
        break
      }
    }
  }
  
  if (k === len && accumulator === undefined) {
    throw new Error('Each element of array is empty')
  }
  for (; k < len; k++) {
    if (k in O) {
      accumulator = callback.call(undefined, accumulator, O[k], k, O)
    }
  }
  return accumulator
}
```

## push

```js
Array.prototype.myPush = function(...items) {
  let O = Object(this)
  let len = O.length >>> 0
  let argCount = items.length >>> 0
  // 2 ** 53 - 1 是 JS 能表示的最大整数
  if (len + argCount > 2 ** 53 - 1) {
    throw new TypeError('The number of array is over the max value restricted')
  }
  for (let i = 0; i < argCount; i++) {
    O[len + i] = items[i]
  }
  let newLength = len + argCount
  O.length = newLength
  return newLength
}
```

## pop

```js
Array.prototype.myPop = function() {
  let O = Object(this)
  let len = O.length >>> 0
  if (len === 0) {
    return undefined
  }
  len--
  let value = O[len]
  delete O[len]
  O.length = len
  return value
}
```

## filter

```js
Array.prototype.myFilter = function(callback, thisArg) {
  if (this === null || this === undefined) {
    throw new TypeError('Cannot read property "filter" of null or undefined')
  }
  if (Object.prototype.toString.call(callback) !== '[object Function]') {
    throw new TypeError(callback + 'is not a function')
  }
  
  let O = Object(thisArg)
  let len = O.length >>> 0
  let resLen = 0
  let res = []
  for (let i = 0; i < len; i++) {
    if (i in O) {
      let element = O[i]
      if (callback.call(thisArg, O[i], i, O)) {
        res[resLen++] = element
      }
    }
  }
  return res
}
```

## splice

```js
const sliceDeleteElements = (array, startIndex, deleteCount, deleteArray) => {
  for (let i = 0; i < deleteCount; i++) {
    let index = startIndex + i
    if (index in array) {
      deleteArray[i] = array[index]
    }
  }
}

const movePostElements = (array, startIndex, len, deleteCount, addElements) => {
  // 如果增加与删除一样多，就不动
  if (deleteCount === addElements.length) return
  // 如果删除比新增多，就左移
  if (deleteCount > addElements.length) {
    for (let i = startIndex + deleteCount; i < len; i++) {
      let fromIndex = i
      let toIndex = i - (deleteCount - addElements.length)
      if (fromIndex in array) {
        array[toIndex] = array[fromIndex]
      } else {
        delete array[toIndex]
      }
    }
    // 删除因为挪动而空出来的位置
    for (let i = len - 1; i >= len + addElements.length - deleteCount; i--) {
      delete array[i]
    }
  }
  // 如果删除比新增少，就右移
  if (deleteCount < addElements.length) {
    for (let i = len - 1; i >= startIndex + deleteCount; i--) {
      let fromIndex = i
      let toIndex = i + (addElements.length - deleteCount)
      if (fromIndex in array) {
        array[toIndex] = array[fromIndex]
      } else {
        delete array[toIndex]
      }
    }
  }
}

const computeStartIndex = (startIndex, len) => {
  // 处理索引负数的情况
  if (startIndex < 0) {
    return startIndex + len > 0 ? startIndex + len : 0
  }
  return startIndex >= len ? len : startIndex
}

const computeDeleteCount = (startIndex, len, deleteCount, argumentsLen) => {
  // 删除数目没有传，默认 startIndex 及后面所有的
  if (argumentsLen === 1) return len - startIndex
  // 删除数目过小
  if (deleteCount < 0) return 0
  // 删除数目过大
  if (deleteCount > len - startIndex) return len - startIndex
  return deleteCount
}

Array.prototype.mySplice = function(startIndex, deleteCount, ...addElements) {
  let argumentsLen = arguments.length
  let O = Object(this)
  let len = O.length
  let deleteArray = new Array(deleteCount)

  // 参数清洗，如果用户传来非法值怎么办
  startIndex = computeStartIndex(startIndex, len)
  deleteCount = computeDeleteCount(startIndex, len, deleteCount, argumentsLen)

  // 判断 seal 对象和 frozen 对象
  // 密封对象不可扩展，不能添加、删除方法和属性，但可以修改属性值
  // 冻结对象还不能修改属性值
  if (Object.isSealed(array) && deleteCount !== addElements.length) {
    throw new TypeError('the object is a sealed object')
  } else if (Object.isFrozen(array) && (deleteCount > 0 || addElements.length > 0)) {
    throw new TypeError('the object is a frozen object')
  }

  // 拷贝删除的元素
  sliceDeleteElements(array, startIndex, deleteCount, deleteArray)
  // 移动原来的数组中的元素
  movePostElements(array, startIndex, len, deleteCount, addElements)
  for (let i = 0; i < addElements.length; i++) {
    array[startIndex + i] = addElements[i]
  }
  array.length = len - deleteCount + addElements.length
  return deleteArray
}
```

## sort

关于数组排序算法的详细介绍请看 [[Array#Sorting|数组排序]]。 

# Promise

[[Promise]] 是 JavaScript 中一种异步编程的解决方案。

## 原版 Promise 的基本用法

```js
new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('FULFILLED')
  })
})
```

## 第一步：创建 class MyPromise

```js
class MyPromise {
  constructor(handle) { // handle 即为 (resolve, reject) => {...}
    if (typeof handle !== 'function') {
      throw new Error('MyPromise must accept a function as parameter')
    }
  }
}
```

## 第二步：处理 `MyPromise` 的状态和值

原版 Promise 对象存在三种状态：`Pending` `Fulfilled` `Rejected`

{% note warning %}
状态只能由 `Pending` 变为 `Fulfilled` 或 `Reject`，并且之后不能再变化
{% endnote %}

`resolve` 会将 Promise 对象的状态变为 `FulFilled`，而 `reject` 则会将其变为 `Pending`

而原版 Promise 的值说的是状态改变时传递给回调函数的值

```js
const PENDING = 'PENDING'
const FULFILLED = 'FULFILLED'
const REJECTED = 'REJECTED'

class MyPromise {
  constructor(handle) {
    if (typeof handle !== 'function') {
      throw new Error('MyPromise must accept a function as parameter')
    }
    // 添加状态
    this._status = PENDING
    // 添加值
    this._value = undefined
    // 执行 handle
    try {
      handle(this._resolve.bind(this), this._reject.bind(this))
    } catch (err) {
      this._reject(err)
    }
  }
  // 添加 resolve
  _resolve(val) {
    if (this._status !== PENDING) return
    this._status = FULFILLED
    this._value = val
  }
  // 添加 reject
  _reject(err) {
    if (this._status !== PENDING) return
    this._status = REJECTED
    this._value = err
  }
}
```

## 第三步：实现 `then` 方法

`then` 方法支持链式调用，可以用两个数组来实现回调队列

```js
const PENDING = 'PENDING'
const FULFILLED = 'FULFILLED'
const REJECTED = 'REJECTED'

class MyPromise {
  constructor(handle) {
    if (typeof handle !== 'function') {
      throw new Error('MyPromise must accept a function as parameter')
    }
    // 添加状态
    this._status = PENDING
    // 添加值
    this._value = undefined
    // 添加成功回调函数队列
    this._fulfilledQueues = []
    // 添加失败回调函数队列
    this._rejectedQueues = []
    // 执行 handle
    try {
      handle(this._resolve.bind(this), this._reject.bind(this))
    } catch (err) {
      this._reject(err)
    }
  }
  // 添加 resolve
  _resolve(val) {
    if (this._status !== PENDING) return
    this._status = FULFILLED
    this._value = val
  }
  // 添加 reject
  _reject(err) {
    if (this._status !== PENDING) return
    this._status = REJECTED
    this._value = err
  }
  // 添加 then
  then(onFulfilled, onRejected) {
    const {_value, _status} = this
    return new MyPromise((onFulfilledNext, onRejectedNext) => {
      // 成功时执行的函数
      let fulfilled = value => {
        try {
          if (typeof onFulfilled !== 'function') {
            onFulfilledNext(value)
          } else {
            let res = onFulfilled(value)
            if (res instanceof MyPromise) {
              res.then(onFulfilledNext, onRejectedNext)
            } else {
              onFulfilledNext(next)
            }
          }
        } catch (err) {
          onRejectedNext(err)
        }
      }
      
      switch (_status) {
        case PENDING:
          this._fulfilledQueues.push(onFulfilled)
          this._rejectedQueues.push(onRejected)
          break
        case FULFILLED:
          onFulfilled(_value)
          break
        case REJECTED:
          onRejected(_value)
          break
    })
    }
  }
}
```

# call apply bind

`call` `apply` 和 `bind` 主要跟[[JavaScript Functions|函数]]中的 `this` 和 `arguments` 有关，原生的 `call` `apply` `bind` 提供了除了直接调用以外的其他函数调用方式。

## 模拟实现的思路

我们有一个对象和一个函数：

```js
var foo = {
    value: 1
};

function bar() {
  console.log(this.value);
}

bar.call(foo); // 1
```

如果我们把 `bar` 变成 `foo` 的一个属性，那么就可以不使用 `call` 了，`this` 会被隐式地传递过去：

```js
var foo = {
  value: 1,
  bar: function() {
    console.log(this.value)
  }
};

foo.bar(); // 1
```

可以借鉴这样的思路，我们先将函数加到对象的属性上，调用之后再删除掉即可：

```js
// 第一步
foo.fn = bar
// 第二步
foo.fn()
// 第三步
delete foo.fn
```

## 开始实现

### 第一步：传递 this

```js
Function.prototype.mayCall = function(context) {
  context.fn = this
  context.fn()
  delete context.fn
}
```

### 第二步：传递 arguments

```js
Function.prototype.myCall = function(context, ...args) {
  context.fn = this
  context.fn(...args)
  delete context.fn
}
```

### 第三步：解决其他问题

```js
Function.prototype.myCall = function(context, ...args) {
  // call 可能传入原始类型，bug 是 falsy 都会变成 window
  context = context ? Object(context) : window  
  context.fn = this
  // 处理函数的返回值
  const result = context.fn(...args)
  // 这里不需要区分有没有 args，因为如果没有传值，他的值将会是空数组 []
  // 你仍然可以使用 ...[] 来展开
  delete context.fn
  return result
}
```

按照同样的思路我们也可以模拟 apply：

```js
Function.prototype.myApply = function(context, args) {
  context = context ? Object(context) : window
  context.fn = this
  let result
  if (!args) {
    result = context.fn()
  } else {
    result = context.fn(...args)
  }
  // 注意这里要区分的原因是，如果没有传入 args，那么他的值就是 undefined
  // 你无法对 undefined 进行展开，而且他的报错是：
  // VM8558:1 Uncaught TypeError: context.fn is not iterable (cannot read property Symbol(Symbol.iterator))
  // 而不是 undefined is not iterable
  delete context.fn
  return result
}
```

也可以用 ES3 的写法来实现：

```js
Function.prototype.myCall = function(context) {
  context = context ? Object(context) : window
  context.fn = this
  var args = []
  for (var i = 1; i < arguments.length; i++) {
    args.push('arguments[' + i + ']')
  }
  var result = eval('context.fn(' + args + ')')
  // eval 表示把里面的字符串当做 JS 代码来执行
  // 这句话相当于执行 context.fn( arguments[1], arguments[2], ...)
  delete context.fn
  return result
}

Function.prototype.myApply = function(context, arr) {
  context = context ? Object(context) : window
  context.fn = this
  var result
  if (!arr) {
    result = context.fn()
  } else {
    var args = []
    for (var i = 0; i < arr.length; i++) {
      args.push('arr[' + i + ']')
    }
    result = eval('context.fn(' + args + ')')
  }
  delete context.fn
  return result
}
```

## bind

### 一个简单实现

```js
Function.prototype.myBind = function(context, ...args) {
  const self = this // 存下这个函数
  return function(...newArgs) {
    return self.apply(context, args.concat(newArgs)) // 支持柯里化
  }
}
```

### 源码分析

源于 MDN 上提供的 polyfill

```js
if (!Function.proptotype.bind) {
  Function.prototype.bind = function(oThis) {
    if (typeof this !== 'function') {
      throw new Error('Function.prototype.bind - what is trying to be bound is not callable')
    }
    // 因为 arguments 是伪数组，他没有 slice 方法
    var aArgs = Array.prototype.slice.call(arguments, 1),
        fToBind = this,
        fNOP = function() {},
        fBound = function() {
          // 这是考虑到了要使用 new 来调用 bind 的返回值的情况
          // 如果使用 new 调用，new 具有最高优先级，函数的 this 由 new 确定，而不是 bind
          return fToBind.apply(this instanceof fNOP
                 ? this
                 : oThis,
          // 柯里化
                 aArgs.concat(Array.prototype.slice.call(arguments)))
        }
    // 下面这两句相当于 fBound.prototype = Object.create(this.prototype)，是比较古老的写法
    // 是为了让 this instanceof fNOP 变得有意义
    // 这样，如果使用 new 运算符
    // 那么 fBound 中的 this instanceof fBound === true
    // 同时也有 this instanceof fNOP === true
    if (this.prototype) {
      fNOP.prototype = this.prototype
    }
    fBound.prototype = new fNOP()
    return fBound
  }
}
```

# new

`new` 是创建一个新[[JavaScript Objects|对象]]的方法，原生的 `new` 使得我们使用某个构造函数创建一个对象实例，这是面向对象编程中很有用的一种模式。在 `new` 的时候，做了这样几件事情：

1. 自动创建空对象
2. 自动为空对象关联原型，原型的地址为 `构造函数.prototype`
3. 自动将空对象作为 `this` 关键字运行构造函数
4. 自动 `return this`（也就是说可以接着写 `new X().getName()`）

```js
function myNew(fun, ...args){
    const newObj = Object.create(fun.prototype)
    // 相当于
    // let newObj = {}
    // newObj.__proto__ = fun.prototype
    const result = fun.apply(newObj, args)
    // 原版的 new 中，如果构造函数返回一个对象，则 new 也返回一个对象；如果构造函数返回一个简单类型，则 new 返回刚刚创建的新对象
    return typeof result === 'object' ? result : newObj
}
```

