# 📘 Lesson 59 — Tokio Runtime (AS2)

> **Series:** Rust From Zero · Advanced Level  
> **Roadmap ID:** AS2 · Category: 🌐 Async  
> **Previous:** [Lesson 58 — Async/Await Basics](../lesson_58_async_await/lesson_58_async_await.md)  
> **Next:** [Lesson 60 — Async I/O](../lesson_60_async_io/lesson_60_async_io.md)  
> **Practice:** [Questions](./lesson_59_questions.md) · [Answers](./lesson_59_answers.md)  
> **Practice Task:** Build a concurrent task runner with tokio::join! and select!

---

## Table of Contents

1. [What Is Tokio?](#1-what-is-tokio)
2. [Setting Up Tokio](#2-setting-up-tokio)
3. [#\[tokio::main\]](#3-tokiomain)
4. [Spawning Tasks](#4-spawning-tasks)
5. [tokio::join! — Run Concurrently](#5-tokiojoin--run-concurrently)
6. [tokio::select! — Race Tasks](#6-tokioselect--race-tasks)
7. [Timers and Sleep](#7-timers-and-sleep)
8. [Channels in Tokio](#8-channels-in-tokio)
9. [Real-World Example: Concurrent Fetcher](#9-real-world-example-concurrent-fetcher)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. What Is Tokio?

**Tokio** is Rust's most popular async runtime. It provides:
- A multi-threaded task scheduler
- Async I/O (TCP, UDP, files)
- Timers and intervals
- Channels for async message passing
- Synchronization primitives

---

## 2. Setting Up Tokio

```toml
# Cargo.toml
[dependencies]
tokio = { version = "1", features = ["full"] }
```

Feature flags for smaller builds:

| Feature | Provides |
|---|---|
| `rt` | Basic runtime |
| `rt-multi-thread` | Multi-threaded runtime |
| `macros` | `#[tokio::main]`, `#[tokio::test]` |
| `time` | `sleep`, `interval`, `timeout` |
| `sync` | Async channels, mutexes |
| `io-util` | `AsyncReadExt`, `AsyncWriteExt` |
| `net` | TCP, UDP |
| `fs` | Async file I/O |
| `full` | Everything above |

---

## 3. #\[tokio::main\]

The `#[tokio::main]` macro sets up the runtime:

```rust
#[tokio::main]
async fn main() {
    println!("Hello from Tokio!");

    let result = fetch_data().await;
    println!("{result}");
}

async fn fetch_data() -> String {
    tokio::time::sleep(std::time::Duration::from_millis(100)).await;
    "Data loaded!".to_string()
}
```

### What the macro expands to:

```rust
// #[tokio::main] expands roughly to:
fn main() {
    tokio::runtime::Runtime::new()
        .unwrap()
        .block_on(async {
            println!("Hello from Tokio!");
            let result = fetch_data().await;
            println!("{result}");
        });
}
```

### Manual runtime creation:

```rust
fn main() {
    let rt = tokio::runtime::Builder::new_multi_thread()
        .worker_threads(4)
        .enable_all()
        .build()
        .unwrap();

    rt.block_on(async {
        println!("Running on custom runtime");
    });
}
```

---

## 4. Spawning Tasks

`tokio::spawn` creates a new async task (like `thread::spawn` but lightweight):

```rust
use tokio::time::{sleep, Duration};

#[tokio::main]
async fn main() {
    // Spawn a background task
    let handle = tokio::spawn(async {
        sleep(Duration::from_millis(100)).await;
        42
    });

    // Do other work while task runs
    println!("Task spawned, doing other work...");

    // Wait for the result
    let result = handle.await.unwrap();
    println!("Task returned: {result}");
}
```

### Many concurrent tasks:

```rust
use tokio::time::{sleep, Duration};

#[tokio::main]
async fn main() {
    let mut handles = vec![];

    for i in 0..10 {
        handles.push(tokio::spawn(async move {
            sleep(Duration::from_millis(100)).await;
            i * i
        }));
    }

    let mut results = vec![];
    for handle in handles {
        results.push(handle.await.unwrap());
    }

    println!("Results: {:?}", results);  // [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
}
```

### Task requirements:
- The future must be `Send + 'static`
- Can't borrow from the outer scope (use `move` or `Arc`)

---

## 5. tokio::join! — Run Concurrently

Execute multiple futures **concurrently** and wait for all:

```rust
use tokio::time::{sleep, Duration};

async fn fetch_users() -> Vec<String> {
    sleep(Duration::from_millis(200)).await;
    vec!["Alice".into(), "Bob".into()]
}

async fn fetch_posts() -> Vec<String> {
    sleep(Duration::from_millis(300)).await;
    vec!["Post 1".into(), "Post 2".into()]
}

async fn fetch_comments() -> Vec<String> {
    sleep(Duration::from_millis(100)).await;
    vec!["Comment 1".into()]
}

#[tokio::main]
async fn main() {
    // Sequential: 200 + 300 + 100 = 600ms
    // let users = fetch_users().await;
    // let posts = fetch_posts().await;
    // let comments = fetch_comments().await;

    // Concurrent: max(200, 300, 100) = 300ms ← much faster!
    let (users, posts, comments) = tokio::join!(
        fetch_users(),
        fetch_posts(),
        fetch_comments(),
    );

    println!("Users: {:?}", users);
    println!("Posts: {:?}", posts);
    println!("Comments: {:?}", comments);
}
```

### try_join! — stops on first error:

```rust
use tokio::try_join;

async fn fetch_a() -> Result<String, String> { Ok("A".into()) }
async fn fetch_b() -> Result<String, String> { Err("B failed".into()) }

#[tokio::main]
async fn main() {
    match try_join!(fetch_a(), fetch_b()) {
        Ok((a, b)) => println!("{a}, {b}"),
        Err(e) => println!("Error: {e}"),  // Error: B failed
    }
}
```

---

## 6. tokio::select! — Race Tasks

Execute the **first** future to complete, cancel the rest:

```rust
use tokio::time::{sleep, Duration};

async fn fast() -> &'static str {
    sleep(Duration::from_millis(100)).await;
    "fast finished"
}

async fn slow() -> &'static str {
    sleep(Duration::from_millis(500)).await;
    "slow finished"
}

#[tokio::main]
async fn main() {
    tokio::select! {
        result = fast() => println!("Fast: {result}"),
        result = slow() => println!("Slow: {result}"),
    }
    // Only prints: "Fast: fast finished"
    // slow is cancelled!
}
```

### Timeout pattern:

```rust
use tokio::time::{sleep, Duration};

async fn long_operation() -> String {
    sleep(Duration::from_secs(5)).await;
    "Done".to_string()
}

#[tokio::main]
async fn main() {
    tokio::select! {
        result = long_operation() => println!("Got: {result}"),
        _ = sleep(Duration::from_secs(2)) => println!("⏰ Timed out!"),
    }
}
```

---

## 7. Timers and Sleep

```rust
use tokio::time::{sleep, interval, timeout, Duration, Instant};

#[tokio::main]
async fn main() {
    // Sleep — pause the current task
    println!("Sleeping...");
    sleep(Duration::from_millis(500)).await;
    println!("Awake!");

    // Interval — repeated timer
    let mut ticker = interval(Duration::from_millis(200));
    for i in 0..5 {
        ticker.tick().await;
        println!("Tick {i}");
    }

    // Timeout — limit execution time
    let result = timeout(Duration::from_secs(1), async {
        sleep(Duration::from_millis(500)).await;
        "completed"
    }).await;

    match result {
        Ok(val) => println!("Got: {val}"),
        Err(_) => println!("Timed out!"),
    }

    // Instant — measure elapsed time
    let start = Instant::now();
    sleep(Duration::from_millis(250)).await;
    println!("Elapsed: {:?}", start.elapsed());
}
```

---

## 8. Channels in Tokio

Tokio provides async-aware channels:

```rust
use tokio::sync::mpsc;

#[tokio::main]
async fn main() {
    // Bounded channel (capacity 32)
    let (tx, mut rx) = mpsc::channel::<String>(32);

    // Producer task
    let tx1 = tx.clone();
    tokio::spawn(async move {
        for i in 0..5 {
            tx1.send(format!("Message {i}")).await.unwrap();
        }
    });

    drop(tx);  // drop original sender

    // Consumer
    while let Some(msg) = rx.recv().await {
        println!("Got: {msg}");
    }
    println!("Channel closed");
}
```

### Other Tokio channel types:

| Type | Description |
|---|---|
| `mpsc` | Multi-producer, single consumer |
| `oneshot` | Single message, single use (req/response) |
| `broadcast` | Multi-producer, multi-consumer |
| `watch` | Single value, latest-value semantics |

```rust
use tokio::sync::oneshot;

#[tokio::main]
async fn main() {
    let (tx, rx) = oneshot::channel();

    tokio::spawn(async move {
        tx.send("result".to_string()).unwrap();
    });

    let result = rx.await.unwrap();
    println!("Got: {result}");
}
```

---

## 9. Real-World Example: Concurrent Fetcher

```rust
use tokio::time::{sleep, Duration, Instant};

async fn fetch_page(url: &str) -> Result<String, String> {
    let delay = match url {
        u if u.contains("fast") => 100,
        u if u.contains("medium") => 300,
        u if u.contains("slow") => 500,
        u if u.contains("error") => return Err(format!("Failed: {url}")),
        _ => 200,
    };
    sleep(Duration::from_millis(delay)).await;
    Ok(format!("Content from {url} ({delay}ms)"))
}

#[tokio::main]
async fn main() {
    let urls = vec![
        "https://fast.example.com",
        "https://medium.example.com",
        "https://slow.example.com",
        "https://fast2.example.com",
    ];

    // Sequential
    let start = Instant::now();
    for url in &urls {
        let result = fetch_page(url).await.unwrap();
        println!("[SEQ] {result}");
    }
    println!("Sequential: {:?}\n", start.elapsed());

    // Concurrent with join_all
    let start = Instant::now();
    let futures: Vec<_> = urls.iter().map(|url| fetch_page(url)).collect();
    let results = futures::future::join_all(futures).await;
    for result in results {
        println!("[PAR] {}", result.unwrap());
    }
    println!("Concurrent: {:?}", start.elapsed());
    // Concurrent is MUCH faster — max of delays instead of sum
}
```

> Note: Add `futures = "0.3"` to Cargo.toml for `join_all`.

---

## 10. Summary Cheat Sheet

```
SETUP
────────────────────────────────────────────────────────────
tokio = { version = "1", features = ["full"] }
#[tokio::main] async fn main() { }

SPAWNING TASKS
────────────────────────────────────────────────────────────
tokio::spawn(async { ... })     lightweight async task
handle.await.unwrap()            get result

CONCURRENT EXECUTION
────────────────────────────────────────────────────────────
tokio::join!(a, b, c)           run all, wait for all
tokio::try_join!(a, b)          stop on first error
tokio::select! { ... }          first to finish wins

TIMERS
────────────────────────────────────────────────────────────
sleep(Duration)                  pause task
interval(Duration)               repeated timer
timeout(Duration, future)        time limit

CHANNELS
────────────────────────────────────────────────────────────
mpsc::channel(32)                bounded async channel
oneshot::channel()               single response
broadcast::channel(16)           fan-out
watch::channel(val)              latest value

TASK REQUIREMENTS
────────────────────────────────────────────────────────────
Future must be Send + 'static
Use move or Arc for shared data
```

---

## What's Next?

**Lesson 60 — Async I/O** — Read and write files, TCP streams, and HTTP requests asynchronously. Build a simple async TCP echo server.

## Further Reading
- [Tokio Tutorial](https://tokio.rs/tokio/tutorial)
- [tokio::main](https://docs.rs/tokio/latest/tokio/attr.main.html)
- [tokio::spawn](https://docs.rs/tokio/latest/tokio/fn.spawn.html)

---

*Tokio: the engine that powers async Rust! 🦀*
