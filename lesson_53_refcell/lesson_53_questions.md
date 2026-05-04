# 🧪 Lesson 53 — Questions: RefCell & Interior Mutability (SP3)

> **Lesson:** [lesson_53_refcell.md](./lesson_53_refcell.md)  
> **Answers:** [lesson_53_answers.md](./lesson_53_answers.md)

---

## Section A — Predict: Compile, Run, or Panic?

### Q1
```rust
use std::cell::RefCell;
fn main() {
    let cell = RefCell::new(5);
    let a = cell.borrow();
    let b = cell.borrow();
    println!("{} {}", *a, *b);
}
```

### Q2
```rust
use std::cell::RefCell;
fn main() {
    let cell = RefCell::new(5);
    let a = cell.borrow();
    let b = cell.borrow_mut();
    println!("{} {}", *a, *b);
}
```

### Q3
```rust
use std::cell::Cell;
fn main() {
    let x = Cell::new(10);
    x.set(20);
    println!("{}", x.get());
}
```

---

## Section B — Write It Yourself

### Q4 — Mock object (Roadmap Practice Task)
Create a `Messenger` trait with `fn send(&self, msg: &str)`. Build a `MockMessenger` using `RefCell<Vec<String>>` that records messages. Verify it collected the right messages.

### Q5 — Shared mutable list
Use `Rc<RefCell<Vec<i32>>>` to create a list shared between two variables. Add items through both and verify they see the same data.

### Q6 — Interior counter
Create a `Counter` struct with a `Cell<u32>` count. Implement `increment(&self)` and `value(&self)`. Demonstrate that no `mut` is needed.

---

## Section C — True or False?

### Q7
1. `RefCell<T>` checks borrow rules at compile time.
2. `borrow()` and `borrow_mut()` active simultaneously causes a panic.
3. `Cell<T>` can only be used with `Copy` types.
4. `Rc<RefCell<T>>` provides shared mutable ownership in single-threaded code.
5. `RefCell<T>` is `Sync` (can be shared across threads).
6. `try_borrow_mut()` returns `Result` instead of panicking.

### Q8
When would you choose `RefCell` over just using `&mut T`? Give two concrete scenarios.

---

*Interior mutability: when &self needs to mutate! 🦀*
