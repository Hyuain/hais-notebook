---
title: JavaScript Scope
date: 2020-01-29 17:22:52
categories:
  - - 前端
tags:
  - FX
---

# Scope

**静态作用域 / 词法作用域（Static Scope / Lexical Scope）**：JavaScript 的作用域为静态作用域（也称词法作用域），作用域的确定与函数的执行无关。（但变量的值在函数执行的时候才能确定）

 **就近原则**：如果有多个作用域有同名变量 `a`，那么查找 `a` 的声明时，就向上取最近的作用域

**块级作用域与函数作用域**：

- `let` `const` 是 **块级作用域**，他们只在同一个代码块 `{}` 内可访问；
- `var` 是 **函数作用域** 。

**全局作用域与模块作用域**：

- 对于 `let` 和 `const` 来说，全局作用域的变量需要在代码的最上方定义；
    ```html
    <script>
      const foo = "foo";
    </script>
    <script>
      console.log(foo); // "foo"
      function bar() {
        if (true) {
          console.log(foo);
        }
      }
      bar(); // "foo"
    </script>
    ```

- 每个模块（Module）都有自己的作用域，只能在模块内访问。
    ```html
    <script type="module">
      const foo = "foo";
    </script>
    <script>
      console.log(foo); // Uncaught ReferenceError: foo is not defined
    </script>
    ```

- 对于 `var` 来说，由于变量提升，`var` 会绑定到 `window` 上。但定义在 Module 内部的 `var` 并不会绑定到 `window` 上，也不能被 Module 外部访问。

# Examples

```js
let foo = function() { console.log(1) };
(function foo() {
    foo = 10  // 非匿名自执行函数，函数变量为 只读 状态，无法修改。由于foo在函数中只为可读，因此赋值无效
    console.log(foo) // ƒ foo() { foo = 10 ; console.log(foo) }
}())
foo() // console.log(1)
```

```js
(function foo() {
  console.log(1)
}())
foo // foo is not defined
```
