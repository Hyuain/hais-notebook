---
title: Disjoint-Set Data Structure
date: 2022-02-05 15:49:12
categories:
  - - 计算机
mathjax: true
tags:
  - CX
---

> 由于支持合并和查询两种操作，并查集也称 Union-Find Data Structure 或 Merge-Find Set。
>
> 在需要判断两个元素是否在同一个集合时，可以使用并查集。

并查集实现为一个森林，**其中的每棵树表示一个集合**，树中的节点对应集合中的元素。**如果两个节点在同一个根节点下，则这两个节点属于一个集合。**

下面是一个内部元素全部为数字类型的并查集的 JavaScript 实现，**注意集合中元素是不重复的，所有种类的集合都有这个性质，并查集也不例外**：

```typescript
class DisjointSet {
  // 该数组记录了每个节点的父子关系，这个数组表示了一个森林
  private parent: number[] = []
  
  // 初始化并查集，将每一个节点的父结点设置为自己（每个节点为一个集合）
  // n 为该森林的最大容量
  constructor(n = 1005) {
    for (let i = 0; i < n; i++) {
      parent[i] = i
    }
  }
  
  // 一直递归向上找到该元素的根节点
  public find(x: number) {
    return parent[x] === x
      ? x
      : parent[x] = find(parent[x])  // 顺便进行路径压缩，让节点与根节点直接相连，提高性能
  }
   
  // 将 y 所在的集合与 x 所在的集合合并
  // 也就是将 x 的根节点作为 y 的根节点的父结点
  public unite(x: number, y: number) {
    parent[find(x)] = find(y)
  }
  
  // 判断是否在同一个集合
  // 也就是判断他们的根节点是否相同
  public isSame(x: number, y: number) {
    return find(x) === find(y)
  }
}
```

在合并的时候除了路径压缩以外还有另外一种方法，称为 **启发式合并** 或 **按秩合并**。秩就是树的高度，所以这种方法简单来说就是将矮的那棵树合并到高的那棵树中，这样合并的结果会比较矮。注意在使用启发式合并的时候就不能使用路径压缩了，因为这样秩的记录就会不准。

{% note Code %}

```typescript
class DisjointSet {
  // 该数组记录了每个节点的父子关系，这个数组表示了一个森林
  private parent: number[] = []
  // 记录了树的高度，即秩
  private rank: number[] = []
  
  // 初始化并查集，将每一个节点的父结点设置为自己（每个节点为一个集合）
  // n 为该森林的最大容量
  constructor(n = 1005) {
    for (let i = 0; i < n; i++) {
      parent[i] = i
      rank[i] = 1  // 也可以不写
    }
  }
  
  // 一直递归向上找到该元素的根节点
  public find(x: number) {
    return parent[x] === x
      ? x
      : find(parent[x])  // 不能进行路径压缩
  }
   
  // 将 y 所在的集合与 x 所在的集合合并
  public unite(x: number, y: number) {
    x = find(x)
    y = find(y)
    // 将秩小的合并入秩大的集合中
    if (rank[x] <= rank[y]) {
      parent[x] = y
    } else {
      parent[y] = x
    }
    // 秩相等的时候，由于是 x 合并入 y 的，所以将 y 的秩 + 1
    if (rank[x] == rank[y] && x !== y) {
      rank[y]++
    }
  }
  
  // 判断是否在同一个集合
  // 也就是判断他们的根节点是否相同
  public isSame(x: number, y: number) {
    return find(x) === find(y)
  }
}
```

{% endnote %}
