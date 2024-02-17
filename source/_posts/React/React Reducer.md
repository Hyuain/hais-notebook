---
title: React Reducer
date: 2020-02-02 16:25:06
categories:
  - - 前端
tags:
  - FX
---

当组件变得庞大，可能会有[[useState|多个事件处理函数用不同的方式来 set 某个 State]]，State 的更新逻辑也会变得散乱。

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
