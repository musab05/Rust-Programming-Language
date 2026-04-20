# 🧪 Lesson 12 — Questions: Structs (S1)

> **Lesson:** [lesson_12_structs.md](./lesson_12_structs.md)  
> **Answers:** [lesson_12_answers.md](./lesson_12_answers.md)

---

## Section A — Predict the Output

### Q1
```rust
#[derive(Debug)]
struct Point { x: i32, y: i32 }

fn main() {
    let p = Point { x: 3, y: 7 };
    println!("{:?}", p);
    println!("x={}, y={}", p.x, p.y);
}
```

### Q2
```rust
struct Counter { value: u32 }

impl Counter {
    fn new() -> Self { Counter { value: 0 } }
    fn increment(&mut self) { self.value += 1; }
    fn get(&self) -> u32 { self.value }
}

fn main() {
    let mut c = Counter::new();
    c.increment();
    c.increment();
    c.increment();
    println!("{}", c.get());
    c.increment();
    println!("{}", c.get());
}
```

### Q3
```rust
#[derive(Debug)]
struct User { name: String, active: bool, logins: u32 }

fn main() {
    let u1 = User {
        name: String::from("Alice"),
        active: true,
        logins: 5,
    };
    let u2 = User {
        name: String::from("Bob"),
        ..u1
    };
    println!("{}", u1.active);   // bool is Copy
    println!("{}", u1.logins);   // u32 is Copy
    println!("{}", u2.name);
    println!("{}", u2.active);
}
```

### Q4
```rust
struct Rectangle { width: f64, height: f64 }

impl Rectangle {
    fn area(&self) -> f64 { self.width * self.height }
    fn perimeter(&self) -> f64 { 2.0 * (self.width + self.height) }
    fn is_square(&self) -> bool { self.width == self.height }
}

fn main() {
    let r = Rectangle { width: 4.0, height: 4.0 };
    println!("{}", r.area());
    println!("{}", r.perimeter());
    println!("{}", r.is_square());
}
```

---

## Section B — Will It Compile?

### Q5
```rust
struct Point { x: f64, y: f64 }

fn main() {
    let p = Point { x: 1.0, y: 2.0 };
    p.x = 5.0;
    println!("{}", p.x);
}
```

### Q6
```rust
struct User { name: String, age: u32 }

fn main() {
    let u1 = User { name: String::from("Alice"), age: 30 };
    let u2 = User { age: 25, ..u1 };   // name NOT overridden — moves from u1
    println!("{}", u1.name);           // u1.name was moved into u2
}
```

### Q7
```rust
#[derive(Debug)]
struct Rect { w: f64, h: f64 }

fn print_it(r: Rect) { println!("{:?}", r); }

fn main() {
    let r = Rect { w: 3.0, h: 4.0 };
    print_it(r);
    println!("{}", r.w);  // after move
}
```

### Q8
```rust
struct Config { debug: bool, level: u8 }

fn main() {
    let c = Config { debug: true, level: 3 };
    println!("{}", c);  // Display not implemented
}
```

---

## Section C — Fix the Bugs

### Q9 — Missing `mut`
```rust
struct Score { value: i32 }

impl Score {
    fn new(v: i32) -> Self { Score { value: v } }
    fn add(&mut self, n: i32) { self.value += n; }
    fn get(&self) -> i32 { self.value }
}

fn main() {
    let s = Score::new(10);   // BUG
    s.add(5);
    println!("{}", s.get());
}
```

### Q10 — Wrong method receiver
```rust
struct Tank { fuel: f64 }

impl Tank {
    fn refuel(self, amount: f64) {   // BUG: takes ownership
        self.fuel += amount;
    }
    fn level(&self) -> f64 { self.fuel }
}

fn main() {
    let mut t = Tank { fuel: 50.0 };
    t.refuel(20.0);
    println!("{}", t.level());  // t was moved
}
```

### Q11 — Wrong constructor return
```rust
struct Circle { radius: f64 }

impl Circle {
    fn new(r: f64) -> Circle {
        Circle { r }   // BUG: field is named 'radius', not 'r'
    }
    fn area(&self) -> f64 { std::f64::consts::PI * self.radius * self.radius }
}

fn main() {
    let c = Circle::new(5.0);
    println!("{:.2}", c.area());
}
```

### Q12 — Implement Display
```rust
struct Point { x: f64, y: f64 }

fn main() {
    let p = Point { x: 1.5, y: 2.5 };
    println!("{p}");   // ERROR: Point doesn't implement Display
}
```
Implement `Display` so it prints `"(1.5, 2.5)"`.

---

## Section D — Write It Yourself

### Q13 — Roadmap Practice Task: Rectangle
Build a complete `Rectangle` type:
- Fields: `width: f64`, `height: f64`
- Associated function: `fn new(width: f64, height: f64) -> Self`
- Associated function: `fn square(side: f64) -> Self`
- Method: `fn area(&self) -> f64`
- Method: `fn perimeter(&self) -> f64`
- Method: `fn is_square(&self) -> bool`
- Method: `fn can_hold(&self, other: &Rectangle) -> bool` — true if `self` fits `other` inside
- Method: `fn scale(&mut self, factor: f64)` — multiply both dimensions
- Derive `Debug`; implement `Display` as `"Rectangle(WxH)"`

Test all methods in `main`.

### Q14 — Student struct
Design a `Student` struct:
- Fields: `name: String`, `id: u32`, `grade: f64`
- `fn new(name: &str, id: u32, grade: f64) -> Self`
- `fn letter_grade(&self) -> char` — A≥90, B≥80, C≥70, D≥60, F otherwise
- `fn is_passing(&self) -> bool` — grade >= 60
- `fn apply_curve(&mut self, points: f64)` — add points, cap at 100

### Q15 — Temperature struct
Build a `Temperature` struct that stores Celsius internally:
- `fn from_celsius(c: f64) -> Self`
- `fn from_fahrenheit(f: f64) -> Self`
- `fn celsius(&self) -> f64`
- `fn fahrenheit(&self) -> f64`
- `fn is_boiling(&self) -> bool` (>= 100°C)
- `fn is_freezing(&self) -> bool` (<= 0°C)
- Implement `Display`: `"23.5°C"`

### Q16 — Nested structs: Circle from Point
```rust
struct Point { x: f64, y: f64 }
struct Circle { center: Point, radius: f64 }
```
Implement:
- `Circle::new(cx, cy, r) -> Self`
- `fn area(&self) -> f64`
- `fn circumference(&self) -> f64`
- `fn contains(&self, p: &Point) -> bool`
- `fn distance_between_centers(&self, other: &Circle) -> f64`
- `fn overlaps(&self, other: &Circle) -> bool` — true if circles overlap

### Q17 — Full program: Bank account
```rust
struct BankAccount { owner: String, balance: f64 }
```
Implement:
- `fn new(owner: &str, initial_balance: f64) -> Self`
- `fn deposit(&mut self, amount: f64)` — panics if amount <= 0
- `fn withdraw(&mut self, amount: f64) -> Result<f64, String>` — returns new balance or error string
- `fn balance(&self) -> f64`
- `fn statement(&self) -> String` — `"Account[owner]: $balance"`
- Implement `Display` to print the statement

In `main`: create an account, make 3 deposits, 2 withdrawals (one that should fail), print final state.

---

## Section E — Deep Understanding

### Q18 — True or False?
1. You can mark individual fields as `mut` in a struct definition.
2. `#[derive(Debug)]` lets you print a struct with `{:?}`.
3. The struct update syntax `..other` always copies every field from `other`.
4. Associated functions are called with `.` dot notation like methods.
5. `Self` inside an `impl` block is a type alias for the struct being implemented.
6. A struct with a `String` field will implement `Copy` automatically.
7. You can have more than one `impl` block for the same struct.
8. `fn method(&self)` borrows the struct — the caller still owns it after the call.
9. Fields inside a struct are public to all code by default.
10. Destructuring a struct with `let Point { x, y } = p;` moves the fields out.

### Q19 — Trace ownership
```rust
struct Wrapper { data: String }

fn consume(w: Wrapper) -> usize { w.data.len() }
fn inspect(w: &Wrapper) -> usize { w.data.len() }

fn main() {
    let w = Wrapper { data: String::from("hello") };
    println!("{}", inspect(&w));    // line A
    println!("{}", consume(w));     // line B
    println!("{}", inspect(&w));    // line C — what happens?
}
```
What happens at line C, and why?

### Q20 — Design challenge
You are building a 2D game. For each entity below, decide: **struct** or **tuple** — and briefly justify.
1. A screen coordinate `(x, y)` used once in a calculation.
2. A game entity with `position`, `health`, `name`, `speed`.
3. An RGB color used in exactly one place.
4. A `Player` type used throughout 10 files.
5. A pair returned from a function: `(min_score, max_score)`.
