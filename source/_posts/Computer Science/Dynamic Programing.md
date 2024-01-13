---
title: Dynamic Programing
date: 2022-02-05 15:49:12
categories:
  - - 计算机
mathjax: true
tags:
  - CX
---


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

# Memorization DFS

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

# 找规律

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

# 背包问题

![背包问题分类](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Algorithm-Bag-1.png)

背包问题变种繁多，其核心是 **遍历的方式和顺序**，`dp` 数组表示的内容可能会随着题目的要求而发生很多变化，但总结起来他的遍历方式和顺序总结起来只有以下几种：

- **遍历方式有**：外层物品 `i`，内层背包容量 `j`；外层背包容量 `j`，内层物品 `i`。一般来讲区别不大，我比较喜欢物品 `i` 在外，容量 `j` 在内。只有在以下这两种情况下才有会有不同：
  - **用滚动数组进行优化时**，由于滚动数组相当于默认使用了逐行遍历，每一行需要依赖上一行的内容，因此需**要外层物品 `i`、内层容量 `j`**。
  - **多重背包求装满方式时**，由于多重背包一个物品可以被多次使用，因此如果 **先遍历容量 `j`，再遍历物品 `i`** 的话，相当于求的是 **排列数**，因为在遍历每个容量时，物品会从头开始考虑；而 **先物品 `i` 再容量 `j`** 的话，求的就是 **组合数**。
- **遍历顺序有**：背包容量 `j` 正序还是倒序。在外层物品 `i` 内层容量 `j` 的情况下，一般区别不大，但 **如果使用了滚动数组**：
  - **正序遍历 `j`** 会使得物品被反复装入（因为反复利用了前面的数据），即 **完全背包**
  - **倒序遍历 `j`** 则不会存在反复利用本行前面数据的这样的问题，即 **01背包**

## 01背包

有 n 件物品和一个最多能背重量为 w 的背包。第 i 件物品的重量是 weight[i]，得到的价值是 value[i] 。**每件物品只能用一次**，求解将哪些物品装入背包里物品价值总和最大。

假设物品如下：

|        | 重量 | 价值 |
| ------ | ---- | ---- |
| 物品 0 | 1    | 15   |
| 物品 1 | 3    | 20   |
| 物品 2 | 4    | 30   |

### 暴力解法

每个物品有两个状态，选或不选，可以用回溯法暴力求解，比如树的每一层就对应着一个物品，左节点是加了的情况，右节点是不加的情况。

时间复杂度是 O(2^n)。

### 二位数组动态规划

#### 1. 确定 dp 数组及下标的含义

`dp[i][j]` 表示从物品 `0 ~ i` 中任意取，放进容量为 `j` 的背包，其总价值是多少。

![`dp[i][j]`的含义](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Algorithm-Bag-2.png)

#### 2. 确定递推公式

- 不放物品 `i`，那么 `dp[i][j]` 就等于 `dp[i-1][j]`

- 放物品 `i`，背包容量为 `j-weight[i]` 时的价值为 `dp[i-1][j-weight[i]]`，在此基础上再加上 `value[i]` 即可
  - 注意 `j - weight[i]` 可能为负，这种情况背包容量为 j 时，只装物品 i 也装不下，`dp[i][j] = dp[i-1][j]`


```text
dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - weight[i]] + value[i])
```

#### 3. dp 数组的初始化

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

#### 4. 确定遍历的顺序

根据递推公式，先遍历 `i` 还是 `j` 都不影响

#### 最终代码

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

### 一维数组动态规划（滚动数组）

在二维数组中，递推公式如下：

```text
dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - weight[i]] + value[i])
```

如果每次把第 `i - 1` 行复制到第 `i` 行，那么公式变为这样：

```text
dp[i][j] = max(dp[i][j], dp[i][j - weight[i]] + value[i])
```

 看出来都是同一行的数据，可以考虑用一维数组来解这个问题。

#### 1. 确定 dp 数组及下标的含义

`dp[j]` 表示容量为 `j` 的背包的最大价值。

#### 2. 确定递推公式

```text
dp[j] = max(dp[j], dp[j - weight[i]] + value[i])
```

也是两种情况，要么放 `i`，要么不放 `i`：

- 不放 `i`：`dp[j] = dp[j]`
- 放 `i`：`dp[j] = dp[j - weight[i]] + value[i]`

相当于就是上面二维数组的公式中把 `[i]` 去掉了。

#### 3. dp 数组的初始化

- 背包容量为 0 时，`dp[0] = 0`
- 根据递推公式，剩下的可以随意初始化，如果题目给的价值都是正整数，那么其他项初始化为 `0` 就可以了

#### 4. 确定遍历的顺序

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

#### 最终代码

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

## 完全背包

有 n 件物品和一个最多能背重量为 w 的背包。第 i 件物品的重量是 weight[i]，其价值是 value[i] 。**每件物品都有无限个（也就是可以放入背包多次）**，求解将哪些物品装入背包里物品价值总和最大。

假设物品如下：

|        | 重量 | 价值 |
| ------ | ---- | ---- |
| 物品 0 | 1    | 15   |
| 物品 1 | 3    | 20   |
| 物品 2 | 4    | 30   |

### 遍历方式

完全背包最核心的就是遍历方式和顺序问题，在完全背包中需要 **从小到大进行遍历**，因为一样物品可以被添加多次：

```js
for (let i = 0; i < goodsNum; i++) {
  for (let j = weight[i]; j <= size; j++) {
    dp[j] = Math.max(dp[j], dp[j - weight[i]] + value[i])
  }
}
```

#### 求最大价值时的遍历方式

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

#### 求装满方式时的遍历方式

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

## Examples

背包问题的难点在于得看出来题目是背包问题，并且找到题目中的什么可以被抽象为 **物品** 和 **背包容量**。

上文在多重背包中提到过 **“求最大价值”** 和 **“求装满方式”** 这两类典型的背包问题，这两类问题遍历方式和遍历顺序上可能有一些差别，不过其中最大的区别的还是 **`dp` 数组的定义不同**，下面我们来看经过包装过的应用题是怎么简化成这两类背包问题的。

### 是否能恰好装满类问题

[LeetCode. 416 分割等和子集](https://leetcode.cn/problems/partition-equal-subset-sum/) 就是一道 **是否能恰好装满** 的问题，他要求我们把一个数组分成两份，其中每份的和恰好等于数组所有元素总和的一半。

这相当于要求 **选择其中的一些元素，使他们的和恰好等于某个值**。那么显然可以理解成：**选择一堆物品，让他们恰好装满背包。**

由于问题问的是 **是否** 而不是 **有多少种方式**，因此用 **“求最大价值”** 和 **“求装满方式”** 都可以解决这类问题。

先看 **“求最大价值”** 的解法：元素的值就是他的价值，元素的值同时也是他的重量，我们在装背包的时候，会尽可能让这个背包的价值更高，背包装满的时候也就是价值最高的时候。
