# 🧪 Lesson 71 — Questions: Weak\<T\> (SP4)

> **Lesson:** [lesson_71_weak.md](./lesson_71_weak.md)  
> **Answers:** [lesson_71_answers.md](./lesson_71_answers.md)

---

## Section A — Predict

### Q1
```rust
use std::rc::{Rc, Weak};
fn main() {
    let weak: Weak<i32>;
    { let rc = Rc::new(42); weak = Rc::downgrade(&rc); }
    println!("{:?}", weak.upgrade());
}
```

### Q2
```rust
use std::rc::Rc;
fn main() {
    let a = Rc::new(1);
    let _b = Rc::clone(&a);
    let _c = Rc::downgrade(&a);
    println!("strong={}, weak={}", Rc::strong_count(&a), Rc::weak_count(&a));
}
```

---

## Section B — Write It Yourself

### Q3 — Tree with parent refs (Roadmap Practice Task)
Build a tree where each node has `children: Vec<Rc<Node>>` and `parent: Weak<Node>`. Add 3 levels. Navigate from a leaf back to the root using `upgrade()`.

### Q4 — Doubly-linked list
Create a doubly-linked list where `next` is `Rc` and `prev` is `Weak`. Add 3 nodes, traverse forward and backward.

---

## Section C — True or False?

### Q5
1. `Weak<T>` prevents the value from being dropped.
2. `upgrade()` returns `Option<Rc<T>>`.
3. A value is dropped when its `strong_count` reaches 0, regardless of `weak_count`.
4. `Rc::downgrade()` increases the strong count.
5. `Weak::new()` creates a weak reference that always fails `upgrade()`.
6. Using `Weak` for parent pointers in trees prevents reference cycles.

---

*Weak: non-owning, cycle-breaking, polite references! 🦀*
