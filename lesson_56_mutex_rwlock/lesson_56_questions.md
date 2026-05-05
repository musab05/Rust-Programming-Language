# 🧪 Lesson 56 — Questions: Mutex & RwLock (CC3)

> **Lesson:** [lesson_56_mutex_rwlock.md](./lesson_56_mutex_rwlock.md)  
> **Answers:** [lesson_56_answers.md](./lesson_56_answers.md)

---

## Section A — Predict: What Happens?

### Q1
```rust
use std::sync::Mutex;
fn main() {
    let m = Mutex::new(10);
    let mut g = m.lock().unwrap();
    *g += 5;
    println!("{}", *g);
}
```

### Q2
```rust
use std::sync::Mutex;
fn main() {
    let m = Mutex::new(10);
    let g1 = m.lock().unwrap();
    let g2 = m.lock().unwrap();
    println!("{} {}", *g1, *g2);
}
```

### Q3 — Compile or not?
```rust
use std::sync::Mutex;
use std::thread;
fn main() {
    let m = Mutex::new(0);
    thread::spawn(move || { *m.lock().unwrap() += 1; });
    println!("{}", *m.lock().unwrap());
}
```

---

## Section B — Write It Yourself

### Q4 — Concurrent counter (Roadmap Practice Task)
Use `Arc<Mutex<i32>>` with 10 threads, each incrementing a counter 100 times. Verify the final count is 1000.

### Q5 — RwLock config
Create a shared config (HashMap) behind `Arc<RwLock<_>>`. Spawn 5 reader threads and 1 writer thread. Readers print a value; the writer updates it.

### Q6 — Atomic counter
Rewrite Q4 using `AtomicI32` instead of `Mutex`. Compare the simplicity.

---

## Section C — True or False?

### Q7
1. `Mutex::lock()` blocks if another thread holds the lock.
2. A `MutexGuard` automatically releases the lock when dropped.
3. `RwLock` allows multiple writers simultaneously.
4. A poisoned mutex can never be used again.
5. `AtomicU64` is faster than `Mutex<u64>` for simple counters.
6. Deadlocks are detected by the Rust compiler.

### Q8
Explain why `Arc<Mutex<T>>` is needed instead of just `Mutex<T>` when sharing data across threads.

---

*Locks: protecting shared data, one guard at a time! 🦀*
