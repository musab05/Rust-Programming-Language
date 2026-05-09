# 🧪 Lesson 72 — Questions: Type Aliases (AT1)

> **Lesson:** [lesson_72_type_aliases.md](./lesson_72_type_aliases.md)  
> **Answers:** [lesson_72_answers.md](./lesson_72_answers.md)

---

## Section A — Compile or Not?

### Q1
```rust
type Score = i32;
fn add_scores(a: Score, b: i32) -> Score { a + b }
fn main() { println!("{}", add_scores(10, 20)); }
```

### Q2
```rust
type Callback = Box<dyn Fn(i32) -> i32>;
fn apply(f: &Callback, x: i32) -> i32 { f(x) }
fn main() {
    let double: Callback = Box::new(|x| x * 2);
    println!("{}", apply(&double, 5));
}
```

---

## Section B — Write It Yourself

### Q3 — Complex closure alias (Roadmap Practice Task)
Create type aliases for: a `HashMap<String, Vec<(u32, String, bool)>>` and a closure `Box<dyn Fn(&str) -> Result<String, String>>`. Use them in function signatures.

### Q4 — Result alias
Create an `AppError` enum and a `type AppResult<T>` alias. Use it in 3 functions.

---

## Section C — True or False?

### Q5
1. Type aliases create a new, distinct type at compile time.
2. `type Meters = f64` allows assigning an `f64` to a `Meters` variable.
3. `std::io::Result<T>` is a type alias for `Result<T, io::Error>`.
4. Type aliases can have generic parameters.
5. Type aliases provide type safety like newtypes.

---

*Type aliases: clarity is kind! 🦀*
