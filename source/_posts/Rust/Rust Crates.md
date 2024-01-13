---
title: Rust Crates
date: 2023-07-18 14:38:01
categories:
  - - 计算机
tags:
  - "#RSX"
---

# Crates

> Multiple modules are grouped into a crate.

Two types of crates:

- **Binary Crates:** the main component (entry point) of the crate (`main.rs`) has a `main` function.

```rust
// in archive.rs

pub mod arch {
    pub fn arch_file(name: &str) {
        println!("Archiving file {}", name);
    }
}
```

```rust
// in main.rs

// use crate::archive::arch::arch_file;
use crate::archive::arch::arch_file as arc;


mod archive;

fn main() {
    // arch_file("file.txt");
    arc("file.txt");
}
```

- **Library Crates:** can be without a entry point, but sometimes has a `lib.rs` as a entry point exporting functions.

Cargo is used to manage crates.

External crates are imported in to the project must be added to the `Cargo.toml` file.

```tom
[dependencies]
rand = "0.8.5"
```

```rust
// use rand::Rng;
use rand::{Rng,};

fn main() {
    let mut rng = rand::thread_rng();
    let a: i32 = rng.gen();
    println!("{}", a);
}
```

# Useful Crates

## Random Generator

```rust
use rand::Rng;
let mut rng = rand::thread_rng();
// generate an integer
rng.gen();
// bounded generation
rng.gen_range(0..10);
rng.gen_range(0.0..10.0);
```

```rust
use rand::{Rng, thread_rng};
use rand::distributions::Alphanumeric;

fn main() {
    let rand_string: String = thread_rng()
        .sample_iter(&Alphanumeric)
        .take(30)
        .map(char::from)
        .collect();
    println!("Gen string: {}", rand_string);
}

```
