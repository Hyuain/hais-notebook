---
title: Data Structure and Algorithm
date: 2022-02-05 15:49:12
categories:
  - - 计算机
mathjax: true
---

本文介绍了常见的数据结构、算法、一些典型例题及代码题中的特殊需要关注的地方。

按照章节来讲主要分为：

- 开始（复杂度）
- 数据结构（数组、字符串、栈、队列、链表、哈希表、二叉树、堆）
  - 数组：数组的基本操作、二分查找、排序算法、数组去重
  - 字符串：大数运算
  - 栈：栈的数组实现时间复杂度、单调栈、用栈实现队列、用队列实现栈
  - 队列：用数组实现环形队列
  - 二叉树：特殊的二叉树、遍历、二叉搜索树
  - 堆：堆与优先级队列及其实现
- 算法（回溯、贪心、动态规划）
  - 回溯：组合问题、切割问题、子集问题、排列问题、棋盘问题
  - 贪心：最大子数组和、加油站、分发糖果、重叠区间
  - 动态规划：记忆化深度优先搜索、找规律、背包问题
  
- 其他注意事项（对 1e9 + 7 取模）

<!-- more -->

# Complexity

> 复杂度

**最好情况与最坏情况**：通常我们只考虑最坏的情况。我们通常很难定义出 *平均情况*，而且也很难知道平均情况出现的概率。

**渐进分析**：描述函数在极限附近的行为，在算法分析中指分析数据集特别大的时候的性能

- 对于大多数算法，运行时间取决于输入的数据集大小
- 关注运行时间虽数据集增大而增加的增长率，而不是执行时间的绝对值
- 在书写代码之前预估执行时间

例如下面的伪代码用于找到数组的最大元素：

```text
arrayMax(A: array, n: lengthOfArray)
  currentMax = A[0]          // c1
  for (i from 1 to n - 1)    // (n - 1) * c2
    if (A[i] > currentMax)   // (n - 1) * c3
      currentMax = A[i]      // (n - 1) * c4 at most
  return currentMax          // c5
```

估计算法运行时间：T(n) = (n - 1) * (c2 + c3 + c4) + c1 + c5

## Big O Notation

> 大 O 表示法

**定义：存在 $g$，使得 $f(n) = O(g(n))$ 中 $n \rightarrow \infty$ 时，$f$ 的增长率小于等于 $g$ 的倍数。**

或者表示为：存在正的常数 $c$ 和 $n_0$，使得 $f(n) = O(g(n))$ 中，对于每一个 $n \ge n_0$ 都有 $0 \le f(n) \le cg(n)$。

比如 $f(n) = 3n^2+5$。那么有 $g(n) = n^2$，$f(n) = O(n^2)$，当 $n$ 趋于正无穷时， $f(n)$ 一定小于 $4g(n)$。

**大 O 表示法是一种讨论算法执行耗时的标准方法，他给出了计算复杂度的上界（最坏情况）。**

在大 O 表示法中，有一些常用的 **基本函数**，他们的大小关系如下：
$$
O(1) < O(log(n)) < O(n) < O (nlog(n)) < O(n^2) < O(n^3) < O(2^n) < O(n!) < O(n^n)
$$
**简化规则**：

- 忽略常数项：$3 = O(1)$，$3n = O(n)$

- 将常数个项组合起来：$3$ 个 $O(n)$ 仍然等于 $O(n)$
- 将 m 个项组合起来：$m$ 个 $O(1)$ 等于 $O(m)$
- 忽略相对更小的项：$O(n^2) + O(n) = O(n^2)$

**加法和乘法规则**：若 $T_1(n) = O(f(n))$，$T_2(n) = O(g(n))$

- $T_1 + T_2 = O(\max(f(n), g(n)))$
- $T_1 \cdot T_2 = O(f(n) \cdot g(n))$ 

# Array

> 数组是存放在连续内存空间上的相同类型数据的集合

- 数组的下标从 0 开始
- 数组的内存空间地址是连续的

## Operations of Array

| 操作 | 时间复杂度 |
| ---- | ---------- |
| 访问 | O(1)       |
| 查找 | O(n)       |
| 插入 | O(n)       |
| 追加 | O(1)       |
| 删除 | O(n)       |

## Searching in Array

### Binary Search

[LeetCode.204](https://leetcode.cn/problems/binary-search/submissions/)

前提

- 数组是有序的
- 数组中无重复元素

时间复杂度 O(log(n))

{% note Code %}
```typescript
// 左闭右闭写法
const binarySearch = (nums: number[], target: number) => {
  if (!nums.length) { return -1 }
  let left = 0
  let right = nums.length - 1
  // 当 left === right 时，区间 [left, right] 依旧有效
  while (l <= r) {
    let mid = Math.floor((left + right) / 2)
    if (nums[mid] === target) {
      return mid
    } else if (nums[mid] > target) {
      // 因为是右闭区间，新的 right 需要在下一次循环被考虑
      right = mid - 1
    } else {
      // 因为是左闭区间，新的 left 需要在下一次循环被考虑
      left = mid + 1
    }
  }
  return -1
}

// 左闭右开写法
const binarySearch = (nums: number[], target: number) => {
  if (!nums.length) { return -1 }
  let left = 0
  let right = nums.length
  // 当 left === right 时，区间 [left, right) 无效了
  while (l < r) {
    let mid = Math.floor((left + right) / 2)
    if (nums[mid] === target) {
      return mid
    } else if (nums[mid] > target) {
      // 因为是右开区间，新的 right 不需要被循环考虑
      right = mid
    } else {
      // 因为是左闭区间，新的 left 需要在下一次循环被考虑
      left = mid + 1
    }
  }
  return -1
}
```
{% endnote %}

## Sorting

### Selection Sort

每次从剩下的部分选择最小的排到前面来，O(n^2)

- 不稳定（交换操作通常都不稳定）
  - `8* 3 5 4 8 3 1 9` -> `1 3 5 4 8 8* 9`
- 原地

```js
const minIndex = numbers => {
  let index = 0
  for (let i = 1; i < numbers.length; i++) {
    if (numbers[i] < numbers[index]) {
      index = i
    }
  }
  return index
}

const selectionSort = numbers => {
  for (let i = 0; i < numbers.length - 1; i++){
    let index = minIndex(numbers.slice(i))
    if (index) {
      [numbers[index + i], numbers[i]] = [numbers[i], numbers[index + i]]
    }
  }  
  return numbers
}
```

### Insertion Sort

从后面未排序的数中取一个，插入前面已经排序好的数组中正确的位置，最坏 O(n^2)，最好 O(n)

- 稳定
- 原地

```js
const insertionSort = numbers => {
  for (let i = 0; i < numbers.length - 1; i++) {
    let j = i - 1
    const current = numbers[i]
    while (j >= 0 && current < numbers[j]) {
      numbers[j + 1] = numbers[j]
      j--
    }
    numbers[j + 1] = current
  }
  return numbers
}
```

### Bubble Sort

两两比较，如果后者比前者小，则交换，最终每次循环使得最大数冒泡到最后去，最坏 O(n^2)，最好 O(n)

- 稳定
- 原地

```js
const bubbleSort = numbers => {
  let length = numbers.length
  while (length > 1) {
    let swapped = false
    for (let i = 0; i < length; i++) {
      if (numbers[i + 1] < numbers[i]) {
        [numbers[i + 1], numbers[i]] = [numbers[i], numbers[i + 1]]
        swapped = true
      }
    }
    if (!swapped) break
    length--
  }
  return numbers
}
```

### Merge Sort

拆成左右两个数组，分别对左右数组进行排序（当长度为 1 的时候，自然是排好序的数组），再合并起来（排序实际上发生在合并的那一步），O(nlogn)

- 稳定
- 非原地

```js
const merge = (left, right) => {
  let i = 0
  let j = 0
  const merged = []
  while (i < left.length && j < right.length) {
    if (left[i] <= right[j]) {
      merged.push(left[i])
      i++
    } else {
      merged.push(right[j])
      j++
    }
  }
  if (i === left.length) {
    // left 已经合并完了，将 right 剩下的部分放进去
    merged.push(...right.slice(j))
  } else {
    // right 已经合并完了，将 left 剩下的部分放进去
    merged.push(...left.slice(i))
  }
  return merged
}

const mergeSort = numbers => {
  // Conquer
  if (numbers.length <= 1) { return numbers }
  // Devide
  const divisionIndex = Math.floor(numbers.length / 2)
  // Combine
  return merge(
    // Conquer
    mergeSort(numbers.slice(0, divisionIndex)),
    mergeSort(numbers.slice(divisionIndex))
  )
}
```

### Quick Sort

找出 Pivot，把小于 Pivot 的放左边，大于 Pivot 的放右边，最好 O(nlogn)，最坏 O(n^2)

- 不稳定
- 原地

```js
// 非原地写法
const quickSort = numbers => {
  if (numbers.length < 2) return numbers
  // Devide
  let pivotIndex = Math.floor(numbers.length / 2)
  let pivot = numbers[pivotIndex]
  let left = []
  let right = []
  // Conquer
  for (let i = 0; i < numbers.length; i++) {
    if (i === pivotIndex) { continue }
    if (numbers[i] <= pivot) {
      left.push(numbers[i])
    } else {
      right.push(numbers[i])
    }
  }
  // Combine
  return quickSort(left).concat([pivot], quickSort(right))
}
```

```js
// 原地写法
const partition = (numbers, l, r) => {
  // 选择最后一个数作为 pivot
  const pivot = numbers[r]
  // 相当于整个数组被分为
  // |左边部分|右边部分|未遍历数组|pivot|
  // i 表示 pivot 左边部分的末尾的下一项（即右边数组的第一项）
  let i = l
  // j 表示未遍历数组的第一项
  for (let j = l; j < r; j++) {
    if (numbers[j] < pivot) {
      // 如果该项小于 pivot，应该放到左边部分中，于是 j 和 i 互换
      // 因为 i 是左边部分最后一项的下一位
      [numbers[i], numbers[j]] = [numbers[j], numbers[i]]
      i++
    }
  }
  // 右边部分的第一项与 pivot 互换，使得 pivot 在左右部分的中间
  [numbers[i], numbers[r]] = [numbers[r], numbers[i]]
  // 返回 pivot 的下标
  return i
}

const quickSort = (numbers, l = 0, r = numbers.length - 1) => {
  // l 表示需要考虑的切片的开始的坐标
  // r 表示需要考虑的切片的结束的坐标（l r 都需要被考虑）
  // l == r 时，只有一项，不需要再算了
  if (l < r) {
    const pivotIndex = partition(numbers, l, r)
    quickSort(numbers, l, pivotIndex - 1)
    quickSort(numbers, pivotIndex + 1, r)
  }
  return numbers
}
```

### Heap Sort

见 [堆](#heap) 章节。

### Sorting in JavaScript

早年的 ECMAScript Spec 并没有规定 `Array#sort` 方法的稳定性，甚至有的 JavaScript Engine 在短数组时是稳定的（插入排序），长数组中不稳定（快速排序）。但 V8 向 ECMAScript 发起了 [数组稳定性的提案](https://v8.dev/features/stable-sort)，并且已经被采纳了，现在的 JavaScript Engine （V8 v7.0 / Chrome 70）均已支持稳定的数组排序。

但具体采用什么样的算法来实现数组排序，在不同的 JavaScript Engine 中有着不同的实现。比如 V8 采用的是 [Timsort](https://v8.dev/blog/array-sort#timsort)。

#### Sorting in V8

- 在 JavaScript 这样的动态类型语言中，比较比访问内存更昂贵，因为两个数的比较通常涉及到用户代码的调用。
- 在 JavaScript 中，用户可以做很多奇怪的事情：
  - 在比较函数中改变数组；
  - 数组中可能有对象，其有 `toString` 方法，而我们默认的比较函数就会调用 `toString` 方法去比较；
  - 给数组的某些项增加访问器（getter 和 setter）；
  - 构造出原型链很奇怪的对象，并且要给这个对象数组排序。

##### Sorting Process

> 收集所有的非 `undefined` 值，放入一个临时列表，为这个临时列表排序，然后将其写回原来的数组或对象中。这使得 V8 可以在真正的排序中不用再关心访问器和原型链。

规范要求 `Array#sort` 排序好的列表可以分为三个部分：

1. 所有的非 `undefined` 值根据比较函数排序；
2. 所有的 `undefined`；
3. 所有的空洞（稀疏数组）。

实际上排序算法只需要应用给第一部分就可以了。

##### Before Sorting

V8 的排序预处理步骤如下：

1. 将 `length` 赋值为数组或对象的 `length` 属性；
2. 将 `numberOfUndefineds` 赋值为 `0`；
3. 对于所有在 `[0, length)` 区间内的 `value`：
   1. 如果 `value` 是空洞，什么也不做；
   2. 如果 `value` 是 `undefined`，`numberOfUndefineds++`；
   3. 否则将 `value` 加入临时列表 `elements`。

这样处理之后，所有的非 `undefined` 值被存在了 `elements` 中，所有的 `undefined` 仅仅只是被计数了（`undefiend` 不会被传给用户提供的比较函数）。

##### After Sorting

排序完成后，排序好的值将会被写回原来的数组或对象中。V8 的排序后处理步骤如下：

1. 将 `elements` 写回原来数组的 `[0, elements.length)` 区间内；
2. 将 `[elements.length, elements.length + numberOfUndefineds)` 区间设置为 `undefined`；
3. 删除区间 `[elements.length + numberOfUndefineds, length)` 的值。

##### During Sorting: Timsort

- 快排的好处是原地排序，可以减少内存占用。缺点是不稳定，并且最坏的情况下时间复杂度是 O(n^2)。

- Timsort 是 2002 年 Tim Peters 为 Python 开发的。
- Timsort 是迭代调用的（而归并排序通常是递归调用）。

> To Be Completed.

## Deduplication

> 恰当地利用对象和 Map key 的唯一性可以实现对 `'1'` 和 `1` 的区分、对象的去重，以及 `undefined` `null` `NaN` 的良好识别。

使用数组 `[ 1, 1, '1', '1', 0, 0, { a: 1 }, { a: 1 }, x, y, z, undefined, undefined, null, null, NaN, NaN ]` 输出进行实验，其中：

```js
const x = { b: 1 }
const y = { b: 1 }
const z = { a: 1 }
```

### 双重循环

```js
Array.prototype.unique = function() {
  const newArray = []
  let isRepeat
  for (let i = 0; i < this.length; i++) {
    isRepeat = false
    for (let j = 0; j < newArray.length; j++) {
      if(this[i] === newArray[j]){
        isRepeat = true
        break
      }
    }
    if (!isRepeat) {
      newArray.push(this[i])
    }
  }
  return newArray
}
```

```js
[ 1, 1, '1', '1', 0, 0, { a: 1 }, { a: 1 }, x, y, z, undefined, undefined, null, null, NaN, NaN ]
    |
    ↓
[ 1, '1', 0, { a: 1 }, { a: 1 }, { b: 1 }, { b: 1 }, { a: 1 }, undefined, null, NaN, NaN]
```

不能区分：对象，`NaN`

### `indexOf`

```js
Array.prototype.unique = function() {
  const newArray = []
  for (let i = 0; i < this.length; i++) {
    const current = this[i]
    if (newArray.indexOf(current) < 0) {
      newArray.push(current)
    }
  }
  return newArray
}
```

```js
[ 1, 1, '1', '1', 0, 0, { a: 1 }, { a: 1 }, x, y, z, undefined, undefined, null, null, NaN, NaN ]
    |
    ↓
[ 1, '1', 0, { a: 1 }, { a: 1 }, { b: 1 }, { b: 1 }, { a: 1 }, undefined, null, NaN, NaN]
```

不能区分：对象，`NaN`

### 相邻元素去重（先排序）

```js
Array.prototype.unique = function() {
  const newArray = []
  this.sort()
  for (let i = 0; i < this.length; i++) {
    if(this[i] !== this[i+1]) {
      newArray.push(this[i])
    }
  }
  return newArray
}
```

```js
[ 1, 1, '1', '1', 0, 0, { a: 1 }, { a: 1 }, x, y, z, undefined, undefined, null, null, NaN, NaN ]
    |
    ↓
[ 0, 1, '1', NaN, NaN, { a: 1 }, { a: 1 }, { b: 1 }, { b: 1 }, { a: 1 }, null]
```

注意：原来的数组会被更改！

不能区分：对象，`NaN`，`undefined`（会丢失）

### `filter`

```js
Array.prototype.unique = function() {
  return newArray = this.filter((item, index, array) => {
    return array.indexOf(item) === index
  })
}
```

```js
[ 1, 1, '1', '1', 0, 0, { a: 1 }, { a: 1 }, x, y, z, undefined, undefined, null, null, NaN, NaN ]
    |
    ↓
[ 1, '1', 0, { a: 1 }, { a: 1 }, { b: 1 }, { b: 1 }, { a: 1 }, undefined, null]
```

不能区分：对象，`NaN`（会丢失）

### `reduce`（先排序）

```js
Array.prototype.unique = function() {
  return this.sort().reduce((result, current) => {
    if (result.length === 0 || result[result.length - 1] !== current) {
      result.push(current)
    }
    return result
  }, [])
}
```

```js
[ 1, 1, '1', '1', 0, 0, { a: 1 }, { a: 1 }, x, y, z, undefined, undefined, null, null, NaN, NaN ]
    |
    ↓
[ 0, 1, '1', NaN, NaN, { a: 1 }, { a: 1 }, { b: 1 }, { b: 1 }, { a: 1 }, null, undefined]
```

注意：原来的数组会被更改！

不能区分：对象，`NaN`

### `includes`

```js
Array.prototype.unique = function() {
  const newArray = []
  this.forEach(item => {
    if (!newArray.includes(item)) {
      newArray.push(item)
    }
  })
  return newArray
}
```

```js
[ 1, 1, '1', '1', 0, 0, { a: 1 }, { a: 1 }, x, y, z, undefined, undefined, null, null, NaN, NaN ]
    |
    ↓
[ 1, '1', 0, { a: 1 }, { a: 1 }, { b: 1 }, { b: 1 }, { a: 1 }, undefined, null, NaN]
```

注意：原来的数组会被更改！

不能区分：对象

### 对象键值对

利用对象中 key 的不可重复性去重

```js
Array.prototype.unique = function() {
  const obj = {}
  const newArray = []
  for (let i = 0; i < this.length; i++) {
    if (!obj[this[i]]) {
      obj[this[i]] = true
      newArray.push(this[i])
    }
  }
  return newArray
}
```

```js
[ 1, 1, '1', '1', 0, 0, { a: 1 }, { a: 1 }, x, y, z, undefined, undefined, null, null, NaN, NaN ]
    |
    ↓
[ 1, 0, { a: 1 }, undefined, null, NaN]
```

不能区分：对象（存进去的时候所有对象都变成了 `[object Object]`）、`'1'` 和 `1`

#### 改进版本 1

可以区分 `'1'` 和 `1`

```js
Array.prototype.unique = function() {
  const obj = {}
  const newArray = []
  for (let i = 0; i < this.length; i++) {
    if (!obj[typeof this[i] + this[i]]) {
      obj[typeof this[i] + this[i]] = true
      newArray.push(this[i])
    }
  }
  return newArray
}
```

#### 改进版本 2

可以区分对象

```js
Array.prototype.unique = function() {
  const obj = {}
  const newArray = []
  for (let i = 0; i < this.length; i++) {
    if(!obj[typeof this[i] + JSON.stringify(this[i])]) {
      obj[typeof this[i] + JSON.stringify(this[i])] = true
      newArray.push(this[i])
    }
  }
  return newArray
}
```

```js
[ 1, 1, '1', '1', 0, 0, { a: 1 }, { a: 1 }, x, y, z, undefined, undefined, null, null, NaN, NaN ]
    |
    ↓
[ 1, '1', 0, { a: 1 }, { b: 1 }, undefined, null, NaN]
```

### Map

利用 Map 的 key 唯一

```js
Array.prototype.unique = function() {
  const newArray = []
  const map = new Map()
  for (let i = 0; i < this.length; i++) {
    if(!map.get(this[i])) {
      map.set(this[i], true)
      newArray.push(this[i])
    }
  }
  return newArray
}

// 简化版
Array.prototype.unique = function() {
  const map = new Map()
  return this.filter((item) => (!map.has(item) && map.set(item, true)))
}
```

```js
[ 1, 1, '1', '1', 0, 0, { a: 1 }, { a: 1 }, x, y, z, undefined, undefined, null, null, NaN, NaN ]
    |
    ↓
[ 1, '1', 0, { a: 1 }, { a: 1 }, { b: 1 }, { b: 1 }, { a: 1 }, undefined, null, NaN]
```

不能去重：对象，可以像上面的对象键值对方法一样处理

### Set

Set 类似于数组，但成员的值都是唯一的

```js
Array.prototype.unique = function() {
  return Array.from(new Set(this))
}

// 可以简化为
Array.prototype.unique = function() {
  return [...new Set(this)]
}

// 还可以简化为
```

```js
[ 1, 1, '1', '1', 0, 0, { a: 1 }, { a: 1 }, x, y, z, undefined, undefined, null, null, NaN, NaN ]
    |
    ↓
[ 1, '1', 0, { a: 1 }, { a: 1 }, { b: 1 }, { b: 1 }, { a: 1 }, undefined, null, NaN]
```

不能去重：对象

## Examples

### 电话号码的组合

[LeetCode.17](https://leetcode.cn/problems/letter-combinations-of-a-phone-number/)

Given a string containing digits from 2-9 inclusive, return all possible letter combinations that the number could represent. Return the answer in any order.

A mapping of digits to letters (just like on the telephone buttons) is given below. Note that 1 does not map to any letters.

{% note Code %} 

#回溯

  ```typescript
  function letterCombinations(digits: string): string[] {
    const letters = [
      "", "", "abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz"
    ]
    const result = []
    const currentChosenLetters = []
    function trackingback() {
      if (currentChosenLetters.length === digits.length) {
        if (currentChosenLetters.length) {
          result.push(currentChosenLetters.join(""))
        }
        return
      }
      const currentAvaliableLetters = letters[digits[currentChosenLetters.length]]
      for (let i = 0; i < currentAvaliableLetters.length; i++) {
        currentChosenLetters.push(currentAvaliableLetters[i])
        trackingback()
        currentChosenLetters.pop()
      }
    }
    trackingback()
    return result
  };
  ```

{% endnote %}

### 卡牌分组 （归类运算）

[LeetCode.914](https://leetcode.cn/problems/x-of-a-kind-in-a-deck-of-cards/)

{% note Code %}

问题的核心是求最大公约数：

```text
辗转相除： gcd(a, b) = gcd(b, a % b)
更相减损： gcd(a, b) = a > b ? gcd(a - b, b) : gcd(a, b - a)
```

```js
const cardGroup = (arr) => {
  // 先排序
  const str = arr.sort().join('')
  // 再将相同的进行分组（多张或者单张为一组）
  const group = str.match(/(\d)\1+|\d/g)
  // 求最大公约数
  const gcd = (a, b) => {
    if (b === 0) {
      return a
    } else {
      return gcd(b, a % b)
    }
  }
  // 进入处理之前检查，如果 group 为空或者 group.length === 1，就不找公约数
  while (group.length > 1) {
    const a = group.shift().length
    const b = group.shift().length
    const v = gcd(a, b)
    if (v === 1) {
      return false
    } else {
      group.unshift('0'.repeat(v))
    }
  }
  // 如果 group 为空，或者每组的数量小于 1，则为 false
  return group.length ? group[0].length > 1 : false
}
```

{% endnote %}

# String

## Big Number Calculation

### Big Number Plus

{% note Code %}

```ts
function addStrings(num1: string, num2: string): string {
    const arr1 = num1.split('')
    const arr2 = num2.split('')
    const results = []
    let carry = 0
    while (arr1.length || arr2.length) {
        const n1 = Number(arr1.pop()) || 0
        const n2 = Number(arr2.pop()) || 0
        let result = n1 + n2 + carry
        carry = Math.floor(result / 10)
        result = result % 10
        results.unshift(result)
    }
    if (carry) {
        results.unshift(carry)
    }
    return results.join('')
};
```

{% endnote %}

### Big Number Minus

{% note Code %}

```ts
function minusString(num1: string, num2: string): string {
  if (num1 === num2) {
    return '0';
  }
  const isNum1Bigger = compare(num1, num2);
  // 让 arr1 永远是大的那个数
  const [arr1, arr2] = isNum1Bigger
    ? [num1.split(''), num2.split('')]
    : [num2.split(''), num1.split('')];
  let borrow = 0;
  const results = [];
  while (arr1.length || arr2.length) {
    const n1 = Number(arr1.pop()) || 0;
    const n2 = Number(arr2.pop()) || 0;
    let result = n1 - n2 - borrow;
    if (result < 0) {
      result += 10;
      borrow = 1;
    } else {
      borrow = 0;
    }
    results.unshift(result);
  }
  const firstNoneZeroIndex = results.findIndex((item) => item);
  return `${isNum1Bigger ? '' : '-'}${results
    .slice(firstNoneZeroIndex)
    .join('')}`;
}

function compare(num1: string, num2: string): boolean {
  if (num1.length !== num2.length) {
    return num1.length > num2.length;
  }
  for (let i = 0; i < num1.length; i++) {
    if (num1[i] > num2[i]) {
      return true;
    } else if (num1[i] < num2[i]) {
      return false;
    }
  }
  return true;
}
```

{% endnote %}

## Pattern Matching

### Single-Pattern: KMP

Knuth-Morris-Pratt (KMP) 算法是一种字符串查找算法。他的核心在于：

1. 两个指针分别用来遍历文本串和模式串；
2. 当遍历到不匹配的位置时，遍历模式串的指针并不会退回最开始重新匹配，而是找到上一次匹配的位置重新尝试，**前缀表（next）** 就记录了上一次匹配的位置。

前缀表是用来指示模式串指针回退位置的，换句话说，**他记录了模式串与文本串不匹配的时候，模式串应该从哪里重新开始匹配。**

具体来说，前缀表记录了下标 `i` 之前（包括 `i`）的字符串中，有多长相同的 **前缀** 和 **后缀**。

前缀是指，不包含最后一个字符的、所有以第一个字符开头的连续子串，比如 `aabaaf`，`a` 是前缀，`aa` 是前缀，甚至 `aabaa` 也能算前缀；

后缀则相反，是指不包含第一个字符的、所有以最后一个字符结尾的连续字串，比如上述的 `abaaf` 是后缀，`baaf` 是后缀，`af` 是后缀，`f` 也是后缀。

来看一个前缀表：

```text
a a b a a f
0 1 0 1 2 0
```

`aa` 的前后缀有一位相同（`a`），所以是 `1`

`aab` 前后缀不相同，所以是 `0`

`aaba` 的前后缀有一位相同（`a`），所以是 `1`

`aabaa` 的前后缀有两位相同（`aa`），所以是 `2`

`aabaaf` 的前后缀不相同，所以是 `0` 

暴力匹配的时间复杂度是 O(n * m)，因为对于长度为 n 的文本串的每一位，都至多要遍历 m 位的模式串；KMP 的时间复杂度是 O (n + m)，因为 KMP 需要先遍历长度为 m 的模式串生成前缀表 `next`，然后再遍历长度为 n 的文本串。

以下是 KMP 算法的流程图，可以看到前缀表的生成与字符串匹配的流程其实基本是一样的，都是看当前 `i` 和 `j` 指向的字符是否相等，如果不相等，`j` 就退回之前的位置（`next[j - 1]`）。不同在于，生成前缀表时 `i` 代表后缀的最后一位，从 `1` 开始；匹配字符串时 `i` 代表文本串指针，从 `0` 开始。

![KMP 算法流程图](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/KMP.png)

{% note Code %}

```javascript
// 返回 p 在 s 中出现的位置，-1 为未找到
function strStr(s, p): number {
  // 构建前缀表 next
  const next = new Array(p.length).fill(0)
  // i 指向后缀的末尾
  let i = 1
  // j 指向前缀的末尾
  let j = 0
  while (i < p.length) {
    while (p[i] !== p[j] && j > 0) {
      j = next[j - 1]
    }
    if (p[i] === p[j]) {
      j++
      next[i] = j
      i++
    } else {
      i++
    }
  }
  
  // 匹配字符串
  // i 指向文本串
  i = 0
  // j 指向模式串
  j = 0
  while (i < s.length) {
    while (s[i] !== p[j] && j > 0) {
      j = next[j - 1]
    }
    if (s[i] === p[j]) {
      if (j === p.length - 1) {
        return i - p.length + 1
      }
      j++
      i++
    } else {
      i++
    }
  }
  return -1
};

```

{% endnote %}

有的实现中会将 `next` 数组中的所有值都初始化为 `-1`，且 `j` 也从 `-1` 开始，本质没有区别。

### Multi-Parttern: AC

> 字符串的多模式匹配常用 Aho-Corasick Automaton（AC 自动机）来解决，其基础是 字典树（Tried）和 KMP 算法。

#### Trie

如下图所示，字典树（Trie）是一个像字典一样的树，**字典树的每个边代表了一个字母，从根节点到某个节点的路径就代表了一个字符串**。

![Trie](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/trie1.png)

可以用 JavaScript 来实现一个 Trie：

{% note Code %}

```javascript
// Trie 节点的实现
class TrieNode {
  constructor(key) {
    // 该节点代表的字符（指向该节点的边代表的字符）
    this.key = key
    // 指向父结点
    this.parent = null
    // 子结点哈希表，该表的键是边所对应的字符，值是子结点的引用
    this.children = {}
    // 看是否是某个字符串的最后一个字符
    this.end = false
  }
  
  // 迭代父结点，得到该节点为终点的字符串
  // 时间复杂度 O(k)，k = 字符串长度
  getWord() {
    const output = []
    const node = this
    
    while (node !== null) {
      output.unshift(node.key)
      node = node.parent
    }
    
    return output.join('')
  }
}
```

```javascript
// Tire 的实现
class Trie {
  root = new TrieNode(null)
  
  constructor() {
  }
  
  // 向 Trie 插入一个字符串
  // 时间复杂度 O(k)，k = 字符串长度
  insert() {
    const node = this.root
    
    for (let i = 0; i < word.length; i++) {
      // 如果该字符在 children 中不存在，就创建一个
      if (!node.children[word[i]]) {
        node.children[word[i]] = new TrieNode(word[i])
        node.children[word[i]].parent = node
      }
      // 进入下一层
      node = node.children[word[i]]
      // 如果是字符串最后一个字符，标记为 end
      if (i === word.length - 1) {
        node.end = true
      }
    }
  }
  
  // 检查某个字符串是否是 Trie 中的一整个字符串
  // 时间复杂度 O(k)，k = 字符串长度
  contains(word) {
    let node = this.root
    
    for (let i = 0; i < word.length; i++) {
      if (node.children[word[i]]) {
        node = node.children[word[i]]
      } else {
        return false
      }
    }
    
    // word 已经遍历完了，看最后这个节点有没有标记 end，是不是某个字符串的终点
    return node.end
  }

  // 返回具有该 prefix 的所有字符串
  // 时间复杂度 O(p + n)，p = prefix.length，n = 子路径的数量
  find(prefix) {
    let node = this.root
    const output = []
    
    for (let i = 0; i < prefix.length; i++) {
      if (node.childrenp[prefix[i]]) {
        node = node.children[prefix[i]]
      } else {
        return output
      }
    }
    
    findAllwords(node, output)
    
    return output
  }
         
  // 递归获取所有以该节点为起始的字符串
  findAllWords(node, arr) {
    if (node.end) {
      arr.unshift(node.getWord())
    }
    for (const child of Object.values(node.children)) {
      findAllWords(child, arr)
    }
  }
}
```

{% endnote %}

#### AC

// TBC

## Examples

### 反转字符串中的单词

[LeetCode.557](https://leetcode.cn/problems/reverse-words-in-a-string-iii/)

```js
const reverse = (str) => {
  const array = str.split(' ')
  const result = array.map(item => {
    return item.split('').reverse().join('')
  })
  return result.join(' ')
}

// 更优雅的写法
const reverse = (str) => {
  return str
          .split(' ')
          .map(item => item.split('').reverse().join(''))
          .join(' ')
}

// 使用正则表达式
const reverse = (str) => {
  return str
          .split(/\s/g)
          .map(item => item.split('').reverse().join(''))
          .join(' ')
}

// 使用 match API
const reverse = (str) => {
  return str
          .match(/[\w']+/g)
          .map(item => item.split('').reverse().join(''))
          .join(' ')
}
```

### 计数二进制子串

[LeetCode.696](https://leetcode.cn/problems/count-binary-substrings/)

```js
const count = (str) => {
  // 建立数据结构，堆栈、保存数据
  const result = []
  // 给定任意子输入都返回第一个符合条件的字符串
  const match = (str) => {
    const j = str.match(/^(0+|1+)/)[0]
    const o = (j[0] ^ 1).toString().repeat(j.length)
    const reg = new RegExp(`${j}${o}`)
    if (reg.test(str)) {
      return RegExp.$1
    } else {
      return ''
    }   
  }
  // 通过 for 循环控制程序运行流程
  for (let i = 0, len = str.length - 1; i < len; i++) {
    const sub = match(str.slice(i))
    if(sub){
      result.push(sub)
    }
  }
  return result
}
```

# Stack

> 栈，后进先出（Last In First Out, LIFO）

## Implementation of Stack

一般可以用数组来实现栈，其对应的时间复杂度如下：

| 操作    | 时间复杂度 |
| ------- | ---------- |
| Push    | O(1)       |
| Pop     | O(1)       |
| Top     | O(1)       |
| Search* | O(n)       |

*栈一般也不提供搜索操作

我们可以用 JavaScript 非常简单地实现栈，之后的相关代码可能会直接使用这段代码所定义的 `Stack` 类：

```javascript
class Stack {
  array = []
  constructor() {}
  push(value) {
    return this.array.push(value)
  }
  pop() {
    return this.array.pop()
  }
  get size() {
    return this.array.length
  }
  get top() {
    return this.array[this.array.length - 1]
  }
}
```

## Monotonic Stack

**何时使用单调栈：**一维数组，要寻找任一元素的右边或左边第一个比自己大或自己小的位置。

**单调栈的核心思想在于 “清算”**，当遍历的 **当前元素 `x`** 满足某种条件之后，可以对之前保存在栈内的部分元素进行 **清算**。清算完成之后，就再也不用考虑那些元素了，所以通常来讲，使用单调栈时，元素遍历的过程中会至多入栈一次、出栈一次，时间复杂度为 O(n)。

### Examples

#### 找到下一个大于自己的元素

比如题目 [LeetCode.496 下一个更大元素 I](https://leetcode.cn/problems/next-greater-element-i/) 题目中，输出右边第一个 **大于** 自己的位置。比如 `[73, 74, 75, 71, 71, 72, 76, 73]` 输出应该是 `[1, 2, 6, 5, 5, 1, -1, -1]`。

**清算条件：若当前遍历的元素 `x` 大于栈内元素，即可对栈内元素进行清算（即栈内元素找到了右边第一个大于自己的元素）。** 

具体来讲：

1. 维持一个 **栈底到栈顶单调递减** 的单调栈。这里我们单调栈里面可以存放元素的下标（存放什么可以根据题目的需要定义，重要的是单调栈的核心思想，这个看下面模拟会有标注）。

2. 遍历一遍数组，每次将 **遍历到的元素 `x`** 与 **栈顶元素 `top`** 相比较，有两种操作：

   - **如果 `x > top`**，说明栈顶元素 `top` 找到了下一位比自己大的元素，**可以对栈顶进行清算**，即栈顶出栈并记录结果；同时新的栈顶也要与当前遍历的元素比较，直到小于等于当前元素，不再满足清算条件

   - **如果 `x <= top`**，说明 **栈内元素都不满足清算条件**，还需要继续找，将其入栈（这样效果上就达成了一个栈底到栈顶递减的单调栈）

模拟步骤如下：

{% note Steps %}

```javascript
const stack = []
const result = [-1, -1, -1, -1, -1, -1, -1, -1]

/*  i == 0  */
!stack.size
stack.push(0)
result: [-1, -1, -1, -1, -1, -1, -1, -1]
stack: [0]

/*  i == 1  */
74 > stack.top(73), 需要在 73 的位置标记已经找到了 74；73 的下一个大的已经找到，可以出栈
  result[stack.pop()] = i, result[0] = 1
stack.push(1)
result: [1, -1, -1, -1, -1, -1, -1, -1]
stack: [1]

/*  i == 2  */
75 > stack.top(74), 需要在 74 的位置标记已经找到了 75；74 的下一个大的已经找到，可以出栈
  result[stack.pop()] = i, result[1] = 2
stack.push(2)
result: [1, 2, -1, -1, -1, -1, -1, -1]
stack: [2]

/*  i == 3  */
71 < stack.top(75), 将 71 入栈，待查找
stack.push(3)
result: [1, 2, -1, -1, -1, -1, -1, -1]
stack: [2, 3]

/*  i == 4  */
71 == stack.top(71), 将 71 入栈，待查找
stack.push(4)
result: [1, 2, -1, -1, -1, -1, -1, -1]
stack: [2, 3, 4]

/*  i == 5  */
72 > stack.top(71), 需要在 71 的位置标记已经找到了 72；71 的下一个大的已经找到，可以出栈
  result[stack.pop()] = i, result[4] = 5
  出栈后 72 仍然大于 stack.top(71)，继续出栈，直到小于等于 stack.top 或栈为空
  result[3] = 5
stack.push(5)
result: [1, 2, -1, 5, 5, -1, -1, -1]
stack: [2, 5]

/*  i == 6  */
76 > stack.top(72), 需要在 72 的位置标记已经找到了 76；72 的下一个大的已经找到，可以出栈
  result[stack.pop()] = i, result[5] = 6
  出栈后 76 仍然大于 stack.top(75)，继续出栈，直到小于等于 stack.top 或栈为空
  result[2] = 6
stack.push(6)
result: [1, 2, 6, 5, 5, 6, -1, -1]
stack: [6]

/*  i == 7  */
73 < stack.top(76), 将 73 入栈，待查找
stack.push(7)
result: [1, 2, 6, 5, 5, 6, -1, -1]
stack: [6, 7]

/*  End  */
```

{% endnote %}

#### 将中缀表达转换为后缀表达

计算器中将中缀表达转为后缀表达其实也类似于单调栈的思想：

- 遍历中缀表达式，遇到运算符的时候，如果栈顶的运算符优先级更高或相同，**对这个优先级更高的运算符进行清算**（即将其出栈）。直到遇到优先级更低的运算符或栈空时，将当前运算符入栈
- 栈维持了从 **栈底到栈顶运算符优先级递增**
- 转为后缀表达式其实是要找 **下一个优先级相同或更低的运算符的位置**，因为 **下一个优先级相同或更低的运算符的位置** 就是 **当前运算符覆盖范围截止的位置**

{% note Steps %}

```javascript
1 + 2 * 3 - 4  => 1 2 3 * + 4 -

const stack = []
const result = []

/*  i == 0  */
数字 1 直接放入结果
result: [1]

/*  i == 1  */
!stack.size:
  stack.push('+')
stack: ['+']
result: [1]

/*  i == 2  */
数字 2 直接放入结果
stack: ['+']
result: [1, 2]

/*  i == 3  */
* 比 stack.top('+') 优先级更高，stack.push('*')
stack: ['+', '*']
result: [1, 2]

/*  i == 4  */
数字 3 直接放入结果
stack: ['+', '*']
result: [1, 2, 3]

/*  i == 5  */
- 比 stack.top('*') 优先级低，result.push(stack.pop())，说明 * 负责的范围已经结束了（2 和 3）
- 跟 stack.top('+') 优先级相同，result.push(stack.pop())，说明 + 负责的范围已经结束了（1 和 2 * 3）
stack.push('-')
stack: ['-']
result: [1, 2, '*', '+']

/*  i == 6  */
数字 4 直接放入结果
stack: ['-']
result: [1, 2, '*', '+', 4]

/*  End  */
清空 stack，按照出栈顺序加入结果
result: [1, 2, '*', '+', 4, '-']
```

{% endnote %}

#### 接雨水

[LeetCode.42 接雨水](https://leetcode.cn/problems/trapping-rain-water/) 是一道比较难的单调栈题目，需要找到 **清算条件** 和 **清算方法**。

**清算条件：如果当前遍历的地势比栈中的高，可以对栈中的较低地势处的积水进行清算。**

**清算方法：找到较低地势的深度和宽度，计算其积水。**

{% note Code %}

```typescript
function trap(height: number[]): number {
  // 维护一个单调递减的栈
  const stack = new Stack()
  let result = 0
  for (let i = 0; i < height.length; i++) {
    while (stack.size && height[stack.top] < height[i]) {
      // 当前的地势比栈顶的地势高，可以对栈顶进行清算
      // 将栈顶出栈，拿到他的 index
      const lowestIndex = stack.pop()
      // 若栈为空，说明 lowestIndex 的左边没有地势更高的地方，不能形成积水
      if (!stack.length) {
        continue
      }
      // 当前积水的深度，取 min(lowestIndex 左边较高地势, lowestIndex 右边较高地势) - lowestIndex 的地势
      const depth = Math.min(height[stack.top], height[i]) - height[lowestIndex]
      // 左边较高地势的 index
      const left = stack.top
      // 右边较高地势的 index
      const right = i
      // 当前的积水
      const water = (right - left - 1) * depth
      result += water
    }
    stack.push(i)
  }
  return result
};
```

{% endnote %}

#### 柱状图中的最大矩形

[LeetCode.84 柱状图中的最大矩形](https://leetcode.cn/problems/largest-rectangle-in-histogram/description/) 与接雨水的清算方法是类似的，不过清算条件恰好相反。

**清算条件：如果当前遍历的矩形比栈中的低，那么对以栈中的项进行清算，找到以其为高的矩形面积。**

**清算方法：找到较高的矩形的宽度，计算其面积。**

{% note Code %}

```typescript
function largestRectangleArea(heights: number[]): number {
  // 维护一个单调递增的栈
  const stack = new Stack()
  let maxArea = 0
  for (let i = 0; i < heights.length; i++) {
    while (stack.size && heights[stack.top] > heights[i]) {
      // 当前的柱更低，可以对以更高的柱为高度的矩形面积进行清算了
      // 更高的柱的高度
      const height = heights[stack.pop()]
      // 以更高的柱为高的矩形面积的左边界
      const left = stack.size ? stack.top : -1
      // 以更高的柱为高的矩形面积的右边界
      const right = i
      // 矩形的面积
      const area = (right - left - 1) * height
      maxArea = Math.max(maxArea, area)
    }
    stack.push(i)
  }
  // 栈中还剩下没有清算的柱，遍历一遍进行清算
  while (stack.size) {
    const height = heights[stack.pop()]
    const left = stack.size ? stack.top : -1
    const right = heights.length
    const area = (right - left - 1) * height
    maxArea = Math.max(maxArea, area)
  }
  return maxArea
};
```

{% endnote %}

该题可以通过给原来的柱状图的左右添加高度为 0 的项，让栈中剩余的元素在遍历到数组最后一项时自行清算：

{% note Code %}

```typescript
function largestRectangleArea(heights: number[]): number {
  const stack = new Stack()
  let maxArea = 0
  heights.unshift(0)
  heights.push(0)
  stack.push(0)
  for (let i = 1; i < heights.length; i++) {
    while (stack.length && heights[stack.top] > heights[i]) {
      const height = heights[stack.pop()]
      const left = stack[stack.top]
      const right = i
      const area = (right - left - 1) * height
      maxArea = Math.max(maxArea, area)
    }
    stack.push(i)
  }
  return maxArea
};
```

{% endnote %}

## Examples

### 实现一个简单的计算器

利用后缀表达和栈可以实现一个简单的计算器

#### 算式的后缀表达

我们平时使用的是中缀表达（Infix Notation），形式是 **操作数 运算符 操作数**，比如 1 + 1

后缀表达（Postfix Notation, Reverse Polish Notation）的形式是 **操作数 操作数 运算符**，比如 1 1 +

要得到后缀表达，只需要：

- 操作数顺序不变
- 操作符根据优先级加在操作数后面

比如：

```text
 10 + (44 - 1) * 3 + 9 / 2
 10       44       1       3       9       2
 10      (44       1 -)    3      (9       2 /)
 10     ((44       1 -)    3 *)   (9       2 /)
(10     ((44       1 -)    3 *) +)(9       2 /)
(10     ((44       1 -)    3 *) +)(9       2 /) +
 10 44 1 - 3 * + 9 2 / +
 
 1 - 2 - 5 + 6 / 5
 1 2 - 5 - 6 5 / +
 
 (22 / 7 + 4) * (6 - 2)
 22 7 / 4 + 6 2 - *
```

如何从中缀表达转换为后缀表达，可以看单调栈部分，以及后面的 Code

#### 计算器的例子

1. 先得到后缀表达

2. 遇到操作数入栈，遇到运算符则将将前两个出栈并进行运算
3. 将结果入栈

计算 3 - 2 - 1：

1. 先得到后缀表达 3 2 - 1 -

2. push 3
3. push 2
4. 遇到运算符 -
   1. pop 出 2
   2. pop 出 3
   3. 3 - 2 = 1
   4. push 1
5. push 1
6. 遇到运算符 -
   1. pop 出 1
   2. pop 出 1
   3. 1 - 1 = 0
   4. push 0
7. 结果是栈顶的 0

#### Code

```js
const priorityMap = new Map([
  ['(', 0],
  ['+', 1],
  ['-', 1],
  ['*', 2],
  ['/', 2],
  [')', 3],
])

// '1+2' -> ['1', '+', '2']
const splitString = (s) => {
  return s.split(/(\(|\)|\+|-|\*|\/)/).reduce((acc, cur) => {
    if (!cur.trim()) {
      return acc
    }
    return acc.concat(cur.trim())
  }, [])
}

// ['1', '+', '2'] -> [1, 2, '+']
const infix2Postfix = (infix) => {
  const postfix = []
  const stack = new Stack()
  // -1 -> 0-1
  if (infix[0] === '-') {
    infix.unshift('0')
  }
  infix.forEach((item) => {
    if (!priorityMap.has(item)) {
      // 数字直接加进结果
      postfix.push(Number(item))
    } else if (item === '(') {
      // 左括号入栈，待处理
      stack.push(item)
    } else if (item === ')') {
      // 遇到右括号，一直pop，直到对应的左括号
      // 相当于一段的完结，后面低优先级的运算符放在一段的最末位
      while (stack.top !== '(') {
        postfix.push(stack.pop())
      }
      // pop掉多余的左括号
      stack.pop()
    } else {
      // 遇到 + - * /，如果stack.top优先级大于等于当前位，就把stack.top给pop出来
      // 注意这里的等于，因为当优先级相同的时候，stack.top在前面出现，也应该pop出来
      while (stack.size && priorityMap.get(stack.top) >= priorityMap.get(item)) {
        postfix.push(stack.pop())
      }
      // 当前位入栈
      stack.push(item)
    }
  })
  // 整体的结束，与括号结束类似，剩下的低优先级的运算符放在末尾
  while (stack.size) {
    postfix.push(stack.pop())
  }
  return postfix
}

// [1, 2, '+'] -> 3
const calcPostfix = (postfix) => {
  const stack = new Stack()
  postfix.forEach((item) => {
    if (typeof item === 'number') {
      // 数字直接入栈
      stack.push(item)
    } else {
      // 遇到运算符，出栈两个数组，与运算符一起做运算
      const r = stack.pop()
      const l = stack.pop()
      if (item === '+') {
        stack.push(l + r)
      } else if (item === '-') {
        stack.push(l - r)
      } else if (item === '*') {
        stack.push(l * r)
      } else if (item === '/') {
        stack.push(l / r)
      }
    }
  })
  return stack.top
} 

const calc = (s) => {
  const splited = splitString(s)
  const postfix = infix2Postfix(splited)
  return calcPostfix(postfix)
}
```

### 用栈实现队列

[LeetCode.232](https://leetcode.cn/problems/implement-queue-using-stacks/)

思路一：用两个栈（Main 和 Helper），每当需要取队列第一个元素的时候，把 Main 的全部 Pop 出来，放到 Helper 里面，这样就是倒着的了。操作完成之后再倒回去。会有很多无用操作。

思路二：Main 倒进 Helper 之后可以不用再倒回去了，就放在里面。可以理解为一个为输入栈 In，另一个为输出栈 Out。当需要取第一个元素的时候，如果 Out 不为空，直接 Push，如果为空，则将 In 的全部倒入 Out，再 Push。

```typescript
class Queue {
  private inStack = new Stack()
  private outStack = new Stack()
  constructor() {}
  
  push(x: number): void {
    this.inStack.push(x)
  }
  
  pop(): number {
    if (this.outStack.isEmpty()) {
      while (!this.inStack.isEmpty()) {
        this.outStack.push(this.inStack.pop())
      }
    }
    return this.outStack.pop()
  }
  
  peek(): number {
    const res = this.pop()
    this.push(res)
    return res
  }
  
  isEmpty(): boolean {
    return this.inStack.isEmpty() && this.outStack.isEmpty()
  }
}
```

### 用队列实现栈

[LeetCode.225](https://leetcode.cn/problems/implement-stack-using-queues/)

思路一：用两个队列（Main 和 Helper），每次需要取最末尾的元素时，将 Main 的一直出队去 Helper 到最后一个之前，然后出队最后一个。然后再把 Helper 的弄回来。

思路二：用一个队列，要取最末尾的元素时，将前面的元素出队再入队，排到后面去，直到遇到最后一个元素。

```typescript
class Stack() {
  private q = new Queue()
  constructor() {}
  
  push(x: number): void {
    this.q.push(x)
  }

  pop(): number {
    let size = this.q.size()
    while (size > 1) {
      this.q.push(this.q.pop())
      size--
    }
    return this.q.pop()
  }
  
  top(): number {
    const x = this.pop()
    this.push(x)
    return x
  }

  isEmpty(): boolean {
    return this.q.isEmpty()
  }
}
```

# Queue

> 队列，先进先出（First In First Out, FIFO）

## Implementation of Queue

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

## Monotonic Queue

单调队列与单调栈有些许类似之处，他常用于求解区间内的的最大值或最小值。单调队列需要保证队列内部的元素单调递增或递减，除此之外，实际算法应用中由于经常会涉及到其他操作，因此大多数需要借助双端队列实现。

实际算法中的单调队列可以看成是变种的单调栈，比如维持一个单调递减的单调队列，我们需要：

- 如果当前需要插入的元素大于队尾元素，则将队尾 `pop` 出来，重复这个过程——这与 [单调栈](#monotonic-stack) 是一样的；
- 除此之外，他还可以通过出队来找到最大元素（队首元素）。

## Examples

### 滑动窗口最大值

[LeetCode.239](https://leetcode.cn/problems/sliding-window-maximum/)

# Linked List

> 链表

- 最后一个节点的 next 指针指向 null
- 通常有 head 指针表示第一个节点，有时候 tail 指针表示最后一个节点
- 双向链表的结点同时有 next 指针和 prev 指针

## Operations of Linked List

| 操作  | 时间复杂度 |
| ----- | ---------- |
| 访问  | O(1)       |
| 插入* | O(1)       |
| 删除* | O(1)       |

*注意如果插入和删除并不知道元素到底在哪里，需要从头开始查找的话，复杂度是 O(n)

# Hash Map

> 哈希表

考虑我们有一堆 Key-Value 值，比如 Key 是 id，Value 是姓名，怎么才能实现快速通过 id 获取到姓名呢？

首先如果 id 本来就是数字，那么可以用数组来实现，id 即为数组的 index，这样就可以通过数组的 index 直接拿到姓名了，**查找的时间复杂度是 O(1)**。

但是这样的话如果 id 分布比较分散，比如对于两个值 1 和 100，我们为了能将他们存起来，需要分配一个长度至少为 101 的数组，index 从 0 取值到 100，这样中间就有大量空间浪费掉了，空间复杂度是 **O(range(id))**。

**哈希函数是一个特定的编码函数， 可以将 Key 转换为一个索引值，这样就可以节约大量空间。**

比如定义一个简单的哈希函数 `H(k) = k % m`，那么就有 `H(293) = 3`，将这个 id 为 293 的项存在 index 为 3 的位置即可。

## Hash Collision

如果映射过去的索引空间太小，就可能将不同的值映射到同样的地方去，比如 `8 % 5 = 3 % 5 = 3`，因此 id 为 8 和 5 时都会被映射到索引为 3 的地方去，这被称为 **哈希碰撞**。

定义 **负载率（Load Factor）** $\lambda = n / m$，其中 $n$ 是 key 的总数， $m$ 是哈希表索引空间的大小，最好保持 $\lambda < 0.5$。

可以通过 **链表（Chaining）**或 **开放寻址（Open Addressing）**解决哈希碰撞问题：

### Chaining

> 将碰撞的那一位 `A[k]` 使用链表来进行存储。

查找的时间复杂度最坏 O(n)，平均 O(λ)（若 λ < 1，则为 O(1)）；因此为了保持良好的性能，需要保持 $\lambda < 1$。

### Open Addressing

> 如果 A[k] 已经被占据了，找另一个地方存。

例如有哈希函数 $H(k, i) = (k+i) \% m$，其中 $i$ 是尝试次数。

注意：

- 插入和查找不能无休止地进行下去，有最大尝试次数；
- 删除不能无休止地进行下去，可以将已经删除的元素标记位 DELETED，遇到 DELETED 的话继续删下去，而遇到什么都没有的空元素，说明删到头了，停止删除。

与链表不同，**必须** 保持 $\lambda \le 1$，否则新的元素就插不进去了。

哈希函数有两种策略：

- **线性探测（Linear Probing）**：$(k+i)\%m$，其性能在 $\lambda > 0.5$ 时显著下降；
- **次方探测（Quadratic Probing）**：$k \% m + a \cdot i + b \cdot i^2$。哈希表中的元素容易堆在一起形成集群，次方探测可以减少集群问题，从而减少碰撞，因而性能下降没有线性探测剧烈。

# Linked Hash Map

> **哈希链表（Linked Hash Map）**是将哈希表与链表组合起来使用的一种数据结构，在设计 LRU 缓存的时候有妙用。

## LRU

> [LeetCode.146 LRU Cache](https://leetcode.cn/problems/lru-cache/description)

**LRU（Least Recently Used）**是一种常见的缓存策略。由于缓存的容量是有限的，需要在缓存容量满了的时候清理掉上次使用时间距离现在最久的缓存，给新的数据腾出空间。

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

## LFU

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

# Binary Tree

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

## Definition

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

## Traversal

### Deep First Traversal

> 深度优先遍历分为前序、中序和后序遍历，均有递归和迭代写法，主要需要借助栈来实现

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/TreeDFS.png)

#### 递归写法

递归三步：

1. 确定参数和返回值
2. 确定终止条件
3. 确定每次需要做的事情

##### 前序（中左右）

```typescript
const traversal = (current: TreeNode | null, cb?: (node: TreeNode) => any) => {
  if (!current) {  return }
  cb?.(current)              // 中
  traversal(current.left)   // 左
  traversal(current.right)  // 右
}
```

##### 中序（左中右）

```typescript
const traversal = (current: TreeNode | null, cb?: (node: TreeNode) => any) => {
  if (!current) { return }
  traversal(current.left)   // 左
  cb?.(current)             // 中
  traversal(current.right)  // 右
}
```

##### 后序（左右中）

```typescript
const traversal = (current: TreeNode | null, cb?: (node: TreeNode) => any) => {
  if (!current) { return }
  traversal(current.left)   // 左
  traversal(current.right)  // 右
  cb?.(current)             // 中
}
```

#### 迭代写法

##### 前序（中左右）

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

##### 中序（左中右）

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

##### 后序（左右中）

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

#### 统一风格的迭代写法

使用标记法，在需要处理的元素入栈之后推入 `null`，这样栈里面就有两种元素：

1. 需要处理的元素，他后面会接一个 `null`
2. 需要进一步访问其子结点的元素，他后面不会接 `null`

##### 中序（左中右）

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

##### 前序（中左右）

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

##### 后序（左右中）

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

### Breadth First Traversal

> 广度优先遍历即层序遍历

#### 迭代法

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

#### 递归法

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

## Construct Binary Trees

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

## Other Operations

**[翻转二叉树](https://leetcode.cn/problems/invert-binary-tree/)**：#后序 #先序 #层序

**[对称二叉树](https://leetcode.cn/problems/symmetric-tree/)**：#内外层 #内外层共用一个数组层序

**[二叉树最大深度](https://leetcode.cn/problems/maximum-depth-of-binary-tree/)**：#最大深度=根节点高度 #后序求高度 #先序求深度 #层序用size控制每次只遍历一层

**[二叉树最小深度](https://leetcode.cn/problems/minimum-depth-of-binary-tree/)**：#最小深度=根节点到叶节点的距离 #后序求高度需要注意最低点的位置判断 #先序求深度 #层序

## Special Binary Trees

### Full Binary Tree

每个节点有 0 或 2 个子结点

### Perfect Binary Tree

> 满二叉树

只有度为 0 或 2 的节点，且度为 0 的节点在同一层上。

深度为 $k$ 时，有 $2^k - 1$ 个节点。

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/PerfectBinaryTree.png)

### Complete Binary Tree

> 完全二叉树，有的作者会将 Perfect Binary Tree 称为 Complete Binary Tree，而称这种树为 Nearly Complete Binary Tree 或 Almost Complete Binary Tree。

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/CompleteBinaryTree.png)

除了最底层，每层都填满了，且最底层节点都集中在左边的位置。

这种树可以很方便地用数组表示，若当前节点的下标为 `i`，则其父节点的下标为 `Math.floor((i - 1) / 2)`，左孩子下标为 `2 * i + 1`，右孩子下标为 `2 * i + 2`，最后一个非叶节点下标为 `Math.floor(size / 2) - 1`。

优先级队列可以用堆来实现，而堆则就是一棵完全二叉树，同时保证了父子节点的顺序关系。

#### Count

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

### Binary Search Tree

> 二叉搜索树

若左子树不为空，则左子树上所有节点的值均小于该根节点的值；若右子树不为空，则右子树上所有节点的值均大于该根节点的值。

**二叉搜索树的中序遍历得到的结果是有序的**，利用这个特性可以解决一些问题。

#### Search

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

#### Validation

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

#### Insertion

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

#### Deletion

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

### AVL Tree

> 平衡二叉树，又称为 Adleson-Velsky and Landis Tree

是空树；或者他的左右两个子树的高度差绝对值不超过 1，并且左右两棵子树都是一棵平衡二叉树。

#### isBalanced

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

#### Convert Sorted Array to Binary Search Tree

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

### Segment Tree

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

# Tree

除了二叉树以外，还有一些常用的树，比如 B Tree、B+ Tree 等。

## B Tree

B 树（B tree）是一种自平衡的树，能够保持数据有序。这种数据结构能够让查找数据、顺序访问、插入数据及删除的动作，都在对数时间内完成。

与普通的二叉树不同，B 树有一些额外的特点：

- B 树的每个节点可以拥有两个以上的子节点；
- B 树中拥有 $k$ 个子节点的非叶节点，拥有 $k - 1$ 个键，且他们升序排列，比如 $7, 16$；
- 每个键的左子树中的所有键都小于他，右子树中所有键都大于他；
- 所有叶节点都位于同一层。



![B Tree](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/b-tree-1.svg)

// TBC

## B+ Tree

B+ 树（B+ Tree）是 B 树的变种，与 B 树有以下不同：

- B 树上的每个节点可以都可以存键和数据，但是 B+ 树的内部节点（非叶结点、索引节点）只存索引，而数据都存在叶节点上；
- 每个键的左子树中的所有键都小于他，右子树中所有键都 **大于等于** 他；
- 每个叶节点都存有相邻叶节点的指针，叶节点本身按照键从小到大排列；
- 父结点存有右孩子的第一个元素的索引。

![B+ Tree](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/bplus-tree-1.png)

// TBC

# Heap

## Priority Queue & Heap

**优先级队列（Priority Queue）是一个抽象数据类型（Abstract Data Type, ADT）**，他支持插入元素（Enqueue）、删除具有最大或最小优先级的元素（Dequeue）。他可以用很多数据结构来实现，比如堆、二叉搜索树、数组等。

**堆（Heap）是一个具体的数据结构**，可以用来实现优先级队列，他实际上就是一棵 **完全二叉树**（最后一层可能不满），并且满足排序特点：大顶堆（Max-Heap）中父结点的优先级高于子结点，小顶堆（Min-Heap）中则相反。堆有很多变种，比如二叉堆、斐波那契堆、二项堆等，他们都有自己的特点。

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

## Heap Sort

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

## Examples

### 前 K 个高频元素

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

# Graph

## Topological Sorting

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

### BFS TopSort

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

### DFS TopSort

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



## Search

跟树类似，图有深度优先搜索（DFS）和广度优先搜索（BFS）两种搜索方式。深度优先搜索就是先沿着某一条路径探索到底，广度优先就是一层一层地先将当前节点可触达的节点处理完成。

### Deep First Search

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

### Breadth First Search

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

## Shortest Path

### Dijkstra

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

### Floyd

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

# Disjoint-Set Data Structure

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

# Backtracing

**回溯算法** 的本质是 **DFS**，而 **回溯操作** 实际上指的是：

- 我们在遍历二叉树的时候，可能会定义一些全局变量来记录当前所在的节点位置、或者遍历到当前节点的时候累计的某些值
- **在我们采用深度优先遍历时，这些变量需要在遍历 B 子树之前，退回到原来的状态，消除遍历 A 子树时产生的影响**

由于回溯算法的本质是遍历操作，因此效率并不高，通常需要进行剪枝来些微提高效率，一般来说只有在解决以下这些比较复杂的问题时，才会考虑回溯法：

- **组合问题：**`n` 个数里面找出 `k` 个数的集合
- **排列问题：**`n` 个数按照一定规则全排列
- **切割问题：**一个字符串按照一定规则进行切割
- **子集问题：**给定一个集合，找到所有可能的子集
- **棋盘问题：**N皇后、解数独

回溯法其实就是对树的遍历，不过对象通常不是二叉树，因此需要两种循环：

- **横向遍历：**`for` 循环
- **纵向遍历：**递归

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

## 组合问题

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

### 组合问题中的去重

[LeetCode.40 Combination Sum II](https://leetcode.cn/problems/combination-sum-ii/description/) 中就要求对组合进行去重，组合问题中去重的特点是：

1. **候选元素集合中有值相同的元素**
2. **候选元素集合中每个元素只能使用一次**，但由于候选集合中有相同的元素，因此结果集合中可能有值相同的元素
3. **不能有相同的结果集合**

由于候选集合中可能有值相同的元素，如果不进行去重处理，**两个结果集可能分别使用了候选集合中位置不同、但值相同的元素**，导致两个结果集相同了。

从第 2 点可以看出来，由于并没有限制单个结果集合内不能出现重复的元素，因此比如在候选集 `[1, 1, 2]` 中，可以出现集合 `[1, 1]`，但不能出现两个 `[1, 2]`。该类去重是 **对树的横向去重（在 `for` 循环中去重）**，而纵向可以重复。

对树的横向去重，实际上就是对 **当层的可选元素数组** 的去重（即对模板代码中的 `selections` 去重），数组去重常用的就是 `Set` 和 **排序后相邻元素去重**。

#### 使用 Set 去重

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

#### 相邻元素去重

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

## 切割问题

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

## 子集问题

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

## 排列问题

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

### 排列问题中的去重

[LeetCode.47 Permutations II](https://leetcode.cn/problems/permutations-ii/description/) 中要求像组合问题一样，对排列问题进行去重。因为该题中出现了相同值的元素，需要进行 **横向去重**，而 LeetCode.46 中每个元素的值都是不同的，不需要进行横向去重。

#### 使用两个 Set 去重

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

#### 使用 used 数组进行去重

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

## 棋盘问题

TBC

# Greedy Algorithm

贪心算法：想要解决一个大问题，那么在解决大问题的路上的每一步都采取当前看起来最优的解法。贪心算法有这样几个特点：

- 不关心当前的选择对后续造成的影响
- 也不是有规律地从之前的某几步中推出当前步的结果（与动态规划不同，动态规划会从之前的步骤推算出当前的结果）
- 其中的某些步骤在整体看来可能并不是最优的，但是他们在当时看来是最优的，并且由这样小的最优的累计，最终能达成整体最优
- 贪心策略通常更偏向脑筋急转弯，更加隐晦和靠直觉，不如动态规划那样清晰
- 不是每个问题都能用贪心算法解决

## Examples

### 最大子数组和

[LeetCode.53 Maximum Subarray](https://leetcode.cn/problems/maximum-subarray/description/)，给定一个数组，找出最大的子数组（里面的项在原来的数组里面必须是连续的）和。

以数组 `[-2,1,-3,4,-1,2,1,-5,4]` 为例尝试从 `index` 为 `0` 时的最小问题开始解决：

1. 第 0 项为 `-2`，那么必选 `-2`，因此在选择第 0 项的时候，`sum` 为 `-2`
2. 第 1 项为 `1`，如果我们选了之前的 `-2`，会导致结果更小，因此我们不选 `-2`，只选 `1`，`sum` 为 `1`
3. 第 2 项为 `-3`，我们肯定会加上之前的 `1`，`sum` 为 `-2`
4. 第 3 项为 `4`，我们必然不能加上之前的结果，`sum` 为 `4`
5. 第 4 项为 `-1`，加上之前的结果 `4`，`sum` 为 `3`
6. 第 5 项为 `2`，虽然前一项为 `-1`，但是如果不选 `-1`，就代表放弃了我们累计的 `sum`，因此还是得选上，`sum` 为 `5`
7. 第 6 项为 `1`，`sum` 为 `6`
8. ...

我们只需要持续追踪到上一项为止的 **累计的和**，如果和为正数，则考虑当前位的时候，把之前的加上，否则就不能加。

这个 **累计的和** 已经是 **考虑选择上一项之后的最大的和**，因此只有两种可能，加上这个和、或者不加，其他的情况只会更糟（我们不需要回顾这个和是从哪里算起的，只知道他是考虑上一项之后最大的和即可）。

{% note Code %}

```typescript
function maxSubArray(nums: number[]): number {
  let maxSum = -Infinity
  let sum = 0
  for (let i = 0; i < nums.length; i++) {
    if (sum < 0) {
      sum = nums[i] 
    } else {
      sum += nums[i]
    }
    maxSum = Math.max(sum, maxSum)
  }
  return maxSum
}
```

{% endnote %}

### 加油站

[LeetCode.134 Gas Station](https://leetcode.cn/problems/gas-station/)，环形道路上 `gas` 数组和 `cost` 数组分别代表该加油站可以加多少油、和去下一个加油站需要耗费多少油，找到从哪里开始。

这一题其实跟最大子数组和有一点类似，我们依然是可以拿示例来进行讨论：

```typescript
gas = [1,2,3,4,5]
cost = [3,4,5,1,2]
```

1. 在 0 时，净余额为 `1 - 3 = -2`
2. 在 1 时，肯定不能从 0 出发，因此净余额为 `2 - 4 = -2`
3. 在 2 时，肯定也不能加上前面的，因此净余额为 `3 - 5 = -2`
4. 在 3 时，不能加上前面的，净余额为 `4 - 1 = 3`
5. 在 4 时，可以加上前面的，净余额为 `3 + 5 - 2 = 6`

因此我们可以在净余额不为负数的时候，就标记 **可以从该点** 出发：

- 如果净余额在累加的过程中为负数了，则只能从下一个不为负数的开始
- 如果到终点净余额都还是非负的，且 `gas` 总量大于等于 `cost` 总量，说明绕圈的话，剩下的油也是够跑完起始点之前的点的。否则就返回 `-1`

考虑，为什么不可以是起始点之后的某一点为起始点呢？因为从起始点开始是更优的选择，因为起始点到他之后的每一点都是有盈余的，肯定比从他之后的某一点开始强。如果算到某一点累加值为负了，那么就需要更换起始点了。

{% note Code %}

```typescript
function canCompleteCircuit(gas: number[], cost: number[]): number {
  let rest = 0
  let start = -1
  let total = 0
  for (let i = 0; i < gas.length; i++) {
    const current = gas[i] - cost[i]
    total += current
    if (rest <= 0) {
      start = i
      rest = current
    } else {
      rest += current
    }
  }
  return total >= 0 ? start : -1
}
```

{% endnote %}

### 分发糖果

[LeetCode.135 Candy](https://leetcode.cn/problems/candy/description/)，有一列孩子，每个孩子至少一颗糖，得分高的孩子要比旁边的人糖果多。

本题实际上用了两个贪心策略：

1. 如果当前孩子比前面的孩子分高，那么他的糖果就是前面的孩子糖果数 + 1
2. 如果从左往右遍历一次，然后再从右往左遍历一次，那么每个小孩就跟左右都比较过了；注意在第二次遍历时，需要跟第一次的结果进行比对，取最大值而不要破坏第一次的计算结果

{% note Code %}

```typescript
function candy(ratings: number[]): number {
  const compareLeft = []
  let candies = 0
  for (let i = 0; i < ratings.length; i++) {
    if (i > 0 && ratings[i] > ratings[i - 1]) {
      compareLeft[i] = compareLeft[i - 1] + 1
    } else {
      compareLeft[i] = 1
    }
  }
  const comapreRight = []
  for (let i = ratings.length - 1; i >= 0; i--) {
    if (i < ratings.length - 1 && ratings[i] > ratings[i + 1]) {
      comapreRight[i] = Math.max(comapreRight[i + 1] + 1, compareLeft[i])
    } else {
      comapreRight[i] = compareLeft[i]
    }
    candies += comapreRight[i]
  }
  return candies
};
```

{% endnote %}

### 重叠区间

**重叠区间其实就是一种遍历方法，与之前二叉树的遍历别无二致。**遍历过程中要做的事情有三件：

1. 对区间进行排序
2. 如果该区间的开始仍在上一次的重叠区间内，需要做什么
3. 如果该区间的开始已经脱离了上一次的重叠区间，需要做什么

需要注意的是，**在重叠区间内** 在不同的题目可能含义不太相同，**有的题目是只要重叠区间内的最早结束的区间结束就结束了，有的题目是重叠区间内最远的区间结束才结束。**

[LeetCode.452 用最少数量的箭引爆气球](https://leetcode.cn/problems/minimum-number-of-arrows-to-burst-balloons/) 就是一道很典型的求重叠区间的问题，按照每个区间的开始从小到大对区间进行排序，然后进行一轮遍历，如果遍历到重叠区间结束，则射出一箭。

当重叠区间结束时，我们射出一箭，那么这个重叠区间的所有气球就都被引爆了，也就是说，所有之后区间内的所有小区间都没用了；因此我们不需要用优先队列来记住区间内的所有小区间，只需要记录该区间内最先结束的小区间的结束点就可以了。

{% note Code %}

```typescript
function findMinArrowShots(points: number[][]): number {
  if (!points.length) { return 0 }
  points.sort((a, b) => a[0] - b[0])
  let shots = 1
  for (let i = 1; i < points.length; i++) {
    if (points[i][0] > points[i - 1][1]) {
      // 重叠区间结束后射一箭
      shots++
    } else {
      // 记录该重叠区间内最先结束的小区间的结束点，其实可以用一个新的变量来记录，但是这里复用了 points 数组
      points[i][1] = Math.min(points[i - 1][1], points[i][1])
    }
  }
  return shots
};
```

{% endnote %}

[LeetCode.435 无重叠区间](https://leetcode.cn/problems/non-overlapping-intervals/description/) 要求我们移除一些区间，使得没有区间重叠，跟上一题很类似，不过上一题是在重叠区间结束的时候射一箭，这一题是在重叠区间内，每有一个区间重叠，就计数一次（该区间需要被移除）。

{% note Code %}

```typescript
function eraseOverlapIntervals(intervals: number[][]): number {
  intervals.sort((a, b) => a[0] - b[0])
  let end = intervals[0][1]
  let overlaps = 0
  for (let i = 1; i < intervals.length; i++) {
    if (intervals[i][0] >= end) {
      end = intervals[i][1]
    } else {
      end = Math.min(end, intervals[i][1])
      // 该区间需要被移除，计数 + 1
      overlaps++
    }
  }
  return overlaps
};
```

{% endnote %}

[LeetCode.56 合并区间](https://leetcode.cn/problems/merge-intervals/) 是在重叠区间的遍历过程中，不断地更新之前生成的合并后区间，在重叠区间结束时创建新的区间。

{% note Code %}

```typescript
function merge(intervals: number[][]): number[][] {
  const result = []
  intervals.sort((a, b) => a[0] - b[0])
  for (let i = 0; i < intervals.length; i++) {
    if (!result.length) {
      result.push(intervals[i])
      continue
    }
    const last = result[result.length - 1]
    if (intervals[i][0] > last[1]) {
      result.push(intervals[i])
    } else {
      last[1] = Math.max(last[1], intervals[i][1])
    }
  }
  return result
};
```

{% endnote %}

[LeetCode.763 划分字母区间](https://leetcode.cn/problems/partition-labels/) 有点类似青蛙跳问题，不断更新当前区间的最大值。当当前遍历的项已经超过最大值时，在蛙跳问题中青蛙将不能跳到终点，而本题中则是新建一个区间再继续跳。

{% note Code %}

```typescript
function partitionLabels(s: string): number[] {
  const map = {}
  for (let i = 0; i < s.length; i++) {
    map[s[i]] = i
  }
  const results = []
  let start = 0
  let end = 0
  for (let i = 0; i < s.length; i++) {
    if (i > end) {
      results.push(end - start + 1)
      start = i
      end = map[s[i]]
      continue
    }
    end = Math.max(end, map[s[i]])
  }
  results.push(end - start + 1)
  return results
};
```

{% endnote %}

# Dynamic Programing

动态规划比较通用的解法是 **找规律**，先根据题意确定递推数组的含义，然后根据我们定义的含义从 0 开始找规律，可以参考 Examples 中的题目。

一些比较典型的动态规划问题，比如 **背包问题**，就被人们总结起来，形成了一类特殊的、有解题模板的题目。

此外，比较复杂的动态规划问题有下面几种，如果常规的找规律想不出来，可以考虑以下几个方向：

1. **多状态的动态规划。**递推数组的每一项并不是只存了一个数字，而是多个状态，前面的状态会影响后面的状态，比如买卖股票问题中上一天持有股票的状态将影响当前天的计算。
2. **状态压缩的动态规划。**多状态动态规划的升级版，使用二进制来保存当前状态，通过二进制运算来改变状态，状态数量在 32 种以下的复杂题目一般可以往这个方向考虑。
3. **高维动态规划。**递推数组有时候会达到二维甚至三维，其实基础版本的背包问题就是二维动态规划，不过可以通过滚动数组的方式将其降维，有些复杂的背包问题就可能会涉及到更三维动态规划了。
4. **非线性动态规划。**有些复杂的动态规划可能涉及到非线型数据结构，而不只是局限于数组，比如树、图等。

**动态规划有以下的思考步骤：**

1. 确定 `dp` 数组下标及含义
2. 确定递推公式
3. 初始化 `dp` 数组
4. 确定遍历顺序
5. 举例推导 `dp` 数组

## Memorization DFS

**动态规划本质上就是特殊优化的一种深度优先搜索。**

我们拿 [LeetCode.63 不同路径](https://leetcode.cn/problems/unique-paths/) 来举例说明，在这道题中，机器人要从左上角走到右下角，每次只能向右或者向下移动一步，求有多少种不同的路径。

我们比较容易想到的 DFS 可以这样写：

{% note Code %}

```typescript
function uniquePaths(m: number, n: number): number {
  function dfs(position: number[]) {
    const [x, y] = position
    if (x >= m || y >= n) { return 0 }
    if (x === m - 1 && y === n - 1) { return 1 }
    return dfs([x + 1, y]) + dfs([x, y + 1])
  }
  return dfs([0, 0])
};
```

{% endnote %}

注意递归逻辑 `dfs([x + 1, y]) + dfs([x, y + 1])`，我把这种写法称为 **“头尾头”** 的写法，类比成树的话，就是从树的根节点 **向下“递”到每个叶节点**，然后将结果从叶节点后序地 **向上“归”回根节点**。

这样写实际上是将所有节点都遍历了一遍，有的节点甚至不止一遍。但是实际上我们 **每个节点只有一个最优解**，并且该最优解在“归”的过程中会一次性地计算出来，**不会有反复更新某个节点上的值的情况**。因此，我们这里可以用一个 Map 将已经在“归”的过程中计算过的值存起来，下次在“递”的时候如果发现之前已经计算过了，就会直接取已经存起来的值，避免对其子节点继续进行递归的过程。**这就被称为记忆化深度优先搜索（Memorization DFS）**，代码如下：

{% note Code %}

```typescript
function uniquePaths(m: number, n: number): number {
  const memo = {}
  function dfs(position: number[]) {
    const [x, y] = position
    if (x >= m || y >= n) { return 0 }
    if (x === m - 1 && y === n - 1) { return 1 }
    const key = getKey(position)
    if (memo[key] !== undefined) { return memo[key] }
    const paths = dfs([x + 1, y]) + dfs([x, y + 1])
    return memo[key] = paths
  }
  return dfs([0, 0])
};

function getKey(position: number[]) {
  return position.toString()
}
```

{% endnote %}

> 这与之前在 **回溯的组合问题** 中提到的 DFS 有点不同。**组合问题中，当每个节点有很多种合法的可能性**，每次先暂时性地确定该节点的值，然后再向下 DFS 到终点之后，**会撤销之前的路径并给节点重新确定一个值**。所以当时我们提到，那种 DFS 与动态规划不同。

除了这种 **“头尾头”** 的写法以外，我们还有一种 **“尾头尾”** 的写法（从尾“递”到头，再从头“归”回尾），只不过是将递归逻辑反过来写成 `dfs([x - 1, y]) + dfs([x, y - 1])`，然后将起始和终止条件也随之调整一下即可：

{% note Code %}

```typescript
function uniquePaths(m: number, n: number): number {
  const memo = {}
  function dfs(position: number[]) {
    const [x, y] = position
    if (x < || y < 0) { return 0 }
    if (x === 0 && y === 0) { return 1 }
    const key = getKey(position)
    if (memo[key] !== undefined) { return memo[key] }
    const paths = dfs([x - 1, y]) + dfs([x, y - 1])
    return memo[key] = paths
  }
  return dfs([m - 1, n - 1])
};

function getKey(position: number[]) {
  return position.toString()
}
```

{% endnote %}

这两种写法其实都可以转换为动态规划，只是 `dp` 数组的含义和遍历顺序不同而已，动态规划中的 `dp` 的遍历顺序实际上就是 DFS 中 **“归”的顺序**，因为 **DFS 中结果的计算与收集实际上是在“归”这一步完成的**（可以看出动态规划实际上就是这类 DFS 的迭代写法）：

|                   | 头尾头                                     | 尾头尾                                     |
| ----------------- | ------------------------------------------ | ------------------------------------------ |
| 递归逻辑          | `return dfs([x + 1, y]) + dfs([x, y + 1])` | `return dfs([x - 1, y]) + dfs([x, y - 1])` |
| `dp[i][j]` 的含义 | 从 `[i, j]` 出发到达终点有多少种路径       | 从起点出发到达 `[i, j]` 有多少种路径       |
| 递推公式          | `dp[i][j] = dp[i + 1][j] + dp[i][j + 1]`   | `dp[i][j] = dp[i - 1][j] + dp[i][j - 1]`   |
| `dp` 遍历顺序     | `终点 -> 起点`                             | `起点 -> 终点`                             |

我们将后者转换为动态规划的写法：

{% note Code %}

```typescript
function uniquePaths(m: number, n: number): number {
  const dp = [new Array(n).fill(1)]
  for (let i = 1; i < m; i++) {
    dp[i] = [1]
    for (let j = 1; j < n; j++) {
      dp[i][j] = dp[i - 1][j] + dp[i][j - 1]
    }
  }
  return dp[m - 1][n - 1]
};
```

{% endnote %}

可以从递推公式中看出，我们只依赖了 **上一行** 的数据，并没有依赖 **再上一行**，因此我们可以使 **用滚动数组的方式来优化空间复杂度**，只使用一维的 `dp` 数组，每次在遍历新的一行时，数组中原来的数据就是上一行的数据了：

{% note Code %}

```typescript
function uniquePaths(m: number, n: number): number {
  const dp = new Array(n).fill(1)
  for (let i = 1; i < m; i++) {
    for (let j = 1; j < n; j++) {
      dp[j] = dp[j - 1] + dp[j]
    }
  }
  return dp[n - 1]
};
```

{% endnote %}

## 找规律

[LeetCode.343 整数拆分](https://leetcode.cn/problems/integer-break/) 就是一道找规律的题目，从第 0 项开始慢慢找规律即可，通过反复利用已经拆过的结果可以。

{% note Code %}

```typescript
function integerBreak(n: number): number {
// 1 => 1 * 0
// 2 => 1 * 1, 2 * 0 => 1
// 3 => 2 * 1, 1 * 1 * 1 => 2
// 4 => 3 * 1, 2 * 2, 2 * 1 * 1 => 3
// 5 => 4 * 1(max4/3), 3 * 2 => 6
// 6 => 5 * 1(max5/6), 4 * 2(max4/3), 3 * 3(max3/2) => 9
// 7 => 6 * 1(max6/9), 5 * 2(max5/6), 4 * 3 => 12
// 8 => 7 * 1(max7/12), 6 * 2(max6/9), 5 * 3(max5/6), 4 * 4 => 18
// 9 => 8 * 1(max8/18), 7 * 2(max7/12), 6 * 3(max6/9), 5 * 4(max5,6) => 27
// 10 => 9 * 1(max9/27), 8 * 2(max8/18), 7 * 3(max7/12), 6 * 4 => 36
  const dp = new Array(n + 1).fill(0)
  dp[1] = 0
  dp[2] = 1
  for (let i = 3; i <= n; i++) {
    for (let j = 0; j <= Math.floor(i / 2); j++) {
      dp[i] = Math.max(
        dp[i],
        Math.max(dp[j], j) * Math.max(dp[i - j], i - j)
      )
    }
  }
  return dp[n]
};
```

{% endnote %}

## 背包问题

![背包问题分类](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Algorithm-Bag-1.png)

背包问题变种繁多，其核心是 **遍历的方式和顺序**，`dp` 数组表示的内容可能会随着题目的要求而发生很多变化，但总结起来他的遍历方式和顺序总结起来只有以下几种：

- **遍历方式有**：外层物品 `i`，内层背包容量 `j`；外层背包容量 `j`，内层物品 `i`。一般来讲区别不大，我比较喜欢物品 `i` 在外，容量 `j` 在内。只有在以下这两种情况下才有会有不同：
  - **用滚动数组进行优化时**，由于滚动数组相当于默认使用了逐行遍历，每一行需要依赖上一行的内容，因此需**要外层物品 `i`、内层容量 `j`**。
  - **多重背包求装满方式时**，由于多重背包一个物品可以被多次使用，因此如果 **先遍历容量 `j`，再遍历物品 `i`** 的话，相当于求的是 **排列数**，因为在遍历每个容量时，物品会从头开始考虑；而 **先物品 `i` 再容量 `j`** 的话，求的就是 **组合数**。
- **遍历顺序有**：背包容量 `j` 正序还是倒序。在外层物品 `i` 内层容量 `j` 的情况下，一般区别不大，但 **如果使用了滚动数组**：
  - **正序遍历 `j`** 会使得物品被反复装入（因为反复利用了前面的数据），即 **完全背包**
  - **倒序遍历 `j`** 则不会存在反复利用本行前面数据的这样的问题，即 **01背包**

### 01背包

有 n 件物品和一个最多能背重量为 w 的背包。第 i 件物品的重量是 weight[i]，得到的价值是 value[i] 。**每件物品只能用一次**，求解将哪些物品装入背包里物品价值总和最大。

假设物品如下：

|        | 重量 | 价值 |
| ------ | ---- | ---- |
| 物品 0 | 1    | 15   |
| 物品 1 | 3    | 20   |
| 物品 2 | 4    | 30   |

#### 暴力解法

每个物品有两个状态，选或不选，可以用回溯法暴力求解，比如树的每一层就对应着一个物品，左节点是加了的情况，右节点是不加的情况。

时间复杂度是 O(2^n)。

#### 二位数组动态规划

##### 1. 确定 dp 数组及下标的含义

`dp[i][j]` 表示从物品 `0 ~ i` 中任意取，放进容量为 `j` 的背包，其总价值是多少。

![dp[i][j]的含义](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Algorithm-Bag-2.png)

##### 2. 确定递推公式

- 不放物品 `i`，那么 `dp[i][j]` 就等于 `dp[i-1][j]`

- 放物品 `i`，背包容量为 `j-weight[i]` 时的价值为 `dp[i-1][j-weight[i]]`，在此基础上再加上 `value[i]` 即可
  - 注意 `j - weight[i]` 可能为负，这种情况背包容量为 j 时，只装物品 i 也装不下，`dp[i][j] = dp[i-1][j]`


```text
dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - weight[i]] + value[i])
```

##### 3. dp 数组的初始化

- 背包容量 j 为 0 时，总价值为 0：`dp[i][0] = 0`
- 在放第 0 个物品（i 为 0）时：
  - 如果放不下（`j < weight[0]`），则 `dp[0][j] = 0`
  - 如果能放下（`j >= weight[0]`），则 `dp[0][j] = value[0]`

```javascript
// 放不下的情况
if (let j = 0; j < weight[0]; j++) {
  dp[0][j] = 0
}
// 能放下的情况
if (let j = weight[0]; j < bagweight; j++) {
  dp[0][j] = value[0]
}
```

其他的值初始化为什么都不影响（因为会被覆盖），我们可以先生成一个全是 0 的二维数组，然后初始化 `value[0]` （能放下物品 0 的情况）

##### 4. 确定遍历的顺序

根据递推公式，先遍历 `i` 还是 `j` 都不影响

##### 最终代码

{% note Code %}

```typescript
const weightBagProblem = (
  weight: number[], value: number[], size: number,
): number => {
  const goodsNum = weight.length
  const dp = new Array(goodsNum)
    .fill(0)
    .map((item) => new Array(size + 1).fill(0))
  // j = 0 的时候全部为 0
  // i = 0 的时候能放下时为 value[0]，否则为 0
  for (let j = weight[0]; j <= size; j++) {
    dp[0][j] = value[0]
  }
  for (let i = 1; i < goodsNum; i++) {
    for (let j = 1; j <= size; j++) {
      if (j < weight[i]) {
        dp[i][j] = dp[i - 1][j]
      } else {
        dp[i][j] = Math.max(dp[i - 1][j], dp[i - 1][j - weight[i]] + value[i])
      }
    }
  }
  return dp[goodsNum - 1][size]
}
```

{% endnote %}

#### 一维数组动态规划（滚动数组）

在二维数组中，递推公式如下：

```text
dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - weight[i]] + value[i])
```

如果每次把第 `i - 1` 行复制到第 `i` 行，那么公式变为这样：

```text
dp[i][j] = max(dp[i][j], dp[i][j - weight[i]] + value[i])
```

 看出来都是同一行的数据，可以考虑用一维数组来解这个问题。

##### 1. 确定 dp 数组及下标的含义

`dp[j]` 表示容量为 `j` 的背包的最大价值。

##### 2. 确定递推公式

```text
dp[j] = max(dp[j], dp[j - weight[i]] + value[i])
```

也是两种情况，要么放 `i`，要么不放 `i`：

- 不放 `i`：`dp[j] = dp[j]`
- 放 `i`：`dp[j] = dp[j - weight[i]] + value[i]`

相当于就是上面二维数组的公式中把 `[i]` 去掉了。

##### 3. dp 数组的初始化

- 背包容量为 0 时，`dp[0] = 0`
- 根据递推公式，剩下的可以随意初始化，如果题目给的价值都是正整数，那么其他项初始化为 `0` 就可以了

##### 4. 确定遍历的顺序

注意：

- 根据递推公式，**必须 `i` 在外层，`j` 在内层**（相当于二维数组的逐行遍历），因为 `dp[i - 1][j - weight[i]]` 相当于要依赖上一行前几位的运算结果
- 遍历 `j` 的时候，**一定要倒序遍历**，因为顺序的话物品可能会被加入多次：

```javascript
weight[0] = 1; value[0] = 15;
dp[1] = dp[1 - weight[0]] + value[0] = 15
dp[2] = dp[2 - weight[0]] + value[0] = 30 // value[0] 又加入了一次
```

这个加入多次的问题在二维数组中不存在，因为：

- 二维数组第 0 行是通过初始化给的，我们遍历是从 1 开始的
- 之后每一行都是借助 **上一行** 的数据算出来的（看递推公式），所以顺序遍历本行并不会有影响

一维数组中，相当于 **把上一行的数据原位重复利用**，如果顺序遍历的话，相当于后面的数据用了 **本行** 的数据，就可能会重复放入物品

##### 最终代码

{% note Code %}

```typescript
const weightBagProblem = (
  weight: number[], value: number[], size: number,
) => {
  const goodsNum = weight.length
  const dp = new Array(size + 1).fill(0)
  for (let i = 0; i < goodsNum; i++) {
    // 这里必须从大遍历到小
    for (let j = size; j >= weight[i]; j--) {
      dp[j] = Math.max(dp[j], dp[j - weight[i]] + value[i])
    }
  }
  return dp[size]
}
```

{% endnote %}

### 完全背包

有 n 件物品和一个最多能背重量为 w 的背包。第 i 件物品的重量是 weight[i]，其价值是 value[i] 。**每件物品都有无限个（也就是可以放入背包多次）**，求解将哪些物品装入背包里物品价值总和最大。

假设物品如下：

|        | 重量 | 价值 |
| ------ | ---- | ---- |
| 物品 0 | 1    | 15   |
| 物品 1 | 3    | 20   |
| 物品 2 | 4    | 30   |

#### 遍历方式

完全背包最核心的就是遍历方式和顺序问题，在完全背包中需要 **从小到大进行遍历**，因为一样物品可以被添加多次：

```js
for (let i = 0; i < goodsNum; i++) {
  for (let j = weight[i]; j <= size; j++) {
    dp[j] = Math.max(dp[j], dp[j - weight[i]] + value[i])
  }
}
```

##### 求最大价值时的遍历方式

另外之前 0 1 背包中，要求物品在外层，重量在内层，因为下一行需要 `dp[i - 1][j - weight[i]]`，会依赖上一行前几位的运算结果。

而 **在完全背包中，是可以重量在外层，物品在内层的**。因为 `dp[j - weight[i]]`，依赖的是本行的运算结果，只有 `dp[0]` 是继承的上一行的数据，如果先遍历列，也可以拿到上一行 `dp[i - 1][0]`（因为是同一列的） 。

如果是问最大价值，**dp 数组里面存的是最大价值，最大价值与怎样凑成最大价值无关，因此物品、重量谁在内层谁在外层并不关键**。

```js
// 先遍历背包容量（列），再遍历物品（行）
for (let j = 0; j <= size; j++) {
  for (let i = 0; i < weight.length; i++) {
    if (j - weight[i] >= 0) {
      dp[j] = Math.max(dp[j], dp[j - weight[i]] + value[i])
    }
  }
}
```

##### 求装满方式时的遍历方式

但如果问背包的装满方式，遍历方式就会影响很大了，得看题目问的是 **组合问题** 还是 **排列问题**。

拿零钱兑换问题（[LeetCode.518 Coin Change II](https://leetcode.cn/problems/coin-change-ii/)）举例，给定一个总额和一堆无限多的币值，问总额有多少种组合方式。组合问题不关心顺序。

1. 确定 dp 数组及下标的含义：`dp[j]` 表示凑成总额为 `j` 时，有多少种钱币组合方式
2. 确定递推公式：`dp[j] += dp[j - coins[i]]`
3. 初始化：`dp[0] = 1`，否则后面推导出来的都是 0（但是这其实并没有什么真正的含义）。其他的初始化为 `0`
4. 确定遍历顺序

如果外层钱币（物品），内层总额（重量）：

```js
for (let i = 0; i <= coins.length; i++) {    // 物品
  for (let i = coins[i]; j <= amount; j++) { // 重量
    dp[j] += dp[j - coins[i]]
  }
}
```

假设 `coins = [1, 5]`，那么这种情况就是先把 1 带进去看，再把 5 带进去看，得到的方法数只会有 {1, 5}，而不会有 {5, 1}，因此 `dp[j]` 里面存的是组合数。

如果交换遍历顺序：

```js
for (let j = 0; j <= amount; j++) {        // 重量
  for (let i = 0; i < coins.length; i++) { // 物品
    if (j - coins[i] >= 0) {
      dp[j] += dp[j - coins[i]]
    }
  }
}
```

背包容量的每一个值，都会重新计算一遍 1 和 5，所以包含了 {1, 5} 和 {5, 1} 两种情况，`dp[j]` 里面存的就是排列数。

### Examples

背包问题的难点在于得看出来题目是背包问题，并且找到题目中的什么可以被抽象为 **物品** 和 **背包容量**。

上文在多重背包中提到过 **“求最大价值”** 和 **“求装满方式”** 这两类典型的背包问题，这两类问题遍历方式和遍历顺序上可能有一些差别，不过其中最大的区别的还是 **`dp` 数组的定义不同**，下面我们来看经过包装过的应用题是怎么简化成这两类背包问题的。

#### 是否能恰好装满类问题

[LeetCode. 416](分割等和子集) 就是一道 **是否能恰好装满** 的问题，他要求我们把一个数组分成两份，其中每份的和恰好等于数组所有元素总和的一半。

这相当于要求 **选择其中的一些元素，使他们的和恰好等于某个值**。那么显然可以理解成：**选择一堆物品，让他们恰好装满背包。**

由于问题问的是 **是否** 而不是 **有多少种方式**，因此用 **“求最大价值”** 和 **“求装满方式”** 都可以解决这类问题。

先看 **“求最大价值”** 的解法：元素的值就是他的价值，元素的值同时也是他的重量，我们在装背包的时候，会尽可能让这个背包的价值更高，背包装满的时候也就是价值最高的时候。

# Difference Array

**差分数组（Difference Array）**是原数组中相邻元素两两作差（右边减左边）得到的。准确来说，对于数组 `a`，定义其差分数组 `d` 为：

```javascript
d[i] = i === 0 ? a[0] : a[i] - a[i - 1]
```

差分数组有两个性质，以 `a = [1,3,3,5,8]` 和 `d = [1,2,0,2,3]` 来举例说明：

**性质 1：从左到右累加 `d` 中的元素，可以得到数组 `a`。**这是显然的。

**性质 2：区间操作与单点操作可以相互转换：**

- **区间操作：**把 `a` 中的子数组 `a[i]` 到 `a[j]` 都加上 `x`；
- **单点操作：**把 `d[i]` 增加 `x`、`d[j + 1]` 减少 `x`。如果 `j + 1 == n`，则只需要把 `d[i]` 增加 `x`。

对性质 2 进行举例说明，假设我们现在要将 `a[1]` 到 `a[3]` 都增加 `10`：

- 直接对 `a` 进行区间操作得到的结果是 `[1,13,13,15,8]`；
- 对差分数组 `d` 进行单点操作得到的结果是 `[1,12,0,2,-7]`，将其按照性质 1还原，得到 `[1,13,13,15,8]`。

再举一个例子，如果我们要将 `a[0]` 到 `a[3]` 都增加 `10`，那么单点操作的结果是 `[11,2,0,2,-7]`，可以还原为 `[11,13,13,15,8]`。

最后举一个例子，如果我们将 `a[1]` 到 `a[4]` 都增加 `10`，那么单点操作的结果是 `[1,12,0,2,3]`，可以还原为 `[1,13,13,15,18]`。

下面是利用差分数组在将区间操作 O(m*n) 的时间复杂度优化为 O(n) 的代码模板：

```typescript
function rangeOperations(a, operations) {
  // 构建差分数组
  const diff = new Array(a.length)
  for (let i = 0; i < a.length; i++) {
    diff[i] = i === 0 ? a[i] : a[i] - a[i - 1]
  }
  // 对差分数组进行单点操作
  for (const [left, right, x] of operations) {
    diff[left] += x
    if (right + 1 < n) {
      diff[right + 1] -= x
    }
  }
  // 还原数组
  for (let i = 0; i < n; i++) {
    diff[i] += diff[i - 1]
  }
  return diff
}
```

# Mod

最简单的模运算：计算乘积的个位数，比如 $1234\cdot6789$ 的个位数是 $(4\cdot9)\%10=6$ ，$1234+6789$ 的个位数是 $(4+9)\%10=3$。

将上述结论抽象成数学公式就是：
$$
(a+b) \% m = ((a \% m) + (b \% m)) \% m
$$

$$
(a \cdot b) \% m = ((a \% m) \cdot (b \% m)) \% m
$$

## Mod 1e9 + 7

由于有的语言对大数并没有进行特殊处理，所以无法进行高精度的大数运算，有的题目便会要求将结果对 1e9 + 7 取模，这样就不用考虑大数运算的问题了。

但是，**不能只是单纯地对结果取模**，因为这样就没有意义了，并不能避免大数运算带来的问题。

此外，还有几个注意点：

1. 取模之后就不能用 Max 比较最大值了，因此有的题就不能用动态规划了（但是仍然可以使用 Sum）

2. 注意是否真的取模了：
   ```js
   // 用 += 的时候注意
   for(int i = 1; i <= n ; i ++) {
       res += i % (1e9 + 7); // 实际上并没有对 res 取模，只是取模了 i
   }
   
   // 由于 * 和 % 是从左到右运算的，因此需要加上括号
   res = res % mod + pre[i][j] % mod * a[i][j] % mod;
   ```

