---
title: Rust Memory Management
date: 2023-07-18 14:38:01
categories:
  - - 计算机
tags:
  - RSX
---

# Heap and Stack Memory

大多数语言都有 **堆内存和栈内存**（比如 [[JavaScript Memory|JavaScript]]），但通常不需要太过深究，但 Rust 中数据是存在堆还是栈中会很大程度上影响其表现。

堆与栈内存都是在运行时代码可以使用的内存，关于堆栈的相关介绍情参阅[[Stack|栈]]或[[Heap|堆]]。

|  | Stack | Heap |
| ---- | ---- | ---- |
| Size | Fixed (known at compile time) | Dynamic |
| Speed | Faster than Heap | Slower than Stack |
| Usage | Local variables, function call data | Dynamic data, largeobjects that outlive their scope |
| Management | Automatic in Rust | Automatic in Rust (ownership system) |
| Scope | Within scope where declared | Anywhere complying with ownership rules |
|  |  |  |

An example of stack:

```rust
fn man() {
    let a = 10;          // stored in stack frame main()
    let b = 20;          // stored in stack frame main()
    some_func {          // create new stack frame {}
        let c = 30;      // stored in stack frame {}
        let d = &c;      // the value of d is the address of c, d is stored in stack frame {}
    }                    // stack frame {} poped, c and d droped
    let e = add_five(a)  // create new stack frame add_five(), num and i stored there
}

fn add_five(num: i32) -> i32 { // num has the same value as a, but in different address
    let i = 5;
    num + 1;
}
```

# Primitive vs. Complex Types

对于简单类型，比如 `integer`，表现跟其他语言类似：

```rust
let x = 5;
```

以上代码，会创建变量 `x`，并 Push 到了 **栈** 中。

而复杂类型则不同，他会使用 **堆** 内存，下面以 `String` 举例。

```rust
// 可以通过字符串字面量创建一个 String
let mut s = String::from("hello");
s.push_str(", world!")
```

`String` 是复杂类型，他由三部分组成（这三部分数据保存在 **栈** 中）：

- 一个指针指向 **堆** 中保存字符串内容的的内存 `ptr`；
- 字符串的长度 `len`；
- 字符串的容量 `capacity`。

因此上面 `s` 会在栈中占据 `3 * 8 = 24 Bytes` 大小。

`&str` 有点微妙，被称为字符串切片（String Slice），他是基础类型 `str` 的引用。`str` 与 `i32` 等基础类型不同，他没有固定的长度，并且可能保存在堆中、栈中、或者是程序文本中。通常我们不能直接使用他，而是通过 `&str` 来使用他，`&str` 作为一个引用，他本身的长度是固定的，因此是保存在栈中的。

Rust 中，`&str` 类型的数据可以直接使用 `str` 的方法，比如 `len()`，因为 Rust 会自动帮我们解引用（Dereference）。

Rust 中，可以通过 Borrowing 从 `String` 中拿到 `&str`：

```rust
let owned_string: String = "Hello, world!".to_string();
// NOTE: Deref coercion happened! &String -> &str
let string_slice: &str = &owned_string; // Borrowing a slice of the whole String
```

# Ownership

> Only one variable can own a piece of memory

Copy and Move:

- For primitive types (like i32), copying data is cheap
- For complex types (like struct), ownership is transferred

Assigning values of **primitives**, **data stored in stack is copied**, previous data is still available.

```rust
let x = 5;
let y = x;

println!("x = {}, y = {}", x, y);
```

For **complex** values, if value of `s1` is assigned to `s2`, **data in stock is copied, while data in heap is not copied.**

This is called **move** in Rust.

```rust
let s1 = String::from("hello");
let s2 = s1;

println!("{}", s1);     // error, because value is nolonger owned by s1
```

Because in Rust  `drop` method is called to clean up memory.

**To prevent the same memory being clean up twice**, which can cause memory safety issues, `s1` is nolonger accessible after ownership transferred.

This way, memory of `s1` is not considered in cleaning up.

Here is another example:

```rust
fn main() {
    let v = vec![1, 2, 3, 4];

    let foo = |v: Vec<i32>| -> Vec<i32> {
        println!("Vector used in foo");
        v
    };
    
    let x = foo(v);
    println!("{:?}", v);   // error, because v is passed into v and nolonger accessible
}
```

**Move** is different from shallow copy in other languages, because the previous variables are nolonger accessible. 

# Clone

如果想要对复杂类型执行 **深拷贝（Deep Copy）**，不仅复制其栈中的数据，还复制其堆中的数据，可以使用这些类型中提供的 `clone` 方法：

```rust
let s1 = String::from("hello");
let s2 = s1.clone();

println!("s1 = {}, s2 = {}", s1, s2);
```

# Borrowing

> Variables can borrow ownership to other pieces of memory

```rust
let a = 6;
let b = &a;            // b borrows ownership from a
println!("{}", *b);    // once b is destroyed, the ownership is transfered back to a
```

The borrow has to match the mutability

- **Immutable variable cannot have mutable references**
- One variable can only have one **mutable reference** (in each scope), but can have **mutiple immutable references**
- Variables can not have **mutable reference** and **immutable reference** at the same time (prevent dangling references)

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

```rust
let mut a = 6;
let b = &mut a;
println!("{}", a);  // Error, mutable borrow occurs after
println!("{}", *b);
```

# Lifetimes

> An indication of how long an object can live

Rust prevents parts of objects outliving the object

```rust
struct Object<'lifetime> {
    field: &'lifetime str
}
```

```rust
fn main() {
    let p1 = Person { name: String::from("John") };
    let d1 = Dog { name: String::from("Max"), owner: &p1 };

    println!("{:?}", d1);
}

#[derive(Debug)]
struct Person {
    name: String
}

#[derive(Debug)]
struct Dog<'l> {
    name: String,
    // a reference to Person
    // Rust need to know how long the Person should live
    // to match the lifetime of Dog and Person
    owner: &'l Person,
}
```

To live as long as the program is running:

```rust
&'static str
```

Lifetime elision: Compiler builds lifetimes for us when evident

```rust
fn main() {
    let p1 = Person { name: String::from("John") };
    let mut a: &String;
    let mut b: &String;
    {
        let p2 = Person { name: String::from("Mary") };
        a = p1.get_name();
        b = p2.get_name();
    }
    println!("{:?}", b);    // error, borrowed value does not live long enough
  	println!("{:?}", a);    // ok
}

#[derive(Debug)]
struct Person {
    name: String
}

impl Person {
    // equals: fn get_name<'l>(&'l self) -> &'l String {
    fn get_name(&self) -> &String {
        &self.name
    }
}
```

# Rc (Referenced Counted Variables)

> A structure that can hold multiple references to a variable, and can be shared in different places

```rust
use std::rc::Rc;

fn main() {
    let brand = Rc::new(String::from("BMW"));
    println!("pointers: {}", Rc::strong_count(&brand));
    {
        let car = Car::new(brand.clone());
        car.drive();
        println!("pointers: {}", Rc::strong_count(&brand));
    }
    println!("My car is a {}", brand);
    println!("pointers: {}", Rc::strong_count(&brand));
}


struct Car {
    brand: Rc<String>
}

impl Car {
    fn new(brand: Rc<String>) -> Car {
        Car { brand }
    }
    fn drive(&self) {
        println!("{} is driving", &self.brand);
    }
}

```

Rc is useful to use multiple references without considering ownership and borrowing.

# Examples

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

