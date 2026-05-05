# 📘 Lesson 58 — Async/Await Basics (AS1)

> **Series:** Rust From Zero · Advanced Level  
> **Roadmap ID:** AS1 · Category: 🌐 Async  
> **Previous:** [Lesson 57 — Send & Sync](../lesson_57_send_sync/lesson_57_send_sync.md)  
> **Next:** [Lesson 59 — Tokio Runtime](../lesson_59_tokio/lesson_59_tokio.md)  
> **Practice:** [Questions](./lesson_58_questions.md) · [Answers](./lesson_58_answers.md)  
> **Practice Task:** Write async functions and compose them with .await

---

## Table of Contents

1. [Why Async?](#1-why-async)
2. [async fn and .await](#2-async-fn-and-await)
3. [Futures — Lazy by Default](#3-futures--lazy-by-default)
4. [Threads vs Async](#4-threads-vs-async)
5. [The Future Trait](#5-the-future-trait)
6. [async Blocks](#6-async-blocks)
7. [Composing Async Functions](#7-composing-async-functions)
8. [Pin and Why It Exists](#8-pin-and-why-it-exists)
9. [Runtimes — You Need One](#9-runtimes--you-need-one)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. Why Async?

Async is for **I/O-bound** workloads — tasks that spend most time **waiting**:

```
Threads (1 thread per task):
Thread 1: [work][───wait───][work]
Thread 2: [work][──wait──][work]
Thread 3: [work][────wait────][work]
→ 3 OS threads, mostly idle (expensive!)

Async (1 thread, many tasks):
Task 1: [work]           [work]
Task 2:       [work]          [work]
Task 3:            [work]          [work]
→ 1 OS thread, no idle time (efficient!)
```

| | Threads | Async |
|---|---|---|
| Best for | CPU-bound work | I/O-bound work |
| Cost per task | ~8KB stack + OS overhead | ~few hundred bytes |
| Scheduling | OS scheduler | Rust runtime |
| Concurrent tasks | Hundreds | Millions |
| Example | Image processing | Web server, DB queries |

---

## 2. async fn and .await

```rust
// An async function returns a Future instead of running immediately
async fn fetch_data() -> String {
    // Simulate network delay
    // In real code: actual HTTP request, DB query, etc.
    String::from("data from server")
}

async fn process() {
    // .await pauses this function until the future completes
    let data = fetch_data().await;
    println!("Got: {data}");
}

// NOTE: You need a runtime to actually run async code
// We'll use tokio in lesson 59. For now, understand the concepts.
```

### Key rules:
1. `async fn` returns `impl Future<Output = T>`
2. `.await` can only be used inside `async fn` or `async` blocks
3. Futures are **lazy** — they do nothing until awaited
4. You need an **async runtime** to execute futures

---

## 3. Futures — Lazy by Default

Unlike JavaScript promises, Rust futures don't start until polled:

```rust
async fn hello() {
    println!("Hello, async world!");
}

fn main() {
    let future = hello();  // creates the future, does NOT run it
    // println! has NOT been called yet!

    // Need a runtime to actually execute:
    // tokio::runtime::Runtime::new().unwrap().block_on(future);
    
    println!("Future created but not executed");
    // "Hello, async world!" was never printed!
}
```

### The polling model:

```
Runtime polls future → Future returns:
├── Poll::Pending  → "Not done yet, wake me when ready"
│                    Runtime works on other tasks
│                    ← Waker signals "try again"
│                    Runtime polls again
└── Poll::Ready(T) → "Done! Here's the result"
```

---

## 4. Threads vs Async

```rust
// THREADS: one thread per task
use std::thread;
use std::time::Duration;

fn thread_example() {
    let handles: Vec<_> = (0..5).map(|i| {
        thread::spawn(move || {
            thread::sleep(Duration::from_secs(1));
            format!("Thread {i} done")
        })
    }).collect();

    for h in handles {
        println!("{}", h.join().unwrap());
    }
}

// ASYNC: one thread, many tasks (conceptual — needs runtime)
async fn async_task(i: u32) -> String {
    // tokio::time::sleep(Duration::from_secs(1)).await;
    format!("Task {i} done")
}

async fn async_example() {
    // All tasks run concurrently on fewer threads
    let results = vec![
        async_task(0).await,
        async_task(1).await,
        async_task(2).await,
    ];
    for r in results { println!("{r}"); }
}
```

---

## 5. The Future Trait

Every async function returns a type implementing `Future`:

```rust
use std::future::Future;
use std::pin::Pin;
use std::task::{Context, Poll};

// The trait (simplified)
// trait Future {
//     type Output;
//     fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
// }

// Manual Future implementation (rare — usually use async fn)
struct CountDown {
    remaining: u32,
}

impl Future for CountDown {
    type Output = String;

    fn poll(mut self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<String> {
        if self.remaining == 0 {
            Poll::Ready("Liftoff!".to_string())
        } else {
            println!("  {}...", self.remaining);
            self.remaining -= 1;
            cx.waker().wake_by_ref();  // tell runtime to poll again
            Poll::Pending
        }
    }
}

// Usage: CountDown { remaining: 3 }.await → prints 3... 2... 1... "Liftoff!"
```

---

## 6. async Blocks

Create inline futures without defining an async function:

```rust
async fn example() {
    // async block — creates an anonymous future
    let future = async {
        let x = 10;
        let y = 20;
        x + y
    };

    let result = future.await;
    println!("Result: {result}");  // 30

    // async block with captured variables
    let name = String::from("Rust");
    let greeting = async {
        format!("Hello, {name}!")
    };
    println!("{}", greeting.await);

    // async move block — takes ownership
    let data = vec![1, 2, 3];
    let future = async move {
        data.iter().sum::<i32>()
    };
    // data was moved into the async block
    println!("Sum: {}", future.await);
}
```

---

## 7. Composing Async Functions

```rust
async fn fetch_user(id: u32) -> String {
    // Simulate DB query
    format!("User_{id}")
}

async fn fetch_posts(user: &str) -> Vec<String> {
    // Simulate API call
    vec![format!("{user}_post1"), format!("{user}_post2")]
}

async fn fetch_comments(post: &str) -> Vec<String> {
    vec![format!("{post}_comment1")]
}

// Sequential composition
async fn get_user_data(id: u32) -> (String, Vec<String>) {
    let user = fetch_user(id).await;           // wait for user
    let posts = fetch_posts(&user).await;      // then wait for posts
    (user, posts)
}

// The real power comes with concurrent execution (Lesson 59):
// let (user, posts) = tokio::join!(fetch_user(1), fetch_posts("User_1"));
```

---

## 8. Pin and Why It Exists

Async functions compile to state machines that may **self-reference**. `Pin` prevents the data from moving in memory:

```rust
// Conceptual — you rarely write Pin manually
use std::pin::Pin;

fn takes_pinned(fut: Pin<&mut dyn Future<Output = i32>>) {
    // fut cannot be moved — safe for self-referential futures
}
```

**When you encounter Pin:**
- `Pin<Box<dyn Future>>` — a heap-pinned, owned future
- `Pin<&mut F>` — a pinned reference to a future
- Most of the time, `async fn` and `.await` handle pinning for you

**Key takeaway:** Pin exists because async state machines may hold references to their own fields. Moving the struct would invalidate those references. Pin prevents that.

---

## 9. Runtimes — You Need One

Rust's standard library defines `Future` but provides **no runtime**. You choose:

| Runtime | Description |
|---|---|
| **tokio** | Most popular, full-featured, production-grade |
| **async-std** | Mirrors std API with async versions |
| **smol** | Minimal, lightweight |

```toml
# Cargo.toml
[dependencies]
tokio = { version = "1", features = ["full"] }
```

```rust
// Minimal tokio example
#[tokio::main]
async fn main() {
    let result = fetch_data().await;
    println!("{result}");
}

async fn fetch_data() -> String {
    tokio::time::sleep(std::time::Duration::from_millis(100)).await;
    "Data loaded!".to_string()
}
```

---

## 10. Summary Cheat Sheet

```
ASYNC FUNCTIONS
────────────────────────────────────────────────────────────
async fn f() -> T { ... }       returns impl Future<Output=T>
let result = f().await;          pause until future completes

ASYNC BLOCKS
────────────────────────────────────────────────────────────
let fut = async { expr };        inline future
let fut = async move { expr };   takes ownership of captures

FUTURE TRAIT
────────────────────────────────────────────────────────────
trait Future {
    type Output;
    fn poll(self: Pin<&mut Self>, cx: &mut Context) -> Poll<Output>;
}
Poll::Ready(val)  → done
Poll::Pending     → not ready, try again later

KEY CONCEPTS
────────────────────────────────────────────────────────────
Futures are LAZY      don't run until polled/awaited
.await is ONLY in     async fn or async blocks
Runtime required      tokio, async-std, or smol
Pin prevents          self-referential futures from moving

THREADS vs ASYNC
────────────────────────────────────────────────────────────
Threads  → CPU-bound, heavy tasks, OS-scheduled
Async    → I/O-bound, lightweight, runtime-scheduled
```

---

## What's Next?

**Lesson 59 — Tokio Runtime** — The most popular async runtime. Learn `#[tokio::main]`, spawning tasks, `tokio::join!`, `tokio::select!`, and timers.

## Further Reading
- [The Rust Async Book](https://rust-lang.github.io/async-book/)
- [Tokio Tutorial](https://tokio.rs/tokio/tutorial)
- [std::future::Future](https://doc.rust-lang.org/std/future/trait.Future.html)

---

*Async/await: doing more with less — millions of tasks on a single thread! 🦀*
