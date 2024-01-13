---
title: Graph
date: 2022-02-05 15:49:12
categories:
  - - 计算机
mathjax: true
tags:
  - CX
---

# Topological Sorting

> 拓扑排序（Topological Sorting）指的是给有向无环图（Directed Acyclic Graph, DAG）的所有节点排序。

下面用 [LeetCode.207 Course Schedule](https://leetcode.cn/problems/course-schedule) 为例介绍拓扑排序。该题有 BFS 和 DFS 两种解法，通过拓扑排序看是否有环，有环则该课程表无法实现。

> 此外还有还可参考 [LeetCode.210 Course Schedule II](https://leetcode.cn/problems/course-schedule-ii)，该题需要输出顺序

两种解法都需要借助 **邻接表（Adjacency List）**，或者 **逆邻接表（Inverse Adjacency List）**来降低时间复杂度，比如：

```javascript
const edges = [[0, 1], [0, 2], [1, 3]]
const adjacency = [
  [],
  [0],
  [0],
  [1],
]
```

上面是一个 **逆邻接表** 的演示，记录了 **指向该顶点的所有边**，而 **邻接表** 则是会记录 **从该顶点出发的所有边**。

## BFS TopSort

BFS 算法除了逆邻接表以外，还需要 **入度表（Indegrees List）**，入度表将会记录每个顶点的入度（指向该顶点的边数）。

```typescript
function canFinish(n: number, prerequisites: number[][]): boolean {
  // 入度表
  const indegrees = new Array(n).fill(0)
  // 邻接表
  const adjacency = new Array(n).fill(0).map((_) => [])
  
  // 构建入度表和邻接表
  for (let i = 0; i < prerequisites.length; i++) {
    const [cur, pre] = prerequisites[i]
    indegrees[cur]++
    adjacency[pre].push(cur)
  }
  
  // BFS 需要借助队列
  const queue = []
  
  // 先取得所有入度为 0 的顶点，他们是拓扑排序的开始
  for (let i = 0; i < indegrees.length; i++) {
    if (indegrees[i] === 0) {
      queue.push(i)
    }
  }
  
  // BFS 拓扑排序
  while (queue.length) {
    const pre = queue.shift()
    n--
    for (let i = 0; i < adjacency[pre]; i++) {
      const cur = adjacency[i]
      indegrees[cur]--
      if (indegrees[cur] === 0) {
        queue.push(cur)
      }
    }
  }
  
  // 如果 n 还有剩下，那么说明有的课程永远无法到达（成环了）
  return !n
}
```

## DFS TopSort

DFS 算法不需要借助入度表，但是需要借助 `flags` 数组来对每个节点的状态进行标记，`flags[i]` 代表节点 `i` 的状态：

- 未被 DFS 访问：`flags[i] === 0`；
- 已被其他节点启动的 DFS 访问：`flags[i] === -1`；
- 已被当前节点启动的 DFS 访问：`flags[i] === 1`。

```typescript
function canFinish(n: number, prerequisites: number[][]): boolean {
  // 邻接表
  const adjacency = new Array(n).fill(0).map((_) => [])
  // 记录状态
  const flags = new Array(n).fill(0)
  
  // 构建邻接表
  for (let i = 0; i < prerequisites.length; i++) {
    const [cur, pre] = prerequisites[i]
    adjacency[pre].push(cur)
  }
  
  // 如果成环了，返回 false
  function dfs(start: number): boolean {
    // 该节点早已被别人访问过
    if (flags[start] === -1) {
      return true
    }
    // 该节点我之前访问过，成环了
    if (flags[start] === 1) {
      return false
    }
    // 将该节点标记为我方问过了
    flags[start] = 1
    for (let i = 0; i < adjacency[i].length; i++) {
      if (!dfs(adjacency[i])) {
        return false
      }
    }
    // 回溯，调整节点访问状态为已被别人访问过，方便其他节点进行判断
    flags[start] = -1
    return true
  }
  
  // 对每个节点进行 DFS
  for (let i = 0; i < n; i++) {
    if (!dfs(i)) {
      return false
    }
  }
  
  return true
}
```


# Search

跟[[Binary Tree|树]]类似，图有深度优先搜索（DFS）和广度优先搜索（BFS）两种搜索方式。深度优先搜索就是先沿着某一条路径探索到底，广度优先就是一层一层地先将当前节点可触达的节点处理完成。

## Deep First Search

DFS 就像 Galgame 一样，我会先打到某个终点，再从最后一个分歧点开始探索新的分支。

**回溯算法实际上就是 DFS 的过程**，DFS 在到达终点之后会从最后一步逐步撤销再探索新的路径，**这个撤销的过程就被成为回溯**。

```typescript
function dfs(params) {
  if (finish()) {
    saveResults()
    return
  }
  for (const child of children) {
    process(child)
    dfs(child, newParams)
    rollback()
  }
}
```

## Breadth First Search

BFS 是一圈一圈地搜索，比较适合解决两个点之间最短路径的问题。因为由于是一圈一圈地搜索，第一次搜到的时候，就是路径最短的时候。

**广度优先搜索习惯上用队列来实现**（不过用栈也可以，用栈的话就是一圈顺时针、一圈逆时针地搜）。比树的广度优先搜索需要额外注意的一点是，由于可能访问到之前的节点，所以需要将已访问过的节点标记一下，防止重复访问。

```typescript
function bfs() {
  const queue = [start]
  visited = new Set()
  while (queue.length) {
    const current = queue.shift()
    if (visited.has(current.key)) {
      continue
    }
    visited.add(current.key)
    for (const neighbor of neighbors) {
      queue.push(neighbor)
    }
  }
}
```

# Shortest Path

## Dijkstra

> Dijkstra（/ˈdikstrɑ/或/ˈdɛikstrɑ/）算法由荷兰计算机科学家 E. W. Dijkstra 于 1956 年发现，1959 年公开发表。是一种求解 **非负权图** 上单源最短路径的算法。

该算法会记录当前已知的最短路径，并在寻找到更短的路径时更新。

一旦找到最短路径，那个节点从未访问集合中被删除，并添加到已找到最短路径的集合中。

重复查找的过程，直到所图中所有的节点都已经添加到路径中，这样就可以得到源节点出发访问其他所有节点的最短路径方案。

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Dijkstra-1.png)

**目标**：如图所示我们从 `0` 出发，查找其他所有节点的最短路径。

**初始化**

1. 初始化距离列表：`distance = [0, Infinity, Infinity, Infinity, Infinity, Infinity]`。

2. 初始化未访问节点集合：`unvisitied = new Set([1, 2, 3, 4, 5, 6])` 和已找到最短路径的集合 `shortest = new Set([0])`，当集合为空时即为计算完成（从节点 `0` 出发可以直接将 `0` 标记为已访问）。

```javascript
distance:   [0, Infinity, Infinity, Infinity, Infinity, Infinity, Infinity]
unvisited:  Set({1, 2, 3, 4, 5, 6})
shortest:   Set({0})
```

**第一次循环**

1. 检查 `0` 到相邻节点 `1` 和 `2` 的距离， 并更新 `distance` 为对应的权重 `[0, 2, 6, Infinity, Infinity, Infinity]`。
2. 根据已知的 `distance` 选择距离源节点最近的节点，即节点 `1`，把他标记为已访问，并添加到路径中。`unvisitied.delete(1); shortest.add(1)`。

```javascript
istance:   [0, 2, 6, Infinity, Infinity, Infinity, Infinity]
unvisited:  Set({2, 3, 4, 5, 6})
shortest:   Set({0, 1})
```

**第二次循环**

1. 检查已经在最短路径的集合中包含的节点的相邻节点 `2` 和 `3`，此时需要计算源节点到 `3` 的距离，由于已经在路径中的集合只有 `0` 和 `1`，因此只能通过 `0 -> 1 -> 3` 进行计算，结果为 7。更新 `distance` 为 `[0, 2, 6, 7, Infinity, Infinity]`。
2. 选择已知的与源节点最近的未访问节点，即节点 `2`，将其标记为已访问并加入路径。

```javascript
distance:   [0, 2, 6, 7, Infinity, Infinity, Infinity]
unvisited:  Set({3, 4, 5, 6})
shortest:   Set({0, 1, 2})
```

**第三次循环**

1. 检查已经在最短路径集合中包含的节点的相邻节点 `3`，可选路径为 `0 -> 1 -> 3` 和 `0 -> 2 -> 3`，结果是前者更小，因此保持 `distance` 中的 7 不变。
2. 将节点 `3` 标记为已访问并添加至路径中。

```javascript
distance:   [0, 2, 6, 7, Infinity, Infinity, Infinity]
unvisited:  Set({4, 5, 6})
shortest:   Set({0, 1, 2, 3})
```

**第四次循环**

1. 重复上述过程，检查节点 `4` 和 `5`，节点 `4` 的最短路径是 `0 -> 3 -> 4`，距离为 17；节点 `5` 的最短路径是 `0 -> 3 -> 5`，距离为 22，用这个结果更新 `distance`。
2. 将 `4` 标记为已访问并添加至路径。

```javascript
distance:   [0, 2, 6, 7, 17, 22, Infinity]
unvisited:  Set({5, 6})
shortest:   Set({0, 1, 2, 3, 4})
```

**第五次循环**

1. 检查 `5` 和 `6`，对于 `5` 可选 `0 -> 3 -> 5` 和 `0 -> 4 -> 5`；对于 `6` 可选 `0 -> 4 -> 6`，更新 `distance`。
2. 将 `6` 标记为已访问并添加至路径。

```javascript
distance:   [0, 2, 6, 7, 17, 22, 19]
unvisited:  Set({5})
shortest:   Set({0, 1, 2, 3, 4, 6})
```

**第六次循环**

1. 检查 `5`，可选 `0 -> 3 -> 5` `0 -> 4 -> 5` 和 `0 -> 6 -> 5`，更新 `distance`。
2. 将 `5` 标记为已访问并添加至路径。

```javascript
distance:   [0, 2, 6, 7, 17, 22, 19]
unvisited:  Set({})
shortest:   Set({0, 1, 2, 3, 4, 6, 5})
```

算法代码如下：

```javascript
function dijkstra(edges, n, source) {
  const distance = new Array(n).fill(Infinity)
  distance[source] = 0
  // TBC 
}
```

## Floyd

Floyd 算法是一种非常简单的求任意两个节点之间最短路径的算法。他的时间复杂度是 O(n^3)，但比较容易实现。适用于任何有最短路径的图，不管有向无向，边权正负，但不能有负环（不存在最短路）。比如 [LeetCode.1462 Course Schedule IV](https://leetcode.cn/problems/course-schedule-iv) 可以用 Floyd 算法来解决。

Floyd 算法实际上是多维动态规划算法：

- 定义数组 `f[k][x][y]`，表示节点 `x` 到节点 `y`，只允许经过 `1` 到 `k` 的最短路（显然节点 `f[n][x][y]` 即为 `x` 到 `y` 的最短路）；

- 初始化 `f[0][x][y]`，当 `x` 等于 `y` 时为 `0`，`x` 与 `y` 直接相连时为他们的边权，否则为 `Infinity`；
- `f[k][x][y] = min(f[k - 1][x][y], f[k - 1][x][k] + f[k - 1][k][y])`，考虑经过和不经过 `k` 两种情况。

```javascript
for (k = 1; k <= n; k++) {
  for (x = 1; x <= n; x++) {
    for (y = 1; y <= n; y++) {
      f[k][x][y] = Math.min(f[k - 1][x][y], f[k - 1][x][k] + f[k - 1][k][y])
    }
  }
}
```

可以看出数组的第一维没什么用，可以用滚动数组优化掉：

```javascript
for (k = 1; k <= n; k++) {
  for (x = 1; x <= n; x++) {
    for (y = 1; y <= n; y++) {
      f[x][y] = Math.min(f[x][y], f[x][k] + f[k][y])
    }
  }
}
```
