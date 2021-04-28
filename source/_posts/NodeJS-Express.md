---
title: Express
date: 2021-03-27 19:26:00
tags:
  - 入门
categories:
  - [全栈, NodeJS]
---

NodeJS WEB 框架之 Express

<!-- more -->

# Express Generator

express 脚手架。

# use

```typescript
import express from "express"

const app = express()
const port = 3000

app.use((request, response, next) => {
  console.log(request.url)
  response.write("first hi\n")
  // 为了让他执行下一个 use，需要调用 next()，否则他不会执行下一个 use
  next()
})

app.use((request, response, next) => {
  console.log("2")
  response.write("second hi\n")
  next()
})

app.use((request, response, next) => {
  response.end()
})

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`)
})
```

express 就是这样通过一个个 `use` 的函数，最终到达结束程序。每一个函数可以读取 request、写入 response、调用 next()。

# 中间件

`use` 的那些函数就被称为 **中间件**。
可以看一下 `express-generator` 为我们创建的 `app.js` 文件的其中一部分。

```javascript
app.use(logger("dev"))
app.use(express.json())
app.use(express.urlencoded({ extended: false }))
app.use(cookieParser())
app.use(express.static(path.join(__dirname, "public")))
```

Express 这样的编程模型的优点就是模块化：每个功能都能通过一个函数实现，然后通过 app.use 将函数们整合起来。如果这些函数放到不同的文件或者 npm 包里，就实现了模块化。

# 路由

用 use 也可以很方便地实现路由：

```typescript
app.use((request, response, next) => {
  if (request.path === "/" && request.method === "get") {
    response.send("ROOT")
  }
  next()
})
```

由此衍生除了一些 API 糖：

```typescript
app.use("/xxx", fn)
app.get("/xxx", fn)
app.post("/xxx", fn)
app.route("/xxx").all(f1).get(f2).post(f3)
```

# 错误处理

`next()` 可以传入一个错误/字符串，就会进入错误处理函数：

```typescript
app.use((request, response, next) => {
  if (hasError) {
    // 传给 next 一个错误，就会直接进入 errorHandler
    return next("出错了！")
  }
  next()
})

app.use((request, response, next) => {
  console.log("如果没有错误，这里才会执行")
})

app.use((error, request, response, next) => {
  response.write(error)
  response.end()
  next()
})
```

# 子应用与挂载点

```javascript
const app = express()
const admin = express()

app.use("/admin", admin) // admin 是子应用，/admin 是 admin 的挂载点
```

# Express APIs

## express.xxx

- `express.static()` 指定静态资源文件夹路径
- `express.json()` 解析 Content-Type 为 application/json（或类似）的内容
- `express.urlencoded()` 解析 Content-Type 为 application/x-www-form-urlencoded（或类似）的内容
- `express.text()` 解析 Content-Type 为 text/plain（或类似）的内容
- `express.raw()` 解析 Content-Type 为 application/octet-stream （或类似）的内容

## app.xxx

- `app.set()` set 一个值，不过有一些特殊值，比如 `case sensitive routing` `env` `views` `view engine`，而且一般这些特殊值放在所有的中间件之前才会生效
- `app.get()` 可以获取到你 set 的东西
- `app.get()` `app.post()` `app.put()` `app.delete()` `app.patch()` `app.all()` 这些都是 `app.use()` 的 API 糖 
- `app.locals` 设置一些局部变量，比如 `app.locals.title = "My Title"`，然后就可以在其他地方进行读取
- `app.mountpath` 挂载路径

## request.xxx

- `request.app` 获取到全局的 app
- `request.get()` 获取请求头
- `request.param()` 获取某个参数
- `request.range()`
  1. 用 HEAD 请求某个资源，如果资源支持分片下载，就可以得到类似于 `Accept-Ranges: bytes`（支持以字节为分片下载） 的响应头。[可以参阅这里](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Range_requests)
  2. 然后用请求头 `Range` 就可以设置分片下载
  3. `request.range` 则是用于解析 `Range` 这个请求头的
  
## response.xxx

- `response.app` 获取到全局的 app
- `response.headersSent` 是否已经发送响应头
- `response.set()` 设置响应头（覆盖之前的）
- `response.append()` 设置请求头（不覆盖之前的）
- `response.status()` 设置状态码
- `repoonse.cookie()` 设置 cookie
- `response.download()` 提供下载
- `response.format()` 根据请求的不同 `Accept` 值返回不同的类型
- `response.json()` 类似于 `JSON.stringify()`
- `response.location()` 一般用于 301/302/307 跳转，设置响应头 `Location`
- `response.redirect()` 相当于 `response.location()` + `response.status()`
- `response.write()` 写入消息体（流式操作）
- `response.end()` 写入消息体（一次性），不能跟 `response.write` 同时使用

## router.xxx

`router` 相当于一个小型的 `app`，是一个单独的用于路由的实例，需要这样声明：

```typescript
const userRouter = express.Router()
const app = express()

userRouter.get("/:id", (request, response, next) => {})
app.use("/users", userRouter) // 将 userRouter 挂载到 app 上
```

`router` 有这些 API，基本被 `app` 包含。

- `router.all()`
- `router.METHOD()`
- `router.param()`
- `router.route()`
- `router.use()`
