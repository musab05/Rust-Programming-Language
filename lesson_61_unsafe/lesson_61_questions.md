# 🧪 Lesson 61 — Questions: Unsafe Rust (U1)

> **Lesson:** [lesson_61_unsafe.md](./lesson_61_unsafe.md)  
> **Answers:** [lesson_61_answers.md](./lesson_61_answers.md)

---

## Section A — Compile or Not?

### Q1
```rust
fn main() {
    let x = 42;
    let ptr = &x as *const i32;
    println!("{}", *ptr);
}
```

### Q2
```rust
fn main() {
    let x = 42;
    let ptr = &x as *const i32;
    unsafe { println!("{}", *ptr); }
}
```

### Q3
```rust
static mut COUNT: u32 = 0;
fn main() {
    COUNT += 1;
    println!("{}", COUNT);
}
```

---

## Section B — Write It Yourself

### Q4 — Safe wrapper (Roadmap Practice Task)
Write a `SafeArray<T>` struct that uses raw pointers internally. Implement `new(size)`, `get(index)`, and `set(index, value)` with bounds checking. The public API must be fully safe.

### Q5 — split_at_mut
Implement your own version of `split_at_mut` for `&mut [i32]` using raw pointers, wrapped in a safe function with an assertion.

---

## Section C — True or False?

### Q6
1. `unsafe` turns off the borrow checker.
2. Creating a raw pointer is safe; dereferencing it is unsafe.
3. `static mut` variables are safe to access from multiple threads.
4. `unsafe impl Send` means YOU guarantee thread safety, not the compiler.
5. An unsafe block lets you do anything without rules.
6. The preferred alternative to `static mut` is `AtomicT` or `Mutex`.

### Q7
List the five unsafe superpowers and give a one-line example of each.

---

*Unsafe: the escape hatch you rarely need but must understand! 🦀*
