---
title: React
date: 2020-02-02 16:25:06
categories:
  - [前端]
---

.

<!-- more -->

2023.03.02 所有类组件的部分都单独转移到 Class Component 下

# Startup

## 引入 React

### CDN 引入

```html
<script src="https://cdn.bootcss.com/react/16.10.2/umd/react.development.js"></script>
<script src="https://cdn.bootcss.com/react-dom/16.10.2/umd/react-dom.development.js"></script>
```

cjs 和 umd 的区别？
- cjs 全称是 CommonJS，是 Node.js 支持的模块规范
- umd 是统一模块定义，兼容各种模块规范（包含浏览器）
- 理论上优先使用 umd，同时支持 Node.js 和浏览器
- 最新的模块规范是使用 `import` 和 `export` 关键字

### 通过 webpack 引入

```bash
yarn add react react-dom
```
```js
import React from 'react'
import ReactDOM from 'react-dom'
```

### 使用 create-react-app

```bash
yarn create react-app my-app
```

## 函数与普通代码的区别

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

## React **元素**和**函数组件**的区别

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

# UI

## JSX

### 引入 babel-loader

- CDN 引入：`<script type="text/babel"></script>`
- webpack 引入：babel-loader
- create-react-app

### JSX 语法

#### 嵌入表达式

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

#### JSX 也是一个表达式

可以返回在 `if` 语句和 `for` 循环中使用，可以传参给变量，可以作为参数接收，可以作为返回值

```jsx harmony
function getGreeting(user) {
  if (user) {
    return <h1>Hello, {formatName(user)}!</h1>;
  }
  return <h1>Hello, Stranger.</h1>;
}
```

#### 指定属性

注意如果要使用大括号包裹 JS 表达式，别在大括号外面写引号

```jsx harmony
const element = <img src={user.avatarUrl}></img>;
```

{% note warning %}

注意 `class` 变成了 `className`、`tabindex` 变成了 `tabIndex` 等

{% endnote %}

#### 指定子元素

```jsx harmony
const element = (
  <div>
    <h1>Hello!</h1>
    <h2>Good to see you here.</h2>
  </div>
);
```

#### JSX 阻止 XSS

```jsx harmony
const title = response.potentiallyMaliciousInput;
// This is safe:
const element = <h1>{title}</h1>;
```

#### JSX 代表了什么

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

### 条件渲染

可以使用 JavaScript 的 `&&` 来控制是否显示右边：

```jsx
return (
  <li className="item">
    {name} {isPacked && '✔'}
  </li>
);
```

但需要注意， 5 个 falsy 值中，尽管其他的值表示什么也不渲染，但数字 `0` 会被显示为 `0`！

所以需要注意下面这种情况：

```jsx
messageCount && <p>New messages</p>
```

### 渲染列表

从哪里获得 `key`？

- 数据库的数据：可以用数据库的 key 或者 ID。
- 本地生成的数据：可以用自增计数器、`crypto.randomUUID` 或 `uuid` 之类的库生成。

`key` 必须满足：

- 在兄弟中是唯一的。
- Key 不能改变，不能在渲染环节生成 Key。

如果用 `index` 作为 `key`，在列表数据插入、删除或排序的时候可能出现 bug。

如果用 `key={Math.random()}`，那么组件和 DOM 将每次都重新被创建。会比较慢，而且会丢失用户输入的信息。

## Component

### Element & Component

```js
const div = React.createElement('div',...) // React 元素
const Div = () => React.createElement('div',...) // React 组件
```

{% note warning %}

目前而言，

React 中，一个返回 React 元素的 **函数** 就是组件

Vue 中，一个 **构造选项** 就可以表示一个组件

{% endnote %}

### Two Types of Components

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

### React.creatElement

`<div/>` 会被翻译成 `React.createElement('div')`

`<Welcome/>` 会被翻译成 `React.createElement(Welcome)`

`React.createElement` 做了下面这些事情：

- 如果传入一个字符串 `'div'` ，则会创建一个 `div`

- 如果传入一个函数，则会调用该函数，获取其返回值

- 如果传入一个类，则会在前面加类前面加 `new` （执行 constructor），获取一个组件的对象，然后调用对象的 render 方法，获取其返回值

### Props

可以查看 [CodeSandbox 上的这个例子](https://codesandbox.io/s/billowing-wind-d8kzw)：

```jsx
import React from "react";
import { createRoot } from 'react-dom/client'
import "./style.css";

function App() {
  return (
    <div className="App">
      爸爸
      <Son messageForSon="儿子你好" />
    </div>
  );
}

class Son extends React.Component {
  render() {
    return (
      <div className="Son">
        我是儿子，爸爸对我说 {this.props.messageForSon}
        <Grandson messageForGrandson={1 + 1} />
      </div>
    );
  }
}
const Grandson = props => {
  return (
    <div className="Grandson">
      我是孙子，爸爸对我说 {props.messageForGrandson}
    </div>
  );
};

const root = createRoot(document.getElementById("root"))
root.render(<App />)

```

## Composition

React 提供了类似于 Vue slot 的组合模式：

```jsx harmony
function Wrapper(props) {
  return (
    <div className='wrapper'>
      {props.children}
    </div>  
  )
}

function App() {
  return (
    <Wrapper>
      <h1> Welcome </h1>
    </Wrapper>
  )
}
```

同样的，也可以预留很多个洞：

```jsx harmony
function SplitPane(props) {
  return (
    <div className="split-pane">
      <div className="left">
        {props.left}
      </div>
      <div className="right">
        {props.right}
      </div>
    </div>
  )
}

function App() {
  return (
    <SplitPane
      left={<Left/>}
      right={<Right/>}
    />
  )
}
```

## Fragments

类似于 `<template>`，使用 Fragments 可以创建类似的一个包裹器，而不会在 DOM 中添加额外的节点：

```jsx harmony
render() {
  return (
    <React.Fragment>
      <ChildA/>
      <ChildB/>
      <ChildC/>
    </React.Fragment>
  )
}
```

还有一个简写版本，简写版不支持使用 key 或其他属性：

```jsx harmony
render() {
  return (
    <>
      <ChildA/>
      <ChildB/>
      <ChildC/>
    </>
  )
}
```

## Render and Commit

React 处理 UI 有三步：

1. Triggering a render
2. Rendering the component
3. Commiting to the DOM

### Step 1: Trigger a render

两个原因：

1. 组件首次渲染：通过 `createRoot` 和 `render` 方法。

   ```jsx
   import { createRoot } from 'react-dom/client'
   const root = createRoot(document.getElementById('root'))
   root.render(<Image />)
   ```

2. 组件或他的某个祖先 State 改变（通过 set 方法）。

### Step 2: React renders your component

所谓渲染（Rendering）指的就是 React 调用组件（函数）：

- 首次渲染，React 将会调用 root 组件；
- 后续渲染，React 会调用触发此次渲染的组件函数。

这个过程是递归的：如果更新的组件返回了别的组件，React 会在下次渲染那个组件；如果那个组件也返回了别的组件，React 又会在下一次渲染别的组件……

以这段代码举例：

```jsx
export default function Gallery() {
  return (
    <section>
      <h1>Inspiring Sculptures</h1>
      <Image />
      <Image />
      <Image />
    </section>
  );
}

function Image() {
  return (
    <img
      src="https://i.imgur.com/ZF6s192.jpg"
      alt="'Floralis Genérica' by Eduardo Catalano: a gigantic metallic flower sculpture with reflective petals"
    />
  );
}
```

- 首次渲染中，React 会为 `<section>` `<h1> ` `<img>` 创建 DOM 节点；
- 重新渲染时，React 会计算他们的属性，看看谁的属性自上次渲染之后发生了变化，**但是他在进入下一个阶段前，都不会根据这个信息做任何事情**。

注意，对于同样的输入，一个组件应该永远有同样的输出。这可以通过 `<Sctrict Mode>` 来进行检查，在严格模式下， React 会调用一个组件两次来帮助发现问题，严格模式在生产环境下不生效。

### Step 3: React commits changes to the DOM

渲染完成后，React 会修改 DOM：

- 首次渲染时，React 会使用 `appendChild()` 将所有创建的 DOM 节点放到页面上；
- 重新渲染时，React 只会执行在渲染阶段计算出来的少量变动到 DOM 上。

## React Implementation

> Fiber 架构让 React 可以异步进行协调过程，将协调过程分解为更小的、可以独立处理的块。
>
> Fiber 允许 React 随时暂停或重启某个任务。

**协调（Reconciliation）**是 React 根据 state 的变化决定 UI 更新的过程，过去的协调是同步进行的，这将会在处理渲染的过程的时候阻塞主线程。

渲染过程可以看成就是执行组件函数，传统意义上讲，**函数的执行过程将由执行栈控制，每个函数可以看成一个 栈帧（Stack Frame），整个过程是无法暂停或终止的，直到执行栈为空。**这样可能一次性会阻塞主线程地执行大量的更新任务，并且前面的有些函数的执行可能是无效的，因为他的效果被后来的更新给覆盖了。

为了解决一长串同步协调过程带来的性能问题，React 提出了三个概念：

- **Fiber**：Fiber 架构相当于是一个自定义的执行栈，**可以将每一个 Fiber 看成一个虚拟的栈帧（Virtual Stack Frame）**。他让 React 可以将任务划分得更细，并且每个 Fiber 可以相对独立地异步执行。

  ```js
  let firstFiber
  // nextFiber 用来跟踪下一步需要执行的 Fiber
  let nextFiber = firstFiber
  // shouldYield 表示需要暂停
  let shouldYield = false
  // 假设有链表 firstFiber -> firstChild -> sibling
  // 从链表的头结点开始执行 performUnitOfWork
  function performUnitOfWork(nextFiber) {
    // ...
    return nextFiber.next
  }
  function workLoop(deadline) {
    while(nextFiber && !shouldYield) {
      nextFiber = performUnitOfWork(nextFiber)
      shouldYield = deadline.timeRemaining < 1
    }
    requestIdleCallback(workLoop)
  }
  // 假设可以使用浏览器的 requestIdelCallback API 来执行 workLoop
  // 这样就可以在浏览器空闲的时候去执行 workLoop
  requestIdleCallback(workLoop)
  ```

- **Scheduler**：利用浏览器的时间片去异步执行这些 Fiber。虽然浏览器自带了 `requestIdelCallback` API，可以在浏览器空闲的时候执行一些任务。但这个 API 存在浏览器兼容性和不稳定等问题，于是 React 自己实现的一套时间片机制就叫 Scheduler。
- **Lane**：用于管理任务的优先级，让高优先级的任务先执行。

### Structure of Fiber

下面给出的是一些 Fiber 的大致结构：

`type` 和 `key` 与 React Element 中的相同，`key` 可以用于追踪该 Fiber 能否被重用；

`child` 和 `sibling` 分别指向子 Fiber 和兄弟 Fiber；

`return` 表示该 Fiber 处理之后应该返回到哪里，就像执行栈一样，一般来讲需要返回父 Fiber；

`pendingProps` 和 `memorizedProps`，如果 `memorizedProps` 与 `pendingProps` 相同，表示该 Fiber 之前的输出可以被重用；

`pendingWorkPriority` 表示该 Fiber 代表的工作的优先级；

`alternate` 表示改 Fiber 当前的状态；

`output` 通常是渲染函数的执行结果。

### Fiber Tree

**Fiber 架构的核心是 Fiber Tree，他表示一个允许 React 跟踪需要执行的各个小工作的 Virtrual DOM。Fiber Tree 上的每一个 Fiber 表示一个需要被处理的工作，比如更新组件或渲染新的元素。**

组件的 State 发生改变之后（a render is triggered）：

1. React 为这个组件创建一个新的 Fiber，并添加到 Fiber Tree 中；
2. 然后安排处理这个 Fiber，将这个 Fiber 放入一个待执行的队列中；
3. 然后开始处理这个队列，一次一个 Fiber。

# Manage State

```jsx
const [n, setN] = useState(0)
```

可以查看 [CodeSandbox 上的这个例子](https://codesandbox.io/s/silly-diffie-9yk38)。

```jsx
const Grandson = () => {
  const [n, setN] = React.useState(0);
  const [m, setM] = React.useState(0);
  // 相当于
  // const array = React.useState(0)
  // const n = array[0]
  // const setN = array[1]
  // setN 会得到一个新的 n
  return (
    <div className="Grandson">
      孙子 n: {n}
      <button onClick={() => setN(n + 1)}>+1</button>
      m: {m}
      <button onClick={() => setM(m + 1)}>+1</button>
      {/* 函数组件的 setState 不会自动合并，建议分开写 */}
    </div>
  );
};
```

{% note warning %}

注意与 class 组件的 setState 不同，他是 **不能** 只更新对象的某个部分的；

并且，如果对象修改前后的地址不变，则不会触发重新渲染；

可以使用 use-immer 库来简单地写出不更改原有对象和数组的代码

{% endnote %}

## useState

> 为什么 useState 的时候没给名字也不会搞混呢？

以下面这段代码举例：

```jsx
 function RenderFunctionComponent() {
   const [firstName, setFirstName] = useState("Rudi");
   const [lastName, setLastName] = useState("Yardley");
 
   return (
     <Button onClick={() => setFirstName("Fred")}>Fred</Button>
   );
 }
```

首先会初始化两个空数组：`setters` 和 `state`，将 `cursor` 指向 `0`。

第一次渲染时，每次遇到 `useState`，会 push **一个 setter 函数** 进 setters 数组，和一个 **state** 进入 state 数组（这都基于 `cursor` 的位置）：

```
 cusor = 0
 
 const [firstName, setFirstName] = useState("Rudi");
 STATE = ["Rudi"]
 SETTERS = [setFirstName]
 cursor++
 
 const [lastName, setLastName] = useState("Yardley");
 STATE = ["Rudi", "Yardley"]
 SETTERS = [setFirstName, setLastName]
```

之后的每次渲染中，`cursor` 都将重新指向 `0`，并且依次从每个数组中读取数据。

如果 setter 被调用了，他就会将 state 中对应的某一项更新。

### myUseState

```jsx harmony
let _state = []
let index = 0
const myUseState = (initialValue) => {
  const currentIndex = index
  _state[currentIndex] = _state[currentIndex] === undefined ? initialValue : _state[currentIndex]
  const setState = (newValue) => {
    _state[currentIndex] = newValue
    render() // 在这里做一个简化
  }
  index ++
  return [_state[currentIndex], setState]
}

const render = () => {
  index = 0 // 这句话很关键，每次渲染之后 index 变成 0
  ReactDOM.render(<App/>, document.getElementById('root'));
}
```

由于是使用数组来实现 state，导致其对顺序依赖非常大，Hook 在每次渲染中必须以 **完全一样的顺序来调用**，比如 React 中不允许使用这种代码：

```jsx harmony
if (n % 2 === 1) {
  [m, setM] = React.useState(0)
}
```

每个函数组件对应一个 React 节点，每个节点将会保存 state（`memorizedState`） 和 index（链表）

## State as Snapshot

**每次渲染的 state 是固定的，调用 set 函数的时候，使用的还是此次渲染的值。**

```jsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 1);
        setNumber(number + 1);
        setNumber(number + 1);
      }}>+3</button>
    </>
  )
}
```

点击 `+3` 按钮后，`number` 会变成 `1`，而不是 `3`，因为在此次渲染中，`number` 一直是 `0`，上面代码相当于这样：

```jsx
setNumber(0 + 1);
setNumber(0 + 1);
setNumber(0 + 1);
```

## Batching

为了减少无效的重复渲染，React 会在等待所有的事件处理函数结束，才会重渲染并更新 state。这被称为批处理（**batching**）。

React 并不会 batch 多个故意事件（比如点击事件，每个点击事件都会被单独处理）。比如第一次点击按钮的时候禁用了这个按钮，第二次点击就不会生效。

```jsx
setColor('orange');
setColor('pink');
setColor('blue');
```

只有 `setColor('blue')` 生效了。

## State Updating Queue

可以给 setter 传一个函数，这个函数被称为 **Updater Function**。

```jsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(n => n + 1);
        setNumber(n => n + 1);
        setNumber(n => n + 1);
      }}>+3</button>
    </>
  )
}
```

这里给 setter 传了函数 `n => n + 1`：

- React 将此函数入队，等待事件处理函数中别的代码执行完成之后再执行；

- 在下次渲染调用 `useState` 时，React 遍历这个队列并且给出最终更新的 state。

有两种特殊情况（`number` 初始均为 0）：

```jsx
<button onClick={() => {
  setNumber(number + 5); // React 将“把值设置成0+5”入队
  setNumber(n => n + 1); // React 将函数入队
}}>
```

最终结果是 6。

```jsx
<button onClick={() => {
  setNumber(number + 5); // React 将“把值设置成 0 + 5” 入队
  setNumber(n => n + 1); // React 将函数入队
  setNumber(42); // React 将“把值设置成42”入队
}}>
```

最终结果是 42。

当事件处理函数完成，React 会触发重渲染。在重渲染中，React 会处理这个队列。所以 **updater functions 必须是纯函数**。不能在 updater functions 里面 setState 或执行其他副作用。

## Preserving and Resetting State

> React 是如何保存函数组件的 State 的？

- 就像 DOM 和 CSSOM 一样，React 创建了一棵 UI 树。React 会根据这棵 UI 树去渲染 DOM。
- 每个组件都有独立的 State，但是这个 State 并不是存在于组件内部的，而是跟组件在 UI 树上的位置绑定的。
- 当 React 移除某个组件的时候，State 也随之销毁了。
- 同样的组件在同样的位置会有相同的 State（即使看起来是销毁了重建了一个）。
- 不同的组件会重置整个子树的 State。
  - 不要在组件里面定义组件，因为这样做的话，外层组件每次渲染都会创建一个 **不同** 的内层组件，导致内层组件状态丢失。
- 如果想要在组件移除之后仍然保持 State。
  - 一开始就全部渲染出来，通过 CSS 来控制是否显示：适合简单的 UI，渲染超级大的树会有性能问题；
  - 提升 State 到父组件去管理：最通用的方法；
  - 使用除了 State 以外的数据源：`localStorage` 等。

### Same component at the same position preserves state

**同样的组件在 UI 树上的同一个位置将会有相同的 State：**

```jsx
const App = () => {
  const [isFancy, setIsFancy] = useState(false)
  // Counter 里面的 State 将会被保留
  // 因为这里一直有一个 Counter 在同样的位置
  return (<>
  	{isFancy
      ? <Counter isFancy={true} />
      : <Counter isFancy={false} />}
  </>)
}
```

注意是在 **UI 树** 的同一个位置，并不是在 JSX 中的同一个位置（React 并不知道我们在哪里写条件判断，他只关心我们返回的这棵树）！

比如下面这种情况下，State 也是保留的（都是 `<Counter />` 在 `<div>` 的第一个子节点）：

```jsx
export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  if (isFancy) {
    return (
      <div>
        <Counter isFancy={true} />
      </div>
    );
  }
  return (
    <div>
      <Counter isFancy={false} />
      <label>
        <input
          type="checkbox"
          checked={isFancy}
          onChange={e => {
            setIsFancy(e.target.checked)
          }}
        />
        Use fancy styling
      </label>
    </div>
  );
}
```

### Reset state at same position

有两个方法可以重置同样位置的 State。

方法一是将他们在不同位置渲染：

```jsx
export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA &&
        <Counter person="Taylor" />
      }
      {!isPlayerA &&
        <Counter person="Sarah" />
      }
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        Next player!
      </button>
    </div>
  );
}
```

`isPlayer` 为 `true` 时，第一个位置是 `Counter`，第二个位置为空；`isPlayer` 为 `false` 时，第一个位置为空，第二个位置是 `Counter`。

每个 `Counter` 的 State 都会在组件从 DOM 中移除的时候销毁。

**更加普遍是方法是使用 Key 来让 React 区分不同的组件**：

- 默认情况下，React 使用在父节点下的顺序来区分组件；
- Key 会显式地告诉 React 这是一个 *特别* 的组件。

```jsx
export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA ? (
        <Counter key="Taylor" person="Taylor" />
      ) : (
        <Counter key="Sarah" person="Sarah" />
      )}
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        Next player!
      </button>
    </div>
  );
}
```

注意 Keys 不是全局唯一的，他只是指明了组件在父结点下面的位置。

## Reducer

当组件变得庞大，可能会有多个事件处理函数用不同的方式来 set 某个 State，State 的更新逻辑也会变得散乱。

这个时候可以将 State 相关的逻辑抽离出来放到一个单独的函数中，这个函数就称为 **Reducer**。

```jsx
// 创建初始状态
const initial = {
  n: 0
}
// 创建 reducer
const reducer = (state, action) => {
  switch (action.type) {
    case 'added':
      return {...state, n: state.n + action.number}
    case 'squared':
      return {...state, n: state.n * 2}
    default:
      throw new Error('unknown type')
  }
}
const App = () => {
  // 使用 useReducer
  const [state, dispatch] = useReducer(reducer, initial)
  const onClick = () => {
    dispatch({
      // action 对象
      // action 对象可以随便定义，但习惯上会有一个 type
      // type 表示了用户意图
      type:'added', number: 1
    })
  }
  return (
    <>
      <div>{n: state.n}</div>
      <button onClick={onClick}>+1</button>
    </>
  )
}
```

Reduce 的词源实际上是数组的 `reduce()` 方法，该方法使用 *result* 和 *current*，返回 *next result*。这与 React 的 Reduce 有异曲同工之妙。

使用 Reducer 的时候需要注意：

- **Reducers 必须是纯函数**。
  - 与 State Updater Functions 类似，Reducers 是在渲染过程中执行的（Actions 都会入队，下次渲染再执行）。
  - 不能发请求，不能用 setTimeout，也不能有其他副作用。
  - 不能修改原来的对象和数组。
- **每个 Action 都描述了一个用户意图，尽管他可能导致了数据中的多个更改**。
  - 比如用户重置了一个表单，最好是发出一个 `rest_form` action，而不是好几个分开的 `set_field` action，这有利于 Debug。

## Context

Context 类似于 Vue / Angular 的 Provide / Eject，使得数据可以跨层传递，而不必显式地通过组件树逐层传递 props。

像 CSS 属性继承，下面的组件都会得到这个 Context，只能用过上层新的 Context Provider 才能覆盖更上层传下来的值。不同的 Context 不会相互覆盖。

**只是需要将属性传递很多层并不意味着需要使用 Context**，在使用 Context 之前还需要考虑一些替代方法：

- **使用 Props**。这样会比较清晰哪个组件使用了什么数据。

- **提取组件，然后把 JSX 作为 children 传给他们**。这样可以越过中间不需要知道这个数据的层。

  ```jsx
  <Layout posts={posts} />
  // 改为
  <Layout><Posts posts={posts} /></Layout>
  ```

Context 的典型使用场景：主题、账号、路由、状态管理。

第一个例子：

```jsx
// 创建一个 context
const C = createContext(null)

const App = () => {
  const [n, setN] = useState(0)
  return (
    // 圈定作用域并给初始值（初始值通常是一个读接口和写接口）
    <C.Provider value={{n, setN}}>
      <Child/>
    </C.Provider>
  )
}

const Child = () => {
  // 子组件拿到读接口和写接口
  const {n, setN} = useContext(C)
  return (
    // ...
  )
}
```

第二个例子：

```jsx
const LevelContext = createContext(0)

const Page = () => {
  return (
    <Section>
      <Heading>Title</Heading>
      <Section>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Section>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Section>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );

const Section = ({ children }) => {
  const level = useContext(LevelContext);
  return (
    <section className="section">
      <LevelContext.Provider value={level + 1}>
        {children}
      </LevelContext.Provider>
    </section>
  );
}

const Heading = ({ children }) => {
  const level = useContext(LevelContext);
  switch (level) {
    case 0:
      throw Error('Heading must be inside a Section!');
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('Unknown level: ' + level);
  }
}
```

## myRedux

````jsx
// 创建数据初始状态
const store = {
  user: null,
  books: null,
  movies: null
}

// 创建 reducer
const reducer = (state, action) => {
  switch (action.type) {
    case 'setUser':
      return {...state, user: action.user}
    case 'setBooks':
      return {...state, user: action.user}
    case 'setMovies':
      return {...state, user: action.user}
    default:
      return state
  }
}

// 创建 Context
const Context = createContext(null)

const App = () => {

  // 创建对数据的读写 API
  const [state, dispatch] = useReducer(reducer, store)

  return (
    // 将读写 API 放到 Context 里面，并使用 Context.Provider 将 Context 提供给组件
    <Context.Provider value={{state, dispatch}}>
      <User/>
    </Context.Provider>
  )
}

const User = () => {
  const {state, dispatch} = useContext(Context)
  useEffect(() => {
    // 通过 AJAX 获取数据，并 dispatch({type: 'setUser', user})
  },[])
  return (
    <div>
      <h1>个人信息</h1>
      <div>name: {state.name ? state.user.name : ''}</div>
    </div>
  )
}
````

也可以将 `useContext` 封装起来：

```jsx
export function useStore() {
  return useContext(Context)
}
```

# Escape Hatches

## Refs

### Referencing Values

```jsx
const ref = useRef(0)
// 之后通过 ref.current 访问及修改
```

Refs 的修改不会触发组件的重新渲染。可以用 Ref 存储计时器 ID、DOM 元素等不希望改变组件渲染结果（JSX）的东西。

**不要在渲染中访问及修改 Ref**（应该在事件处理函数中使用），因为 Ref 的更新并不会触发重新渲染，使得渲染中使用的 Ref 仍然是旧值。

比如下面代码就是一个不会被更新的 Ref：

```jsx
<button onClick={handleClick}>
  You clicked {countRef.current} times
</button>
```

#### Inside of useRef

React 实际上是基于 `useState` 实现的 `useRef`，就像这样：

```js
// Inside of React
function useRef(initialValue) {
  const [ref, unused] = useState({ current: initialValue })
  return ref
}
```

最终达成的效果就是类似于面向对象中的实例属性，比如 `somethingRef.current` 就像 `this.something`。

### Refs and DOM

```jsx
export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}
```

#### Ref Callback

可以给 Ref 传一个回调函数，React 会在设置 Ref 的时候调用这个回调函数，并传递一个 DOM 节点作为参数；在清除的时候也会调用函数并传递 `null` 作为参数。

这样就可以管理一堆 Ref 列表了。

```jsx
export default function CatFriends() {
  const itemsRef = useRef(null);

  function scrollToId(itemId) {
    const map = getMap();
    const node = map.get(itemId);
    node.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest',
      inline: 'center'
    });
  }

  function getMap() {
    if (!itemsRef.current) {
      // Initialize the Map on first usage.
      itemsRef.current = new Map();
    }
    return itemsRef.current;
  }

  return (
    <>
      <nav>
        <button onClick={() => scrollToId(0)}>
          Tom
        </button>
        <button onClick={() => scrollToId(5)}>
          Maru
        </button>
        <button onClick={() => scrollToId(9)}>
          Jellylorum
        </button>
      </nav>
      <div>
        <ul>
          {catList.map(cat => (
            <li
              key={cat.id}
              ref={(node) => {
                const map = getMap();
                if (node) {
                  map.set(cat.id, node);
                } else {
                  map.delete(cat.id);
                }
              }}
            >
              <img
                src={cat.imageUrl}
                alt={'Cat #' + cat.id}
              />
            </li>
          ))}
        </ul>
      </div>
    </>
  );
}

const catList = [];
for (let i = 0; i < 10; i++) {
  catList.push({
    id: i,
    imageUrl: 'https://placekitten.com/250/200?image=' + i
  });
}
```

#### forwardRef

 `forwardRef` 允许组件接收 ref 并将其向下传递，最终可以使得 ref 指向最底层的 HTML DOM

```jsx harmony
const MyComp = () => {
  // const myRef = React.createRef()
  const myRef = useRef(null)
  
  return (
    <Child ref={myRef}> I am a Button </Child>
  ) imperative
}
const Child = React.forwardRef((props, ref) => (
  <button ref={ref} className="button-wrapper">
    {props.children}
  </button>
))
```

通过这样的方式，我们在 MyComp 中就可以通过 `myRef.current` 获取到原生的 button 了

#### Imperative Handle

`useImperativeHandle` 让我们可以自定义父组件访问到的子组件 Ref 的内容：

```jsx
const MyInput = forwardRef((props, ref) => {
  const realInputRef = useRef(null);
  useImperativeHandle(ref, () => ({
    // Only expose focus and nothing else
    focus() {
      realInputRef.current.focus();
    },
  }));
  return <input {...props} ref={realInputRef} />;
});

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}

```

#### When React Attahces the Refs

如前面所说，一次更新分为渲染（Render）和提交（Commit）两个阶段。

- 渲染过程中一般不访问 Ref（这一点在引用值的时候已经说明了）：
  - 在首次渲染时，DOM 节点没有被创建，因此 `ref.current` 是 `null`；
  - 重新渲染时，DOM 节点并没有被更新，因此获取 Ref 也太早了。
- React 是在提交时设置 `ref.current` 的：
  - 更新 DOM 之前，React 会将受到影响的 `ref.current` 设置为 `null`；
  - 更新 DOM 之后，React 立即将他们设置成对应的 DOM 节点。
- 通常需要在事件处理函数中访问 Refs。
- 如果没有合适的事件处理函数的话，也可以在 Effect 中访问 Refs。

#### flushSync

下面代码中，我们试图增加一个新的 TODO 后，立即滚动到新增的那一项：

```jsx
export default function TodoList() {
  const listRef = useRef(null);
  const [text, setText] = useState('');
  const [todos, setTodos] = useState(
    initialTodos
  );

  function handleAdd() {
    const newTodo = { id: nextId++, text: text };
    setText('');
    setTodos([ ...todos, newTodo]);
    listRef.current.lastChild.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest'
    });
  }

  return (
    <>
      <button onClick={handleAdd}>
        Add
      </button>
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <ul ref={listRef}>
        {todos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </>
  );
}

let nextId = 0;
let initialTodos = [];
for (let i = 0; i < 20; i++) {
  initialTodos.push({
    id: nextId++,
    text: 'Todo #' + (i + 1)
  });
}
```

但是这是做不到的，因为 `setTodos` 并不会立即更新 DOM，因此我们滚动的时候，还拿不到最新的那个节点。

可以通过 `flushSync` 来让 React 在其包裹的代码执行完成之后，**同步地** 更新 DOM：

```jsx
flushSync(() => {
  setTodos([ ...todos, newTodo ])
})
listRef.current.lastChild.scrollIntoView()
```

这样就可以拿到最新的 DOM 了。

## Effects

- Effects 说的是不由某个特定的事件触发，而是由 **渲染本身** 触发的副作用。
- Effects 专门是指用来跟除了 React 以外的系统，比如浏览器 API、第三方组件、网络等进行同步。**如果我们只是基于某些 State 而改变另一些 State，可能不需要使用 Effect。**
- **Effects 在渲染之后执行**，这是一个同步 React 组件和外部系统的好时机。
- 由于闭包，每次执行 Effect 都有自己的 State。
- **每个 Effect 都应该描述了一个独立的同步事件**，不应该把不同的任务写在一个 Effect 中。
- React 通过 `Object.is` 来判断依赖是否发生了变化。

### useEffect

书写 Effect 需要三步：

1. 声明 Effect；
2. 确认依赖，什么时候执行该 Effect：
   1. 不加依赖表示每次渲染都执行，`[]` 表示首次渲染执行（mount）；
   2. 依赖应该只加那些 **Reactive** 值（响应式的），Props、States 以及 **其他所有在组件内部定义的数据** 都是 Reactive（因为他们是在渲染过程中计算出来的，并且属于 React 数据流的一员，是可能改变的）；
   3. `loaction.pathname` 这样的 Mutable 值不能被加入依赖。因为他可能在 React 数据流外部随意改变且不会触发重渲染。同时在渲染时读取 Mutable 数据也破坏了函数组件的纯粹性。最好使用 `useSyncExternalStore` 来订阅外部数据的变化；
   4. `ref.current` 这样的 Mutable 值也不能作为依赖。由 `useRef` 返回的 Ref 对象可以作为依赖，但是其 `current` 值是故意设置为 Mutable 的，他的变化不会触发重渲染。
3. 如果需要，增加清理函数。

```jsx
function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    // 1. 声明 Effect
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
    // 2. 确认依赖，首次渲染及 isPlaying 改变的时候执行
  }, [isPlaying]);

  return <video ref={ref} src={src} loop playsInline />;
}
```

如果在 Effect 中使用连接网络、开启计时器等时，可能需要在组件卸载（unmount）的时候进行清理，否则再次 mount 的时候可能会又开启好多重复的计时器。

**为了帮助及时发现问题，Strict Mode 下，React 会在开发环境中组件 mount 之后立即 remount 一下。**

```jsx
useEffect(() => {
  const connection = createConnection();
  connection.connect();
  // 这个函数会在组件 unmount 的时候执行
  // 这个函数也会在下一次重渲染时 Effect 执行之前执行
  return () => {
    connection.disconnect();
  };
}, []);
```

### Data Fetching in Effects

在 Effects 中获取数据是一种非常常见的方法，但他也有一些缺陷：

- **在服务端渲染时无法使用。**浏览器需要下载所有的 JavaScript 代码再获取数据，比较低效。
- **容易产生“网络瀑布流”。**父组件获取数据，再渲染子组件，子组件再获取数据，串行结构会比较慢。
- **无法预加载或缓存数据。**
- **容易产生诸如竞态条件（Race Conditions）之类的 BUG。**

这其实是在 mount 的时候获取数据的库的通病，通常推荐以下方法解决：

- 使用框架（Next、Gatsby、Remix、Razzle）继承的数据获取机制；
- 使用或搭建一个客户端缓存，比如可以用 React Query、useSWR、React Router 6.4+ 等库。

以下是一个手动封装获取数据方法的简单例子：

```jsx
function SearchReasults({ query }) {
  const [page, setPage] = useState(1)
  const params = new URLSearchParams({ query, page })
  const results = useData(`/api/search?${params}`)
  
  function handleNextPageClick() {
    setPage(page + 1)
  }
}

function useData(url) {
  const [data, setData] = useState(null)
  useEffect(() => {
    let ignore = false
    fetch(url)
      .then(res => res.json())
      .then(json => {
        if (!ignore) {
          setData(json)
        }
      })
    return () => {
      ignore = true
    }
  }, [url])
  return data
}
```

#### Race Conditions

有时候会遇到这样的情况，组件接受一个 `id` 参数，当 `id` 改变的时候，就调用接口获取数据。此时，若接口返回的速度慢于 `id` 改变的速度，**屏幕上获取到的数据就会不停的变化**，最终停止为 **最后返回的数据**（不一定是最后一次 `id` 对应的数据）。

下面这段代码模拟了这种情况：

```jsx
  useEffect(() => {
    const fetchData = async () => {
      setTimeout(async () => {
        const response = await fetch(
          `https://swapi.dev/api/people/${props.id}/`
        );
        const newData = await response.json();
        setFetchedId(props.id);
        setData(newData);
      }, Math.round(Math.random() * 12000));
    };

    fetchData();
  }, [props.id]);
```

此时可以通过加 cleanup 函数的方式来解决。

第一种方法，是添加 `active` 或 `ignore` 标记来忽略掉之前几次的返回：

```jsx
useEffect(() => {
  let active = true;

  const fetchData = async () => {
    setTimeout(async () => {
      const response = await fetch(`https://swapi.dev/api/people/${props.id}/`);
      const newData = await response.json();
      if (active) {
        setFetchedId(props.id);
        setData(newData);
      }
    }, Math.round(Math.random() * 12000));
  };

  fetchData();
  return () => {
    active = false;
  };
}, [props.id]);
```

当 `props.id` 发生变化后，前面几次请求对应的 `active` 会被置为 `false`，并且他们的请求结果会被忽略。

第二种方法是直接中断前面的请求，比如通过 `abort`：

```jsx
useEffect(() => {
  const abortController = new AbortController();

  const fetchData = async () => {
    setTimeout(async () => {
      try {
        const response = await fetch(`https://swapi.dev/api/people/${id}/`, {
          signal: abortController.signal,
        });
        const newData = await response.json();

        setFetchedId(id);
        setData(newData);
      } catch (error) {
        if (error.name === 'AbortError') {
          // Aborting a fetch throws an error
          // So we can't update state afterwards
        }
        // Handle other request errors here
      }
    }, Math.round(Math.random() * 12000));
  };

  fetchData();
  return () => {
    abortController.abort();
  };
}, [id]);
```

### You Might Not Need an Effect

Effect 是用来跟外部系统交互的地方，如果我们不需要同步外部系统，通常是不需要 Effect 的。比如：

- **不需要通过 Effect 来转换数据。**比如想要 filter 一个列表，每次列表变动时我就让他自动执行 filter。然而，**这是低效的**。当我们更新组件的 State，React 会执行渲染函数并计算，然后 commit 到 DOM、更新视图，然后才会执行 Effects。如果 Effect 又要立即更新 State，整个过程都会重新执行一遍。**为了减少不必要的渲染过程，在组件最开头就转换数据，这些转换代码会在 State 改变的时候自动重新执行。**
- **不要用 Effects 来处理用户事件。**因为这样你就会找不到具体是哪里触发的事件了。

下面是一些具体的列子：

**更新一些基于 Props 或 State 的 State**

```jsx
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');

  // 🔴 Avoid: redundant state and unnecessary Effect
  const [fullName, setFullName] = useState('');
  useEffect(() => {
    setFullName(firstName + ' ' + lastName);
  }, [firstName, lastName]);
  // ...
}
```

```jsx
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');
  // ✅ Good: calculated during rendering
  const fullName = firstName + ' ' + lastName;
  // ...
}
```

**缓存一些复杂的计算**

```jsx
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');

  // 🔴 Avoid: redundant state and unnecessary Effect
  const [visibleTodos, setVisibleTodos] = useState([]);
  useEffect(() => {
    setVisibleTodos(getFilteredTodos(todos, filter));
  }, [todos, filter]);

  // ...
}
```

```jsx
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  // ✅ This is fine if getFilteredTodos() is not slow.
  const visibleTodos = getFilteredTodos(todos, filter);
  // ...
}
```

```jsx
// `useMemo` 会让 React 记住其回调的返回结果，并且不会在 `todos` 和 `filter` 没有变化的时候重复执行。
// `useMemo` 包裹的函数将会在 **渲染（rendering）** 时执行，因此需要是纯函数。
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  const visibleTodos = useMemo(() => {
    // ✅ Does not re-run unless todos or filter change
    return getFilteredTodos(todos, filter);
  }, [todos, filter]);
  // ...
}
```

**当 Props 改变的时候重置所有的 State**

```jsx
export default function ProfilePage({ userId }) {
  const [comment, setComment] = useState('');

  // 🔴 Avoid: Resetting state on prop change in an Effect
  useEffect(() => {
    setComment('');
  }, [userId]);
  // ...
}
```

```jsx
export default function ProfilePage({ userId }) {
  return (
    <Profile
      userId={userId}
      key={userId}
    />
  );
}

function Profile({ userId }) {
  // ✅ This and any other state below will reset on key change automatically
  const [comment, setComment] = useState('');
  // ...
}
```

**当某个 Prop 改变的时候改变 State**

```jsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // 🔴 Avoid: Adjusting state on prop change in an Effect
  useEffect(() => {
    setSelection(null);
  }, [items]);
  // ...
}
```

```jsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // Better: Adjust the state while rendering
  const [prevItems, setPrevItems] = useState(items);
  // 因为当 React 遇到下面代码的时候就会在本次组件函数执行完成之后立即重新执行
  // 在这个时候，React 还没有渲染他的子组件，也没有更新 DOM
  // 为了减少级联，React 只允许在渲染时更新自己组件的 State，而不能更新其他组件的 State
  if (items !== prevItems) {
    setPrevItems(items);
    setSelection(null);
  }
  // ...
}
```

```jsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selectedId, setSelectedId] = useState(null);
  // ✅ Best: Calculate everything during rendering
  const selection = items.find(item => item.id === selectedId) ?? null;
  // ...
}
```

**在事件处理函数中共享逻辑**

```jsx
function ProductPage({ product, addToCart }) {
  // 🔴 Avoid: Event-specific logic inside an Effect
  // Effect 只能用于“展示”本身就触发了事件，而不能用于本来就是有清晰事件触发者的情况
  useEffect(() => {
    if (product.isInCart) {
      showNotification(`Added ${product.name} to the shopping cart!`);
    }
  }, [product]);

  function handleBuyClick() {
    addToCart(product);
  }

  function handleCheckoutClick() {
    addToCart(product);
    navigateTo('/checkout');
  }
  // ...
}
```

```jsx
function ProductPage({ product, addToCart }) {
  // ✅ Good: Event-specific logic is called from event handlers
  function buyProduct() {
    addToCart(product);
    showNotification(`Added ${product.name} to the shopping cart!`);
  }

  function handleBuyClick() {
    buyProduct();
  }

  function handleCheckoutClick() {
    buyProduct();
    navigateTo('/checkout');
  }
  // ...
}
```

**发送一个 POST 请求**

```jsx
function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  // ✅ Good: This logic should run because the component was displayed
  useEffect(() => {
    post('/analytics/event', { eventName: 'visit_form' });
  }, []);

  // 🔴 Avoid: Event-specific logic inside an Effect
  const [jsonToSubmit, setJsonToSubmit] = useState(null);
  useEffect(() => {
    if (jsonToSubmit !== null) {
      post('/api/register', jsonToSubmit);
    }
  }, [jsonToSubmit]);

  function handleSubmit(e) {
    e.preventDefault();
    setJsonToSubmit({ firstName, lastName });
  }
  // ...
}
```

```jsx
function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  // ✅ Good: This logic runs because the component was displayed
  useEffect(() => {
    post('/analytics/event', { eventName: 'visit_form' });
  }, []);

  function handleSubmit(e) {
    e.preventDefault();
    // ✅ Good: Event-specific logic is in the event handler
    post('/api/register', { firstName, lastName });
  }
  // ...
}
```

**计算链**

```jsx
function Game() {
  const [card, setCard] = useState(null);
  const [goldCardCount, setGoldCardCount] = useState(0);
  const [round, setRound] = useState(1);
  const [isGameOver, setIsGameOver] = useState(false);

  // 🔴 Avoid: Chains of Effects that adjust the state solely to trigger each other
  useEffect(() => {
    if (card !== null && card.gold) {
      setGoldCardCount(c => c + 1);
    }
  }, [card]);

  useEffect(() => {
    if (goldCardCount > 3) {
      setRound(r => r + 1)
      setGoldCardCount(0);
    }
  }, [goldCardCount]);

  useEffect(() => {
    if (round > 5) {
      setIsGameOver(true);
    }
  }, [round]);

  useEffect(() => {
    alert('Good game!');
  }, [isGameOver]);

  function handlePlaceCard(nextCard) {
    if (isGameOver) {
      throw Error('Game already ended.');
    } else {
      setCard(nextCard);
    }
  }

  // ...
```

```jsx
function Game() {
  const [card, setCard] = useState(null);
  const [goldCardCount, setGoldCardCount] = useState(0);
  const [round, setRound] = useState(1);

  // ✅ Calculate what you can during rendering
  const isGameOver = round > 5;

  function handlePlaceCard(nextCard) {
    if (isGameOver) {
      throw Error('Game already ended.');
    }

    // ✅ Calculate all the next state in the event handler
    setCard(nextCard);
    if (nextCard.gold) {
      if (goldCardCount <= 3) {
        setGoldCardCount(goldCardCount + 1);
      } else {
        setGoldCardCount(0);
        setRound(round + 1);
        if (round === 5) {
          alert('Good game!');
        }
      }
    }
  }

  // ...
```

**应用初始化**

```jsx
function App() {
  // 🔴 Avoid: Effects with logic that should only ever run once
  useEffect(() => {
    loadDataFromLocalStorage();
    checkAuthToken();
  }, []);
  // ...
}
```

```jsx
let didInit = false;

function App() {
  useEffect(() => {
    if (!didInit) {
      didInit = true;
      // ✅ Only runs once per app load
      loadDataFromLocalStorage();
      checkAuthToken();
    }
  }, []);
  // ...
}
```

```jsx
if (typeof window !== 'undefined') { // Check if we're running in the browser.
   // ✅ Only runs once per app load
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {
  // ...
}
```

**通知父组件 State 变化**

```jsx
function Toggle({ onChange }) {
  const [isOn, setIsOn] = useState(false);

  // 🔴 Avoid: The onChange handler runs too late
  useEffect(() => {
    onChange(isOn);
  }, [isOn, onChange])

  function handleClick() {
    setIsOn(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      setIsOn(true);
    } else {
      setIsOn(false);
    }
  }

  // ...
}
```

```jsx
function Toggle({ onChange }) {
  const [isOn, setIsOn] = useState(false);

  function updateToggle(nextIsOn) {
    // ✅ Good: Perform all updates during the event that caused them
    setIsOn(nextIsOn);
    onChange(nextIsOn);
  }

  function handleClick() {
    updateToggle(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      updateToggle(true);
    } else {
      updateToggle(false);
    }
  }

  // ...
}
```

```jsx
// ✅ Also good: the component is fully controlled by its parent
function Toggle({ isOn, onChange }) {
  function handleClick() {
    onChange(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      onChange(true);
    } else {
      onChange(false);
    }
  }

  // ...
}
```

**给父组件传值**

```jsx
function Parent() {
  const [data, setData] = useState(null);
  // ...
  return <Child onFetched={setData} />;
}

function Child({ onFetched }) {
  const data = useSomeAPI();
  // 🔴 Avoid: Passing data to the parent in an Effect
  useEffect(() => {
    if (data) {
      onFetched(data);
    }
  }, [onFetched, data]);
  // ...
}
```

```jsx
function Parent() {
  const data = useSomeAPI();
  // ...
  // ✅ Good: Passing data down to the child
  return <Child data={data} />;
}

function Child({ data }) {
  // ...
}
```

**订阅外部数据**

```jsx
function useOnlineStatus() {
  // Not ideal: Manual store subscription in an Effect
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function updateState() {
      setIsOnline(navigator.onLine);
    }

    updateState();

    window.addEventListener('online', updateState);
    window.addEventListener('offline', updateState);
    return () => {
      window.removeEventListener('online', updateState);
      window.removeEventListener('offline', updateState);
    };
  }, []);
  return isOnline;
}

function ChatIndicator() {
  const isOnline = useOnlineStatus();
  // ...
}
```

```jsx
function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}

function useOnlineStatus() {
  // ✅ Good: Subscribing to an external store with a built-in Hook
  return useSyncExternalStore(
    subscribe, // React won't resubscribe for as long as you pass the same function
    () => navigator.onLine, // How to get the value on the client
    () => true // How to get the value on the server
  );
}

function ChatIndicator() {
  const isOnline = useOnlineStatus();
  // ...
}
```



# Hooks

像 `useState` 这种 `use` 开头的函数被称为 Hook。

Hook 是只在 React 渲染的时候才可用的特殊函数。

Hooks 只能在函数或自定义 Hooks 的顶层调用，不能放在条件、循环或其他嵌套函数中。

- 因为 Hooks 依赖于组件在每次渲染时调用 Hooks 的稳定顺序。
- React 会为每一个组件维护一个 **状态对数组** 和 **当前的状态对序号**（初始为 0），每次调用 `useState` 的时候，序号就会增加一个，这样就知道 `useState` 每个对应的都是谁了。

## useLayoutEffect

useEffect 将会在渲染完成之后被执行，因此有时候可能会出现闪烁，而 useLayoutEffect 将会在渲染完成之前（生成 DOM 之后）被执行

但是因为很多很时候我们不需要直接操作 DOM，不需要改变最后渲染的结果（外观），而且 useLayoutEffect 可能会延迟用户看到渲染完成结果的时间，因此优先使用 useEffect

## useMemo

### React.memo

使用 React.memo 封装函数组件，使得其只在 props 变化的时候执行，但是有时候会出现这样的情况：给子组件添加一个事件监听，并传一个回调函数，因为父组件重新执行的时候，这个函数地址会改变，因此子组件还是会执行，使用 useMemo 解决

### useMemo

```jsx harmony
const App = () => {
  const onClick = useMemo(() => {
    return () => {
      // 这个返回值是才我想要缓存的那个函数，也可以返回一个对象
    }
  }, []) // 同样可以设置依赖
  return (
    <Child onClick={onClick}></Child>
  )
}
```

这个其实很像 Vue 2 的 computed

## useCallback

其实跟 useMemo 一样，只是不是在返回值中写：

```jsx harmony
const App = () => {
  const onClick = useCallback(() => {
    // 这个就是你想缓存的函数
  }, []) // 同样可以设置依赖
  return (
    <Child onClick={onClick}></Child>
  )
}
```

## Custom Hooks

- 自定义 Hooks 里面的代码会在每次重渲染的过程中执行。就像组件函数一样，Hooks 也应该是纯函数，Hooks 实际上就是组件的一部分。
- 由于自定义 Hooks 在每次重渲染的时候都会随着组件函数一起执行，他们总是可以使用到最新的 Props 和 State。

```jsx harmony
const useList = () => {
  const [list, setList] = useState(null)
  useEffect(() => {
    ajax('/list').then(list => {
      setList(list)
    })
  }, [])
  return {
    list,
    setList
  }
}
```

# Algebraic Effects

> 不同设备的性能和网络状况不同，React 需要具备分离副作用的能力，来保证副作用在不同设备上表现尽可能一致。
>
> 引用自 [Algebraic Effects for the Rest of Us](https://overreacted.io/algebraic-effects-for-the-rest-of-us/) 和 [代数效应与React](https://zhuanlan.zhihu.com/p/169805499)

**代数效应（Algebraic Effects）**是函数式编程中的一个概念，主要是将副作用从函数调用中分离。这个概念只在一些特定的编程语言中被实现并支持。

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

# Portals

借助 Portal，可以将子节点渲染到存在于父组件以外的 DOM 节点。

```jsx harmony
ReactDOM.createPortal(child, container)
```

其中，`child` 是任何可以渲染的 React 子元素，比如组件、字符串、fragment，`container` 是一个 DOM 元素

通常我们这样写一个子组件，然后在父组件中调用它：

```jsx harmony
render() {
  return (
    <div>
      {this.props.children}
    </div>
  )
}
```

但是有些时候如果父组件有 `overflow: hidden` `z-index` 等样式，如果子组件是对话框、悬浮卡或者提示框等时，我们需要让子组件跳脱出容器：

```jsx harmony
render() {
  return ReactDOM.createPortal(
    this.props.children,
    domNode
  )
}
```

## 事件处理

尽管 portal 可以被放在 DOM 树的任何位置，但他仍然在 React 树中，且与其在 DOM 树中的位置无关，因此像 context 之类的功能特性均不变。
事件冒泡也是这样，他会在 React 树中冒泡至 React 树的祖先，与 DOM 树无关。

# Profiler

`Profiler` 可以测量 React 多久渲染一次以及每次渲染的开销

```jsx harmony
render(
  <App>
    {/* Profiler 需要两个 Prop，一个是 id(string)，一个是当组件树中提交更新时被 React 调用的回调函数 onRender */}
    <Profiler id="Navigation" onRender={callback}>
      <Navigation {...props}/>
    </Profiler>
    <Profiler id="Main" onRender={callback}>
      <Main {...props}/>    
    </Profiler>
  </App>
)
```

```jsx harmony
function onRenderCallback(
  id, // 发生提交的 Profiler 树的 “id”
  phase, // "mount" （如果组件树刚加载） 或者 "update" （如果它重渲染了）之一
  actualDuration, // 本次更新 committed 花费的渲染时间
  baseDuration, // 估计不使用 memoization 的情况下渲染整颗子树需要的时间
  startTime, // 本次更新中 React 开始渲染的时间
  commitTime, // 本次更新中 React committed 的时间
  interactions // 属于本次更新的 interactions 的集合
) {
  // 合计或记录渲染时间
}
```

# Redux

可以看看 [这个例子](https://codesandbox.io/s/jovial-gates-wbs8e)：

```js
import { createStore } from "redux";

const reducer = function(previousState, action) {
  if (previousState === undefined) {
    previousState = 0;
  }
  switch (action.type) {
    case "INCREMENT":
      return previousState + 1;
    case "DECREMENT":
      return previousState - 1;
    default:
      return previousState;
  }
};

const store = createStore(reducer);

const render = function() {
  document.getElementById("value").innerHTML = store.getState();
};

render();

store.subscribe(render); // 每次 dispatch 就会触发 render

const addOne = function() {
  store.dispatch({ type: "INCREMENT" });
};

const minusOne = function() {
  store.dispatch({ type: "DECREMENT" });
};

document.getElementById("increment").addEventListener("click", addOne);
document.getElementById("decrement").addEventListener("click", minusOne);
document.getElementById("incrementIfOdd").addEventListener("click", () => {
  if (store.getState() % 2 === 1) {
    addOne();
  }
});
document.getElementById("incrementAsync").addEventListener("click", () => {
  setTimeout(addOne, 1000);
});
```

# Class Component

## Creation

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

- 类组件修改 state 的时候会进行 **Shallow Merge**，新旧 state 进行一级合并

- `this.state.n += 1` 无效，UI 不会自动更新，需要用 `setState`
- `setState` 不会马上改变 `state`，是异步更新的，推荐使用 `setState(函数)`
- 不推荐 `this.setState(this.state)`，因为 React 不推荐我们修改旧的 `state`（不可变数据）

```jsx
class Son extends React.Component {
  constructor() {
    super();
    this.state = {
      n: 0,
      m: 0
    };
  }
  addN() {
    this.setState(state => {
      const n = state.n + 1;
      return { n };
    });
    // 用函数写 setState，因为 setState 是异步的，他会等一会儿再改变 state,如果用函数就能很清楚的知道哪个是新的 n，哪个是旧的 n
  }
  addM() {
    this.setState({ m: this.state.m + 1 });
    // 类组件会自动合并第一层属性
  }
  render() {
    return (
      <div className="Son">
        儿子 n: {this.state.n}
        <button onClick={() => this.addN()}>n+1</button>
        m: {this.state.m}
        <button onClick={() => this.addM()}>m+1</button>
        <Grandson />
      </div>
    );
  }
}
```

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

## Lifecycle

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

## Event

可以这样写事件

```jsx harmony
class MyComponent extends React.Component {
  constructor() {
    super()
    this.state = {
      n: 0
    }    
  }  
  // 这样写的 addN 是挂在原型上的  
  addN() {
    this.setState((state) => {
      const n = state.n + 1
      return { n }      
    })
  }
  render() {
    return (
      <div>
        n: {this.state.n}            
        <button onClick={() => this.addN()}> n + 1 </button>
        {/*   这样最安全最好懂，箭头函数的 this 不会变   */}
      </div>
    )
  }  
}
```

我们不能在 JSX 里面这样写：

```jsx harmony
<button onClick={this.addN}> n + 1 </button>
{/*   这里面 addN 的 this 会变成 window   */}
```

因为在点击的时候 React 实际上运行的是 `button.onClick.call(null,event)`，`this` 被 React 改了，当然我们可以通过 `bind` 来绑定 `this`

```jsx harmony
<button onClick={this.addN.bind(this)}> n + 1 </button>
```

也可以给箭头函数取个名字再来调用：

```jsx harmony
class MyComponent extends React.Component {
  constructor() { ... }  
  addN() { ... }
  _addN() {
    () => {
      this.addN()
    }
  }
  render() {
    return (
      <div>
        n: {this.state.n}            
        <button onClick={this._addN}> n + 1 </button>
      </div>
    )
  }  
}
```

为了解决这个问题，我们可以将 `addN` 写在 `constructor` 里面：

```jsx harmony
class MyComponent extends React.Component {
  constructor() {
    super()
    this.state = {
      n: 0
    }
    // 这样 addN 就是挂在每个实例对象上了  
    this.addN = () => {
      this.setState((state) => {
        const n = state.n + 1
        return { n }      
      })
    }
  }
  render() {
    return (
      <div>
        n: {this.state.n}            
        <button onClick={this.addN}> n + 1 </button>
      </div>
    )
  }  
}
```

下面这种写法本质与上面的一样，只是 ES 6 的语法糖：

```jsx harmony
class MyComponent extends React.Component {
  constructor() {
    super()
    this.state = {
      n: 0
    }
  }
  addN = () => {
    this.setState((state) => {
      const n = state.n + 1
      return { n }      
    })
  }
  render() {
    return (
      <div>
        n: {this.state.n}            
        <button onClick={this.addN}> n + 1 </button>
      </div>
    )
  }  
}
```

## Context

```jsx
const MyContext = React.createContext('defaultValue')
class App extends React.Component {
  render() {
    return (
      <MyContext value="someValue">
        <ButtonWrapper/>
      </MyContext>
    )
  }
}
function ButtonWrapper() {
  return (
    <div>
      <Button/>
    </div>
  )
}
class MyButton extends React.Component {
  static contextType = MyContext
  render() {
    return <Button themeColor={this.context}/>
  }
}
```

### React.createContext

```jsx harmony
const MyContext = React.createContext(defaultValue)
```

一旦有一个组件订阅了这个 Context 对象，这个组件会从组件树中寻找最近的匹配的 `Provider` 中读取 context 值；如果没有匹配到，则使用 `defaultValue`。需要注意的是，就算给 Provider 传递的是 `undefined`，`defaultValue` 也不会生效。

### Context.Provider

```jsx harmony
<MyContext.Provider value={/* SomeValue */}>
  <Child/>
</MyContext.Provider>
```

当 Provider 的 `value` 值发生变化时，内部所有的消费组件都会重新渲染，不受制于 `shouldComponentUpdate`，变化与否通过 `Object.is` 判定。

比如在下面这种情况时：

```jsx harmony
class App extends React.component {
  render() {
    return (
      <Provider value={{something: 'something'}}>
        <Child/>
      </Provider>
    )
  }
}
```

一旦 Provider 的父组件 App 进行重新渲染，每次 value 就会被赋值为新的对象，会引起所有下面的 consumer 组件的重新渲染。为了防止这么做，需要使用 state：

```jsx harmony
class App extends React.component {
  constructor(props) {
    super(props)
    this.state = {
      value: {something: 'something'}
    }
  }
  render() {
    return (
      <Provider value={this.state.value}>
        <Child/>
      </Provider>
    )
  }
}
```

### Class.contextType

```jsx harmony
class MyClass extends React.Component {
  componentDidMount() {
    let value = this.context
  }
  componentDidUpdate() {
    let value = this.context
  }
  componentWillUnmount() {
    let value = this.context
  }
  render() {
    let value = this.context
  }
}
MyClass.contextType = MyContext
```

通过挂载在 class 上的 `contextType` 属性获取一个 Context 对象，可以通过 `this.context` 来消费最近 Context 上的那个值，可以在包括 render 在内的任何生命周期中访问到他。

也可以通过 `static` 来初始化 `contextType`：

```jsx harmony
class MyClass extends React.Component {
  static contextType = MyContext
  render() {
    let value = this.context
  }
}
```

### Context.Consumer

```jsx harmony
<MyContext.Consumer>
  {value => /* 基于 context 值进行渲染*/}
</MyContext.Consumer>
```

在函数式组件中，我们需要这样使用 context，中间的部分接受一个 context 值，返回一个 React 节点

### Context.displayName

修改在 React DevTools 中显示的名字

```jsx harmony
const MyContext = React.createContext(/* someValue */)
MyContext.displayName = 'MyDisplayName'

<MyContext.Provider> // 在 DevTools 中显示 "MyDisplayName.Provider"
<MyContext.Consumer> // 在 DevTools 中显示 "MyDisplayName.Consumer"
```



## Refs

### React.createRef

可以这样创建一个属性 `myRef`，然后传递给 DOM 元素 `div`，后续就可以使用 `this.myRef.current` 访问到这个 `div`

```jsx harmony
class MyComp extends React.Component {
  constructor(props) {
    super(props)
    this.myRef = React.createRef()
  }
  render() {
    return (
      <div ref={this.myRef}></div>
    )
  }
}
```

当 Ref 用于普通 HTML 元素时，`this.myRef.current` 为 HTML 元素：

```jsx harmony
class MyComp extends React.Component {
  constructor(props) {
    super(props)
    this.textInput = React.createRef()
  }
  focusEvent = () => {
    this.textInput.current.focus()
  }
  render() {
    return (
      <div ref={this.myRef}>
        <input type="text" ref={this.textInput}/>
        <button onClick={this.focusEvent}>Focus</button>
      </div>
    )
  }
}
```

当 Ref 用于自定义 class 组件时，`this.myRef.current` 为组件的实例：

```jsx harmony
class MyComp extends React.Component {
  constructor(props) {
    super(props)
    this.textInput = React.createRef()
  }
  componentDidMount() {
    // 可以调用到 Child 组件里面的 focusTextInput 方法
    this.textInput.current.focusTextInput()
  }
  render() {
    return (
      <Child ref={this.textInput}/>
    )
  }
}
class Child extends React.Component {
  // ...
}
```

注意，不能在函数组件 **上** 使用 ref（函数组件没有实例，ref.current 没办法指向函数组件），但是可以在函数组件 **里面** 使用 ref。

### 回调 Refs

在这种方式中，传递的不是 `createRef()` 创建 `ref` 属性，而是一个函数。这个函数中接受 React 组件实例或者 HTML DOM 元素作为参数。

```jsx harmony
class MyComp extends React.Component {
  constructor(props) {
    super(props)
    this.textInput = null
  }
  setTextInputRef = element => {
    this.textInput = element
  }
  focusTextInput = () => {
    // 使用原生 DOM API 使得 textInput 获得焦点
    if (this.textInput) this.textInput.focus()
  }
  componentDidMount() {
    this.focusTextInput()
  }
  render() {
    // 给 ref 属性传一个一个回调函数，React 组件挂载式会调用回调函数并传入 element
    return (
      <div>
        <input type="text" ref={this.setTextInputRef}/>
        <button onClick={this.focusTextInput}>Focus</button>
      </div>
    )
  }
}
```

同样的，你可以让 `textInput` 指向一个 class 组件，但仍然不能让其指向一个函数组件（因为函数组件没有实例）；但是你可以给函数组件传递一个回调，并将其赋值给函数组件内部的 `ref` 属性，React 在挂载的时候将调用这个回调，并传入 `element` 作为参数：

```jsx harmony
class MyComp extends React.Component {
  constructor(props) {
    super(props)
    this.textInput = null
  }  
  render() {
    return (
      <div>
        <Child inputRef={el => this.textInput = el}/>
      </div>
    )
  }
}
function Child() {
  return (
    <div>
      <input type="text" ref={props.inputRef}/>
    </div>
  )
}
```

↑ 在 `forWardRef` 出现之前，函数组件需要借用回调函数达到 Ref 转发的目的

### 高阶组件的 Ref 转发

有时候我们会使用高阶组件：

```jsx harmony
function logProps(Component) {
  return class extends React.Component {
    componentDidUpdate(prevProps) {
      console.log('old props: ', prevProps)
      console.log('new props: ', this.props)
    }
    render() {
      return <Component {...this.props}/>
    }
  }
}
class Child extends React.Component {
  // ...
}
export default logProps(Child)
```

需要注意的是，如果对 HOC 添加 ref，该 ref 将会引用到其外层的容器组件，而不是被包裹的组件：

```jsx harmony
import Child from './Child'

class MyComp extends React.Component {
  constructor(props) {
    super(props)
    this.textInput = React.createRef()
  }
  render() {
    return (
      // 按照以前的逻辑，这里的 ref 应该指向组件 Child
      // 但实际上，这里的 ref 指向的并不是 Child，而是他外面的 LogProps
      <Child ref={this.textInput}/>
    )
  }
}
```

因此，我们需要使用 `React.forward` 来将 ref 透传下去（就像之前函数组件使用 `React.forward` 将 ref 透传，指向了最终的 HTML DOM 元素一样），使得 ref 指向原来被包裹的组件：

```jsx harmony
function logProps(Component) {
  class LongProps extends React.Component {
    componentDidUpdate(prevProps) {
      console.log('old props: ', prevProps)
      console.log('new props: ', this.props)
    }
    render() {
      const {forwardedRef, ...rest} = this.props
      return <Component ref={forwardedRef} {...rest}/>
    }
  }
  return React.fowardRef((props, ref) => {
    return <LongProps {...props} forwardedRef={ref}/>  
  })
}
class Child extends React.Component {
  // ...
}
export default logProps(Child)
```



## Render Props

目的：封装组件，提高可复用性，可以这样使用：

```jsx harmony
class Cat extends React.Component {
  render() {
    const mouse = this.props.mouse
    return (
      <img src="/cat.jpg" style={{position: 'abosolute', left: mouse.x, top: mouse.y}}/>
    )
  }
}
class Mouse extends React.Component {
  constructor(props) {
    super(props)
    this.state = {x: 0, y: 0}
  }
  handleMouseMove = (event) => {
    this.setState({
      x: event.clientX,
      y: event.clientY
    })
  }
  render() {
    return (
      <div style={{height: '100%'}} onMouseMove={this.handleMouseMove}>
        {this.props.render(this.state)}
      </div>
    )
  }
}
class MouseTracker extends React.Component {
  render() {
    return (
      <div>
        <h1>滑动鼠标</h1>
        <Mouse render={mouse => (
          <Cat mouse={mouse}/>
        )}/>
      </div>
    )
  }
}
```

也可以创建一个 HOC：

```jsx harmony
function withMouse(Component) {
  return class extends React.Component {
    render() {
      return (
        <Mouse render={mouse => (
          <Component {...this.props} mouse={mouse}/>
        )}/>
      ) 
    }
  }
}
```

类似的功能也可以通过在 `<Mouse>` 中使用 `props.children`，然后直接使用类似于这样的写法：

```jsx harmony
<Mouse>
  {mouse => (
    <Cat mouse={mouse}/>
  )}
</Mouse>
```

需要注意的是，使用 render prop 会导致 `React.PureComponent` 失效，因为外层组件更新的时候，render prop 的函数总是新的，除非你把它写成一个实例方法。

## Higher-Order Component

React 中经常会使用高阶组件（HOC, Higher-Order Component）来复用组件逻辑，它是一种设计模式。
具体来说，高阶组件是参数为组件，返回值为新组建的函数：

```jsx harmony
const EnhancedComponent = higherOrderComponent(WrappedComponent)
```

### HOC 的使用场景

#### 修改 props

```jsx harmony
function enhance(WrappedComponent) {
  return class EnhancedComponent extends React.Component {
    render() {
      let props = {
        ...this.props,
        // 增加 message 这个 Prop
        message: 'Hello'
      }
      return <WrappedComponent {...props}/>
    }
  }
}
```

#### 渲染劫持

```jsx harmony
function enhance(WrappedComponent) {
  return class EnhancedComponent extends React.Component {
    render() {
      if (!this.props.data) {
        return <div>loading...</div>
      }
      return <WrappedComponent {...this.props}/>
    }
  }
}
```

### HOC 的例子

#### React Redux

通过 HOC 监听 redux store，然后把下级组件需要的 state、action creator 绑定到 WrappedComponent 的 props 上

#### logger 和 debugger

```jsx harmony
function logProps(WrappedComponent) {
  return class extends React.Component {
    componentWillReceiveProps(nextProps) {
      console.log(`WrappedComponent: ${WrappedComponent.displayName}, Current props: `, this.props)
      console.log(`WrappedComponent: ${WrappedComponent.displayName}, Next props: `, nextProps)
    }
    render() {
      return <WrappedComponent {...this.props}/>
    }
  }
}
```

#### 页面权限管理

通过 HOC 对组件进行包裹，当用户跳转到其他页面的时候，检查用户是否含有对应的权限，如果有的话，渲染页面，如果没有的话，跳转到其他页面

## Reconciliation

Reconciliation 直译为协调，即 React 的渲染机制，他有以下几步：

1. props 或 state 改变
2. render 函数返回不同的元素树（虚拟 DOM）
3. 新旧 DOM 对比（vDOM Diff）
4. 针对差异的地方进行更新
5. 渲染为真实的 DOM 树

### DOM Diff

#### 设计思想

1. 永远只比较同层的节点，不会跨层级比较
2. 不同的两个节点产生不同的树（两个类型不同的节点直接用新的全部替代旧的，包括其后代）
3. 通过 key 判断哪些元素是相同的（比如列表如果没有 key，从头部插入元素会导致列表全部更新），因此 key 需要在列表中保持唯一（不需要全局唯一）

#### 比较流程

- 若元素类型不相同：直接用新的树替换掉原来的树
- 若元素类型相同：
  - 若都是 DOM 节点：更新 DOM 属性，比如 `style`、`title` 等，再向下递归找
  - 若都是组件节点：组件实例保持不变，更新 Props

### 如何减少 Diff 过程

> 利用 `shouldComponentUpdate`

默认的 `shouldComponentUpdate` 会在 props 或 state 发生变化的时候返回 true，表示组件会重新渲染，然后调用 render 函数，进行 vDOM Diff；相对的，我们也可以通过控制它的返回值来控制是否发生 vDOM Diff

### 浅比较与深比较

如果不想使用 `shouldComponentUpdate` 来一个一个检查，可以使用 `pureComponent` 对所有 `props` 和 `state` 进行 **浅比较**：

```jsx harmony
class MyComp extends React.PureComponent {
  // ...
}
```

对于函数组件，可以使用 `React.memo`：

```jsx harmony
function Component() { }
export default React.memo(Component)
```

{% note warning %}

浅比较：

1. 首先会使用等价于 `Object.is` 的方法进行判断，因此若引用类型的地址相同则判定为没有发生变化；
2. 然后对于地址不同的，会进行 **一层** key 和 value 的比较

{% endnote %}

因此，如果数据结构比较复杂，浅比较会损失掉很多信息，比如说通过 `push` 方法改变数组的值，浅比较将不能发现变化。

```jsx harmony
handleClick() {
// 这部分代码很糟，而且还有 bug
  const words = this.state.words;
  words.push('marklar');
  this.setState({words: words});
}
```

这个时候应该使用不可变数据，尽量避免修改正在用于 props 或 state 的值，而是创建一个新的值，去覆盖原来的值：

```jsx harmony
handleClick() {
  this.setState(state => ({
    words: state.words.concat(['marklar'])
  }))
}

// 或者使用 ES6 里的扩展运算符
handleClick() {
  this.setState(state => ({
    words: [...state.words, 'marklar']
  }))
}
```

对于对象，我们可以用 `Object.assign`

```jsx harmony
function updateColorMap(colorMap) {
  return Object.assign({}, colorMap, {right: 'blue'})
}

// 或者使用扩展运算符
function updateColorMap(colorMap) {
  return {...colorMap, right: 'blue'}
}
```

需要注意的是，尽管扩展运算符和 `Object.assign` 创建了新的引用，但是他们仍然是 **浅拷贝**，这并不冲突

### immutable 数据结构

immutable 的意义：浅比较缺点很明显，深比较有时候又比较浪费性能

简单来说：

1. **节省性能**：immutable 内部采用多叉树结构，如果它里面有节点被改变，那么则更新 **这个节点** 和他有关的所有 **上级节点**
2. **返回一个新的引用**，即使是浅比较也能感知到数据的变化

#### 一些 immutable API

##### fromJS

将 JS 对象转换为 immutable 对象

```js
import {fromJS} from 'immutable'
const immutableState = fromJS ({
  count: 0
})
```

##### toJS

将 immutable 对象转换为 JS 对象

```js
const jsObj = immutableState.toJS()
```

##### get/getIn

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

##### set

用来给 immutable 对象的属性赋值

```js
let immutableObj = fromJS({a: 1})
immutableObj.set('a', 2)
```

##### merge

新旧数据对比，旧数据中不存在的属性直接添加，存在的属性用新数据覆盖

```js
let immutableObj = fromJS({a: 1})
immutableObj.merge({
  a: 2,
  b: 3
})
```

# Vue & React

共同点：

- 都是对视图的封装，React 是用类和函数表示一个组件，Vue 是用构造选项表示一个组件
- 都提供了 `creatElement` 的 XML 简写，React 是 JSX，Vue 是 template

不同点：

1. 监听数据变化的实现原理不同
   - Vue 通过 getter/setter 和代理等方式进行劫持，能够精确知道数据的变化，不需要特别的优化
   - React 默认是通过比较引用的方式来进行的，可能会造成不必要的 vDOM 重新渲染，需要用诸如 PureComponent/shouldComponentUpdate 等方式来进行优化
2. 数据流不同
   - Vue 在 1.0 的时候 **父子组件** props 可以双向绑定，还有 v-model 可以实现 **组件与 DOM** 双向绑定
   - Vue 2 的时候 **父子组件** 不能双向绑定了（虽然仍然提供了 .sync 语法糖），并且还有 v-model
   - React 中是单向数据流，并且组件与 DOM 之间也需要使用 onChange/setState。
3. HoC 和 mixins
   - Vue 中使用不同功能的组合是通过 mixins 实现的
   - React 中则使用 HoC
4. 组件间的通信
   - 都有 props，也都可以跨层级通信，比如 Vue 的 provide/inject、React 的 context
   - React 本身不支持自定义事件，因此 Vue 里面一般使用事件，React 中一般使用父组件传来的回调函数
5. 更新视图
   - Vue 中一个对象，对应一个虚拟 DOM，当对象的属性改变时，把属性相关的 DOM 节点全部更新
   - React 一个对象，对应一个虚拟 DOM，另一个对象对应另一个虚拟 DOM，对比两个更新，用 DOM Diff 算法找不同，然后局部更新 DOM
6. 写法不同
   - Vue 是 JS in HTML
   - React 是 HTML in JS
7. Vuex 和 Redux 的区别
   - Vue 中的 `$store` 直接注入到了组件实例中，可以直接用 dispatch、commit、mapState、`this.$store` 等；React 中需要用 connect 把 props 和 dispatch、state 连接起来
   - Vue 中可以用 dispatch action、commit mutation；React 中只能使用 dispatch，不能直接操作 reduce
   - Redux 中的使用的是不可变数据，每次都要用新的 State 替换旧的，而 Vue 中是直接修改的
