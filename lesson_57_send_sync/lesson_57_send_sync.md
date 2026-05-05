# 📘 Lesson 57 — Send & Sync (CC4)

> **Series:** Rust From Zero · Advanced Level  
> **Roadmap ID:** CC4 · Category: ⚡ Concurrency  
> **Previous:** [Lesson 56 — Shared State: Mutex & RwLock](../lesson_56_mutex_rwlock/lesson_56_mutex_rwlock.md)  
> **Next:** [Lesson 58 — Async/Await Basics](../lesson_58_async_await/lesson_58_async_await.md)  
> **Practice:** [Questions](./lesson_57_questions.md) · [Answers](./lesson_57_answers.md)  
> **Practice Task:** Analyze which types are Send/Sync and build thread-safe wrappers

---

## Table of Contents

1. [What Are Send and Sync?](#1-what-are-send-and-sync)
2. [Send — Ownership Transfer Between Threads](#2-send--ownership-transfer-between-threads)
3. [Sync — Shared References Between Threads](#3-sync--shared-references-between-threads)
4. [Automatic Implementation](#4-automatic-implementation)
5. [Types That Are NOT Send/Sync](#5-types-that-are-not-sendsync)
6. [Making Custom Types Thread-Safe](#6-making-custom-types-thread-safe)
7. [Unsafe impl Send/Sync](#7-unsafe-impl-sendsync)
8. [Common Patterns](#8-common-patterns)
9. [Summary Cheat Sheet](#9-summary-cheat-sheet)

---

## 1. What Are Send and Sync?

Two **marker traits** that the compiler uses to enforce thread safety:

| Trait | Meaning | Example |
|---|---|---|
| `Send` | Value can be **moved** to another thread | `thread::spawn(move \|\| { uses_val })` |
| `Sync` | Value can be **shared** (via `&T`) across threads | `Arc::new(val)` requires `T: Sync` |

**Key insight:** These traits have no methods. They are **promises** checked at compile time.

```rust
// Simplified definitions
pub unsafe auto trait Send { }
pub unsafe auto trait Sync { }

// "auto" → compiler implements automatically for types whose fields are Send/Sync
// "unsafe" → manually implementing can cause UB if wrong
```

---

## 2. Send — Ownership Transfer Between Threads

A type is `Send` if it's safe to move it to another thread:

```rust
use std::thread;

fn must_be_send<T: Send>(val: T) {
    thread::spawn(move || {
        drop(val);  // val is used in another thread
    }).join().unwrap();
}

fn main() {
    must_be_send(42);                          // ✅ i32 is Send
    must_be_send(String::from("hello"));       // ✅ String is Send
    must_be_send(vec![1, 2, 3]);               // ✅ Vec<i32> is Send
    must_be_send(Box::new(42));                // ✅ Box<i32> is Send

    // must_be_send(std::rc::Rc::new(42));     // ❌ Rc is NOT Send
}
```

---

## 3. Sync — Shared References Between Threads

A type is `Sync` if `&T` is safe to share between threads:

```rust
use std::sync::Arc;
use std::thread;

fn must_be_sync<T: Sync + Send>(val: T) {
    let shared = Arc::new(val);
    let mut handles = vec![];

    for i in 0..3 {
        let shared = Arc::clone(&shared);
        handles.push(thread::spawn(move || {
            println!("Thread {i}: {:?}", &*shared);
        }));
    }

    for h in handles { h.join().unwrap(); }
}

fn main() {
    must_be_sync(42_i32);               // ✅ i32 is Sync
    must_be_sync(String::from("hi"));   // ✅ String is Sync
    // must_be_sync(std::cell::Cell::new(42));  // ❌ Cell is NOT Sync
}
```

### The relationship:

```
T is Sync  ←→  &T is Send

If you can safely share &T across threads (Sync),
then sending &T to another thread (Send for &T) is safe.
```

---

## 4. Automatic Implementation

The compiler auto-implements `Send` and `Sync` for types composed of `Send`/`Sync` fields:

```rust
// All fields are Send + Sync → automatically Send + Sync
struct MyStruct {
    name: String,     // Send + Sync ✅
    count: i32,       // Send + Sync ✅
    data: Vec<u8>,    // Send + Sync ✅
}
// MyStruct is automatically Send + Sync ✅

// One non-Send field → the whole struct is NOT Send
use std::rc::Rc;
struct NotSendStruct {
    name: String,     // Send ✅
    shared: Rc<i32>,  // NOT Send ❌
}
// NotSendStruct is NOT Send ❌
```

---

## 5. Types That Are NOT Send/Sync

| Type | Send? | Sync? | Why |
|---|---|---|---|
| `Rc<T>` | ❌ | ❌ | Non-atomic ref count — race condition |
| `RefCell<T>` | ✅ | ❌ | Non-atomic borrow counter |
| `Cell<T>` | ✅ | ❌ | Interior mutability without sync |
| `*mut T` | ❌ | ❌ | Raw pointers are unsafe |
| `MutexGuard` | ❌ | ✅ | Must unlock on same thread (OS rule) |

### Why Rc is not Send:

```rust
use std::rc::Rc;
use std::thread;

fn main() {
    let data = Rc::new(42);

    // ❌ Compiler prevents this:
    // thread::spawn(move || {
    //     println!("{data}");
    // });
    // Error: `Rc<i32>` cannot be sent between threads safely

    // ✅ Fix: use Arc instead
    use std::sync::Arc;
    let data = Arc::new(42);
    thread::spawn(move || println!("{data}")).join().unwrap();
}
```

### Why Cell/RefCell is not Sync:

```rust
use std::cell::Cell;

fn main() {
    let val = Cell::new(42);

    // If Cell were Sync, two threads could call .set() simultaneously
    // on the same Cell through shared &Cell — data race!
    // That's why Cell is NOT Sync.

    // ✅ Fix: use Mutex or Atomic for thread-safe interior mutability
}
```

---

## 6. Making Custom Types Thread-Safe

### Replace non-thread-safe types:

```rust
use std::sync::{Arc, Mutex};

// ❌ Not thread-safe
// struct AppState {
//     cache: Rc<RefCell<HashMap<String, String>>>,
// }

// ✅ Thread-safe equivalent
use std::collections::HashMap;

struct AppState {
    cache: Arc<Mutex<HashMap<String, String>>>,
}

impl AppState {
    fn new() -> Self {
        AppState {
            cache: Arc::new(Mutex::new(HashMap::new())),
        }
    }

    fn get(&self, key: &str) -> Option<String> {
        self.cache.lock().unwrap().get(key).cloned()
    }

    fn set(&self, key: String, value: String) {
        self.cache.lock().unwrap().insert(key, value);
    }
}
```

### Thread-safe conversion table:

| Single-threaded | Multi-threaded |
|---|---|
| `Rc<T>` | `Arc<T>` |
| `RefCell<T>` | `Mutex<T>` or `RwLock<T>` |
| `Cell<T>` | `AtomicT` |
| `Rc<RefCell<T>>` | `Arc<Mutex<T>>` |

---

## 7. Unsafe impl Send/Sync

You can manually implement these traits with `unsafe` — but only if you guarantee correctness:

```rust
struct MyWrapper {
    ptr: *mut i32,  // raw pointer — NOT Send/Sync by default
}

// ⚠️ Only do this if you guarantee thread safety!
unsafe impl Send for MyWrapper {}
unsafe impl Sync for MyWrapper {}
```

### When to use:
- FFI wrappers where you control the underlying C library's thread safety
- Performance-critical code with manual synchronization

### Opting OUT of Send/Sync:

```rust
use std::marker::PhantomData;

struct NotSend {
    data: i32,
    _marker: PhantomData<*const ()>,  // *const () is NOT Send
}
// NotSend is now NOT Send due to PhantomData
```

---

## 8. Common Patterns

### Assert Send/Sync at compile time:

```rust
fn assert_send<T: Send>() {}
fn assert_sync<T: Sync>() {}

fn main() {
    assert_send::<String>();           // ✅
    assert_sync::<String>();           // ✅
    assert_send::<Vec<i32>>();         // ✅
    // assert_send::<std::rc::Rc<i32>>();  // ❌ won't compile
}
```

### Require Send in function signatures:

```rust
use std::thread;

fn spawn_task<F, T>(f: F) -> thread::JoinHandle<T>
where
    F: FnOnce() -> T + Send + 'static,
    T: Send + 'static,
{
    thread::spawn(f)
}

fn main() {
    let handle = spawn_task(|| {
        42
    });
    println!("{}", handle.join().unwrap());
}
```

---

## 9. Summary Cheat Sheet

```
SEND
────────────────────────────────────────────────────────────
T: Send → can move T to another thread
Most types are Send
❌ Rc<T>, *mut T, MutexGuard

SYNC
────────────────────────────────────────────────────────────
T: Sync → can share &T across threads
T: Sync ↔ &T: Send
Most types are Sync
❌ Rc<T>, Cell<T>, RefCell<T>

AUTO-IMPLEMENTATION
────────────────────────────────────────────────────────────
Struct is Send if ALL fields are Send
Struct is Sync if ALL fields are Sync
One non-Send field → whole struct is not Send

THREAD-SAFE REPLACEMENTS
────────────────────────────────────────────────────────────
Rc<T>          → Arc<T>
RefCell<T>     → Mutex<T> / RwLock<T>
Cell<T>        → AtomicT
Rc<RefCell<T>> → Arc<Mutex<T>>

UNSAFE IMPL
────────────────────────────────────────────────────────────
unsafe impl Send for MyType {}
unsafe impl Sync for MyType {}
Only when YOU guarantee thread safety!
```

---

## What's Next?

**Lesson 58 — Async/Await Basics** — Enter the world of asynchronous Rust. Learn `async fn`, `.await`, `Future`, and why async is different from threads.

## Further Reading
- [The Rust Book — Ch 16.4: Send and Sync](https://doc.rust-lang.org/book/ch16-04-extensible-concurrency-sync-and-send.html)
- [Rustonomicon — Send and Sync](https://doc.rust-lang.org/nomicon/send-and-sync.html)

---

*Send & Sync: Rust's compile-time guarantee against data races! 🦀*
