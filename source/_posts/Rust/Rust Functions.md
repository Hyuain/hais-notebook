---
title: Rust Functions
date: 2023-07-18 14:38:01
categories:
  - - 计算机
tags:
  - "#RSX"
---
# Function

```rust
fn main() {
    for i in 1..6 {
        say_hi();
    }
}

fn say_hi() {
    printl!("Hello there");
}
```

```rust
fn main() {
    let mut name = "John";
    say_hi(name);
    printl!("{}", name);         // John
}

// Pass by value
fn say_hi(name: &str) {
    printl!("Hello {}", name);   // Hello John
}
```

```rust
fn main() {
    let mut name = "John";
    say_hi(&mut name);
    printl!("{}", name);         // Alex
}

// Pass by reference
fn say_hi(name: &mut &str) {
    printl!("Hello {}", name);   // Hello John
    *name = "Alex";
}
```

```rust
fn main () {
    let mut name = "John";
    let greeting = say_hi(&mut name);
    println!("{}", greeting);      // Hello John
}

fn say_hi(name: &mut &str) -> String {
    let greeting = format!("Hello {}", name);
    return greeting;               
    // "return and ;" can be omitted
    // greeting
}
```

# Anonymous Function

[[Rust Closures]]
# Higher Order Function (HOF)

> Functions that take another function as a parameter

```rust
let square = |a: i32| { a * a };
apply(square, 3);

fn apply(f: fn(i32) -> i32, a: i32) {
    println!("Result {}", f(a));
}
```

Usage of HOFs with built-in libs:

```rust
// Calculate the sum of all the squares less than 500
// Only for even numbers
let limit = 500;
let mut sum = 0;
for i in 0.. {
    let isq = i * i;
    if isq > limit { break; }
    if is_even(!isq) { continue; }
    sum += isq;
}
println!("The sum is {}", sum);

// With HOFs
let sum2 =
    (0..).map(|x| x * x)
        .take_while(|&x| x <= limit)
        .filter(|x| is_even(*x))
        .fold(0, |sum, x| sum + x);

println!("The sum2 is {}", sum2);

fn is_even(x: i32) -> bool {
    x % 2 == 0
}
```
