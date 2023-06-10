---
title: Data Structure and Algorithm
date: 2022-02-05 15:49:12
categories:
  - [计算机]
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
- 算法（动态规划）
  - 动态规划：背包问题
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

<detail>
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
</detail>

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

<details>
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
</details>
### 卡牌分组 （归类运算）

[LeetCode.914](https://leetcode.cn/problems/x-of-a-kind-in-a-deck-of-cards/)

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

# String

## Big Number Calculation

### Big Number Plus

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

### Big Number Minus

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

## Monotonic Stack

一维数组，要寻找任一元素的右边或左边第一个比自己大或自己小的位置。

比如 `[73, 74, 75, 71, 71, 72, 76, 73]` 为例，输出下一个 **大于** 自己的位置，输出应该是 `[1, 2, 6, 5, 5, 1, -1, -1]`。

这个时候需要维持一个 **栈底到栈顶单调递减** 的单调栈。这里我们单调栈里面可以存放元素的下标（存放什么可以根据题目的需要定义，重要的是单调栈的核心思想，这个看下面模拟会有标注）。

遍历一遍数组，每次将遍历到的元素与栈顶元素相比较，有两种操作：

- **如果当前遍历的元素大于栈顶的元素**，说明栈顶元素找到了下一位比自己大的元素，栈顶出栈并记录结果；同时新的栈顶也要与当前遍历的元素比较，直到小于等于当前元素
- **如果当前遍历的元素小于等于栈顶元素**，说明还需要继续找，将其入栈（这样效果上就达成了一个栈底到栈顶递减的单调栈）

模拟步骤如下：

```text
const stack = []
const result = [-1, -1, -1, -1, -1, -1, -1, -1]

--i == 0--
!stack.size
stack.push(0)
result: [-1, -1, -1, -1, -1, -1, -1, -1]
stack: [0]

--i == 1--
74 > stack.top(73), 需要在 73 的位置标记已经找到了 74；73 的下一个大的已经找到，可以出栈
  result[stack.pop()] = i, result[0] = 1
stack.push(1)
result: [1, -1, -1, -1, -1, -1, -1, -1]
stack: [1]

--i == 2--
75 > stack.top(74), 需要在 74 的位置标记已经找到了 75；74 的下一个大的已经找到，可以出栈
  result[stack.pop()] = i, result[1] = 2
stack.push(2)
result: [1, 2, -1, -1, -1, -1, -1, -1]
stack: [2]

--i == 3--
71 < stack.top(75), 将 71 入栈，待查找
stack.push(3)
result: [1, 2, -1, -1, -1, -1, -1, -1]
stack: [2, 3]

--i == 4--
71 == stack.top(71), 将 71 入栈，待查找
stack.push(4)
result: [1, 2, -1, -1, -1, -1, -1, -1]
stack: [2, 3, 4]

--i == 5--
72 > stack.top(71), 需要在 71 的位置标记已经找到了 72；71 的下一个大的已经找到，可以出栈
  result[stack.pop()] = i, result[4] = 5
  出栈后 72 仍然大于 stack.top(71)，继续出栈，直到小于等于 stack.top 或栈为空
  result[3] = 5
stack.push(5)
result: [1, 2, -1, 5, 5, -1, -1, -1]
stack: [2, 5]

--i == 6--
76 > stack.top(72), 需要在 72 的位置标记已经找到了 76；72 的下一个大的已经找到，可以出栈
  result[stack.pop()] = i, result[5] = 6
  出栈后 76 仍然大于 stack.top(75)，继续出栈，直到小于等于 stack.top 或栈为空
  result[2] = 6
stack.push(6)
result: [1, 2, 6, 5, 5, 6, -1, -1]
stack: [6]

--i == 7--
73 < stack.top(76), 将 73 入栈，待查找
stack.push(7)
result: [1, 2, 6, 5, 5, 6, -1, -1]
stack: [6, 7]

-- End --
```

计算器中将中缀表达转为后缀表达其实也类似于单调栈的思想，有三种方式可以解读：

- 遍历中缀表达式，遇到运算符的时候，如果栈顶的运算符优先级更高或相同，将其出栈（保证其先被运算），直到遇到优先级更低的运算符或栈空时，将当前运算符入栈
- 栈维持了从 **栈底到栈顶运算符优先级递增**
- 转为后缀表达式其实是要找 **下一个优先级相同或更低的运算符的位置**，因为 **下一个优先级相同或更低的运算符的位置** 就是 **当前运算符覆盖范围截止的位置**

```text
1 + 2 * 3 - 4  => 1 2 3 * + 4 -

const stack = []
const result = []

--i == 0--
数字 1 直接放入结果
result: [1]

--i == 1--
!stack.size:
  stack.push('+')
stack: ['+']
result: [1]

--i == 2--
数字 2 直接放入结果
stack: ['+']
result: [1, 2]

--i == 3--
* 比 stack.top('+') 优先级更高，stack.push('*')
stack: ['+', '*']
result: [1, 2]

--i == 4--
数字 3 直接放入结果
stack: ['+', '*']
result: [1, 2, 3]

--i == 5--
- 比 stack.top('*') 优先级低，result.push(stack.pop())，说明 * 负责的范围已经结束了（2 和 3）
- 跟 stack.top('+') 优先级相同，result.push(stack.pop())，说明 + 负责的范围已经结束了（1 和 2 * 3）
stack.push('-')
stack: ['-']
result: [1, 2, '*', '+']

--i == 6--
数字 4 直接放入结果
stack: ['-']
result: [1, 2, '*', '+', 4]

-- End --
清空 stack，按照出栈顺序加入结果
result: [1, 2, '*', '+', 4, '-']
```

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

# Examples

## 滑动窗口最大值

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

# Hash Table

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
  if (!current) {	return }
  cb?.(current)							// 中
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
  cb?.(current)							// 中
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
    cb?.(current)								// 处理
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
      cb?.(current)							 // 处理
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
    if (!current) {					   	// 出栈时遇到 null，说明下一个元素需要处理
      cb?.(stack.pop())
      continue
    }
    if (current.right) { stack.push(current.right) }	// 右入栈，待访问
    stack.push(current, null)													// 中入栈，待处理
    if (current.left) { stack.push(current.left) }    // 左入栈，待访问
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

## Special Binary Trees

### Full Binary Tree

每个节点有 0 或 2 个子结点

### Perfect Binary Tree

> 满二叉树

只有度为 0 或 2 的节点，且度为 0 的节点在同一层上。

深度为 $k$ 时，有 $2^k - 1$ 个节点。

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/PerfectBinaryTree.png)

### Complete Binary Tree

> 完全二叉树，有的作者会将 Perfect Binary Tree 称为 Complete Binary Tree，而称这种树为 Nearly Complete Binary Tree 或 Almost Binary Tree。

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/CompleteBinaryTree.png)

除了最底层，每层都填满了，且最底层节点都集中在左边的位置。

这种树可以很方便地用数组表示，若当前节点的下标为 `i`，则其父节点的下标为 `Math.floor((i - 1) / 2)`，左孩子下标为 `2 * i + 1`，右孩子下标为 `2 * i + 2`，最后一个非叶节点下标为 `Math.floor(size / 2) - 1`。

优先级队列可以用堆来实现，而堆则就是一棵完全二叉树，同时保证了父子节点的顺序关系。

### Binary Search Tree

> 二叉搜索树

若左子树不为空，则左子树上所有节点的值均小于该根节点的值；若右子树不为空，则右子树上所有节点的值均大于该根节点的值。

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

#### Convert Sorted Array to Binary Search Tree

[LeetCode.108](https://leetcode.cn/problems/convert-sorted-array-to-binary-search-tree/)

Given an integer array nums where the elements are sorted in ascending order, convert it to a height-balanced binary search tree.

<detail>
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
</detail>

# Heap

## Priority Queue & Heap

**优先级队列（Priority Queue）是一个抽象数据类型（Abstract Data Type, ADT）**，他支持插入元素（Enqueue）、删除具有最大或最小优先级的元素（Dequeue）。他可以用很多数据结构来实现，比如堆、二叉搜索树、数组等。

**堆（Heap）是一个具体的数据结构**，可以用来实现优先级队列，他实际上就是一棵 **完全二叉树**（最后一层可能不满），并且满足排序特点：大顶堆（Max-Heap）中父结点的优先级高于子结点，小顶堆（Min-Heap）中则相反。堆有很多变种，比如二叉堆、斐波那契堆、二项堆等，他们都有自己的特点。

JavaScript 中并没有提供原生的堆函数，而完全二叉树可以高效地用数组表示，因此我们可以手动用数组实现一个堆：

<detail>

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

</detail>

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
\le n\sum_{k=0}^\infin\frac{k}{2^k} \\
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

<detail>
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

</detail>

# Graph

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
  
}
```



# Dynamic Programing

## 背包问题

![背包问题分类](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Algorithm-Bag-1.png)

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

根据递推公式，先遍历 i 还是 j 都不影响

##### 最终代码

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

- 根据递推公式，必须 `i` 在外层，`j` 在内层（相当于二维数组的逐行遍历），因为 `dp[i - 1][j - weight[i]]` 相当于要依赖上一行前几位的运算结果
- 遍历 `j` 的时候，一定要倒序遍历，因为顺序的话物品可能会被加入多次：

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

### 完全背包

有 n 件物品和一个最多能背重量为 w 的背包。第 i 件物品的重量是 weight[i]，其价值是 value[i] 。**每件物品都有无限个（也就是可以放入背包多次）**，求解将哪些物品装入背包里物品价值总和最大。

假设物品如下：

|        | 重量 | 价值 |
| ------ | ---- | ---- |
| 物品 0 | 1    | 15   |
| 物品 1 | 3    | 20   |
| 物品 2 | 4    | 30   |

#### 遍历顺序问题

完全背包最核心的就是遍历顺序问题，在完全背包中需要从小到大进行遍历，因为一样物品可以被添加多次：

```js
  for (let i = 0; i < goodsNum; i++) {
    for (let j = weight[i]; j <= size; j++) {
      dp[j] = Math.max(dp[j], dp[j - weight[i]] + value[i])
    }
  }
```

##### 求最大价值时的遍历顺序

另外之前 0 1 背包中，要求物品在外层，重量在内层，因为下一行需要 `dp[i - 1][j - weight[i]]`，会依赖上一行前几位的运算结果

而在完全背包中，是可以重量在外层，物品在内层的。因为 `dp[j - weight[i]]`，依赖的是本行的运算结果，只有 `dp[0]` 是继承的上一行的数据，如果先遍历列，也可以拿到上一行 `dp[i - 1][0]`（因为是同一列的） 

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

##### 求装满方式时的遍历顺序

如果是问最大价值，dp 数组里面存的是最大价值，最大价值与怎样凑成最大价值无关，因此物品、重量谁在内层谁在外层并不关键。

但如果问背包的装满方式，遍历顺序就会影响很大了，得看题目问的是组合问题还是排列问题。

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

背包容量的每一个值，都会重新计算一遍 1 和 5，所以包含了 {1, 5} 和 {5, 1} 两种情况。，`dp[j]` 里面存的就是排列数。

## 例题

### 整数拆分

[LeetCode.343](https://leetcode.cn/problems/integer-break/)

Given an integer n, break it into the sum of k positive integers, where k >= 2, and maximize the product of those integers.

Return the maximum product you can get.

<detail>
#双重循环
```typescript
function integerBreak(n: number): number {
  const dp = new Array(n)
  // dp[i] = max(dp[i],max(dp[i-j]*j,(i-j)*j))
  // 1, 2, 4, 6, 9
  dp[2] = 1
  for (let i = 3; i <= n; i++) {
    for (let j = 1; j < i - 1; j++) {
      dp[i] = Math.max(dp[i] || 0, Math.max(dp[i - j] * j, (i - j) * j))
    }
  }
  console.log(dp)
  return dp[n]
};
```
</detail>

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

