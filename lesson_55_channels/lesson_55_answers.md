# ✅ Lesson 55 — Answers: Channels (CC2)

---

## Section A

### A1
```
42
```
Single-threaded channel: send then receive on the same thread works fine.

### A2 — ❌ Won't compile
`tx.send(msg)` **moves** `msg` into the channel. The `println!("{msg}")` after it tries to use a moved value. Error: `value used after move`.

### A3
```
Err(RecvError)
```
Dropping the sender closes the channel. `recv()` returns `Err(RecvError)` immediately since no more messages will ever come.

---

## Section B

### A4
```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    for id in 0..3 {
        let tx = tx.clone();
        thread::spawn(move || {
            for i in 1..=3 {
                tx.send(format!("[Thread {id}] Log message {i}")).unwrap();
            }
        });
    }
    drop(tx);  // close channel when all producers finish

    for msg in rx {
        println!("{msg}");
    }
    println!("All logs received.");
}
```

### A5
```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (task_tx, task_rx) = mpsc::channel::<i32>();
    let (result_tx, result_rx) = mpsc::channel::<(i32, i32)>();
    let task_rx = std::sync::Arc::new(std::sync::Mutex::new(task_rx));

    // 3 workers
    for id in 0..3 {
        let task_rx = std::sync::Arc::clone(&task_rx);
        let result_tx = result_tx.clone();
        thread::spawn(move || {
            loop {
                let task = task_rx.lock().unwrap().recv();
                match task {
                    Ok(n) => {
                        result_tx.send((n, n * n)).unwrap();
                    }
                    Err(_) => break,
                }
            }
        });
    }
    drop(result_tx);

    // Send 10 tasks
    for i in 1..=10 { task_tx.send(i).unwrap(); }
    drop(task_tx);

    // Collect results
    let mut results: Vec<_> = result_rx.into_iter().collect();
    results.sort();
    for (n, sq) in &results { println!("{n}² = {sq}"); }
}
```

### A6
```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx_a, rx_a) = mpsc::channel::<String>();
    let (tx_b, rx_b) = mpsc::channel::<String>();

    // Thread A: sends first, then receives
    let h1 = thread::spawn(move || {
        for i in 0..5 {
            let msg = format!("ping {i}");
            println!("A sends: {msg}");
            tx_a.send(msg).unwrap();
            let reply = rx_b.recv().unwrap();
            println!("A got: {reply}");
        }
    });

    // Thread B: receives first, then sends
    let h2 = thread::spawn(move || {
        for _ in 0..5 {
            let msg = rx_a.recv().unwrap();
            println!("  B got: {msg}");
            let reply = msg.replace("ping", "pong");
            println!("  B sends: {reply}");
            tx_b.send(reply).unwrap();
        }
    });

    h1.join().unwrap();
    h2.join().unwrap();
}
```

---

## Section C

### A7
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | `mpsc` = Multiple Producer, Single Consumer |
| 2 | **False** | `send()` **moves** the value, transferring ownership |
| 3 | **True** | `recv()` blocks until a message arrives or all senders are dropped |
| 4 | **True** | When all `Sender` instances are dropped, the channel closes |
| 5 | **False** | `mpsc` has exactly one receiver — it's not clonable |
| 6 | **True** | A zero-capacity sync_channel acts as a rendezvous point |
| 7 | **True** | The iterator yields messages until the channel closes |

### A8
- **`channel()`** (unbounded): `send()` never blocks. The internal buffer grows as needed. Best when the producer is fast and you don't want it to wait. Risk: unbounded memory growth if consumer is slow.
- **`sync_channel(n)`** (bounded): `send()` blocks when the buffer is full (n messages). Provides **backpressure** — slows down the producer when the consumer can't keep up. Use when you need to control memory usage or balance producer/consumer speeds.

---

## 🏆 Lesson 55 Complete!

✅ Channel creation (bounded and unbounded)  
✅ Send/receive with ownership transfer  
✅ Multiple producers  
✅ Channel iteration and closing  
✅ Worker pools and pipelines  

**Next up:** Lesson 56 — Shared State: Mutex & RwLock 🦀
