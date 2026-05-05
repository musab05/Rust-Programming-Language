# ✅ Lesson 59 — Answers: Tokio Runtime (AS2)

---

## Section A

### A1
`#[tokio::main]` is a proc-macro that transforms `async fn main()` into a regular `fn main()` that creates a Tokio runtime and calls `block_on()` to run the async code. It sets up the multi-threaded scheduler, I/O driver, and timer.

### A2
- `join!` — runs all futures **concurrently** and waits for **ALL** to complete. Returns a tuple of all results.
- `select!` — runs all futures concurrently and completes when the **FIRST** one finishes. Cancels the rest.

### A3
- **`Send`**: The task may be moved between threads by the runtime's work-stealing scheduler.
- **`'static`**: The task might run after the current scope ends; it can't borrow local data.

---

## Section B

### A4
```rust
use tokio::time::{sleep, Duration, Instant};

async fn task_a() -> &'static str { sleep(Duration::from_millis(100)).await; "A" }
async fn task_b() -> &'static str { sleep(Duration::from_millis(200)).await; "B" }
async fn task_c() -> &'static str { sleep(Duration::from_millis(300)).await; "C" }

#[tokio::main]
async fn main() {
    let start = Instant::now();
    let (a, b, c) = tokio::join!(task_a(), task_b(), task_c());
    let elapsed = start.elapsed();

    println!("{a}, {b}, {c}");
    println!("Time: {:?}", elapsed);  // ~300ms, not 600ms
    assert!(elapsed.as_millis() < 400);
}
```

### A5
```rust
use tokio::time::{sleep, timeout, Duration};

async fn slow_operation() -> String {
    sleep(Duration::from_secs(2)).await;
    "Done".to_string()
}

#[tokio::main]
async fn main() {
    match timeout(Duration::from_millis(500), slow_operation()).await {
        Ok(result) => println!("Got: {result}"),
        Err(_) => println!("⏰ Operation timed out after 500ms"),
    }
}
```

### A6
```rust
use tokio::sync::mpsc;

#[tokio::main]
async fn main() {
    let (tx, mut rx) = mpsc::channel::<String>(32);

    for id in 0..2 {
        let tx = tx.clone();
        tokio::spawn(async move {
            for i in 0..3 {
                tx.send(format!("[P{id}] msg {i}")).await.unwrap();
            }
        });
    }
    drop(tx);

    while let Some(msg) = rx.recv().await {
        println!("Got: {msg}");
    }
    println!("All done");
}
```

---

## Section C

### A7
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | Default is multi-threaded; use `#[tokio::main(flavor = "current_thread")]` for single |
| 2 | **True** | Both spawn concurrent work; `tokio::spawn` creates lightweight tasks |
| 3 | **False** | `join!` waits for ALL tasks; use `try_join!` to stop on errors |
| 4 | **False** | `select!` finishes when the FIRST branch completes, cancels the rest |
| 5 | **False** | `tokio::time::sleep` yields to the runtime; it does NOT block the OS thread |
| 6 | **True** | `oneshot` is designed for one sender, one receiver, one message |

---

## 🏆 Lesson 59 Complete!

**Next up:** [Lesson 60 — Async I/O](../lesson_60_async_io/lesson_60_async_io.md) 🦀
