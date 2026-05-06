# 🧪 Lesson 65 — Questions: Advanced Lifetimes (AL1)

> **Lesson:** [lesson_65_advanced_lifetimes.md](./lesson_65_advanced_lifetimes.md)  
> **Answers:** [lesson_65_answers.md](./lesson_65_answers.md)

---

## Section A — Compile or Not?

### Q1
```rust
fn first<'a, 'b>(s1: &'a str, s2: &'b str) -> &'a str { s1 }
fn main() {
    let s1 = String::from("hello");
    let result;
    { let s2 = String::from("world"); result = first(&s1, &s2); }
    println!("{result}");
}
```

### Q2
```rust
fn pick<'a>(s1: &'a str, s2: &'a str) -> &'a str { s1 }
fn main() {
    let s1 = String::from("hello");
    let result;
    { let s2 = String::from("world"); result = pick(&s1, &s2); }
    println!("{result}");
}
```

### Q3
```rust
fn requires_static<T: 'static>(val: T) {}
fn main() {
    let s = String::from("hello");
    requires_static(&s);
}
```

---

## Section B — Write It Yourself

### Q4 — Multiple lifetimes (Roadmap Practice Task)
Write a function `extract_field<'data, 'schema>` where the return borrows from `data` only (not `schema`). Demonstrate that the schema can be dropped before the result is used.

### Q5 — HRTB
Write a `Processor` struct that stores a `Box<dyn for<'a> Fn(&'a str) -> String>`. Implement a `process` method.

### Q6 — Lifetime in struct iterator
Create a `LineIter<'a>` struct that iterates over lines of a `&'a str`. Implement `Iterator` with `Item = &'a str`.

---

## Section C — True or False?

### Q7
1. `T: 'static` means the value lives for the entire program.
2. `&'static str` is the type of string literals.
3. `for<'a> Fn(&'a str)` means "this function works for any lifetime."
4. If `'b: 'a`, then `'b` is at least as long as `'a`.
5. Async tasks with `tokio::spawn` require `'static` data.
6. `'_` lets the compiler infer the lifetime.

### Q8
Explain the difference between `T: 'static` and `&'static T`.

---

*Advanced lifetimes: the final frontier of Rust's ownership system! 🦀*
