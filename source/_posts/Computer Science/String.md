---
title: String
date: 2022-02-05 15:49:12
categories:
  - - 计算机
mathjax: true
tags:
  - CX
---

# Big Number Calculation

## Big Number Plus

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

## Big Number Minus

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

# Pattern Matching

## Single-Pattern: KMP

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

## Multi-Parttern: AC

> 字符串的多模式匹配常用 Aho-Corasick Automaton（AC 自动机）来解决，其基础是 字典树（Tried）和 KMP 算法。

### Trie

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

### AC

// TBC

# Examples

## 反转字符串中的单词

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

## 计数二进制子串

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
