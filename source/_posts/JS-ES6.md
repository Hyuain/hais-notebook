---
title: JavaScript ES6 的新玩意儿
date: 2020-02-28 19:49:48
tags:
  - 入门
categories:
  - [前端, JavaScript, 原生 JavaScript]
---

关于 ES6 新增的一些东西，这里的 ES6 泛指 ES6 及之后出现的各种新语法。

<!-- more -->

# ES6 常用语法

作用域、箭头函数、默认参数、展开运算符、模板字符串、解构赋值、import export、class、for of、Generator、新的数据类型（Symbol、Map、Set）

# Generator

## 生成器执行流程

生成器是一个带星号的“函数”（其实他并不是真正的函数），可以通过 yield 关键字暂停执行和恢复执行

```js
function* gen() {
  console.log('enter')
  let a = yield 1
  let b = yield (function() {
    return 2
  })
  return 3
}

var g = gen()
```

1. 调用 `gen()` 之后，程序会阻塞住，不会执行任何语句
2. 调用 `g.next()` 之后，程序会继续执行，直到遇到 `yield`
3. `next` 方法会返回一个对象，有 `value` 和 `done` 属性，`value` 为 当前 `yield` 后面的结果，`done` 表示是否执行完，遇到 `return` 后，`done` 变为 `true`

## yield*

```js
function* gen1() {
  yield 1
  yield* gen2()
  yield 4
}
function* gen2() {
  yield 2
  yield 3
}

// 会按 1 2 3 4 的顺序执行
```

## 生成器的实现机制：协程

一个线程可以存在多个协程，协程不受操作系统的管理，而是被具体的应用程序代码所控制，因此没有进程的上下文切换的开销，性能高。

一个线程一次只能执行一个协程，可以通过 `yield` 暂停当前协程，并将 JS 线程的控制权转移给另一个协程。

## Generator 与异步

### thunk 函数

thunk 函数即偏函数，核心是接受一定的参数，生产出定制化的函数，然后使用定制化的函数去完成功能，相当于一群函数的抽象和封装

### Generator 与异步

#### thunk

```js
const readFileThunk = (filename) => {
  return (callback) => {
    fs.readFile(filename, callback)
  }
}

const gen = function* () {
  const data1 = yield readFileThunk('001.txt')
  console.log(data1.toString())
  const data2 = yield readFileThunk('002.txt')
  console.log(data2.toString())
}

let g = gen()
// value 值即为 thunk 生成的定制化函数
g.next().value((err, data1) => {
  // 拿到上一次的结果，调用 next，将结果作为参数传入
  g.next(data1).value((err, data2) => {
    g.next(data2)
  })
})

// 可以将上面的代码进行封装，减少嵌套
function run(gen) {
  const next = (err, data) => {
    let res = gen.next(data)
    if (res.done) return
    res.value(next)
  }
  next()
}
run(g)
```

#### Promise

也可以用 Promise 来实现：

```js
const readFilePromise = (filename) => {
  return new Promise((resolve, reject) => {
    fs.readFile(filename, (err, data) => {
      if (err) {
        reject(err)
      } else {
        resolve(data)
      }
    })
  }).then(res => res)
}

const gen = function* () {
  const data1 = yield readFilePromise('001.txt')
  console.log(data1.toString())
  const data2 = yield readFilePromise('002.txt')
  console.log(data2.toString())
}

let g = gen()
function getGenPromise(gem ,data) {
  return gen.next(data).value
}
getGenPromise(g).then(data1 => {
  return getGenPromise(g, data1)
}).then(data2 => {
  return getGenPromise(g, data2)
})
```
