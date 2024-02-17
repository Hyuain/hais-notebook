---
title: useMemo
date: 2020-02-02 16:25:06
categories:
  - - 前端
tags:
  - FX
---


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

