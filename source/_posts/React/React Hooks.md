---
title: React Hooks
date: 2020-02-02 16:25:06
categories:
  - - 前端
tags:
  - FX
---
[[useState]] [[useMemo]] [[React Reducer]] [[React Context]]

像 `useState` 这种 `use` 开头的函数被称为 Hook。

Hook 是只在 React 渲染的时候才可用的特殊函数。

Hooks 只能在函数或自定义 Hooks 的顶层调用，不能放在条件、循环或其他嵌套函数中。

- 因为 Hooks 依赖于组件在每次渲染时调用 Hooks 的稳定顺序。
- React 会为每一个组件维护一个 **状态对数组** 和 **当前的状态对序号**（初始为 0），每次调用 `useState` 的时候，序号就会增加一个，这样就知道 `useState` 每个对应的都是谁了。

# Custom Hooks

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

# Behind Hooks

