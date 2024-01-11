---
title: Document Flow
date: 2020-02-01 20:12:49
categories:
  - - 前端
---

# Normal Flow

> 通常文档流指的是 **Normal Flow**，也就是 **正常布局流**
> 
> 当我们没有使用过任何 CSS 规则来改变元素的展现方式的时候，他们会按照正常的布局流来组织
> 
> ——也就是默认情况下的元素布局——这也就是我们所说的 **Noraml Flow**，即 **文档流**

| `display` | 流动方向 | 宽度 | 高度 |
| --- | --- | --- | --- |
| `inline` | 从左到右排列，**行尾会截断成两行** | 里面所有 inline 元素之和，不接受指定 `width` | **实际高度**由**行高（`line-height`）间接**（与字体有关，具体可以看 [这个](https://zhuanlan.zhihu.com/p/25808995?group_id=825729887779307520)）**确定**，`padding` 只能改变**看得见**的高度（而不是实际高度），如果没有内容，也有高度，为 `line-height` |
| `block` | 从上到下排列（每个占一行） | 默认为 `auto`（不是100%），可以指定 `width`，但永远不要写 `width: 100%` | 由里面**所有**的**文档流**元素决定，也可以设置 `height`，如果没有内容，高度为 `0`
| `inline-block` | 从左到右排列，**行尾不会分成两块** | 默认为里面所有 `inline` 元素之和，但接受指定 `width` | 由里面**所有**的**文档流**元素决定，也可以设置 `height` |

# 脱离文档流

> 也就是 **Out of flow**

{% note warning %}
block 不计算其高度
{% endnote %}

```css
position: absolute | fixed;
/* OR */
float: left | right;
```
