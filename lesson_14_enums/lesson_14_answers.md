# ✅ Lesson 14 — Answers: Enums (S3)

---

## Section A — Predict the Output

### A1
```
West
North
```

### A2
```
9
3
```
`filter_map(|x| *x)` unwraps Some and discards None: 1+3+5=9. Three Somes.

### A3
```
Some(2.0)
None
Some(6.0)
0
```
`safe_sqrt(9.0)` → `Some(3.0)`, `.map(|v| v*2.0)` → `Some(6.0)`. `unwrap_or(0.0)` on None → 0.

### A4
```
3.14
12.00
12.00
```
Circle(1): π×1² ≈ 3.14. Rect(3,4): 12. Triangle(6,4): ½×6×4 = 12.

---

## Section B — Will It Compile?

### A5 — ✅ Yes
`Coin` has only unit variants — no data — so `Copy` works. `as u8` casts to discriminant. Output: `3\n3` (Quarter is index 3).

### A6 — ❌ No
`#[derive(Copy)]` requires all fields to be `Copy`. `String` is not `Copy`. **Error:** the trait `Copy` cannot be implemented for this type.

### A7 — ❌ No
Recursive enum `Tree` has infinite size — `Node` contains `Tree` directly. **Error:** recursive type `Tree` has infinite size. Fix: use `Box<Tree>`.

### A8 — ❌ No
`Color` derives `Debug` but not `Display`. `{}` requires `Display`. Fix: use `{:?}` or implement `fmt::Display`.

---

## Section C — Fix the Bugs

### A9 — Incomplete enum
```rust
#[derive(Debug)]
enum Season { Spring, Summer, Autumn, Winter }  // add Winter

fn describe(s: &Season) -> &str {
    match s {
        Season::Spring => "bloom",
        Season::Summer => "hot",
        Season::Autumn => "fall",
        Season::Winter => "cold",  // now valid
    }
}
fn main() { println!("{}", describe(&Season::Autumn)); } // fall
```

### A10 — Option misuse
```rust
fn first_even(nums: &[i32]) -> Option<i32> {
    nums.iter().find(|&&x| x % 2 == 0).copied()
}
fn main() {
    println!("{:?}", first_even(&[1, 3, 4, 6])); // Some(4)
    println!("{:?}", first_even(&[1, 3, 5]));     // None — no panic
}
```

### A11 — Recursive without Box
```rust
enum Expr { Num(f64), Add(Box<Expr>, Box<Expr>) }
fn main() { let _ = Expr::Num(1.0); }
```

---

## Section D — Write It Yourself

### A12 — Shape (roadmap practice task)
```rust
use std::fmt;
use std::f64::consts::PI;

#[derive(Debug, Clone)]
enum Shape { Circle(f64), Rect(f64, f64), Triangle(f64, f64) }

impl Shape {
    fn area(&self) -> f64 {
        match self {
            Shape::Circle(r)      => PI * r * r,
            Shape::Rect(w, h)     => w * h,
            Shape::Triangle(b, h) => 0.5 * b * h,
        }
    }

    fn perimeter(&self) -> f64 {
        match self {
            Shape::Circle(r)      => 2.0 * PI * r,
            Shape::Rect(w, h)     => 2.0 * (w + h),
            Shape::Triangle(b, h) => b + h + (b*b + h*h).sqrt(),
        }
    }

    fn name(&self) -> &str {
        match self {
            Shape::Circle(_)    => "Circle",
            Shape::Rect(_, _)   => "Rectangle",
            Shape::Triangle(..) => "Triangle",
        }
    }

    fn scale(&self, factor: f64) -> Shape {
        match self {
            Shape::Circle(r)      => Shape::Circle(r * factor),
            Shape::Rect(w, h)     => Shape::Rect(w * factor, h * factor),
            Shape::Triangle(b, h) => Shape::Triangle(b * factor, h * factor),
        }
    }
}

impl fmt::Display for Shape {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        match self {
            Shape::Circle(r) =>
                write!(f, "Circle(r={r:.1}, area={:.2})", self.area()),
            Shape::Rect(w, h) =>
                write!(f, "Rect({w:.1}×{h:.1}, area={:.2})", self.area()),
            Shape::Triangle(b, h) =>
                write!(f, "Triangle(b={b:.1}, h={h:.1}, area={:.2})", self.area()),
        }
    }
}

fn main() {
    let shapes = vec![
        Shape::Circle(5.0),
        Shape::Rect(4.0, 6.0),
        Shape::Triangle(3.0, 4.0),
    ];
    for s in &shapes {
        println!("{s}");
        println!("  perimeter={:.2}", s.perimeter());
        println!("  scaled×2: {}", s.scale(2.0));
    }
}
```

### A13 — Coin calculator
```rust
#[derive(Debug)]
enum Coin { Penny, Nickel, Dime, Quarter }

impl Coin {
    fn value_cents(&self) -> u32 {
        match self { Coin::Penny => 1, Coin::Nickel => 5, Coin::Dime => 10, Coin::Quarter => 25 }
    }
    fn name(&self) -> &'static str {
        match self { Coin::Penny => "penny", Coin::Nickel => "nickel",
                     Coin::Dime => "dime", Coin::Quarter => "quarter" }
    }
    fn from_cents(c: u32) -> Option<Coin> {
        match c { 1 => Some(Coin::Penny), 5 => Some(Coin::Nickel),
                  10 => Some(Coin::Dime), 25 => Some(Coin::Quarter), _ => None }
    }
}

fn make_change(mut amount: u32) -> Vec<Coin> {
    let coins = [Coin::Quarter, Coin::Dime, Coin::Nickel, Coin::Penny];
    let mut result = Vec::new();
    for coin in &coins {
        while amount >= coin.value_cents() {
            amount -= coin.value_cents();
            // Re-create coin to avoid move issues
            result.push(match coin { Coin::Quarter => Coin::Quarter,
                Coin::Dime => Coin::Dime, Coin::Nickel => Coin::Nickel, Coin::Penny => Coin::Penny });
        }
    }
    result
}

fn main() {
    let change = make_change(67);
    let names: Vec<&str> = change.iter().map(|c| c.name()).collect();
    println!("{names:?}"); // ["quarter", "quarter", "dime", "nickel", "penny", "penny"]
    println!("{:?}", Coin::from_cents(10)); // Some(Dime)
    println!("{:?}", Coin::from_cents(7));  // None
}
```

### A14 — Option chain
```rust
fn safe_div(a: f64, b: f64) -> Option<f64> {
    if b == 0.0 { None } else { Some(a / b) }
}
fn safe_log(x: f64) -> Option<f64> {
    if x <= 0.0 { None } else { Some(x.ln()) }
}
fn div_then_log(a: f64, b: f64) -> Option<f64> {
    safe_div(a, b).and_then(safe_log)
}
fn first_positive(nums: &[i32]) -> Option<i32> {
    nums.iter().find(|&&x| x > 0).copied()
}
fn main() {
    println!("{:?}", safe_div(10.0, 2.0));    // Some(5.0)
    println!("{:?}", safe_div(10.0, 0.0));    // None
    println!("{:?}", div_then_log(10.0, 2.0)); // Some(~1.609)
    println!("{:?}", div_then_log(10.0, 0.0)); // None
    println!("{:?}", first_positive(&[-1, 0, 3, 5])); // Some(3)
    println!("{:?}", first_positive(&[-1, -2]));       // None
}
```

### A15 — Expression tree
```rust
enum Expr {
    Num(f64),
    Add(Box<Expr>, Box<Expr>),
    Mul(Box<Expr>, Box<Expr>),
}
impl Expr {
    fn eval(&self) -> f64 {
        match self {
            Expr::Num(n)      => *n,
            Expr::Add(l, r)   => l.eval() + r.eval(),
            Expr::Mul(l, r)   => l.eval() * r.eval(),
        }
    }
}
fn main() {
    // (2 + 3) * 4
    let expr = Expr::Mul(
        Box::new(Expr::Add(Box::new(Expr::Num(2.0)), Box::new(Expr::Num(3.0)))),
        Box::new(Expr::Num(4.0)),
    );
    println!("{}", expr.eval()); // 20
}
```

### A16 — Task status
```rust
use std::fmt;

#[derive(Debug)]
enum Priority { Low, Medium, High }

#[derive(Debug, PartialEq)]
enum TaskStatus { Todo, InProgress, Done }

struct Task { title: String, priority: Priority, status: TaskStatus }

impl Task {
    fn new(title: &str, priority: Priority) -> Self {
        Task { title: title.to_string(), priority, status: TaskStatus::Todo }
    }
    fn start(&mut self) {
        if self.status == TaskStatus::Todo { self.status = TaskStatus::InProgress; }
    }
    fn complete(&mut self) {
        if self.status == TaskStatus::InProgress { self.status = TaskStatus::Done; }
    }
    fn is_done(&self) -> bool { self.status == TaskStatus::Done }
}

impl fmt::Display for Task {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "[{:?}] {} — {:?}", self.priority, self.title, self.status)
    }
}

fn main() {
    let mut tasks = vec![
        Task::new("Buy milk", Priority::Low),
        Task::new("Write tests", Priority::High),
        Task::new("Deploy app", Priority::High),
        Task::new("Read docs", Priority::Medium),
    ];
    tasks[0].start(); tasks[0].complete();
    tasks[1].start();
    tasks[2].start(); tasks[2].complete();
    for t in &tasks { println!("{t}"); }
    let done = tasks.iter().filter(|t| t.is_done()).count();
    println!("Done: {done}/{}", tasks.len());
}
```

---

## Section E — Deep Understanding

### A17 — True or False?
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | Core feature of Rust enums — each variant carries its own type/shape of data |
| 2 | **False** | `None` is type-safe; the type system forces you to handle it. `null` is not |
| 3 | **False** | `Copy` requires ALL variant data to be `Copy`. `String` is not |
| 4 | **True** | Without `Box`, `Node(i32, Tree)` would have infinite size at compile time |
| 5 | **True** | `None.map(f)` returns `None` — the closure is never called |
| 6 | **True** | `?` on `None` causes early return of `None` from the enclosing function |
| 7 | **True** | `impl MyEnum { ... }` works exactly like `impl MyStruct { ... }` |
| 8 | **False** | Different variants can hold completely different (or no) data |
| 9 | **True** | `unwrap()` on `None` panics with "called `Option::unwrap()` on a `None` value" |
| 10 | **False** | `Some` and `None` are automatically imported — no `Option::` prefix needed |

### A18 — Design challenge
1. **Enum** — payment methods are mutually exclusive and each has different fields (card number vs email vs IBAN)
2. **Struct** — a user profile always has all fields at once
3. **Enum** — log levels are mutually exclusive named choices with no extra data
4. **Struct** — a packet always has all three parts simultaneously
5. **Enum** — this is exactly `Option<T>`: `Found(value)` = `Some(T)`, `NotFound` = `None`

---

## 🏆 Lesson 14 Complete!

✅ Unit variants, tuple variants, struct variants  
✅ Mixed variants in one enum  
✅ Methods on enums with `impl`  
✅ `Option<T>` — the type-safe replacement for null  
✅ Key Option methods: `unwrap`, `unwrap_or`, `map`, `and_then`, `filter`  
✅ `?` operator for early Option propagation  
✅ Recursive enums with `Box<T>`  
✅ Deriving traits on enums  
✅ Shape enum — roadmap practice ✓  

**Up next: [Lesson 15 — Pattern Matching: match](../lesson_15_pattern_matching/lesson_15_pattern_matching.md) 🦀**
