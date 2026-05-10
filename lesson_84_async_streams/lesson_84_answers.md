# ✅ Lesson 84 — Answers: Async Streams & Channels (CC8)

---

## Section A

### A1
| Channel | Senders | Receivers | Delivery |
|---|---|---|---|
| **mpsc** | Many | One | Each message → one consumer |
| **broadcast** | One (or clone) | Many | Each message → ALL consumers |
| **watch** | One | Many | Only latest value — consumers may skip |

### A2
Use `oneshot` when you need a single response — like a request/response pattern. `mpsc` is for ongoing streams of messages. `oneshot` is lighter and guarantees exactly one value.

---

## Section B

### A3
```rust
use tokio::sync::mpsc;
use tokio::time::{Duration, Instant};

#[tokio::main]
async fn main() {
    let (tx, mut rx) = mpsc::channel::<(String, String)>(100);
    let start = Instant::now();

    for name in ["web", "db", "auth"] {
        let tx = tx.clone();
        let name = name.to_string();
        tokio::spawn(async move {
            for i in 1..=3 {
                tx.send((name.clone(), format!("event {i}"))).await.unwrap();
                tokio::time::sleep(Duration::from_millis(80)).await;
            }
        });
    }
    drop(tx);

    while let Some((src, msg)) = rx.recv().await {
        println!("[{:>6.0?}] [{src:>4}] {msg}", start.elapsed());
    }
}
```

### A4
```rust
use tokio::sync::broadcast;

#[tokio::main]
async fn main() {
    let (tx, _) = broadcast::channel::<String>(16);
    let mut handles = vec![];

    for id in 1..=2 {
        let mut rx = tx.subscribe();
        handles.push(tokio::spawn(async move {
            while let Ok(msg) = rx.recv().await {
                println!("  Sub {id}: {msg}");
            }
        }));
    }

    for i in 1..=5 { tx.send(format!("Event #{i}")).unwrap(); }
    drop(tx);
    for h in handles { h.await.unwrap(); }
}
```

---

## Section C

### A5
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | mpsc = **m**ultiple **p**roducers, **s**ingle **c**onsumer |
| 2 | **False** | broadcast delivers each message to **ALL** subscribers |
| 3 | **True** | watch stores only the latest; missed updates are normal |
| 4 | **False** | oneshot can send exactly **one** value |
| 5 | **True** | `Stream` is the async version of `Iterator` |
| 6 | **True** | When all senders are dropped, the channel closes |

---

## 🏆 Lesson 84 Complete!

**Next up:** [Lesson 85 — Never Type (!)](../lesson_85_never_type/lesson_85_never_type.md) 🦀
