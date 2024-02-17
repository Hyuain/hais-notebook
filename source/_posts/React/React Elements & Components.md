---
title: React Elements & Components
date: 2020-02-02 16:25:06
categories:
  - - 前端
tags:
  - FX
---

# 函数与普通代码的区别

看 [这个例子](https://codesandbox.io/s/spring-waterfall-iyekc)：

```js
const React = window.React;
const ReactDOM = window.ReactDOM;
const root = document.querySelector("#root");

let n = 0;
const App = () =>
  React.createElement("div", { className: "red" }, [
    n,
    React.createElement(
      "button",
      {
        onClick: () => {
          n += 1;
          console.log(n);
          ReactDOM.render(App(), root);
        }
      },
      "+1"
    )
  ]);

// 如果 App 不是函数，就不会更新
ReactDOM.render(App(), root);
```

- 普通代码**立即求值**，读取当前值
- 函数会等调用的时候再求值（**延迟求值**），求值时才会读取 `a` 的最新值

# React Element & Component

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

# Two Types of Components

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
<Welcome name="harvey"/>
```

可以看看 [这个例子](https://codesandbox.io/s/tender-nightingale-eu1ne)
