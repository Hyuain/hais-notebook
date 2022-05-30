---
title: Vuex
date: 2020-02-01 17:01:32
tags:
  - 入门
  - 文档
categories:
  - [前端, Vue]
---

用了几次 Vuex 之后抄一下文档。

<!-- more -->

可以先看一下 [这个例子](https://codesandbox.io/s/quizzical-kalam-303z8)

# 基本使用

Vuex 的核心就是一个状态仓库，与普通的全局对象有所不同：

1. Vuex 的状态存储是响应式的，store 变化 -> 组件响应式更新
2. 不能直接修改 store 中的状态，只能通过提交 mutation 的方式来改变状态

```js
// 1. 引入 Vuex
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

// 2. 创建一个 store
const store = new Vuex.Store({
  // 定义一个初始的状态
  state: {
    count: 0
  },
  // 定义一些 mutation 来操作状态的变更
  mutation: {
    increment (state) {
      state.count++
    }
  }
})

// 3. 可以通过 store.state 获取对象，或者通过 store.commit 来触发变更
store.commit('increment')

console.log(store.state.count) // -> 1
```

# State

Vuex 使用单一状态树，每个应用仅仅包含一个 store 实例

## 在 Vue 组件中获得 Vuex 状态

### 方法一：通过计算属性

```js
// 创建一个 Counter 组件
const Counter = {
  template: `<div>{{ count }}</div>`,
  computed: {
    count () {
      return store.state.count
    }
  }
}
```

这样每个需要使用 state 的组件中就都需要导入 store

### 方法二：将状态从根组件注入每个子组件

```js
const app = new Vue({
  el: '#app',
  store,
  components: { Counter },
  template: `
    <div class="app">
      <counter/>
    </div>
  `
})
```
这样子组件就可以通过 `this.$store` 访问到
```js
const Counter = {
  template: `<div>{{ count }}</div>`,
  computed: {
    count() {
      return this.$store.state.count  
    }
  }
}
```

### 方法三：`mapState`

```js
import { mapState } from 'vuex'

export default {
  computed: mapState({
    // 可以使用箭头函数
    count: state => state.count,

    // 传字符串参数 'count' 等同于 `state => state.count`
    countAlias: 'count',
    
    // 也可以使用普通函数
    countPlusLocalState (state) {
      return state.count + this.localCount
    }
  })
}
```
若计算属性的名称与子节点名称相同，的也可以给 mapState 传一个数组
```js
computed: mapState([
  // 映射 this.count 为 store.state.count
  'count'
])
```

### 将 mapState 与局部计算属性混用

```js
computed: {
  localComputed () { /* ... */ },
  // 使用对象展开运算符将此对象混入到外部对象中
  ...mapState({
    // ...
  })
}
```

# Getter

从 store 的 state 中派生出一些状态，类似于 store 的计算属性

```js
const store = new Vuex.Store({
  state: {
    todos: [
      { id: 1, text: '...', done: true },
      { id: 2, text: '...', done: false }
    ]
  },
  getters: {
    // 可以传入 state 作为第一个参数
    doneTodos: state => {
      return state.todos.filter(todo => todo.done)
    },
    // 可以传入其他 getter 作为第二个参数
    doneTodosCount: (state, getters) => {
      return getters.doneTodos.length
    },
    // 可以让 getter 返回一个函数来实现给 getter 传参
    getTodoById: (state) => (id) => {
      return state.todo.find(todo => todo.id === id)
    }
    // 到时候通过 store.getters.getTodoById(2) 这样访问
    // 这样通过方法访问就不会有缓存
  }
})
```

可以像访问 State 访问这些 Getter

```js
store.getters.doneTodos // -> [{ id: 1, text: '...', done: true }]

// 或者在组件中
computed: {
  doneTodosCount () {
    return this.$store.getters.doneTodosCount
  }
}

// 或者通过 mapGetters
computed: {
  // 使用对象展开运算符将 getter 混入 computed 对象中
  ...mapGetters([
  'doneTodosCount',
  'anotherGetter',
  // ...
  ])
}
```

# Mutation

```js
const store = new Vuex.Store({
  state: {
    count: 1
  },
  mutations: {
    // 将 state 作为第一个参数
    increment (state) {
      // 变更状态
      state.count++
    },
    // 将 payload 作为第二个参数，大多数情况下 payload 应该传一个对象
    decrement (state, payload) {
      state.count -= payload.amount
    }
  }
})
```
然后这样提交
```js
store.commit('increment')
store.commit('decrement', {
  amount: 10
})

// 或者写成含有 type 属性的对象
sotre.commit({
  type: 'increment',
  amount: 10
})

// 或者在组件中提交
methods: {
  increment () {
    this.$store.commit('increment')
  },
  // 或者使用 mapMutations
  ...mapMutations([
    'increment' // 将 `this.increment()` 映射为 `this.$store.commit('increment')`
  ]),
  ...mapMutations({
    add: 'increment' // 将 `this.add()` 映射为 `this.$store.commit('increment')`
  })
}
```

{% note warning %}
Mutation 必须是同步函数，因为异步会使得很难调试，无法区分谁先回调
{% endnote %}

# Action

Action 类似于 mutation，不同在于：
- Action 提交的是 mutation，而不是直接变更状态
- Action 可以包含任意异步操作

```js
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  },
  // 接受一个与 store 实例具有相同方法和属性的 context 对象
  // 这个 context 对象也有 context.state context.getters 这种方法
  actions: {
    increment (context) {
      context.commit('increment')
    }
  },

  // 可以这样简化代码：
  action: {
    incrementAsync ({ commit }) {
      setTimeout(() => {
        commit('increment')
      }, 1000)
    }
  }
})
```
然后这样触发，同 Mutation 一样，也支持 payload 和含有 type 的对象的形式
```js
store.dispatch('incrementAsync')

// 或者在组件中触发
methods: {
  increment () {
    this.$store.dispatch('incrementAsync')
  },
  // 或者使用 mapActions
  ...mapActions([
    'incrementAsync' // 将 `this.increment()` 映射为 `this.$store.dispatch('incrementAsync')`
  ]),
  ...mapActions({
    add: 'incrementAsync' // 将 `this.add()` 映射为 `this.$store.dispatch('incrementAsync')`
  })
}
```

# Module

```js
const moduleA = {
  state: { ... },
  mutations: { ... },
  actions: { ... },
  getters: { ... }
}

const moduleB = {
  state: { ... },
  mutations: { ... },
  actions: { ... }
}

const store = new Vuex.Store({
  modules: {
    a: moduleA,
    b: moduleB
  }
})

store.state.a // -> moduleA 的状态
store.state.b // -> moduleB 的状态
```
