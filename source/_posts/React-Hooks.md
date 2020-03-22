---
title: React Hooks
date: 2020-03-16 10:22:48
tags:
  - 入门
  - 文档
  - 饥人谷
categories:
  - [前端, JavaScript, 框架, React]
---

关于 React Hooks。

<!-- more -->

# useState

{% note warning %}
注意与 class 组件的 setState 不同，他是 **不能** 只更新对象的某个部分的；
并且，如果对象修改前后的地址不变，则不会触发重新渲染，因此，最好使用函数
{% endnote %}

```jsx harmony
const [n, setN] = useState(0)
```

## 尝试自己实现一个 useState

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

## 实现一个贯穿始终的状态

使用 useState 的话每次重新渲染会产生不同的 state，如果非要实现一个贯穿始终状态，除了使用全局变量 `window` 以外，还有这两种方法：

### 使用 useRef

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

### 使用 useContext

详见 React Advanced 中的 Context 部分

# useReducer

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

## 使用 useReducer 替代 redux

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

# useContext

```jsx harmony
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

# useEffect

若同时有多个 useEffect，则他们将会依次执行

## 什么是副作用

对环境的改变就是副作用，比如改变 `document.title`

## 模拟生命周期

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

# useLayoutEffect

useEffect 将会在渲染完成之后被执行，因此有时候可能会出现闪烁，而 useLayoutEffect 将会在渲染完成之前（生成 DOM 之后）被执行

但是因为很多很时候我们不需要直接操作 DOM，不需要改变最后渲染的结果（外观），而且 useLayoutEffect 可能会延迟用户看到渲染完成结果的时间，因此优先使用 useEffect

# useMemo

## React.memo

使用 React.memo 封装函数组件，使得其只在 props 变化的时候执行，但是有时候会出现这样的情况：给子组件添加一个事件监听，并传一个回调函数，因为父组件重新执行的时候，这个函数地址会改变，因此子组件还是会执行，使用 useMemo 解决

## useMemo

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

# useCallback

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

# useRef

可以通过 useRef 获得一个贯穿全局的状态

## forwardRef

由于 props 不包含 ref，所以需要 forwardRef，他实现了 ref 的传递

```jsx harmony
const App = () => {
  const buttonRef = useRef(null)
  return (
    <div>
      <Button ref={buttonRef}></Button>
    </div>
  )
}

const Button = React.forwardRef((props, ref) => {
  return (
    <button ref={ref} {...props}></button>
  )
})
```

# useImperativeHandle

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

# 自定义 Hook

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

# Stale Closure