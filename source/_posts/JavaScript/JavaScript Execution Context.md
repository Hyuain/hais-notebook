---
title: JavaScript Execution Context
date: 2020-01-29 17:22:52
categories:
  - - 前端
tags:
  - FX
---

> DEPRECATED: 本文中的概念没有经过多方查证，信息来源有所矛盾，未来可能会重写。

[JavaScript Execution Context](https://www.freecodecamp.org/news/execution-context-how-javascript-works-behind-the-scenes/)

> 当浏览器遇到 `<script>` 标签或包含 JavaScript 的属性（比如 `onclick`）的时候，会把这些 JavaScript 代码交给 JavaScript Engine。JavaScript Engine 随即创建一个执行 JavaScript 代码的环境，被称为执行上下文（Execution Context）。

执行上下文有两种：

- **全局执行上下文（Global Execution Context, GEC）**。基本的、默认的执行上下文，所有不在函数里面的代码都在这里运行。对于所有的 JavaScript 文件，只有产生一个 GEC。
- **函数执行上下文（Function Execution Context, FEC）**。当函数被调用的时候，JavaScript Engine 会在 GEC 中为每一个函数创建一个 FEC。

<!-- more -->

# Creation

EC 会与与一个对象相关联，该对象被称为 Execution Context Object (ECO)。

EC 的创建被分为三个阶段（创建 VO、创建作用域链、设置 `this`），在这三个阶段中，ECO 上面的属性会被逐步定义。

## Creation of the Variable Object (VO)

> 变量对象（Variable Object, VO）是 EC 内部的 Object-Like 容器，他存储了 EC 中的变量和函数定义。

在 GEC 中，每一个用 `var` 声明的变量都会有一个对应的属性在 VO 中（这个属性指向那个变量），并且值为 `undefined`。

同样，每一个函数也会有一个对应的属性在 VO 中（属性指向那个函数），并且函数会被存储在内存中，这意味着函数声明的时候就已经在 VO 中是可访问的了。

*FEC 不会创建 VO（存疑）*，而会创建一个 Array-Like 对象，称为 `argument`，该对象包含了所有需要提供给函数的参数。

这个在执行代码之前的，存储变量和函数声明的过程被称为 **提升（Hoisting）**。

### Hoisting

> JavaScript 中的函数和变量会被提升

#### Function Hoisting

函数提升使得在 JavaScript 中可以先调用函数，再进行定义：

```js
getAge() // 'called'
function getAge() { console.log('called') }
```

#### Variable Hoisting

使用 `var` 声明的变量会被存到当前 EC 的 VO 中，并且初始值为 `undefiend`。因此，与函数不同，如果在赋值之前使用变量，会得到 `undefined`：

```js
console.log(greetings) // condefined
var greetings = 'Hi!'
```

#### Ground Rules of Hoisting

提升只对函数定义有用，对函数表达式无效，比如按照下面的方式定义，函数不会被提升：

```js
getAge() // TypeError: getAge is not a function
var getAge = function() { console.log('called') }
```

使用 `let` 和 `const` 定义的变量不会被提升，在定义他们之前调用，会得到 `ReferenceError`。

## Creation of the Scope Chain

> VO 创建完毕之后，会创建 **作用域链 (Scope Chain)**。作用域决定了一片代码如何被另一片代码访问。

作用域解决了这些问题：

- 一片代码从哪里可以被访问？从哪里不能被访问？
- 什么可以访问他？什么不能访问？

每个 FEC 都会创建自己的作用域。在其中，他定义的函数和变量可以通过 **作用域规则（Scoping）** 来访问。

**词法作用域规则（Lexical Scoping）**：当个函数定义在另一个函数中，里面的函数可以访问定义在外面函数的变量。作用域的概念带来了 **闭包（Closure）** 现象。

**作用域链（Scope Chain）**：JavaScript Engine 从最里面的 EC 的作用域向外查找。

## Setting "this"

> JavaScript 中的 `this` 表明了 EC 属于谁。

### "this" in GEC

GEC 中的 `this` 指向全局对象，即 `window`，因此用 `var` 声明的函数会被绑定到 `window` 上：

```js
var occupation = "Frontend Developer"; 
function addOne(x) { 
  console.log(x + 1) 
}

// 与下面的代码类似
window.occupation = "Frontend Developer"; 
window.addOne = (x) => { 
  console.log(x + 1)
};
```

### "this" in FEC

FEC 中没有创建 `this` 对象，他内部的 `this` 会访问到 **函数定义的环境**。

比如 GEC 中定义的函数中的 `this` 指向 `window`：

```js
var msg = "I will rule the world!"; 
function printMsg() { 
    console.log(this.msg); 
} 
printMsg(); // logs "I will rule the world!" to the console.
```

对象中定义的则会指向对象，需要通过 `theObject.thePropertyOrMethodDefinedInIt` 这样来引用：

```js
var msg = "I will rule the world!"; 
const Victor = {
    msg: "Victor will rule the world!", 
    printMsg() { console.log(this.msg) }, 
}; 
Victor.printMsg(); // logs "Victor will rule the world!" to the console.
const x = Victor.printMsg;
x(); // logs "I will rule the world!" to the console.
```

# Execution

JavaScript Engine 再一次读取当前 EC 的代码，并更新其中的变量在 VO 中的值。然后代码被解析器解析，转移为机器码，然后执行。

## Execution Stack

> **执行栈** 又称 **调用栈（Call Stack）**，用于跟踪代码生命周期中创建的所有的 EC。
>
> JavaScript 是单线程语言，因此他一次只能执行一个任务，由将要被执行的 EC 堆起来的栈，就被称为 **执行栈**。

当代码被加载到浏览器之后，GEC 被作为默认 EC 创建出来，并被放到 ES 的底部。

当函数被调用的时候，会创建一个新的 FEC，并放到 ES 的顶部。

ES 顶部的 EC 被称为活动 EC，他总是会被 JS Engine 最先执行。

当活动 EC 的所有代码都执行完毕，JS Engine 将其出栈，并将栈顶指向下一个 EC。

### Example

```js
var name = "Victor";
function first() {
  var a = "Hi!";
  second();
  console.log(`${a} ${name}`);
}
function second() {
  var b = "Hey!";
  third();
  console.log(`${b} ${name}`);
}
function third() {
  var c = "Hello!";
  console.log(`${c} ${name}`);
}
first();
```

最开始，会创建一个 GEC 放到栈底：

```text
[GEC]
```

`name` `first` `second` `third` 都会放到 GEC 的 VO 中。

当 JS Engine 遇到 `first` 函数调用的时候，会创建一个新的 FEC，并推入栈中：

```text
[FEC: frist]
[GEC]
```

`a` 保存在 FEC 中。然后 `second` 函数被调用，`first` 函数的执行会被暂停，直到 `second` 执行完毕，然后一个新的 FEC 又被推入栈中：

```text
[FEC: second]
[FEC: frist]
[GEC]
```

`b` 保存在 `second` 的 FEC 中。然后 `third` 被调用，又推一个 FEC 进入 ES：

```text
[FEC: third]
[FEC: second]
[FEC: frist]
[GEC]
```

 `c` 保存在 `third` 的 FEC 中。

然后 `third` 执行完毕，`return`，他的 FEC 出栈，继续执行 `second`。以此类推，直到所有代码执行完毕，GEC 出栈。

