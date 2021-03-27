---
title: Child Process
date: 2021-03-26 22:59:12
tags:
  - 入门
categories:
  - [全栈, NodeJS]
---

进程、线程，进程控制与线程控制。

<!-- more -->

# 进程 Process

**进程**是**程序**（比如 exe）的执行实例。**程序**在CPU上执行的活动叫做**进程**。

一个进程可以创建另一个进程（父进程与子进程）。

## 多程序并发执行

单核 CPU 在一个时刻，只能做一件事情，为了让用户在同一时刻可以做多件事，就需要在不同进程中快速切换。

也就是“多程序并发执行”——多个程序在宏观上并行，微观上串行。多个进程之间会出现抢资源（比如打印机就很明显）的现象。

## 进程的两个状态

进程有 **非运行态** 与 **运行态**。分派程序将 CPU 分配给非运行态的进程，进程从而进入运行态；过一段时间之后进程暂停，又进入非运行态。

## 阻塞

在进程队列中等待执行的进程都处于 **非运行态**，一些进程(A)在等待 CPU 资源，一些进程(B)在等待 I/O 完成（比如文件读取）。如果这时候把 CPU 分配给 B 进程，B 进程并不会使用 CPU，而是继续等 I/O——我们把 B 称为 **阻塞进程**。为了避免资源的浪费，分派程序只会把 CPU 分配给 **非阻塞进程**。

# 线程 Thread

早期的面向进程设计的操作系统中，进程是程序的基本执行实体。现在的面向线程设计的系统中，进程本身不是基本运行单位，而是线程的容器。

进程是执行的基本实体，也是资源分配的基本实体——这就导致进程的创建、切换、销毁太消耗 CPU 的时间了。因此引入线程，线程作为执行的基本实体，而进程作为资源分配的基本实体。

## 概念

- CPU 调度和执行的最小单元
- 一个进程中可以有一个或多个线程
- 一个进程中的线程共享该进程的所有资源
- 进程的第一个线程叫做初始化线程
- 线程的调度可以由操作系统负责，也可以由用户自己负责

# 使用 NodeJS 操作进程

使用 `child_process` 模块可以创建子进程。
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

# 使用 NodeJS 操作线程

操作线程需要使用 `new Worker`，而这个 API 非常新，在 v10.5.0 才加入，并且在 v11.7.0 之前使用都需要 `--experimental-worker` 来开启，因此用得会比较少，并且效率也不高。

> Workers (threads) are useful for performing CPU-intensive JavaScript operations. They do not help much with I/O-intensive work. The Node.js built-in asynchronous I/O operations are more efficient than Workers can be.

具体内容可以参考 [官方文档](https://nodejs.org/api/worker_threads.html)