---
title: JavaScript Closures
date: 2020-01-29 17:22:52
categories:
  - - 前端
tags:
  - FX
---

# Closures

> 函数用到了外部的变量，则函数+这个变量=闭包（闭包维持了这个变量的引用，使得函数可以访问他外部的变量），[[JavaScript Scope|作用域]]遵循就近原则

闭包的作用：**隐藏局部变量，暴露操作函数**。

闭包的缺点：容易内存泄露。注意，虽然闭包并不会造成内存泄露，真实原因是 JS 引擎的实现有问题。

闭包中的变量保存在堆内存，而不是栈内存中，使得他在函数调用结束后仍能被访问，具体参阅 [[JavaScript Memory|JavaScript 内存]]。

# Examples

回调函数

```js
function getCarsByMake(make) {
  // filter 的回调函数引用了外部的 make
  return cars.filter(x => x.make === make)
}
```

保存状态

```js
function makePerson(name) {
  let _name = name
  return {
    setName: (newName) => (_name = newName),
    getName: () => _name,
  }
}
const me = makePerson('Zach')
console.log(me.getName())   // 'Zach'
me.setName('Zach Snoek')
console.log(me.getName())   // 'Zach Snoek'
```

私有方法

```js
function makePerson(name) {
  let _name = name
  function privateSetName(newName) {
    _name = newName
  }
  return {
    setName: (newName) => privateSetName(newName),
    getName: () => _name,
  }
}
```

React Event Handlers

```jsx
function Counter({ initialCount }) {
  const [count, setCount] = React.useState(initialCount);
  return (
    <>
      <button onClick={() => setCount(initialCount)}>Reset</button>
      <button onClick={() => setCount((prevCount) => prevCount - 1)}>
        -
      </button>
      <button onClick={() => setCount((prevCount) => prevCount + 1)}>
        +
      </button>
      <button onClick={() => alert(count)}>Show count</button>
    </>
  );
}

function App() {
  return <Counter initialCount={0} />;
}
```
