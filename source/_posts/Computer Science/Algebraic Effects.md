---
title: Algebraic Effects
date: 2020-02-02 16:25:06
categories:
  - - 前端
tags:
  - CX
---

[[React]]
# Algebraic Effects

> 不同设备的性能和网络状况不同，React 需要具备分离副作用的能力，来保证副作用在不同设备上表现尽可能一致。
>
> 引用自 [Algebraic Effects for the Rest of Us](https://overreacted.io/algebraic-effects-for-the-rest-of-us/) 和 [代数效应与React](https://zhuanlan.zhihu.com/p/169805499)

**代数效应（Algebraic Effects）** 是函数式编程中的一个概念，主要是将副作用从函数调用中分离。这个概念只在一些特定的编程语言中被实现并支持。

考虑一个普通的 `try / catch` 块，当 `name` 为空的时候直接抛出错误：

```js
function getName(user) {
  let name = user.name;
  if (name === null) {
  	throw new Error('A girl has no name');
  }
  return name;
}

function makeFriends(user1, user2) {
  user1.friendNames.push(getName(user2));
  user2.friendNames.push(getName(user1));
}

const arya = { name: null, friendNames: [] };
const gendry = { name: 'Gendry', friendNames: [] };
try {
  makeFriends(arya, gendry);
} catch (err) {
  console.log("Oops, that didn't work out: ", err);
}
```

这个错误会向上冒泡（不管中间嵌套了多少层函数执行），并最终在 `catch` 语句中得到处理。

**但是，执行过程就此终止，所有的临时变量被销毁，执行栈被清空，我们不能再恢复到之前抛出错误的地方继续运行。**

**但是，我们可以通过代数效应做到。**

下面我们假设一种新的语法，该语法实现了代数效应：

```js
function getName(user) {
  let name = user.name;
  if (name === null) {
  	// 1. We perform an effect here
  	name = perform 'ask_name';
  	// 4. ...and end up back here (name is now 'Arya Stark')
  }
  return name;
}

// ...

try {
  makeFriends(arya, gendry);
} handle (effect) {
  // 2. We jump to the handler (like try/catch)
  if (effect === 'ask_name') {
  	// 3. However, we can resume with a value (unlike try/catch!)
  	resume with 'Arya Stark';
  }
}
```

上述代码中我们引入了 `perform` 来执行一个 `effect`，用 `try / handle` 来截获这个 `effect`，然后用 `resume with` 来返回到最开始执行 `effect` 的地方。

对于异步操作，我们也许可以考虑这样的代码：

```js
try {
  makeFriends(arya, gendry);
} handle (effect) {
  if (effect === 'ask_name') {
  	setTimeout(() => {
      resume with 'Arya Stark';
  	}, 1000);
  }
}
```

上述代码在延迟 1000ms 后返回到原来的地方。

代数效应有一些好处：

1. 比起 `try / catch`，他可以返回到原来的地方继续执行；
2. 比起 `async / await`，他不具备传染性，也就是说，他不需要让受到影响的地方也变成 `async`；并且如果 `perform effect` 的地方嵌套得很深，中间层的函数也不需要变成 `async`，他只需要在其上层的某处 `handle` 即可，就像 `try / catch` 一样。

## Algebraic Effects in React

比如 `useState` 等 Hooks，我们不需要关心 Hooks 内部是如何处理的，`useState` 就好像 `perform State()`，React 在我们调用这个 `effect` 之后为我们提供 State。

React Fiber 也可以理解成代数效应思想的一种应用，每个 Fiber 有自己的优先级，可以中断与恢复，恢复之后可以复用之前的中间状态。
