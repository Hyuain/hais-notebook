---
title: useCallback
date: 2020-02-02 16:25:06
categories:
  - - 前端
tags:
  - FX
---
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
