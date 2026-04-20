# ✅ Lesson 13 — Answers: Methods & `impl` (S2)

---

## Section A — Predict the Output

### A1
```
32
212
```
0°C → 32°F. After adding 100°C → 100°C = 212°F.

### A2
```
3
```
`Counter::new()` → 0. `.inc()` → 1. `.inc()` → 2. `.inc()` → 3. Each `inc` takes ownership and returns modified Self. Final `.val()` = 3.

### A3
```
144
Rect { w: 6.0, h: 8.0 }
```
`new(3,4)` → `.scale(2)` → Rect{w:6, h:8}. Area = 6×8 = 144.

### A4
```
(3,-1)
x=3
```
Display prints `(x,y)` format. Field access still works.

---

## Section B — Will It Compile?

### A5 — ❌ No
`b` is not `mut`. `stretch` takes `&mut self`. Fix: `let mut b = ...`

### A6 — ❌ No
`p.swap()` takes `self` — moves `p`. Then `p.a` uses the moved value. Fix: either borrow in `swap` or don't use `p` after.

### A7 — ✅ Yes
`greet` takes `&self` — borrows `g`. After the call `g` is still valid. `g.name` is accessible. Output: `Hello, Rust!\nRust`

---

## Section C — Fix the Bugs

### A8 — Wrong self forms
```rust
struct Stack { items: Vec<i32> }
impl Stack {
    fn new() -> Stack { Stack { items: vec![] } }
    fn push(&mut self, v: i32) { self.items.push(v); }       // fix 1: &mut self
    fn pop(&mut self) -> Option<i32> { self.items.pop() }    // fix 2: &mut self
    fn peek(&self) -> Option<i32> { self.items.last().copied() } // fix 3: &self + copy value
    fn len(&self) -> usize { self.items.len() }              // fix 4: &self
}
fn main() {
    let mut s = Stack::new();
    s.push(1); s.push(2); s.push(3);
    println!("{:?} len={}", s.peek(), s.len()); // Some(3) len=3
    println!("{:?}", s.pop());                   // Some(3)
}
```

### A9 — Fix the constructor
```rust
impl Circle {
    fn new(r: f64) -> Circle {
        if r <= 0.0 { panic!("negative radius"); }
        Circle { radius: r }  // fix: use field name 'radius'
    }
}
fn main() {
    let c = Circle::new(5.0);
    println!("{:.2}", c.area()); // 78.54
}
```

### A10 — Implement the missing trait
```rust
use std::fmt;

#[derive(Debug)]
struct Rgb { r: u8, g: u8, b: u8 }

impl fmt::Display for Rgb {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "rgb({}, {}, {})", self.r, self.g, self.b)
    }
}

fn main() {
    let red = Rgb { r: 255, g: 0, b: 0 };
    println!("{red}");   // rgb(255, 0, 0)
    println!("{red:?}"); // Rgb { r: 255, g: 0, b: 0 }
}
```

---

## Section D — Write It Yourself

### A11 — Rectangle (roadmap practice task)
```rust
use std::fmt;

#[derive(Debug)]
struct Rectangle { width: f64, height: f64 }

impl Rectangle {
    fn new(width: f64, height: f64) -> Self { Rectangle { width, height } }
    fn square(side: f64) -> Self { Self { width: side, height: side } }
    fn area(&self) -> f64 { self.width * self.height }
    fn perimeter(&self) -> f64 { 2.0 * (self.width + self.height) }
    fn is_square(&self) -> bool { (self.width - self.height).abs() < f64::EPSILON }
    fn scale(&mut self, factor: f64) { self.width *= factor; self.height *= factor; }
    fn scaled(mut self, factor: f64) -> Self { self.width *= factor; self.height *= factor; self }
}

impl fmt::Display for Rectangle {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "{} × {}", self.width, self.height)
    }
}

fn main() {
    let mut r = Rectangle::new(5.0, 3.0);
    println!("{r}");                      // 5 × 3
    println!("area={}", r.area());        // 15
    println!("perimeter={}", r.perimeter()); // 16

    r.scale(2.0);
    println!("after scale: {r}");         // 10 × 6

    let big = Rectangle::new(2.0, 2.0).scaled(5.0);
    println!("big: {big}");               // 10 × 10
    println!("is_square: {}", big.is_square()); // true

    let s = Rectangle::square(4.0);
    println!("{s:?}");
}
```

### A12 — Stack
```rust
use std::fmt;

struct Stack { items: Vec<i32> }

impl Stack {
    fn new() -> Self { Stack { items: vec![] } }
    fn push(&mut self, val: i32) { self.items.push(val); }
    fn pop(&mut self) -> Option<i32> { self.items.pop() }
    fn peek(&self) -> Option<i32> { self.items.last().copied() }
    fn len(&self) -> usize { self.items.len() }
    fn is_empty(&self) -> bool { self.items.is_empty() }
    fn clear(&mut self) { self.items.clear(); }
}

impl fmt::Display for Stack {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        let top_first: Vec<String> = self.items.iter().rev().map(|x| x.to_string()).collect();
        write!(f, "[{}]", top_first.join(", "))
    }
}

fn main() {
    let mut s = Stack::new();
    s.push(1); s.push(2); s.push(3);
    println!("{s}");             // [3, 2, 1]
    println!("peek={:?}", s.peek()); // Some(3)
    println!("pop={:?}", s.pop());   // Some(3)
    println!("{s}");             // [2, 1]
    s.clear();
    println!("empty={}", s.is_empty()); // true
}
```

### A13 — Vec2
```rust
use std::fmt;

#[derive(Debug, Clone)]
struct Vec2 { x: f64, y: f64 }

impl Vec2 {
    fn new(x: f64, y: f64) -> Self { Vec2 { x, y } }
    fn zero() -> Self { Vec2 { x: 0.0, y: 0.0 } }
    fn length(&self) -> f64 { (self.x*self.x + self.y*self.y).sqrt() }
    fn normalize(&self) -> Self {
        let len = self.length();
        if len == 0.0 { return Self::zero(); }
        Vec2 { x: self.x / len, y: self.y / len }
    }
    fn dot(&self, other: &Vec2) -> f64 { self.x*other.x + self.y*other.y }
    fn add(&self, other: &Vec2) -> Self { Vec2 { x: self.x+other.x, y: self.y+other.y } }
    fn scale(&self, factor: f64) -> Self { Vec2 { x: self.x*factor, y: self.y*factor } }
}

impl fmt::Display for Vec2 {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "Vec2({:.2}, {:.2})", self.x, self.y)
    }
}

impl Default for Vec2 { fn default() -> Self { Vec2::zero() } }

fn main() {
    let a = Vec2::new(3.0, 4.0);
    println!("{a}");                        // Vec2(3.00, 4.00)
    println!("length={}", a.length());      // 5
    println!("norm={}", a.normalize());
    let b = Vec2::new(1.0, 0.0);
    println!("dot={}", a.dot(&b));          // 3
    println!("add={}", a.add(&b));
    println!("scale={}", a.scale(2.0));
    println!("default={}", Vec2::default());
}
```

### A14 — Counter with builder
```rust
struct Counter { value: i32, step: i32, max: Option<i32> }

impl Counter {
    fn new(start: i32) -> Self { Counter { value: start, step: 1, max: None } }
    fn with_step(mut self, step: i32) -> Self { self.step = step; self }
    fn with_max(mut self, max: i32) -> Self { self.max = Some(max); self }
    fn tick(&mut self) {
        if self.is_done() { return; }
        self.value += self.step;
        if let Some(m) = self.max {
            if self.value > m { self.value = m; }
        }
    }
    fn value(&self) -> i32 { self.value }
    fn is_done(&self) -> bool { self.max.map_or(false, |m| self.value >= m) }
}

fn main() {
    let mut c = Counter::new(0).with_step(2).with_max(10);
    while !c.is_done() {
        c.tick();
        print!("{} ", c.value()); // 2 4 6 8 10
    }
    println!();
}
```

### A15 — RGB color
```rust
use std::fmt;

#[derive(Debug, PartialEq, Clone)]
struct Color { r: u8, g: u8, b: u8 }

impl Color {
    fn new(r: u8, g: u8, b: u8) -> Self { Color { r, g, b } }

    fn from_hex(hex: &str) -> Self {
        let hex = hex.trim_start_matches('#');
        let r = u8::from_str_radix(&hex[0..2], 16).unwrap_or(0);
        let g = u8::from_str_radix(&hex[2..4], 16).unwrap_or(0);
        let b = u8::from_str_radix(&hex[4..6], 16).unwrap_or(0);
        Color { r, g, b }
    }

    fn to_hex(&self) -> String { format!("#{:02X}{:02X}{:02X}", self.r, self.g, self.b) }

    fn brightness(&self) -> f64 {
        (self.r as f64 + self.g as f64 + self.b as f64) / (3.0 * 255.0)
    }

    fn invert(&self) -> Self { Color { r: 255-self.r, g: 255-self.g, b: 255-self.b } }

    fn blend(&self, other: &Self) -> Self {
        Color {
            r: ((self.r as u16 + other.r as u16) / 2) as u8,
            g: ((self.g as u16 + other.g as u16) / 2) as u8,
            b: ((self.b as u16 + other.b as u16) / 2) as u8,
        }
    }
}

impl fmt::Display for Color {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "rgb({}, {}, {})", self.r, self.g, self.b)
    }
}

fn main() {
    let red = Color::new(255, 0, 0);
    println!("{red}");
    println!("{}", red.to_hex());           // #FF0000
    println!("brightness={:.2}", red.brightness()); // 0.33
    println!("invert={}", red.invert());

    let orange = Color::from_hex("#FF8800");
    println!("{orange}");
    println!("blend={}", red.blend(&orange));
}
```

---

## Section E — Deep Understanding

### A16 — True or False?
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | `self` (consuming) moves the value — `mut` on the binding not needed, just ownership |
| 2 | **True** | `&mut self` needs the binding to be `let mut x = ...` |
| 3 | **True** | Inside `impl Rectangle`, `Self` = `Rectangle` exactly |
| 4 | **True** | Multiple `impl` blocks are valid and merged at compile time |
| 5 | **True** | This is Universal Function Call Syntax (UFCS) |
| 6 | **False** | You can also use `write!`/`format!` — but to make `{}` work via `println!`, you do need `impl Display` |
| 7 | **True** | `#[derive(Default)]` works if all fields implement `Default` (most primitives and String do) |
| 8 | **True** | Implementing `From<A> for B` automatically provides `Into<B> for A` via a blanket impl |
| 9 | **True** | Multiple `impl` blocks for the same type share `self` — they are all equivalent |
| 10 | **True** | Associated functions (no `self`) must be called as `Type::fn()` not `instance.fn()` |

---

## 🏆 Lesson 13 Complete!

✅ Three forms of `self`: `&self`, `&mut self`, `self`  
✅ `Self` type alias for cleaner code  
✅ Auto-ref: Rust inserts `&`, `&mut`, or moves automatically  
✅ Associated functions as constructors with `::` syntax  
✅ Multiple `impl` blocks — organisation tool  
✅ Methods on enums  
✅ Method chaining — consuming vs `&mut Self`  
✅ Standard traits: `Display`, `Default`, `From`/`Into`  
✅ Getter naming convention: `fn field_name(&self)` not `get_field()`  
✅ Rectangle with `new()` and `scale()` — roadmap practice ✓  

**Up next: [Lesson 14 — Enums](../lesson_14_enums/lesson_14_enums.md) 🦀**
