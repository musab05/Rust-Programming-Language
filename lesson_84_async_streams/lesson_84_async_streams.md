# 📘 Lesson 84 — Async Streams & Channels (CC8)

> **Series:** Rust From Zero · Advanced Level (Gap Fill)  
> **Roadmap ID:** CC8 · Category: ⚙️ Concurrency  
> **Previous:** [Lesson 83 — Cow\<T\>](../lesson_83_cow/lesson_83_cow.md)  
> **Next:** [Lesson 85 — Never Type (!)](../lesson_85_never_type/lesson_85_never_type.md)  
> **Practice:** [Questions](./lesson_84_questions.md) · [Answers](./lesson_84_answers.md)  
> **Practice Task:** Real-time log aggregator with multiple async producers

---

## Table of Contents

1. [Async Channels Overview](#1-async-channels-overview)
2. [tokio::sync::mpsc](#2-tokiosyncmpsc)
3. [tokio::sync::broadcast](#3-tokiosyncbroadcast)
4. [tokio::sync::watch](#4-tokiosyncwatch)
5. [tokio::sync::oneshot](#5-tokiosynconeshot)
6. [The Stream Trait](#6-the-stream-trait)
7. [Creating Streams](#7-creating-streams)
8. [Real-World: Log Aggregator](#8-real-world-log-aggregator)
9. [Choosing the Right Channel](#9-choosing-the-right-channel)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. Async Channels Overview

Tokio provides 4 channel types for async communication:

| Channel | Pattern | Use Case |
|---|---|---|
| `mpsc` | Many producers → one consumer | Task queues, pipelines |
| `broadcast` | One producer → many consumers | Event bus, pub/sub |
| `watch` | One producer → many consumers (latest only) | Config updates, state |
| `oneshot` | One send → one receive | Request/response, results |

---

## 2. tokio::sync::mpsc

Multiple producers, single consumer — the most common:

```rust
use tokio::sync::mpsc;

#[tokio::main]
async fn main() {
    // Bounded channel (capacity 32)
    let (tx, mut rx) = mpsc::channel::<String>(32);

    // Spawn 3 producers
    for id in 1..=3 {
        let tx = tx.clone();
        tokio::spawn(async move {
            for i in 1..=3 {
                let msg = format!("Worker {id}: message {i}");
                tx.send(msg).await.unwrap();
                tokio::time::sleep(tokio::time::Duration::from_millis(100 * id)).await;
            }
            println!("Worker {id} done");
        });
    }

    // Drop original sender so rx knows when all are done
    drop(tx);

    // Consumer
    while let Some(msg) = rx.recv().await {
        println!("Received: {msg}");
    }
    println!("All producers finished!");
}
```

### Unbounded channel:

```rust
use tokio::sync::mpsc;

#[tokio::main]
async fn main() {
    let (tx, mut rx) = mpsc::unbounded_channel::<i32>();

    tx.send(1).unwrap();  // never blocks — unbounded
    tx.send(2).unwrap();
    tx.send(3).unwrap();
    drop(tx);

    while let Some(val) = rx.recv().await {
        println!("{val}");
    }
}
```

---

## 3. tokio::sync::broadcast

One producer, many consumers — each consumer gets every message:

```rust
use tokio::sync::broadcast;

#[tokio::main]
async fn main() {
    let (tx, _) = broadcast::channel::<String>(16);

    // Create 3 subscribers
    let mut handles = vec![];
    for id in 1..=3 {
        let mut rx = tx.subscribe();
        handles.push(tokio::spawn(async move {
            while let Ok(msg) = rx.recv().await {
                println!("  Subscriber {id}: {msg}");
            }
        }));
    }

    // Publish messages
    for i in 1..=3 {
        tx.send(format!("Event #{i}")).unwrap();
        tokio::time::sleep(tokio::time::Duration::from_millis(50)).await;
    }

    drop(tx);  // close channel
    for h in handles { h.await.unwrap(); }
}
```

---

## 4. tokio::sync::watch

Single value that many consumers can observe — only the **latest** value matters:

```rust
use tokio::sync::watch;

#[derive(Debug, Clone)]
struct AppConfig {
    log_level: String,
    max_connections: u32,
}

#[tokio::main]
async fn main() {
    let initial = AppConfig { log_level: "info".into(), max_connections: 100 };
    let (tx, rx) = watch::channel(initial);

    // Spawn 2 watchers
    for id in 1..=2 {
        let mut rx = rx.clone();
        tokio::spawn(async move {
            while rx.changed().await.is_ok() {
                let config = rx.borrow();
                println!("  Watcher {id}: level={}, max={}",
                    config.log_level, config.max_connections);
            }
        });
    }

    tokio::time::sleep(tokio::time::Duration::from_millis(50)).await;

    // Update config
    tx.send(AppConfig { log_level: "debug".into(), max_connections: 200 }).unwrap();
    tokio::time::sleep(tokio::time::Duration::from_millis(50)).await;

    tx.send(AppConfig { log_level: "warn".into(), max_connections: 50 }).unwrap();
    tokio::time::sleep(tokio::time::Duration::from_millis(50)).await;
}
```

---

## 5. tokio::sync::oneshot

Single message — perfect for request/response patterns:

```rust
use tokio::sync::oneshot;

async fn compute(input: i32, reply: oneshot::Sender<i32>) {
    let result = input * input;
    tokio::time::sleep(tokio::time::Duration::from_millis(100)).await;
    reply.send(result).unwrap();
}

#[tokio::main]
async fn main() {
    let (tx, rx) = oneshot::channel();

    tokio::spawn(compute(42, tx));

    let result = rx.await.unwrap();
    println!("Result: {result}");  // 1764
}
```

### With select:

```rust
use tokio::sync::oneshot;
use tokio::time::{timeout, Duration};

#[tokio::main]
async fn main() {
    let (tx, rx) = oneshot::channel::<String>();

    tokio::spawn(async move {
        tokio::time::sleep(Duration::from_secs(2)).await;
        tx.send("done".into()).ok();
    });

    match timeout(Duration::from_secs(1), rx).await {
        Ok(Ok(msg)) => println!("Got: {msg}"),
        Ok(Err(_)) => println!("Sender dropped"),
        Err(_) => println!("Timed out!"),
    }
}
```

---

## 6. The Stream Trait

Streams are async iterators — yield items over time:

```toml
[dependencies]
tokio-stream = "0.1"
futures = "0.3"
```

```rust
use tokio_stream::StreamExt;

#[tokio::main]
async fn main() {
    // Create a stream from an iterator
    let mut stream = tokio_stream::iter(vec![1, 2, 3, 4, 5]);

    // Consume like an async iterator
    while let Some(val) = stream.next().await {
        println!("Got: {val}");
    }
}
```

### Stream adaptors:

```rust
use tokio_stream::StreamExt;

#[tokio::main]
async fn main() {
    let stream = tokio_stream::iter(1..=10);

    let result: Vec<i32> = stream
        .filter(|x| x % 2 == 0)
        .map(|x| x * x)
        .take(3)
        .collect()
        .await;

    println!("{:?}", result);  // [4, 16, 36]
}
```

---

## 7. Creating Streams

### Interval stream:

```rust
use tokio::time::{interval, Duration};
use tokio_stream::wrappers::IntervalStream;
use tokio_stream::StreamExt;

#[tokio::main]
async fn main() {
    let mut stream = IntervalStream::new(interval(Duration::from_millis(200)))
        .take(5);

    let mut count = 0;
    while let Some(_tick) = stream.next().await {
        count += 1;
        println!("Tick #{count}");
    }
}
```

### Channel as stream:

```rust
use tokio::sync::mpsc;
use tokio_stream::wrappers::ReceiverStream;
use tokio_stream::StreamExt;

#[tokio::main]
async fn main() {
    let (tx, rx) = mpsc::channel::<String>(10);

    tokio::spawn(async move {
        for i in 1..=5 {
            tx.send(format!("item-{i}")).await.unwrap();
        }
    });

    let mut stream = ReceiverStream::new(rx);
    while let Some(item) = stream.next().await {
        println!("Stream: {item}");
    }
}
```

---

## 8. Real-World: Log Aggregator

```rust
use tokio::sync::mpsc;

#[derive(Debug)]
struct LogEntry {
    source: String,
    level: String,
    message: String,
}

async fn web_server_logs(tx: mpsc::Sender<LogEntry>) {
    let logs = vec!["GET /api/users 200", "POST /api/login 401", "GET /health 200"];
    for msg in logs {
        tx.send(LogEntry {
            source: "web".into(), level: "INFO".into(), message: msg.into(),
        }).await.unwrap();
        tokio::time::sleep(tokio::time::Duration::from_millis(100)).await;
    }
}

async fn db_logs(tx: mpsc::Sender<LogEntry>) {
    let logs = vec!["Query took 250ms", "Connection pool: 8/10", "Slow query detected"];
    for msg in logs {
        tx.send(LogEntry {
            source: "database".into(), level: "WARN".into(), message: msg.into(),
        }).await.unwrap();
        tokio::time::sleep(tokio::time::Duration::from_millis(150)).await;
    }
}

async fn auth_logs(tx: mpsc::Sender<LogEntry>) {
    let logs = vec!["Login: alice", "Failed login: bob", "Token refreshed: alice"];
    for msg in logs {
        tx.send(LogEntry {
            source: "auth".into(), level: "INFO".into(), message: msg.into(),
        }).await.unwrap();
        tokio::time::sleep(tokio::time::Duration::from_millis(120)).await;
    }
}

#[tokio::main]
async fn main() {
    let (tx, mut rx) = mpsc::channel::<LogEntry>(100);

    tokio::spawn(web_server_logs(tx.clone()));
    tokio::spawn(db_logs(tx.clone()));
    tokio::spawn(auth_logs(tx));

    // Aggregator
    let mut count = 0;
    while let Some(entry) = rx.recv().await {
        count += 1;
        println!("[{:>8}] [{:>4}] {}", entry.source, entry.level, entry.message);
    }
    println!("\nAggregated {count} log entries");
}
```

---

## 9. Choosing the Right Channel

```
                    ┌─────────────────────────────┐
                    │  How many senders/receivers? │
                    └──────┬──────────────────────┘
                           │
            ┌──────────────┼──────────────┐
            ▼              ▼              ▼
     One → One       Many → One      One → Many
     ┌────────┐      ┌────────┐      ┌──────────────┐
     │oneshot │      │ mpsc   │      │ Need ALL msgs?│
     └────────┘      └────────┘      └──┬───────────┘
                                        │
                                  ┌─────┼─────┐
                                  ▼           ▼
                              Yes            No (latest only)
                            ┌──────────┐   ┌───────┐
                            │broadcast │   │ watch  │
                            └──────────┘   └───────┘
```

---

## 10. Summary Cheat Sheet

```
CHANNELS
────────────────────────────────────────────────────────────
mpsc::channel(cap)       many→one, bounded
mpsc::unbounded_channel  many→one, unbounded
broadcast::channel(cap)  one→many, all get every msg
watch::channel(initial)  one→many, latest value only
oneshot::channel()       one→one, single value

STREAM
────────────────────────────────────────────────────────────
tokio_stream::iter(vec)             from iterator
ReceiverStream::new(rx)             from mpsc receiver
IntervalStream::new(interval)       periodic ticks
stream.next().await                 get next item
stream.filter().map().collect()     adaptor chain
```

---

## What's Next?

**Lesson 85 — Never Type (!)** — Diverging functions, the `!` type, and when functions never return.

## Further Reading
- [Tokio — Channels](https://tokio.rs/tokio/tutorial/channels)
- [tokio-stream](https://docs.rs/tokio-stream/)

---

*Async streams & channels: data flowing through your async world! 🦀*
