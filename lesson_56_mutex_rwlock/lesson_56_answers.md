# ✅ Lesson 56 — Answers: Mutex & RwLock (CC3)

---

## Section A

### A1
```
15
```
Lock acquired, value changed from 10 to 15, printed.

### A2 — 💥 Deadlock (hangs forever)
Single-threaded deadlock: `g1` holds the lock, then `g2` tries to acquire the same lock on the same thread — blocks forever.

### A3 — ❌ Won't compile
`m` is moved into the thread, so the main thread can't access it. Need `Arc<Mutex<i32>>` to share.

---

## Section B

### A4
```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        handles.push(thread::spawn(move || {
            for _ in 0..100 {
                *counter.lock().unwrap() += 1;
            }
        }));
    }

    for h in handles { h.join().unwrap(); }
    println!("Count: {}", *counter.lock().unwrap());  // 1000
}
```

### A5
```rust
use std::sync::{Arc, RwLock};
use std::collections::HashMap;
use std::thread;

fn main() {
    let config = Arc::new(RwLock::new(HashMap::from([
        ("theme".to_string(), "dark".to_string()),
        ("lang".to_string(), "en".to_string()),
    ])));
    let mut handles = vec![];

    for i in 0..5 {
        let config = Arc::clone(&config);
        handles.push(thread::spawn(move || {
            let r = config.read().unwrap();
            println!("Reader {i}: theme = {:?}", r.get("theme"));
        }));
    }

    let config_w = Arc::clone(&config);
    handles.push(thread::spawn(move || {
        let mut w = config_w.write().unwrap();
        w.insert("theme".to_string(), "light".to_string());
        println!("Writer: updated theme to light");
    }));

    for h in handles { h.join().unwrap(); }
    println!("Final: {:?}", config.read().unwrap());
}
```

### A6
```rust
use std::sync::atomic::{AtomicI32, Ordering};
use std::sync::Arc;
use std::thread;

fn main() {
    let counter = Arc::new(AtomicI32::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        handles.push(thread::spawn(move || {
            for _ in 0..100 {
                counter.fetch_add(1, Ordering::Relaxed);
            }
        }));
    }

    for h in handles { h.join().unwrap(); }
    println!("Count: {}", counter.load(Ordering::Relaxed));  // 1000
}
```

---

## Section C

### A7
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | `lock()` blocks the calling thread until the lock is available |
| 2 | **True** | `MutexGuard` implements `Drop` which releases the lock |
| 3 | **False** | `RwLock` allows multiple readers OR one writer, never multiple writers |
| 4 | **False** | You can recover data from a poisoned mutex via `into_inner()` |
| 5 | **True** | Atomics use hardware instructions; Mutex has OS overhead |
| 6 | **False** | Rust prevents data races, not deadlocks — deadlocks are logic errors |

### A8
`Mutex<T>` can't be shared across threads by itself because ownership would move to one thread. `Arc` (Atomic Reference Counted) provides shared ownership that's thread-safe. Together, `Arc` handles "who owns it" (everyone) and `Mutex` handles "who can access it right now" (one at a time).

---

## 🏆 Lesson 56 Complete!

**Next up:** [Lesson 57 — Send & Sync](../lesson_57_send_sync/lesson_57_send_sync.md) 🦀
