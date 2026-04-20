# ✅ Lesson 15 — Answers: Pattern Matching: `match` (S4)

---

## Section A — Predict the Output

### A1
```
odd and > 6
```
7 doesn't match 1, 2|3, or 4..=6. The guard `x if x % 2 == 0` fails (7 is odd). Falls to `_` → "odd and > 6".

### A2
```
x=0, y=-5
```
`(0, -5)` matches `(0, y)` first — x is 0, y gets -5. Arm fires immediately.

### A3
```
61
```
Quarter(25) + Dime(10) + Penny(1) + Quarter(25) = 61.

### A4
```
small: 1
none
big: 3
big: 4
none
```
`Some(1)`: matches `Some(x)` with guard `*x > 2` → false, falls to `Some(x)` → "small: 1". `None` → "none". `Some(3)` and `Some(4)` → guard passes → "big". `None` → "none".

### A5
```
empty
one: 99
two: 1,2
many: 10..40
```

---

## Section B — Will It Compile?

### A6 — ❌ No
`Direction::East` is not covered. Rust's exhaustive match rejects this. **Error:** `non-exhaustive patterns: Direction::East not covered`. Fix: add `Direction::East => "east"` or a `_ =>` arm.

### A7 — ✅ Yes
`u32` is matched against literals and `_`. Exhaustive. Output: `other`.

### A8 — ✅ Yes
`(1, 2, 3)` matches the first arm `(1, y, z)` — y=2, z=3. Output: `2 3`. (The second arm is unreachable but not a compile error — Rust warns about unreachable patterns, not errors.)

### A9 — ✅ Yes
`x = 10` matches `_` → "positive". `x` is still usable after the match because the match borrowed it (all arms use guards or literals, no ownership taken). Output: `positive\n10`.

### A10 — ✅ Yes
`n = 42` matches `medium @ 10..=99`. `medium` is bound to 42. Output: `medium: 42`.

---

## Section C — Fix the Bugs

### A11 — Non-exhaustive match
```rust
enum Season { Spring, Summer, Autumn, Winter }

fn describe(s: Season) -> &'static str {
    match s {
        Season::Spring => "bloom",
        Season::Summer => "hot",
        Season::Autumn => "fall",
        Season::Winter => "cold",   // add the missing arm
    }
}
fn main() { println!("{}", describe(Season::Winter)); } // cold
```

### A12 — Wrong arm types
```rust
fn main() {
    let score = 85u32;
    let grade = match score {
        90..=100 => 'A',
        80..=89  => 'B',   // fix: char, not &str
        70..=79  => 'C',
        _        => 'F',
    };
    println!("{grade}"); // B
}
```
All arms must return the same type. Either make them all `char` or all `&str`.

### A13 — Guard logic fix
The code compiles fine. For `(-1, 2)`: neither guard fires (not both positive, not both negative) → "mixed". That is actually correct behaviour.

The question asks to also handle `(0, anything)` as "mixed". The existing `_` already catches it — `(0, 5)` fails both guards and hits `_`. The existing code is correct.

If you want to be explicit, you can add:
```rust
fn main() {
    let pair = (0, 5);
    let result = match pair {
        (x, y) if x > 0 && y > 0 => "both positive",
        (x, y) if x < 0 && y < 0 => "both negative",
        (0, _) | (_, 0)           => "on an axis",  // explicit zero handling
        _                          => "mixed",
    };
    println!("{result}"); // on an axis
}
```

### A14 — Binding shadowing confusion
```rust
fn main() {
    let x = 5;
    match x {
        x => println!("matched x = {x}"),  // 'x' here is a NEW binding, not the outer x
        1 => println!("one"),               // unreachable: x above catches everything
        _ => println!("other"),             // unreachable
    }
}
```

**Why it compiles but is wrong:** In a `match` arm, a bare name like `x` is a **binding pattern** — it always matches and captures the value. It does **not** compare against the outer `x = 5`. So the first arm catches every value. The remaining arms are unreachable (Rust will warn).

**Fix:** Use a literal to compare, and rename the binding if needed:
```rust
fn main() {
    let x = 5;
    let result = match x {
        1 => "one",
        2 => "two",
        5 => "five",    // literal — checks if value equals 5
        n => { println!("caught {n}"); "other" }
    };
    println!("{result}");
}
```

---

## Section D — Write It Yourself

### A15 — Shape area via match (roadmap practice task)
```rust
use std::f64::consts::PI;

enum Shape { Circle(f64), Rect(f64, f64), Triangle(f64, f64) }

fn area(s: &Shape) -> f64 {
    match s {
        Shape::Circle(r)      => PI * r * r,
        Shape::Rect(w, h)     => w * h,
        Shape::Triangle(b, h) => 0.5 * b * h,
    }
}

fn describe(s: &Shape) -> String {
    match s {
        Shape::Circle(r)      => format!("Circle r={r:.1}  → area={:.2}", area(s)),
        Shape::Rect(w, h)     => format!("Rect {w:.1}×{h:.1} → area={:.2}", area(s)),
        Shape::Triangle(b, h) => format!("Triangle b={b:.1} h={h:.1} → area={:.2}", area(s)),
    }
}

fn main() {
    let shapes = [
        Shape::Circle(5.0),
        Shape::Rect(4.0, 6.0),
        Shape::Triangle(3.0, 8.0),
    ];
    for s in &shapes {
        println!("{}", describe(s));
    }
}
```
**Output:**
```
Circle r=5.0  → area=78.54
Rect 4.0×6.0 → area=24.00
Triangle b=3.0 h=8.0 → area=12.00
```

### A16 — HTTP status classifier
```rust
fn classify_status(code: u32) -> &'static str {
    match code {
        100..=199 => "Informational",
        200..=299 => "Success",
        300..=399 => "Redirection",
        400..=499 => "Client Error",
        500..=599 => "Server Error",
        _         => "Unknown",
    }
}

fn is_ok(code: u32) -> bool {
    matches!(code, 200..=299)
}

fn main() {
    for code in [200, 301, 404, 500, 999] {
        println!("{code}: {} (ok={})", classify_status(code), is_ok(code));
    }
}
```

### A17 — Tuple pattern classifier
```rust
fn describe_coords(x: i32, y: i32) -> &'static str {
    match (x, y) {
        (0, 0)              => "origin",
        (x, y) if x > 0 && y > 0 => "positive quadrant",
        (x, y) if x < 0 && y < 0 => "negative quadrant",
        (_, 0)              => "on x-axis",
        (0, _)              => "on y-axis",
        _                   => "mixed",
    }
}

fn main() {
    let cases = [(0,0),(3,4),(-2,-5),(5,0),(0,-3),(2,-1)];
    for (x, y) in cases {
        println!("({x},{y}) → {}", describe_coords(x, y));
    }
}
```

### A18 — Temperature enum with guards
```rust
#[derive(Debug)]
enum Temperature { Celsius(f64), Fahrenheit(f64), Kelvin(f64) }

impl Temperature {
    fn to_celsius(&self) -> f64 {
        match self {
            Temperature::Celsius(c)    => *c,
            Temperature::Fahrenheit(f) => (f - 32.0) * 5.0 / 9.0,
            Temperature::Kelvin(k)     => k - 273.15,
        }
    }
}

fn classify_temp(t: &Temperature) -> &'static str {
    let c = t.to_celsius();
    match c {
        c if c < 0.0  => "freezing",
        c if c <= 15.0 => "cold",
        c if c <= 25.0 => "comfortable",
        _              => "hot",
    }
}

fn main() {
    let temps = [
        Temperature::Celsius(-10.0),
        Temperature::Fahrenheit(50.0),   // 10°C — cold
        Temperature::Celsius(20.0),
        Temperature::Kelvin(310.0),      // ~36.85°C — hot
    ];
    for t in &temps {
        println!("{:?} → {}", t, classify_temp(t));
    }
}
```

### A19 — Struct destructuring in match
```rust
struct Player { name: String, health: i32, level: u32 }

fn player_status(p: &Player) -> String {
    match p {
        Player { name, health, .. } if *health <= 0 =>
            format!("[DEAD] {name}"),
        Player { name, health, .. } if *health <= 20 =>
            format!("[CRITICAL] {name} (HP:{health})"),
        Player { name, health, level, .. } if *level >= 10 =>
            format!("[{name}] ★Lvl:{level} HP:{health}"),
        Player { name, health, level } =>
            format!("[{name}] Lvl:{level} HP:{health}"),
    }
}

fn main() {
    let players = vec![
        Player { name: "Alice".into(), health: 0,  level: 5  },
        Player { name: "Bob".into(),   health: 15, level: 3  },
        Player { name: "Cara".into(),  health: 80, level: 7  },
        Player { name: "Dave".into(),  health: 95, level: 12 },
    ];
    for p in &players { println!("{}", player_status(p)); }
}
```
**Output:**
```
[DEAD] Alice
[CRITICAL] Bob (HP:15)
[Cara] Lvl:7 HP:80
[Dave] ★Lvl:12 HP:95
```

### A20 — @ binding practice
```rust
fn describe_number(n: i32) -> String {
    match n {
        zero @ 0 =>
            format!("{zero} is zero"),
        neg @ i32::MIN..=-1 =>
            format!("{neg} is a negative number"),
        even @ _ if even % 2 == 0 =>
            format!("{even} is a positive even number"),
        odd =>
            format!("{odd} is a positive odd number"),
    }
}

fn main() {
    for n in [0, -3, 42, 7, -100, 8] {
        println!("{}", describe_number(n));
    }
}
```
**Output:**
```
0 is zero
-3 is a negative number
42 is a positive even number
7 is a positive odd number
-100 is a negative number
8 is a positive even number
```

### A21 — Calculator with match
```rust
enum Op { Add, Sub, Mul, Div }

struct Expr { left: f64, op: Op, right: f64 }

fn evaluate(e: &Expr) -> Result<f64, String> {
    match e.op {
        Op::Add => Ok(e.left + e.right),
        Op::Sub => Ok(e.left - e.right),
        Op::Mul => Ok(e.left * e.right),
        Op::Div => {
            if e.right == 0.0 {
                Err(String::from("division by zero"))
            } else {
                Ok(e.left / e.right)
            }
        }
    }
}

fn main() {
    let exprs = vec![
        Expr { left: 10.0, op: Op::Add, right: 5.0  },
        Expr { left: 10.0, op: Op::Sub, right: 3.0  },
        Expr { left: 6.0,  op: Op::Mul, right: 7.0  },
        Expr { left: 20.0, op: Op::Div, right: 4.0  },
        Expr { left: 5.0,  op: Op::Div, right: 0.0  },
    ];

    for e in &exprs {
        match evaluate(e) {
            Ok(result) => println!("{:.1} {} {:.1} = {:.2}",
                e.left,
                match e.op { Op::Add => "+", Op::Sub => "-", Op::Mul => "×", Op::Div => "÷" },
                e.right, result),
            Err(msg) => println!("{:.1} ÷ {:.1} → Error: {msg}", e.left, e.right),
        }
    }
}
```
**Output:**
```
10.0 + 5.0 = 15.00
10.0 - 3.0 = 7.00
6.0 × 7.0 = 42.00
20.0 ÷ 4.0 = 5.00
5.0 ÷ 0.0 → Error: division by zero
```

---

## Section E — Deep Understanding

### A22 — True or False?
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | `match` is an expression — `let x = match n { ... };` is valid |
| 2 | **True** | Type inference requires all arms to agree on one type |
| 3 | **False** | First matching arm wins, not the last |
| 4 | **False** | `_` matches but does **not** bind — the value is discarded |
| 5 | **True** | Pattern must match first; then the guard is evaluated |
| 6 | **True** | `n @ 1..=9` tests the range and binds the value to `n` |
| 7 | **False** | Adding a variant causes a **compile error** (not silent) in all non-exhaustive matches |
| 8 | **True** | `Variant(a) | OtherVariant(a)` — valid if the bindings are compatible |
| 9 | **True** | `matches!(x, Some(_))` checks if `x` is `Some(anything)` = same as `x.is_some()` |
| 10 | **True** | `Struct { field, .. }` — `..` ignores all other named fields |

### A23 — Trace the match
```
None       → arm 1: "nothing"
Some(0)    → arm 2: "zero"
Some(-5)   → arm 3 guard: -5 < 0 → true → "negative: -5"
Some(200)  → arm 4 guard: 200 > 100 → true → "huge: 200"
Some(42)   → guards 3 and 4 both fail → arm 5: "normal: 42"
```

### A24 — Big debug challenge (6 bugs fixed)
```rust
enum Shape { Circle(f64), Rect(f64, f64) }

// Bug 1 fixed: take &Shape (borrow)
// Bug 2 fixed: added _ arm for future variants
fn area(s: &Shape) -> f64 {
    match s {
        Shape::Circle(r)  => std::f64::consts::PI * r * r,
        Shape::Rect(w, h) => w * h,
        // _ would go here for future variants (currently exhaustive)
    }
}

// Bug 3 fixed: 90..=100 not 90..100
fn grade(score: u32) -> &'static str {
    match score {
        100         => "perfect",
        90..=99     => "A",       // fixed: ..= for inclusive
        80..=89     => "B",
        70..=79     => "C",
        _           => "needs work",
    }
}

// Bug 4 fixed: destructure the tuple first, then use guard
fn describe(pair: (i32, i32)) -> &'static str {
    match pair {
        (x, _) if x > 0 => "positive first",  // fixed: destructure (x, _)
        (0, _)           => "first is zero",
        _                => "other",
    }
}

fn main() {
    let s = Shape::Circle(5.0);
    println!("{:.2}", area(&s));   // Bug 5 fixed: pass &s so s is not moved
    println!("{:.2}", area(&s));   // now works ✅
    println!("{}", grade(95));
    println!("{}", describe((3, 4)));

    let n = 42;
    // Bug 6 fixed: use a literal, not a binding
    match n {
        42 => println!("matched 42"),
        _  => println!("other"),
    }
}
```

**Bug summary:**
1. `area(s: Shape)` — took ownership; changed to `&Shape`
2. Non-exhaustive `match` — covered when the enum is small, but a `_` arm future-proofs it
3. `90..100` — exclusive end misses 100; should be `90..=99` (or keep `100` as its own arm)
4. `x if x.0 > 0` — can't use `.0` before destructuring; must destructure `(x, _)` first
5. `area(s)` called twice — second call uses moved `s`; fix by passing `&s`
6. `n => ...` in match — a bare name always matches (binding pattern), shadowing the outer `n`; use `42 =>` literal

---

## 🏆 Lesson 15 Complete!

✅ `match` as an expression that produces a value  
✅ Exhaustiveness — the compiler guarantees all cases are handled  
✅ Literal, variable binding, and wildcard patterns  
✅ OR patterns `|`, range patterns `..=`  
✅ Tuple, struct, and enum destructuring  
✅ Guards — `if condition` after a pattern  
✅ `@` bindings — test and capture at the same time  
✅ Nested patterns for deep data  
✅ Slice patterns `[first, .., last]`  
✅ `matches!` macro for quick boolean pattern checks  
✅ Shape area via `match` — roadmap practice ✓  

**Up next: [Lesson 16 — if let / while let / let else](../lesson_16_if_let/lesson_16_if_let.md) 🦀**
