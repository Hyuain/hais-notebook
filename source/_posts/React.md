---
title: React
date: 2020-02-02 16:25:06
tags:
  - 入门
  - 文档
categories:
  - [前端]
---

- React 的安装与使用
- JSX

<!-- more -->

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

# JSX

## 引入 babel-loader

- CDN 引入：`<script type="text/babel"></script>`
- webpack 引入：babel-loader
- create-react-app

## JSX 语法

### 嵌入表达式

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

### JSX 也是一个表达式

可以返回在 `if` 语句和 `for` 循环中使用，可以传参给变量，可以作为参数接收，可以作为返回值

```jsx harmony
function getGreeting(user) {
  if (user) {
    return <h1>Hello, {formatName(user)}!</h1>;
  }
  return <h1>Hello, Stranger.</h1>;
}
```

### 指定属性

注意如果要使用大括号包裹 JS 表达式，别在大括号外面写引号

```jsx harmony
const element = <img src={user.avatarUrl}></img>;
```

{% note warning %}

注意 `class` 变成了 `className`、`tabindex` 变成了 `tabIndex` 等

{% endnote %}

### 指定子元素

```jsx harmony
const element = (
  <div>
    <h1>Hello!</h1>
    <h2>Good to see you here.</h2>
  </div>
);
```

### JSX 阻止 XSS

```jsx harmony
const title = response.potentiallyMaliciousInput;
// This is safe:
const element = <h1>{title}</h1>;
```

### JSX 代表了什么

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

# Component

## 元素（Element）与组件（Component）

```js
const div = React.createElement('div',...) // React 元素
const Div = () => React.createElement('div',...) // React 组件
```

{% note warning %}

目前而言，

React 中，一个返回 React 元素的 **函数** 就是组件

Vue 中，一个 **构造选项** 就可以表示一个组件

{% endnote %}

## 两种组件

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

## React 中的标签会被翻译成什么？

`<div/>` 会被翻译成 `React.createElement('div')`

`<Welcome/>` 会被翻译成 `React.createElement(Welcome)`

### `React.creatElement`

- 如果传入一个字符串 `'div'` ，则会创建一个 `div`
- 如果传入一个函数，则会调用该函数，获取其返回值
- 如果传入一个类，则会在前面加类前面加 `new` （执行 constructor），获取一个组件的对象，然后调用对象的 render 方法，获取其返回值

## 类组件

### 创建类组件

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

### Props

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

#### `componentWillReceiveProps`

会在 Props 变化的时候调用，目前已经不用了，并更名为 `UNSAFE_componentWillReceiveProps`

```js
componentWillReceiveProps(nextProps, nextContext) {
  //...
}
```

#### Props 的作用

- 接受外部的数据：只能读不能写
- 接受外部的函数：在恰当时机调用

### State

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

### React Lifecycle

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

#### `constructor`

##### 用途

- 初始化 props
- 初始化 state，但此时不能调用 `setState`
- 用来写 `bind this`

```jsx harmony
constructor() {
  this.onClick = this.onClick.bind(this)
}
// 也可以用新语法代替
```

#### `shouldComponentUpdate`

##### 用途

- 返回 true 表示不阻止 UI 更新
- 返回 false 表示阻止 UI 更新

```jsx harmony
shouldComponentUpdate(nextProps, nextState)
```

##### `React.PureComponent`

会在 `render` 之前对新旧 state 和 props 进行浅对比（只比较一层），来控制是否 `render`，只要有任何一个 key 的值不同，就会 `render`

#### `render`

用于展示视图，可以用 `<React.Fragment>` 将多个标签括起来

可以在 `render` 中写

- if / else
- map

#### `componentDidMount`

- 在元素插入页面之后执行代码，这些代码通常依赖 DOM
- 同时官方推荐将加载数据的 AJAX 请求写在这里
- 首次渲染 **会** 执行这个钩子

此外，推荐在使用 Ref 之前先赋值为一个 `undefined`

#### `componentDidUpdate`

- 在视图更新后执行代码
- 此处也可以发起 AJAX 请求，通常是用于更新数据
- 首次渲染 **不会** 执行这个钩子
- 在这里 `setState` 可能会引起无限循环，除非用 `if` 进行限制
- `shouldComponentUpdate` 返回 `false` 时不会调用

```jsx harmony
compoentDidUpdate(prevProps, prevState, snapshot)
```

#### `componentWillUnmount`

- 组件将要被移除页面并销毁时，执行代码
- Unmount 过的组件不会再次 Mount
- 通常需要在这里取消监听、计时器、AJAX 请求等

## 函数组件

### 创建函数组件

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

### 函数组件没有 state

使用 `useState`

```jsx harmony
const [x, setX] = React.useState(0)
```

### 函数组件没有生命周期

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

# Props

可以查看 [CodeSandbox 上的这个例子](https://codesandbox.io/s/billowing-wind-d8kzw)：

```jsx
import React from "react";
import ReactDOM from "react-dom";
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

const rootElement = document.getElementById("root");
ReactDOM.render(<App />, rootElement);

```

# State

可以查看 [CodeSandbox 上的这个例子](https://codesandbox.io/s/silly-diffie-9yk38)。

## 类组件

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

## 函数组件

使用 `useState`：

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

### useState

{% note warning %}

注意与 class 组件的 setState 不同，他是 **不能** 只更新对象的某个部分的；

并且，如果对象修改前后的地址不变，则不会触发重新渲染，因此，最好使用函数

{% endnote %}

```jsx harmony
const [n, setN] = useState(0)
```

#### 尝试自己实现一个 useState

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

#### 实现一个贯穿始终的状态

使用 useState 的话每次重新渲染会产生不同的 state，如果非要实现一个贯穿始终状态，除了使用全局变量 `window` 以外，还有这两种方法：

##### 使用 useRef

```jsx harmony
const nRef = React.useRef(0) // {current: 0}
// 之后使用 nRef.current
```

但是修改 `nRef.current` 不会让组件重新渲染，因此页面上的数据不会同步改变（但是 Vue 3 可以）

可以像这样手动让他更新：

```jsx harmony
const update = React.useState(null)[1]
// 这样修改 nRef.current
const onClick = () => {
  nRef.current += 1
  update(nRef.current)
} 
```

##### 使用 useContext

详见 Context 部分

## setState 的注意事项

- `this.state.n += 1` 无效，UI 不会自动更新，需要用 `setState`
- `setState` 不会马上改变 `state`，是异步更新的，推荐使用 `setState(函数)`
- 不推荐 `this.setState(this.state)`，因为 React 不推荐我们修改旧的 `state`（不可变数据）

### 复杂 state

- 类组件的 `setState` 会自动合并第一层，建议使用 `Object.assign` 或者 `...sate`
- 函数组件不会自动合并，建议分开写

# Composition

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

# Context

Context 类似于 Vue 的 provide / eject，使得数据可以跨层传递，而不必显式地通过组件树逐层传递 props。

```jsx harmony
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

除了 Context 之外还有一种避免中间组件显式传递底层组件的各种属性值的方法：直接将底层组件作为属性传递下去。

## 类组件

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

## 函数组件

使用 `useContext`：

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

# Refs

## 类组件

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

## 函数组件

- 可以不使用 `React.creatRef`，而使用 `useRef`
- 可以不使用回调函数手动转发 Ref，而使用 `forwardRef`，他允许组件接收 ref 并将其向下传递，最终可以使得 ref 指向最底层的 HTML DOM

```jsx harmony
const MyComp = () => {
  // const myRef = React.createRef()
  const myRef = useRef(null)
  
  return (
    <Child ref={myRef}> I am a Button </Child>
  )
}
const Child = React.forwardRef((props, ref) => (
  <button ref={ref} className="button-wrapper">
    {props.children}
  </button>
))
```

通过这样的方式，我们在 MyComp 中就可以通过 `myRef.current` 获取到原生的 button 了

## 高阶组件的 Ref 转发

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

# Other Hooks

## useReducer

```jsx harmony
// 创建初始状态
const initial = {
  n: 0
}
// 创建 reducer
const reducer = (state, action) => {
  switch (action.type) {
    case 'add':
      return {...state, n: state.n + action.number}
    case 'multi':
      return {...state, n: state.n * 2}
    default:
      throw new Error('unknown type')
  }
}
const App = () => {
  // 使用 useReducer
  const [state, dispatch] = useReducer(reducer, initial)
  const onClick = () => {
    dispatch({type:'add', number: 1})
  }
  return (
    <>
      <div>{n: state.n}</div>
      <button onClick={onClick}>+1</button>
    </>
  )
}
```

### 使用 useReducer 替代 redux

```jsx harmony
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
```

## useEffect

若同时有多个 useEffect，则他们将会依次执行

### 什么是副作用

对环境的改变就是副作用，比如改变 `document.title`

### 模拟生命周期

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

## useImperativeHandle

相当于 "setRef"，可以定义一个 ref 的封装

```jsx harmony
useImperativeHandle(ref, () => {
  return {
    x: () => {
      realButton.current.remove()
    },
    realButton: realButton
  }
})
```

## 自定义 Hook

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

# Event

## 类组件的事件

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

# Fragments

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

# Higher-Order Component

React 中经常会使用高阶组件（HOC, Higher-Order Component）来复用组件逻辑，它是一种设计模式。
具体来说，高阶组件是参数为组件，返回值为新组建的函数：

```jsx harmony
const EnhancedComponent = higherOrderComponent(WrappedComponent)
```

## HOC 的使用场景

### 修改 props

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

### 渲染劫持

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

## HOC 的例子

### React Redux

通过 HOC 监听 redux store，然后把下级组件需要的 state、action creator 绑定到 WrappedComponent 的 props 上

### logger 和 debugger

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

### 页面权限管理

通过 HOC 对组件进行包裹，当用户跳转到其他页面的时候，检查用户是否含有对应的权限，如果有的话，渲染页面，如果没有的话，跳转到其他页面

# Reconciliation

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

# Render Props

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
