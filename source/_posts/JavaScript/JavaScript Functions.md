---
title: JavaScript Functions
date: 2020-01-29 17:22:52
categories:
  - - 前端
tags:
  - FX
---

JavaScript 中函数主要用 `function` 关键字或以箭头函数的方式定义，其中箭头函数没有 `this` `arguments` `super`，不能用于 [[JavaScript Generators|Generator]]，不能用作构造函数，因此也没有 `new.target` 关键字。

<!-- more -->

# Define a Function

## 具名函数

```js
function 函数名(形参) { 函数体 }
```

## 匿名函数

```js
let a = function(形参) { 函数体 } // 右边的部分也叫函数表达式
let a = function fn(形参){ 函数体 }
// 如果函数的声明是在 = 右边
// fn 的作用域只能在 = 右边
// 别的地方可以用 fn 这个名字
```

## 使用构造函数

```js
let a = new Function()
```

## 箭头函数

```js
输入参数 => 输出参数
( 输入参数1, 输入参数2 )  => 输出参数
( 输入参数1, 输入参数2 )  => {
  语句1
  语句2
  return 语句
}
x => ({ name: '...' }) // 如果要返回对象，就要加圆括号
```

# 参数

## 参数传递

复制内存图中 stack 里面的东西（简单类型复制值，对象复制地址）

## 形参

实际上就是变量声明： `var x = arguments[0]`

{% note warning %}

形参可多可少

{% endnote %}

```js
function add(x) {
  return x + arguments[1]
}
add(1, 2) // 3

function add(x, y, z) {
  return x + y
}
add(3, 4) // 7
```

# 返回值

- 每个函数都有返回值，默认是 `undefined`
- 只有函数才有返回值

# 调用栈

- JS 在调用函数之前，需要把函数的环境 push 到一个数组（调用栈）里面，等函数执行完之后把环境 pop 出来，然后 return 到之前的环境中，详细可参考 [[JavaScript Execution Context]]。
- 递归很容易把栈压满：爆栈
- 调用栈最长有多少？Chrome 12578；Firefox 26773；Node 12536

# this & arguments

> 关于 this 更多的内容可以看看这篇文章—— [再看 this](https://hais-teatime.com/post/2019-12-24-this/)。

## arguments

调用函数即传入 arguments， arguments 是包含传入参数的伪数组

## this

- `this` 的存在是为了解决函数获取一个对象的引用的问题
- 如果不给任何的条件， `this` 为 `window`
- 如果想要指定 `this`，要用 `fn.call(xxx, 1, 2, 3)` 传入 `this` 和 `arguments`（如果传入的 `this` 不是对象，将会默认封装成对象，除非加上 `'use strict'`，JS的糟粕）

```js
function fn() {
  'use strict'
  console.log('this: ' + this)
}
fn.call(1) // 'this: 1'
fn.call(undefined) // 'this: undefined'
```

### 两种调用函数的方法

#### 隐式传递

```js
person.sayHi()
// 会自动地把 person 传到函数里，作为 this
```

#### 显式传递

```js
person.sayHi.call(person)
// 手动把 person 传到函数里，作为 this
// apply 跟 call 的区别就是后面要加中括号（参数传的是数组）
```

### 绑定 this

```js
function f1(p1, p2){ console.log(this, p1, p2) }
let f3 = f1.bind( {name:'hai'}, 'hi' )
f3()  // 等价于 f1.call( {name:'hai'}, 'hi' )
      // 相当于让 f3 的 this 和 arguments 永远等于这个
```

### 箭头函数

没有 `arguments` 和 `this`，里面的 `this` 就是外面的 `this`，就算加 `call` 也没有

```js
console.log(this) // window
let fn = () => console.log(this)
fn() // window
fn.call( {name:'hai'} ) // window
```

# 立即执行函数

只想要一个局部变量，而不想要一个函数

```js
+/-/1*/! function (){ var a = 2; console.log(a) } ()
// 声明了一个全局函数，立即调用
```
