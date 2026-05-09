# 🧪 Lesson 73 — Questions: Newtype Pattern (AT2)

> **Lesson:** [lesson_73_newtype.md](./lesson_73_newtype.md)  
> **Answers:** [lesson_73_answers.md](./lesson_73_answers.md)

---

## Section A — Compile or Not?

### Q1
```rust
struct Meters(f64);
struct Seconds(f64);
fn main() { let m = Meters(5.0); let s: Seconds = m; }
```

### Q2
```rust
type Meters = f64;
type Seconds = f64;
fn main() { let m: Meters = 5.0; let s: Seconds = m; }
```

---

## Section B — Write It Yourself

### Q3 — Validated Email (Roadmap Practice Task)
Create an `Email` newtype that validates format on construction. Implement `Display`, `as_str()`, and `domain()`.

### Q4 — Unit-safe conversions
Create `Celsius(f64)` and `Fahrenheit(f64)` newtypes. Implement `From<Celsius>` for `Fahrenheit` and vice versa. Show that they can't be mixed in function signatures.

### Q5 — Display for Vec
Use the newtype pattern to implement `Display` for `Vec<i32>`, printing values as `{1, 2, 3}`.

---

## Section C — True or False?

### Q6
1. Newtypes have runtime overhead compared to the inner type.
2. `struct Meters(f64)` creates a type distinct from `f64`.
3. Newtypes can bypass the orphan rule for trait implementation.
4. `#[repr(transparent)]` guarantees the newtype has the same memory layout as its field.
5. `impl Deref for MyType` makes the newtype behave like its inner type in some contexts.

---

*Newtypes: type safety with zero cost! 🦀*
