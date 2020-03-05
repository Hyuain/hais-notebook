---
title: FAQs：React
date: 2020-02-13 21:43:16
tags:
  - 入门
  - 收集
  - FAQ
categories:
  - [前端, 其他]
---

关于 React 的一些问题与答案。

<!-- more -->

# React 实现组件间通信？

1. 父子组件：通过 props 直接传参（onChange、onClick）
2. 爷孙组件：传两次 props
2. 任意组件：Redux

# DOM Diff

## Reconciliation

Reconciliation 直译为协调，即 React 的渲染机制，他有以下几步：

1. props 或 state 改变
2. render 函数返回不同的元素树（虚拟 DOM）
3. 新旧 DOM 对比（vDOM Diff）
4. 针对差异的地方进行更新
5. 渲染为真实的 DOM 树

## DOM Diff 原理

### 设计思想

1. 永远只比较同层的节点，不会跨层级比较
2. 不同的两个节点产生不同的树（两个类型不同的节点直接用新的全部替代旧的，包括其后代）
3. 通过 key 判断哪些元素是相同的（比如列表如果没有 key，从头部插入元素会导致列表全部更新），因此 key 需要在列表中保持唯一（不需要全局唯一）

### 比较流程

- 若元素类型不相同：直接用新的树替换掉原来的树
- 若元素类型相同：
    - 若都是 DOM 节点：更新 DOM 属性，比如 `style`、`title` 等，再向下递归找
    - 若都是组件节点：组件实例保持不变，更新 Props

## 如何减少 Diff 过程

> 利用 `shouldComponentUpdate`

默认的 `shouldComponentUpdate` 会在 props 或 state 发生变化的时候返回 true，表示组件会重新渲染，然后调用 render 函数，进行 vDOM Diff；相对的，我们也可以通过控制它的返回值来控制是否发生 vDOM Diff

## 浅比较与深比较

- 类组件的 `shouldComponentUpdate` 中可以对 props 和 state 进行浅比较（使用 `pureComponent` API，比较一层 key 和 value，类似于浅拷贝），也可以进行深比较（递归）
- 函数组件使用 `memo` 方法可以进行肩比较，但是只比较了 props：

```js
function Component() { }
export default React.memo(Component)
```

## immutable 数据结构

immutable 的意义：浅比较缺点很明显，深比较有时候又比较浪费性能

简单来说：

1. **节省性能**：immutable 内部采用多叉树结构，如果它里面有节点被改变，那么则更新 **这个节点** 和他有关的所有 **上级节点**
2. **返回一个新的引用**，即使是浅比较也能感知到数据的变化

### 一些 immutable API

#### fromJS

将 JS 对象转换为 immutable 对象

```js
import {fromJS} from 'immutable'
const immutableState = fromJS ({
  count: 0
})
```

#### toJS

将 immutable 对象转换为 JS 对象

```js
const jsObj = immutableState.toJS()
```

#### get/getIn

用来获取 immutable 对象属性

```js
let jsObj = {a: 1}
let res = jsObj.a

let immutableObj = fromJS(jsObj)
let res = immutableObj.get('a')
```
```js
let jsObj = {a:{b: 1}}
let res = jsObj.a.b

let immutableObj = fromJS(jsObj)
let res = immutableObj.getIn(['a', 'b']) // 传入一个数组
```

#### set

用来给 immutable 对象的属性赋值

```js
let immutableObj = fromJS({a: 1})
immutableObj.set('a', 2)
```

#### merge

新旧数据对比，旧数据中不存在的属性直接添加，存在的属性用新数据覆盖

```js
let immutableObj = fromJS({a: 1})
immutableObj.merge({
  a: 2,
  b: 3
})
```