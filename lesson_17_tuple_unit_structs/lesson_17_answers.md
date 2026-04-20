# ✅ Lesson 17 — Answers: Tuple Structs & Unit Structs (S6)

---

## Section A — Predict the Output

### A1
```
Color(255, 0, 0)
r=255 g=0 b=0
255 0 0
```
`{:?}` uses the derived Debug format. `.0/.1/.2` access fields. Destructuring `let Color(r,g,b) = red` works like tuple destructuring.

### A2
```
10.44
```
100.0 / 9.58 ≈ 10.438... rounded to 2 decimal places = 10.44.

### A3
```
false
5.0
```
`Point(0,0) != Point(3,4)` so `false`. Distance = √(9+16) = √25 = 5.0.

### A4
```
16
16
0
```
Both named and tuple structs with the same fields have identical layout (2 × f64 = 16 bytes). Unit struct = 0 bytes.

---

## Section B — Will It Compile?

### A5 — ✅ Yes
`m.0` and `s.0` both give `f64`. Adding two `f64` values is valid. The types `Meters` and `Seconds` are distinct, but we're accessing their inner `f64` values explicitly. Output: `110`.

### A6 — ❌ No
`Fahrenheit` and `Celsius` are different types. `print_temp` expects `Celsius`. **Error:** mismatched types: expected `Celsius`, found `Fahrenheit`. This is the whole point of newtypes — preventing this mistake.

### A7 — ✅ Yes
`w.0` accesses the inner `Vec<i32>`. `.iter()` iterates it. Output: `1\n2\n3`.

### A8 — ❌ No
`Unit` doesn't implement `PartialEq`. `==` requires it. Fix: add `#[derive(PartialEq)]`.

---

## Section C — Fix the Bugs

### A9 — Wrong field access
```rust
struct Rgb(u8, u8, u8);

fn brightness(c: &Rgb) -> f64 {
    (c.0 as f64 + c.1 as f64 + c.2 as f64) / (3.0 * 255.0)  // fix: .0, .1, .2
}

fn main() {
    let c = Rgb(128, 200, 64);
    println!("{:.2}", brightness(&c)); // 0.51
}
```

### A10 — Type confusion fix
```rust
struct Kilometers(f64);
struct KmPerHour(f64);

fn travel_time(distance: Kilometers, speed: KmPerHour) -> f64 {
    distance.0 / speed.0
}

fn main() {
    let distance = Kilometers(150.0);
    let speed    = KmPerHour(100.0);
    println!("{:.2} hours", travel_time(distance, speed)); // 1.50 hours
    // travel_time(speed, distance); // ❌ COMPILE ERROR — KmPerHour where Kilometers expected
}
```

### A11 — Implement Display
```rust
use std::fmt;

struct Meters(f64);

impl fmt::Display for Meters {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "{}m", self.0)
    }
}

fn main() {
    let d = Meters(42.5);
    println!("{d}"); // 42.5m
}
```

### A12 — Unit struct instantiation
```rust
struct Marker;

fn main() {
    let m = Marker;   // fix: no () for unit struct
    println!("created marker");
}
```

---

## Section D — Write It Yourself

### A13 — Meters newtype (roadmap practice task)
```rust
use std::fmt;
use std::ops::Add;

#[derive(Debug, Clone, Copy, PartialEq, PartialOrd)]
struct Meters(f64);

#[derive(Debug, Clone, Copy)]
struct Centimeters(f64);

#[derive(Debug, Clone, Copy)]
struct Kilometers(f64);

impl Meters {
    fn new(val: f64) -> Self {
        assert!(val >= 0.0, "distance cannot be negative");
        Meters(val)
    }
    fn value(&self) -> f64 { self.0 }
    fn to_cm(&self) -> Centimeters { Centimeters(self.0 * 100.0) }
    fn to_km(&self) -> Kilometers  { Kilometers(self.0 / 1000.0) }
}

impl fmt::Display for Meters      { fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result { write!(f, "{:.2}m", self.0) } }
impl fmt::Display for Centimeters { fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result { write!(f, "{:.1}cm", self.0) } }
impl fmt::Display for Kilometers  { fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result { write!(f, "{:.4}km", self.0) } }

impl Add for Meters {
    type Output = Meters;
    fn add(self, other: Meters) -> Meters { Meters(self.0 + other.0) }
}

// struct Seconds(f64);
// Meters::new(5.0) + Seconds(2.0) → COMPILE ERROR:
//   cannot add `Seconds` to `Meters` — no Add<Seconds> impl for Meters.
// This is exactly the compile-time safety newtypes provide.

fn main() {
    let a = Meters::new(100.0);
    let b = Meters::new(250.0);
    let total = a + b;
    println!("{total}");             // 350.00m
    println!("{}", total.to_cm());   // 35000.0cm
    println!("{}", total.to_km());   // 0.3500km
}
```

### A14 — RGB color
```rust
use std::fmt;

#[derive(Debug, Clone, PartialEq)]
struct Rgb(u8, u8, u8);

impl Rgb {
    fn new(r: u8, g: u8, b: u8) -> Self { Rgb(r, g, b) }
    fn red()   -> Self { Rgb(255, 0, 0) }
    fn green() -> Self { Rgb(0, 255, 0) }
    fn blue()  -> Self { Rgb(0, 0, 255) }
    fn black() -> Self { Rgb(0, 0, 0) }
    fn white() -> Self { Rgb(255, 255, 255) }

    fn brightness(&self) -> f64 {
        (self.0 as f64 + self.1 as f64 + self.2 as f64) / (3.0 * 255.0)
    }

    fn invert(&self) -> Self { Rgb(255 - self.0, 255 - self.1, 255 - self.2) }

    fn blend(&self, other: &Self) -> Self {
        Rgb(
            ((self.0 as u16 + other.0 as u16) / 2) as u8,
            ((self.1 as u16 + other.1 as u16) / 2) as u8,
            ((self.2 as u16 + other.2 as u16) / 2) as u8,
        )
    }

    fn to_hex(&self) -> String { format!("#{:02X}{:02X}{:02X}", self.0, self.1, self.2) }
}

impl fmt::Display for Rgb {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "rgb({}, {}, {})", self.0, self.1, self.2)
    }
}

fn main() {
    let r = Rgb::red();
    let b = Rgb::blue();
    println!("{r} → hex: {}", r.to_hex());
    println!("brightness: {:.2}", r.brightness());
    println!("invert: {}", r.invert());
    println!("blend r+b: {}", r.blend(&b));
    println!("{:?}", r.clone());
    println!("equal: {}", r == Rgb::red());
}
```

### A15 — ID newtypes
```rust
use std::fmt;

struct UserId(u64);
struct ProductId(u64);
struct OrderId(u64);

impl UserId    { fn new(id: u64) -> Self { UserId(id) }    fn value(&self) -> u64 { self.0 } }
impl ProductId { fn new(id: u64) -> Self { ProductId(id) } fn value(&self) -> u64 { self.0 } }
impl OrderId   { fn new(id: u64) -> Self { OrderId(id) }   fn value(&self) -> u64 { self.0 } }

impl fmt::Display for UserId    { fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result { write!(f, "User#{}", self.0) } }
impl fmt::Display for ProductId { fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result { write!(f, "Product#{}", self.0) } }
impl fmt::Display for OrderId   { fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result { write!(f, "Order#{}", self.0) } }

fn get_user(id: UserId)       -> String { format!("Found user {id}") }
fn get_product(id: ProductId) -> String { format!("Found product {id}") }

fn main() {
    println!("{}", get_user(UserId::new(42)));
    println!("{}", get_product(ProductId::new(7)));

    // get_user(ProductId::new(1));
    // ❌ COMPILE ERROR: mismatched types
    //    expected `UserId`, found `ProductId`
    // The types are structurally identical (both wrap u64) but the
    // compiler treats them as completely different types — preventing
    // the bug of accidentally looking up a product record with a user ID.
}
```

### A16 — Unit struct Logger
```rust
use std::fmt;

struct Logger;
struct Separator;

impl Logger {
    fn info (&self, msg: &str) { println!("[INFO]  {msg}"); }
    fn warn (&self, msg: &str) { println!("[WARN]  {msg}"); }
    fn error(&self, msg: &str) { println!("[ERROR] {msg}"); }
    fn log  (&self, level: &str, msg: &str) { println!("[{level}] {msg}"); }
}

impl fmt::Display for Separator {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "━━━━━━━━━━━━━━━━━━━━")
    }
}

fn main() {
    let log = Logger;
    let sep = Separator;

    log.info("Application started");
    println!("{sep}");
    log.warn("Disk space low: 15% remaining");
    log.error("Failed to connect to database");
    println!("{sep}");
    log.log("DEBUG", "Variable x = 42");
}
```

### A17 — Unit converter
```rust
use std::fmt;

#[derive(Debug, Clone, Copy)] struct Meters(f64);
#[derive(Debug, Clone, Copy)] struct Feet(f64);
#[derive(Debug, Clone, Copy)] struct Inches(f64);
#[derive(Debug, Clone, Copy)] struct Centimeters(f64);

impl fmt::Display for Meters      { fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result { write!(f, "{:.3} m",  self.0) } }
impl fmt::Display for Feet        { fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result { write!(f, "{:.3} ft", self.0) } }
impl fmt::Display for Inches      { fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result { write!(f, "{:.2} in", self.0) } }
impl fmt::Display for Centimeters { fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result { write!(f, "{:.2} cm", self.0) } }

impl From<Meters> for Feet        { fn from(m: Meters) -> Feet        { Feet(m.0 * 3.28084) } }
impl From<Feet>   for Meters      { fn from(f: Feet)   -> Meters      { Meters(f.0 / 3.28084) } }
impl From<Feet>   for Inches      { fn from(f: Feet)   -> Inches      { Inches(f.0 * 12.0) } }
impl From<Inches> for Feet        { fn from(i: Inches) -> Feet        { Feet(i.0 / 12.0) } }
impl From<Meters> for Centimeters { fn from(m: Meters) -> Centimeters { Centimeters(m.0 * 100.0) } }
impl From<Centimeters> for Meters { fn from(c: Centimeters) -> Meters { Meters(c.0 / 100.0) } }

fn print_all_units(m: Meters) {
    let ft: Feet        = m.into();
    let i:  Inches      = ft.into();
    let cm: Centimeters = m.into();
    println!("{m} = {ft} = {i} = {cm}");
}

fn main() {
    print_all_units(Meters(1.0));
    print_all_units(Meters(5.0));
    print_all_units(Meters(100.0));
}
```

---

## Section E — Deep Understanding

### A18 — True or False?
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | Same fields = same memory layout. The distinction is purely at the source-code level |
| 2 | **False** | Tuple struct fields are accessed by index: `.0`, `.1`, `.2` — not by name |
| 3 | **False** | Unit structs are **zero bytes** — no discriminant, no data at all |
| 4 | **True** | The newtype pattern creates distinct types even with the same inner representation |
| 5 | **True** | `impl std::ops::Add for Meters { ... }` enables the `+` operator |
| 6 | **True** | Unit structs can implement any trait — `Logger` with no data is a clean example |
| 7 | **True** | `Rgb` has type `fn(u8,u8,u8) -> Rgb`, which can be passed to `.map()` |
| 8 | **True** | `Copy` requires all fields to be `Copy`. `f64` is `Copy`, so `Meters(f64)` can derive `Copy` |
| 9 | **False** | `Meters` is **your** type — you can implement any trait (including `Display`) for it. The orphan rule prevents implementing *external* traits for *external* types |
| 10 | **True** | Unit struct instantiation: `let m = Marker;` — no `()` needed (that's tuple syntax) |

### A19 — Design decisions
1. **Tuple struct** — `struct Email(String)` — wraps one value, newtype pattern
2. **Named struct** — 5 fields with distinct meanings, used widely across the codebase
3. **Unit struct** — `struct Encrypted;` — zero-sized marker, no data needed
4. **Tuple struct** — `struct LatLon(f64, f64)` — two positional floats, brief use; a named struct also works
5. **Named struct** — `struct Rgb { r: u8, g: u8, b: u8 }` — many methods make named fields cleaner
6. **Tuple struct** — `struct DisplayableBytes(Vec<u8>)` — newtype for implementing a foreign trait
7. **Unit struct** — `struct InitCalled;` — zero-sized token proves something happened at compile time

### A20 — Newtype analysis
```rust
// AFTER — type-safe version
struct NewtonSeconds(f64);
struct Seconds(f64);
struct Newtons(f64);

fn compute_thrust(impulse: NewtonSeconds, time: Seconds) -> Newtons {
    Newtons(impulse.0 / time.0)
}

fn main() {
    let impulse = NewtonSeconds(4000.0);
    let time    = Seconds(2.0);

    let thrust = compute_thrust(impulse, time);
    println!("Thrust: {:.1} N", thrust.0); // 2000.0 N — correct!

    // The original bug — swapped arguments — is now a compile error:
    // compute_thrust(time, impulse);
    // ERROR: expected `NewtonSeconds`, found `Seconds`
    //        expected `Seconds`, found `NewtonSeconds`
    // The compiler catches exactly what caused the Mars orbiter to crash.
}
```

---

## 🏆 Lesson 17 Complete!

✅ Tuple struct definition `struct Name(T1, T2)`  
✅ Field access via `.0`, `.1`, `.2`  
✅ Destructuring tuple structs in `let` and `match`  
✅ The newtype pattern — distinct types with same inner representation  
✅ Type safety: prevents unit/ID confusion at compile time  
✅ Methods on tuple structs with `impl`  
✅ Implementing `Display`, `From`/`Into`, `Add` on newtypes  
✅ Unit structs — zero-sized types  
✅ Unit structs as marker types and type-state encoding  
✅ `Meters` newtype — roadmap practice ✓  

**Up next: [Lesson 18 — String vs `&str`](../lesson_18_string_vs_str/lesson_18_string_vs_str.md) 🦀**
