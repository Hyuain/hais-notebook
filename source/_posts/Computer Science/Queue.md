---
title: Queue
date: 2022-02-05 15:49:12
categories:
  - - 计算机
mathjax: true
tags:
  - CX
---


> 队列，先进先出（First In First Out, FIFO）

# Implementation of Queue

使用数组实现时，可以将 front 的指针一直指向 0，rear 的指针指向队尾元素；也可以将 rear 的指针一直指向 0，front 指针一直指向队尾元素。但因为数组删除的时间复杂度是 O(n)，所以这两种方法不能同时保证 O(1) 的入队和出队复杂度。

用环形队列可以节约内存，并且入队和出队都能得到 O(1) 的时间复杂度：

![环形队列](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Algorithmn-Queue-Implementation.jpg)

```typescript
// 这里使用 TypeScript 仅供参考，提供此种实现方案的思路
class Queue {
  
  // 该队列最多 9 个元素
  private q = new Array(10)
  private front = 0
  private rear = 0
  
  constructor() {}
  

  enqueue(x) {
    if (this.isFull()) {
      throw new Error('Full')
    }
    this.q[this.rear] = x
    if (this.rear === this.q.length - 1) {
      this.rear = 0
    } else {
      this.rear++
    }
  }
  
  dequeue() {
    if (this.isEmpty()) {
      return
    }
    const x = this.q[this.front]
    if (this.front === this.q.length - 1) {
      this.front = 0
    } else {
      this.front ++
    }
  }
  
  
  isFull() {
    // rear 在 front 左边，表示套圈了
    // 另外注意 rear 指向的是队尾的最后一位，是一个空元素
    // 所以队列的最大容量是数组的长度 - 1
    return this.rear + 1 === this.front || this.front === 0 && this.rear === this.q.length - 1
  }
  
  isEmpty() {
    // rear 和 front 相等时，定义为空
    return this.rear === this.front
  }
}
```

# Monotonic Queue

单调队列与单调栈有些许类似之处，他常用于求解区间内的的最大值或最小值。单调队列需要保证队列内部的元素单调递增或递减，除此之外，实际算法应用中由于经常会涉及到其他操作，因此大多数需要借助双端队列实现。

实际算法中的单调队列可以看成是变种的单调栈，比如维持一个单调递减的单调队列，我们需要：

- 如果当前需要插入的元素大于队尾元素，则将队尾 `pop` 出来，重复这个过程——这与[[Stack#Monotonic Stack|单调栈]]是一样的；
- 除此之外，他还可以通过出队来找到最大元素（队首元素）。

# Examples

## 滑动窗口最大值

[LeetCode.239](https://leetcode.cn/problems/sliding-window-maximum/)
