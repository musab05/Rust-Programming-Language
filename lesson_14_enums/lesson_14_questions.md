# 🧪 Lesson 14 — Questions: Enums (S3)

> **Lesson:** [lesson_14_enums.md](./lesson_14_enums.md)  
> **Answers:** [lesson_14_answers.md](./lesson_14_answers.md)

---

## Section A — Predict the Output

### Q1
```rust
#[derive(Debug)]
enum Direction { North, South, East, West }
impl Direction {
    fn opposite(&self) -> Direction {
        match self {
            Direction::North => Direction::South,
            Direction::South => Direction::North,
            Direction::East  => Direction::West,
            Direction::West  => Direction::East,
        }
    }
}
fn main() {
    let d = Direction::East;
    println!("{:?}", d.opposite());
    println!("{:?}", Direction::South.opposite());
}
```

### Q2
```rust
fn main() {
    let nums: Vec<Option<i32>> = vec![Some(1), None, Some(3), None, Some(5)];
    let total: i32 = nums.iter().filter_map(|x| *x).sum();
    println!("{total}");
    let count = nums.iter().filter(|x| x.is_some()).count();
    println!("{count}");
}
```

### Q3
```rust
fn safe_sqrt(x: f64) -> Option<f64> {
    if x < 0.0 { None } else { Some(x.sqrt()) }
}
fn main() {
    println!("{:?}", safe_sqrt(4.0));
    println!("{:?}", safe_sqrt(-1.0));
    let result = safe_sqrt(9.0).map(|v| v * 2.0);
    println!("{:?}", result);
    let fallback = safe_sqrt(-1.0).unwrap_or(0.0);
    println!("{fallback}");
}
```

### Q4
```rust
#[derive(Debug)]
enum Shape { Circle(f64), Rect(f64, f64), Triangle(f64, f64) }
impl Shape {
    fn area(&self) -> f64 {
        match self {
            Shape::Circle(r)      => std::f64::consts::PI * r * r,
            Shape::Rect(w, h)     => w * h,
            Shape::Triangle(b, h) => 0.5 * b * h,
        }
    }
}
fn main() {
    let shapes = [Shape::Circle(1.0), Shape::Rect(3.0, 4.0), Shape::Triangle(6.0, 4.0)];
    for s in &shapes { println!("{:.2}", s.area()); }
}
```

---

## Section B — Will It Compile?

### Q5
```rust
#[derive(Clone, Copy)]
enum Coin { Penny, Nickel, Dime, Quarter }
fn main() {
    let c = Coin::Quarter;
    let c2 = c;
    println!("{}", c as u8);
    println!("{}", c2 as u8);
}
```

### Q6
```rust
#[derive(Clone, Copy)]
enum Wrapper { Val(String) }
fn main() { let _ = Wrapper::Val(String::from("hi")); }
```

### Q7
```rust
enum Tree { Leaf(i32), Node(i32, Tree, Tree) }
fn main() { let _ = Tree::Leaf(1); }
```

### Q8
```rust
#[derive(Debug)]
enum Color { Red, Green, Blue }
fn main() {
    let c = Color::Red;
    println!("{c}"); // Display
}
```

---

## Section C — Fix the Bugs

### Q9 — Incomplete enum + wrong arm
```rust
#[derive(Debug)]
enum Season { Spring, Summer, Autumn } // Winter missing

fn describe(s: &Season) -> &str {
    match s {
        Season::Spring => "bloom",
        Season::Summer => "hot",
        Season::Autumn => "fall",
        Season::Winter => "cold",   // BUG: Winter not in enum
    }
}
fn main() { println!("{}", describe(&Season::Autumn)); }
```
Fix by adding `Winter` to the enum, then add its arm.

### Q10 — Option misuse
```rust
fn first_even(nums: &[i32]) -> i32 {
    nums.iter().find(|&&x| x % 2 == 0).unwrap()  // BUG: returns &i32; panics if empty
}
fn main() {
    println!("{}", first_even(&[1, 3, 4, 6]));
    println!("{}", first_even(&[1, 3, 5])); // panic!
}
```
Fix to return `Option<i32>` safely.

### Q11 — Recursive enum without Box
```rust
enum Expr { Num(f64), Add(Expr, Expr) }  // BUG: recursive without Box
fn main() { let _ = Expr::Num(1.0); }
```

---

## Section D — Write It Yourself

### Q12 — Roadmap Practice Task: Shape enum
Build the full roadmap Shape:
```rust
enum Shape { Circle(f64), Rect(f64, f64), Triangle(f64, f64) }
```
Add these methods:
- `fn area(&self) -> f64`
- `fn perimeter(&self) -> f64` — for Triangle, use right-triangle formula
- `fn name(&self) -> &str`
- `fn scale(&self, factor: f64) -> Shape` — returns new scaled shape
- Implement `Display`: `"Circle(r=5.0, area=78.54)"`

Test all shapes and all methods in `main`.

### Q13 — Coin calculator
Build `enum Coin { Penny, Nickel, Dime, Quarter }`:
- `fn value_cents(&self) -> u32`
- `fn name(&self) -> &'static str`
- `fn from_cents(c: u32) -> Option<Coin>` — None if no coin matches
- Write `fn make_change(amount_cents: u32) -> Vec<Coin>` — minimum coins greedy

### Q14 — Option chain
Write these functions:
- `fn safe_div(a: f64, b: f64) -> Option<f64>` — None if b == 0
- `fn safe_log(x: f64) -> Option<f64>` — None if x <= 0
- `fn div_then_log(a: f64, b: f64) -> Option<f64>` — divide then log using `and_then`
- `fn first_positive(nums: &[i32]) -> Option<i32>`

Test with valid and invalid inputs.

### Q15 — Recursive expression tree
```rust
enum Expr {
    Num(f64),
    Add(Box<Expr>, Box<Expr>),
    Mul(Box<Expr>, Box<Expr>),
}
```
Implement `fn eval(&self) -> f64`. Build and evaluate `(2 + 3) * 4`.

### Q16 — Full program: Task status system
```rust
#[derive(Debug)]
enum Priority { Low, Medium, High }

#[derive(Debug, PartialEq)]
enum TaskStatus { Todo, InProgress, Done }

struct Task { title: String, priority: Priority, status: TaskStatus }
```
Implement on `Task`:
- `fn new(title: &str, priority: Priority) -> Self`
- `fn start(&mut self)`  — Todo → InProgress
- `fn complete(&mut self)` — InProgress → Done
- `fn is_done(&self) -> bool`
- Implement `Display`: `"[High] Buy milk — InProgress"`

Create 4 tasks, advance some, print all.

---

## Section E — Deep Understanding

### Q17 — True or False?
1. An enum variant can hold completely different types of data from other variants.
2. `Option::None` is the same as `null` — both are type-unsafe.
3. `#[derive(Copy)]` works on an enum with a `String` field.
4. Recursive enums need `Box<T>` because the size would otherwise be infinite.
5. `Option::map` returns `None` if the original was `None`.
6. `?` in a function returning `Option<T>` propagates `None` upward.
7. You can implement methods on enums using `impl`.
8. All variants of an enum must hold the same type of data.
9. The `unwrap()` method panics if the Option is `None`.
10. `Some(5)` and `None` must be used as `Option::Some(5)` and `Option::None`.

### Q18 — Design challenge
For each scenario, choose **enum** or **struct** and justify:
1. A payment method: credit card, PayPal, bank transfer (each with different fields).
2. A user profile with name, email, age, and join date.
3. A log level: Trace, Debug, Info, Warn, Error.
4. A network packet with a header, payload, and checksum.
5. A search result that is either found (with value) or not found.
