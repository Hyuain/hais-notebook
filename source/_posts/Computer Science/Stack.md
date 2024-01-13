---
title: Stack
date: 2022-02-05 15:49:12
categories:
  - - 计算机
mathjax: true
tags:
  - CX
---


> 栈，后进先出（Last In First Out, LIFO）

# Implementation of Stack

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

# Monotonic Stack

**何时使用单调栈：**一维数组，要寻找任一元素的右边或左边第一个比自己大或自己小的位置。

**单调栈的核心思想在于 “清算”**，当遍历的 **当前元素 `x`** 满足某种条件之后，可以对之前保存在栈内的部分元素进行 **清算**。清算完成之后，就再也不用考虑那些元素了，所以通常来讲，使用单调栈时，元素遍历的过程中会至多入栈一次、出栈一次，时间复杂度为 O(n)。

## Examples

### 找到下一个大于自己的元素

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

### 将中缀表达转换为后缀表达

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

### 接雨水

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

### 柱状图中的最大矩形

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

# Examples

## 实现一个简单的计算器

利用后缀表达和栈可以实现一个简单的计算器

### 算式的后缀表达

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

### 计算器的例子

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

### Code

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

## 用栈实现队列

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

## 用队列实现栈

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
