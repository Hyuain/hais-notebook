---
title: NodeJS
date: 2020-03-09 23:21:35
tags:
  - 入门
categories:
  - [全栈, NodeJS]
---

关于 NodeJS 的一些东西。

<!-- more -->

# 一些有用的工具

- commander：可以很方便地写出命令行，有多种语言的版本。
- node-dev：文件更新时自动重启的 node，不宜在生产环境使用。
- ts-node：让 node 支持直接运行 TypeScript 代码，不宜在生产环境使用。
- ts-node-dev：node-dev + ts-node，不宜在生产环境使用。
- 注意用 ts 开发时，需要安装 @types/node

# Node.js 简介

## Node.js 是什么

- 不是 web 框架，不能把 Node.js 与 Flask 或 Spring 对比
- 不是编程语言，不能与 Python 或 PHP 对比
- 是一个平台，将多种技术组合起来，让 JavaScript 也能调用系统接口、开发后端应用
  - V8 引擎
  - libuv
  - C/C++ 实现的 c-ares、http-parser、OpenSSL、zlib 等

## Node.js 技术架构

### 第三层：Node.js API

### 第二层：Node.js bindings、C/C++ 插件

#### bindings

沟通 Node.js 与 C/C++，比如 C/C++ 实现了 http-parsers 库，bindings 帮助我们调用这个库，比如将 C++ 编译成 .node，然后 JS require 他。

#### C/C++ 插件

自己写其他能力。

### 第一层：V8、libuv、c-ares、OpenSSL、zlib、http-parser、zlib 等

#### libuv

- 每个操作系统上有不同的异步处理模块，比如 FreeBSD 上的 kqueue、Linux 上的 epoll、Windows 上的 IOCP
- libuv 是一个跨平台的异步 I/O 库，他会根据系统选择合适的方案
- 可以用于 TCP/UDP/DNS/文件 等异步操作

### V8

- 将 JS 源代码变成本地代码（机器代码）执行
- 维护调用栈
- 内存管理
- 垃圾回收，重复利用无用的内存
- 实现 JS 的标准库

注意：

- V8 不提供 DOM API，DOM API 是浏览器提供的
- V8 是多线程的，比如垃圾回收为单独的线程，但执行 JS 是单线程的
- 可以开启两个线程来执行 JS，但这两个线程之间没什么关系
- V8 自带 event loop，但 Node.js 基于 libuv 自己做了一个 event loop

#### Event Loop

- Event：包括内部事件（计时器到期了）、外部事件（文件可以读取了、socket 有内容了）
- Loop：事件是有优先级的，处理顺序是有先后的，Node.js 要按顺序轮询每种事件，这种轮询往往都是循环的

[官方文档](https://nodejs.org/zh-cn/docs/guides/event-loop-timers-and-nexttick/)

重点阶段：

- timers 检查计时器
- poll 轮询，大部分时间会停留在这个阶段，大部分事件将在 poll 得到处理，比如文件读取、网络请求
- check 检查 setImmediate 回调

> setTimout 与 setImmediate 谁先执行？
> 不确定，大多数情况是 setImmediate 先执行，但有时刚进入的时候，会先执行 setTimout 

# 文件模块

```javascript
const fs = require("fs")
const p = require("path")

const defaultDbPath = p.join(__dirname, ".todo")

module.exports = {
  read(path = defaultDbPath) {
    return new Promise((resolve, reject) => {
      fs.readFile(path, {
        flag: "a+"
      }, (error, data) => {
        if (error) {
          return reject(error)
        }
        resolve(data)
      })
    })
  },
  write(content = "defaultContent", path = defaultDbPath) {
    return new Promise((resolve, reject) => {
      fs.writeFile(path, content, (error) => {
        if (error) {
          return reject(error)
        }
        resolve()
      })
    })
  }
}

```

# HTTP 模块

一个最简单的服务器：

```typescript
import * as http from "http"
import { IncomingMessage, ServerResponse } from "http"

const server = http.createServer()

server.on("request", (request: IncomingMessage, response: ServerResponse) => {
  console.log("request.method", request.method)
  console.log("request.url", request.url)
  console.log("request.headers", request.headers)
  // 监听 data 事件才能拿到 post 的请求体，用户每上传一点内容，就会触发一次 data 事件
  const array = []
  request.on("data", (chunk) => {
    array.push(chunk)
  })
  // 当流中没有数据了，会触发 end 事件
  request.on("end", () => {
    const body = Buffer.concat(array)
    console.log("body", body.toString())

    // 设置 response
    response.statusCode = 400
    response.setHeader("x-harvey", "WOW")
    response.write("1\n")
    response.write("2\n")
    response.end("request end")
  })
})

server.listen(8888)
```

可以配合 [文档](https://nodejs.org/api/http.html#http_event_request) 食用

## 根据 url 返回不同的文件

```typescript
import * as http from "http"
import { IncomingMessage, ServerResponse } from "http"
import * as fs from "fs"
import * as p from "path"

const server = http.createServer()
const publicDir = p.resolve(__dirname, "public")

server.on("request", (request: IncomingMessage, response: ServerResponse) => {
  const { method, url, headers } = request
  switch (url) {
  case "/index.html":
    fs.readFile(p.resolve(publicDir, "index.html"), ((err, data) => {
      if (err) { throw err }
      response.end(data.toString())
    }))
    break
  case "/style.css":
    response.setHeader("Content-Type", "text/css; charset=utf-8")
    fs.readFile(p.resolve(publicDir, "style.css"), ((err, data) => {
      if (err) { throw err }
      response.end(data.toString())
    }))
    break
  case "/index.js":
    response.setHeader("Content-Type", "text/javascript; charset=utf-8")
    fs.readFile(p.resolve(publicDir, "index.js"), ((err, data) => {
      if (err) { throw err }
      response.end(data.toString())
    }))
    break
  }
})

server.listen(8888)
```

## 处理查询参数

```typescript
const { pathname, searchParams } = new URL(requestUrl, "http://localhost:8888")
```

## 匹配任意文件

```typescript
server.on("request", (request: IncomingMessage, response: ServerResponse) => {
  const { method, url: requestUrl, headers } = request
  const { pathname } = new URL(requestUrl, "http://localhost:8888")
  const filename = pathname.substr(1) || "index.html"
  fs.readFile(p.resolve(publicDir, filename), (err, data) => {
    if (err) {
      if (err.errno === -4058) {
        response.statusCode = 404
      } else {
        response.statusCode = 500
      }
      return response.end()
    }
    response.end(data)
  })
})
```

## 处理非 get 请求

```typescript
if (method !== "GET") {
  response.statusCode = 405
  return response.end()
}
```

## 添加缓存

```typescript
response.setHeader("Cache-Control", `public, max-age=${cacheAge}`)
```