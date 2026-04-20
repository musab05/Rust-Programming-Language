# 🧪 Lesson 17 — Questions: Tuple Structs & Unit Structs (S6)

> **Lesson:** [lesson_17_tuple_unit_structs.md](./lesson_17_tuple_unit_structs.md)  
> **Answers:** [lesson_17_answers.md](./lesson_17_answers.md)

---

## Section A — Predict the Output

### Q1
```rust
#[derive(Debug)]
struct Color(u8, u8, u8);

fn main() {
    let red = Color(255, 0, 0);
    println!("{:?}", red);
    println!("r={} g={} b={}", red.0, red.1, red.2);
    let Color(r, g, b) = red;
    println!("{r} {g} {b}");
}
```

### Q2
```rust
struct Meters(f64);
struct Seconds(f64);

fn speed(d: Meters, t: Seconds) -> f64 { d.0 / t.0 }

fn main() {
    let dist = Meters(100.0);
    let time = Seconds(9.58);
    println!("{:.2}", speed(dist, time));
}
```

### Q3
```rust
#[derive(Debug, PartialEq)]
struct Point(f64, f64);

impl Point {
    fn distance(&self, other: &Point) -> f64 {
        let dx = self.0 - other.0;
        let dy = self.1 - other.1;
        (dx*dx + dy*dy).sqrt()
    }
}

fn main() {
    let a = Point(0.0, 0.0);
    let b = Point(3.0, 4.0);
    println!("{}", a == b);
    println!("{:.1}", a.distance(&b));
}
```

### Q4
```rust
use std::mem::size_of;

struct Named { x: f64, y: f64 }
struct Tuple(f64, f64);
struct Unit;

fn main() {
    println!("{}", size_of::<Named>());
    println!("{}", size_of::<Tuple>());
    println!("{}", size_of::<Unit>());
}
```

---

## Section B — Will It Compile?

### Q5
```rust
struct Meters(f64);
struct Seconds(f64);

fn main() {
    let m = Meters(100.0);
    let s = Seconds(10.0);
    let result = m.0 + s.0;  // add the inner f64 values
    println!("{result}");
}
```

### Q6
```rust
struct Celsius(f64);
struct Fahrenheit(f64);

fn print_temp(t: Celsius) { println!("{:.1}°C", t.0); }

fn main() {
    let f = Fahrenheit(212.0);
    print_temp(f);  // pass Fahrenheit where Celsius expected
}
```

### Q7
```rust
struct Wrapper(Vec<i32>);

fn main() {
    let w = Wrapper(vec![1, 2, 3]);
    for x in w.0.iter() {
        println!("{x}");
    }
}
```

### Q8
```rust
struct Unit;

fn main() {
    let a = Unit;
    let b = Unit;
    println!("{}", a == b);  // no PartialEq
}
```

---

## Section C — Fix the Bugs

### Q9 — Wrong field access
```rust
struct Rgb(u8, u8, u8);

fn brightness(c: &Rgb) -> f64 {
    (c.r as f64 + c.g as f64 + c.b as f64) / (3.0 * 255.0)  // BUG: named fields
}

fn main() {
    let c = Rgb(128, 200, 64);
    println!("{:.2}", brightness(&c));
}
```

### Q10 — Type confusion (the point of newtypes!)
```rust
fn travel_time(distance_km: f64, speed_kmh: f64) -> f64 {
    distance_km / speed_kmh
}

fn main() {
    let distance = 150.0_f64;  // km
    let speed    = 100.0_f64;  // km/h
    // BUG: easy to pass in wrong order — no type safety
    println!("{:.2} hours", travel_time(speed, distance)); // swapped!
}
```
Refactor using newtypes `Kilometers` and `KmPerHour` to make the bug a compile error.

### Q11 — Implement Display
```rust
struct Meters(f64);

fn main() {
    let d = Meters(42.5);
    println!("{d}");   // ERROR: Meters doesn't implement Display
}
```
Implement `Display` to print `"42.5m"`.

### Q12 — Unit struct instantiation
```rust
struct Marker;

fn main() {
    let m = Marker();  // BUG: unit struct doesn't need ()
    println!("created marker");
}
```

---

## Section D — Write It Yourself

### Q13 — Roadmap Practice Task: Meters newtype
This is the exact roadmap task. Create a `Meters(f64)` newtype that:
- `fn new(val: f64) -> Self` — panics if negative
- `fn value(&self) -> f64`
- `fn to_cm(&self) -> Centimeters`
- `fn to_km(&self) -> Kilometers`
- Implement `Display`: `"42.5m"`
- Implement `Add` (std::ops::Add) so `Meters + Meters = Meters`

Also create `Centimeters(f64)` and `Kilometers(f64)` with matching conversions.

Demonstrate that `Meters + Seconds` is a compile error (write a comment explaining how the type system catches it).

### Q14 — RGB color with tuple struct
Build `struct Rgb(u8, u8, u8)`:
- `fn new(r: u8, g: u8, b: u8) -> Self`
- `fn red() -> Self`, `fn green() -> Self`, `fn blue() -> Self`, `fn black() -> Self`, `fn white() -> Self`
- `fn brightness(&self) -> f64` — 0.0..=1.0
- `fn invert(&self) -> Self`
- `fn blend(&self, other: &Self) -> Self`
- `fn to_hex(&self) -> String` — `"#FF8800"`
- Implement `Display`: `"rgb(255, 128, 0)"`
- Derive `Debug, Clone, PartialEq`

### Q15 — ID newtypes
Create three newtypes: `UserId(u64)`, `ProductId(u64)`, `OrderId(u64)`:
- Each with `fn new(id: u64) -> Self` and `fn value(&self) -> u64`
- Implement `Display` for each: `"User#42"`, `"Product#7"`, `"Order#99"`
- Write functions `fn get_user(id: UserId) -> String`, `fn get_product(id: ProductId) -> String`
- In `main`, show that calling `get_user(ProductId(1))` is a compile error (write it as a comment with explanation)

### Q16 — Unit struct Logger
Build a `Logger` unit struct with methods:
- `fn info(&self, msg: &str)` — prints `"[INFO] msg"`
- `fn warn(&self, msg: &str)` — prints `"[WARN] msg"`
- `fn error(&self, msg: &str)` — prints `"[ERROR] msg"`
- `fn log(&self, level: &str, msg: &str)` — generic version

Also create a `Separator` unit struct that implements `Display` as `"━━━━━━━━━━━━━━━━━━━━"`.

### Q17 — Full program: Unit converter
Build a complete unit conversion system:

```rust
struct Meters(f64);
struct Feet(f64);
struct Inches(f64);
struct Centimeters(f64);
```

Implement `From` conversions between all four types (at minimum: Meters↔Feet, Feet↔Inches, Meters↔Centimeters). Implement `Display` for each. Write a `fn print_all_units(m: Meters)` that prints the value in all four units.

---

## Section E — Deep Understanding

### Q18 — True or False?
1. A tuple struct `struct Point(f64, f64)` uses the same memory layout as `struct Point { x: f64, y: f64 }`.
2. You can access tuple struct fields by name like `point.x`.
3. A unit struct takes exactly 1 byte of memory for the discriminant.
4. `struct Meters(f64)` and `struct Seconds(f64)` are incompatible types even though both wrap `f64`.
5. You can implement `std::ops::Add` for a tuple struct to support the `+` operator.
6. A unit struct can be used as a trait implementation with no associated data.
7. Tuple struct constructors can be passed to `map` as function pointers.
8. `#[derive(Copy)]` works on `struct Meters(f64)` because `f64` is `Copy`.
9. The orphan rule prevents you from implementing `Display` on `Meters(f64)`.
10. A unit struct `struct Marker;` can be instantiated as `let m = Marker;` (no parentheses).

### Q19 — Design decisions
For each situation, choose **named struct**, **tuple struct**, or **unit struct**:

1. Storing a validated email address as a distinct type over `String`.
2. A game entity with `name`, `health`, `position`, `speed`, and `team`.
3. A type-level marker for "this data has been encrypted".
4. A pair of `(latitude, longitude)` used briefly in one function.
5. An RGB colour used throughout a graphics library with many methods.
6. A wrapper for `Vec<u8>` to implement `Display` for it.
7. A compile-time token proving a function has been called.

### Q20 — Newtype analysis
The NASA Mars Climate Orbiter was lost because of a unit mismatch. Below is simplified code that caused the problem. Refactor it using newtypes so the compiler catches the error:

```rust
// BEFORE (buggy — no type safety)
fn compute_thrust(impulse: f64, time: f64) -> f64 {
    impulse / time  // expects Newton-seconds and seconds
}

fn main() {
    let impulse_ns   = 4000.0;  // Newton-seconds (correct unit)
    let time_s       = 2.0;     // seconds
    let thrust_wrong = compute_thrust(time_s, impulse_ns);  // SWAPPED — silent bug!
    println!("Thrust: {thrust_wrong}");
}
```
