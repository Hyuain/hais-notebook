---
title: Greedy Algorithm
date: 2022-02-05 15:49:12
categories:
  - - 计算机
mathjax: true
tags:
  - CX
---

# Greedy Algorithm

贪心算法：想要解决一个大问题，那么在解决大问题的路上的每一步都采取当前看起来最优的解法。贪心算法有这样几个特点：

- 不关心当前的选择对后续造成的影响
- 也不是有规律地从之前的某几步中推出当前步的结果（与动态规划不同，动态规划会从之前的步骤推算出当前的结果）
- 其中的某些步骤在整体看来可能并不是最优的，但是他们在当时看来是最优的，并且由这样小的最优的累计，最终能达成整体最优
- 贪心策略通常更偏向脑筋急转弯，更加隐晦和靠直觉，不如动态规划那样清晰
- 不是每个问题都能用贪心算法解决

# Examples

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

**重叠区间其实就是一种遍历方法，与之前二叉树的遍历别无二致。** 遍历过程中要做的事情有三件：

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
