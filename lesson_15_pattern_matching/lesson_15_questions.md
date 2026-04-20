# 🧪 Lesson 15 — Questions: Pattern Matching: `match` (S4)

> **Lesson:** [lesson_15_pattern_matching.md](./lesson_15_pattern_matching.md)  
> **Answers:** [lesson_15_answers.md](./lesson_15_answers.md)

---

## Section A — Predict the Output

### Q1
```rust
fn main() {
    let n = 7;
    let result = match n {
        1         => "one",
        2 | 3     => "two or three",
        4..=6     => "four to six",
        x if x % 2 == 0 => "even",
        _         => "odd and > 6",
    };
    println!("{result}");
}
```

### Q2
```rust
fn main() {
    let pair = (0, -5);
    match pair {
        (0, y)          => println!("x=0, y={y}"),
        (x, 0)          => println!("y=0, x={x}"),
        (x, y) if x == y => println!("equal: {x}"),
        (x, y)          => println!("({x},{y})"),
    }
}
```

### Q3
```rust
#[derive(Debug)]
enum Coin { Penny, Nickel, Dime, Quarter }

fn cents(c: &Coin) -> u32 {
    match c {
        Coin::Penny   => 1,
        Coin::Nickel  => 5,
        Coin::Dime    => 10,
        Coin::Quarter => 25,
    }
}

fn main() {
    let wallet = [Coin::Quarter, Coin::Dime, Coin::Penny, Coin::Quarter];
    let total: u32 = wallet.iter().map(cents).sum();
    println!("{total}");
}
```

### Q4
```rust
fn main() {
    let values = vec![Some(1), None, Some(3), Some(4), None];
    for v in &values {
        match v {
            Some(x) if *x > 2 => println!("big: {x}"),
            Some(x)           => println!("small: {x}"),
            None              => println!("none"),
        }
    }
}
```

### Q5
```rust
fn describe(data: &[i32]) -> String {
    match data {
        []              => String::from("empty"),
        [x]             => format!("one: {x}"),
        [a, b]          => format!("two: {a},{b}"),
        [first, .., last] => format!("many: {first}..{last}"),
    }
}
fn main() {
    println!("{}", describe(&[]));
    println!("{}", describe(&[99]));
    println!("{}", describe(&[1, 2]));
    println!("{}", describe(&[10, 20, 30, 40]));
}
```

---

## Section B — Will It Compile?

### Q6
```rust
enum Direction { North, South, East }

fn describe(d: Direction) -> &'static str {
    match d {
        Direction::North => "north",
        Direction::South => "south",
        // Direction::East missing
    }
}
fn main() { println!("{}", describe(Direction::North)); }
```

### Q7
```rust
fn main() {
    let x = 5u32;
    let msg = match x {
        1 => "one",
        2 => "two",
        _ => "other",
    };
    println!("{msg}");
}
```

### Q8
```rust
fn main() {
    let t = (1, 2, 3);
    match t {
        (1, y, z) => println!("{y} {z}"),
        (x, 2, z) => println!("{x} {z}"),
        _         => println!("other"),
    }
}
```

### Q9
```rust
fn main() {
    let x = 10;
    let result = match x {
        n if n < 0  => "negative",
        0           => "zero",
        _           => "positive",
    };
    println!("{result}");
    println!("{x}");   // is x still usable?
}
```

### Q10
```rust
fn main() {
    let n = 42;
    match n {
        small @ 1..=9   => println!("small: {small}"),
        medium @ 10..=99 => println!("medium: {medium}"),
        large           => println!("large: {large}"),
    }
}
```

---

## Section C — Fix the Bugs

### Q11 — Non-exhaustive match
```rust
enum Season { Spring, Summer, Autumn, Winter }

fn describe(s: Season) -> &'static str {
    match s {
        Season::Spring => "bloom",
        Season::Summer => "hot",
        Season::Autumn => "fall",
        // Winter missing!
    }
}
fn main() { println!("{}", describe(Season::Winter)); }
```

### Q12 — Wrong arm types
```rust
fn main() {
    let score = 85u32;
    let grade = match score {
        90..=100 => 'A',
        80..=89  => "B",   // BUG: char vs &str
        70..=79  => 'C',
        _        => 'F',
    };
    println!("{grade}");
}
```

### Q13 — Guard doesn't compile
```rust
fn main() {
    let pair = (2, 3);
    let result = match pair {
        (x, y) if x > 0 && y > 0 => "both positive",
        (x, y) if x < 0 && y < 0 => "both negative",
        _                          => "mixed",
    };
    println!("{result}");
}
// This actually compiles — spot what's wrong with the LOGIC instead:
// What does this print for (-1, 2)?  Is that the right answer?
// Fix the match to also handle (0, anything) as "mixed".
```

### Q14 — Binding shadowing confusion
```rust
fn main() {
    let x = 5;
    match x {
        x => println!("matched x = {x}"),  // BUG: this always matches!
        1 => println!("one"),               // unreachable
        _ => println!("other"),
    }
}
```
Explain why this compiles but the last two arms are unreachable. Fix it.

---

## Section D — Write It Yourself

### Q15 — Roadmap Practice Task: Shape area via match
This is the exact roadmap task. Build the `Shape` enum and compute area using `match`:
```rust
enum Shape { Circle(f64), Rect(f64, f64), Triangle(f64, f64) }
```
- `fn area(s: &Shape) -> f64` using `match`
- `fn describe(s: &Shape) -> String` — human description with computed area
- In `main`: create one of each, print descriptions

### Q16 — HTTP status classifier
Write `fn classify_status(code: u32) -> &'static str` using `match` with ranges:
- 100..=199 → "Informational"
- 200..=299 → "Success"
- 300..=399 → "Redirection"
- 400..=499 → "Client Error"
- 500..=599 → "Server Error"
- _ → "Unknown"

Also write `fn is_ok(code: u32) -> bool` using `matches!`.

### Q17 — Tuple pattern classifier
Write `fn describe_coords(x: i32, y: i32) -> &'static str` that uses a match on `(x, y)` to return:
- "origin" for (0,0)
- "positive quadrant" if both > 0
- "negative quadrant" if both < 0
- "on x-axis" if y == 0 (any x)
- "on y-axis" if x == 0 (any y)
- "mixed" otherwise

### Q18 — Enum match with guards
```rust
#[derive(Debug)]
enum Temperature { Celsius(f64), Fahrenheit(f64), Kelvin(f64) }
```
Write `fn classify_temp(t: &Temperature) -> &'static str` that returns:
- "freezing" if below 0°C equivalent
- "cold" if 0–15°C
- "comfortable" if 15–25°C
- "hot" if above 25°C

Convert all to Celsius inside the match arms (or use guards).

### Q19 — Struct destructuring in match
```rust
struct Player { name: String, health: i32, level: u32 }
```
Write `fn player_status(p: &Player) -> String` that returns:
- `"[DEAD] name"` if health <= 0
- `"[CRITICAL] name (HP:n)"` if health is 1–20
- `"[name] Lvl:l HP:h"` for levels 1–9
- `"[name] ★Lvl:l HP:h"` for level 10+

Use struct destructuring in the match arms.

### Q20 — @ binding practice
Write `fn describe_number(n: i32) -> String` that uses `@` bindings to produce messages like:
- `"42 is a positive even number"`
- `"-3 is a negative number"`
- `"0 is zero"`
- `"7 is a positive odd number"`

Show the exact value in the message using the `@` binding.

### Q21 — Full program: Calculator with match
Build a simple expression evaluator:
```rust
enum Op { Add, Sub, Mul, Div }
struct Expr { left: f64, op: Op, right: f64 }
```
- `fn evaluate(e: &Expr) -> Result<f64, String>` — returns `Err` for division by zero
- In `main`: evaluate 5 expressions (including one division by zero), print results

---

## Section E — Deep Understanding

### Q22 — True or False?
1. `match` in Rust is an expression — it can be assigned to a variable.
2. All arms of a `match` must return the same type.
3. Arms are evaluated in order; if multiple patterns match, the last one wins.
4. `_` binds the matched value to the name `_`.
5. A guard (`if condition`) is checked after the pattern matches.
6. `n @ 1..=9` both tests that n is in 1..=9 **and** binds n to that value.
7. Adding a new variant to an enum will silently break `match` expressions that don't cover it.
8. OR patterns (`|`) can be used inside enum variant destructuring.
9. `matches!(x, Some(_))` is equivalent to `x.is_some()` when `x: Option<T>`.
10. `..` in a struct pattern ignores remaining named fields.

### Q23 — Trace the match
For each input, trace which arm fires and what gets printed:

```rust
fn process(val: Option<i32>) -> String {
    match val {
        None               => String::from("nothing"),
        Some(0)            => String::from("zero"),
        Some(n) if n < 0   => format!("negative: {n}"),
        Some(n) if n > 100 => format!("huge: {n}"),
        Some(n)            => format!("normal: {n}"),
    }
}
```
Trace for: `None`, `Some(0)`, `Some(-5)`, `Some(200)`, `Some(42)`.

### Q24 — Big debug challenge (6 bugs)
Find and fix every bug:
```rust
enum Shape { Circle(f64), Rect(f64, f64) }

fn area(s: Shape) -> f64 {            // Bug 1: should borrow
    match s {
        Shape::Circle(r)   => 3.14 * r * r,
        Shape::Rect(w, h)  => w * h,
        // Bug 2: non-exhaustive if we add a variant later — add _ arm
    }
}

fn grade(score: u32) -> &'static str {
    match score {
        100         => "perfect",
        90..100     => "A",           // Bug 3: should be ..= for inclusive
        80..=89     => "B",
        70..=79     => "C",
        _           => "needs work",
    }
}

fn describe(pair: (i32, i32)) -> &'static str {
    match pair {
        x if x.0 > 0 => "positive first",  // Bug 4: can't guard on whole tuple like this when binding
        (0, _)        => "first is zero",
        _             => "other",
    }
}

fn main() {
    let s = Shape::Circle(5.0);
    println!("{:.2}", area(s));
    println!("{}", area(s));        // Bug 5: s was moved
    println!("{}", grade(95));
    println!("{}", describe((3, 4)));

    let n = 42;
    match n {
        n => println!("matched: {n}"),  // Bug 6: shadows outer n, always fires
        _ => println!("other"),         // unreachable
    }
}
```
