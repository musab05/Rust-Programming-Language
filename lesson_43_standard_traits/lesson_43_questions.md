# 🧪 Lesson 43 — Questions: Standard Traits (T6)

> **Lesson:** [lesson_43_standard_traits.md](./lesson_43_standard_traits.md)  
> **Answers:** [lesson_43_answers.md](./lesson_43_answers.md)

---

## Section A — Quick Answer

### Q1
Which trait enables `println!("{}", val)`? Which enables `println!("{:?}", val)`?

### Q2
Can you `#[derive(Display)]`? Why or why not?

### Q3
What is the relationship between `Clone` and `Copy`? Can you have `Copy` without `Clone`?

### Q4
Why can't `String` implement `Copy`?

---

## Section B — Write It Yourself

### Q5 — Color struct (Roadmap Practice Task)
Create a `Color` struct with `r`, `g`, `b` (all `u8`). Implement:
1. `Display` — format as `#RRGGBB` hex
2. `Debug`, `Clone`, `Copy` (derive)
3. `Default` — black (0, 0, 0)
4. `PartialEq` + `Eq` (derive)
5. `From<(u8, u8, u8)>`

### Q6 — Temperature
Create `Celsius(f64)` and `Fahrenheit(f64)`. Implement `From<Celsius>` for `Fahrenheit` and vice versa. Implement `Display` for both.

### Q7 — Custom Default
Create a `DatabaseConfig` with `host`, `port`, `database`, `pool_size`. Implement `Default` with production-ready defaults.

---

## Section C — Deep Understanding

### Q8 — True or False?
1. `Eq` adds new methods beyond what `PartialEq` provides.
2. If you implement `From<A> for B`, you get `Into<B> for A` automatically.
3. `Hash` can be derived only if all fields implement `Hash`.
4. `f64` implements `Eq`.
5. `Default::default()` for `bool` returns `false`.
6. `Display` is required for the `.to_string()` method.

### Q9
You have a `User` struct with an `id: u64` and `email: String`. You want to use it as a `HashMap` key based only on `id`. How would you implement `PartialEq`, `Eq`, and `Hash` manually?

---

*Standard traits: the shared language of all Rust types! 🦀*
