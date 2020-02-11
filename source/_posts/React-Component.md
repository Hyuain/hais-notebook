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

# `React.creatElement`

- 如果传入一个字符串 `'div'` ，则会创建一个 `div`
- 如果传入一个函数，则会调用该函数，获取其返回值
- 如果传入一个类，则会在前面加类前面加 `new` （执行 constructor），获取一个组件的对象，然后调用对象的 render 方法，获取其返回值