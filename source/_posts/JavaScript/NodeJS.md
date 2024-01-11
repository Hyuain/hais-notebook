---
title: NodeJS
date: 2020-03-09 23:21:35
categories:
  - [全栈]
---

.

<!-- more -->

# 一些有用的工具

- commander：可以很方便地写出命令行，有多种语言的版本。
- node-dev：文件更新时自动重启的 node，不宜在生产环境使用。
- ts-node：让 node 支持直接运行 TypeScript 代码，不宜在生产环境使用。
- ts-node-dev：node-dev + ts-node，不宜在生产环境使用。
- 注意用 ts 开发时，需要安装 @types/node
- tsc --init 命令会自动创建 `tsconfig.json` 文件

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

# Stream

数据流叫 Stream，每次写的小数据叫 chunk，产生数据的一段叫 source，得到数据的一段叫 sink。

## NodeJS 中的 Stream

### 用 Stream 写文件

```javascript
const fs = require("fs")
const stream = fs.createWriteStream("./big_file.txt")
for (let i = 0; i < 1000; i++) {
  stream.write(`这是第 ${i} 行的内容，哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈\n`)
}
stream.end()
console.log("done")
```

### 不用 Stream 读文件

```javascript
const http = require("http")
const fs = require("fs")

const server = http.createServer()

server.on("request", (request, response) => {
  fs.readFile("./big_file.txt", (err, data) => {
    if (err) throw err
    response.end(data)
    console.log("done")
  })
})

server.listen(8888)
```

可以看到 Nodejs 进程的内存占用非常大。

### 用 Stream 读文件

```javascript
const http = require("http")
const fs = require("fs")

const server = http.createServer()

server.on("request", (request, response) => {
  const stream = fs.createReadStream("./big_file.txt")
  stream.pipe(response) // response 也是 stream，文件 stream 和 response stream 通过管道连接
  stream.on("end", () => console.log("done"))
})

server.listen(8888)
```

使用 Stream，Nodejs 的内存占用会更低。

管道也可以通过这样来实现：

```javascript
// stream1 有数据就塞给 stream2
stream1.on("data", (chunk) => {
  stream2.write(chunk)
})
// stream1 停了，就停掉 steam2
stream1.on("end", () => {
  stream2.end()
})
```

## Stream 的原型链

```javascript
s = fs.createReadStream(path)
```

```text
s -> fs.ReadStream -> stream.Readable -> stream.Stream -> event.EventEmitter -> Object
```

### 支持的事件和方法

#### stream.Readable

- 事件：data, end, error, close, readable, pause, resume ...

- 方法：
  - pipe()
  - unpipe()
  - wrap()
  - destroy()
  - read()
  - unshift()
  - resume()
  - pause()
  - isPaused()
  - setEncoding()

#### stream.Writable

- 事件：drain, finish, error, close, pipe, unpipe ...

- 方法：
  - write()
  - destroy()
  - end()
  - cork()
  - uncork()
  - setDefaultEncoding()

## Stream 的分类

- Readable，可读，数据的生产者
- Writable，可写，数据的消费者
- Duplex，可读可写（双向，读和写的内容无关）
- Transform，可读可写（变化，比如先把内容读出来，再写到别的地方）

### Readable Stream

- paused 静止态
- flowing 流动态

默认处于 paused；添加 data 事件监听，就会变成 flowing；删掉 data 事件监听就又变成 paused。
pause() 可以让他变为 paused，resume() 可以让他变为 flowing。

### Writable Stream

- drain：writable.write(chunk) 将会返回一个 boolean，如果 boolean 为 false，这个 stream 堵车了，这个时候就需要等待。当 writable 监听到 drain 事件，表示不堵车了，可以继续传输数据。
- finish：调用 stream.end() 方法，并且缓冲区的数据都已经传给底层系统后会触发。

### Transform Stream

NodeJS 内置了一些 Transform Stream，可以看这个使用 gzip 压缩的例子：

```javascript
const fs = require("fs")
const zlib = require("zlib")
const { Transform } = require("stream")
const crypto = require("crypto")
const file = process.argv[2]

const reportProgress = new Transform({
  transform(chunk, encoding, callback) {
    process.stdout.write(".")
    // 可以将 chunk push 进去
    // this.push(chunk)
    // 也可以把 chunk 给 callback
    callback(null, chunk)
  }
})

fs.createReadStream(file)
  .pipe(crypto.createCipher("aes-192-gcm", "123456"))
  .pipe(zlib.createGzip())
  // 可以用 on 来进行监听并作出副作用（不会影响原来的数据）
  // .on("data", () => process.stdout.write("."))
  // 也可以插入一个 TransformStream
  .pipe(reportProgress)
  .pipe(fs.createWriteStream(file + ".gz"))
  .on("finish", () => console.log("Done"))


```

## 创建一个 Stream

### 创建一个 Writable Stream

```javascript
const { Writable } = require("stream")

const consumer = new Writable({
  write(chunk, encoding, callback) {
    console.log(chunk.toString())
    // callback 调用这句必须要写，否则就会卡住
    callback()
  }
})

// 用户的输入 stream -> consumer
process.stdin.pipe(consumer)

// 也可以这样写：
// process.stdin.on("data", (chunk) => {
//   consumer.write(chunk)
// })
```

### 创建一个 Readable Stream

```javascript
const { Readable } = require("stream")

const producer = new Readable()

producer.push("HAHAHA")
producer.push("XIXIXI")
// push null 必须写，表示已经结束了
producer.push(null)

producer.pipe(process.stdout)

// 也可以这样写
// producer.on("data", (chunk) => {
//   process.stdout.write(chunk)
//   console.log("WRITE")
// })
```

上面的写法是先将所有的数据 push 进流里面，那么有没有办法让他边读边写呢？

```javascript
const { Readable } = require("stream")

const producer = new Readable({
  read(size) {
    this.push(String.fromCharCode(this.currentCharCode++))
    if (this.currentCharCode > 90) {
      // push null 表示终止
      this.push(null)
    }
  }
})

producer.currentCharCode = "A".charCodeAt()

producer.pipe(process.stdout)
```

上述的代码就是在 `process.stdout` 要数据的时候（调用 `read` 的时候），`producer` 才往里面 push 一个字符。 

### 创建一个 Duplex Stream

```javascript
const { Duplex } = require("stream")

const duplex = new Duplex({
  write(chunk, encoding, callback) {
    console.log(chunk.toString())
    callback()
  },
  read(size) {
    this.push(String.fromCharCode(this.currentCharCode++))
    if (this.currentCharCode > 90) {
      this.push(null)
    }
  }
})

duplex.currentCharCode = 65

process.stdin.pipe(duplex).pipe(process.stdout)
```

简单来说就是把 Readable 和 Writable Stream 写在一起，就成了一个双向的 Stream，它既可写又可读，既可以消费信息、又可以生产信息。他的生产和消费过程彼此相互独立、互不影响。

### Transform Stream

Transform 虽然也被称为“双向流”，但跟 Duplex 不同，他是先读数据，转换之后再写数据。

```javascript
const { Transform } = require("stream")

const upperCaseTransformer = new Transform({
  transform(chunk, encoding, callback) {
    this.push(chunk.toString().toUpperCase())
    callback()
  }
})

process.stdin.pipe(upperCaseTransformer).pipe(process.stdout)
```

# 进程和线程

## 进程 Process

**进程**是**程序**（比如 exe）的执行实例。**程序**在CPU上执行的活动叫做**进程**。

一个进程可以创建另一个进程（父进程与子进程）。

### 多程序并发执行

单核 CPU 在一个时刻，只能做一件事情，为了让用户在同一时刻可以做多件事，就需要在不同进程中快速切换。

也就是“多程序并发执行”——多个程序在宏观上并行，微观上串行。多个进程之间会出现抢资源（比如打印机就很明显）的现象。

### 进程的两个状态

进程有 **非运行态** 与 **运行态**。分派程序将 CPU 分配给非运行态的进程，进程从而进入运行态；过一段时间之后进程暂停，又进入非运行态。

### 阻塞

在进程队列中等待执行的进程都处于 **非运行态**，一些进程(A)在等待 CPU 资源，一些进程(B)在等待 I/O 完成（比如文件读取）。如果这时候把 CPU 分配给 B 进程，B 进程并不会使用 CPU，而是继续等 I/O——我们把 B 称为 **阻塞进程**。为了避免资源的浪费，分派程序只会把 CPU 分配给 **非阻塞进程**。

## 线程 Thread

早期的面向进程设计的操作系统中，进程是程序的基本执行实体。现在的面向线程设计的系统中，进程本身不是基本运行单位，而是线程的容器。

进程是执行的基本实体，也是资源分配的基本实体——这就导致进程的创建、切换、销毁太消耗 CPU 的时间了。因此引入线程，线程作为执行的基本实体，而进程作为资源分配的基本实体。

### 概念

- CPU 调度和执行的最小单元
- 一个进程中可以有一个或多个线程
- 一个进程中的线程共享该进程的所有资源
- 进程的第一个线程叫做初始化线程
- 线程的调度可以由操作系统负责，也可以由用户自己负责

## 使用 NodeJS 操作进程

使用 `child_process` 模块可以创建子进程。
子进程的运行结果存在系统缓存之中（最大 200kb），等到子进程结束之后，主进程再用回调函数读取子进程的运行结果。

### exec

`exec(command[, options][, callback])`

```javascript
const child_process = require("child_process")
const { exec } = child_process

// exec 可以执行命令行命令，从而开启一个子进程，子进程如果有输出，会存在缓存之中，等子进程结束之后输出给 stdout
exec("ls ../", (error, stdout, stderr) => {
  console.log(error, stdout, stderr)
})
```

上面这段简单的代码表示使用 exec 可以执行命令行命令，使用命令行我们就可以开启一个子进程。

exec 也会返回一个包含流的对象：

```javascript
const child_process = require("child_process")
const { exec } = child_process

const streams = exec("ls ../")

// streams.stdout 是一个流
streams.stdout.on("data", (chunk) => {
  console.log("getData", chunk)
})

// stream.stderror 也是一个流
streams.stderr.on("data", (chunk) => {
  console.log("getError", chunk)
})
```

此外，还可以用 `util.promisify` 将其包裹为一个 Promise，避免回调地狱：

```javascript
const child_process = require("child_process")
const { exec } = child_process
const util = require("util")

const promisifiedExec = util.promisify(exec)

promisifiedExec("ls ../")
  .then((data) => {
    console.log("success", data.stdout)
  })
```

注意！如果 `exec` 执行的命令行命令可以被用户指定，就会存在注入的风险，所以通常我们不使用 `exec`。

### execFile

`execFile(file[, args][, options][, callback])`

跟 exec 用法基本相同，也可以以流的方式使用，但命令行参数需要用数组的形式传入，无法注入。

```javascript
const child_process = require("child_process")
const { execFile } = child_process

execFile("ls", ["-la", "."], (error, stdout) => {
  console.log(error)
  console.log(stdout)
})
```

#### options

options 可传可不传，他有一些比较重要的参数：

- cwd: 在哪个目录执行，默认是当前目录
- env: 环境变量，默认是 process.env
- maxBuffer: 用于暂存 stdout 和 stderr 的最大缓存区大小，默认 1024 * 1024 Byte
- shell: 指定运行的 shell

### spawn

用法跟 execFile 基本一样，但没有回调函数，只能通过流事件来获取结果，也没有最大的缓存限制（因为是流，并没有暂存 stdout 和 stderr）。

一般用 spawn，不用 execFile，因为流比较方便，且不会占用太多内存

```javascript
const child_process = require("child_process")
const { spawn } = child_process

const streams = spawn("ls", ["-la", "."])

streams.stdout.on("data", (chunk) => {
  console.log(chunk.toString())
})
```

### fork

创建一个执行 Node 脚本的子进程，可以监听他的 message 事件。

```javascript
/** main.js */
const child_process = require("child_process")
const { fork } = child_process

// 其实类似于 execFile("node", ["./child.js"])
const childProcess = fork("./child.js")

// 父进程监听子进程传来的消息
childProcess.on("message", (message) => {
  console.log("父进程得到 message", message)
})

childProcess.send({ age: 20 })

/** child.js */
console.log("This is child.js")

// 子进程监听父进程传来的消息
process.on("message", (message) => {
  console.log("子进程得到 message", message)
})

setTimeout(() => {
  process.send({ name: "harvey" })
  // 如果没有这句话，node 会因为父子进程相互监听而无法退出
  process.exit()
}, 2000)
```

## 使用 NodeJS 操作线程

操作线程需要使用 `new Worker`，而这个 API 非常新，在 v10.5.0 才加入，并且在 v11.7.0 之前使用都需要 `--experimental-worker` 来开启，因此用得会比较少，并且效率也不高。

> Workers (threads) are useful for performing CPU-intensive JavaScript operations. They do not help much with I/O-intensive work. The Node.js built-in asynchronous I/O operations are more efficient than Workers can be.

具体内容可以参考 [官方文档](https://nodejs.org/api/worker_threads.html)

# Express

## Express Generator

express 脚手架。

## use

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

## 中间件

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

## 路由

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

## 错误处理

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

## 子应用与挂载点

```javascript
const app = express()
const admin = express()

app.use("/admin", admin) // admin 是子应用，/admin 是 admin 的挂载点
```

## Express APIs

### express.xxx

- `express.static()` 指定静态资源文件夹路径
- `express.json()` 解析 Content-Type 为 application/json（或类似）的内容
- `express.urlencoded()` 解析 Content-Type 为 application/x-www-form-urlencoded（或类似）的内容
- `express.text()` 解析 Content-Type 为 text/plain（或类似）的内容
- `express.raw()` 解析 Content-Type 为 application/octet-stream （或类似）的内容

### app.xxx

- `app.set()` set 一个值，不过有一些特殊值，比如 `case sensitive routing` `env` `views` `view engine`，而且一般这些特殊值放在所有的中间件之前才会生效
- `app.get()` 可以获取到你 set 的东西
- `app.get()` `app.post()` `app.put()` `app.delete()` `app.patch()` `app.all()` 这些都是 `app.use()` 的 API 糖 
- `app.locals` 设置一些局部变量，比如 `app.locals.title = "My Title"`，然后就可以在其他地方进行读取
- `app.mountpath` 挂载路径

### request.xxx

- `request.app` 获取到全局的 app
- `request.get()` 获取请求头
- `request.param()` 获取某个参数
- `request.range()`
  1. 用 HEAD 请求某个资源，如果资源支持分片下载，就可以得到类似于 `Accept-Ranges: bytes`（支持以字节为分片下载） 的响应头。[可以参阅这里](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Range_requests)
  2. 然后用请求头 `Range` 就可以设置分片下载
  3. `request.range` 则是用于解析 `Range` 这个请求头的

### response.xxx

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

### router.xxx

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

# Koa

## Koa 与 Express 的区别

- 中间件模型不同：Koa 是 U 型，Express 是线型
- 语法特性不同，一般来说 NodeJS 7.6.0 之后才算完美支持 Koa
- Koa 没有内置中间件

## Koa 的中间件模型：U 型/洋葱型

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

### `await next()`

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

## Koa APIs

### app.xxx

- `app.env` 获取环境变量
- `app.listen()`
- `app.use()` 插入中间件
- `app.on("error", fn)` 错误处理
- `app.emit()` 触发事件

### ctx.xxx

- `ctx.req` NodeJS 封装的请求
- `ctx.request` Koa 封装的请求
- `ctx.res`
- `ctx.response`
- `ctx.state` 跨中间件分享数据
- `ctx.cookies.get` `ctx.cookies.set`
- `ctx.throw`
- `ctx.assert`

#### Request 和 Response 委托

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

#### `ctx.request.xxx` `ctx.response.xxx`

可以直接查看 [相关文档](https://koa.bootcss.com/)

# Next

NodeJS WEB 框架之 Next。
Tip：如何保存密码？

1. 存在环境变量中：用 `export xxx=` 设置环境变量，用 `process.env.xxx` 使用环境变量
2. 或者参考 [这篇文章](https://nextjs.org/docs/basic-features/environment-variables)

## 开始一个 Next.js 项目

```bash
npx create-next-app next-demo
```

## Next 的一些特性

### 使用 Link 快速导航

```jsx
// 原来这样写
<a href="xxx">点击跳转</a>
// 现在这样写
<Link href="xxx"><a>点击跳转</a></Link>
```

- 页面不会刷新，用 AJAX 请求新页面的内容
- 不会请求重复的 HTML、CSS、JS
- 自动在页面插入新内容、删除旧内容
- 省去了很多请求和解析的过程、速度极快

### 同构代码

同一份代码，在浏览器和 Node 两端都执行了，比如在组件里面写一个 console.log，会发现 Node 控制台和 Chrome 中都会有 log。

但：

- 不是所有的代码都会在两端运行，比如需要用户触发的操作，就不会在 Node 中运行
- 不是所有的 API 都能用，比如 `window` 就会报错，因为在 Node 中不存在

### `<Head>`

可以用 `<Head>` 标签书写 `title` `meta:viewport` 等。

```html
<Head>
  <title>My Blog</title>
  <link rel="icon" href="/favicon.ico" />
  <meta name="viewport" content="width=device-width,initial-scale=1,minimum-scale=1,maximum-scale=1,user-scalable=no,viewport-fit=cover" />
</Head>
```

也可以在 `pages/_app.js` 中进行全局配置：

```jsx
function MyApp({ Component, pageProps }) {
  return (
    <>
      <Head>
        <title>My Blog</title>
      </Head>
      <Component {...pageProps} />
    </>
  )
}

export default MyApp
```

切换页面的时候 MyApp 不会被销毁，MyApp 里面的代码会重新执行，但是里面的状态会保留。

### Style

在 `_app.js` 中还可以用 `import` 直接引入全局生效的 `global.css` 文件

```js
import '../styles/globals.css'
```

默认情况下都是相对路径引入，用配置指定根目录，可以参考 [这篇文档](https://nextjs.org/docs/advanced-features/module-path-aliases)

也可以在组件中书写局部的 `<style>`

```jsx
<style jsx>{`
  h1 { 
    color: red;
  }
`}</style>
```

### 静态资源

官方推荐静态资源放在 public 中，但在这里面的资源不能加 hash。 因此不推荐这样用，可以另外新建 `asstes/images` 文件夹来存放图片静态资源。
但直接这样就需要自己配置 file-loader，也就是自定义 webpack 配置：

```js
// next.config.js
module.exports = {
  webpack: (config, options) => {
    config.module.rules.push({
      test: /\.(png|jpg|jpeg|gif|svg)$/,
      use: [{
        loader: "file-loader",
        options: {
          name: "[name].[contenthash].[ext]",
          outputPath: "static",
          publicPath: "_next/static",
        }
      }]
    })
    return config
  }
}
```

也可以使用 [next-images 插件](https://github.com/twopluszero/next-images#readme) 来引入图片

## Next.js API 模式

在 api 目录下创建文件 `v1/post.tsx`

```tsx
import { NextApiHandler } from "next"
import p from "path"
import fs, { promises as fsPromise } from "fs"
import matter from "gray-matter"

export const getPosts = async () => {
  const markdownDir = p.join(process.cwd(), "markdown")
  const fileNames = await fsPromise.readdir(markdownDir)
  const posts = fileNames.map((name) => {
    const id = name.replace("/.md$/", "")
    const filePath = p.join(markdownDir, name)
    const text = fs.readFileSync(filePath, "utf-8")
    const { data: { title, date }, content } = matter(text)
    return {
      id, title, date,
    }
  })
  return posts
}

// 注意这里跟 express 和 koa 不同，没有将 next 直接暴露给我们，但是由于 next 基于 express，他其实也支持 express 类似的中间件，具体请查阅文档
const Posts: NextApiHandler = async (request, response) => {
  const posts = await getPosts()
  response.statusCode = 200
  response.setHeader("Content-Type", "application/json")
  response.write(JSON.stringify(posts))
  response.end()
}

export default Posts
```

可以试着访问 `/api/v1/posts`，这段代码只会运行在 node，不会运行在浏览器中

## Next.js 三种渲染方式

### 客户端渲染（BSR）

在浏览器上执行的渲染（Browser Side Render），类似用 JS（Vue、React）去创建 HTML

缺点：

- 白屏，而且在拿到响应之前都需要显示 loading 状态
- SEO 不友好，搜索引擎拿不到具体的数据；搜索引擎也不会执行 JS，因此看到的 HTML 信息量极少

```tsx
// pages/posts/index.tsx
import { NextPage } from "next"
import axios from "axios"
import { useCallback, useEffect, useState } from "react"

type Post = {
  id: string
  date: string
  title: string
}

const PostsIndex: NextPage = () => {
  const [posts, setPosts] = useState<Post[]>([])

  useEffect(() => {
    axios.get("/api/v1/posts").then((res) => {
      setPosts(res.data)
    })
  }, [])

  const handleClick = useCallback(() => {
    console.log("clicked")
  }, [])

  return (
    <div>
      {/* 静态内容 */}
      <h1 onClick={handleClick}>文章列表</h1>
      {/* 动态内容 */}
      {
        posts.map((item) => <div key={item.id}>{item.title}</div>)
      }
    </div>
  )
}

export default PostsIndex
```

实际上上述的静态内容是先由服务器渲染出来，然后再在浏览器上增加事件绑定的，可以看 [这篇文档](https://zh-hans.reactjs.org/docs/react-dom-server.html#rendertostring)：

服务端调用 `ReactDOMServer.renderToString(element)`，将 React 元素渲染为初始 HTML。
再在前端调用 `ReactDOM.hydrate()`，会在标记过的节点上绑定事件。

但事实上，前端也会渲染一次，检查一下前端跟服务端渲染的内容是否一致（前端渲染的目的是为了检查一致性，而不是为了替换）。
如果我们在函数组件最开始打一个 debugger，然后趁 debugger 的时间在页面上把服务端渲染出来的 HTML 改一下，再继续执行，会发现报错：

![如果前后端渲染不一致，会报错](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/next-1.png)

### 静态页面生成（SSG）

静态页面生成（Static Site Generation）是预渲染（Pre-rendering）的一种，类似于把 PHP 提前渲染成 HTML。

时机：

  - 生产环境中，静态化是在 `yarn build` 的时候实现的

优点：

  - 能解决白屏问题
  - 解决 SEO 问题
  - 前端重复渲染相同内容的问题（可以将动态内容静态化，将需要客户端渲染的动态内容提前由服务端渲染好，然后作为静态内容发给前端）

缺点： 

  - 无法生成用户相关内容（每个人看到的都是一样的）

```tsx
import { GetStaticProps, NextPage } from "next"
import axios from "axios"
import { useCallback, useEffect, useState } from "react"
import { getPosts } from "../../lib/posts"

type Post = {
  id: string
  date: string
  title: string
}

type Props = {
  posts: Post[]
}

const PostsIndex: NextPage<Props> = (props) => {
  // 这个 props 其实前后端都能拿到
  const { posts } = props
  return (
    <div>
      <h1>文章列表</h1>
      {
        posts.map((item) => <div key={item.id}>{item.title}</div>)
      }
    </div>
  )
}

// 注意这里不要忘了 export，如果不写的话，这段代码会被前端执行，会报错
// 在生产环境中，是在 build 的时候执行的
export const getStaticProps: GetStaticProps = async () => {
  // 服务端显然没必要通过 AJAX 去拿数据，直接去文件系统或者数据库拿
  const posts = await getPosts()
  return {
    props: {
      posts: JSON.parse(JSON.stringify(posts)),
    }
  }
}

export default PostsIndex
```

前端也能拿到 props，因为 Next 会将这些数据放在 HTML 中给前端：

![Next 会将数据放在 HTML 中给前端](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/next-2.png)

#### 同构代码的好处

将数据放在 HTML 中之后，前端也能很方便地使用了，其实其他后端语言也能做到，但 Next.js 更加方便，因为：

- Next 支持 jsx，写出来的代码可以与 React 无缝对接，可以只写一份代码
- Next 写出来的对象都是 JavaScript 对象，无需类型转换，如果是其他语言就需要进行额外的类型转换

#### yarn build

`yarn build` 的时候会提示你页面的类型：

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/next-3.png)

比如空心圆代表静态页面，而实心圆代表使用了 getStaticProps 进行静态化。

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/next-3.png)

可以看到编译之后，一个 SSG 页面（实心圆）变成了三个文件：

- html 是用户拿到的静态页面
- js 里面也含有静态内容，是快速导航（Link）需要用到的，因为快速导航实际上是使用 js + json 做的，而不是再去请求一次 html
- json 是 getStaticProps 得到的数据，为了方便 js 接入不同的数据，因此 js 和 json 需要分开

#### 根据路径创建静态页面

```tsx
// pages/posts/[id].tsx
import { getPost, getPostIds } from "../../lib/posts"
import { GetStaticPaths, GetStaticProps, NextPage } from "next"
type Props = {
  post: Post
}

type Params = {
  id: string
}

const PostShow: NextPage<Props> = (props) => {
  const { post } = props
  return (
    <div>
      <h1>{post.title}</h1>
      <article>
        {post.content}
      </article>
    </div>
  )
}

export default PostShow

// 声明路由：用这个来穷举静态页面的不同路径
export const getStaticPaths: GetStaticPaths = async () => {
  const idList = await getPostIds()
  const paths = idList.map((id) => {
    return {
      params: { id },
    }
  })
  return {
    paths,
    // false：当请求的 id 不再 path 里面，直接返回 404
    // true：若不存在，则依然渲染页面
    fallback: false,
  }
}

// 通过 context 中的 params 拿到上一个函数穷举出的路径，也就是我们的 id，然后借由 id 去获取内容并生成静态文件
export const getStaticProps: GetStaticProps<Props, Params> = async (context) => {
  const id = context.params!.id
  const post = await getPost(id)
  return {
    props: {
      post,
    }
  }
}
```

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/next-5.png)

可以看到生成来的文件中只有一个 JS，但是有两个 JSON 和两个 HTML 文件，这就是根据不同路由的到的不同页面和数据。

### 服务端渲染（SSR）

服务端渲染（Server Side Render）也是预渲染的一种，类似于 PHP、Python、Ruby、Java 后台的基本功能。
比如当需要通过不同的用户 id 等渲染不同的内容，这时候就较难提前做静态化，这个时候就可以使用 SSR，由服务端渲染好首屏，再下拉刷新。

时机：

  - 无论是在开发环境还是生产环境，都是每次请求来到之后运行的

优点：

  - 能解决白屏问题
  - 解决 SEO 问题
  - 可以生成跟用户有关系的内容

缺点：

  - 无法获取客户端信息，比如浏览器窗口大小等

```tsx
import { GetServerSideProps, NextPage } from "next"
import { IBrowser, UAParser } from "ua-parser-js"
import { useEffect, useState } from "react"

type Props = {
  browser: IBrowser
}

const Home: NextPage<Props> = (props) => {
  const { browser } = props
  const [clientWidth, setClientWidth] = useState(0)
  // 如果想要获取浏览器窗口的大小，不能使用 SSR，只能使用客户端渲染，这样的写法保证了下面的代码只会在浏览器中运行，防止 node 报错
  useEffect(() => {
    setClientWidth(document.documentElement.clientWidth)
  }, [])
  return (
    <>
      <h1>你的浏览器是 {browser.name}</h1>
      <h1>你的窗口宽度是 {clientWidth}</h1>
    </>
  )
}

export default Home

// 每次请求的时候才会执行
export const getServerSideProps: GetServerSideProps = async (context) => {
  // context 中有 req 和 res，一般只需要 req
  const ua = context.req.headers["user-agent"]
  const result = new UAParser(ua).getResult()
  return {
    props: {
      browser: result.browser,
    }
  }
}
```

### 三种渲染应该如何选择

1. 有动态内容吗？

  - 有，跳到 2
  - 没有，直接写 HTML 即可

2. 跟客户端相关（比如获取窗口大小等）吗？

  - 相关，BSR
  - 无关，跳到 3

3. 我的响应跟用户或请求相关吗？

  - 相关，SSR（getServerSideProps）或 BSR
  - 无关，SSG（getStaticProps） 或 SSR 或 BSR

# TypeORM

TypeORM 是一个 Node 中对 TypeScript 支持比较好的关系对象映射，支持关联、事务、数据库迁移，同类产品还有 Sequelize。
Mac 和 Linux 可以用 & 同时执行两个命令，Windows 可以用 concurrently 这个 npm 库。

## 启动 Postgresql 数据库

在项目目录里面新增 `blog-data` 目录，并将其添加进 `.gitignore` 中。

然后启动 postgresql：

```bash
docker run -v "$PWD/blog-data":/var/lib/postgresql/data -p 5432:5432 -e POSTGRES_USER=blog -e POSTGRES_HOST_AUTH_METHOD=trust -d postgres
```

将上述命令的 `-e POSTGRES_HOST_AUTH_METHOD=trust` 替换成 `-e POSTGRES_PASSWORD=123456` 就可以设置密码

然后可以这样进入 postgresql：

```bash
docker exec -it id bash
psql -U username -W
```

然后可以执行 psql 的命令：

```bash
\l # list databases
\c # connect to a databases
\dt # display tables
```

## 创建 database

由于 TypeORM 没有为我们提供单纯创建数据库的 API，我们需要进入数据库用 SQL 语句来进行创建：

```postgresql
CREATE DATABASE blog_production ENCODING 'UTF8' LC_COLLATE 'en_US.utf8' LC_CTYPE 'en_US.utf8';
```

一般来说需要创建三个数据库：development、test、production

## 安装 TypeORM

[官方文档在这里](https://typeorm.io/#/)。

注意！当按照官方文档操作，运行命令 `typeorm init --database postgres` 之后，`.gitignore` 、 `package.json` 和 `tsconfig.json` 将会被更改，这时需要检查他的更改并酌情处理！

比如当我们在使用 Next 搭配 TypeORM 的时候，会遇到问题是 Next 使用 Babel 处理 TypeScript，而 TypeORM 则推荐使用 ts-node，这样就冲突了：

1. 先从 `package.json` 中删除命令擅自添加的 `ts-node`
2. 再想办法运行 `src/index.ts`

### 自行使用 babel 来进行编译，然后再用 node 运行编译好的 js

TypeORM 想要我们直接运行 `src/index.ts`，但是我们需要安装 `@babel/cli` 之后使用命令 `npx babel ./src --out-dir dist --extensions ".ts,.tsx"` 来运行。

这时可能会遇到报错 `Support for the experimental syntax 'decorators-legacy' isn't currently enabled`。

解决方案可以看[这篇文章](https://stackoverflow.com/questions/52262084/syntax-error-support-for-the-experimental-syntax-decorators-legacy-isnt-cur)。
同时可以参考[next 文档](https://nextjs.org/docs/advanced-features/customizing-babel-config)来配置 `.babelrc`

然后我们就可以在 dist 里面找到编译好的 `index.js`，用 node 直接运行他，但是注意需要修改 `ormconfig.json` 里面的配置，尤其是这一部分：

```json
{
  "entities": [
    "dist/entity/**/*.js"
  ]
}
```

### 让 Next 帮我们编译，帮我们运行

我们可以将上面 TypeORM 生成的 `src/index.ts` 放到 `pages` 目录下，这样 Next 就会在 请求(开发)/打包(生产) 的时候帮我们编译并执行。同样需要注意修改 `ormconfig.json`。

## synchronize 功能

同步功能可以在 `ormconfig.json` 里面的 `synchronize` 项进行配置。

如果打开 synchronize，每次 `createConnection` 的时候都会将 entity 里面的表同步到我们的数据库中，这样就很可能导致我们在修改 User 的时候把数据删掉，这在生产环境肯定是不允许的。因此我们可以在一开始就把这个功能给关掉。

## 通过 migration 创建表

```bash
npx typeorm migration:create -n CreatePost
```

然后对其创建的 `/src/migration/[timestamp]-CreatePosts.ts` 进行修改：

```typescript
import { MigrationInterface, QueryRunner, Table } from "typeorm"

export class CreatePosts1620523637192 implements MigrationInterface {

  public async up(queryRunner: QueryRunner): Promise<void> {
    // 升级数据库
    return await queryRunner.createTable(new Table({
      name: "posts",
      columns: [
        { name: "id", type: "int", isPrimary: true, isGenerated: true, generationStrategy: "increment" },
        { name: "title", type: "varchar" },
        { name: "content", type: "text" },
      ],
    }))
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    // 降级数据库
    return await queryRunner.dropTable("posts")
  }

}
 
```

然后同样的，我们需要将其编译成 js，然后配置 `ormconfig.json` 中的 `migrations` 路径，然后再执行：

```bash
npx typeorm migration:run
```

他会执行 `up` 中的代码，于是就创建了一个表。

当发生错误需要撤销这次 migration 的时候，执行：

```bash
npx typeorm migration:revert
```

他会执行 `down` 中的代码，在这里就会删除这个表。

## 将数据映射到实体从而操作他

在数据库中创建一个表之后，如果想操作他，我们通常需要他映射到 实体(Entity) 上，可以通过这个命令达成：

```bash
npx typeorm entity:create -n Post
```

需要将 Entity Post 对应上数据库里面的信息：

```typescript
import { Column, Entity, PrimaryGeneratedColumn } from "typeorm"

@Entity("posts")
export class Post {

  @PrimaryGeneratedColumn("increment")
  id: number

  @Column("varchar")
  title: string

  @Column("text")
  content: string
}
```

可以使用 [EntityManager](https://typeorm.io/#/entity-manager-api) 和 [Repository](https://typeorm.io/#/repository-api) 这两种使用实体的方式，他们只是封装思路不同而已。

## Seed

也叫数据填充，我们可以通过 seed 脚本来构造数据。

## 创建数据表关联

可以参考 [这篇文档](https://typeorm.io/#/relations)

```typescript
@Entity()
export class User {
  //... 其他字段
  
  @OneToMany(type => Post, post => post.author)
  posts: Post[]
}
```

