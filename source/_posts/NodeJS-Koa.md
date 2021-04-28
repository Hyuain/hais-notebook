---
title: Koa
date: 2021-04-06 22:50:00
tags:
  - 入门
categories:
  - [全栈, NodeJS]
---

NodeJS WEB 框架之 Koa

<!-- more -->

# Koa 与 Express 的区别

- 中间件模型不同：Koa 是 U 型，Express 是线型
- 语法特性不同，一般来说 NodeJS 7.6.0 之后才算完美支持 Koa
- Koa 没有内置中间件

# Koa 的中间件模型：U 型/洋葱型

```text
请求 -> f1 -> f2 -> f3 -> ...
                           ↓
响应 <- f1 <- f2 <- f3 <- ...
```

下面是一个简单的中间件使用示例，按照注释的顺序运行，可以看到一个以 `await next()` 为轴的 U 型：

```typescript
app.use(async (ctx, next) => {
  // 1. 什么也不做
  await next()
  // 5. 读取 X-Response-Time 中的时间，打印出来
  const time = ctx.response.get("X-Response-Time")
  console.log(`${ctx.url} - ${time}`)
})

app.use(async (ctx, next) => {
  // 2. 记录开始时间
  const start = Date.now()
  await next()
  // 4. 计算第三个中间件写 "Hello World" 的时间，并写入 X-Response-Time 响应头中
  const time = Date.now() - start
  ctx.set("X-Response-Time", `${time}ms`)
})

app.use(async (ctx, next) => {
  // 3. body = "Hello World"
  ctx.body = "Hello World"
  // 最后一个中间件可以不写 await next()
})
```

## `await next()`

`next()` 表示进入下一个函数，并返回一个 Promise，当下一个函数执行完成之后，会将 Promise 置为成功，然后 await 继续执行剩下的代码

实际上相当于：

```typescript
app.use((ctx, next) => {
  const start = Date.now()
  // 中间件需要返回一个 Promise 对象，使用 async 的写法将会自动 return 一个 Promise 对象
  return next().then(() => {
    const time = Date.now() - start
    ctx.set("X-Response-Time", `${time}ms`)
  }, (err) => { })
})
```

# Koa APIs

## app.xxx

- `app.env` 获取环境变量
- `app.listen()`
- `app.use()` 插入中间件
- `app.on("error", fn)` 错误处理
- `app.emit()` 触发事件

## ctx.xxx

- `ctx.req` NodeJS 封装的请求
- `ctx.request` Koa 封装的请求
- `ctx.res`
- `ctx.response`
- `ctx.state` 跨中间件分享数据
- `ctx.cookies.get` `ctx.cookies.set`
- `ctx.throw`
- `ctx.assert`

### Request 和 Response 委托

`ctx` 是 Request 和 Response 的委托，比如写 `ctx.body = ` 相当于写 `ctx.response.body = `。
如果要自己实现委托操作的话，可以这样实现：

```javascript
Object.defineProperty(ctx, "body", {
  get: () => {
    return ctx.response.body
  },
  set: (value) => {
    ctx.response.body = value
  }
})
```

### `ctx.request.xxx` `ctx.response.xxx`

可以直接查看 [相关文档](https://koa.bootcss.com/)