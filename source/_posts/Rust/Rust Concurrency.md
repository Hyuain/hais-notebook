---
title: Rust Concurrency
date: 2023-07-18 14:38:01
categories:
  - - 计算机
tags:
  - RSX
---

# Threads

Rust allow us to run code in parallel in different [[Thread|threads]]

Ownership/ borrowing mechanism gives us

- memory safety (even in multi-threads programs)
- no data races (because the data only have exactly one owner)

Create a thread

```rust
use std::thread;
thread::spawn(|| {
    println!("new thread");
});
```

`move` keyword helps to use variables inside the clousre

```rust
for i in 0..10 {
    // to use i inside the clouser, we must add `move`, moving data into the closure
    thread::spawn(move || {
        println!("new thread {}", i);
    });
}

println!("Main thread");

/*
new thread 1
new thread 2
new thread 3
new thread 0
new thread 4
new thread 5
new thread 6
new thread 7
new thread 8
Main thread
new thread 9
*/
```

Sleep a thread using `sleep()` method

```rust
for i in 0..10 {
    thread::spawn(move || {
        // sleep the program
        sleep(Duration::from_millis(i * 1000));
        println!("new thread {}", i);
    });
}

println!("Main thread");

/*
new thread 0
Main thread (main program finished before other threads have finished)
*/
```

Wait for a thread use `join()` method

```rust
let mut threads = vec![];

for i in 0..10 {
    let th = thread::spawn(move || {
        sleep(Duration::from_millis(i * 1000));
        println!("new thread {}", i);
    });
    threads.push(th);
}

for th in threads {
    th.join();
}

println!("Main thread");

/*
new thread 0
new thread 1
new thread 2
new thread 3
new thread 4
new thread 5
new thread 6
new thread 7
new thread 8
new thread 9
Main thread
*/
```

# Channels

A way to send data between threads

> **MPSC: Multiple producer single receiver**

Create a channel

```rust
use std::sync::mpsc;
// transimiter and receiver
let (tx, rx) = mpsc::chanel();
```

Send a message

```rust
tx.send()
```

Receive a message

```rust
rx.recv()      // blocking
rx.try_recv()  // non blocking
```

A example of blocked receving. The main thread will wait until it receives message.

```rust
let (tx, rx) = mpsc::channel();
thread::spawn(move || {
    tx.send(42).unwrap();
});
println!("received {}", rx.recv().unwrap());
```

Use `rx` to let the main thread wait for all other threads like `join`:

```rust
const NUM_THREADS: usize = 20;

fn main() {
    let (tx, rx) = mpsc::channel();
    for i in 0..NUM_THREADS {
        start_thread(i, tx.clone());
    }

    for j in rx.iter().take(NUM_THREADS) {
        println!("received {}", j);
    }
}

fn start_thread(d: usize, tx: mpsc::Sender<usize>) {
    thread::spawn(move || {
        println!("setting timer {}", d);
        thread::sleep(Duration::from_secs(d as u64));
        println!("sending {}", d);
        tx.send(d).unwrap();
    });
}

```

# Mutex

> Mutual exclusion lock. Only one thread can access the data at any one time.

**Arc - Atomically referenced counted type.** Convert data into primitive types, safe to share across threads.

Create a lock:

```rust
use std::sync::{Mutex, Arc};
let lock = Arc::new{Mutex::new(0)};
```

Acquire a lock:

```rust
lock.lock();
lock.try_lock();
```

A example that a mutal value is stored using `Mutex::new()` and shared with `Arc`:

```rust
let c = Arc::new(Mutex::new(0));
let mut threads = vec![];

for _i in 0..10 {
    let c = Arc::clone(&c);
    let t = thread::spawn(move || {
        let mut num = c.lock().unwrap();
        *num += 1;
    });
    threads.push(t);
}

for th in threads {
    th.join().unwrap();
}

println!("Result {}", *c.lock().unwrap());    // 10
```

Poisoned lock - when a thread that holds the lock panics:

```rust
lock.is_poisoned();
```

