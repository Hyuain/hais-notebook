---
title: JSX
date: 2020-02-02 16:25:06
categories:
  - - 前端
tags:
  - FX
---

[JSX](https://react.dev/learn/writing-markup-with-jsx) 对 [[JavaScript]] 进行了额外的语法扩充，使得可以在 JavaScript 中嵌入类 XML 代码，可以使用 [Babel](https://babeljs.io/docs/babel-plugin-transform-react-jsx) 将 JSX 编译成 JavaScript 代码。

比如下面的 JSX 可以被 Babel 编译成对应的 JavaScript 代码：

```jsx harmony
const element = (
  <h1 className="greeting">
    Hello, world!
  </h1>
);
```

```javascript
const element = React.createElement(
  'h1',
  {className: 'greeting'},
  'Hello, world!'
);
```

# 条件渲染

可以使用 JavaScript 的 `&&` 来控制是否显示右边：

```jsx
return (
  <li className="item">
    {name} {isPacked && '✔'}
  </li>
);
```

> 但需要注意， 5 个 falsy 值中，尽管其他的值表示什么也不渲染，但数字 `0` 会被显示为 `0`！

所以需要注意下面这种情况：

```jsx
messageCount && <p>New messages</p>
```

# 列表渲染

从哪里获得 `key`？

- 数据库的数据：可以用数据库的 key 或者 ID。
- 本地生成的数据：可以用自增计数器、`crypto.randomUUID` 或 `uuid` 之类的库生成。

`key` 必须满足：

- 在兄弟中是唯一的。
- Key 不能改变，不能在渲染环节生成 Key。

如果用 `index` 作为 `key`，在列表数据插入、删除或排序的时候可能出现 bug。

如果用 `key={Math.random()}`，那么组件和 DOM 将每次都重新被创建。会比较慢，而且会丢失用户输入的信息。
