# 🧪 Lesson 66 — Questions: Advanced Traits (AL2)

> **Lesson:** [lesson_66_advanced_traits.md](./lesson_66_advanced_traits.md)  
> **Answers:** [lesson_66_answers.md](./lesson_66_answers.md)

---

## Section A — Predict the Output

### Q1
```rust
trait A { fn greet(&self) -> &str { "A" } }
trait B { fn greet(&self) -> &str { "B" } }
struct S;
impl A for S {}
impl B for S {}
impl S { fn greet(&self) -> &str { "S" } }
fn main() { println!("{} {} {}", S.greet(), A::greet(&S), <S as B>::greet(&S)); }
```

---

## Section B — Write It Yourself

### Q2 — Newtype pattern (Roadmap Practice Task)
Create a `Celsius(f64)` newtype and a `Fahrenheit(f64)` newtype. Implement `From<Celsius>` for `Fahrenheit`. Demonstrate that you can't accidentally mix them.

### Q3 — Extension trait
Write a `VecExt` trait that adds `fn median(&self) -> f64` to `Vec<f64>`. Use it.

### Q4 — Type-state
Model a `Connection` with states `Disconnected`, `Connected`, `Authenticated`. Ensure `send_data` is only callable in `Authenticated` state.

### Q5 — Blanket impl
Create a trait `Summarize` with method `fn summary(&self) -> String`. Write a blanket implementation for all `T: std::fmt::Debug`.

---

## Section C — True or False?

### Q6
1. Supertraits require the implementing type to also implement the super trait.
2. Blanket implementations apply to every type regardless of bounds.
3. The newtype pattern has runtime overhead.
4. Sealed traits prevent implementations outside the defining crate.
5. `<Type as Trait>::method()` is the fully qualified syntax.
6. Extension traits can add methods to types from other crates.

---

*Advanced traits: compile-time guarantees are the best guarantees! 🦀*
