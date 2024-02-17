---
title: myRedux
date: 2020-02-02 16:25:06
categories:
  - - 前端
tags:
  - FX
---


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

