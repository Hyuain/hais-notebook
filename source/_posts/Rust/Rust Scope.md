---
title: Rust Scope
date: 2023-07-18 14:38:01
categories:
  - - 计算机
tags:
  - "#RSX"
---

**Local scope** in Rust can be specified using `{}`:

```rust
{                      // s is not valid here, it’s not yet declared
    let s = "hello";   // s is valid from this point forward

    // do stuff with s
}                      // this scope is now over, and s is no longer valid
```

**Global varibales** can be declared but they are unsafe:

```rust
static mut a = 3;
fn main() {
    unsafe {
        println!("{}", a);  
    }
}
```
