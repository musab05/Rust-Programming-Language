# 🧪 Lesson 44 — Questions: Operator Overloading (T7)

> **Lesson:** [lesson_44_operator_overloading.md](./lesson_44_operator_overloading.md)  
> **Answers:** [lesson_44_answers.md](./lesson_44_answers.md)

---

## Section A — Predict the Output

### Q1
```rust
use std::ops::Add;
#[derive(Debug, Copy, Clone)]
struct Num(i32);
impl Add for Num {
    type Output = Num;
    fn add(self, rhs: Num) -> Num { Num(self.0 + rhs.0) }
}
fn main() { println!("{:?}", Num(3) + Num(7)); }
```

### Q2
```rust
use std::ops::Neg;
#[derive(Debug, Copy, Clone)]
struct Val(f64);
impl Neg for Val {
    type Output = Val;
    fn neg(self) -> Val { Val(-self.0) }
}
fn main() { println!("{:?}", -Val(42.0)); }
```

---

## Section B — Write It Yourself

### Q3 — Vector2D (Roadmap Practice Task)
Implement `Add` for a `Vector2D { x: f64, y: f64 }`. Test by adding two vectors.

### Q4 — Money type
Create `Money { cents: i64 }`. Implement `Add`, `Sub`, and `Display` (format as `$X.XX`). Test with additions and subtractions.

### Q5 — Scalar multiplication both ways
For a `Vec2` struct, implement `Vec2 * f64` AND `f64 * Vec2`.

---

## Section C — Deep Understanding

### Q6 — True or False?
1. `a + b` is syntactic sugar for `Add::add(a, b)`.
2. The `Output` associated type in `Add` must always be `Self`.
3. You can implement `Add<String>` for your custom type.
4. `Index` returns a reference, not an owned value.
5. You must implement `Add` before you can implement `AddAssign`.

### Q7
Why does `impl Add for MyType` consume `self` (take ownership)? What are the implications?

---

*Operators + traits = elegant, readable math! 🦀*
