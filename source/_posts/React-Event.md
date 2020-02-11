---
title: React Event
date: 2020-02-04 09:27:59
tags:
  - 入门
  - 饥人谷
  - 文档
categories:
  - [前端, JavaScript, 框架, React]
---

主要记录了 React 事件的相关东西。

<!-- more -->

# 类组件的事件

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