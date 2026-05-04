# ✅ Lesson 53 — Answers: RefCell & Interior Mutability (SP3)

---

## Section A

### A1 — ✅ Runs fine
Multiple immutable borrows (`borrow()`) are allowed simultaneously. Output: `5 5`.

### A2 — 💥 Runtime panic
`borrow()` and `borrow_mut()` at the same time violates the rules. Panics with: `already borrowed: BorrowMutError`.

### A3 — ✅ Runs fine
`Cell<T>` uses `get()`/`set()` — no borrowing. Output: `20`.

---

## Section B

### A4
```rust
use std::cell::RefCell;

trait Messenger {
    fn send(&self, msg: &str);
}

struct MockMessenger {
    messages: RefCell<Vec<String>>,
}

impl MockMessenger {
    fn new() -> Self {
        MockMessenger { messages: RefCell::new(vec![]) }
    }
}

impl Messenger for MockMessenger {
    fn send(&self, msg: &str) {
        self.messages.borrow_mut().push(msg.to_string());
    }
}

fn main() {
    let mock = MockMessenger::new();
    mock.send("hello");
    mock.send("world");

    let msgs = mock.messages.borrow();
    assert_eq!(msgs.len(), 2);
    assert_eq!(msgs[0], "hello");
    assert_eq!(msgs[1], "world");
    println!("Mock collected {} messages: {:?}", msgs.len(), *msgs);
}
```

### A5
```rust
use std::rc::Rc;
use std::cell::RefCell;

fn main() {
    let shared = Rc::new(RefCell::new(vec![1, 2, 3]));
    let view_a = Rc::clone(&shared);
    let view_b = Rc::clone(&shared);

    view_a.borrow_mut().push(4);
    view_b.borrow_mut().push(5);

    println!("A sees: {:?}", view_a.borrow());  // [1, 2, 3, 4, 5]
    println!("B sees: {:?}", view_b.borrow());  // [1, 2, 3, 4, 5]
    println!("Same data: {}", *view_a.borrow() == *view_b.borrow());  // true
}
```

### A6
```rust
use std::cell::Cell;

struct Counter { count: Cell<u32> }

impl Counter {
    fn new() -> Self { Counter { count: Cell::new(0) } }
    fn increment(&self) { self.count.set(self.count.get() + 1); }
    fn value(&self) -> u32 { self.count.get() }
}

fn main() {
    let c = Counter::new();  // no mut!
    c.increment();
    c.increment();
    c.increment();
    println!("Count: {}", c.value());  // 3
}
```

---

## Section C

### A7
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **False** | `RefCell` checks borrow rules at **runtime**, not compile time |
| 2 | **True** | Simultaneous immutable + mutable borrow causes a panic |
| 3 | **True** | `Cell::get()` copies the value, so `T` must be `Copy` |
| 4 | **True** | `Rc` gives shared ownership, `RefCell` gives mutability |
| 5 | **False** | `RefCell` is NOT `Sync` — single-threaded only |
| 6 | **True** | `try_borrow_mut()` returns `Result<RefMut, BorrowMutError>` |

### A8
1. **Mock objects in tests** — trait methods take `&self`, but the mock needs to record calls. `RefCell` lets you mutate the recorded messages through `&self`.
2. **Shared mutable data with `Rc`** — when multiple owners need to read AND write shared data (e.g., observer pattern, tree nodes), `Rc<RefCell<T>>` provides shared mutable access without `&mut`.

---

## 🏆 Lesson 53 Complete!

**Next up:** [Lesson 54 — Threads & spawn](../lesson_54_threads/lesson_54_threads.md) 🦀
