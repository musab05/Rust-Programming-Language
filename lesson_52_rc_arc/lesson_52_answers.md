# ✅ Lesson 52 — Answers: Rc & Arc (SP2)

---

## Section A

### A1
```
2 42
```
After `Rc::clone`, count is 2. `*b` dereferences to 42.

### A2
```
1
```
`_b` is dropped when the inner scope ends, so count goes back to 1.

### A3 — ❌ Won't compile
`Rc` only gives immutable access. `push` requires `&mut Vec`, but `Rc<Vec>` only provides `&Vec`. Error: `cannot borrow as mutable`.

---

## Section B

### A4
```rust
use std::rc::Rc;

struct Config { debug: bool, port: u16 }

struct DbService { config: Rc<Config> }
struct WebService { config: Rc<Config> }
struct LogService { config: Rc<Config> }

fn main() {
    let config = Rc::new(Config { debug: true, port: 8080 });
    println!("Count: {}", Rc::strong_count(&config));  // 1

    let db = DbService { config: Rc::clone(&config) };
    println!("Count: {}", Rc::strong_count(&config));  // 2

    let web = WebService { config: Rc::clone(&config) };
    println!("Count: {}", Rc::strong_count(&config));  // 3

    let log = LogService { config: Rc::clone(&config) };
    println!("Count: {}", Rc::strong_count(&config));  // 4

    println!("Port: {}", config.port);
}
```

### A5
```rust
use std::sync::Arc;
use std::thread;

fn main() {
    let data = Arc::new(vec!["hello".to_string(), "world".to_string(), "rust".to_string()]);
    let mut handles = vec![];

    for i in 0..3 {
        let data = Arc::clone(&data);
        handles.push(thread::spawn(move || {
            println!("Thread {i}: len = {}", data.len());
        }));
    }
    for h in handles { h.join().unwrap(); }
}
```

### A6
```rust
use std::rc::{Rc, Weak};

fn main() {
    let strong = Rc::new(42);
    let weak: Weak<i32> = Rc::downgrade(&strong);

    println!("Before drop: {:?}", weak.upgrade());  // Some(42)
    drop(strong);
    println!("After drop: {:?}", weak.upgrade());   // None
}
```

---

## Section C

### A7
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **False** | `Rc::clone` only increments the reference count — no data copy |
| 2 | **False** | `Rc` is NOT `Send` — single-threaded only. Use `Arc` for threads |
| 3 | **True** | `Arc` uses atomic operations (slightly slower than Rc) |
| 4 | **True** | When strong count reaches 0, the data is deallocated |
| 5 | **True** | `upgrade()` returns `Some(Rc)` if data still exists, `None` if dropped |
| 6 | **False** | `Rc` only provides `&T` (immutable). Use `Rc<RefCell<T>>` for mutation |

---

## 🏆 Lesson 52 Complete!

**Next up:** [Lesson 53 — RefCell & Interior Mutability](../lesson_53_refcell/lesson_53_refcell.md) 🦀
