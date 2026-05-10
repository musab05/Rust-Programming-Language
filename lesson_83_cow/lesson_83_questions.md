# 🧪 Lesson 83 — Questions: Cow\<T\> (SP5)

> **Lesson:** [lesson_83_cow.md](./lesson_83_cow.md)  
> **Answers:** [lesson_83_answers.md](./lesson_83_answers.md)

---

## Section A — Predict

### Q1
```rust
use std::borrow::Cow;
fn f(s: &str) -> Cow<str> {
    if s.contains(' ') { Cow::Owned(s.replace(' ', "_")) }
    else { Cow::Borrowed(s) }
}
fn main() {
    let r1 = f("hello");
    let r2 = f("hello world");
    println!("{} {}", matches!(r1, Cow::Borrowed(_)), matches!(r2, Cow::Borrowed(_)));
}
```

---

## Section B — Write It Yourself

### Q2 — Conditional modification (Roadmap Practice Task)
Write a function `ensure_lowercase(s: &str) -> Cow<str>` that returns borrowed if already lowercase, owned if modification was needed.

### Q3 — HTML escape
Write `escape_html(s: &str) -> Cow<str>` that escapes `<`, `>`, `&`. Only allocate if the input actually contains those characters.

---

## Section C — True or False?

### Q4
1. `Cow::Borrowed` holds a reference and causes no allocation.
2. `Cow::Owned` holds owned data on the heap.
3. `cow.to_mut()` clones the data if it's currently `Borrowed`.
4. `Cow<str>` can be used wherever `&str` is expected (via Deref).
5. You should always use `Cow` instead of `&str` for better performance.
6. `cow.into_owned()` always clones the data.

---

*Cow: the laziest smart pointer! 🦀*
