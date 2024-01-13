---
title: Linked Hash Map
date: 2022-02-05 15:49:12
categories:
  - - 计算机
mathjax: true
tags:
  - CX
---

> **哈希链表（Linked Hash Map）** 是将[[Hash Map|哈希表]]与[[Linked List|链表]]组合起来使用的一种数据结构，在设计 LRU 缓存的时候有妙用。

# LRU

> [LeetCode.146 LRU Cache](https://leetcode.cn/problems/lru-cache/description)

**LRU（Least Recently Used）** 是一种常见的缓存策略。由于缓存的容量是有限的，需要在缓存容量满了的时候清理掉上次使用时间距离现在最久的缓存，给新的数据腾出空间。

可以用一个类 `LRUCache` 来进行示意，他需要 `capacity` 作为初始化参数，表示该内存的最大容量，然后提供 `get` 和 `put` 两个方法，分别用于获取缓存数据和更新缓存数据：

```typescript
class LRUCache {
  public capacity: number
  public get(key :number): number {}
  public put(key: number, value: number): void {}
}
```

借助 **哈希链表（Linked Hash Map）**，我们可以在 O(1) 的时间复杂度内实现 `get` 和 `put` 方法。**哈希链表有哈希表的优点，可以在 O(1) 的时间复杂度内快速取值；也有链表的优点，即有一个可以在 O(1) 的时间复杂度内进行增删操作的线型数据结构。**

哈希链表如下图所示，他有：

- **一个哈希表**，键为 Key，该 Key 就是缓存的 Key，值为指向链表中某个节点的指针；
- **一个双向链表**，链表中有 Key 和 Value，其中的 Key 与指向他的哈希表中的 Key 相同，Value 则是任意值，比如缓存的内容。

![哈希链表](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/1647580694-NAtygG-4.jpg)

**在实现 LRU 时，我们将链表尾部的数据视为最新的，如果数据被访问了，就提升到链表尾部；如果链表满了，就删除链表头部的节点即可。**

Java 中内置了 `LinkedHashList` 类，但在 JavaScript 中，我们需要从双向链表开始实现。

{% note Code %}

先实现链表节点，他有四个属性，`next` `prev` `key` 和 `value`：

```typescript
class LinkedListNode<K, V> {
  constructor(
    public key: K,
    public value: V,
    public next: LinkedListNode<K, V> = null,
    public prev: LinkedListNode<K, V> = null,
  ) {}
}
```

然后实现双向链表，他只提供三个 API，往尾部添加节点 `appendToTail`，删除某节点 `delete`，和删除头部节点 `deleteFromHead`，，其实更加完善的解决方案是提供一个遍历的方法，让我们可以获取到头，然后再删除他，这里简化成一个 `deleteFromHead` 方法：

```typescript
class DoubleLinkedList<K, V> {
  private head: LinkedListNode<K, V> = new LinkedListNode(null, null)
  private tail: LinkedListNode<K, V> = new LinkedListNode(null, null)
  public size = 0

  constructor() {
    this.head.next = this.tail
    this.tail.prev = this.head
  }

  public appendToTail(node: LinkedListNode<K, V>) {
    const prev = this.tail.prev
    prev.next = node
    node.next = this.tail
    node.prev = prev
    this.tail.prev = node
    this.size++
  }

  public delete(node: LinkedListNode<K, V>): void {
    const prev = node.prev
    const next = node.next
    prev.next = next
    next.prev = prev
    node.prev = null
    node.next = null
    this.size--
  }

  public deleteFromHead(): LinkedListNode<K, V> {
    if (!this.size) { return }
    const first = this.head.next
    this.delete(first)
    return first
  }
}
```

接着实现哈希链表，除了哈希表的 `has` `get` `set` `delete` 方法以外，额外提供了 `deleteFirst` 方法，用于删去第一个节点，其实更加完善的解决方案是提供一个遍历的方法，让我们可以获取到第一个节点，然后再删除他，这里简化成一个 `deleteFirst` 方法：

```typescript
class LinkedHashMap<K, V> {
  private map = new Map<K, LinkedListNode<K, V>>()
  private list = new DoubleLinkedList<K, V>()

  constructor() {
  }

  public has(key: K): boolean {
    return this.map.has(key)
  }

  public get(key: K): V {
    const node = this.map.get(key)
    if (!node) { return }
    return node.value
  }

  public get size() {
    return this.map.size
  }

  public set(key: K, value: V) {
    if (this.map.has(key)) {
      const node = this.map.get(key)
      node.value = value
    } else {
      const node = new LinkedListNode(key, value)
      this.list.appendToTail(node)
      this.map.set(key, node)
    }
  }

  public delete(key: K): V {
    const node = this.map.get(key)
    if (!node) { return }
    this.list.delete(node)
    this.map.delete(key)
    return node.value
  }

  public deleteFirst() {
    if (!this.size) { return }
    const node = this.list.deleteFromHead()
    this.map.delete(node.key)
  }
}
```

最后实现 LRU 算法，在每次 `get` 和更新的时候将该值删掉并在链表尾部重新添加，增加的时候如果该缓存已满，就删掉链表头部的节点：

```typescript
class LRUCache {
  private cache = new LinkedHashMap<number, number>()

  constructor(public capacity: number) {
  }

  get(key: number): number {
    if (!this.cache.has(key)) {
      return -1
    }
    this.makeResent(key)
    return this.cache.get(key)
  }

  put(key: number, value: number): void {
    if (this.cache.has(key)) {
      this.cache.set(key, value)
      this.makeResent(key)
      return
    }
    if (this.cache.size >= this.capacity) {
      this.cache.deleteFirst()
    }
    this.cache.set(key, value)
  }

  // 删掉该值并在链表尾部重新添加
  private makeResent(key: number): void {
    const value = this.cache.delete(key)
    this.cache.set(key, value)
  }
}
```

{% endnote %}

# LFU

> [LeetCode.460 LFU Cache](https://leetcode.cn/problems/lfu-cache/)

**LFU（Least Frequently Used）**是另一种缓存策略，他会淘汰访问频率最低的元素，如果频率相同，淘汰最久的那个元素。

同样我们用类 `LFUCache` 来进行示意，他需要 `capacity` 作为初始化参数，表示该内存的最大容量，然后提供 `get` 和 `put` 两个方法，分别用于获取缓存数据和更新缓存数据：

```typescript
class LRUCache {
  public capacity: number
  public get(key :number): number {}
  public put(key: number, value: number): void {}
}
```

但是 LFU 比 LRU 复杂一些，需要用 **两个哈希表 + N 个双向链表** 才能实现。

- **Key-Value 哈希表**
  - 哈希表的 Key 为 **缓存的 Key**
  - 哈希表的 Value 中保存了一个 **Node**，其中包含 **缓存的 Key、缓存的 Value、使用的频率**，这个 Node 也会出现在第二个哈希表（频率哈希表）中
- **频率哈希表**
  - 哈希表的 Key 为 **频率**
  - 哈希表的 Value 是一个 **双向链表**，链表中的节点就是 KV 哈希表中的提到的 **Node**

![LFU.jpg](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/bb3811c03de13fc8548a01c9ab094f5ed38d7ef9b5f5c6ef82340e222750ae92-3.jpg)

**此外我们还需要维护一个 `minFreqency` 变量来记录当前哈希表中的最低频率，来实现 O(1) 复杂度淘汰元素：**

- 更新/查找的时候，将元素频率 + 1，如果 `minFreq` 已经不在频率哈希表中了，就将 `minFreqency + 1`；
- 插入的时候，将 `minFreqency` 修改为 `1`。

由于 LRU 并没有提供 `delete` 接口，因此不可能存在主动将某个频率删除掉、导致 `minFreq` 找不到的情况，`minFreq` 肯定会跟着元素不停向上加。

{% note Code %}

双向链表相关的部分（`LinkedListNode` 和 `DoubleLinkedList`）与 LRU 基本一致，不过 `LinkedListNode` 中增加了 `frequency` 属性，表示该元素的使用频率。

```typescript
class LinkdedListNode<K, V> {
  constructor(
    public key: K,
    public value: V,
    public frequency: number = 0,
    public next: LinkedListNode<K, V> = null,
    public prev: LinkedListNode<K, V> = null,
  ) {}
}
```

然后我们直接在 `LFUCache` 类中来实现：

```typescript
class LFUCache {
  private kvMap = new Map<number, LinkedListNode<number, number>>()
  private freqMap = new Map<number, DoubleLinkedList<number, number>>()
  private minFrequency: number

  constructor(public capacity: number) {

  }

  get(key: number): number {
    if (!this.kvMap.has(key)) {
      return -1
    }
    const node = this.kvMap.get(key)
    // 增加频率
    this.increseFrequency(node)
    return node.value
  }

  put(key: number, value: number): void {
    if (!this.capacity) { return }
    // 如果存在就增加频率
    if (this.kvMap.has(key)) {
      const node = this.kvMap.get(key)
      node.value = value
      this.increseFrequency(node)
      return
    }
    // 如果超出容量就删除掉频率最低的中最靠近头部的
    if (this.kvMap.size >= this.capacity) {
      const list = this.freqMap.get(this.minFrequency)
      const node = list.deleteFromHead()
      if (!list.size) {
        this.freqMap.delete(this.minFrequency)
      }
      this.kvMap.delete(node.key)
    }
    // 插入新结点，并将 minFrequency 标记为 1
    const newNode = new LinkedListNode(key, value, 1)
    this.addToFreqMap(newNode)
    this.kvMap.set(key, newNode)
    this.minFrequency = 1
  }

  // 增加频率
  private increseFrequency(node: LinkedListNode<number, number>) {
    // 将老的频率链表找到，删除节点
    const oldFrequency = node.frequency
    const list = this.freqMap.get(oldFrequency)
    list.delete(node)
    node.frequency++
    // 将节点插入新的频率链表尾部
    this.addToFreqMap(node)
    // 删除空的频率链表，如果删除的频率链表是 minFrequency，就将 minFrequency + 1
    if (!list.size) {
      this.freqMap.delete(oldFrequency)
      if (oldFrequency === this.minFrequency) {
        this.minFrequency++
      }
    }
  }

  // 在频率链表中新插入一项
  private addToFreqMap(node: LinkedListNode<number, number>) {
    if (this.freqMap.has(node.frequency)) {
      const list = this.freqMap.get(node.frequency)
      list.appendToTail(node)
      return
    }
    const list = new DoubleLinkedList<number, number>()
    list.appendToTail(node)
    this.freqMap.set(node.frequency, list)
  }
}
```

{% endnote %}
