# 🧪 Lesson 52 — Questions: Rc & Arc (SP2)

> **Lesson:** [lesson_52_rc_arc.md](./lesson_52_rc_arc.md)  
> **Answers:** [lesson_52_answers.md](./lesson_52_answers.md)

---

## Section A — Predict the Output

### Q1
```rust
use std::rc::Rc;
fn main() {
    let a = Rc::new(42);
    let b = Rc::clone(&a);
    println!("{} {}", Rc::strong_count(&a), *b);
}
```

### Q2
```rust
use std::rc::Rc;
fn main() {
    let a = Rc::new(String::from("hello"));
    { let _b = Rc::clone(&a); }
    println!("{}", Rc::strong_count(&a));
}
```

### Q3 — Compile or not?
```rust
use std::rc::Rc;
fn main() {
    let data = Rc::new(vec![1, 2, 3]);
    data.push(4);
}
```

---

## Section B — Write It Yourself

### Q4 — Shared config (Roadmap Practice Task)
Create a `Config` struct. Share it via `Rc` between three services. Print the reference count at each step.

### Q5 — Arc with threads
Share a `Vec<String>` between 3 threads using `Arc`. Each thread prints the vec's length.

### Q6 — Weak reference
Create an `Rc`, downgrade to `Weak`, print whether `upgrade()` works before and after dropping the `Rc`.

---

## Section C — True or False?

### Q7
1. `Rc::clone` performs a deep copy of the data.
2. `Rc<T>` can be sent between threads.
3. `Arc<T>` uses atomic operations for the reference count.
4. When all `Rc` clones are dropped, the data is freed.
5. `Weak::upgrade()` returns `Option<Rc<T>>`.
6. `Rc<T>` provides mutable access to T.

---

*Shared ownership: because sometimes one owner isn't enough! 🦀*
