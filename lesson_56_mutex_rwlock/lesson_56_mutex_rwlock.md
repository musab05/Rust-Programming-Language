# 📘 Lesson 56 — Shared State: Mutex & RwLock (CC3)

> **Series:** Rust From Zero · Advanced Level begins!  
> **Roadmap ID:** CC3 · Category: ⚡ Concurrency  
> **Previous:** [Lesson 55 — Channels (mpsc)](../lesson_55_channels/lesson_55_channels.md)  
> **Next:** [Lesson 57 — Send & Sync](../lesson_57_send_sync/lesson_57_send_sync.md)  
> **Practice:** [Questions](./lesson_56_questions.md) · [Answers](./lesson_56_answers.md)  
> **Practice Task:** Build a thread-safe concurrent counter and bank account

---

## Table of Contents

1. [Shared State vs Message Passing](#1-shared-state-vs-message-passing)
2. [Mutex\<T\> — Mutual Exclusion](#2-mutext--mutual-exclusion)
3. [Arc\<Mutex\<T\>\> — Shared Mutable State](#3-arcmutext--shared-mutable-state)
4. [Lock Poisoning](#4-lock-poisoning)
5. [RwLock\<T\> — Reader-Writer Lock](#5-rwlockt--reader-writer-lock)
6. [Deadlocks](#6-deadlocks)
7. [Atomic Types](#7-atomic-types)
8. [Choosing the Right Tool](#8-choosing-the-right-tool)
9. [Real-World Example: Thread-Safe Bank](#9-real-world-example-thread-safe-bank)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. Shared State vs Message Passing

Two approaches to concurrency:

| | Message Passing (channels) | Shared State (mutex) |
|---|---|---|
| Model | Actors send messages | Threads share data |
| Safety | Ownership transferred | Locks protect data |
| Rust tools | `mpsc::channel` | `Mutex`, `RwLock` |
| Best for | Pipelines, producer/consumer | Shared counters, caches |

---

## 2. Mutex\<T\> — Mutual Exclusion

A `Mutex` wraps data and ensures only **one thread** accesses it at a time:

```rust
use std::sync::Mutex;

fn main() {
    let m = Mutex::new(5);

    {
        // .lock() acquires the lock and returns MutexGuard
        let mut num = m.lock().unwrap();
        *num = 10;
        println!("Inside lock: {num}");
        // MutexGuard dropped here → lock released
    }

    println!("After lock: {:?}", m.lock().unwrap());  // 10
}
```

### How it works:

```
Thread A calls m.lock()  → acquires lock, gets MutexGuard
Thread B calls m.lock()  → BLOCKS (waits for A to release)
Thread A's guard dropped → lock released
Thread B unblocks        → acquires lock, gets MutexGuard
```

---

## 3. Arc\<Mutex\<T\>\> — Shared Mutable State

`Arc` for shared ownership + `Mutex` for safe mutation:

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        handles.push(thread::spawn(move || {
            let mut num = counter.lock().unwrap();
            *num += 1;
        }));
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Final count: {}", *counter.lock().unwrap());  // 10
}
```

### Why both Arc AND Mutex?

```rust
// Rc<Mutex<T>>  → ❌ Rc isn't thread-safe
// Arc<T>        → ❌ Arc only gives &T (immutable)
// Mutex<T>      → ❌ Can't share Mutex across threads without Arc
// Arc<Mutex<T>> → ✅ Shared ownership + mutual exclusion
```

---

## 4. Lock Poisoning

If a thread panics while holding a lock, the mutex becomes **poisoned**:

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let data = Arc::new(Mutex::new(vec![1, 2, 3]));

    let data_clone = Arc::clone(&data);
    let handle = thread::spawn(move || {
        let mut lock = data_clone.lock().unwrap();
        lock.push(4);
        panic!("thread panicked while holding lock!");
    });

    let _ = handle.join();  // thread panicked

    // Lock is now poisoned!
    match data.lock() {
        Ok(val) => println!("Got: {:?}", *val),
        Err(poisoned) => {
            // Can still recover the data
            let val = poisoned.into_inner();
            println!("Recovered from poison: {:?}", *val);  // [1, 2, 3, 4]
        }
    }
}
```

---

## 5. RwLock\<T\> — Reader-Writer Lock

`RwLock` allows **multiple readers OR one writer**:

```rust
use std::sync::{Arc, RwLock};
use std::thread;

fn main() {
    let config = Arc::new(RwLock::new(String::from("initial")));
    let mut handles = vec![];

    // Spawn 5 readers
    for i in 0..5 {
        let config = Arc::clone(&config);
        handles.push(thread::spawn(move || {
            let val = config.read().unwrap();  // shared read lock
            println!("Reader {i}: {val}");
        }));
    }

    // Spawn 1 writer
    {
        let config = Arc::clone(&config);
        handles.push(thread::spawn(move || {
            let mut val = config.write().unwrap();  // exclusive write lock
            *val = "updated".to_string();
            println!("Writer: updated config");
        }));
    }

    for h in handles { h.join().unwrap(); }
    println!("Final: {}", config.read().unwrap());
}
```

### Mutex vs RwLock:

| | `Mutex<T>` | `RwLock<T>` |
|---|---|---|
| Readers | One at a time | Many simultaneous |
| Writers | One at a time | One at a time (exclusive) |
| Best when | Write-heavy | Read-heavy |
| Overhead | Lower | Higher (tracking readers) |

---

## 6. Deadlocks

Deadlocks occur when two threads wait for each other's locks:

```rust
// ⚠️ DEADLOCK EXAMPLE — don't do this!
use std::sync::{Arc, Mutex};
use std::thread;

fn deadlock_example() {
    let a = Arc::new(Mutex::new(1));
    let b = Arc::new(Mutex::new(2));

    let a1 = Arc::clone(&a);
    let b1 = Arc::clone(&b);
    let t1 = thread::spawn(move || {
        let _lock_a = a1.lock().unwrap();  // locks A
        thread::sleep(std::time::Duration::from_millis(100));
        let _lock_b = b1.lock().unwrap();  // waits for B ← DEADLOCK
    });

    let a2 = Arc::clone(&a);
    let b2 = Arc::clone(&b);
    let t2 = thread::spawn(move || {
        let _lock_b = b2.lock().unwrap();  // locks B
        thread::sleep(std::time::Duration::from_millis(100));
        let _lock_a = a2.lock().unwrap();  // waits for A ← DEADLOCK
    });

    // t1.join().unwrap();  // hangs forever!
    // t2.join().unwrap();
}
```

### Prevention strategies:

```rust
// 1. Always lock in the same order
let _a = mutex_a.lock().unwrap();
let _b = mutex_b.lock().unwrap();  // always A then B

// 2. Use try_lock with timeout
match mutex.try_lock() {
    Ok(guard) => { /* got lock */ }
    Err(_) => { /* couldn't acquire — try again later */ }
}

// 3. Minimize lock scope
{
    let mut data = mutex.lock().unwrap();
    *data += 1;
}  // lock released immediately

// 4. Prefer channels over shared state when possible
```

---

## 7. Atomic Types

For simple counters, atomics are faster than `Mutex`:

```rust
use std::sync::atomic::{AtomicU64, Ordering};
use std::sync::Arc;
use std::thread;

fn main() {
    let counter = Arc::new(AtomicU64::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        handles.push(thread::spawn(move || {
            for _ in 0..1000 {
                counter.fetch_add(1, Ordering::Relaxed);
            }
        }));
    }

    for h in handles { h.join().unwrap(); }
    println!("Count: {}", counter.load(Ordering::Relaxed));  // 10000
}
```

### Common atomic types:

| Type | For |
|---|---|
| `AtomicBool` | Flags, stop signals |
| `AtomicI32/U32` | Counters |
| `AtomicI64/U64` | Counters, IDs |
| `AtomicUsize` | Indexes, sizes |

---

## 8. Choosing the Right Tool

```
Do threads need shared mutable data?
├── No  → Use channels (mpsc)
├── Yes → Is it a simple counter/flag?
│   ├── Yes → Atomic types (fastest)
│   └── No  → Is it read-heavy?
│       ├── Yes → Arc<RwLock<T>>
│       └── No  → Arc<Mutex<T>>
```

---

## 9. Real-World Example: Thread-Safe Bank

```rust
use std::sync::{Arc, Mutex};
use std::thread;
use std::collections::HashMap;

#[derive(Debug)]
struct Bank {
    accounts: Mutex<HashMap<String, f64>>,
}

impl Bank {
    fn new() -> Self {
        Bank { accounts: Mutex::new(HashMap::new()) }
    }

    fn create_account(&self, name: &str, balance: f64) {
        self.accounts.lock().unwrap().insert(name.to_string(), balance);
    }

    fn deposit(&self, name: &str, amount: f64) -> Result<f64, String> {
        let mut accounts = self.accounts.lock().unwrap();
        match accounts.get_mut(name) {
            Some(balance) => {
                *balance += amount;
                Ok(*balance)
            }
            None => Err(format!("Account '{name}' not found")),
        }
    }

    fn withdraw(&self, name: &str, amount: f64) -> Result<f64, String> {
        let mut accounts = self.accounts.lock().unwrap();
        match accounts.get_mut(name) {
            Some(balance) => {
                if *balance >= amount {
                    *balance -= amount;
                    Ok(*balance)
                } else {
                    Err(format!("Insufficient funds: {:.2} < {:.2}", *balance, amount))
                }
            }
            None => Err(format!("Account '{name}' not found")),
        }
    }

    fn balance(&self, name: &str) -> Option<f64> {
        self.accounts.lock().unwrap().get(name).copied()
    }
}

fn main() {
    let bank = Arc::new(Bank::new());
    bank.create_account("Alice", 1000.0);
    bank.create_account("Bob", 500.0);

    let mut handles = vec![];

    // 5 threads deposit to Alice
    for i in 0..5 {
        let bank = Arc::clone(&bank);
        handles.push(thread::spawn(move || {
            bank.deposit("Alice", 100.0).unwrap();
            println!("Thread {i}: deposited $100 to Alice");
        }));
    }

    // 3 threads withdraw from Bob
    for i in 0..3 {
        let bank = Arc::clone(&bank);
        handles.push(thread::spawn(move || {
            match bank.withdraw("Bob", 100.0) {
                Ok(bal) => println!("Thread {i}: withdrew $100 from Bob, balance: ${bal:.2}"),
                Err(e) => println!("Thread {i}: {e}"),
            }
        }));
    }

    for h in handles { h.join().unwrap(); }

    println!("\n💰 Final Balances:");
    println!("  Alice: ${:.2}", bank.balance("Alice").unwrap());  // 1500
    println!("  Bob:   ${:.2}", bank.balance("Bob").unwrap());    // 200
}
```

---

## 10. Summary Cheat Sheet

```
MUTEX<T>
────────────────────────────────────────────────────────────
Mutex::new(val)          create
mutex.lock().unwrap()    acquire lock → MutexGuard
Guard auto-unlocks on drop
Arc<Mutex<T>> for sharing across threads

RWLOCK<T>
────────────────────────────────────────────────────────────
RwLock::new(val)         create
rwlock.read().unwrap()   shared read lock (many)
rwlock.write().unwrap()  exclusive write lock (one)
Arc<RwLock<T>> for sharing across threads

ATOMICS
────────────────────────────────────────────────────────────
AtomicU64::new(0)        create
.fetch_add(1, Ordering)  atomic increment
.load(Ordering)          atomic read
.store(val, Ordering)    atomic write
No lock needed — hardware-level atomic operations

DEADLOCK PREVENTION
────────────────────────────────────────────────────────────
Lock in consistent order
Use try_lock() for non-blocking
Minimize lock scope
Prefer channels when possible

CHOOSING
────────────────────────────────────────────────────────────
Simple counter → Atomic
Read-heavy     → RwLock
Write-heavy    → Mutex
Pipeline       → Channel
```

---

## What's Next?

**Lesson 57 — Send & Sync** — The marker traits that underpin thread safety. Understand why Rust prevents data races at compile time.

## Further Reading
- [The Rust Book — Ch 16.3: Shared State](https://doc.rust-lang.org/book/ch16-03-shared-state.html)
- [std::sync::Mutex](https://doc.rust-lang.org/std/sync/struct.Mutex.html)
- [std::sync::RwLock](https://doc.rust-lang.org/std/sync/struct.RwLock.html)

---

*Mutex & RwLock: safe shared state with zero data races! 🦀*
