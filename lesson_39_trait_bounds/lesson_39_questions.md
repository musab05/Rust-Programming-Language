# 🧪 Lesson 39 — Questions: Trait Bounds (T2)

> **Lesson:** [lesson_39_trait_bounds.md](./lesson_39_trait_bounds.md)  
> **Answers:** [lesson_39_answers.md](./lesson_39_answers.md)

---

## Section A — Predict: Compile or Not?

### Q1
```rust
fn print_it<T>(item: T) {
    println!("{item}");
}
fn main() { print_it(42); }
```

### Q2
```rust
use std::fmt::Display;
fn print_it<T: Display>(item: T) {
    println!("{item}");
}
fn main() { print_it(42); }
```

### Q3
```rust
fn add_them<T: std::ops::Add<Output = T>>(a: T, b: T) -> T {
    a + b
}
fn main() { println!("{}", add_them(5, 10)); }
```

---

## Section B — Write It Yourself

### Q4 — largest() (Roadmap Practice Task)
Write a generic `largest<T>` function that returns the largest item from a slice. What trait bounds do you need? Test with integers and characters.

### Q5 — print_if_long
Write `fn print_if_long<T: Display>(items: &[T], min_len: usize)` that prints only items whose `to_string()` representation is at least `min_len` characters.

### Q6 — where clause
Rewrite this signature using a `where` clause:
```rust
fn process<T: Display + Clone + Debug, U: PartialOrd + Hash>(a: T, b: U) -> String
```

### Q7 — Conditional methods
Create a `Container<T>` struct. Add:
- `new()` available for all T
- `print_all()` available only when `T: Display`
- `sort_contents()` available only when `T: Ord`

---

## Section C — Deep Understanding

### Q8 — True or False?
1. `T: PartialOrd` means T can be compared with `<` and `>`.
2. `where` clauses change the behavior of trait bounds.
3. Blanket implementations implement a trait for all types unconditionally.
4. `f64` implements `Ord`.
5. `impl Trait` in parameter position is syntactic sugar for a generic with trait bound.

### Q9
Why does `largest` need `Copy` or `Clone` if it returns `T` instead of `&T`? What happens without it?

---

*Trait bounds: telling the compiler exactly what your generics can do! 🦀*
