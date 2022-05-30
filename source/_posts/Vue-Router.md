---
title: Vue Router
date: 2020-02-01 11:31:49
tags:
  - 入门
  - 文档
categories:
  - [前端, Vue]
---

用了几次 VueRouter 之后抄一下文档。

<!-- more -->

# 基本使用

## HTML

`<router-link/>` 就像 `<a>` 标签，来链接到不同的路由

```html
<router-link to="/foo">Go to Foo</router-link>
```

`<router-view/>` 用来装路由匹配出来的内容

```html
<router-view/>
```

## JavaScript

```js
// 0. 引入 VueRouter
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter)

// 1. 引入要被路由的组件
import Foo from '../Foo.vue'
import Bar from '../Bar.vue'

// 2. 定义路由，相当于定义 router 的配置选项
const routes = [
  {path: '/foo', component: Foo},      
  {path: '/bar', component: Bar}      
]

// 3. 创建 router 实例
const router = new VueRouter({
  routes  
})

// 4. 创建根实例，并使用 VueRouter
const App = new Vue({
    router
}).$mount('#app')
```

# 路由匹配

| 模式 | 匹配路径 | $route.params |
| --- | --- | --- |
| /user/:username | /user/evan | `{ username: 'evan' }` |
| /user/:username/post/:post_id | /user/evan/post/123 | `{ username: 'evan', post_id: '123' }` |

{% note warning %}
当使用路由参数时，例如从 `/user/foo` 导航到 `/user/bar`，原来的组件实例会被复用，组件的生命周期钩子不会再被调用
{% endnote %}

如果想要对路由参数的变化做出响应，需要 watch $route 对象：

```js
const User = {
  template: '...',
  watch: {
    '$route' (to, from) {
      // 对路由变化作出响应...
    }
  }
}
```

或者使用 beforeRouterUpdate：
```js
const User = {
  template: '...',
  beforeRouteUpdate (to, from, next) {
    // react to route changes...
    // don't forget to call next()
  }
}
```

# 嵌套路由

一个被渲染组件同样可以包含自己的嵌套 `<router-view>`，此时要在 `VueRouter` 的参数中增加 `children` 的配置：

```js
const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User,
      children: [
        {
          // 当 /user/:id 匹配成功，
          // UserHome 会被渲染在 User 的 <router-view> 中
          path: '',
          component: UserHome
        },
        {
          // 当 /user/:id/profile 匹配成功，
          // UserProfile 会被渲染在 User 的 <router-view> 中
          path: 'profile',
          component: UserProfile
        },
        {
          // 当 /user/:id/posts 匹配成功
          // UserPosts 会被渲染在 User 的 <router-view> 中
          path: 'posts',
          component: UserPosts
        }
      ]
    }
  ]
})
```

{% note warning %}
以 / 开头的嵌套路径会被当作根路径
{% endnote %}

# 编程式导航

## `router.push(location, onComplete?, onAbort?)`

相当于 `<router-link :to="...">`，两者的语法规则相同

{% note warning %}
在 Vue 实例里面，可以通过 $router 访问路由实例 `router`。因此可以调用 this.$router.push
{% endnote %}

参数可以是字符串路径或者一个对象：

```js
// 字符串
router.push('home')

// 对象
router.push({ path: 'home' })

// 命名的路由
router.push({ name: 'user', params: { userId: '123' }})

// 带查询参数，变成 /register?plan=private
router.push({ path: 'register', query: { plan: 'private' }})
```

```js
const userId = '123'
router.push({ name: 'user', params: { userId }}) // -> /user/123
router.push({ path: `/user/${userId}` }) // -> /user/123
// 提供了 path 之后 params 不生效
router.push({ path: '/user', params: { userId }}) // -> /user
```

`onComplete` 和 `onAbort` 参数可以传入两个回调：将会在导航成功完成 (在所有的异步钩子被解析之后) 或终止 (导航到相同的路由、或在当前导航完成之前导航到另一个不同的路由) 的时候进行相应的调用。

{% note warning %}
如果目的地和当前路由相同，只有参数发生了改变 (比如从一个用户资料到另一个 /users/1 -> /users/2)，你需要使用 beforeRouteUpdate 来响应这个变化 (比如抓取用户信息)
{% endnote %}

## `router.replace(location, onComplete?, onAbort?`

相当于 `router-link :to="..." replace`，不会向 history 添加新的记录，而是替换掉当前的 history 目录

{% note warning %}
跟 `push` 的主要区别就是 `replace` 可以回去，`push` 则不能
{% endnote %}

## `router.go(n)`

类似 `window.history.go(n)` 意为在 history 记录中前进或后退多少步

```js
// 在浏览器记录中前进一步，等同于 history.forward()
router.go(1)

// 后退一步记录，等同于 history.back()
router.go(-1)

// 前进 3 步记录
router.go(3)

// 如果 history 记录不够用，那就默默地失败呗
router.go(-100)
router.go(100)
```

# 命名视图

```html
<router-view class="view one"></router-view>
<router-view class="view two" name="a"></router-view>
<router-view class="view three" name="b"></router-view>
```

```js
const router = new VueRouter({
  routes: [
    {
      path: '/',
      components: {
        default: Foo,
        a: Bar,
        b: Baz
      }    
    }
  ]  
})
```

# 导航守卫

## 全局前置导航守卫 `router.beforeEach`

```js
const router = new VueRouter({ ... })

router.beforeEach((to, from, next) => {
  // ...
})
```

{% note warning %}
守卫是异步解析执行，此时导航在所有守卫 resolve 完之前一直处于 等待中
确保要调用 next 方法，否则钩子就不会被 resolved
{% endnote %}

- `next()`: 进行管道中的下一个钩子。如果全部钩子执行完了，则导航的状态就是 confirmed (确认的)。

- `next(false)`: 中断当前的导航。如果浏览器的 URL 改变了 (可能是用户手动或者浏览器后退按钮)，那么 URL 地址会重置到 `from` 路由对应的地址。

- `next('/')` 或者 `next({ path: '/' })`: 跳转到一个不同的地址。当前的导航被中断，然后进行一个新的导航。你可以向 `next` 传递任意位置对象，且允许设置诸如 `replace: true`、`name: 'home'` 之类的选项以及任何用在 `router-link` 的 `to` prop 或 `router.push` 中的选项。

- `next(error)`: 如果传入 `next` 的参数是一个 `Error` 实例，则导航会被终止且该错误会被传递给 `router.onError()` 注册过的回调。

## 全局解析守卫 `router.beforeResolve`
 
类似 `router.beforeEach`，在导航被确认之前，同时在所有组件内守卫和异步路由组件被解析之后，解析守卫就被调用

## 全局后置钩子 `router.afterEach`

```js
router.afterEach((to, from) => {
  // ...
})
```

## 路由独享的守卫 `beforeEnter`

```js
const router = new VueRouter({
  routes: [
    {
      path: '/foo',
      component: Foo,
      beforeEnter: (to, from, next) => {
        // ...
      }
    }
  ]
})
```

## 组件内的守卫

```js
const Foo = {
  template: `...`,
  beforeRouteEnter (to, from, next) {
    // 在渲染该组件的对应路由被 confirm 前调用
    // 不！能！获取组件实例 `this`
    // 因为当守卫执行前，组件实例还没被创建
    // 但是可以通过传给 next 一个回调来访问组件实例          
    next(vm => {
      // 通过 `vm` 访问组件实例
    })
  },
  beforeRouteUpdate (to, from, next) {
    // 在当前路由改变，但是该组件被复用时调用
    // 举例来说，对于一个带有动态参数的路径 /foo/:id，在 /foo/1 和 /foo/2 之间跳转的时候，
    // 由于会渲染同样的 Foo 组件，因此组件实例会被复用。而这个钩子就会在这个情况下被调用。
    // 可以访问组件实例 `this`
  },
  beforeRouteLeave (to, from, next) {
    // 导航离开该组件的对应路由时调用
    // 可以访问组件实例 `this`
    // 通常用来禁止用户在还未保存修改前突然离开，通过 next(false) 来取消用户的导航    
  }
}
```

## 导航的解析流程

1. 导航被触发
2. 在失活的组件里调用离开守卫
3. 调用全局的 `beforeEach` 守卫
4. 在重用的组件里调用 `beforeRouteUpdate` 守卫 (2.2+)
5. 在路由配置里调用 `beforeEnter`
6. 解析异步路由组件
7. 在被激活的组件里调用 `beforeRouteEnter`
8. 调用全局的 `beforeResolve` 守卫
9. 导航被确认
10. 调用全局的 `afterEach` 钩子
11. 触发 DOM 更新
12. 用创建好的实例调用 `beforeRouteEnter` 守卫中传给 `next` 的回调函数
