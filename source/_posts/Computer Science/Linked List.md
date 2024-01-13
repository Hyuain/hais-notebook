---
title: Linked List
date: 2022-02-05 15:49:12
categories:
  - - 计算机
mathjax: true
tags:
  - CX
---

> 链表

- 最后一个节点的 next 指针指向 null
- 通常有 head 指针表示第一个节点，有时候 tail 指针表示最后一个节点
- 双向链表的结点同时有 next 指针和 prev 指针

# Operations of Linked List

| 操作  | 时间复杂度 |
| ----- | ---------- |
| 访问  | O(1)       |
| 插入* | O(1)       |
| 删除* | O(1)       |

*注意如果插入和删除并不知道元素到底在哪里，需要从头开始查找的话，复杂度是 O(n)
