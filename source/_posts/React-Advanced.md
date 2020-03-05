---
title: React Advanced
date: 2020-02-02 16:54:25
tags:
  - 入门
  - 文档
categories:
  - [前端, JavaScript, 框架, React]
---

主要抄了一下 React 文档。

<!-- more -->

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

## API

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

也可以通过 public class fields 语法中的 `static` 来初始化 `contextType`：

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

# Refs

## 如何使用 ref

### React.createRef

可以这样创建一个属性 myRef，然后传递给 DOM 元素 `div`，后续就可以使用 `this.myRef.current` 访问到这个 `div`

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

注意，不能在函数组件上使用 ref（函数组件没有实例，ref.current 没办法指向函数组件），但是可以在函数组件里面使用 ref。

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

同样的，你可以让 `textInpu` 指向一个 class 组件，但仍然不能让其指向一个函数组件（因为函数组件没有实例）；但是你可以给函数组件传递一个回调，并将其赋值给函数组件内部的 `ref` 属性，React 在挂载的时候将调用这个回调，并传入 `element` 作为参数：

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

## Refs 转发

为了解决上面这个问题，`forwardRef` 出现了，他允许组件接收 ref 并将其向下传递，最终可以使得 ref 指向最底层的 HTML DOM：

```jsx harmony
const MyComp = () => {
  const myRef = React.createRef()
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

需要注意的是，使用 render prop 会导致 `React.PureComponent` 失效，因为外层组件更新的时候，render prop 的函数总是新的，除非你把它写成一个实例方法

# LifeCycle

![React LifeCycle](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/React-LifeCycle.png)

- **`render()`**：当 state 或 props 发生变化时调用，可以通过 `shouldComponentUpdate` 调解调用时机
- **`constructor(props)`**：用于初始化 state 和为事件处理函数绑定实例（`bind(this)`）
- **`componentDidMount()`**：会在组件挂载后（插入 DOM 树中）立即调用
- **`componentDidUpdate(prevProps, prevState, snapshot)`**： 会在更新后会被立即调用。首次渲染不会执行此方法
- **`componentWillUnmount()`**：会在组件卸载及销毁之前直接调用
- **`shouldComponentUpdate()`**：判断 React 组件的输出是否受当前 state 或 props 更改的影响
- **`static getDerivedStateFromProps(props, state)`**：会在调用 render 方法之前调用，并且在初始挂载及后续更新时都会被调用。它应返回一个对象来更新 state，如果返回 null 则不更新任何内容
- **`getSnapshotBeforeUpdate(prevProps, prevState)`**：会在最近一次渲染输出之前调用。它使得组件能在发生更改之前从 DOM 中捕获一些信息（例如，滚动位置）
- **`static getDerivedStateFromError(error)`**：此生命周期会在后代组件抛出错误后被调用。 它将抛出的错误作为参数，并返回一个值以更新 state
- **`componentDidCatch(error, info)`**：此生命周期在后代组件抛出错误后被调用