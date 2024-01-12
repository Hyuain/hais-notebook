---
title: JavaScript Quiz
date: 2020-01-29 17:22:52
categories:
  - - 前端
tags:
  - FX
---

一些偏门的 JavaScript 题目。

<!-- more -->

# 面向对象 1

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

# 面向对象 2

```js
var a = {name: 'x'}
a.x = a = {}
console.log(a.x) // undefined

// 代码执行分为两步： 1. 确定 a 是什么；2. 执行（从右向左）
// 执行到 a.x 的时候，a 还是原来的地址，但是他右边的 a 已经是新的地址了
```
