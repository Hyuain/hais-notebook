---
title: Spinlock
date: 2024-01-13 15:37:30
categories:
  - - 计算机
tags:
  - CX
---
Refs:

- [Rust OS](https://github.com/Hyuain/rust-os/blob/main/docs/s2-vga_buffer.md)

**Spinlock(自旋锁)** is a lock that causes a thread trying to acquire it to simply wait in a loop ("spin") while repeatedly checking whether the lock is available. An overview of how a spinlock works:

1. A thread that wants to access shared resource tries to acquire the spin lock  
2. If the lock is **available** (not currently held by another thread), the thread acquires the lock and proceeds to access the shared resource  
3. If the lock is **unavailable**, **the thread enter a busy-wait loop**, repeatedly checking if the lock has been released  
4. Once the lock is released by the owning thread, another thread can acquire it and access the shared resource
