---
title: LeetCodeDiray
date: 2023-02-05 15:49:12
categories:
  - [计算机]
mathjax: true
---

与 {% post_link Algorithm %} 不同，算法与数据结构一文中主要是从理论、宏观、模板的角度体系化地记录常见的算法与数据结构。而本文是做算法题中的学习记录，主要记录主观的心路历程和一些实际做题中的坑点、想法和解法。

本文将按照一定的 Tag，比如二分查找、双指针等来进行展开。

<!-- more -->

# 二分查找

## LeetCode 704. BinarySearch

> [LeetCode 704. 二分查找](https://leetcode.cn/problems/binary-search/)
>
> Last Reviewed: 2023.6.8

该题主要需要注意熟悉 **左闭右开** 和 **左闭右闭** 的两种写法，具体写法见 {% post_link Algorithm %} 对应章节。

## LeetCode 35. SearchInsertPosition

> [LeetCode 35. 搜索插入位置](https://leetcode.cn/problems/search-insert-position/)
>
> Last Reviewed: 2023.6.8

该题与二分查找别无二致，返回值既可以是右指针 `r`，也可以是左指针 `l`，比如我们如果使用左闭右闭区间 `[l, r]` 的写法：

```javascript
while (l <= r) {
  if (num[mid] === target) {
  	return mid
  } else if (nums[mid] < target) {
    l = mid + 1
  } else {
    r = mid - 1
  }
}
return l
```

可以看到三种情况下返回值均可以为 `l`：

- 当 `target < nums[0]`，我们要将 `target` 插入第 `0` 位，此时 `l` 也将永远为 `0`。
- 当 `target` 在 `nums` 范围内时，比如 `[0, 2]` 中插入 `1`，应该返回 `1`，也就是大于 `1` 的第一个数字的 `index`；由于 `l` 最后加了一，所以正好是这一位。
- 当 `target > nums[nums.length - 1]` 时，我们要将 `target` 第 `nums.length` 位，同上，也正好可以用 `l` 来表示。

## LeetCode 34. FindFirstAndLastPositionOfElementInSortedArray

> [LeetCode 34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/)
>
> Last Reviewed: 2023.6.8

该题刚拿到的时候毫无头绪，想着可能是二分查找嵌套。看了题解之后其实也是有点懵，题解将整体拆成三步：

1. 查找区间的右边界（约定右边界也不包含 target）
2. 查找区间的左边界（约定左边界不包含 target）
3. 根据左右边界确定最终 target 的 position
   1. target 不在 nums 的范围内：表现为该区间的右边界不在 nums 的范围内（target 位于 nums 的左侧），或该区间的左边界不在 nums 的范围内（target 位于 nums 的右侧）。
   2. target 在 nums 的范围内，但 nums 中不存在 target：表现为左右区间中间的元素数量小于 1。
   3. target 在 nums 的范围内，且 nums 中存在 target：输出结果即可。

## LeetCode 69. SqrtX

> [LeetCode 69. X 的平方根](https://leetcode.cn/problems/sqrtx/)
>
> Last Reviewed: 2023.6.8

因为要找到的结果是整数，因此直接在整数范围用二分查找即可。

## LeetCode 367. ValidPerfectSquare

> [LeetCode 367. 有效的完全平方数](https://leetcode.cn/problems/valid-perfect-square/)
>
> Last Reviewed: 2023.6.8

与 69 题相同。

# 双指针

## LeetCode 27. RemoveElement

> [LeetCode 27. 移除元素](https://leetcode.cn/problems/remove-element/)
>
> Last Reviewed: 2023.6.8

一眼快慢指针。

## LeetCode 283. MoveZeros

> [LeetCode 283. 移动零](https://leetcode.cn/problems/move-zeroes/)
>
> Last Reviewd: 2023.6.8

快慢指针，需要想清楚慢指针的定义，我的解法是慢指针指向最后一个非 0 的后一位。

## LeetCode 844. BackspaceStringCompare

> [LeetCode 844. 比较含退格的字符串](https://leetcode.cn/problems/backspace-string-compare/)
>
> Last Reviewed: 2023.6.8

刚拿到这道题的时候只能想到用栈来模拟退格的过程，创建两个新的结果字符串来进行比较，想不到怎样将使用双指针来解。

题解确实巧妙，将字符串反向遍历，将 # 看做是 Skip 而不是 Back。

## LeetCode 977. SquaresOfASortedArray

> [LeetCode 977. 有序数组的平方](https://leetcode.cn/problems/squares-of-a-sorted-array/)
>
> Last Reviewed: 2023.6.8

刚拿到这道题的时候，想的是找到第一个非负数，然后用两个指针向两头移动，逐个按平方大小的顺序推入结果数组。

题解也确实巧妙，不是从中间向两边发散，而是从两头向中间走，相当于反向生成结果，与 844 有异曲同工之妙。

## LeetCode 209. MinimunSizeSubarraySum

> [LeetCode 209. 长度最小的子数组](https://leetcode.cn/problems/minimum-size-subarray-sum/)
>
> Last Reviewed: 2023.6.8

这道题也是刚拿到的时候毫无头绪，结果发现是题目看得有问题，他要求的子数组是连续的而不是随便选，这样自然就可以用双指针（滑动窗口）来做了。

# 哈希表
## LeetCode 209. MinimunSizeSubarraySum

> [LeetCode 209. 长度最小的子数组](https://leetcode.cn/problems/minimum-size-subarray-sum/)
>
> Last Reviewed: 2023.6.8

这道题也是刚拿到的时候毫无头绪，结果发现是题目看得有问题，他要求的子数组是连续的而不是随便选，这样自然就可以用双指针（滑动窗口）来做了。

## LeetCode 242. Valid Anagram

> [LeetCode 242. 有效的字母异位词](https://leetcode.cn/problems/valid-anagram/)
>
> Last Reviewed: 2023.6.12

该题非常简单，可以考虑直接用dic实现或者直接用数组实现。

## LeetCode 349. Intersection of Two Arrays

> [LeetCode 349. 两个数组的交集](https://leetcode.cn/problems/intersection-of-two-arrays/)
>
> Last Reviewed: 2023.6.12

Set中元素不重复，能很好的解决这个问题。

## LeetCode 202. Happy Number

> [LeetCode 202. 快乐数](https://leetcode.cn/problems/happy-number/)
>
> Last Reviewed: 2023.6.12

不是快乐数的话会进入循环，即后面出现的数是之前出现过的，可以根据这个判断是否为快乐数。

## LeetCode 1. Two Sum

> [LeetCode 1. 两数之和](https://leetcode.cn/problems/two-sum/)
>
> Last Reviewed: 2023.6.12

一边遍历一边把数和index加入dic，遍历数i时只需要考虑dic中是否存在target-i即可

## LeetCode 59. SpiralMatrixII

> [LeetCode 59. 螺旋矩阵](https://leetcode.cn/problems/spiral-matrix-ii/)
>
> Last Reviewed: 2023.6.8

该题题解是可以以一圈为一个大循环，在每个大循环内确定该次大循环的四个小循环的边界，然后填充矩阵。

我自己考虑的解法是使用一个 `next` 变量维护下一次应该朝哪个方向走，比如 `right` `down` 等，然后走到边界之后再换向，模拟人思考的过程。

## LeetCode 19. RemoveNthNodeFromEndOfList

> [LeetCode 19. 删除链表的倒数第 N 个节点](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/)
>
> Last Rveiewed: 2023.6.10

该题首先想到的就是先遍历一遍找到 size，然后再找到正数的 index，该方法需要遍历两次链表。

题解使用快慢指针可以达仅用一次遍历的效果。具体做法是先将快指针指向 n + 1 的位置，然后快慢指针同时向后遍历，此时若快指针指向 null，则慢指针则会指向要删除的节点的父结点，利用这个快慢指针的差值记录 n，可谓非常巧妙。

## Interview 2.7. IntersectionOfTwoLinkedListLCCI

> [Interview 2.7. 链表相交]([Intersection of Two Linked Lists LCCI](https://leetcode.cn/problems/intersection-of-two-linked-lists-lcci/))
>
> Last Reviewed: 2023.6.10

该题首先想到是用 Set 存起来已经遍历的节点，该解法要求额外 O(n) 的空间。

同样的，该题也可以用快慢指针来解决。首先找到两个链表的长度差值 diff，然后通过将快指针（用来遍历长链表）先走 diff 步，使得两个链表右对齐。此时再让快（遍历长链表）慢（遍历短链表）指针同时出发，就可以找到相交的节点了。利用了如果两个链表相交，那么他们相交之后的部分长度相同的特点。

## LeetCode 142. LinkedListCycleII

> [LeetCode 142. 环形链表 II](https://leetcode.cn/problems/linked-list-cycle-ii/)
>
> Last Reviewd: 2023.6.10

该题其实也可以通过 Set 来解，用 Set 的话就无比简单，但需要额外 O(n) 的空间。

如果用快慢指针的话，该题需要让快指针的一次走两步，慢指针一次走一步，慢指针若能追上快指针，则链表有环。

找到环开始的地方就相对比较麻烦了，可以设三段的长度，环开始节点的左边长度为 $x$，环开始的地方到追上的地方长度为 $y$，环剩下的部分长度为 z。那么快指针走过的距离为 $x + k * (y + z)$；也可以表示为慢指针的两倍，即 $2 * (x + y)$，那么有以下等式：
$$
x + k * (y + z) +y = 2 * (x + y)
$$
可以化简为：
$$
x + y = k * (y + z)
$$
如果此时快指针只多走了一圈，那么有 $k = 1$，则：
$$
x = z
$$
也就是说，从相遇的地方开始一个指针，从链表头开始一个指针，两个指针相交的地方则为环开始的地方。

如果 $k \neq 1$，那么其实也可以用上述算法，只是在环中开始的节点会在环中多转几圈。
