---
title: Binary Tree
date: 2022-02-05 15:49:12
categories:
  - - 计算机
mathjax: true
tags:
  - CX
---
二叉树是常用的一种特殊的[[Tree|树]]。

# Glossary

| 英文                 | 中文       | 说明                                                         |
| -------------------- | ---------- | ------------------------------------------------------------ |
| Root                 | 根节点     |                                                              |
| Leaf                 | 叶节点     | 没有子节点的节点                                             |
| Parent               | 父节点     |                                                              |
| Length of a Path     | 路径的长度 | 路径上边的数量                                               |
| Ancestor of a Node   | 节点的祖先 | 父节点、父节点的父节点等                                     |
| Descendant of a Node | 节点的后代 | 子节点、子节点的子节点等                                     |
| The Height of a Node | 节点的高度 | 该节点到叶节点的、最大的、不含祖先节点的路径长度，叶节点高度为 0 |
| Tree Height          | 树的高度   | 根节点的高度                                                 |
| The Depth of a Node  | 节点的深度 | 该节点到根节点的路径长度，根节点深度为 0                     |
| Subtree              | 子树       | 包括节点及其后代                                             |

# Definition

```typescript
class TreeNode {
  public val: number
  public left: TreeNode | null
  public right: TreeNode | null
  constructor(val: number = 0, left: TreeNode | null = null, right: TreeNode | null = null) {
    this.val = val
    this.left = left
    this.right = right
  }
}
```

# Traversal

## Deep First Traversal

> 深度优先遍历分为前序、中序和后序遍历，均有递归和迭代写法，主要需要借助栈来实现

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/TreeDFS.png)

### 递归写法

递归三步：

1. 确定参数和返回值
2. 确定终止条件
3. 确定每次需要做的事情

#### 前序（中左右）

```typescript
const traversal = (current: TreeNode | null, cb?: (node: TreeNode) => any) => {
  if (!current) {  return }
  cb?.(current)              // 中
  traversal(current.left)   // 左
  traversal(current.right)  // 右
}
```

#### 中序（左中右）

```typescript
const traversal = (current: TreeNode | null, cb?: (node: TreeNode) => any) => {
  if (!current) { return }
  traversal(current.left)   // 左
  cb?.(current)             // 中
  traversal(current.right)  // 右
}
```

#### 后序（左右中）

```typescript
const traversal = (current: TreeNode | null, cb?: (node: TreeNode) => any) => {
  if (!current) { return }
  traversal(current.left)   // 左
  traversal(current.right)  // 右
  cb?.(current)             // 中
}
```

### 迭代写法

#### 前序（中左右）

![二叉树的前序遍历（迭代法）](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Algorithm-Tree-1.gif)

入栈顺序为 中、**右、左**，这样才能保证出栈顺序为 中、**左、右**

```typescript
const traversal = (root: TreeNode | null, cb?: (node: TreeNode) => any) => {
  if (!root) { return }
  const stack = [root]          // 中入栈
  while (stack.length) {
    const current = stack.pop() // 出栈，可以看出中最先出栈，然后是左，然后是右
    cb?.(current)               // 处理
    if (current.right) {
      stack.push(current.right) // 右入栈
    }
    if (current.left) {
      stack.push(current.left)  // 左入栈
    }
  }
}
```

#### 中序（左中右）

![二叉树的中序遍历（迭代法）](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Algorithm-Tree-2.gif)

需要借助指针来跟踪当前访问的节点

```typescript
const traversal = (root: TreeNode | null, cb?: (node: TreeNode) => any) => {
  if (!root) { return }
  const current = root
  const stack = []
  while (current || stack.length) {
    if (current) {
      stack.push(current)        // 入栈
      current = current.left     // 准备左入栈（第一次会一直入栈直到最左下角的元素，那才是处理的起点）
    } else {
      current = stack.pop()      // 出栈，可以看出左最先出栈，然后是中，然后是右
      cb?.(current)               // 处理
      current = current.right    // 准备右入栈
    }
  }
}
```

#### 后序（左右中）

先写前序遍历（中左右），写的时候把左右入栈顺序颠倒，就是中右左，然后反转 result 数组就是后序遍历

```typescript
const traversal = (root: TreeNode | null): TreeNode[] => {
  if (!root) { return }
  const result = []
  const stack = [root]
  while (stack.length) {
    const current = stack.pop()
    result.push(current)
    if (current.left) {
      stack.push(current.left)
    }
    if (current.right) {
      stack.push(current.right)
    }
  }
  return result.reverse()
}
```

### 统一风格的迭代写法

使用标记法，在需要处理的元素入栈之后推入 `null`，这样栈里面就有两种元素：

1. 需要处理的元素，他后面会接一个 `null`
2. 需要进一步访问其子结点的元素，他后面不会接 `null`

#### 中序（左中右）

```typescript
const traversal = (root: TreeNode | null, cb?: (node: TreeNode) => any) => {
  if (!root) { return }
  const stack = [root]
  while (stack.length) {
    const current = stack.pop()
    if (!current) {               // 出栈时遇到 null，说明下一个元素需要处理
      cb?.(stack.pop())
      continue
    }
    if (current.right) { stack.push(current.right) }   // 右入栈，待访问
    stack.push(current, null)                          // 中入栈，待处理
    if (current.left) { stack.push(current.left) }     // 左入栈，待访问
  }
}
```

![二叉树的中序遍历（统一迭代法）](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Algorithm-Tree-3.gif)

#### 前序（中左右）

```typescript
const traversal = (root: TreeNode | null, cb?: (node: TreeNode) => any) => {
  if (!root) { return }
  const stack = [root]
  while (stack.length) {
    const current = stack.pop()
    if (!current) {
      cb?.(stack.pop())
      continue
    }
    if (current.right) { stack.push(current.right) }
    if (current.left) { stack.push(current.left) }
    stack.push(current, null)
  }
}
```

#### 后序（左右中）

```typescript
const traversal = (root: TreeNode | null, cb?: (node: TreeNode) => any) => {
  if (!root) { return }
  const stack = [root]
  while (stack.length) {
    const current = stack.pop()
    if (!current) {
      cb?.(stack.pop())
      continue
    }
    stack.push(current, null)
    if (current.right) { stack.push(current.right) }
    if (current.left) { stack.push(current.left) }
  }
}
```

## Breadth First Traversal

> 广度优先遍历即层序遍历

### 迭代法

使用队列和一个缓存当前层的数组

```typescript
function traversal(root: TreeNode | null): number[][] {
  if (!root) { return [] }
  const queue: TreeNode[] = [root]
  const res = []
  while (queue.length) {
    const currentLevel = []
    const currentLevelSize = queue.length
    for (let i = 0; i < currentLevelSize; i++) {
      const current = queue.shift()
      currentLevel.push(current.val)
      if (current.left) { queue.push(current.left) }
      if (current.right) { queue.push(current.right) }
    }
    res.push(currentLevel)
  }
  return res
}
```

### 递归法

传入 deep 可以不管当前递归到哪里了，自己属于哪一层

```typescript
function levelOrder(root: TreeNode | null): number[][] {
    function traversal(current: TreeNode | null, result: number[][], deep: number) {
        if (!current) { return }
        if (!result[deep]) { result[deep] = [] }
        result[deep].push(current.val)
        traversal(current.left, result, deep + 1)
        traversal(current.right, result, deep + 1)
    }
    const result = []
    traversal(root, result, 0)
    return result
}
```

# Construct Binary Trees

**[从中序和后序遍历构造二叉树](https://leetcode.cn/problems/construct-binary-tree-from-inorder-and-postorder-traversal/)**

**[从前序与中序遍历构造二叉树](https://leetcode.cn/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)**

{% note Code %}

两题类似：

1. 后序遍历的最后一位就是 `root`，前序遍历的第一位就是 `root`
2. 再在中序遍历中找到 `root` 从而找到左右子树
3. 再在后序遍历中分开左右子树

```typescript
function buildTree(inorder: number[], postorder: number[]): TreeNode | null {
  if (!inorder.length) { return null }
  const rootVal = postorder[postorder.length - 1]
  const rootIndex = inorder.indexOf(rootVal)
  const leftInorder = inorder.slice(0, rootIndex)
  const rightInorder = inorder.slice(rootIndex + 1)
  const leftPostorder = postorder.slice(0, leftInorder.length)
  const rightPostorder = postorder.slice(leftInorder.length, postorder.length - 1)
  return new TreeNode(
    rootVal,
    buildTree(leftInorder, leftPostorder),
    buildTree(rightInorder, rightPostorder)
  )
}
```

{% endnote %}

# Other Operations

**[翻转二叉树](https://leetcode.cn/problems/invert-binary-tree/)**：#后序 #先序 #层序

**[对称二叉树](https://leetcode.cn/problems/symmetric-tree/)**：#内外层 #内外层共用一个数组层序

**[二叉树最大深度](https://leetcode.cn/problems/maximum-depth-of-binary-tree/)**：#最大深度=根节点高度 #后序求高度 #先序求深度 #层序用size控制每次只遍历一层

**[二叉树最小深度](https://leetcode.cn/problems/minimum-depth-of-binary-tree/)**：#最小深度=根节点到叶节点的距离 #后序求高度需要注意最低点的位置判断 #先序求深度 #层序

# Special Binary Trees

## Full Binary Tree

每个节点有 0 或 2 个子结点

## Perfect Binary Tree

> 满二叉树

只有度为 0 或 2 的节点，且度为 0 的节点在同一层上。

深度为 $k$ 时，有 $2^k - 1$ 个节点。

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/PerfectBinaryTree.png)

## Complete Binary Tree

> 完全二叉树，有的作者会将 Perfect Binary Tree 称为 Complete Binary Tree，而称这种树为 Nearly Complete Binary Tree 或 Almost Complete Binary Tree。

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/CompleteBinaryTree.png)

除了最底层，每层都填满了，且最底层节点都集中在左边的位置。

这种树可以很方便地用数组表示，若当前节点的下标为 `i`，则其父节点的下标为 `Math.floor((i - 1) / 2)`，左孩子下标为 `2 * i + 1`，右孩子下标为 `2 * i + 2`，最后一个非叶节点下标为 `Math.floor(size / 2) - 1`。

优先级队列可以用堆来实现，而堆则就是一棵完全二叉树，同时保证了父子节点的顺序关系。

### Count

**完全二叉树的节点个数可以用时间复杂度为 O(logn * logn)、空间复杂度为 O(logn) 的解法来求，并不需要遍历整棵树：**

- 完全二叉树是由一棵棵满二叉树构成的，其中最小的满二叉树只有一个节点；
- 完全二叉树中，一个节点如果 **一直朝左下方遍历到底的层数** 等于其 **一直朝右下方遍历到底的层数**，那么以这个节点为根节点的子树一定是满二叉树；
- 因此我们只需要判断该节点是否是满二叉树，如果是则直接返回计算结果，否则再分别遍历左右两棵子树即可。

{% note Code %}

```typescript
function countNodes(root: TreeNode | null): number {
  if (!root) { return 0 }
  let leftDept = 1
  let rightDept = 1
  let node = root
  while (node.left) {
    node = node.left
    leftDept++
  }
  node = root
  while (node.right) {
    node = node.right
    rightDept++
  }
  if (leftDept === rightDept) {
    return (1 << leftDept) - 1
  }
  return countNodes(root.left) + countNodes(root.right) + 1
};
```

{% endnote %}

## Binary Search Tree

> 二叉搜索树

若左子树不为空，则左子树上所有节点的值均小于该根节点的值；若右子树不为空，则右子树上所有节点的值均大于该根节点的值。

**二叉搜索树的中序遍历得到的结果是有序的**，利用这个特性可以解决一些问题。

### Search

> 时间复杂度 O(log n)

```typescript
const search = (root: TreeNode | null, target: any) => {
  let current = root
  while (current) {
    if (current.value === target) {
      return current
    }
    current = current.value < target ? current.right : current.left
  }
  return null
}
```

### Validation

```typescript
function isValidBST(root: TreeNode | null): boolean {
  function checkValidBST(node: TreeNode | null, min: number, max: number): boolean {
    if (!node) { return true }
    if (node.val >= max || node.val <= min) { return false }
    return checkValidBST(node.left, min, Math.min(node.val, max))
    && checkValidBST(node.right, Math.max(min, node.val), max)
  }
  return checkValidBST(root, -Infinity, +Infinity)
}
```

### Insertion

> 时间复杂度 O(log n)

```typescript
const insert = (root: TreeNode | null, node: TreeNode) => {
  if (!root) {
    return node
  }
  let current = root
  let prev
  // find a node to insert
  while (current) {
    prev = current
    if (node.value < current.value) {
      current = current.left
    } else if (node.value > current.value) {
      current = current.right
    } else {
      // already exist
      return root
    }
  }
  if (node.value < prev) {
    prev.left = node.value
  } else {
    prev.right = node.value
  }
}
```

### Deletion

- 如果没有子节点：直接删除
- 如果有一个子节点：用子结点替代该节点
- 如果有两个子结点：删除右子树最小节点 x，然后用 x 替代该节点
  - 称 x 为继任者，x 是比该节点大一位的节点
  - 也可以用比 x 节点小一位的节点（左子树的最大节点）来替代，但是这样得到的树高度差会更大（？）

```typescript
function deleteNode(root: TreeNode | null, key: number): TreeNode | null {
  let current = root
  let currentParent = null
  while (current) {
    if (current.val === key) {
      break
    } else if (current.val < key) {
      currentParent = current
      current = current.right
    } else {
      currentParent = current
      current = current.left
    }
  }
  if (!current) { return root }
  if (!current.left && !current.right) {
    if (!currentParent) { return null }
    if (currentParent.left === current) {
      currentParent.left = null
    } else {
      currentParent.right = null
    }
  } else if (current.left && !current.right) {
    if (!currentParent) { return current.left }
    if (currentParent.left === current) {
      currentParent.left = current.left
    } else {
      currentParent.right = current.left
    }
  } else if (!current.left && current.right) {
    if (!currentParent) { return current.right }
    if (currentParent.left === current) {
      currentParent.left = current.right
    } else {
      currentParent.right = current.right
    }
  } else {
    let successor = current.right
    let successorParent = current
    while (successor.left) {
      successorParent = successor
      successor = successor.left
    }
    current.val = successor.val
    if (successorParent.left === successor) {
      successorParent.left = successor.right
    } else {
      successorParent.right = successor.right
    }
  }
  return root
};
```

## AVL Tree

> 平衡二叉树，又称为 Adleson-Velsky and Landis Tree

是空树；或者他的左右两个子树的高度差绝对值不超过 1，并且左右两棵子树都是一棵平衡二叉树。

### isBalanced

[LeetCode.110 Balanced Binary Tree](https://leetcode.cn/problems/balanced-binary-tree/)

{% note Code %}

思路：用后序遍历求分别求子树高度

```typescript
// 递归法，用 -1 传递已经不平衡了的状态
function isBalanced(root: TreeNode | null): boolean {
  return getHeight(root) !== -1
}

function getHeight(node: TreeNode | null): number {
  if (!node) { return 0 }
  const leftHeight = getHeight(node.left)
  const rightHeight = getHeight(node.right)
  if (leftHeight === -1 || rightHeight === -1 || Math.abs(leftHeight - rightHeight) > 1) {
    return -1
  }
  return Math.max(leftHeight, rightHeight)
}
```

```typescript
// 迭代法，用栈模拟后序遍历，然后用先序遍历或者层序遍历求节点高度(深度)、或后序遍历求高度
// 该方法会有很多冗余的过程，效率比较低下
function isBalanced(root: TreeNode | null): boolean {
  if (!root) { return true }
  const stack = [root]
  while (stack.length) {
    let current = stack.pop()
    if (current === null) {
      current = stack.pop()
      const leftHeight = getHeight(current.left)
      const rightHeight = getHeight(current.right)
      if (Math.abs(leftHeight - rightHeight) > 1) {
        return false
      }
      continue
    }
    stack.push(current, null)
    if (current.right) {
      stack.push(current.right)
    }
    if (current.left) {
      stack.push(current.left)
    }
  }
  return true
};

function getHeight(node: TreeNode | null): number {
  if (!node) { return 0 }
  const queue = [node]
  let depth = 0
  while (queue.length) {
    const length = queue.length
    for (let i = 0; i < length; i++) {
      const current = queue.shift()
      if (current.left) {
        queue.push(current.left)
      }
      if (current.right) {
        queue.push(current.right)
      }
    }
    depth++
  }
  return depth
}

function getHeight(node: TreeNode | null): number {
  if (!node) { return 0 }
  const stack = [node]
  let depth = 0
  let maxDepth = 0
  while (stack.length) {
    let current = stack.pop()
    if (current === null) {
      current = stack.pop()
      depth--
      continue
    }
    stack.push(current, null)
    depth++
    if (current.right) {
      stack.push(current.right)
    }
    if (current.left) {
      stack.push(current.left)
    }
    maxDepth = Math.max(maxDepth, depth)
  }
  return maxDepth
}
```

{% endnote %}

### Convert Sorted Array to Binary Search Tree

[LeetCode.108](https://leetcode.cn/problems/convert-sorted-array-to-binary-search-tree/)

Given an integer array nums where the elements are sorted in ascending order, convert it to a height-balanced binary search tree.

{% note Code %}

思路：对有序数组按照中点进行切分，左边给左子树，右边给右子树。

```typescript
function sortedArrayToBST(nums: number[]): TreeNode | null {
  if (!nums.length) { return null }
  const midIndex = Math.floor(nums.length / 2)
  const root = new TreeNode(nums[midIndex])
  root.left = sortedArrayToBST(nums.slice(0, midIndex))
  root.right = sortedArrayToBST(nums.slice(midIndex + 1))
  return root
}
```
{% endnote %}

## Segment Tree

> 线段树是一种二叉树，用来存储区间或线段，可以用来快速查询结构内包含某一点的所有区间

比如对于数组 `arr = [10, 11, 12, 13, 14, 15]` 我们可以生成这样的一个树形结构，其中每个节点维护的就是这个节点所表示的区间总和。

```text
                               60[0,4]
                33[0,2]                        27[3,4]
       21[0,1]           12[2,2]     13[3,3]            7[4,4]
10[0,0]   11[1,1]
```

我们可以用数组 $d$ 来存储线段树，其中 $d_i$ 表示的就是其区间内值的和，比如 $d_0$ 表示区间 $[0,4]$ 的总和。

若 $d_i$ 保存区间 $[s,t]$ 的总和，那么：

他的左孩子 $d_{2i}$ 则保存区间 $[s, {s+t\over2}]$ 的总和；右孩子 $d_{2i+1}$ 保存区间 $[{s+t\over2}+1, t]$ 的总和。

下面的代码是线段树的 JavaScript 实现：

{% note Code %}

```typescript
class SegmentTree {
  private d = []
  private n = 0
  
  constructor(arr: number[]) {
    this.build(arr, 0, arr.length - 1, 0)
  }
  
  // 对 arr 的区间 [start, end] 建立线段树
  // 当前根节点的编号为 p
  build(
    arr: number[],
    start: number,
    end: number,
    p: number
  ) {
    this.n = arr.length
    if (start === end) {
      d[p] = arr[start]
      return
    }
    // 类似二分查找，找到中点的位置，下面相当于 start + Math.floor((end - start) / 2)
    let mid = start + ((end - start) >> 1)
    // 递归对左右区间建树
    this.build(arr, start, mid, p * 2)
    this.build(arr, mid + 1, end, p * 2 + 1)
    d[p] = d[p * 2] + d[p * 2 + 1]
  }
  
  // 在区间 [start,end] 中查询 [queryL,queryR] 区间的总和
  // 当前根节点编号为 p
  getSum(
    queryL: number,
    queryR: number,
    start: number = 0,
    end: number = this.n - 1,
    p: number = 0
  ) {
    // 当前区间为查询区间的子区间，返回当前区间的总和
    if (queryL <= start && end <= queryR) {
      return d[p]
    }
    let mid = start + ((end - start) >> 1)
    let sum = 0
    // 左孩子的区间与查询区间有交集，递归查询左孩子
    if (queryL <= mid) {
      sum += getSum(queryL, queryR, start, mid, p * 2)
    }
    // 右孩子的区间与查询区间有交集，递归查询右孩子
    if (queryR > mid) {
      sum += getSum(queryL, queryR, mid + 1, end, p * 2 + 1)
    }
    return sum
  }
}
```

{% endnote %}
