# 📘 Lesson 15 — Pattern Matching: `match` (S4)

> **Series:** Rust From Zero · Beginner Level  
> **Roadmap ID:** S4 · Category: 🏗 Structs/Enums  
> **Previous:** [Lesson 14 — Enums](./lesson_14_enums.md)  
> **Next:** [Lesson 16 — if let / while let / let else](./lesson_16_if_let.md)  
> **Practice:** [Questions](./lesson_15_questions.md) · [Answers](./lesson_15_answers.md)  
> **Practice Task:** Use `match` on a `Shape` enum and compute the area for each variant

---

## Table of Contents

1. [What Is Pattern Matching?](#1-what-is-pattern-matching)
2. [The `match` Expression — Basics](#2-the-match-expression--basics)
3. [Match Is Exhaustive](#3-match-is-exhaustive)
4. [Literal Patterns](#4-literal-patterns)
5. [Variable (Binding) Patterns](#5-variable-binding-patterns)
6. [Wildcard `_` and Catch-All `other`](#6-wildcard-_-and-catch-all-other)
7. [OR Patterns — `|`](#7-or-patterns--)
8. [Tuple and Struct Destructuring](#8-tuple-and-struct-destructuring)
9. [Enum Destructuring](#9-enum-destructuring)
10. [Guards — `if` Conditions Inside Arms](#10-guards--if-conditions-inside-arms)
11. [Binding with `@`](#11-binding-with-)
12. [Range Patterns](#12-range-patterns)
13. [Nested Patterns](#13-nested-patterns)
14. [Slice Patterns](#14-slice-patterns)
15. [The `matches!` Macro](#15-the-matches-macro)
16. [Summary Cheat Sheet](#16-summary-cheat-sheet)

---

## 1. What Is Pattern Matching?

Pattern matching compares a value against a set of **shapes** and runs the code for the first shape that fits, optionally **destructuring** the value into its parts.

Think of it as a supercharged `switch`/`if-else` that:
- Works on any type (integers, enums, structs, tuples, slices)
- Destructures and names inner values in the same step
- Is checked by the compiler for **completeness** — you can't miss a case

```rust
match value {
    pattern_1 => expression_1,
    pattern_2 => expression_2,
    _         => default_expression,
}
```

---

## 2. The `match` Expression — Basics

`match` is an **expression** — it produces a value:

```rust
fn main() {
    let n = 3;

    let description = match n {
        1 => "one",
        2 => "two",
        3 => "three",
        _ => "something else",
    };
    println!("{description}"); // three
}
```

Key rules:
- Arms are evaluated **top to bottom**; first match wins
- All arms must produce the **same type**
- The match must be **exhaustive** — every possible value covered
- Multi-line arm bodies use `{ }`

```rust
let result = match n {
    1 => {
        println!("got one!");
        "one"
    }
    2 | 3 => "two or three",  // OR pattern
    _ => "other",
};
```

---

## 3. Match Is Exhaustive

The compiler checks that every possible input is covered. If you miss a case, it won't compile:

```rust
enum Color { Red, Green, Blue }

fn describe(c: Color) -> &'static str {
    match c {
        Color::Red   => "red",
        Color::Green => "green",
        // ❌ COMPILE ERROR: missing Color::Blue
    }
}
```

This is one of Rust's most powerful safety features — you can never silently forget a case.

Adding a new variant to an enum causes every `match` on it to produce a compile error until you handle the new case. Your entire codebase is automatically updated.

---

## 4. Literal Patterns

Match against specific literal values:

```rust
fn http_status(code: u32) -> &'static str {
    match code {
        200 => "OK",
        201 => "Created",
        301 => "Moved Permanently",
        404 => "Not Found",
        500 => "Internal Server Error",
        _   => "Unknown",
    }
}

fn main() {
    println!("{}", http_status(200)); // OK
    println!("{}", http_status(999)); // Unknown
}
```

Works on integers, `bool`, `char`, and string slices (`&str`):

```rust
fn greet(lang: &str) -> &str {
    match lang {
        "en" => "Hello",
        "es" => "Hola",
        "fr" => "Bonjour",
        "de" => "Hallo",
        _    => "Hi",
    }
}
```

---

## 5. Variable (Binding) Patterns

A plain name in a pattern **binds** the matched value to that name:

```rust
fn classify(n: i32) -> String {
    match n {
        0         => String::from("zero"),
        positive if positive > 0 => format!("{positive} is positive"),
        negative  => format!("{negative} is negative"),  // binds any i32
    }
}
```

When you write a name (not a literal), it's a binding — it always matches and captures the value.

---

## 6. Wildcard `_` and Catch-All `other`

`_` matches any value and **discards** it (no binding created):

```rust
match some_value {
    1 => println!("one"),
    2 => println!("two"),
    _ => println!("something else"), // match-all, discard value
}
```

To match-all **and keep** the value, use a name instead:

```rust
match some_value {
    1 => println!("one"),
    other => println!("got {other}"),  // binds the value
}
```

`_` can also ignore specific fields:

```rust
struct Point { x: i32, y: i32, z: i32 }

let p = Point { x: 1, y: 2, z: 3 };
match p {
    Point { x, .. } => println!("x = {x}"),  // .. ignores y and z
}
```

---

## 7. OR Patterns — `|`

Match multiple patterns in a single arm:

```rust
fn is_weekend(day: &str) -> bool {
    match day {
        "Saturday" | "Sunday" => true,
        _ => false,
    }
}

fn grade(score: u32) -> char {
    match score {
        90..=100 => 'A',
        80..=89  => 'B',
        70..=79  => 'C',
        60..=69  => 'D',
        _        => 'F',
    }
}
```

OR patterns with enum variants:

```rust
#[derive(Debug)]
enum Command { Quit, Move(i32, i32), Write(String), Resize(u32) }

fn is_gui_event(cmd: &Command) -> bool {
    matches!(cmd, Command::Move(_, _) | Command::Resize(_))
}
```

---

## 8. Tuple and Struct Destructuring

### Tuples

```rust
fn describe_point(point: (i32, i32)) -> &'static str {
    match point {
        (0, 0) => "origin",
        (x, 0) => "on x-axis",   // x binds the first element
        (0, y) => "on y-axis",
        (x, y) => "somewhere",
    }
}

fn main() {
    println!("{}", describe_point((0, 0)));  // origin
    println!("{}", describe_point((5, 0)));  // on x-axis
    println!("{}", describe_point((3, 7)));  // somewhere
}
```

### Structs

```rust
struct Point { x: i32, y: i32 }

fn main() {
    let p = Point { x: 3, y: -5 };

    match p {
        Point { x: 0, y }         => println!("on y-axis at {y}"),
        Point { x, y: 0 }         => println!("on x-axis at {x}"),
        Point { x, y } if x == y  => println!("on diagonal: {x}"),
        Point { x, y }            => println!("at ({x}, {y})"),
    }
}
```

Shorthand when the field name equals the binding name:

```rust
match p {
    Point { x, y } => println!("{x} {y}"),  // binds p.x to x, p.y to y
}
```

---

## 9. Enum Destructuring

The most common and important use of `match`:

```rust
#[derive(Debug)]
enum Shape { Circle(f64), Rect(f64, f64), Triangle(f64, f64) }

fn area(s: &Shape) -> f64 {
    match s {
        Shape::Circle(r)      => std::f64::consts::PI * r * r,
        Shape::Rect(w, h)     => w * h,
        Shape::Triangle(b, h) => 0.5 * b * h,
    }
}
```

This is the **roadmap practice task** — matching on Shape to compute area.

### Struct-style enum variants

```rust
#[derive(Debug)]
enum Event {
    Click { x: i32, y: i32 },
    KeyPress { key: char, shift: bool },
}

fn handle(e: &Event) {
    match e {
        Event::Click { x, y }            => println!("click at ({x},{y})"),
        Event::KeyPress { key, shift: true }  => println!("SHIFT+{key}"),
        Event::KeyPress { key, shift: false } => println!("key {key}"),
    }
}
```

---

## 10. Guards — `if` Conditions Inside Arms

A guard adds an extra condition after the pattern:

```rust
fn classify(n: i32) -> &'static str {
    match n {
        x if x < 0   => "negative",
        0             => "zero",
        x if x % 2 == 0 => "positive even",
        _             => "positive odd",
    }
}

fn main() {
    println!("{}", classify(-5));  // negative
    println!("{}", classify(0));   // zero
    println!("{}", classify(4));   // positive even
    println!("{}", classify(7));   // positive odd
}
```

Guards are checked **after** the pattern matches:

```rust
let pair = (2, -2);
match pair {
    (x, y) if x == y  => println!("equal"),
    (x, y) if x + y == 0 => println!("sum is zero"),
    (x, _)             => println!("first is {x}"),
}
```

---

## 11. Binding with `@`

`@` binds a value to a name **while also testing it** against a pattern:

```rust
fn categorize(n: u32) -> String {
    match n {
        small @ 1..=9   => format!("{small} is a single digit"),
        teen  @ 10..=19 => format!("{teen} is a teen"),
        big             => format!("{big} is something else"),
    }
}

fn main() {
    println!("{}", categorize(5));   // 5 is a single digit
    println!("{}", categorize(15));  // 15 is a teen
    println!("{}", categorize(42));  // 42 is something else
}
```

`@` is useful when you need the value **and** to test it against a range or complex pattern:

```rust
enum Message { Hello { id: i32 } }

let msg = Message::Hello { id: 7 };
match msg {
    Message::Hello { id: id_var @ 1..=12 } => println!("id {id_var} in range"),
    Message::Hello { id }                  => println!("id {id} out of range"),
}
```

---

## 12. Range Patterns

Match numeric ranges with `..=` (inclusive):

```rust
fn age_group(age: u32) -> &'static str {
    match age {
        0..=12  => "child",
        13..=17 => "teenager",
        18..=64 => "adult",
        65..=   => "senior",
    }
}
```

Note: open-ended ranges (`65..`) are allowed in patterns. Exclusive end (`..`) is not supported in patterns — use `..=` or a guard.

```rust
fn grade(score: u32) -> char {
    match score {
        90..=100 => 'A',
        80..=89  => 'B',
        70..=79  => 'C',
        60..=69  => 'D',
        0..=59   => 'F',
        _        => unreachable!("score > 100"),
    }
}
```

---

## 13. Nested Patterns

Patterns can be nested arbitrarily deep:

```rust
#[derive(Debug)]
enum Color { Rgb(u8, u8, u8), Named(&'static str) }

#[derive(Debug)]
enum Shape { Circle { color: Color, radius: f64 }, Square(f64) }

fn describe(s: &Shape) {
    match s {
        Shape::Circle { color: Color::Named(name), radius } =>
            println!("{name} circle, r={radius}"),
        Shape::Circle { color: Color::Rgb(r, g, b), radius } =>
            println!("rgb({r},{g},{b}) circle, r={radius}"),
        Shape::Square(side) =>
            println!("square, side={side}"),
    }
}
```

---

## 14. Slice Patterns

Match on slices by their structure:

```rust
fn first_and_last(data: &[i32]) -> String {
    match data {
        []              => String::from("empty"),
        [x]             => format!("single: {x}"),
        [first, .., last] => format!("first={first}, last={last}"),
    }
}

fn main() {
    println!("{}", first_and_last(&[]));         // empty
    println!("{}", first_and_last(&[5]));         // single: 5
    println!("{}", first_and_last(&[1, 2, 3, 4])); // first=1, last=4
}
```

`..` in a slice pattern matches any number of middle elements.

---

## 15. The `matches!` Macro

`matches!` is a shorthand that returns `true` if a value matches a pattern. It's cleaner than a full `match` when you just need a bool:

```rust
#[derive(Debug)]
enum Status { Active, Inactive, Banned }

fn main() {
    let s = Status::Active;

    // Without matches!:
    let is_active = match s { Status::Active => true, _ => false };

    // With matches!:
    let is_active = matches!(s, Status::Active);

    // Supports OR patterns and guards:
    let is_usable = matches!(s, Status::Active | Status::Inactive);
    println!("{is_active} {is_usable}");
}
```

---

## 16. Summary Cheat Sheet

```
MATCH BASICS
────────────────────────────────────────────────────────────
match value {
    pattern => expression,   // single expression arm
    pattern => { block }     // multi-line arm
}
match is an expression — it produces a value
Arms evaluated top to bottom — first match wins
ALL arms must return the same type

PATTERN TYPES
────────────────────────────────────────────────────────────
42              literal match
'a'..='z'       range (inclusive)
x               binding — always matches, captures value
_               wildcard — always matches, discards value
..              ignore remaining fields/elements

(a, b)          tuple destructuring
Point { x, y }  struct destructuring
Point { x, .. } struct with ignored fields

Variant(a, b)   tuple enum variant
Variant { f }   struct enum variant
Some(x)         Option::Some
None            Option::None

p1 | p2         OR — matches either
x @ 1..=9       @ binding with pattern test

GUARDS
────────────────────────────────────────────────────────────
pat if condition => expr    extra condition after pattern

SLICE PATTERNS
────────────────────────────────────────────────────────────
[]              empty slice
[x]             exactly one element
[a, b]          exactly two
[first, ..]     first + rest (any length)
[.., last]      any + last
[first, .., last] first, middle ignored, last

MATCHES! MACRO
────────────────────────────────────────────────────────────
matches!(value, pattern)              → bool
matches!(value, p1 | p2)             → bool OR
matches!(value, p if condition)      → bool with guard
```

---

## What's Next?

**Lesson 16 — if let / while let / let else** — Concise alternatives to `match` when you only care about one pattern.

## Further Reading
- [The Rust Book — Ch 6.2: The match Control Flow Construct](https://doc.rust-lang.org/book/ch06-02-match.html)
- [The Rust Book — Ch 18: Patterns and Matching](https://doc.rust-lang.org/book/ch18-00-patterns.html)
- [Rust by Example — Patterns](https://doc.rust-lang.org/rust-by-example/flow_control/match.html)
