---
title: Rust
date: 2023-07-18 14:38:01
categories:
  - [计算机]
---

熟练掌握三门语言计划之三：Rust。

资料来源：

- [The Book](https://doc.rust-lang.org/)
- [Learn Rust Programming - Complete Course](https://www.youtube.com/watch?v=BpPEoZW5IiY)

<!-- more -->

# Common Concepts

## Variables

###  Immutibility

Rust 中可以使用 `let` 来定义变量，但定义的 **变量默认是不可变（Immutable）** 的，重复给变量赋值会在编译时报错：

```rust
let x = 5;
x = 6;       // error
```

要想让变量变得可变，需要加上 `mut`：

```rust
let mut x = 5;
x = 6;       // ok 
```

此外还可以用 `const` 定义常量：

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

常量有这样几个特点：

1. 永远不可以改变
2. 需要定义数据类型
3. 在程序运行的所有生命周期内可在作用域内访问

### Scope

Rust 有两种作用域，**全局作用域（Global Scope）和局部作用域（Local Scope）**。Rust 局部的作用域可以用花括号来区分：

```rust
{                      // s is not valid here, it’s not yet declared
    let s = "hello";   // s is valid from this point forward

    // do stuff with s
}                      // this scope is now over, and s is no longer valid
```

### Shadowing

Rust 还有一个特性叫 **Shadowing**，这个特性允许变量被重复定义（在 JavaScript 中，只有在新的作用域里面才能重新定义同名变量）：

```rust
let x = 5;
let x = 6;       // ok
{
  let x = x * 2; // ok
}
```

Shadowing 允许两次定义的变量类型不同，因为他们实际上是两个变量，而 `mut` 则不允许两次赋值为不一样的类型：

```rust
let spaces = "    ";
let spaces = spaces.len();    // ok
```

```rust
let mut spaces = "    ";
spaces = spaces.len();        // error
```

### Unused Varibales

如果定义了一个变量但没有使用，Rust 编译器会报 Warning：

```rust
fn main() {
    let x = 1;
}

// Warning: unused variable: `x`
```

有两种方式可以解决，一种是在变量前面加下划线：

```rust
fn main() {
    let _x = 1;
}
```

另一种是在函数上面加 `#[allow(unused_varibales)]`：

```rust
#[allow(unused_varibales)]
fn main() {
    let x = 1;
}
```

### Destructring

Rust 支持解构元组：

```rust
let (mut x, y) = (1, 2);
```

还支持这样解构赋值：

```rust
let (x, y);
(x,..) = (3, 4);
[.., y] = [1, 2];
```

## Data Types

Rust 中的数据类型分为 **标量类型（Scalar Types）** 和 **组合类型（Compond Types）**。

**标量类型** 包括：

- **整型（Integer）**：包括 **有符号整型（`i` 开头）**和 **无符号整型（`u` 开头）**，比如 `i8` `u128` 等，数字表示他占了多少位。其中 `isize` 和 `usize` 类型取决于程序运行所处的计算机的体系结构，比如 64 位或 32 位等。
- **浮点型（Floating-Point）**：包括 `f32` 和 `f64`。
- **布尔型（Boolean）**：`bool`，可以用 `true` 或 `false` 显式指定。
- **字符型（Character）**：`char`，与字符串字面量不同，用单引号 `' '` 指定，所有的 `char` 都占 4 Bytes（不管是中文还是英文还是特殊字符），可以用 `size_of_val()` 函数看到。

**组合类型** 包括：

- **元组（Tuple）**：用括号框起来的、逗号隔开的一组值，比如 `let tup: (i32, f64, u8) = (500, 6.4, 1)`，可以用 `tup.0` `tup.1` 等来取到里面的值。**元组中的值可以是不同类型的。**空的元组类型也被称为 **单位类型（Unit Type）**，当函数没有指定返回值时，返回值就是一个 Unit Type，**并且大小是 0**：

  ```rust
  fn main() {
    let v: () = ();
    assert_eq!(v, implicitly_return_unit());
  }
  
  fn implicitly_return_unit() {
    println("I will return a ()");
  }
  ```
- **数组（Array）**：**数组中的值类型必须相同，且 Rust 中的数组是定长的，且不能越界访问。** 可以这样定义数组类型 `let a: [i32; 5] = [1, 2, 3, 4, 5];` 也可以在最开始为所有元素赋相同的值 `let a = [3; 5];`。

## Statements & Expression

```rust
let x = 4;
let y = {
    let x_squared = x * x;
    let x_cube = x_squared * x;
    // 注意这里没有分号，这个值将成为这个 {} 表达式的值，否则表达式的值将会是 Unit Type ()
    x_cube + x_squared + x
}
```

我们将 `let y = {...}` 称为一个 **Statement**，他是一个赋值操作，并且不会产生任何值；将等号右边的部分称为一个 **Expression**，因为他最后没有以分号结尾，并且会产生一个值。

## Functions

```rust
fn main() {
    let s: i32 = sum(1, 2);
}

fn sum(x: i32, y: i32) -> i32 {
    x + y
}
```

### Diverging Functions

 **分流函数（Diverging Functions）**永远不会返回：

```rust
fn main() {
    never_return();
    // 下面的永远不会执行
}

fn never_return() -> ! {
    // 抛出一个错误
    panic!()
}
```

```rust
fn main() {
  
}

fn get_option(tp: u8) -> Option<i32> {
    match tp {
        1 => {
            // TODO  
        }
        _ => {
            // TODO  
        }
    };
  
    // 使用分流函数，而不是返回 None
    never_return_fn()
}

fn never_return_fn() -> ! {
    // panic!()
    // unimplemented!()
    todo!()
}
```

## Flow Control

### 条件

```rust
if number < 5 {
    // ..
} else if number < 10 {
    // ..
} else {
    // ..
}
```

```rust
let condition = true
let number = if condition { 5 } else { 6 }
```

### 循环

#### loop

```rust
loop {
    // ..
}
```

```rust
let result = loop {
    counter += 1;
    if counter == 10 {
        break counter * 2;  
    }
}
```

```rust
let mut count = 0;
'counting_up: loop {
    println!("count = {count}")
    let mut remaining = 10;
  
    loop {
        printl!("reamianing = {remaining}")    // always 10 -> 9
        if remaining == 9 {
            break;  
        }
        if count == 2 {
            break 'counting_up;  
        }
        remaining -= 1;
    }
  
    count += 1;
}
println!("End count = {count}")  // 2
```

#### while

```rust
while number != 0 {
    number -= 1;
}
```

#### for

```rust
let a = [10, 20, 30, 40, 50];
for element in a {
    println!("the value is: {element}")
}
```

```rust
for number in (1..4).rev() {
    println!("{number}!")
}
```

# Ownership

Rust 中的一个核心概念是 **所有权（Ownership）**。Rust 中没有 GC，但也不需要开发者手动释放内存来防止内存泄漏，而是使用了 Ownership 与一系列的编译时规则来实现内存安全。

## Heap and Stack Memory

大多数语言都有 **堆内存和栈内存**，但通常不需要太过深究，但 Rust 中数据是存在堆还是栈中会很大程度上影响其表现。

堆与栈内存都是在运行时代码可以使用的内存，关于堆栈的相关介绍情看 {% post_link Algorithm %}  相关部分。

**重要的是：**

- **在栈中存储的数据必须是确定的、固定的大小；**
- **在堆中分配（Allocate）内存时，会返回一个指针指向这个内存的地址，该定长地址可以存储在栈中；**
- 向栈中 Push 数据比在堆中 Allocate 要快得多，因为不需要先找到合适的地方来存储这个数据；
- 访问栈中的数据要比对堆中的快得多，因为堆中的数据要顺着指针来找；
- 调用函数的时候，传入函数的参数与函数本地的数据会 Push 进栈中，函数运行结束后，栈中的数据会被 Pop 出来；
- 栈内存有天然的 Push 和 Pop，使其不会有冗余的数据，但堆内存则需要及时堆不需要的部分进行清理。

## Ownership Rules

Rust 中有三条 Ownership Rules：

- **Rust 中的每个值都有一个 Owner。**
- **每个值同时只能有一个 Owner。**
- **当 Owner 离开作用域（Scope）时，值会被丢弃。**

## Complex Types

对于简单类型，比如 `integer`，表现跟其他语言类似：

```rust
let x = 5;
let y = x;
```

以上代码，会创建两个变量 `x` 和 `y`，均为 `5`，并 Push 到了 **栈** 中。

而复杂类型则不同，他会使用 **堆** 内存，下面以 `String` 举例。

### String vs. &str

```rust
// 可以通过字符串字面量创建一个 String
let mut s = String::from("hello");
s.push_str(", world!")
```

`String` 是复杂类型，他由三部分组成（这三部分数据保存在 **栈** 中）：

- 一个指针指向 **堆** 中保存字符串内容的的内存 `ptr`；
- 字符串的长度 `len`；
- 字符串的容量 `capacity`。

因此上面 `s` 会在栈中占据 `3 * 8 = 24 Bytes` 大小，

### Copy

简单类型赋值时，会**自然地将栈中的数据复制一份**，并且原来的变量也不会失效，这称为 **Copy**：

```rust
let x = 5;
let y = x;

println!("x = {}, y = {}", x, y);
```

### Move

而复杂类型则不同，比如对于 `String`，如果我们将 `s1` 赋值给 `s2`，那么 **堆中的数据不会被复制，栈中的数据会被复制一份**：

```rust
let s1 = String::from("hello");
let s2 = s1;
```

此外，由于变量离开作用域时，Rust 会自动调用 `drop` 方法来清理内存，这就导致如果 `s1` 和 `s2` 都离开作用域时，堆中的同一块内存会被清理两次，这会导致内存安全问题。

**因此，当 `s1` 赋值给 `s1` 之后，`s1` 将不再可用。**这样的话 Rust 就不会再清理 `s1` 的内存。

**这种特性与其他语言的浅拷贝（Shallow Copy）有所不同，Rust 称其为移动（Move），因为之前的变量将不再可用。**

### Clone

如果想要对复杂类型执行 **深拷贝（Deep Copy）**，不仅复制其栈中的数据，还复制其堆中的数据，可以使用这些类型中提供的 `clone` 方法：

```rust
let s1 = String::from("hello");
let s2 = s1.clone();

println!("s1 = {}, s2 = {}", s1, s2);
```

## Ownership and Functions

```rust
fn main() {
    let s = String::from("hello");  // s comes into scope

    takes_ownership(s);             // s's value moves into the function...
                                    // ... and so is no longer valid here

    let x = 5;                      // x comes into scope

    makes_copy(x);                  // x would move into the function,
                                    // but i32 is Copy, so it's okay to still
                                    // use x afterward

} // Here, x goes out of scope, then s. But because s's value was moved, nothing
  // special happens.

fn takes_ownership(some_string: String) { // some_string comes into scope
    println!("{}", some_string);
} // Here, some_string goes out of scope and `drop` is called. The backing
  // memory is freed.

fn makes_copy(some_integer: i32) { // some_integer comes into scope
    println!("{}", some_integer);
} // Here, some_integer goes out of scope. Nothing special happens.
```

```rust
fn main() {
    let s1 = gives_ownership();         // gives_ownership moves its return
                                        // value into s1

    let s2 = String::from("hello");     // s2 comes into scope

    let s3 = takes_and_gives_back(s2);  // s2 is moved into
                                        // takes_and_gives_back, which also
                                        // moves its return value into s3
} // Here, s3 goes out of scope and is dropped. s2 was moved, so nothing
  // happens. s1 goes out of scope and is dropped.

fn gives_ownership() -> String {             // gives_ownership will move its
                                             // return value into the function
                                             // that calls it

    let some_string = String::from("yours"); // some_string comes into scope

    some_string                              // some_string is returned and
                                             // moves out to the calling
                                             // function
}

// This function takes a String and returns one
fn takes_and_gives_back(a_string: String) -> String { // a_string comes into
                                                      // scope

    a_string  // a_string is returned and moves out to the calling function
}
```

## Reference and Borrowing

如果传参的时候交出了 Ownership，原来的变量就不可用了，因此需要将原来的值再手动返回回来，重获 Ownership：

```rust
fn main() {
    let s1 = String::from("hello");

    let (s2, len) = calculate_length(s1);

    println!("The length of '{}' is {}.", s2, len);
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() returns the length of a String

    (s, length)
}
```

**这时可以使用引用（Reference），这样函数就不会拿到 Ownership，在其作用域结束的时候也就不会被清除：**

```rust
fn main() {
    let s1 = String::from("hello");

    // &s1 创建了一个 Reference，指向 s1，但并没有 **拥有** 它，Ownership 没有发生转移
    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

// &String s 指向 String s1
fn calculate_length(s: &String) -> usize {
    s.len()
} // Here, s goes out of scope. But because it does not have ownership of what
  // it refers to, it is not dropped.
```

创建 Reference 的过程称为 **借用（Borrowing）**。

### Mutable Reference

可以通过 `&mut` 来创建可变引用：

```rust
fn main() {
    let mut s = String::from("hello");
    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

**Mutable References 有一些限制：**

- **每个变量同时只能有一个 Mutable Reference（在一个作用域中），但可以拥有多个 Immutable References；**
- **变量不能同时拥有 Mutable Reference 和 Immutable Reference（防止 Dangling References）；**

```rust
let mut s = String::from("hello");

let r1 = &s;
let r2 = &s;
let r3 = &mut s;   // Error

println("{}, {}, and {}", r1, r2, r3);
```

```rust
let mut s = String::from("hello");

let r1 = &s;
let r2 = &s;
println("{} and {}", r1, r2);
// variables r1 and r2 will not be used after this point

let r3 = &mut s;   // No problem
println("{}", r3);
```

