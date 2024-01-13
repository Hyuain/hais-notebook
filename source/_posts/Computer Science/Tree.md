---
title: Tree
date: 2022-02-05 15:49:12
categories:
  - - 计算机
mathjax: true
tags:
  - CX
---

树实际上是一种特殊的[[Graph|图]]，其中比较常用的有[[Binary Tree|二叉树]]、B Tree、B+ Tree 等。

# B Tree

B 树（B tree）是一种自平衡的树，能够保持数据有序。这种数据结构能够让查找数据、顺序访问、插入数据及删除的动作，都在对数时间内完成。

与普通的二叉树不同，B 树有一些额外的特点：

- B 树的每个节点可以拥有两个以上的子节点；
- B 树中拥有 $k$ 个子节点的非叶节点，拥有 $k - 1$ 个键，且他们升序排列，比如 $7, 16$；
- 每个键的左子树中的所有键都小于他，右子树中所有键都大于他；
- 所有叶节点都位于同一层。



![B Tree](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/b-tree-1.svg)

// TBC

# B+ Tree

B+ 树（B+ Tree）是 B 树的变种，与 B 树有以下不同：

- B 树上的每个节点可以都可以存键和数据，但是 B+ 树的内部节点（非叶结点、索引节点）只存索引，而数据都存在叶节点上；
- 每个键的左子树中的所有键都小于他，右子树中所有键都 **大于等于** 他；
- 每个叶节点都存有相邻叶节点的指针，叶节点本身按照键从小到大排列；
- 父结点存有右孩子的第一个元素的索引。

![B+ Tree](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/bplus-tree-1.png)

// TBC
