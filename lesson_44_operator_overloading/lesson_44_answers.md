# ✅ Lesson 44 — Answers: Operator Overloading (T7)

---

## Section A

### A1
```
Num(10)
```
`Num(3) + Num(7)` calls `Add::add(Num(3), Num(7))` → `Num(10)`.

### A2
```
Val(-42.0)
```
`-Val(42.0)` calls `Neg::neg(Val(42.0))` → `Val(-42.0)`.

---

## Section B

### A3
```rust
use std::ops::Add;

#[derive(Debug, Clone, Copy)]
struct Vector2D { x: f64, y: f64 }

impl Add for Vector2D {
    type Output = Vector2D;
    fn add(self, rhs: Vector2D) -> Vector2D {
        Vector2D { x: self.x + rhs.x, y: self.y + rhs.y }
    }
}

fn main() {
    let a = Vector2D { x: 1.0, y: 2.0 };
    let b = Vector2D { x: 3.0, y: 4.0 };
    println!("{:?}", a + b);  // Vector2D { x: 4.0, y: 6.0 }
}
```

### A4
```rust
use std::ops::{Add, Sub};
use std::fmt;

#[derive(Debug, Clone, Copy)]
struct Money { cents: i64 }

impl Money {
    fn new(dollars: i64, cents: i64) -> Self { Money { cents: dollars * 100 + cents } }
}

impl Add for Money {
    type Output = Money;
    fn add(self, rhs: Money) -> Money { Money { cents: self.cents + rhs.cents } }
}

impl Sub for Money {
    type Output = Money;
    fn sub(self, rhs: Money) -> Money { Money { cents: self.cents - rhs.cents } }
}

impl fmt::Display for Money {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        let sign = if self.cents < 0 { "-" } else { "" };
        let abs = self.cents.abs();
        write!(f, "{}${}.{:02}", sign, abs / 100, abs % 100)
    }
}

fn main() {
    let a = Money::new(10, 50);
    let b = Money::new(3, 75);
    println!("{} + {} = {}", a, b, a + b);  // $10.50 + $3.75 = $14.25
    println!("{} - {} = {}", a, b, a - b);  // $10.50 - $3.75 = $6.75
}
```

### A5
```rust
use std::ops::Mul;

#[derive(Debug, Clone, Copy)]
struct Vec2 { x: f64, y: f64 }

impl Mul<f64> for Vec2 {
    type Output = Vec2;
    fn mul(self, s: f64) -> Vec2 { Vec2 { x: self.x * s, y: self.y * s } }
}

impl Mul<Vec2> for f64 {
    type Output = Vec2;
    fn mul(self, v: Vec2) -> Vec2 { Vec2 { x: self * v.x, y: self * v.y } }
}

fn main() {
    let v = Vec2 { x: 1.0, y: 2.0 };
    println!("{:?}", v * 3.0);    // Vec2 { x: 3.0, y: 6.0 }
    println!("{:?}", 3.0 * v);    // Vec2 { x: 3.0, y: 6.0 }
}
```

---

## Section C

### A6
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | `+` desugars to `Add::add(a, b)` |
| 2 | **False** | `Output` can be any type — e.g., adding two vectors could return a scalar |
| 3 | **True** | `Add<Rhs>` is generic over `Rhs`, so you can use any type |
| 4 | **True** | `Index::index` returns `&Self::Output` |
| 5 | **False** | They are independent traits |

### A7
`Add::add(self, rhs)` takes `self` by value, which moves (or copies) the operand. For `Copy` types this is fine — the values are copied. For non-`Copy` types, `a + b` **consumes** both `a` and `b`. This is why math types typically derive `Copy`. For heap types, you'd implement `Add` for `&MyType` references instead.

---

## 🏆 Lesson 44 Complete!

**Next up:** [Lesson 45 — Associated Types](../lesson_45_associated_types/lesson_45_associated_types.md) 🦀
