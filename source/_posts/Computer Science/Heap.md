---
title: Heap
date: 2022-02-05 15:49:12
categories:
  - - 计算机
mathjax: true
tags:
  - CX
---

# Priority Queue & Heap

**优先级队列（Priority Queue）是一个抽象数据类型（Abstract Data Type, ADT）**，他支持插入元素（Enqueue）、删除具有最大或最小优先级的元素（Dequeue）。他可以用很多数据结构来实现，比如堆、[[Binary Tree#Binary Search Tree|二叉搜索树]]、[[Array|数组]]等。

**堆（Heap）是一个具体的数据结构**，可以用来实现优先级队列，他实际上就是一棵[[Binary Tree#Complete Binary Tree|完全二叉树]]（最后一层可能不满），并且满足排序特点：大顶堆（Max-Heap）中父结点的优先级高于子结点，小顶堆（Min-Heap）中则相反。堆有很多变种，比如二叉堆、斐波那契堆、二项堆等，他们都有自己的特点。

JavaScript 中并没有提供原生的堆函数，而完全二叉树可以高效地用数组表示，因此我们可以手动用数组实现一个堆：

{% note Code %}

```typescript
class Heap<T = any> {
  // 二叉堆实际上是一棵树，但保证了节点的顺序
  private tree: T[] = []
  private compareFn: (a: T, b: T) => number

  // compareFn 指定堆的特性，按照什么优先级构建堆，返回正数即意味着在 a 应当在 b 下面
  // 比如当 a, b 均为 number，传入 a - b 就意味着 a > b 时，a 应当在 b 的下面，即为小顶堆
  // arr 可以指定通过特定的数组进行堆的初始化
  constructor(compareFn: (a: T, b: T) => number, arr?: T[]) {
    this.compareFn = compareFn
    if (arr) {
      this.build(arr)
    }
  }

  // 向堆底添加元素
  public push(item: T) {
    // 先将新的 HeapItem 直接放到堆的最底部
    this.tree.push(item)
    // 将刚刚 push 的 HeapItem 往上浮
    this.up(this.size() - 1)
  }

  // 弹出堆顶元素
  public pop(): T {
    if (this.size() <= 1) {
      return this.tree.pop()
    }
    const top = this.top()
    // 将堆底最后一个元素弹出，替代掉堆顶的元素
    const bottom = this.tree.pop()
    this.tree[0] = bottom
    // 将新的堆顶下沉
    this.down(0)
    return top
  }

  public top() {
    return this.tree[0]
  }

  public size() {
    return this.tree.length
  }

  // 返回正数时，指示 index1 应该在 index2 的下方，应该把 index1 往下沉，反之则代表 index2 需要往下沉
  // 注意这与这是大顶堆还是小顶堆是没关系的，不管是大顶堆还是小顶堆，都是正数表示 index1 需要往下沉
  private compare(index1: number, index2: number): number {
    // 讨论越界情况，保证 undfined 值总是在最下面
    // 当 index1 越界了，应该把 index1 所代表的 undefined 值往下沉
    if (this.tree[index1] === undefined) { return 1 }
    // 当 index2 越界了，应该把 index2 所代表的 undefined 值往下沉
    if (this.tree[index2] === undefined) { return -1 }
    // 正常情况由传入的 compareFn 决定
    // 比如如果是小顶堆，应该写作类似 (a, b) => a - b，这样当 a 的值 > b 的值时，a 就会往下沉
    return this.compareFn(this.tree[index1], this.tree[index2])
  }

  private down(index: number) {
    // left 是左孩子 index，left + 1 是右孩子 index
    let left = this.leftIndex(index)
    // 找到要下沉到哪里，如果左孩子比右孩子更低，那么应该跟右孩子进行比较
    // 否则可能出现操作完成后的节点比左孩子高、比右孩子低的情况（本应该节点比两个孩子都高）
    let searchChild = this.compare(left, left + 1) > 0 ? left + 1 : left
    // 注意 compare 的传参，因为要把 index 向下沉
    while (searchChild < this.size() && this.compare(index, searchChild) > 0) {
      [this.tree[index], this.tree[searchChild]] = [this.tree[searchChild], this.tree[index]]
      index = searchChild
      left = this.leftIndex(index)
      searchChild = this.compare(left, left + 1) > 0 ? left + 1 : left
    }
  }

  private up(index: number) {
    // parent 是父结点的 index
    let parent = this.parentIndex(index)
    // 注意 compare 的传参，因为 index 是要从最底下往上浮
    while (parent >= 0 && this.compare(parent, index) > 0) {
      [this.tree[index], this.tree[parent]] = [this.tree[parent], this.tree[index]]
      index = parent
      parent = this.parentIndex(index)
    }
  }

  // 通过数组构建堆
  private build(arr: T[]) {
    this.tree = [...arr]
    for (let i = Math.floor(arr.length / 2) - 1; i >= 0; i--) {
      this.down(i)
    }
  }

  private leftIndex(index: number) {
    return 2 * index + 1
  }

  private parentIndex(index: number) {
    return Math.floor((index - 1) / 2)
  }
}
```

{% endnote %}

# Heap Sort

**堆排序实际上会经历两个过程：**

1. 由数组构建大顶堆/小顶堆
2. 将堆顶不停 pop 出来，因为每次 pop 完成之后堆内部都会重建以保证其大顶堆/小顶堆的特性，因此每次 pop 出来的一定是最大值/最小值

**堆的构建过程有两种方案：**

1. **自底向上（Bottom-Up）**建堆，使用 Floyd 算法。先把用于构建堆的数组存入堆中，再从最后一个非叶节点（n / 2 - 1）遍历至第 0 个元素，将遍历到的每个元素向下沉，也是上面代码示例中所使用的方法。
   令 $h$ 为堆的总高度，$h_i$ 为高度为第 $i$ 层的高度，即 $h - i$；$n_i$ 为第 $i$ 层的节点数量，即 $2^i$。那么 **自底向上建堆的时间复杂度为 O(n)**：

$$
T(n) = \sum_{i=0}^{h} n_i h_i \\
= \sum_{i=0}^{h}2^i(h - i) \\
= \sum_{i=0}^{h}\frac{h-i}{2^{h-i}}{2^h} \\
= 2^h\sum_{k=0}^{h}\frac{k}{2^k} \\
\le n\sum_{k=0}^\infty\frac{k}{2^k} \\
= O(n)
$$

2. **自顶向下（Top-Down）**建堆。可以理解为最开始只有一个空堆，每次都 push 一个元素进去，然后让其上浮，堆重建，时间复杂度为 $log(1)+log(2)+\cdots+log(n)$，即 **O(nlog(n))**。

**堆的排序过程还可以描述为：**每次将倒数第 $i$ 个元素与堆顶元素兑换，这样就能保证第 $i$ 个元素到最后一个元素都是排好序的了。然后再将兑换后的堆顶元素往下沉（不能沉到 $i$ 之后），重建堆。**这样我们就可以实现原位的堆排序了**。

# Examples

## 前 K 个高频元素

[LeetCode.347](https://leetcode.cn/problems/top-k-frequent-elements/)

Given an integer array nums and an integer k, return the k most frequent elements. You may return the answer in any order.

Example 1:

Input: nums = [1,1,1,2,2,3], k = 2

Output: [1,2]

{% note Code %}

#二叉堆

```typescript
function topKFrequent(nums: number[], k: number): number[] {
  // 哈希表统计元素数量 O(n)
  const map = new Map()
  for (let i = 0; i < nums.length; i++) {
    map.set(nums[i], (map.get(nums[i]) || 0) + 1)
  }
  // 遍历哈希表，用二叉堆（优先级队列）统计出前 k 个值
  // 这里用的是小顶堆，因为如果堆的size超过k，是把堆顶的那个值给扔掉
  // 如果是大顶堆就是把最大值扔掉了，我们要逐步扔掉比较小的值，把大的值沉在堆底
  const heap = new Heap<[number, number]>((a, b) => a[1] - b[1])
  for (const entry of map.entries()) {
    heap.push(entry)
    if (heap.size() > k) {
      heap.pop()
    }
  }
  // 将堆剩下的输出到 result 数组中
  const result = []
  while (heap.size()) {
    result.push(heap.pop()[0])
  }
  return result
}
```

{% endnote %}
