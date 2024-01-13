---
title: Rust Project
date: 2023-07-18 14:38:01
categories:
  - - 计算机
tags:
  - RSX
---

Key Points:

- Manage Rust environment using `rustup`
- Manage Rust project using `cargo`
- Handle user input and print values to the console
- Unit testing

<!-- more -->

# Rustup

Rust is installed and managed by the [`rustup`](https://rust-lang.github.io/rustup/) tool. It installs Rust from the official release channels, enabling you to easily switch between stable, beta, and nightly compilers and keep them updated.

# Cargo

Cargo is the project manager for Rust.

```bash
cargo new
cargo build
cargo run
cargo clean
cargo check
cargo doc
```

We can use `cargo add` to add dependencies to a cargo project:

````bash
cargo add tokio
````

And use `--features` to specify features:

```bash
cargo add --features full
```

# Input & Output

## User Input

Rust is not typically designed for user interaction, and the way it handle user input is a little bit complex.

```rust
use std::io;

fn main() {
    let mut input = String::new();
    println!("Say something");
    match io::stdin().read_line(&mut input) {
        Ok(_) => {
            println!("You said {}", input);
        },
        Err(e) => {
            println!("Something went wrong {}", e);
        }
    }
}
```

## Printing Values

```rust
println!("Hello world!");

// Formatting
println!("My name is {} and I am {}", "Harvey", 26);

// Expressions
println!("a + b = {}", 3 + 6);

// Positional Arguments
println!("{0} has a {2} and {0} has a {1}", "Harvey", "cat", "dog");

// Named arguments
println!("{name} {surname}", surname="Z", name="Harvey");

// Printing traits
println!("Binary: {:b}, Hex: {:x}, Octal: {:o}", 5, 5, 5);

// Debug
println!("Array {:?}", [1, 2, 3]);
```

# Comments

```rust
// line coment
/*
  multiline coment
*/
/// doc comments, used to document functionality
//! doc comments, used to document crates
//! # Heading
//! ```
//!   document code
//! ```
```

Run `cargo doc` to generate documents according to comments.

# Unit Testing

 ```rust
 pub fn add_five(num: u32) -> u32 {
     num + 5
 }
 
 // the code only runs in `cargo test`
 #[cfg(test)]
 mod test {
     // inherit libraries imported
     use super::*;
   
     #[test]
     fn add_five_test() {
         let x = 100;
         let y = add_five(x);
         println!("x and y are from test {}", x, y);
         assert_eq!(y, 105);
     }
 }
 ```

Then use `cargo test` to run tests.

To make content in `println!()` printed in the console, use the following command:

```bash
# `--` is to separate the arguments for `cargo test` and the test binary itself
cargo test -- --nocapture
```
