---
title: Rust Expressions & Statements
date: 2023-07-18 14:38:01
categories:
  - - 计算机
tags:
  - RSX
---

# Expressions & Statements

```rust
let x = 4;
let y = {
    let x_squared = x * x;
    let x_cube = x_squared * x;
    // 注意这里没有分号，这个值将成为这个 {} 表达式的值，否则表达式的值将会是 Unit Type ()
    x_cube + x_squared + x
}
```

我们将 `let y = {...}` 称为一个 **Statement**，他是一个赋值操作，并且不会产生任何值；

将等号右边的部分称为一个 **Expression**，因为他最后没有以分号结尾，并且会产生一个值。

# Control Structures

## Conditions

### If Statement

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

### Match

```rust
let choice = Suit::Heart;

match choice {
    Suit::Heart => {
      // ...
    }
    Suit::Spade => {
      // ...
    }
    Suit::Club => {
      // ...  
    }
    Suit::Diamond => {
      // ...
    }
}

enum Suit {
    Heart,
    Spade,
    Club,
    Diamond,
}
```

```rust
fn country(code: i32) {
    let country = match code {
        44 => {
            "UK"
        }
        34 => "Span",
        1..=999 => "unknown",
        _ => "invalid"
    }
    println!("Country is {}", country)
}
```

### Pattern Matching

```rust
fn get_oranges(amount: i32) -> &'static str {
    return match amount {
        0 => "no",
        // multiple values
        1 | 2 => "one or two",
        // ranges
        3..=7 => "a few",
        // conditions
        _ if amount % 2 == 0 => "an even amount of",
        // default
        _ => "lots of"
    }
}
```

Tuple matching:

```rust
let point = (1, 0);

match point {
    (0, 0) => println!("origin"),
    (x, 0) => println!("x axis ({}, 0)", x),
    (0, y) => println!("y axis (0, {})", y),
    (x, y) => println!("({}, {})", x, y)
}
```

## Loops

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

### For Loop

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

```rust
let pets = ["cat", "dog", "chihuahua", "hamster", "bear"];

// pet is &str
for pet in pets {
    if pet == "chihuahua" {
        println!("found chihuahua!")
    }
}

// pet is &&str
for pet in pets.iter() {
    // add & before "chihuahua"
    if pet == &"chihuahua" {
        println!("found chihuahua!")
    }
}
```

### While Loop

```rust
while number != 0 {
    number -= 1;
}
```
