# 📘 Lesson 44 — Operator Overloading (T7)

> **Series:** Rust From Zero · Intermediate Level  
> **Roadmap ID:** T7 · Category: 🔷 Traits  
> **Previous:** [Lesson 43 — Standard Traits](../lesson_43_standard_traits/lesson_43_standard_traits.md)  
> **Next:** [Lesson 45 — Associated Types](../lesson_45_associated_types/lesson_45_associated_types.md)  
> **Practice:** [Questions](./lesson_44_questions.md) · [Answers](./lesson_44_answers.md)  
> **Practice Task:** Implement Add for a Vector2D struct

---

## Table of Contents

1. [How Operator Overloading Works](#1-how-operator-overloading-works)
2. [Add — The + Operator](#2-add--the--operator)
3. [Sub, Mul, Div](#3-sub-mul-div)
4. [Neg — Unary Minus](#4-neg--unary-minus)
5. [Index — The [] Operator](#5-index--the--operator)
6. [PartialEq — The == Operator](#6-partialeq--the--operator)
7. [Mixed Types in Operations](#7-mixed-types-in-operations)
8. [Real-World Example: Vector2D](#8-real-world-example-vector2d)
9. [Summary Cheat Sheet](#9-summary-cheat-sheet)

---

## 1. How Operator Overloading Works

Operators in Rust are syntactic sugar for trait method calls:

```rust
// a + b    →  std::ops::Add::add(a, b)
// a - b    →  std::ops::Sub::sub(a, b)
// a * b    →  std::ops::Mul::mul(a, b)
// -a       →  std::ops::Neg::neg(a)
// a[i]     →  std::ops::Index::index(&a, i)
// a == b   →  std::cmp::PartialEq::eq(&a, &b)
```

You implement the corresponding trait to make operators work with your types.

---

## 2. Add — The + Operator

```rust
use std::ops::Add;

#[derive(Debug, Clone, Copy)]
struct Point {
    x: f64,
    y: f64,
}

impl Add for Point {
    type Output = Point;  // a + b returns a Point

    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

fn main() {
    let a = Point { x: 1.0, y: 2.0 };
    let b = Point { x: 3.0, y: 4.0 };
    let c = a + b;
    println!("{:?}", c);  // Point { x: 4.0, y: 6.0 }

    // Because we derived Copy, a and b are still valid
    println!("a = {:?}, b = {:?}", a, b);
}
```

---

## 3. Sub, Mul, Div

```rust
use std::ops::{Sub, Mul, Div};

#[derive(Debug, Clone, Copy)]
struct Vec2 { x: f64, y: f64 }

impl Vec2 {
    fn new(x: f64, y: f64) -> Self { Vec2 { x, y } }
    fn length(&self) -> f64 { (self.x * self.x + self.y * self.y).sqrt() }
}

impl Sub for Vec2 {
    type Output = Vec2;
    fn sub(self, rhs: Vec2) -> Vec2 {
        Vec2 { x: self.x - rhs.x, y: self.y - rhs.y }
    }
}

// Scalar multiplication: Vec2 * f64
impl Mul<f64> for Vec2 {
    type Output = Vec2;
    fn mul(self, scalar: f64) -> Vec2 {
        Vec2 { x: self.x * scalar, y: self.y * scalar }
    }
}

// Scalar division: Vec2 / f64
impl Div<f64> for Vec2 {
    type Output = Vec2;
    fn div(self, scalar: f64) -> Vec2 {
        Vec2 { x: self.x / scalar, y: self.y / scalar }
    }
}

fn main() {
    let a = Vec2::new(10.0, 20.0);
    let b = Vec2::new(3.0, 4.0);

    println!("a - b = {:?}", a - b);        // Vec2 { x: 7.0, y: 16.0 }
    println!("b * 3 = {:?}", b * 3.0);      // Vec2 { x: 9.0, y: 12.0 }
    println!("a / 2 = {:?}", a / 2.0);      // Vec2 { x: 5.0, y: 10.0 }
    println!("|b| = {:.2}", b.length());     // 5.00
}
```

---

## 4. Neg — Unary Minus

```rust
use std::ops::Neg;

#[derive(Debug, Clone, Copy)]
struct Vec2 { x: f64, y: f64 }

impl Neg for Vec2 {
    type Output = Vec2;
    fn neg(self) -> Vec2 {
        Vec2 { x: -self.x, y: -self.y }
    }
}

fn main() {
    let v = Vec2 { x: 3.0, y: -4.0 };
    let neg_v = -v;
    println!("{:?}", neg_v);  // Vec2 { x: -3.0, y: 4.0 }
}
```

---

## 5. Index — The [] Operator

```rust
use std::ops::Index;

struct Matrix {
    data: Vec<Vec<f64>>,
    rows: usize,
    cols: usize,
}

impl Matrix {
    fn new(rows: usize, cols: usize) -> Self {
        Matrix {
            data: vec![vec![0.0; cols]; rows],
            rows,
            cols,
        }
    }
}

// m[row] returns a &Vec<f64> (the row)
impl Index<usize> for Matrix {
    type Output = Vec<f64>;

    fn index(&self, row: usize) -> &Vec<f64> {
        &self.data[row]
    }
}

// m[(row, col)] returns a &f64
impl Index<(usize, usize)> for Matrix {
    type Output = f64;

    fn index(&self, (row, col): (usize, usize)) -> &f64 {
        &self.data[row][col]
    }
}

fn main() {
    let mut m = Matrix::new(3, 3);
    m.data[0] = vec![1.0, 2.0, 3.0];
    m.data[1] = vec![4.0, 5.0, 6.0];
    m.data[2] = vec![7.0, 8.0, 9.0];

    println!("Row 1: {:?}", m[1]);       // [4.0, 5.0, 6.0]
    println!("(2,1): {}", m[(2, 1)]);     // 8.0
}
```

---

## 6. PartialEq — The == Operator

```rust
#[derive(Debug)]
struct ApproxFloat {
    value: f64,
    epsilon: f64,
}

impl ApproxFloat {
    fn new(value: f64) -> Self {
        ApproxFloat { value, epsilon: 1e-10 }
    }
}

impl PartialEq for ApproxFloat {
    fn eq(&self, other: &Self) -> bool {
        (self.value - other.value).abs() < self.epsilon
    }
}

fn main() {
    let a = ApproxFloat::new(0.1 + 0.2);
    let b = ApproxFloat::new(0.3);

    // Normal f64: 0.1 + 0.2 != 0.3 (floating point!)
    println!("f64: {} == {} → {}", 0.1 + 0.2, 0.3, (0.1_f64 + 0.2) == 0.3);

    // ApproxFloat: handles it
    println!("Approx: {:?} == {:?} → {}", a, b, a == b);  // true
}
```

---

## 7. Mixed Types in Operations

You can implement operators between different types:

```rust
use std::ops::Mul;

#[derive(Debug, Clone, Copy)]
struct Vec2 { x: f64, y: f64 }

// Vec2 * f64
impl Mul<f64> for Vec2 {
    type Output = Vec2;
    fn mul(self, scalar: f64) -> Vec2 {
        Vec2 { x: self.x * scalar, y: self.y * scalar }
    }
}

// f64 * Vec2  (reverse order)
impl Mul<Vec2> for f64 {
    type Output = Vec2;
    fn mul(self, vec: Vec2) -> Vec2 {
        Vec2 { x: self * vec.x, y: self * vec.y }
    }
}

fn main() {
    let v = Vec2 { x: 1.0, y: 2.0 };
    println!("{:?}", v * 3.0);    // Vec2 { x: 3.0, y: 6.0 }
    println!("{:?}", 3.0 * v);    // Vec2 { x: 3.0, y: 6.0 }
}
```

---

## 8. Real-World Example: Vector2D

The roadmap practice task:

```rust
use std::fmt;
use std::ops::{Add, Sub, Mul, Neg};

#[derive(Clone, Copy, PartialEq)]
struct Vector2D {
    x: f64,
    y: f64,
}

impl Vector2D {
    fn new(x: f64, y: f64) -> Self { Vector2D { x, y } }
    fn zero() -> Self { Vector2D { x: 0.0, y: 0.0 } }
    fn length(&self) -> f64 { (self.x * self.x + self.y * self.y).sqrt() }
    fn normalized(&self) -> Self { let l = self.length(); *self * (1.0 / l) }
    fn dot(&self, other: &Self) -> f64 { self.x * other.x + self.y * other.y }
}

impl Add for Vector2D {
    type Output = Self;
    fn add(self, rhs: Self) -> Self { Vector2D::new(self.x + rhs.x, self.y + rhs.y) }
}

impl Sub for Vector2D {
    type Output = Self;
    fn sub(self, rhs: Self) -> Self { Vector2D::new(self.x - rhs.x, self.y - rhs.y) }
}

impl Mul<f64> for Vector2D {
    type Output = Self;
    fn mul(self, s: f64) -> Self { Vector2D::new(self.x * s, self.y * s) }
}

impl Neg for Vector2D {
    type Output = Self;
    fn neg(self) -> Self { Vector2D::new(-self.x, -self.y) }
}

impl fmt::Display for Vector2D {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "({:.2}, {:.2})", self.x, self.y)
    }
}

impl fmt::Debug for Vector2D {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "Vec2D{self}")
    }
}

fn main() {
    let a = Vector2D::new(3.0, 4.0);
    let b = Vector2D::new(1.0, 2.0);

    println!("a + b = {}", a + b);
    println!("a - b = {}", a - b);
    println!("a * 2 = {}", a * 2.0);
    println!("-a    = {}", -a);
    println!("|a|   = {:.2}", a.length());
    println!("a·b   = {:.2}", a.dot(&b));
    println!("norm  = {}", a.normalized());
}
```

---

## 9. Summary Cheat Sheet

```
ARITHMETIC OPERATORS
────────────────────────────────────────────────────────────
Add       +      fn add(self, rhs: Rhs) -> Self::Output
Sub       -      fn sub(self, rhs: Rhs) -> Self::Output
Mul       *      fn mul(self, rhs: Rhs) -> Self::Output
Div       /      fn div(self, rhs: Rhs) -> Self::Output
Rem       %      fn rem(self, rhs: Rhs) -> Self::Output

UNARY OPERATORS
────────────────────────────────────────────────────────────
Neg       -x     fn neg(self) -> Self::Output
Not       !x     fn not(self) -> Self::Output

INDEXING
────────────────────────────────────────────────────────────
Index     x[i]      fn index(&self, idx) -> &Self::Output
IndexMut  x[i]=v    fn index_mut(&mut self, idx) -> &mut Self::Output

ASSIGNMENT OPERATORS
────────────────────────────────────────────────────────────
AddAssign   +=    fn add_assign(&mut self, rhs: Rhs)
SubAssign   -=    fn sub_assign(&mut self, rhs: Rhs)

ALL OPERATORS USE type Output
────────────────────────────────────────────────────────────
impl Add for MyType {
    type Output = MyType;  // what a + b returns
    fn add(self, rhs: MyType) -> MyType { ... }
}
```

---

## What's Next?

**Lesson 45 — Associated Types** — Understand `type Item` in traits like `Iterator`, and when to use associated types versus generics.

## Further Reading
- [std::ops module](https://doc.rust-lang.org/std/ops/index.html)
- [The Rust Book — Operator Overloading](https://doc.rust-lang.org/book/ch19-03-advanced-traits.html#default-generic-type-parameters-and-operator-overloading)

---

*Operators are just trait methods in disguise! 🦀*
