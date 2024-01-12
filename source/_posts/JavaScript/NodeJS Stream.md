---
title: NodeJS Stream
date: 2020-03-09 23:21:35
categories:
  - - 全栈
---

数据流叫 Stream，每次写的小数据叫 chunk，产生数据的一段叫 source，得到数据的一段叫 sink。

# Stream

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

## 不用 Stream 读文件

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

## 用 Stream 读文件

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

使用 Stream，NodeJs 的内存占用会更低。

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


# Stream 的分类

- Readable，可读，数据的生产者
	- `paused` 和 `flowing` 两个状态
	- 默认处于 `paused`；添加 data 事件监听，就会变成 `flowing`；删掉 data 事件监听就又变成 `paused`
	- `pause()` 可以让他变为 `paused`，`resume()` 可以让他变为 `flowing`
- Writable，可写，数据的消费者
	- 有 `drain` 和 `finish` 两个重要事件
	- `drain`：`writable.write(chunk)` 将会返回一个 `boolean`，如果 `boolean` 为 `false`，这个 stream 堵车了，这个时候就需要等待。当 writable 监听到 `drain` 事件，表示不堵车了，可以继续传输数据
	- `finish`：调用 `stream.end()` 方法，并且缓冲区的数据都已经传给底层系统后会触发
- Duplex，可读可写（双向，读和写的内容无关）
- Transform，可读可写（变化，比如先把内容读出来，再写到别的地方）

## Examples

### Writable Stream

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

### Readable Stream

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

### Duplex Stream

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

### Stream

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
