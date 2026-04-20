# ✅ Lesson 12 — Answers: Structs (S1)

---

## Section A — Predict the Output

### A1
```
Point { x: 3, y: 7 }
x=3, y=7
```

### A2
```
3
4
```
Three increments → 3. One more → 4.

### A3
```
true
5
Bob
true
```
`u1.active` (bool, Copy) and `u1.logins` (u32, Copy) are not moved — safe to use after `..u1`. `u2.name` = "Bob" (explicitly set). `u2.active` copied from u1 = `true`.

### A4
```
16
16
true
```
4×4 = 16. 2×(4+4) = 16. 4.0 == 4.0 → true.

---

## Section B — Will It Compile?

### A5 — ❌ No
`p` is not `mut`. `p.x = 5.0` requires a mutable binding. Fix: `let mut p = ...`

### A6 — ❌ No
`u1.name` (String) is **moved** into `u2` by `..u1` (since `name` wasn't overridden in `u2`). Then `u1.name` is used — borrow of moved value. Fix: override `name` in `u2` or clone: `name: u1.name.clone()`.

### A7 — ❌ No
`print_it(r)` takes ownership of `r`. After the call, `r` is moved and `r.w` cannot be accessed. Fix: either borrow in `print_it` (`fn print_it(r: &Rect)`) or derive `Clone` and pass `r.clone()`.

### A8 — ❌ No (compile error)
`Config` doesn't implement `Display`. `{}` requires it. Fix: add `#[derive(Debug)]` and use `{:?}`, or implement `fmt::Display`.

---

## Section C — Fix the Bugs

### A9 — Missing `mut`
```rust
fn main() {
    let mut s = Score::new(10);   // fix: add mut
    s.add(5);
    println!("{}", s.get()); // 15
}
```

### A10 — Wrong method receiver
```rust
impl Tank {
    fn refuel(&mut self, amount: f64) {   // fix: &mut self
        self.fuel += amount;
    }
    fn level(&self) -> f64 { self.fuel }
}
fn main() {
    let mut t = Tank { fuel: 50.0 };
    t.refuel(20.0);
    println!("{}", t.level()); // 70
}
```

### A11 — Wrong constructor return
```rust
impl Circle {
    fn new(r: f64) -> Circle {
        Circle { radius: r }   // fix: use correct field name
    }
    fn area(&self) -> f64 { std::f64::consts::PI * self.radius * self.radius }
}
fn main() {
    let c = Circle::new(5.0);
    println!("{:.2}", c.area()); // 78.54
}
```

### A12 — Implement Display
```rust
use std::fmt;

struct Point { x: f64, y: f64 }

impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}

fn main() {
    let p = Point { x: 1.5, y: 2.5 };
    println!("{p}"); // (1.5, 2.5)
}
```

---

## Section D — Write It Yourself

### A13 — Rectangle (roadmap practice task)
```rust
use std::fmt;

#[derive(Debug)]
struct Rectangle { width: f64, height: f64 }

impl Rectangle {
    fn new(width: f64, height: f64) -> Self { Rectangle { width, height } }
    fn square(side: f64) -> Self { Rectangle { width: side, height: side } }

    fn area(&self) -> f64 { self.width * self.height }
    fn perimeter(&self) -> f64 { 2.0 * (self.width + self.height) }
    fn is_square(&self) -> bool { (self.width - self.height).abs() < f64::EPSILON }

    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }

    fn scale(&mut self, factor: f64) {
        self.width  *= factor;
        self.height *= factor;
    }
}

impl fmt::Display for Rectangle {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "Rectangle({}×{})", self.width, self.height)
    }
}

fn main() {
    let mut r = Rectangle::new(10.0, 5.0);
    let s = Rectangle::square(4.0);

    println!("{r}");
    println!("{r:?}");
    println!("area      = {}", r.area());
    println!("perimeter = {}", r.perimeter());
    println!("is_square = {}", r.is_square());
    println!("can_hold s = {}", r.can_hold(&s));

    r.scale(0.5);
    println!("after scale: {r}");
    println!("is_square now = {}", Rectangle::square(5.0).is_square());
}
```

### A14 — Student struct
```rust
struct Student { name: String, id: u32, grade: f64 }

impl Student {
    fn new(name: &str, id: u32, grade: f64) -> Self {
        Student { name: name.to_string(), id, grade }
    }

    fn letter_grade(&self) -> char {
        match self.grade as u32 {
            90..=100 => 'A',
            80..=89  => 'B',
            70..=79  => 'C',
            60..=69  => 'D',
            _        => 'F',
        }
    }

    fn is_passing(&self) -> bool { self.grade >= 60.0 }

    fn apply_curve(&mut self, points: f64) {
        self.grade = (self.grade + points).min(100.0);
    }
}

fn main() {
    let mut s = Student::new("Alice", 1001, 72.5);
    println!("{} — {} — passing: {}", s.name, s.letter_grade(), s.is_passing());
    s.apply_curve(10.0);
    println!("After curve: {:.1} — {}", s.grade, s.letter_grade());
}
```

### A15 — Temperature struct
```rust
use std::fmt;

struct Temperature { celsius: f64 }

impl Temperature {
    fn from_celsius(c: f64) -> Self    { Temperature { celsius: c } }
    fn from_fahrenheit(f: f64) -> Self { Temperature { celsius: (f - 32.0) * 5.0 / 9.0 } }
    fn celsius(&self) -> f64    { self.celsius }
    fn fahrenheit(&self) -> f64 { self.celsius * 9.0 / 5.0 + 32.0 }
    fn is_boiling(&self) -> bool  { self.celsius >= 100.0 }
    fn is_freezing(&self) -> bool { self.celsius <= 0.0 }
}

impl fmt::Display for Temperature {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "{:.1}°C", self.celsius)
    }
}

fn main() {
    let boiling = Temperature::from_celsius(100.0);
    println!("{boiling} = {:.1}°F", boiling.fahrenheit());
    println!("boiling: {}", boiling.is_boiling());

    let freezing = Temperature::from_fahrenheit(32.0);
    println!("{freezing} — freezing: {}", freezing.is_freezing());
}
```

### A16 — Circle with nested Point
```rust
struct Point { x: f64, y: f64 }
struct Circle { center: Point, radius: f64 }

impl Circle {
    fn new(cx: f64, cy: f64, r: f64) -> Self {
        Circle { center: Point { x: cx, y: cy }, radius: r }
    }

    fn area(&self) -> f64 { std::f64::consts::PI * self.radius * self.radius }
    fn circumference(&self) -> f64 { 2.0 * std::f64::consts::PI * self.radius }

    fn contains(&self, p: &Point) -> bool {
        let dx = self.center.x - p.x;
        let dy = self.center.y - p.y;
        (dx * dx + dy * dy).sqrt() <= self.radius
    }

    fn distance_between_centers(&self, other: &Circle) -> f64 {
        let dx = self.center.x - other.center.x;
        let dy = self.center.y - other.center.y;
        (dx * dx + dy * dy).sqrt()
    }

    fn overlaps(&self, other: &Circle) -> bool {
        self.distance_between_centers(other) < self.radius + other.radius
    }
}

fn main() {
    let c1 = Circle::new(0.0, 0.0, 5.0);
    let c2 = Circle::new(3.0, 0.0, 3.0);
    let c3 = Circle::new(20.0, 0.0, 2.0);

    println!("area = {:.2}", c1.area());
    println!("c1 contains (3,4): {}", c1.contains(&Point { x: 3.0, y: 4.0 }));
    println!("c1 overlaps c2: {}", c1.overlaps(&c2));
    println!("c1 overlaps c3: {}", c1.overlaps(&c3));
}
```

### A17 — Bank account
```rust
use std::fmt;

struct BankAccount { owner: String, balance: f64 }

impl BankAccount {
    fn new(owner: &str, initial_balance: f64) -> Self {
        BankAccount { owner: owner.to_string(), balance: initial_balance }
    }

    fn deposit(&mut self, amount: f64) {
        assert!(amount > 0.0, "deposit amount must be positive");
        self.balance += amount;
    }

    fn withdraw(&mut self, amount: f64) -> Result<f64, String> {
        if amount <= 0.0 { return Err("amount must be positive".to_string()); }
        if amount > self.balance {
            return Err(format!("insufficient funds: have {:.2}, need {:.2}", self.balance, amount));
        }
        self.balance -= amount;
        Ok(self.balance)
    }

    fn balance(&self) -> f64 { self.balance }
    fn statement(&self) -> String {
        format!("Account[{}]: ${:.2}", self.owner, self.balance)
    }
}

impl fmt::Display for BankAccount {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "{}", self.statement())
    }
}

fn main() {
    let mut acc = BankAccount::new("Alice", 1000.0);
    acc.deposit(200.0);
    acc.deposit(50.0);
    acc.deposit(300.0);
    println!("{acc}");

    match acc.withdraw(100.0) {
        Ok(bal)  => println!("Withdrew $100, balance: ${bal:.2}"),
        Err(e)   => println!("Error: {e}"),
    }
    match acc.withdraw(9999.0) {
        Ok(bal)  => println!("Withdrew $9999, balance: ${bal:.2}"),
        Err(e)   => println!("Error: {e}"),
    }
    println!("{acc}");
}
```

---

## Section E — Deep Understanding

### A18 — True or False?
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **False** | Mutability is on the binding (`let mut x = ...`), not individual fields |
| 2 | **True** | `#[derive(Debug)]` implements the `Debug` trait for `{:?}` printing |
| 3 | **False** | It only copies fields **not** explicitly set; overriding stops that field from being copied |
| 4 | **False** | Associated functions are called with `::` syntax: `Type::function()` |
| 5 | **True** | `Self` inside `impl Rectangle` = `Rectangle` |
| 6 | **False** | `Copy` requires ALL fields to be `Copy`. `String` is not `Copy` |
| 7 | **True** | Multiple `impl` blocks for the same type are valid and equivalent |
| 8 | **True** | `&self` borrows — caller retains ownership after the method returns |
| 9 | **False** | Fields are private by default outside the module; use `pub` to expose them |
| 10 | **True** | `let Point { x, y } = p;` moves `x` and `y` out of `p` (unless they're Copy) |

### A19 — Trace ownership
```
Line A: inspect(&w)  — borrows w, returns 5. w still owned.
Line B: consume(w)   — MOVES w. Ownership transferred to consume(). Returns 5.
Line C: inspect(&w)  — ERROR: borrow of moved value 'w'.
```
After `consume(w)`, `w` has been moved into the function and dropped at the end of it. `w` no longer exists in `main`. The compiler will reject line C with "use of moved value: `w`".

**Fix options:**
1. Call `inspect` after `consume` is wrong order — swap lines B and C
2. Change `consume` to take `&Wrapper`
3. Derive `Clone` on `Wrapper` and pass `w.clone()` to `consume`

### A20 — Design challenge
1. **Tuple** — `(x, y)` used once, throwaway calculation, naming adds no value
2. **Struct** — multiple fields, used everywhere, named fields make the code readable
3. **Tuple** — used in exactly one place; OR a `struct Color(u8, u8, u8)` tuple struct if type safety matters
4. **Struct** — used in 10 files, named fields are essential for maintainability
5. **Tuple** — return value with obvious positions, used once at the call site; a `struct ScoreRange { min, max }` would be better if used across many files

---

## 🏆 Lesson 12 Complete!

✅ Struct definition, instantiation, field access  
✅ Field init shorthand and struct update syntax  
✅ Ownership in structs — owned fields, moves, borrows  
✅ `#[derive(Debug)]` and implementing `Display`  
✅ Methods with `&self`, `&mut self`, and associated functions  
✅ `Self` type alias  
✅ Nested structs  
✅ Rectangle with `area()` and `perimeter()` — roadmap practice ✓  

**Up next: [Lesson 13 — Methods & impl](../lesson_13_methods_impl/lesson_13_methods_impl.md) 🦀**
