# 📘 Lesson 55 — Channels: mpsc (CC2)

> **Series:** Rust From Zero · Intermediate Level  
> **Roadmap ID:** CC2 · Category: ⚡ Concurrency  
> **Previous:** [Lesson 54 — Threads & spawn](../lesson_54_threads/lesson_54_threads.md)  
> **Next:** [Lesson 56 — Shared State: Mutex & RwLock](../lesson_56_mutex_rwlock/lesson_56_mutex_rwlock.md)  
> **Practice:** [Questions](./lesson_55_questions.md) · [Answers](./lesson_55_answers.md)  
> **Practice Task:** Build a multi-producer logging system

---

## Table of Contents

1. [Message Passing Concurrency](#1-message-passing-concurrency)
2. [Creating Channels](#2-creating-channels)
3. [Sending and Receiving](#3-sending-and-receiving)
4. [Ownership and Channels](#4-ownership-and-channels)
5. [Multiple Messages](#5-multiple-messages)
6. [Multiple Producers](#6-multiple-producers)
7. [Channel Patterns](#7-channel-patterns)
8. [sync_channel — Bounded Channels](#8-sync_channel--bounded-channels)
9. [Real-World Example: Logging Pipeline](#9-real-world-example-logging-pipeline)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. Message Passing Concurrency

Instead of sharing memory, threads communicate by **sending messages**:

> "Do not communicate by sharing memory; share memory by communicating."
> — Go proverb (Rust supports both!)

```
Thread A ──[message]──→ Channel ──[message]──→ Thread B
                 (Sender)    (Receiver)
```

Rust's `mpsc` = **Multiple Producer, Single Consumer**.

---

## 2. Creating Channels

```rust
use std::sync::mpsc;

fn main() {
    // Create a channel — returns (Sender, Receiver)
    let (tx, rx) = mpsc::channel();

    // tx = transmitter (sender)
    // rx = receiver

    tx.send("hello").unwrap();
    let msg = rx.recv().unwrap();
    println!("Received: {msg}");
}
```

---

## 3. Sending and Receiving

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    // Sender goes to a new thread
    thread::spawn(move || {
        let message = String::from("Hello from thread!");
        tx.send(message).unwrap();
        // println!("{message}");  // ❌ message was moved into send()
    });

    // Receiver waits on the main thread
    let received = rx.recv().unwrap();
    println!("Got: {received}");
}
```

### Receiving methods:

| Method | Behavior |
|---|---|
| `rx.recv()` | Blocks until a message arrives (or channel closes) |
| `rx.try_recv()` | Returns immediately with `Ok(msg)` or `Err(TryRecvError)` |
| `rx.recv_timeout(dur)` | Blocks for at most `dur` duration |

```rust
use std::sync::mpsc;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        thread::sleep(Duration::from_millis(500));
        tx.send("delayed").unwrap();
    });

    // Non-blocking check
    match rx.try_recv() {
        Ok(msg) => println!("Got: {msg}"),
        Err(_) => println!("Nothing yet..."),
    }

    // Blocking with timeout
    match rx.recv_timeout(Duration::from_secs(2)) {
        Ok(msg) => println!("Got: {msg}"),
        Err(_) => println!("Timed out!"),
    }
}
```

---

## 4. Ownership and Channels

`send()` **moves** the value — ensuring only one thread owns it:

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let data = vec![1, 2, 3, 4, 5];
        tx.send(data).unwrap();
        // data is MOVED — thread can't use it anymore
        // println!("{:?}", data);  // ❌ value used after move
    });

    let received = rx.recv().unwrap();
    println!("Got: {:?}", received);  // [1, 2, 3, 4, 5]
    // Main thread now owns the Vec
}
```

---

## 5. Multiple Messages

Use `rx` as an iterator to receive until the channel closes:

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let messages = vec![
            "Hello",
            "from",
            "the",
            "thread!",
        ];

        for msg in messages {
            tx.send(msg.to_string()).unwrap();
            thread::sleep(Duration::from_millis(200));
        }
        // tx is dropped here → channel closes
    });

    // Iterate over received messages
    for received in rx {
        println!("Got: {received}");
    }
    // Loop ends when sender is dropped (channel closed)

    println!("Channel closed, all messages received.");
}
```

---

## 6. Multiple Producers

Clone the sender for multiple producers:

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();

    // Producer 1
    let tx1 = tx.clone();
    thread::spawn(move || {
        for i in 1..=3 {
            tx1.send(format!("[P1] message {i}")).unwrap();
            thread::sleep(Duration::from_millis(100));
        }
    });

    // Producer 2
    let tx2 = tx.clone();
    thread::spawn(move || {
        for i in 1..=3 {
            tx2.send(format!("[P2] message {i}")).unwrap();
            thread::sleep(Duration::from_millis(150));
        }
    });

    // Drop the original sender (we cloned it for each thread)
    drop(tx);

    // Single consumer
    for msg in rx {
        println!("Received: {msg}");
    }
    println!("All producers done.");
}
```

---

## 7. Channel Patterns

### Worker pool:

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    // Spawn 4 workers, give each a cloned sender
    let mut handles = vec![];
    for id in 0..4 {
        let tx = tx.clone();
        handles.push(thread::spawn(move || {
            let result = id * id;
            tx.send((id, result)).unwrap();
        }));
    }
    drop(tx);  // drop original sender

    // Collect results
    let mut results: Vec<(i32, i32)> = rx.into_iter().collect();
    results.sort();
    for (id, result) in &results {
        println!("Worker {id}: {result}");
    }

    for h in handles { h.join().unwrap(); }
}
```

### Request-response:

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (request_tx, request_rx) = mpsc::channel::<(String, mpsc::Sender<String>)>();

    // Server thread
    thread::spawn(move || {
        for (request, response_tx) in request_rx {
            let response = format!("Processed: {request}");
            response_tx.send(response).unwrap();
        }
    });

    // Client sends request and gets response
    let (resp_tx, resp_rx) = mpsc::channel();
    request_tx.send(("hello".to_string(), resp_tx)).unwrap();

    let response = resp_rx.recv().unwrap();
    println!("{response}");  // Processed: hello
}
```

---

## 8. sync_channel — Bounded Channels

`mpsc::channel()` is unbounded (infinite buffer). `sync_channel` has a fixed capacity:

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    // Buffer size of 2 — send blocks when full
    let (tx, rx) = mpsc::sync_channel(2);

    thread::spawn(move || {
        for i in 1..=5 {
            println!("Sending {i}...");
            tx.send(i).unwrap();  // blocks when buffer is full
            println!("Sent {i}");
        }
    });

    thread::sleep(Duration::from_millis(500));

    for msg in rx {
        println!("  Received: {msg}");
        thread::sleep(Duration::from_millis(200));
    }
}
```

| | `channel()` | `sync_channel(n)` |
|---|---|---|
| Buffer | Unbounded | Fixed size n |
| `send()` | Never blocks | Blocks when full |
| Memory | Can grow | Bounded |
| Use case | Fast producers | Backpressure control |

---

## 9. Real-World Example: Logging Pipeline

The roadmap practice task:

```rust
use std::sync::mpsc;
use std::thread;
use std::time::{Duration, Instant};

#[derive(Debug)]
enum LogLevel { Info, Warn, Error }

#[derive(Debug)]
struct LogMessage {
    level: LogLevel,
    source: String,
    message: String,
    timestamp_ms: u128,
}

fn main() {
    let (tx, rx) = mpsc::channel::<LogMessage>();
    let start = Instant::now();

    // Producer 1: Web server
    let tx1 = tx.clone();
    let start1 = start;
    thread::spawn(move || {
        let events = vec![
            (LogLevel::Info, "Request received: GET /api/users"),
            (LogLevel::Info, "Response sent: 200 OK"),
            (LogLevel::Warn, "Slow query detected: 230ms"),
        ];
        for (level, msg) in events {
            tx1.send(LogMessage {
                level,
                source: "web".into(),
                message: msg.into(),
                timestamp_ms: start1.elapsed().as_millis(),
            }).unwrap();
            thread::sleep(Duration::from_millis(50));
        }
    });

    // Producer 2: Database
    let tx2 = tx.clone();
    let start2 = start;
    thread::spawn(move || {
        let events = vec![
            (LogLevel::Info, "Connection pool initialized"),
            (LogLevel::Error, "Query timeout: SELECT * FROM orders"),
            (LogLevel::Info, "Connection pool recycled"),
        ];
        for (level, msg) in events {
            tx2.send(LogMessage {
                level,
                source: "db".into(),
                message: msg.into(),
                timestamp_ms: start2.elapsed().as_millis(),
            }).unwrap();
            thread::sleep(Duration::from_millis(80));
        }
    });

    // Producer 3: Cache
    let tx3 = tx.clone();
    let start3 = start;
    thread::spawn(move || {
        let events = vec![
            (LogLevel::Info, "Cache hit: user:123"),
            (LogLevel::Warn, "Cache miss rate above 50%"),
        ];
        for (level, msg) in events {
            tx3.send(LogMessage {
                level,
                source: "cache".into(),
                message: msg.into(),
                timestamp_ms: start3.elapsed().as_millis(),
            }).unwrap();
            thread::sleep(Duration::from_millis(60));
        }
    });

    // Drop original sender so channel closes when all producers finish
    drop(tx);

    // Consumer: Logger
    println!("{:>6} {:>5} {:>8} {}", "ms", "LEVEL", "SOURCE", "MESSAGE");
    println!("{}", "-".repeat(60));

    for log in rx {
        let level = match log.level {
            LogLevel::Info => "INFO",
            LogLevel::Warn => "WARN",
            LogLevel::Error => "ERROR",
        };
        println!("{:>6} {:>5} {:>8} {}",
            log.timestamp_ms, level, log.source, log.message);
    }

    println!("\n✅ All producers finished. Log stream closed.");
}
```

---

## 10. Summary Cheat Sheet

```
CREATING CHANNELS
────────────────────────────────────────────────────────────
let (tx, rx) = mpsc::channel();         unbounded
let (tx, rx) = mpsc::sync_channel(n);   bounded (n capacity)

SENDING
────────────────────────────────────────────────────────────
tx.send(value).unwrap()     moves value to receiver
tx.clone()                  create another producer

RECEIVING
────────────────────────────────────────────────────────────
rx.recv()                   blocks until message/close
rx.try_recv()               non-blocking (returns Result)
rx.recv_timeout(dur)        blocks up to dur
for msg in rx { ... }       iterate until channel closes

MULTIPLE PRODUCERS
────────────────────────────────────────────────────────────
let tx2 = tx.clone();       clone sender for each thread
drop(tx);                   drop original after cloning

CHANNEL CLOSES WHEN
────────────────────────────────────────────────────────────
All senders are dropped → rx.recv() returns Err
                        → for loop ends

PATTERNS
────────────────────────────────────────────────────────────
Fan-in:     multiple senders → one receiver
Worker pool: one sender → work items → collect results
Pipeline:    tx1→rx1 | tx2→rx2  chained processing
```

---

## What's Next?

**Lesson 56 — Shared State: Mutex & RwLock** — Protect shared data with locks. Learn `Mutex`, `RwLock`, `Arc<Mutex<T>>`, and deadlock avoidance.

## Further Reading
- [The Rust Book — Ch 16.2: Message Passing](https://doc.rust-lang.org/book/ch16-02-message-passing.html)
- [std::sync::mpsc](https://doc.rust-lang.org/std/sync/mpsc/index.html)

---

*Channels: safe message passing between threads! 🦀*
