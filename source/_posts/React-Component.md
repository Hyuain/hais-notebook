---
title: React Component
date: 2020-02-02 16:51:41
tags:
  - 入门
  - 饥人谷
  - 文档
categories:
  - [前端, JavaScript, 框架, React]
---

主要记录了 React 的组件的用法。

<!-- more -->

# 元素（Element）与组件（Component）

```js
const div = React.createElement('div',...) // React 元素
const Div = () => React.createElement('div',...) // React 组件
```

{% note warning %}
目前而言，
React 中，一个返回 React 元素的**函数**就是组件
Vue 中，一个 **构造选项** 就可以表示一个组件
{% endnote %}

# 两种组件

- 函数组件：

```jsx harmony
function Welcome(props) {
  return <h1>Hello,{props.name}</h1> // 会自动变成 React.createElement(...)
}
```

- 类组件：

```jsx harmony
class Welcome extends React.component {
  render() {
    return <h1>Hello,{this.props.name}</h1>
  }
}
```

两者的使用方法都是：

```jsx harmony
<Welcome name="frank"/>
```

可以看看 [这个例子](https://codesandbox.io/s/tender-nightingale-eu1ne)

# React 中的标签会被翻译成什么？

`<div/>` 会被翻译成 `React.createElement('div')`

`<Welcome/>` 会被翻译成 `React.createElement(Welcome)`

## `React.creatElement`

- 如果传入一个字符串 `'div'` ，则会创建一个 `div`
- 如果传入一个函数，则会调用该函数，获取其返回值
- 如果传入一个类，则会在前面加类前面加 `new` （执行 constructor），获取一个组件的对象，然后调用对象的 render 方法，获取其返回值

# 类组件

## 创建类组件

有两种方式创建类组件：

- ES 5 方式（过时）

```jsx harmony
import React from 'react'

const A = React.createClass({
  render() {
    return (
      <div>hi</div>
    )
  }
})

export default A
```

- ES 6 方式（使用 class）

```jsx harmony
import React from 'react'

class B extends React.Component {
  constructor(props) {
    super(props)
  }
  // 其实如果不在 constructor 里面加其他的东西，上面几行是可以省略掉的
  render() {
    return (
      <div>hi</div>
    )
  }
}

export default B
```

## Props

```jsx harmony
class Parent extends React.Component {
  constructor(props) {
    super(props)
    this.state = {name: 'harvey'}
  }
  onClick = () => {}
  render() {
    return (
      <Child name={this.state.name}
             onClick={this.onClick}>hi</Child>
      // 外部数据会被包装为一个对象：
      // {name: 'harvey', onClick:..., children: 'hi'} 
    )
  }
}

class Child extends React.Component {
  constructor(props) {
    super(props) // 这样会将 props 放到 this 上，这就是 props 的初始化
  }
  render() {
    // 这样读取 props
    return (
      <div onClick={this.props.onClick}>
        {this.props.name}
        <div>
          {this.props.children}
        </div>
      </div>
    )
  }
}
```

{% note warning %}
不允许修改 Props
{% endnote %}

### `componentWillReceiveProps`

会在 Props 变化的时候调用，目前已经不用了，并更名为 `UNSAFE_componentWillReceiveProps`

```js
componentWillReceiveProps(nextProps, nextContext) {
  //...
}
```

### Props 的作用

- 接受外部的数据：只能读不能写
- 接受外部的函数：在恰当时机调用

## State

```jsx harmony
class Parent extends React.Component {
  constructor(props) {
    super(props)
    // 初始化 State
    this.state = {x: 1}
  }
  onClick = () => {
    // 修改 State，注意 setState 不会立即修改 State，会在 set 成功之后调用 fn
    // 因此最后 this.state.x 不会 +2
    this.setState({
      x: this.state.x + 1
    }, fn)
    this.setState({
      x: this.state.x + 1
    }, fn)
  }
  onClick2 = () => {
    // 第二种写法，这种情况下 this.state.x 会 +2
    this.setState((prevState) => ({
      x: prevState.x + 1
    }), fn)
    this.setState((prevState) => ({
      x: prevState.x + 1
    }), fn)
  }
  render() {
    return (
      // 读取 State
      <div onClick={this.onClick}>{this.state.x}</div>
    )
  }
}
```

修改 state 的时候会进行 **Shallow Merge**，新旧 state 进行一级合并

## React Lifecycle

![React LifeCycle](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/React-LifeCycle.png)

- **`constructor(props)`**：用于初始化 state 和为事件处理函数绑定实例（`bind(this)`）
- `static getDerivedStateFromProps(props, state)`：会在调用 render 方法之前调用，并且在初始挂载及后续更新时都会被调用。它应返回一个对象来更新 state，如果返回 null 则不更新任何内容
- **`shouldComponentUpdate()`**：判断 React 组件的输出是否受当前 state 或 props 更改的影响
- **`render()`**：当 state 或 props 发生变化时调用，可以通过 `shouldComponentUpdate` 调解调用时机
- `getSnapshotBeforeUpdate(prevProps, prevState)`：会在最近一次渲染输出之前调用。它使得组件能在发生更改之前从 DOM 中捕获一些信息（例如，滚动位置）
- **`componentDidMount()`**：会在组件挂载后（插入 DOM 树中）立即调用
- **`componentDidUpdate(prevProps, prevState, snapshot)`**： 会在更新后会被立即调用。首次渲染不会执行此方法
- **`componentWillUnmount()`**：会在组件卸载及销毁之前直接调用
- `static getDerivedStateFromError(error)`：此生命周期会在后代组件抛出错误后被调用。 它将抛出的错误作为参数，并返回一个值以更新 state
- `componentDidCatch(error, info)`：此生命周期在后代组件抛出错误后被调用

### `constructor`

#### 用途

- 初始化 props
- 初始化 state，但此时不能调用 `setState`
- 用来写 `bind this`
```jsx harmony
constructor() {
  this.onClick = this.onClick.bind(this)
}
// 也可以用新语法代替
```

### `shouldComponentUpdate`

#### 用途

- 返回 true 表示不阻止 UI 更新
- 返回 false 表示阻止 UI 更新

```jsx harmony
shouldComponentUpdate(nextProps, nextState)
```

#### `React.PureComponent`

会在 `render` 之前对新旧 state 和 props 进行浅对比（只比较一层），来控制是否 `render`，只要有任何一个 key 的值不同，就会 `render`

### `render`

用于展示视图，可以用 `<React.Fragment>` 将多个标签括起来

可以在 `render` 中写

- if / else
- map

### `componentDidMount`

- 在元素插入页面之后执行代码，这些代码通常依赖 DOM
- 同时官方推荐将加载数据的 AJAX 请求写在这里
- 首次渲染 **会** 执行这个钩子

此外，推荐在使用 Ref 之前先赋值为一个 `undefined`

### `componentDidUpdate`

- 在视图更新后执行代码
- 此处也可以发起 AJAX 请求，通常是用于更新数据
- 首次渲染 **不会** 执行这个钩子
- 在这里 `setState` 可能会引起无限循环，除非用 `if` 进行限制
- `shouldComponentUpdate` 返回 `false` 时不会调用

```jsx harmony
compoentDidUpdate(prevProps, prevState, snapshot)
```

### `componentWillUnmount`

- 组件将要被移除页面并销毁时，执行代码
- Unmount 过的组件不会再次 Mount
- 通常需要在这里取消监听、计时器、AJAX 请求等

# 函数组件

## 创建函数组件

```jsx harmony
// 箭头函数
const Hello1 = (props) => {
  return <div>{props.message}</div>
}

// 箭头函数缩写
const Hello2 = props => <div>{props.message}</div>

// 普通函数
function Hello3(props) {
  return <div>{props.message}</div>
}
```

## 函数组件没有 state

使用 `useState`

```jsx harmony
const [x, setX] = React.useState(0)
```

## 函数组件没有生命周期

使用 `useEffect`

- 模拟 `componentDidMount`（第一次渲染）
```jsx harmony
React.useEffect(() => {
  // do something
}, [])
```
- 模拟 `componentDidUpdate`（更新时执行）
```jsx harmony
React.useEffect(() => {
  // do something
}, [n])
// 这样的话第一次渲染也会执行，可以使用自定义 Hook 来解决
const useUpdate = (fn, dep) => {
  const [updateCount, setUpdateCount] = React.useState(0)
  React.useEffect(() => {
    setUpdateCount(count => count + 1)
  }, [dep])
  React.useEffect(() => {
    if (updateCount > 1) {
      fn()
    }
  }, [updateCount, fn])
}
useUpdate(() => {
  // do something
}, n)
```
- 模拟 `componentWillUnmount`（将要销毁时执行）
```jsx harmony
React.useEffect(() => {
  return () => {
    // do something
  }
}, [n])
```