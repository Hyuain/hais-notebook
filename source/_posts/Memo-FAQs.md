---
title: FAQs
date: 2022-03-12 10:30:50
categories:
  - [Memo]
---

.

<!-- more -->

# CSS

## 滚动条的宽度

滚动条的宽度在 14 ~ 19px 之间，根据操作系统不同

## 回溯机制

## flex 的常见属性

## 页面导入样式时，`link` 和 `@import` 有什么区别

| link                                      | @import              |
| ----------------------------------------- | -------------------- |
| HTML 标签，除了 CSS 之外还可以定义 RSS 等 | CSS 提供的           |
| 加载页面时同时加载                        | 页面加载完之后再加载 |
| 没有兼容性问题                            | 不兼容 IE 5 以下     |
| 可以通过 DOM 动态引入                     | 不能动态引入         |


## 移动端响应式方案

1. @media
2. PC 固定布局，移动端 rem
3. flex
4. vh/vw

## CSS 选择器优先级

1. 越具体优先级越高
2. 写在后面的覆盖写在前面的
3. `important!` 最高，但是少用

## 清除浮动

# DOM

##  事件委托

父级通过 `target` 来判断是谁上面发生的事件，并作出不同的响应

```js
ul.addEventListener('click', (e) => {
  if(e.target.tagName.toLowerCase() === 'li') {
    console.log('点击了 li')
  }
})
```

```js
const bindEvent = (element, eventType, selector, fn) => {
  element.addEventListener(eventType, e => {
    let el = e.target
    while (!el.matches(selector)) {
      if (el === element) {
        el = null
        break
      }
      el = el.parentNode
    }
    el && fn.call(el, e)
  })
}
```

## 用 mouse 事件写一个可拖拽的 div

```js
let dragging = false
let position = null

div.addEventListener('mousedown', (e) => {
  dragging = true
  position = [e.clientX, e.clientY]
})

document.addEventListener('mousemove', (e) => {
  if (!dragging) return
  const x = e.clientX
  const y = e.clientY
  const deltaX = x - position[0]
  const deltaY = y - position[1]
  const left = parseInt(div.style.left || 0)
  const top = parseInt(div.style.top || 0)
  div.style.left = left + deltaX + 'px'
  div.style.top = top + deltaY + 'px'
  position = [x, y]
})

document.addEventListener('mouseup', (e) => {
  dragging = false
})
```

# HTTP

## PWA

## GET 和 POST 的区别

- GET 不安全，POST 不安全（其实都不安全）
- GET 是有长度限制的，POST 没有长度限制（但一般 GET 的长度不超过 1 KB，POST 上限 4M ~ 20M 左右）
- GET 参数在 URL 中，POST 消息在消息体中
- GET 只需要一个报文，POST 需要两个以上（因为有消息体）
- GET 幂等，POST 不幂等
- 语义不同，GET 是为了获取数据，POST 是为了提交数据

## nslookup

直接使用可以查询到域名的 A 记录。比如：

```
nslookup hais-teatime.com
```

## 域名

比如对于 www.baidu.com

- 顶级域名：com
- ：baidu.com
- 三级域名：www.baidu.com

三级域名跟二级域名可以没关系，有可能都不是属于一家公司，注意辨别。

# jQuery

## 制作 Tab

```js
$tabBar.on('click', 'li', e => {
  const $li = $(e.currentTarget)
  $li
    .addClass('selected')
    .siblings()
    .removeClass('selected')
  const index = $li.index()
  $tabContent
    .children()
    .eq(index)
    .addClass('active') // 一定不要用.css 和.show，样式和行为分离
    .siblings()
    .removeClass('active')
})

$tabBar.children().eq(0).trigger('click')  // 默认触发第一个
```

# Vue

## 组件间通信

1. 父子组件：`$emit` 和 `$on`
2. 爷孙组件、兄弟组件：eventBus
3. Vuex

## 渲染过程

```js
// 普通的同步渲染过程
const div = document.createElement('div')
const child = document.createElement('div')
div.appendChild(child) // -> Vue.$mounted 调用，$mounted 是异步的，队列 1
body.appendChild(div) // -> Vue.$mounted 调用，$mounted 是异步的，队列 2
console.log(div.outerHTML)
```

不能让  A —更新—> B 的同时 B —更新—> A

# React

## 组件间通信

1. 父子组件：通过 props 直接传参（onChange、onClick）
2. 爷孙组件：传两次 props
3. 任意组件：Redux

## DOM Diff

### Reconciliation

Reconciliation 直译为协调，即 React 的渲染机制，他有以下几步：

1. props 或 state 改变
2. render 函数返回不同的元素树（虚拟 DOM）
3. 新旧 DOM 对比（vDOM Diff）
4. 针对差异的地方进行更新
5. 渲染为真实的 DOM 树

### DOM Diff 原理

#### 设计思想

1. 永远只比较同层的节点，不会跨层级比较
2. 不同的两个节点产生不同的树（两个类型不同的节点直接用新的全部替代旧的，包括其后代）
3. 通过 key 判断哪些元素是相同的（比如列表如果没有 key，从头部插入元素会导致列表全部更新），因此 key 需要在列表中保持唯一（不需要全局唯一）

#### 比较流程

- 若元素类型不相同：直接用新的树替换掉原来的树
- 若元素类型相同：
  - 若都是 DOM 节点：更新 DOM 属性，比如 `style`、`title` 等，再向下递归找
  - 若都是组件节点：组件实例保持不变，更新 Props

### 如何减少 Diff 过程

> 利用 `shouldComponentUpdate`

默认的 `shouldComponentUpdate` 会在 props 或 state 发生变化的时候返回 true，表示组件会重新渲染，然后调用 render 函数，进行 vDOM Diff；相对的，我们也可以通过控制它的返回值来控制是否发生 vDOM Diff

### 浅比较与深比较

- 类组件的 `shouldComponentUpdate` 中可以对 props 和 state 进行浅比较（使用 `pureComponent` API，比较一层 key 和 value，类似于浅拷贝），也可以进行深比较（递归）
- 函数组件使用 `memo` 方法可以进行肩比较，但是只比较了 props：

```js
function Component() { }
export default React.memo(Component)
```

### immutable 数据结构

immutable 的意义：浅比较缺点很明显，深比较有时候又比较浪费性能

简单来说：

1. **节省性能**：immutable 内部采用多叉树结构，如果它里面有节点被改变，那么则更新 **这个节点** 和他有关的所有 **上级节点**
2. **返回一个新的引用**，即使是浅比较也能感知到数据的变化

#### 一些 immutable API

##### fromJS

将 JS 对象转换为 immutable 对象

```js
import {fromJS} from 'immutable'
const immutableState = fromJS({
  count: 0
})
```

##### toJS

将 immutable 对象转换为 JS 对象

```js
const jsObj = immutableState.toJS()
```

##### get/getIn

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

##### set

用来给 immutable 对象的属性赋值

```js
let immutableObj = fromJS({a: 1})
immutableObj.set('a', 2)
```

##### merge

新旧数据对比，旧数据中不存在的属性直接添加，存在的属性用新数据覆盖

# Webpack

[更多问题可以查看这篇知乎专栏](zhuanlan.zhihu.com/p/44438844)

## 提高构建速度

- happypack 使用多线程打包
- dllplugin

## webpack 文件过大

- 提取通用模块文件
- 压缩 JS、CSS、图片
- 按需加载

## import alias

### 在 JS 或 TS 中使用 `@`

可以直接使用

### 在 CSS 或 SCSS 中使用 `@`

需要使用 `~@`，但是 webstorm 中会报错，需要点开 `settings`-`webpack`，在路径中找到 `node_modules\@vue\cli-service\webpack.config.js`

# 数据结构

## 哈希表

按照一定的规则（有很多种不同的规则，可以按实际情况选择）去存储数据，这样找数据的时候就会很快了。
有时候会出现冲突（不同的数据根据规则计算应该放到同样的地方），这个时候需要解决冲突。

```js
class HashTable {
  constructor(length = 1000) {
    this.slots = Array(length) // slots 就是哈希表的这个数组
  }
  // 计算出要存的位置，这里的方法是：将这个字符串的每一位的 ASCII 码算出来，再把每一位的加起来，再%哈希表的总长度
  hash(v) {
    // 将字符串解构为一个个的数组
    let value =[...v.toString()].map(char => char.charCodeAt(0)).reduce((result, item) => result + item)
    return value%this.slots.length
  }
  // 存数据，因为 hash 值可能相同，所以用 push
  add(value) {
    let key = this.hash(value)
    if(this.slots[key]) {
      if(!this.slots[key].includes((value))) this.slots[key].push[value]
    } else {
      this.slots[key] = [value]
    }
  }
  // 删数据
  remove(value){
    let key = this.hash(value)
    if(this.slots[key] && this.slots[key].includes(value)) {
      this.slots[key] = this.slots[key].filter(v => v !== value)
      return true
    }
    return false
  }
  search(value){
    let key = this.hash(value)
    return !!(this.slots[key] && this.slots[key].includes(value));
  }
  show(){
    console.log(this.slots)
  }
}
```

## 图

顶点：vertex
边：edge

可以这样表示：

```text
[[V1, V2, V4], [V0, V2, V3], [V0, V1], [V1], [V0]]
```

每一个数组表示对应的顶点与那些点相邻

```js
class Graph {
  constructor(vCount) {
    this.vertexCount = vCount // 顶点数
    this.adjacencyList = [...Array(vCount)].map(v => []) // 相邻顶点的列表，得到类似于数组 [[], [], [], []]
    this.verticesStatus = [...Array(vCount)].map(v => false) // 记录点是不是找过了
  }
  addEdge(v1, v2) {
    this.adjacencyList[v1].push(v2)
    this.adjacencyList[v2].push(v1)
  }
  showGraph() {
    this.adjacencyList.forEach((adjVertices, v) => {
      console.log(`${v} -> ${adjVertices.toString()}`)
    })
  }
  resetStatus() {
    this.verticesStatus = [...Array(this.vertexCount)].map(v => false)
  }
  // 深度优先搜索
  dfs(v) {
    this.verticesStatus[v] = true
    console.log(`访问到：${v}`)
    // Array.isArray 判断传递进来的是否是一个 Array
    if (Array.isArray(this.adjacencyList[v])) {
      this.adjacencyList[v].forEach(adjVertex => {
        if(!this.verticesStatus[adjVertex]) this.dfs(adjVertex)      
      })
    }
  }
  // 广度优先搜索
  bfs(v) {
    let queue = []
    queue.push(v)
    while (queue.length > 0) {
      let v = queue.shift()
      if (this.verticesStatus[v]) continue
      this.verticesStatus[v] = true
      console.log(`访问到 ${v}`)
      this.adjacencyList[v].forEach(adjVertex => {
        queue.push(adjVertex)
      })
    }
  }
}
```

# 架构

## 新建一个项目

1. 创建仓库
2. 声明 LICENCE
   ![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/free_software_licenses.png)

3. 要用什么第三方的东西？ npm
4. 在 webstorm 里面搜 VCS

## 单元测试

- BDD（Behavior-Driven Development）：行为驱动开发，用自然语言描述需求
- TDD（Test-Driven Development）：测试驱动开发，目的是为了让测试通过
- Assert：断言

## 持续集成

- 持续测试
- 持续交付
- 持续部署

# MySQL

## nodejs Client does not support authentication protocol requested by server; consider upgrading MySQL client

```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';
```

# Redis

- web server 常用的缓存数据库，数据放在内存中
- 相比于 mysql，访问速度快；但是成本更高，数据量更小
- 解决：将 web server 和 redis 拆分成两个服务，不会占用 web server 的内存，可以跨进程访问
- 为何用 redis 来存 session？session 访问频繁，对性能要求高；不用考虑断电数据丢失的问题；session 数据量不会太大

# 其他

## 发展趋势

### 最重要的

- React Hooks
- ES6
- TypeScript
- Flutter

### 值得一学的

- Graph QL
- PWA
- WebAssembly
- WebGL 3D
- 《计算的本质》

### 需要了解的

#### HTML 5

- 语义化标签
- 音视频处理
- canvas / webGL
- history API
- requestAnimationFrame
- 地理位置
- web socket

#### CSS 3

- 常规
- 动画
- 盒子模型
- 响应式布局

#### JavaScript

- ES 3/5/6/7/8/9
- DOM
- BOM
- 设计模式
- 底层原理
  - 堆栈内存
  - 闭包作用域 AO/VO/GO/EC/ESTACK
  - 面向对象 OOP
  - This
  - EventLoop
  - 浏览器渲染原理
  - 回流重绘

#### 网络通信层

- AJAX / Fetch / axios
- HTTP 1.0/2.0
- TCP
- 跨域处理方案
- 性能优化

#### Hybrid / APP / 小程序

- Hybrid
- uni-app
- RN
- Flutter
- MPVUE
- Weex
- PWA

#### 工程化

- webpack
- git
- linux / nginx

