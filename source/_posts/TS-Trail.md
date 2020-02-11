---
title: TypeScript 基本用法
date: 2020-02-06 20:33:48
tags:
  - 入门
  - Tip
  - 踩坑
categories:
  - [前端, TypeScript]
---

TypeScript 三个好处： 1. 自动提示更智能；2. 不能随便写 .xxx()；3. 错误前移到编译时 

<!-- more -->

# Vue 的 TS 组件

```typescript
import Vue from 'vue';
import {Component, Prop} from 'vue-property-decorator';

// 这是一个装饰器，告诉 Vue 这是一个组件，Vue 会自动把下面的东西处理成 data、method 等
@Component({
  // components: {Child,...}
})
export default class MyComponent extends Vue {
  num = 0; // 这就是 TS 组件的 data

  @Prop(Number) propA: number | undefined; // 这是 vue-property-decorator 带来的 props 写法
  // @Prop 是装饰器，告诉 Vue，后面的东西不是 data，是 prop
  // 左边的 Number 是告诉 Vue，propsA 是 Number，是**运行时**的类型检查
  // 右边的 number 是告诉 TS propsA 的类型，是**编译时**的类型检查，比如说你没办法写 this.propA.xxx，会无法编译成功

  add(num: number) { // 这就是 TS 组件的 methods
    this.num = num ++;
  }

  mounted() {
    console.log(this.propA);
  }
}
```

## 编译时与运行时

TypeScript --编译--> JavaScript --运行--> 浏览器

编译时：编译错误 -> 无法得到 JS，会在编译的终端 Error
运行时：运行错误 -> 控制台 Error（浏览器）

# TypeScript 的本质

{% cq %}
JS: 类型
{% endcq %}

TSC（TypeScriptCompiler）会利用类型来检查 JS 代码：
- 若检查出错误，就会编译报错，但还是会编译成 JS
- 若没有检查出错误，就会删掉类型，然后编译成 JS（TSC 或者 Babel 都可以编译，有一点点区别）

# 一些问题

## 强制指定类型

```typescript
inputNumber(event: MouseEvent | TouchEvent) {
  const button = (event.target as HTMLButtonElement); // 因为有的元素的 textContent 可能为空（比如图片），所以我们需要强制指定为 Button 元素
  console.log(button.textContent);
}

// 也可以给返回值强制指定类型
function Func() {
  return JSON.parse(localStorage.getItem('records') || '[]') as RecordItem[];
}
```

## 配合 JS 使用

```typescript
const xxx require('...')
```

## 全局声明 `type`

在 `src` 中创建 `custom.d.ts`（也可以叫 `xxx.d.ts`），在里面声明 `type`