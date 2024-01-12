---
title: NodeJS
date: 2020-03-09 23:21:35
categories:
  - - 全栈
tags:
  - FX
---

NodeJS 是一个 JavaScript 的一个运行时，让 JavaScript 拥有调用操作系统接口的能力，使得 JavaScript 可以作为命令行脚本语言和后端开发语言使用。

NodeJS 中提供了 Stream 的概念用于读写数据，减少内存占用。

Links:

- [Express](https://expressjs.com/)：一个使用线型中间件模型的后端框架，他提供了很多内置的中间件
- [Koa](https://koajs.com/)：一个使用 U 型中间件模型的后端框架，他相交 Express 更加轻便
- [[NextJS]]：一个全栈 Web 框架
- [[TypeORM]]：一个处理数据库对象关系映射的库

<!-- more -->

# 一些有用的工具

- commander：可以很方便地写出命令行，有多种语言的版本。
- node-dev：文件更新时自动重启的 node，不宜在生产环境使用。
- ts-node：让 node 支持直接运行 TypeScript 代码，不宜在生产环境使用。
- ts-node-dev：node-dev + ts-node，不宜在生产环境使用。
- 注意用 ts 开发时，需要安装 @types/node
- tsc --init 命令会自动创建 `tsconfig.json` 文件

# Node.js 是什么

- 不是 web 框架，不能把 Node.js 与 Flask 或 Spring 对比
- 不是编程语言，不能与 Python 或 PHP 对比
- 是一个平台，将多种技术组合起来，让 JavaScript 也能调用系统接口、开发后端应用
  - V8 引擎
  - libuv
  - C/C++ 实现的 c-ares、http-parser、OpenSSL、zlib 等

# Node.js 技术架构

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
- V8 自带 Event Loop，但 Node.js 基于 libuv 自己做了一个 [[Event Loop]]

# 使用 NodeJS 操作进程

使用 `child_process` 模块可以创建子[[Process|进程]]。

子进程的运行结果存在系统缓存之中（最大 200kb），等到子进程结束之后，主进程再用回调函数读取子进程的运行结果。

## exec

`exec(command[, options][, callback])`

```javascript
const child_process = require("child_process")
const { exec } = child_process

// exec 可以执行命令行命令，从而开启一个子进程，子进程如果有输出，会存在缓存之中，等子进程结束之后输出给 stdout
exec("ls ../", (error, stdout, stderr) => {
  console.log(error, stdout, stderr)
})
```

上面这段简单的代码表示使用 `exec` 可以执行命令行命令，使用命令行我们就可以开启一个子进程。

`exec` 也会返回一个包含流的对象：

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

## execFile

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

### options

options 可传可不传，他有一些比较重要的参数：

- cwd: 在哪个目录执行，默认是当前目录
- env: 环境变量，默认是 process.env
- maxBuffer: 用于暂存 stdout 和 stderr 的最大缓存区大小，默认 1024 * 1024 Byte
- shell: 指定运行的 shell

## spawn

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

## fork

创建一个执行 Node 脚本的子进程，可以监听他的 `message` 事件。

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

# 使用 NodeJS 操作线程

操作[[Thread|线程]]需要使用 `new Worker`，而这个 API 非常新，在 v10.5.0 才加入，并且在 v11.7.0 之前使用都需要 `--experimental-worker` 来开启，因此用得会比较少，并且效率也不高。

> Workers (threads) are useful for performing CPU-intensive JavaScript operations. They do not help much with I/O-intensive work. The Node.js built-in asynchronous I/O operations are more efficient than Workers can be.

具体内容可以参考 [官方文档](https://nodejs.org/api/worker_threads.html)
