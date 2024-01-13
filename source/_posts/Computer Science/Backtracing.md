---
title: Backtracing
date: 2022-02-05 15:49:12
categories:
  - - 计算机
mathjax: true
tags:
  - CX
---

**回溯算法** 的本质是 [[Binary Tree#Deep First Traversal|DFS]]，而 **回溯操作** 实际上指的是：

- 我们在遍历[[Binary Tree|二叉树]]的时候，可能会定义一些全局变量来记录当前所在的节点位置、或者遍历到当前节点的时候累计的某些值
- **在我们采用深度优先遍历时，这些变量需要在遍历 B 子树之前，退回到原来的状态，消除遍历 A 子树时产生的影响**

由于回溯算法的本质是遍历操作，因此效率并不高，通常需要进行剪枝来些微提高效率，一般来说只有在解决以下这些比较复杂的问题时，才会考虑回溯法：

- **组合问题：** `n` 个数里面找出 `k` 个数的集合
- **排列问题：** `n` 个数按照一定规则全排列
- **切割问题：** 一个字符串按照一定规则进行切割
- **子集问题：** 给定一个集合，找到所有可能的子集
- **棋盘问题：** N皇后、解数独

回溯法其实就是对[[Tree|树]]的遍历，不过对象通常不是二叉树，因此需要两种循环：

- **横向遍历：** `for` 循环
- **纵向遍历：** 递归

```typescript
// 由于回溯算法本质就是 dfs，因此这里用 dfs 作为函数名，当然也可以用 backtracing 等
// 可以看到这个函数其实就是在 Graph 一章中讲的 dfs
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

# 组合问题

组合问题实质上是指的 `n` 中选 `k`，不需要考虑顺序，即 `[1, 2]` 和 `[2, 1]` 认为是一种情况，因此一般不需要考虑已经考虑过的位，一位一位地考虑即可：

- **横向遍历** 为当前位可选择的项；
- **纵向遍历** 为当前位已经确定，进入下一位。

模板代码如下：

```typescript
function combine(k: number) {
  const results = []
  const temp = []
  function dfs(start: number) {
    // 已经选了 k 个了，结束
    if (temp.length === k) {
      results.push([...temp])
      return
    }
    // 确定当前位可选的内容，通常需要借助 start 的帮忙
    const selections = [....]
    // 横向遍历当前位的可选内容
    for (const selection of selections) {
      // 确定当前位并进入下一位
      temp.push(selection)
      dfs()
      temp.pop()
    }
  }
  dfs(0)
}
```

[LeetCode.77 Combinations](https://leetcode.cn/problems/combinations/description/)

> 从 `n` 个数里面找出 `k` 个数的集合

{% note Code %}

```typescript
function combine(n: number, k: number): number[][] {
  const results = []
  const temp = []
  function dfs(start: number) {
    if (temp.length === k) {
      results.push([...temp])
      return
    }
    // 剪枝，因为最终集合需要 k 个数，因此在不够的情况下可以提前结束
    for (let i = start; i <= n - k + temp.length + 1; i++) {
      temp.push(i)
      dfs(i + 1)
      temp.pop()
    }
  }
  dfs(1)
  return results
};
```
{% endnote %}

[LeetCode.17 Letter Combinations of a Phone Number](https://leetcode.cn/problems/letter-combinations-of-a-phone-number/) 和 [LeetCode.216 Combination Sum III](https://leetcode.cn/problems/combination-sum-iii/description/) 也是类似的解法。

[LeetCode.39 Combination Sum](https://leetcode.cn/problems/combination-sum/) 稍微有点特殊，**组合中的元素可以重复**，但这也并不意味着我们需要考虑以前考虑过的项，只需要考虑当前项重复的情况即可。即在 LeetCode.77 等题中，纵向遍历采用 `dfs(i + 1)`；而在本题中，需要使用 `dfs(i)`。

同时，**此类组合还有另外一种 DFS 的思路，该思路可以转换为递推公式**，即：将问题拆分到每一个元素上，对于每个元素来说，只有两种可能性，选或不选，因此可以得到类似这样的代码：

{% note Code %}

```typescript
// LeetCode.39 Combination Sum
function combinationSum(candidates: number[], target: number): number[][] {
  const temp = []
  const results = []
  function dfs(start: number, rest: number) {
    if (rest <= 0 || start >= candidates.length) {
      if (rest === 0) {
        results.push([...temp])
      }
      return
    }
    const current = candidates[start]
    // 选当前项
    temp.push(current)
    dfs(start, rest - current)
    temp.pop()
    // 不选当前项
    dfs(start + 1, rest)
  }
  dfs(0, target)
  return results
};
```

{% endnote %}

与动态规划不同的是，这种 DFS 是全搜索 DFS，可以看到并没有加中间判断以选出每个子问题最优解，而是对所有子问题都进行了遍历（这一点会在动态规划中有更详细的讨论）。

## 组合问题中的去重

[LeetCode.40 Combination Sum II](https://leetcode.cn/problems/combination-sum-ii/description/) 中就要求对组合进行去重，组合问题中去重的特点是：

1. **候选元素集合中有值相同的元素**
2. **候选元素集合中每个元素只能使用一次**，但由于候选集合中有相同的元素，因此结果集合中可能有值相同的元素
3. **不能有相同的结果集合**

由于候选集合中可能有值相同的元素，如果不进行去重处理，**两个结果集可能分别使用了候选集合中位置不同、但值相同的元素**，导致两个结果集相同了。

从第 2 点可以看出来，由于并没有限制单个结果集合内不能出现重复的元素，因此比如在候选集 `[1, 1, 2]` 中，可以出现集合 `[1, 1]`，但不能出现两个 `[1, 2]`。该类去重是 **对树的横向去重（在 `for` 循环中去重）**，而纵向可以重复。

对树的横向去重，实际上就是对 **当层的可选元素数组** 的去重（即对模板代码中的 `selections` 去重），数组去重常用的就是 `Set` 和 **排序后相邻元素去重**。

### 使用 Set 去重

在 `for` 循环遍历的时候使用 `Set` 即可很方便地进行横向去重：

{% note Code %}

```typescript
function combinationSum2(candidates: number[], target: number): number[][] {
  candidates.sort((a, b) => a - b)
  const results = []
  const temp = []
  function dfs(start: number, rest: number) {
    if (rest <= 0 || start === candidates.length) {
      if (rest === 0) {
        results.push([...temp])
      }
      return
    }
    // 用 Set 记录本层已经检查过的值
    const visted = new Set()
    for (let i = start; i < candidates.length; i++) {
      const current = candidates[i]
      // 如果这个值在本层已经被用过了，就跳过他
      if (visted.has(current)) { continue }
      visted.add(current)
      temp.push(current)
      dfs(i + 1, rest - current)
      temp.pop()
    }
  }
  dfs(0, target)
  return results
};
```

{% endnote %}

### 相邻元素去重

{% note Code %}

首先需要对 `candidates` 进行排序：

```typescript
candidates.sort((a, b) => a - b)
```

然后直接看 `for` 循环里面的逻辑：

```typescript
for (let i = start; i < candidates.length; i++) {
  if (i > start && candidates[i] === candidates[i - 1]) {
    continue
  }
  // ...
}
```

{% endnote %}

# 切割问题

切割问题其实与组合问题类似：

- **横向遍历** 为当前切割的终点；
- **纵向遍历** 为当前切割的终点已经确定，作为下次的起点。

模板代码如下：

```typescript
function partition(s: string) {
  const results = []
  const temp = []
  function dfs(start: number) {
    // 字符串遍历完成了，结束
    if (start === results.length) {
      results.push([...temp])
      return
    }
    // 这个字符串的结尾可以是 [start, s.length]
    for (let end = start; end < s.length; end++) {
      // 确定当前切割，并将当前的终止位 +1 作为下一次的起始位置
      temp.push(s.slice(start, end + 1))
      dfs(end + 1)
      temp.pop()
    }
  }
  dfs(0)
  return results
}
```

[LeetCode.131 Palindrome Partitioning](https://leetcode.cn/problems/palindrome-partitioning/description/) 只需要在每次切割完成之后判断回文串、[LeetCode.93 Restore IP Address](https://leetcode.cn/problems/restore-ip-addresses/description/) 只需要在每次切割完成之后判断合法 IP 即可。

# 子集问题

子集问题有个很重要的特征是，他不是在最后递归终止的时候保存结果的，而是在递归的过程中保存结果。

[LeetCode.78 Subsets](https://leetcode.cn/problems/subsets/description/) 就是最简单的子集问题：

{% note Code %}

```typescript
function subsets(nums: number[]): number[][] {
  const results = []
  const temp = []
  function dfs(start: number) {
    // 在递归的过程中收集结果集
    results.push([...temp])
    // 当遍历到原集合的最后一个元素时，自然终止
    for (let i = start; i < nums.length; i++) {
      temp.push(nums[i])
      dfs(i + 1)
      temp.pop()
    }
  }
  dfs(0)
  return results
};
```

{% endnote %}

[LeetCode.90 Subsets II](https://leetcode.cn/problems/subsets-ii/) 要求不能出现重复的子集，子集问题的去重与组合问题一模一样；[LeetCode.491 Non-dcreasing Subsequences](https://leetcode.cn/problems/non-decreasing-subsequences/) 只是多了一些额外的判断。

# 排列问题

排列问题跟组合问题有所不同，排列问题关心顺序，因此：

1. **排列问题每次都需要从 `0` 开始遍历**
2. **排列问题通常都需要纵向去重**

组合问题不需要考虑之前已经选过的项，因此到叶节点的路径上不会选择同一个元素，因此在纵向递归的过程中隐藏着 **可选元素数组** `selections` 随着 `start` 的增长而改变的逻辑，也就是说 **组合问题自带纵向去重**。

而排列问题没有 `start` 的帮忙，如果不做处理，**到叶节点的路径上就会多次出现同一个的元素**。因此需要每次递归之后都需要在 `selections` 中去手动去除路径中已有的元素，也就是说进行 **纵向去重（对递归进行去重）**，最简单的去重方法就是使用 `Set`。

[LeetCode.46 Permutations](https://leetcode.cn/problems/permutations/description/) 是一个最简单的全排列问题，注意其中的去重逻辑：

{% note Code %}

```typescript
function permute(nums: number[]): number[][] {
  const results = []
  const temp = []
  // 记录当前 DFS 到叶节点的路径上已经用了哪些元素，注意这里存的是下标！
  const used = new Set()
  function dfs() {
    if (temp.length === nums.length) {
      results.push([...temp])
      return
    }
    for (let i = 0; i < nums.length; i++) {
      // 跳过在路径上已经用过的元素
      if (used.has(i)) { continue }
      const current = nums[i]
      // 进入下一层之前，标记当前元素已使用
      used.add(i)
      temp.push(current)
      dfs()
      temp.pop()
      // 从下一层退出，删除当前元素已使用标记
      used.delete(i)
    }
  }
  dfs()
  return results
};
```

{% endnote %}

## 排列问题中的去重

[LeetCode.47 Permutations II](https://leetcode.cn/problems/permutations-ii/description/) 中要求像组合问题一样，对排列问题进行去重。因为该题中出现了相同值的元素，需要进行 **横向去重**，而 LeetCode.46 中每个元素的值都是不同的，不需要进行横向去重。

### 使用两个 Set 去重

既然纵向去重和横向去重都可以使用 `Set`，那么可以使用两个 `Set` 来分别对纵向和横向进行去重：

{% note Code %}

```typescript
function permuteUnique(nums: number[]): number[][] {
  const results = []
  const temp = []
  // 记录当前 DFS 到叶节点的路径上已经用了哪些元素，注意这里存的是下标！
  const deepVisited = new Set()

  function dfs() {
    if (temp.length === nums.length) {
      results.push([...temp])
      return
    }
    // 记录当前层遍历过哪些元素，注意这里是值！
    const breadthVisited = new Set()
    for (let i = 0; i < nums.length; i++) {
      // 如果路径上已经有该元素，跳过
      if (deepVisited.has(i)) { continue }
      const current = nums[i]
      // 如果当前层已经遍历过值相同的元素，跳过
      if (breadthVisited.has(current)) { continue }
      // 标记元素为当前层已用过
      breadthVisited.add(current)
      // 进入下一层之前，标记当前元素已使用
      deepVisited.add(i)
      temp.push(current)
      dfs()
      temp.pop()
      // 从下一层退出，删除当前元素已使用标记
      deepVisited.delete(i)
    }
  }
  dfs()
  return results
};
```

{% endnote %}

### 使用 used 数组进行去重

这是一个比较神奇的方法，将 Set 和 相邻元素去重结合了一下，首先需要对候选集进行排序，然后用数组 `used` 标记当前的使用状态：

```typescript
function permuteUnique(nums: number[]): number[][] {
  const results = []
  const temp = []
  nums.sort((a, b) => a - b)
  // 标记当前元素的使用状态，会将本次路径上的元素标记为 true，其实与 deepVisited Set 类似
  const used = new Array(nums.length).fill(false)

  function dfs() {
    if (temp.length === nums.length) {
      console.log(temp)
      results.push([...temp])
      return
    }
    for (let i = 0; i < nums.length; i++) {
      if (used[i]) { continue }
      // 如果当前元素与上一个相同并且 used[i - 1] === false
      // 也就是说当前路径中没有 used[i - 1]
      // 也就是说当前在纵向遍历的过程中，而是在横向遍历的过程中
      // 横向去重时不能选择值相同的元素
      if (i > 0 && nums[i] === nums[i - 1] && !used[i - 1]) { continue }
      const current = nums[i]
      // 将当前元素标记为在路径中使用
      used[i] = true
      temp.push(current)
      dfs()
      temp.pop()
      // 将当前元素标记为路径中未使用
      used[i] = false
    }
  }
  dfs()
  return results
};
```

# 棋盘问题

TBC
