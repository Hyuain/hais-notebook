---
title: Vue 组件
date: 2020-02-09 20:17:46
tags:
  - 入门
  - 饥人谷
  - 文档
categories:
  - [前端, JavaScript, 框架, Vue]
---

记录 Vue Component 中的内容。

<!-- more -->

# Vue 组件的三种写法

1. 用 JS 对象

```js
export default {
  options
}
```

2. 使用 TS 类

```typescript
import Vue from 'source/_posts/Others-QA-Vue';
import {Component, Prop} from 'vue-property-decorator';

@Component({
  // components: {...}
})
export default class MyComponent extends Vue {
  num: number = 0;
  @Prop(Number) propA: number | undefined;
  add(num: number) {
    this.num = num ++;
  }
  mounted() {
    console.log(this.propA);
  }
}
```

3. 使用 JS 类

```js
import Vue from 'vue';
import {Component, Prop} from 'vue-property-decorator';

@Component
export default class MyComponent extends Vue {
  num = 0;
  @Prop(Number) propA
  add(num) {
    this.num = num ++;
  }
  mounted() {
    console.log(this.propA);
  }
}
```

