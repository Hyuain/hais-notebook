---
title: React JSX
date: 2020-02-02 16:30:06
tags:
  - 入门
  - 饥人谷
  - 文档
categories:
  - [前端, JavaScript, 框架, React]
---

JSX 介绍。

<!-- more -->

# 引入 babel-loader

## CDN 引入

每次需要写

```html
<script type="text/babel"></script>
```

## webpack

使用 babel-loader

## create-react-app

# JSX 语法

## 嵌入表达式

可以在大括号中使用任何合法的 JavaScript 表达式。

```jsx harmony
const name = 'Josh Perez';
const element = <h1>Hello, {name}</h1>;
```
```jsx harmony
function formatName(user) {
  return user.firstName + ' ' + user.lastName;
}

const user = {
  firstName: 'Harper',
  lastName: 'Perez'
};

const element = (
  <h1>
    Hello, {formatName(user)}!
  </h1>
);
```

{% note warning %}
使用小括号将多行的 JSX 包裹起来，避免 JS 自动加分号的缺陷
{% endnote %}

## JSX 也是一个表达式

可以返回在 `if` 语句和 `for` 循环中使用，可以传参给变量，可以作为参数接收，可以作为返回值

```jsx harmony
function getGreeting(user) {
  if (user) {
    return <h1>Hello, {formatName(user)}!</h1>;
  }
  return <h1>Hello, Stranger.</h1>;
}
```

## 指定属性

注意如果要使用大括号包裹 JS 表达式，别在大括号外面写引号

```jsx harmony
const element = <img src={user.avatarUrl}></img>;
```

{% note warning %}
注意 `class` 变成了 `className`、`tabindex` 变成了 `tabIndex` 等
{% endnote %}

## 指定子元素

```jsx harmony
const element = (
  <div>
    <h1>Hello!</h1>
    <h2>Good to see you here.</h2>
  </div>
);
```

## JSX 阻止 XSS

```jsx harmony
const title = response.potentiallyMaliciousInput;
// This is safe:
const element = <h1>{title}</h1>;
```

## JSX 代表了什么

下面两个表达方式是一样的：

```jsx harmony
const element = (
  <h1 className="greeting">
    Hello, world!
  </h1>
);
```
```jsx harmony
const element = React.createElement(
  'h1',
  {className: 'greeting'},
  'Hello, world!'
);
```

