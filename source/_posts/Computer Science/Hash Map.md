---
title: Hash Map
date: 2022-02-05 15:49:12
categories:
  - - 计算机
mathjax: true
tags:
  - CX
---

> 哈希表

# Hash

考虑我们有一堆 Key-Value 值，比如 Key 是 id，Value 是姓名，怎么才能实现快速通过 id 获取到姓名呢？

首先如果 id 本来就是数字，那么可以用数组来实现，id 即为数组的 index，这样就可以通过数组的 index 直接拿到姓名了，**查找的时间复杂度是 O(1)**。

但是这样的话如果 id 分布比较分散，比如对于两个值 1 和 100，我们为了能将他们存起来，需要分配一个长度至少为 101 的数组，index 从 0 取值到 100，这样中间就有大量空间浪费掉了，空间复杂度是 **O(range(id))**。

**哈希函数是一个特定的编码函数， 可以将 Key 转换为一个索引值，这样就可以节约大量空间。**

比如定义一个简单的哈希函数 `H(k) = k % m`，那么就有 `H(293) = 3`，将这个 id 为 293 的项存在 index 为 3 的位置即可。

# Hash Collision

如果映射过去的索引空间太小，就可能将不同的值映射到同样的地方去，比如 `8 % 5 = 3 % 5 = 3`，因此 id 为 8 和 5 时都会被映射到索引为 3 的地方去，这被称为 **哈希碰撞**。

定义 **负载率（Load Factor）** $\lambda = n / m$，其中 $n$ 是 key 的总数， $m$ 是哈希表索引空间的大小，最好保持 $\lambda < 0.5$。

可以通过 **链表（Chaining）** 或 **开放寻址（Open Addressing）** 解决哈希碰撞问题：

## Chaining

> 将碰撞的那一位 `A[k]` 使用链表来进行存储。

查找的时间复杂度最坏 O(n)，平均 O(λ)（若 λ < 1，则为 O(1)）；因此为了保持良好的性能，需要保持 $\lambda < 1$。

## Open Addressing

> 如果 A[k] 已经被占据了，找另一个地方存。

例如有哈希函数 $H(k, i) = (k+i) \% m$，其中 $i$ 是尝试次数。

注意：

- 插入和查找不能无休止地进行下去，有最大尝试次数；
- 删除不能无休止地进行下去，可以将已经删除的元素标记位 DELETED，遇到 DELETED 的话继续删下去，而遇到什么都没有的空元素，说明删到头了，停止删除。

与链表不同，**必须** 保持 $\lambda \le 1$，否则新的元素就插不进去了。

哈希函数有两种策略：

- **线性探测（Linear Probing）**：$(k+i)\%m$，其性能在 $\lambda > 0.5$ 时显著下降；
- **次方探测（Quadratic Probing）**：$k \% m + a \cdot i + b \cdot i^2$。哈希表中的元素容易堆在一起形成集群，次方探测可以减少集群问题，从而减少碰撞，因而性能下降没有线性探测剧烈。
