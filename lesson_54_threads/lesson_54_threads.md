# 📘 Lesson 54 — Threads & spawn (CC1)

> **Series:** Rust From Zero · Intermediate Level  
> **Roadmap ID:** CC1 · Category: ⚡ Concurrency  
> **Previous:** [Lesson 53 — RefCell & Interior Mutability](../lesson_53_refcell/lesson_53_refcell.md)  
> **Next:** [Lesson 55 — Channels (mpsc)](../lesson_55_channels/lesson_55_channels.md)  
> **Practice:** [Questions](./lesson_54_questions.md) · [Answers](./lesson_54_answers.md)  
> **Practice Task:** Parallel word-count across multiple "files"

---

## Table of Contents

1. [Why Concurrency?](#1-why-concurrency)
2. [Creating Threads with spawn](#2-creating-threads-with-spawn)
3. [JoinHandle — Waiting for Threads](#3-joinhandle--waiting-for-threads)
4. [move Closures with Threads](#4-move-closures-with-threads)
5. [Returning Values from Threads](#5-returning-values-from-threads)
6. [Thread Safety: Send and Sync](#6-thread-safety-send-and-sync)
7. [Scoped Threads](#7-scoped-threads)
8. [Thread Builder](#8-thread-builder)
9. [Real-World Example: Parallel Word Count](#9-real-world-example-parallel-word-count)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. Why Concurrency?

Concurrency lets you run multiple tasks simultaneously, utilizing multi-core CPUs:

```
Sequential:     [Task A ████████][Task B ████████]  → 16 seconds
Concurrent:     [Task A ████████]                   →  8 seconds
                [Task B ████████]
```

Rust's ownership system prevents data races at **compile time** — fearless concurrency!

---

## 2. Creating Threads with spawn

```rust
use std::thread;
use std::time::Duration;

fn main() {
    // Spawn a new thread
    thread::spawn(|| {
        for i in 1..=5 {
            println!("  Spawned thread: {i}");
            thread::sleep(Duration::from_millis(100));
        }
    });

    // Main thread continues
    for i in 1..=3 {
        println!("Main thread: {i}");
        thread::sleep(Duration::from_millis(150));
    }

    println!("Main thread finished");
    // ⚠️ Spawned thread may not finish! It dies when main exits.
}
```

---

## 3. JoinHandle — Waiting for Threads

`join()` waits for the thread to complete:

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..=5 {
            println!("  Worker: {i}");
            thread::sleep(Duration::from_millis(100));
        }
        println!("  Worker done!");
    });

    println!("Main: doing other work...");
    thread::sleep(Duration::from_millis(200));

    // Wait for the thread to finish
    handle.join().unwrap();
    println!("Main: worker finished, exiting.");
}
```

### Multiple threads:

```rust
use std::thread;

fn main() {
    let mut handles = vec![];

    for i in 0..5 {
        let handle = thread::spawn(move || {
            println!("Thread {i} starting");
            thread::sleep(std::time::Duration::from_millis(100));
            println!("Thread {i} done");
        });
        handles.push(handle);
    }

    // Wait for ALL threads to finish
    for handle in handles {
        handle.join().unwrap();
    }

    println!("All threads completed!");
}
```

---

## 4. move Closures with Threads

Threads need to **own** their data — use `move`:

```rust
use std::thread;

fn main() {
    let name = String::from("Alice");

    // ❌ Without move — won't compile
    // thread::spawn(|| println!("{name}"));
    // Error: closure may outlive the current function

    // ✅ With move — transfers ownership
    let handle = thread::spawn(move || {
        println!("Hello from thread, {name}!");
    });

    // println!("{name}");  // ❌ name was moved into the thread
    handle.join().unwrap();
}
```

### Sharing read-only data with Arc:

```rust
use std::thread;
use std::sync::Arc;

fn main() {
    let data = Arc::new(vec![1, 2, 3, 4, 5]);

    let mut handles = vec![];
    for i in 0..3 {
        let data = Arc::clone(&data);  // cheap clone (ref count)
        handles.push(thread::spawn(move || {
            let sum: i32 = data.iter().sum();
            println!("Thread {i}: sum = {sum}");
        }));
    }

    for h in handles { h.join().unwrap(); }
}
```

---

## 5. Returning Values from Threads

The closure's return value comes back through `join()`:

```rust
use std::thread;

fn main() {
    let handle = thread::spawn(|| {
        let mut sum = 0;
        for i in 1..=100 {
            sum += i;
        }
        sum  // return value
    });

    let result = handle.join().unwrap();
    println!("Sum 1..100 = {result}");  // 5050
}
```

### Parallel computation:

```rust
use std::thread;

fn main() {
    let numbers: Vec<i32> = (1..=1_000_000).collect();
    let chunk_size = numbers.len() / 4;

    let mut handles = vec![];

    for chunk in numbers.chunks(chunk_size) {
        let chunk = chunk.to_vec();  // own the data
        handles.push(thread::spawn(move || {
            chunk.iter().sum::<i32>()
        }));
    }

    let total: i32 = handles.into_iter()
        .map(|h| h.join().unwrap())
        .sum();

    println!("Total: {total}");
}
```

---

## 6. Thread Safety: Send and Sync

Rust uses two marker traits:

| Trait | Meaning |
|---|---|
| `Send` | Type can be **transferred** to another thread |
| `Sync` | Type can be **shared** (via `&T`) across threads |

```rust
// ✅ Most types are Send + Sync
// i32, String, Vec<T>, Box<T>, Arc<T> — all Send + Sync

// ❌ NOT Send/Sync:
// Rc<T>      — not thread-safe reference counting
// RefCell<T> — not thread-safe borrow checking
// *mut T     — raw pointers

use std::rc::Rc;

fn main() {
    let data = Rc::new(42);
    // thread::spawn(move || println!("{data}"));
    // ❌ Error: Rc<i32> cannot be sent between threads safely
}
```

---

## 7. Scoped Threads

Since Rust 1.63, scoped threads let you borrow data without `move`:

```rust
use std::thread;

fn main() {
    let mut data = vec![1, 2, 3, 4, 5];

    thread::scope(|s| {
        // Scoped thread can BORROW from outer scope!
        s.spawn(|| {
            println!("Thread 1: {:?}", &data[0..3]);
        });

        s.spawn(|| {
            println!("Thread 2: {:?}", &data[3..]);
        });

        // All scoped threads are joined when scope ends
    });
    // Guaranteed: all threads are done here

    // Can use data again — it was only borrowed
    data.push(6);
    println!("After: {:?}", data);
}
```

### Scoped threads with mutable access:

```rust
use std::thread;

fn main() {
    let mut results = vec![0; 4];

    thread::scope(|s| {
        for (i, slot) in results.iter_mut().enumerate() {
            s.spawn(move || {
                *slot = (i + 1) * 10;  // each thread writes to its own slot
            });
        }
    });

    println!("{:?}", results);  // [10, 20, 30, 40]
}
```

---

## 8. Thread Builder

Customize thread names and stack sizes:

```rust
use std::thread;

fn main() {
    let builder = thread::Builder::new()
        .name("worker-1".to_string())
        .stack_size(4 * 1024 * 1024);  // 4 MB stack

    let handle = builder.spawn(|| {
        let name = thread::current().name().unwrap_or("unnamed").to_string();
        println!("Running on thread: {name}");
    }).unwrap();

    handle.join().unwrap();

    // Check current thread info
    println!("Main thread: {:?}", thread::current().name());
    println!("Available parallelism: {:?}", thread::available_parallelism());
}
```

---

## 9. Real-World Example: Parallel Word Count

The roadmap practice task:

```rust
use std::thread;
use std::collections::HashMap;
use std::sync::Arc;

fn word_count(text: &str) -> HashMap<String, usize> {
    let mut counts = HashMap::new();
    for word in text.split_whitespace() {
        let word = word.to_lowercase();
        let word = word.trim_matches(|c: char| !c.is_alphanumeric());
        if !word.is_empty() {
            *counts.entry(word.to_string()).or_insert(0) += 1;
        }
    }
    counts
}

fn merge_counts(a: &mut HashMap<String, usize>, b: HashMap<String, usize>) {
    for (word, count) in b {
        *a.entry(word).or_insert(0) += count;
    }
}

fn main() {
    let documents = Arc::new(vec![
        "the quick brown fox jumps over the lazy dog",
        "the fox is quick and the dog is lazy",
        "rust is fast and rust is safe",
        "the quick rust fox is the best fox",
    ]);

    let mut handles = vec![];

    for i in 0..documents.len() {
        let docs = Arc::clone(&documents);
        handles.push(thread::spawn(move || {
            println!("Thread {i}: counting words...");
            word_count(docs[i])
        }));
    }

    // Merge all results
    let mut total_counts = HashMap::new();
    for handle in handles {
        let counts = handle.join().unwrap();
        merge_counts(&mut total_counts, counts);
    }

    // Sort by count and display
    let mut sorted: Vec<_> = total_counts.iter().collect();
    sorted.sort_by(|a, b| b.1.cmp(a.1));

    println!("\n📊 Word Frequencies:");
    for (word, count) in sorted.iter().take(10) {
        println!("  {word:12} → {count}");
    }
}
```

---

## 10. Summary Cheat Sheet

```
CREATING THREADS
────────────────────────────────────────────────────────────
thread::spawn(|| { ... })        basic spawn
thread::spawn(move || { ... })   with owned data

JOINING
────────────────────────────────────────────────────────────
handle.join().unwrap()           wait for thread
Returns Result<T> with closure return value

SCOPED THREADS (Rust 1.63+)
────────────────────────────────────────────────────────────
thread::scope(|s| {
    s.spawn(|| { ... });         can borrow from outer scope
});                               all threads joined here

DATA SHARING
────────────────────────────────────────────────────────────
move || { ... }                  transfer ownership
Arc::clone(&data)                share read-only data
Arc<Mutex<T>>                    share mutable data (lesson 55)

THREAD SAFETY TRAITS
────────────────────────────────────────────────────────────
Send    type can move between threads
Sync    &T can be shared between threads
❌ Rc<T>, RefCell<T> — not thread-safe

UTILITIES
────────────────────────────────────────────────────────────
thread::sleep(Duration)          pause
thread::current().name()         thread name
thread::available_parallelism()  CPU core count
```

---

## What's Next?

**Lesson 55 — Channels (mpsc)** — Message passing between threads. Learn `mpsc::channel`, send/receive, multiple producers, and building concurrent pipelines.

## Further Reading
- [The Rust Book — Ch 16.1: Threads](https://doc.rust-lang.org/book/ch16-01-threads.html)
- [std::thread](https://doc.rust-lang.org/std/thread/index.html)

---

*Fearless concurrency: Rust's type system prevents data races at compile time! 🦀*
