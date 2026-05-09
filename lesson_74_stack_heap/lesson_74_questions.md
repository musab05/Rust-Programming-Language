# 🧪 Lesson 74 — Questions: Stack vs Heap (P1)

> **Lesson:** [lesson_74_stack_heap.md](./lesson_74_stack_heap.md)  
> **Answers:** [lesson_74_answers.md](./lesson_74_answers.md)

---

## Section A — Stack or Heap?

### Q1
For each variable, state whether its data lives on the stack, heap, or both:
```rust
let a: i32 = 42;
let b: String = String::from("hello");
let c: [u8; 100] = [0; 100];
let d: Vec<i32> = vec![1, 2, 3];
let e: &str = "static";
let f: Box<f64> = Box::new(3.14);
```

---

## Section B — Write It Yourself

### Q2 — Compare allocation costs (Roadmap Practice Task)
Write two versions of a function that sums numbers 0..N: one using stack-only variables, one creating a `Box<i64>` each iteration. Time both with `Instant::now()`.

### Q3 — Pre-allocation
Demonstrate the difference between `Vec::new()` with repeated `push` vs `Vec::with_capacity(n)` by printing capacity changes.

---

## Section C — True or False?

### Q4
1. Stack allocation requires a system call to the OS allocator.
2. `String` stores its metadata (ptr, len, cap) on the stack.
3. `Option<Box<T>>` is larger than `Box<T>`.
4. Each thread gets its own stack.
5. `Vec::with_capacity(n)` avoids reallocations if you stay within `n`.
6. `std::mem::size_of::<String>()` returns the size of the string data.

---

*Stack vs Heap: the foundation of Rust performance! 🦀*
