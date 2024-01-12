---
title: DOM
date: 2020-01-29 17:22:52
categories:
  - - 前端
tags:
  - FX
---

# 获取节点

```js
window.id // 或者直接 id

document.getElementById('idxxx')
document.getElementsByTagName('div')[0]
document.getElementsByClassName('div')[0]

document.querySelector('div>span:nth-child(2)')
document.querySelectorAll('.red')
```

## 获取特定的元素

- 获取 html： `document.documentElement`
- 获取 head： `document.head`
- 获取 body： `document.body`
- 获取 window： `window`
- 获取所有元素： `document.all`，第 6 个 `falsy` 值，别的浏览器为了不使用为了 IE 设计的代码（这个是 IE 发明的）

## 获取的元素的原型

div 的原型链：

`HTMLDivElement.prototype` → `HTMLElement.prototype` → `Elment.prototype` → `Node.prototype` → `EventTarget.prototype` → `Object.prototype`

## 节点（Node）和元素（Element）的区别

使用 x.nodeType 可以得到一个数字：

- 1 表示 Element（也叫 Tag ）
- 3 表示 Text
- 8 表示 Comment
- 9 表示 Document
- 11 表示 Document Fragment

## 获取附近的元素

- 查爸爸： `div.parentNode` 或者 `div.parentElement`
- 查子代： `div.childNodes` 或者 `div.children`，推荐使用后者，因为前者包括了文本节点，比如空格，两者都会实时变化
- 查兄弟姐妹： `div.parentNode.childNodes` 或者 `div.parentNode.children`，然后再排除自己
- 查看老大： `div.firstChild`
- 查看老幺： `div.lastChild`
- 查看上一个哥哥： `div.previousSibling`（有可能查到文本节点，可以用 `div.previousElementSibling`）
- 查看下一个弟弟： `div.nextSibling`（有可能查到文本节点，可以用 `div.nextElementSibling`）

```js
// 遍历一个 div 里面所有的元素
travel = (node, fn) => {
  fn(node)
    if(node.children) {
      for(let i = 0; i < node.children.length; i++) {
      travel(node.children[i], fn)
    }
  }
}
travel(div1, (node) => console.log(node))
```

# 创建节点

## 创建一个标签

```js
let div1 = document.createElement('div')
document.createElement('style')
document.createElement('script')
document.createElement('li')
```

## 创建一个文本节点

```js
text = document.createTextNode('你好')
```

## 在标签里面插入文本

```js
div1.appendChild(text1)
div1.innerText = '你好'
// 或者
div1.textContent = '你好'
//但是不能用
div.appendChild('你好')
```

## 插入到页面中

`document.body.appendChild(div)` 或者 `已经在页面中的元素.appendChild(div)`

```js
// 页面中有 div#test1 和 div#test2
let div = document.createElement('div')
test1.appendChild(div)
test2.appendChild(div)
// div 最终只会出现在 test2 里面
```

让两个地方都有： `let div2 = div1.cloneNode(true)`，如果后面为 `true` 则使用深拷贝，所有的后代也会被拷贝

# 删除节点

```js
div.parentNode.removeChild(div)
div.remove()
```

{% note warning %}

如果一个 Node 被移除了页面（DOM）树，它还可以再被放回去，它暂存在了内存里面，如果想删除，`div = null`，一会儿会自动回收

{% endnote %}

# 编辑节点

## 编辑属性

```js
div.className = 'red blue'// 会覆盖掉之前的 class，要用 div.className += ' red'

div.classList.add('red')

div.style = 'color: blue'
// 会覆盖掉之前的 style，一般就写 style 的一部分
// 比如 div.style.color = 'blue'，注意大小写问题：div.style.backgroundColor

div.setAttribute('data-x', 'test')

div.getAtrribute('data-x') // 强加的，这样写也行
```

{% note warning %}

如果用 `a.href` 获取属性，然后是相对路径的话，他会把域名一起弄下来

如果用 `a.href.getAttribute('href')` 只获取属性里面写了的部分

{% endnote %}

## 编辑事件

- `div.onclick` 默认为 `null`，如果改成 `fn`，那么在点击的时候就会调用 `fn.call(div, event)`，`event` 包含了点击事件的所有信息，比如坐标
- `div.addEventListener`

## 编辑内容

### 编辑文本内容

```js
test.innerText = 'hi'
test.textContent = 'hi'
```

### 修改 HTML 内容

```js
test.innerHTML = '' // 里面的字符不能超过两万个
```

### 修改标签

```js
test.innerHTML = '' // 先清空
test.appendChild(div) //再加内容
```

# 跨线程

一个例子：

```css
.start{
  border: 1px solid red;
  width: 100px;
  height: 100px;
  transition: width 1s;
}
.end{
  width: 200px;
}
```

```js
test.classList.add('start')
test.clientWidth // 如果没有这句话，上下两句话就会合并，这句话会触发重新渲染
test.classList.add('end')
```

## 属性同步

- 标准属性，会自动同步，比如 `id` `className` `title`
- `data-*` 属性，也会自动同步
- 非标准属性，不会同步到页面里，只会停留在JS 线程中，而不会自动同步到页面上

![JSProperty-1](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/JS-Property-1.png)
![JSProperty-2](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/JS-Property-2.png)

{% note warning %}

自定义属性最好以 `data-` 为前缀

{% endnote %}

## Property 和 Attribute

- Property 是 JS 线程中 div1 的所有属性
- Attribute 是 渲染引擎中 div1 对应标签的属性
- 大部分时候，同名的 Property 和 Attribute 值相等
- 如果不是标准属性，那么他们俩只会在一开始的时候相等
- 但注意 Attribute 只支持字符串，而 Property 支持字符串、布尔等类型

