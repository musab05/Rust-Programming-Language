# 🧪 Lesson 62 — Questions: Declarative Macros (MC1)

> **Lesson:** [lesson_62_macros.md](./lesson_62_macros.md)  
> **Answers:** [lesson_62_answers.md](./lesson_62_answers.md)

---

## Section A — Predict the Output

### Q1
```rust
macro_rules! double {
    ($x:expr) => { $x * 2 };
}
fn main() { println!("{}", double!(3 + 1)); }
```

### Q2
```rust
macro_rules! count {
    () => { 0 };
    ($x:expr $(, $rest:expr)*) => { 1 + count!($($rest),*) };
}
fn main() { println!("{}", count!(a, b, c, d)); }
```

---

## Section B — Write It Yourself

### Q3 — hashmap! macro (Roadmap Practice Task)
Write a `hashmap!` macro that creates a `HashMap` from key-value pairs:
```rust
let m = hashmap!{ "a" => 1, "b" => 2 };
```

### Q4 — debug_print! macro
Write a `debug_print!` macro that prints variable names alongside their debug values. Usage: `debug_print!(x, y, z)` → prints `x = 42`, etc.

### Q5 — min! macro
Write a `min!` macro that accepts two OR more expressions and returns the minimum.

---

## Section C — True or False?

### Q6
1. Macros are expanded at runtime.
2. `$x:ident` matches an identifier like `my_var`.
3. `$($x:expr),*` matches zero or more comma-separated expressions.
4. Rust macros are unhygienic (can capture outer variables accidentally).
5. `stringify!(expr)` converts an expression to a string literal at compile time.
6. `#[macro_export]` makes a macro available outside its crate.

---

*macro_rules!: Rust's compile-time code generator! 🦀*
