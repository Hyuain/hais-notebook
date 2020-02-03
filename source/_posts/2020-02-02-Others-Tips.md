---
title: 一些 Tips
date: 2020-02-02 16:56:07
tags:
  - 入门
  - 收集
  - Tip
  - 踩坑
categories:
  - [前端, 其他]
---

收集一些 Tips。

<!-- more -->

# 滚动条的宽度

滚动条的宽度在 14 ~ 19px 之间，根据操作系统不同

# jQuery 制作 Tab

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

# jQuery 如果有 class 就删掉，没有就加上

```js
$square.on('click', () => {
  $square.toggleClass('active')
})
```

# JS 的代码不能以括号开头

# ***方法** 和 **函数**

- 方法是面向对象概念，一定是依附于一个对象的
- 函数是数学概念
- 都是指的 function

# 常用的画草图的软件

- balsamiq
- figma

# Promise

```js
Promise.reject('error')
  .then( ()=>{console.log('success1')}, ()=>{console.log('error1')} )
  .then( ()=>{console.log('success2')}, ()=>{console.log('error2')} )
// error1 -> success2
```

# 新建一个项目要考虑哪些东西？

- 创建仓库
- 声明 LICENCE
![](https://www.ruanyifeng.com/blogimg/asset/201105/free_software_licenses.png)
- 要用什么第三方的东西？ npm
- 在 webstorm 里面搜 VCS

# 单元测试

- BDD（Behavior-Driven Development）：行为驱动开发，用自然语言描述需求
- TDD（Test-Driven Development）：测试驱动开发，目的是为了让测试通过
- Assert：断言

# 持续集成

- 持续测试
- 持续交付
- 持续部署

# nodejs Client does not support authentication protocol requested by server; consider upgrading MySQL client

```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';
```

# redis

- web server 常用的缓存数据库，数据放在内存中
- 相比于 mysql，访问速度快；但是成本更高，数据量更小
- 解决：将 web server 和 redis 拆分成两个服务，不会占用 web server 的内存，可以跨进程访问
- 为何用 redis 来存 session？session 访问频繁，对性能要求高；不用考虑断电数据丢失的问题；session 数据量不会太大

# Vue 的渲染过程

```js
// 普通的同步渲染过程
const div = document.createElement('div')
const child = document.createElement('div')
div.appendChild(child) // -> Vue.$mounted 调用，$mounted 是异步的，队列 1
body.appendChild(div) // -> Vue.$mounted 调用，$mounted 是异步的，队列 2
console.log(div.outerHTML)
```

# 不能让  A —更新—> B 的同时 B —更新—> A

# 造轮子

1. 没有需求不要写代码，没有设计稿不要写代码
2. 单元测试是重构的前提
3. 设计模式
    1. 发布订阅模式
    2. 单向数据流
    3. 正交（props）