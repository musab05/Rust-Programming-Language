# 🧪 Lesson 85 — Questions: Never Type (!) (AT3)

> **Lesson:** [lesson_85_never_type.md](./lesson_85_never_type.md)  
> **Answers:** [lesson_85_answers.md](./lesson_85_answers.md)

---

## Section A — Predict

### Q1
Does this compile? What's the type of `x`?
```rust
fn main() {
    let x = if true { 42 } else { panic!("nope") };
    println!("{x}");
}
```

### Q2
Does this compile?
```rust
fn foo() -> ! { loop {} }
fn main() { let x: String = foo(); }
```

---

## Section B — Write It Yourself

### Q3 — Fatal error function (Roadmap Practice Task)
Write a function `fn fatal(msg: &str) -> !` that prints an error and exits. Use it in a `match` arm where the other arm returns a `String`.

### Q4 — Diverging explanation
List 5 expressions/macros that have type `!` and explain one practical use case for each.

---

## Section C — True or False?

### Q5
1. `!` can coerce into any type.
2. `loop { }` (without break) has type `!`.
3. A function returning `!` can be used wherever any return type is expected.
4. `continue` in a loop has type `!`.
5. `todo!()` returns `()`.
6. `std::process::exit()` returns `!`.

---

*Never type: the bottom of the type hierarchy! 🦀*
