---
title: 数据结构与算法
date: 2022-02-05 15:49:12
categories:
  - [计算机]
---

.

<!-- more -->

# 数组

## 查找

### 二分查找

[LeetCode.204](https://leetcode.cn/problems/binary-search/submissions/)

前提

- 数组是有序的
- 数组中无重复元素

时间复杂度 O(log(n))

```typescript
const binarySearch = (nums: number[], target: number) => {
  if (!nums.length) { return -1 }
  let left = 0
  let right = nums.length - 1
  while (l <= r) {
    let mid = Math.floor((left + right) / 2)
    if (nums[mid] === target) {
      return mid
    } else if (nums[mid] > target) {
      // 如果写成这样，意味着新的 right 需要在下一次循环被考虑
      // 所以当 left === right 的时候，循环还不能终止
      right = mid - 1
    } else {
      // 新的 left 总是需要被考虑的，所以下面这行一般没有第二种写法
      left = mid + 1
    }
  }
  return -1
}
```

## 排序

### 选择排序

每次从剩下的部分选择最小的排到前面来，O(n^2)

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
    if (i !== index) {
      let temp = numbers[index]
      numbers[index] = numbers[i]
      numbers[i] = temp
    }
  }  
}
```

### 快速排序

找出 Pivot，把小于 Pivot 的放左边，大于 Pivot 的放右边，最好 O(nlogn)，最坏 O(n^2)

```js
const quickSort = numbers => {
  if (numbers.length < 2) return numbers
  let pivotIndex = Math.floor(numbers.length / 2)
  let pivot = numbers.splice(pivotIndex, 1)[0]
  let left = []
  let right = []
  for (let i = 0; i < numbers.length; i++) {
    if (numbers[i] <= pivot) {
      left.push(numbers[i])
    } else {
      right.push(numbers[i])
    }
  }
  return quickSort(left).concat([pivot], quickSort(right))
}
```

### 冒泡排序

两两比较，如果后者比前者小，则交换，最终每次循环使得最大数冒泡到最后去（O(n^2)，最好 O(n)）

```js
const bubbleSort = numbers => {
  let length = numbers.length
  while (length > 1) {
    let swapped = false
    for (let i = 0; i < numbers.length - 1; i++) {
      if (numbers[i + 1] < numbers[i]) {
        let temp = numbers[i + 1]
        numbers[i + 1] = numbers[i]
        numbers[i] = temp
        swapped = true
      }
    }
    if (!swapped) break
    length--
  }
}
```

## 数组去重

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

## 例题

### 电话号码的组合（公式运算）

[LeetCode.17](https://leetcode.cn/problems/letter-combinations-of-a-phone-number/)

```js
const telComb = (str) => {
  const map = ['', 1, 'abc', 'def', 'ghi', 'jkl', 'mno', 'pqrs', 'tuv', 'wxyz']
  const num = str.split('')
  // 保存键盘映射后的字母内容，比如 23 => ['abc', 'def']
  const code = []
  num.forEach(item => {
    if (map[item]) {
      code.push(map[item])
    }
  })
  const combine = (arr) => {
    const result = []
    for (let i = 0; i < arr[0].length; i++) {
      for (let j = 0; j < arr[1].length; j++) {
        result.push(`${arr[0][i]}${arr[1][j]}`)
      }
    }
    // 递归的精髓，用临时结果 result 替换掉前面两项
    arr.splice(0, 2, result)
    if (arr.length > 1) {
      combine(arr)
    } else {
      return result
    }
    return arr[0]
  }
  return combine(code)
}
```

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

# 字符串

## 例题

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

# 栈

## 例题

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

#### 计算器的例子

1. 先得到后缀表达

2. 遇到操作数入栈，遇到运算符则将将前两个出栈并进行运算
3. 将结果入栈

计算 3 - 2 - 1：

1. 先得到后缀表达 3 2 - 1

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

# 队列

## 队列的实现

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



# 二叉树

## 定义

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

## 遍历

### 深度优先

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

### 广度优先

> 即层序遍历

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

# 动态规划

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
