# 📘 Lesson 52 — Rc & Arc (SP2)

> **Series:** Rust From Zero · Intermediate Level  
> **Roadmap ID:** SP2 · Category: 📌 Smart Pointers  
> **Previous:** [Lesson 51 — Smart Pointers: Box](../lesson_51_box/lesson_51_box.md)  
> **Next:** [Lesson 53 — RefCell & Interior Mutability](../lesson_53_refcell/lesson_53_refcell.md)  
> **Practice:** [Questions](./lesson_52_questions.md) · [Answers](./lesson_52_answers.md)  
> **Practice Task:** Model a shared-ownership DAG with Rc

---

## Table of Contents

1. [The Shared Ownership Problem](#1-the-shared-ownership-problem)
2. [Rc\<T\> — Reference Counting](#2-rct--reference-counting)
3. [Rc::clone and Reference Counts](#3-rcclone-and-reference-counts)
4. [Rc in Data Structures](#4-rc-in-data-structures)
5. [Rc Limitations](#5-rc-limitations)
6. [Arc\<T\> — Thread-Safe Rc](#6-arct--thread-safe-rc)
7. [Weak\<T\> — Breaking Cycles](#7-weakt--breaking-cycles)
8. [Rc vs Arc vs Box](#8-rc-vs-arc-vs-box)
9. [Real-World Example: Shared Config](#9-real-world-example-shared-config)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. The Shared Ownership Problem

Sometimes multiple parts of your program need to own the same data:

```rust
// ❌ Can't give the same Vec to two owners
// fn main() {
//     let data = vec![1, 2, 3];
//     let a = data;
//     let b = data;  // error: value used after move
// }
```

`Rc<T>` and `Arc<T>` solve this with **reference counting**.

---

## 2. Rc\<T\> — Reference Counting

`Rc` tracks how many owners share the data. Data is freed when the last owner is dropped.

```rust
use std::rc::Rc;

fn main() {
    let data = Rc::new(vec![1, 2, 3, 4, 5]);

    // Clone bumps the reference count (cheap — no data copy)
    let a = Rc::clone(&data);
    let b = Rc::clone(&data);

    println!("data: {:?}", data);
    println!("a:    {:?}", a);
    println!("b:    {:?}", b);

    // All three point to the SAME Vec on the heap
    println!("Same address: {}", Rc::ptr_eq(&data, &a));  // true
    println!("Ref count: {}", Rc::strong_count(&data));     // 3
}
```

---

## 3. Rc::clone and Reference Counts

```rust
use std::rc::Rc;

fn main() {
    let original = Rc::new(String::from("shared data"));
    println!("Count after creation: {}", Rc::strong_count(&original));  // 1

    {
        let clone1 = Rc::clone(&original);
        println!("Count after clone1: {}", Rc::strong_count(&original));  // 2

        let clone2 = Rc::clone(&original);
        println!("Count after clone2: {}", Rc::strong_count(&original));  // 3

        // clone2 dropped here
    }
    // clone1 also dropped here

    println!("Count after scope: {}", Rc::strong_count(&original));  // 1

    // original dropped at end — ref count hits 0 → memory freed
}
```

### `Rc::clone` vs `.clone()`:

```rust
use std::rc::Rc;

fn main() {
    let data = Rc::new(String::from("hello"));

    // Rc::clone — increments ref count (CHEAP: just a counter bump)
    let a = Rc::clone(&data);

    // data.clone() also works — but Rc::clone is idiomatic
    // It makes clear you're sharing, not deep-copying
    let b = data.clone();

    println!("Count: {}", Rc::strong_count(&data));  // 3
}
```

---

## 4. Rc in Data Structures

### Shared nodes in a graph:

```rust
use std::rc::Rc;

#[derive(Debug)]
struct Node {
    value: i32,
    children: Vec<Rc<Node>>,
}

fn main() {
    // Shared leaf node
    let shared_leaf = Rc::new(Node { value: 10, children: vec![] });

    // Two parents share the same child
    let parent_a = Node {
        value: 1,
        children: vec![Rc::clone(&shared_leaf)],
    };

    let parent_b = Node {
        value: 2,
        children: vec![Rc::clone(&shared_leaf)],
    };

    println!("Shared leaf refs: {}", Rc::strong_count(&shared_leaf));  // 3
    println!("Parent A: {:?}", parent_a);
    println!("Parent B: {:?}", parent_b);
}
```

### Shared linked list suffix:

```rust
use std::rc::Rc;

#[derive(Debug)]
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

fn main() {
    // Shared tail: 3 → 4 → Nil
    let shared_tail = Rc::new(List::Cons(3, Rc::new(List::Cons(4, Rc::new(List::Nil)))));

    // Two lists sharing the same tail
    let list_a = List::Cons(1, Rc::clone(&shared_tail));  // 1 → 3 → 4 → Nil
    let list_b = List::Cons(2, Rc::clone(&shared_tail));  // 2 → 3 → 4 → Nil

    println!("Tail refs: {}", Rc::strong_count(&shared_tail));  // 3
}
```

---

## 5. Rc Limitations

```rust
use std::rc::Rc;

fn main() {
    let data = Rc::new(vec![1, 2, 3]);

    // ❌ Rc gives immutable access only
    // data.push(4);  // error: cannot borrow as mutable

    // ❌ Rc is NOT thread-safe
    // std::thread::spawn(move || { println!("{:?}", data); });
    // error: Rc cannot be sent between threads safely

    // ✅ Read-only access works
    println!("Length: {}", data.len());
    println!("First: {:?}", data.first());
}
```

**Key limitations:**
1. **Immutable only** — `Rc<T>` only gives `&T`. Use `Rc<RefCell<T>>` for mutation.
2. **Single-threaded** — Not `Send` or `Sync`. Use `Arc<T>` for threads.
3. **Cycles cause leaks** — If A→B→A, ref counts never reach 0. Use `Weak<T>`.

---

## 6. Arc\<T\> — Thread-Safe Rc

`Arc` (Atomic Reference Counted) works across threads:

```rust
use std::sync::Arc;
use std::thread;

fn main() {
    let data = Arc::new(vec![1, 2, 3, 4, 5]);

    let mut handles = vec![];

    for i in 0..3 {
        let data_clone = Arc::clone(&data);
        let handle = thread::spawn(move || {
            let sum: i32 = data_clone.iter().sum();
            println!("Thread {i}: sum = {sum}");
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Final ref count: {}", Arc::strong_count(&data));  // 1
}
```

### Rc vs Arc:

| | `Rc<T>` | `Arc<T>` |
|---|---|---|
| Thread safety | ❌ Single-threaded only | ✅ Multi-threaded |
| Performance | Faster (no atomic ops) | Slight overhead |
| Implements | Not `Send`/`Sync` | `Send + Sync` |
| Use when | No threads needed | Sharing across threads |

---

## 7. Weak\<T\> — Breaking Cycles

`Weak<T>` doesn't prevent dropping — it's a non-owning reference:

```rust
use std::rc::{Rc, Weak};

#[derive(Debug)]
struct Node {
    value: i32,
    parent: Option<Weak<Node>>,    // weak ref — won't prevent drop
    children: Vec<Rc<Node>>,        // strong ref — owns children
}

fn main() {
    let parent = Rc::new(Node {
        value: 1,
        parent: None,
        children: vec![],
    });

    // Create weak reference
    let weak_ref = Rc::downgrade(&parent);

    println!("Strong: {}", Rc::strong_count(&parent));  // 1
    println!("Weak: {}", Rc::weak_count(&parent));      // 1

    // Upgrade weak to strong (returns Option<Rc<T>>)
    if let Some(strong) = weak_ref.upgrade() {
        println!("Value: {}", strong.value);  // 1
    }

    drop(parent);  // strong count hits 0 → data freed

    // Weak ref is now invalid
    println!("After drop: {:?}", weak_ref.upgrade());  // None
}
```

---

## 8. Rc vs Arc vs Box

| | `Box<T>` | `Rc<T>` | `Arc<T>` |
|---|---|---|---|
| Ownership | Single | Shared | Shared |
| Thread-safe | ✅ (moveable) | ❌ | ✅ |
| Mutability | `&mut` via owner | Immutable (need RefCell) | Immutable (need Mutex) |
| Overhead | None | Ref count | Atomic ref count |
| Use case | Heap alloc, recursive types | Shared data, single thread | Shared data, multi-thread |

---

## 9. Real-World Example: Shared Config

```rust
use std::rc::Rc;

#[derive(Debug)]
struct Config {
    database_url: String,
    max_connections: u32,
    debug_mode: bool,
}

struct DatabaseService {
    config: Rc<Config>,
}

struct LoggingService {
    config: Rc<Config>,
}

struct CacheService {
    config: Rc<Config>,
}

impl DatabaseService {
    fn connect(&self) {
        println!("DB connecting to {} (max: {})",
            self.config.database_url, self.config.max_connections);
    }
}

impl LoggingService {
    fn log(&self, msg: &str) {
        if self.config.debug_mode {
            println!("[DEBUG] {msg}");
        }
    }
}

impl CacheService {
    fn status(&self) {
        println!("Cache using config: debug={}", self.config.debug_mode);
    }
}

fn main() {
    let config = Rc::new(Config {
        database_url: "postgres://localhost:5432/app".into(),
        max_connections: 10,
        debug_mode: true,
    });

    let db = DatabaseService { config: Rc::clone(&config) };
    let logger = LoggingService { config: Rc::clone(&config) };
    let cache = CacheService { config: Rc::clone(&config) };

    println!("Config shared by {} services", Rc::strong_count(&config) - 1);

    db.connect();
    logger.log("Application started");
    cache.status();
}
```

---

## 10. Summary Cheat Sheet

```
Rc<T> — SINGLE-THREADED SHARED OWNERSHIP
────────────────────────────────────────────────────────────
Rc::new(val)             create
Rc::clone(&rc)           increment ref count (cheap)
Rc::strong_count(&rc)    current ref count
Rc::downgrade(&rc)       create Weak<T>

Arc<T> — THREAD-SAFE SHARED OWNERSHIP
────────────────────────────────────────────────────────────
Arc::new(val)            create
Arc::clone(&arc)         atomic increment
Same API as Rc, but Send + Sync

Weak<T> — NON-OWNING REFERENCE
────────────────────────────────────────────────────────────
Rc::downgrade(&rc)       create Weak
weak.upgrade()           → Option<Rc<T>> (None if dropped)
Use to break reference cycles

CHOOSING
────────────────────────────────────────────────────────────
Single owner           → Box<T>
Shared, single thread  → Rc<T>
Shared, multi thread   → Arc<T>
Need mutation?         → Rc<RefCell<T>> or Arc<Mutex<T>>
```

---

## What's Next?

**Lesson 53 — RefCell & Interior Mutability** — Mutate data behind an immutable reference. Learn runtime borrow checking and the `Rc<RefCell<T>>` pattern.

## Further Reading
- [The Rust Book — Ch 15.4: Rc](https://doc.rust-lang.org/book/ch15-04-rc.html)
- [std::rc::Rc](https://doc.rust-lang.org/std/rc/struct.Rc.html)
- [std::sync::Arc](https://doc.rust-lang.org/std/sync/struct.Arc.html)

---

*Rc & Arc: shared ownership through reference counting! 🦀*
