---
title: Rust Modules
date: 2023-07-18 14:38:01
categories:
  - - 计算机
tags:
  - "#RSX"
---


```rust
// in player.rs, the file it self become a mod, named "player"

pub fn play_movie(name: &str) {
    println!("Playing movie {}", name);
}

pub fn play_audio(name: &str) {
    println!("Playing audio {}", name);
}
```

```rust
// in main.rs

mod player;

fn main() {
    player::play_movie("snatch.mp4");
  
    clean::perform_clean();
    clean::files::clean_files();
}

mod clean {
    pub fn perform_clean() {
        prinln!("Cleaning HDD");
    }
  
    // nested module
    pub mod files() {
        pub fn clean_files() {
            prinln!("Cleaning files");
        }
    }
}
```
