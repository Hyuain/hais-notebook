---
title: FAQs：数据结构
date: 2020-02-10 20:45:38
tags:
  - 入门
  - 收集
  - FAQ
categories:
  - [前端, 其他]
---

关于数据结构的一些问题与答案。

<!-- more -->

# 哈希表

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

# 图

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