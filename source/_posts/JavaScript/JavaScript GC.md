---
title: JavaScript GC
date: 2020-01-29 17:22:52
categories:
  - - 前端
tags:
  - FX
---

JavaScript 的 **垃圾回收器（Garbage Collector, GC）** 可能根据 JavaScript Engine 不同而有不同的实现，粗略来说有引用计数法和标记清除法两种算法。

<!-- more -->

# 内存的生命周期

JavaScript 环境中的内存生命周期由以下三部分组成：

1. 内存分配：当我们声明变量、函数、对象的时候，系统会自动分配内存
2. 内存使用：使用变量、函数等
3. 内存回收

# 垃圾回收

## 什么是垃圾

- 没有被引用的，所谓引用就是一个对象拥有访问另一个对象的权限（隐式或显式）
- 引用的几个对象互相组成环

## 垃圾回收算法

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

# 内存泄漏

经验法则是，如果连续五次垃圾回收之后，内存占用一次比一次大，就有内存泄漏

## 常见的内存泄漏

### 意外的全局变量

```js
function foo() {
  bar1 = 'some text' // 实际上声明了全局变量 window.bar1
  this.bar2 = 'some text' // 全局变量 window.bar2
}
```

### 被遗忘的计时器和回调函数

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

### DOM 引用

前端除了 JS 线程以外，还有渲染线程等：

- 只使用 `div.remove` 只是将 `div` 从页面中删掉，但在内存中还在。如果没有其他引用指向这个对象，那么他可能会在下一个垃圾回收周期中被回收掉
- 只使用 `div = null`，而没有 `remove` 的话，`div` 还在 DOM 中，他就不会被回收

此外还有一些特殊情况，比如如果我们引用了一个表格中的 td 元素，一旦在 DOM 中删除了整个表格，我们直观的觉得内存回收应该回收除了被引用的 td 外的其他元素。但是事实上，这个 td 元素是整个表格的一个子元素，并保留对于其父元素的引用。这就会导致对于整个表格，都无法进行内存回收。

为了避免 DOM 泄露：

- 在删除 DOM 元素之前，确保没有变量或属性指向这些元素
- 如果必须保留引用，确保只保留对独立元素的引用，而不是整个 DOM 结构的引用