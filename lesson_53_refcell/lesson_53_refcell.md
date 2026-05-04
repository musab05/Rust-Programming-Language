# 📘 Lesson 53 — RefCell & Interior Mutability (SP3)

> **Series:** Rust From Zero · Intermediate Level  
> **Roadmap ID:** SP3 · Category: 📌 Smart Pointers  
> **Previous:** [Lesson 52 — Rc & Arc](../lesson_52_rc_arc/lesson_52_rc_arc.md)  
> **Next:** [Lesson 54 — Threads & spawn](../lesson_54_threads/lesson_54_threads.md)  
> **Practice:** [Questions](./lesson_53_questions.md) · [Answers](./lesson_53_answers.md)  
> **Practice Task:** Build a mock object using RefCell for testing

---

## Table of Contents

1. [The Mutability Problem](#1-the-mutability-problem)
2. [Interior Mutability Pattern](#2-interior-mutability-pattern)
3. [RefCell\<T\> — Runtime Borrow Checking](#3-refcellt--runtime-borrow-checking)
4. [borrow() and borrow_mut()](#4-borrow-and-borrow_mut)
5. [Runtime Panics](#5-runtime-panics)
6. [Rc\<RefCell\<T\>\> — Shared Mutable Data](#6-rcrefcellt--shared-mutable-data)
7. [Cell\<T\> — Copy Types](#7-cellt--copy-types)
8. [Real-World Example: Mock Object](#8-real-world-example-mock-object)
9. [Choosing the Right Tool](#9-choosing-the-right-tool)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. The Mutability Problem

Rust's borrow rules enforce safety at compile time:
- Multiple `&T` OR one `&mut T`, never both

But sometimes you need to mutate data behind an immutable reference:

```rust
// ❌ Can't mutate through &self
struct Counter { count: u32 }
impl Counter {
    fn increment(&self) {
        // self.count += 1;  // error: cannot mutate through &self
    }
}
```

**Interior mutability** lets you bend this rule safely — with runtime checks instead of compile-time checks.

---

## 2. Interior Mutability Pattern

The idea: the **outer** value looks immutable, but its **interior** can be mutated:

```rust
use std::cell::RefCell;

struct Counter {
    count: RefCell<u32>,  // wraps the mutable part
}

impl Counter {
    fn new() -> Self {
        Counter { count: RefCell::new(0) }
    }

    fn increment(&self) {  // takes &self (immutable!)
        *self.count.borrow_mut() += 1;  // mutates interior
    }

    fn value(&self) -> u32 {
        *self.count.borrow()
    }
}

fn main() {
    let counter = Counter::new();  // no `mut` needed!
    counter.increment();
    counter.increment();
    counter.increment();
    println!("Count: {}", counter.value());  // 3
}
```

---

## 3. RefCell\<T\> — Runtime Borrow Checking

`RefCell<T>` enforces Rust's borrow rules **at runtime** instead of compile time:

```rust
use std::cell::RefCell;

fn main() {
    let data = RefCell::new(vec![1, 2, 3]);

    // Immutable borrow via borrow()
    {
        let borrowed = data.borrow();
        println!("Data: {:?}", *borrowed);
        // Multiple immutable borrows are OK
        let borrowed2 = data.borrow();
        println!("Same data: {:?}", *borrowed2);
    }
    // borrows dropped here

    // Mutable borrow via borrow_mut()
    {
        let mut mutable = data.borrow_mut();
        mutable.push(4);
        mutable.push(5);
    }
    // mutable borrow dropped here

    println!("Updated: {:?}", data.borrow());  // [1, 2, 3, 4, 5]
}
```

---

## 4. borrow() and borrow_mut()

| Method | Returns | Rule |
|---|---|---|
| `borrow()` | `Ref<T>` (like `&T`) | Multiple OK, no active `borrow_mut()` |
| `borrow_mut()` | `RefMut<T>` (like `&mut T`) | Exactly one, no active `borrow()` |

```rust
use std::cell::RefCell;

fn main() {
    let cell = RefCell::new(String::from("hello"));

    // borrow() returns Ref<T>
    let r1 = cell.borrow();
    println!("Length: {}", r1.len());
    drop(r1);  // must drop before borrow_mut

    // borrow_mut() returns RefMut<T>
    let mut r2 = cell.borrow_mut();
    r2.push_str(", world!");
    drop(r2);

    println!("{}", cell.borrow());  // hello, world!
}
```

### Try-variants (non-panicking):

```rust
use std::cell::RefCell;

fn main() {
    let cell = RefCell::new(42);

    let r = cell.borrow();
    match cell.try_borrow_mut() {
        Ok(mut val) => *val += 1,
        Err(e) => println!("Can't borrow: {e}"),  // ← this runs
    }
    drop(r);

    // Now it works
    *cell.borrow_mut() += 1;
    println!("{}", cell.borrow());  // 43
}
```

---

## 5. Runtime Panics

Violating borrow rules with `RefCell` causes a **runtime panic**, not a compile error:

```rust
use std::cell::RefCell;

fn main() {
    let cell = RefCell::new(42);

    let r1 = cell.borrow();      // immutable borrow active
    // let r2 = cell.borrow_mut();  // 💥 PANIC! can't mut borrow while immutably borrowed

    // Thread panicked: 'already borrowed: BorrowMutError'
}
```

**Trade-off:**
- Compile-time checking (`&T` / `&mut T`) → catches bugs during compilation
- Runtime checking (`RefCell`) → more flexible, but bugs cause panics at runtime

---

## 6. Rc\<RefCell\<T\>\> — Shared Mutable Data

Combine `Rc` (shared ownership) with `RefCell` (interior mutability):

```rust
use std::rc::Rc;
use std::cell::RefCell;

#[derive(Debug)]
struct SharedList {
    items: Rc<RefCell<Vec<String>>>,
}

impl SharedList {
    fn new() -> Self {
        SharedList { items: Rc::new(RefCell::new(vec![])) }
    }

    fn add(&self, item: &str) {
        self.items.borrow_mut().push(item.to_string());
    }

    fn share(&self) -> SharedList {
        SharedList { items: Rc::clone(&self.items) }
    }

    fn print(&self) {
        println!("{:?}", self.items.borrow());
    }
}

fn main() {
    let list_a = SharedList::new();
    let list_b = list_a.share();  // shares the same underlying Vec

    list_a.add("hello");
    list_b.add("world");  // modifying through the second reference!
    list_a.add("rust");

    list_a.print();  // ["hello", "world", "rust"]
    list_b.print();  // ["hello", "world", "rust"] — same data!

    println!("Ref count: {}", Rc::strong_count(&list_a.items));  // 2
}
```

---

## 7. Cell\<T\> — Copy Types

For `Copy` types, `Cell<T>` is simpler than `RefCell<T>`:

```rust
use std::cell::Cell;

struct Sensor {
    id: u32,
    readings: Cell<u32>,  // no need for borrow/borrow_mut
}

impl Sensor {
    fn new(id: u32) -> Self {
        Sensor { id, readings: Cell::new(0) }
    }

    fn record(&self) {
        self.readings.set(self.readings.get() + 1);
    }

    fn count(&self) -> u32 {
        self.readings.get()
    }
}

fn main() {
    let sensor = Sensor::new(1);
    sensor.record();
    sensor.record();
    sensor.record();
    println!("Sensor {} readings: {}", sensor.id, sensor.count());  // 3
}
```

| | `Cell<T>` | `RefCell<T>` |
|---|---|---|
| For types | `Copy` types only | Any type |
| API | `get()` / `set()` | `borrow()` / `borrow_mut()` |
| Can panic | Never | Yes (runtime borrow violation) |
| Overhead | None | Borrow counter tracking |

---

## 8. Real-World Example: Mock Object

The roadmap practice task — using `RefCell` for testing:

```rust
trait Messenger {
    fn send(&self, msg: &str);
}

// Real implementation
struct EmailSender;
impl Messenger for EmailSender {
    fn send(&self, msg: &str) {
        println!("📧 Sending email: {msg}");
    }
}

// Mock for testing — records messages instead of sending
struct MockMessenger {
    messages: RefCell<Vec<String>>,  // interior mutability!
}

use std::cell::RefCell;

impl MockMessenger {
    fn new() -> Self {
        MockMessenger { messages: RefCell::new(vec![]) }
    }

    fn sent_messages(&self) -> Vec<String> {
        self.messages.borrow().clone()
    }
}

impl Messenger for MockMessenger {
    fn send(&self, msg: &str) {
        // &self is immutable, but we can still record messages!
        self.messages.borrow_mut().push(msg.to_string());
    }
}

// The system under test
struct LimitTracker<'a, T: Messenger> {
    messenger: &'a T,
    value: usize,
    max: usize,
}

impl<'a, T: Messenger> LimitTracker<'a, T> {
    fn new(messenger: &'a T, max: usize) -> Self {
        LimitTracker { messenger, value: 0, max }
    }

    fn set_value(&mut self, value: usize) {
        self.value = value;

        let percentage = self.value as f64 / self.max as f64;

        if percentage >= 1.0 {
            self.messenger.send("🚨 Error: You've exceeded your quota!");
        } else if percentage >= 0.9 {
            self.messenger.send("⚠️ Warning: You've used 90% of your quota!");
        } else if percentage >= 0.75 {
            self.messenger.send("ℹ️ You've used 75% of your quota.");
        }
    }
}

fn main() {
    // Using mock for testing
    let mock = MockMessenger::new();
    let mut tracker = LimitTracker::new(&mock, 100);

    tracker.set_value(80);
    tracker.set_value(95);
    tracker.set_value(105);

    println!("Messages sent:");
    for msg in mock.sent_messages() {
        println!("  {msg}");
    }
}
```

---

## 9. Choosing the Right Tool

```
Need mutation?
├── Through &mut self → normal mutation (no smart pointer needed)
├── Through &self → interior mutability
│   ├── Copy type? → Cell<T>
│   ├── Non-Copy, single thread? → RefCell<T>
│   └── Non-Copy, multi-thread? → Mutex<T> (lesson 55)
│
Shared ownership + mutation?
├── Single thread → Rc<RefCell<T>>
└── Multi-thread  → Arc<Mutex<T>>
```

---

## 10. Summary Cheat Sheet

```
RefCell<T> — RUNTIME BORROW CHECKING
────────────────────────────────────────────────────────────
RefCell::new(val)        create
cell.borrow()            → Ref<T>     (like &T)
cell.borrow_mut()        → RefMut<T>  (like &mut T)
cell.try_borrow()        → Result (non-panicking)
cell.try_borrow_mut()    → Result (non-panicking)

RULES (checked at RUNTIME)
────────────────────────────────────────────────────────────
Multiple borrow()        ✅ OK
One borrow_mut()         ✅ OK
borrow() + borrow_mut()  💥 PANIC

Cell<T> — FOR COPY TYPES
────────────────────────────────────────────────────────────
Cell::new(val)           create
cell.get()               copy value out
cell.set(val)            replace value

COMMON PATTERNS
────────────────────────────────────────────────────────────
Rc<RefCell<T>>           shared + mutable (single-thread)
Arc<Mutex<T>>            shared + mutable (multi-thread)
Cell<T>                  simple counter, flags

WHEN TO USE
────────────────────────────────────────────────────────────
Mock objects in tests    → RefCell to record through &self
Shared mutable state     → Rc<RefCell<T>>
Graph nodes              → Rc<RefCell<Node>>
```

---

## What's Next?

**Lesson 54 — Threads & spawn** — Enter concurrency! Learn `std::thread::spawn`, `move` closures with threads, and `JoinHandle`.

## Further Reading
- [The Rust Book — Ch 15.5: RefCell](https://doc.rust-lang.org/book/ch15-05-interior-mutability.html)
- [std::cell::RefCell](https://doc.rust-lang.org/std/cell/struct.RefCell.html)
- [std::cell::Cell](https://doc.rust-lang.org/std/cell/struct.Cell.html)

---

*Interior mutability: bending the rules safely, one runtime check at a time! 🦀*
