---
title: React 开始
date: 2020-02-02 16:25:06
tags:
  - 入门
  - 饥人谷
  - 文档
categories:
  - [前端, JavaScript, 框架, React]
---

React 笔记的开始，主要记录了 React 的安装。

<!-- more -->

# 引入 React

## CDN 引入

```html
<script src="https://cdn.bootcss.com/react/16.10.2/umd/react.development.js"></script>
<script src="https://cdn.bootcss.com/react-dom/16.10.2/umd/react-dom.development.js"></script>
```

cjs 和 umd 的区别？
- cjs 全称是 CommonJS，是 Node.js 支持的模块规范
- umd 是统一模块定义，兼容各种模块规范（包含浏览器）
- 理论上优先使用 umd，同时支持 Node.js 和浏览器
- 最新的模块规范是使用 `import` 和 `export` 关键字

## 通过 webpack 引入

```bash
yarn add react react-dom
```
```js
import React from 'react'
import ReactDOM from 'react-dom'
```

## 使用 create-react-app

# 函数与普通代码的区别

看 [这个例子](https://codesandbox.io/s/spring-waterfall-iyekc)

- 普通代码**立即求值**，读取当前值
- 函数会等调用的时候再求值（**延迟求值**），求值时才会读取 a 的最新值

# 对比 React **元素**和**函数组件**的区别

```js
const App1 = React.createElement('div', null, n) // App1 是一个 React 元素
const App2 = () => React.createElement('div', null, n) // App2 是一个 React 函数组件
// App2 是延迟执行的代码，会在被调用的时候执行（会获取到 n 的最新值）
```

- React 元素
    - `createElement` 的返回值 `element` 可以代表一个 `div`
    - 但是 `element` 不是真正的 DOM 对象，而是一个 **虚拟 DOM** 对象
- () ⇒ React 元素
    - 返回 `element` 的函数，也可以代表一个 `div`
    - 函数可以多次执行，每次获取到最新的虚拟 `div`
    - React 会对比两个虚拟 `div` ，找出不同，局部更新视图
    - 找不同的算法叫做 **DOM Diff 算法**