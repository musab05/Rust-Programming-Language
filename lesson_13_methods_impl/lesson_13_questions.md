# 🧪 Lesson 13 — Questions: Methods & `impl` (S2)

> **Lesson:** [lesson_13_methods_impl.md](./lesson_13_methods_impl.md)  
> **Answers:** [lesson_13_answers.md](./lesson_13_answers.md)

---

## Section A — Predict the Output

### Q1
```rust
struct Temp { celsius: f64 }
impl Temp {
    fn new(c: f64) -> Self { Temp { celsius: c } }
    fn to_f(&self) -> f64  { self.celsius * 9.0/5.0 + 32.0 }
    fn heat(&mut self, d: f64) { self.celsius += d; }
}
fn main() {
    let mut t = Temp::new(0.0);
    println!("{}", t.to_f());
    t.heat(100.0);
    println!("{}", t.to_f());
}
```

### Q2
```rust
struct Counter { n: u32 }
impl Counter {
    fn new() -> Self { Counter { n: 0 } }
    fn inc(mut self) -> Self { self.n += 1; self }
    fn val(&self) -> u32 { self.n }
}
fn main() {
    let c = Counter::new().inc().inc().inc();
    println!("{}", c.val());
}
```

### Q3
```rust
#[derive(Debug)]
struct Rect { w: f64, h: f64 }
impl Rect {
    fn new(w: f64, h: f64) -> Self { Rect { w, h } }
    fn scale(mut self, f: f64) -> Self { self.w *= f; self.h *= f; self }
    fn area(&self) -> f64 { self.w * self.h }
}
fn main() {
    let r = Rect::new(3.0, 4.0).scale(2.0);
    println!("{}", r.area());
    println!("{r:?}");
}
```

### Q4
```rust
use std::fmt;
struct Point { x: i32, y: i32 }
impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "({},{})", self.x, self.y)
    }
}
fn main() {
    let p = Point { x: 3, y: -1 };
    println!("{p}");
    println!("x={}", p.x);
}
```

---

## Section B — Will It Compile?

### Q5
```rust
struct Box3D { l: f64, w: f64, h: f64 }
impl Box3D {
    fn volume(&self) -> f64 { self.l * self.w * self.h }
    fn stretch(&mut self, factor: f64) { self.l *= factor; }
}
fn main() {
    let b = Box3D { l: 2.0, w: 3.0, h: 4.0 };
    b.stretch(2.0);   // b is not mut
    println!("{}", b.volume());
}
```

### Q6
```rust
struct Pair { a: i32, b: i32 }
impl Pair {
    fn swap(self) -> Self { Pair { a: self.b, b: self.a } }
}
fn main() {
    let p = Pair { a: 1, b: 2 };
    let q = p.swap();
    println!("{} {}", p.a, q.a);  // p used after move
}
```

### Q7
```rust
struct Greeter { name: String }
impl Greeter {
    fn new(name: &str) -> Self { Greeter { name: name.to_string() } }
    fn greet(&self) -> String { format!("Hello, {}!", self.name) }
}
fn main() {
    let g = Greeter::new("Rust");
    let s = g.greet();
    println!("{s}");
    println!("{}", g.name); // still valid?
}
```

---

## Section C — Fix the Bugs

### Q8 — Wrong self forms (4 bugs)
```rust
struct Stack { items: Vec<i32> }
impl Stack {
    fn new() -> Stack { Stack { items: vec![] } }
    fn push(self, v: i32) { self.items.push(v); }       // Bug 1
    fn pop(self) -> Option<i32> { self.items.pop() }    // Bug 2
    fn peek(&mut self) -> Option<&i32> { self.items.last() } // Bug 3
    fn len(&mut self) -> usize { self.items.len() }     // Bug 4
}
fn main() {
    let mut s = Stack::new();
    s.push(1); s.push(2); s.push(3);
    println!("{:?} len={}", s.peek(), s.len());
    println!("{:?}", s.pop());
}
```

### Q9 — Fix the constructor
```rust
struct Circle { radius: f64 }
impl Circle {
    fn new(r: f64) -> Circle {
        if r <= 0.0 { panic!("negative radius"); }
        Circle { r }   // Bug: wrong field name
    }
    fn area(&self) -> f64 { std::f64::consts::PI * self.radius * self.radius }
}
fn main() {
    let c = Circle::new(5.0);
    println!("{:.2}", c.area());
}
```

### Q10 — Implement the missing trait
```rust
struct Rgb { r: u8, g: u8, b: u8 }
fn main() {
    let red = Rgb { r: 255, g: 0, b: 0 };
    println!("{red}");     // ERROR: no Display
    println!("{red:?}");   // ERROR: no Debug
}
```
Implement `Display` (prints `"rgb(255, 0, 0)"`) and add `#[derive(Debug)]`.

---

## Section D — Write It Yourself

### Q11 — Roadmap Practice Task: Rectangle with `new()` and `scale()`
Build the full Rectangle from the roadmap practice task:
- `fn new(width: f64, height: f64) -> Self`
- `fn square(side: f64) -> Self`
- `fn area(&self) -> f64`
- `fn perimeter(&self) -> f64`
- `fn is_square(&self) -> bool`
- `fn scale(&mut self, factor: f64)` — mutates in place
- `fn scaled(self, factor: f64) -> Self` — returns new scaled rectangle
- Implement `Display`: `"5.0 × 3.0"`
- Derive `Debug`

Test `new()`, `square()`, `scale()`, `scaled()`, chaining.

### Q12 — Stack with full API
Build a generic-ish (use `i32` for now) stack:
- `fn new() -> Self`
- `fn push(&mut self, val: i32)`
- `fn pop(&mut self) -> Option<i32>`
- `fn peek(&self) -> Option<i32>` — copy the top value
- `fn len(&self) -> usize`
- `fn is_empty(&self) -> bool`
- `fn clear(&mut self)`
- Implement `Display`: prints the stack top-to-bottom like `"[3, 2, 1]"` (top first)

### Q13 — Vec2 with operator chaining
Build a `Vec2 { x: f64, y: f64 }`:
- `fn new(x: f64, y: f64) -> Self`
- `fn zero() -> Self`
- `fn length(&self) -> f64`
- `fn normalize(&self) -> Self`
- `fn dot(&self, other: &Vec2) -> f64`
- `fn add(&self, other: &Vec2) -> Self`
- `fn scale(&self, factor: f64) -> Self`
- Implement `Display`: `"Vec2(1.00, 2.00)"`
- Implement `Default` (returns `Vec2::zero()`)

### Q14 — Counter with builder methods
```rust
struct Counter { value: i32, step: i32, max: Option<i32> }
```
Implement:
- `fn new(start: i32) -> Self` (step=1, no max)
- `fn with_step(mut self, step: i32) -> Self`
- `fn with_max(mut self, max: i32) -> Self`
- `fn tick(&mut self)` — advance by step, stop at max if set
- `fn value(&self) -> i32`
- `fn is_done(&self) -> bool` — true if at max

Test by building a counter that counts 0→10 by 2s.

### Q15 — Full program: RGB color type
Build a `Color { r: u8, g: u8, b: u8 }`:
- `fn new(r: u8, g: u8, b: u8) -> Self`
- `fn from_hex(hex: &str) -> Self` — parse `"#FF8800"` format
- `fn to_hex(&self) -> String` — produce `"#FF8800"`
- `fn brightness(&self) -> f64` — (r+g+b)/3 as 0.0..=1.0
- `fn invert(&self) -> Self`
- `fn blend(&self, other: &Self) -> Self` — average of the two
- Implement `Display`: `"rgb(255, 128, 0)"`
- Implement `PartialEq` manually (or derive it)

---

## Section E — Deep Understanding

### Q16 — True or False?
1. `self` (consuming) can be called on a non-`mut` binding.
2. `&mut self` methods require the binding to be declared `mut`.
3. `Self` is always identical to the name of the struct.
4. You can have two `impl` blocks for the same struct in the same file.
5. A method named `fn area(&self)` can be called as `Rectangle::area(&rect)`.
6. `impl fmt::Display for MyType` is the only way to enable `{}` formatting.
7. The `Default` trait can be derived if every field implements `Default`.
8. `impl From<A> for B` automatically provides `B::from(a)` and `a.into()`.
9. Methods in different `impl` blocks for the same type share `self`.
10. An associated function with no `self` parameter must be called with `::`.
