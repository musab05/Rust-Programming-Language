# 📘 Lesson 16 — `if let` / `while let` / `let else` (S5)

> **Series:** Rust From Zero · Beginner Level  
> **Roadmap ID:** S5 · Category: 🏗 Structs/Enums  
> **Previous:** [Lesson 15 — Pattern Matching: match](./lesson_15_pattern_matching.md)  
> **Next:** [Lesson 17 — Tuple Structs & Unit Structs](./lesson_17_tuple_unit_structs.md)  
> **Practice:** [Questions](./lesson_16_questions.md) · [Answers](./lesson_16_answers.md)  
> **Practice Task:** Refactor `match` expressions to `if let` where appropriate

---

## Table of Contents

1. [Why Concise Pattern Matching?](#1-why-concise-pattern-matching)
2. [`if let` — Match One Pattern](#2-if-let--match-one-pattern)
3. [`if let` with `else`](#3-if-let-with-else)
4. [`if let` Chains](#4-if-let-chains)
5. [`while let` — Loop While a Pattern Matches](#5-while-let--loop-while-a-pattern-matches)
6. [`let else` — Destructure or Diverge](#6-let-else--destructure-or-diverge)
7. [Nested `if let`](#7-nested-if-let)
8. [Comparing All Three Syntaxes](#8-comparing-all-three-syntaxes)
9. [When to Use `match` vs `if let`](#9-when-to-use-match-vs-if-let)
10. [Common Patterns & Idioms](#10-common-patterns--idioms)
11. [Summary Cheat Sheet](#11-summary-cheat-sheet)

---

## 1. Why Concise Pattern Matching?

`match` is exhaustive and powerful — but sometimes you only care about **one** specific pattern. Writing a full `match` with a wildcard arm just to handle a single case feels verbose:

```rust
// Full match — wordy when you only care about Some
let config: Option<String> = Some(String::from("dark"));

match config {
    Some(theme) => println!("Theme: {theme}"),
    _           => {}   // do nothing — why write this?
}
```

Rust provides three concise alternatives:
- `if let` — run code if one pattern matches
- `while let` — loop while one pattern matches
- `let else` — destructure or exit early

---

## 2. `if let` — Match One Pattern

`if let pattern = value { ... }` runs the block only when the value matches the pattern, binding inner values:

```rust
fn main() {
    let config: Option<String> = Some(String::from("dark"));

    if let Some(theme) = config {
        println!("Theme: {theme}"); // Theme: dark
    }
    // if config was None, nothing happens
}
```

Equivalent `match`:
```rust
match config {
    Some(theme) => println!("Theme: {theme}"),
    _           => {}
}
```

### Works with any pattern, not just Option

```rust
#[derive(Debug)]
enum Coin { Penny, Quarter(String) }  // Quarter carries a state name

fn main() {
    let coin = Coin::Quarter(String::from("Alaska"));

    if let Coin::Quarter(state) = coin {
        println!("Quarter from {state}!");
    }
}
```

### Enum with struct variants

```rust
#[derive(Debug)]
enum Event { Click { x: i32, y: i32 }, KeyPress(char), Other }

fn main() {
    let e = Event::Click { x: 100, y: 200 };

    if let Event::Click { x, y } = e {
        println!("Clicked at ({x}, {y})");
    }
}
```

---

## 3. `if let` with `else`

Add an `else` branch for when the pattern doesn't match:

```rust
fn main() {
    let score: Option<u32> = None;

    if let Some(s) = score {
        println!("Score: {s}");
    } else {
        println!("No score recorded");
    }
}
```

This is exactly equivalent to:
```rust
match score {
    Some(s) => println!("Score: {s}"),
    None    => println!("No score recorded"),
}
```

### When to choose `if let` + `else` vs `match`

Prefer `if let` + `else` when:
- You only care about **one** variant (the `else` is a fallback)
- The match would have `_ => {}` or a trivial fallback

Prefer `match` when:
- You need to handle **multiple** variants differently
- Exhaustiveness checking matters to you
- The code is clearer with all cases explicit

---

## 4. `if let` Chains

You can chain `if let` with regular `if` and even multiple `if let` conditions:

```rust
fn main() {
    let name: Option<&str> = Some("Alice");
    let age: Option<u32>   = Some(30);

    if let Some(n) = name {
        if let Some(a) = age {
            println!("{n} is {a} years old");
        }
    }
}
```

As of Rust 1.64+, `if let` chains in a single expression:

```rust
fn main() {
    let name: Option<&str> = Some("Bob");
    let age:  Option<u32>  = Some(17);

    // chain with && — both must match
    if let (Some(n), Some(a)) = (name, age) {
        if a >= 18 {
            println!("{n} can vote");
        } else {
            println!("{n} cannot vote yet");
        }
    }
}
```

Mix `if let` with regular `if`:

```rust
fn main() {
    let value: Option<i32> = Some(42);

    if let Some(v) = value && v > 10 {  // Rust 1.79+ let-chain syntax
        println!("big value: {v}");
    }
}
```

---

## 5. `while let` — Loop While a Pattern Matches

`while let` keeps looping as long as the pattern matches:

```rust
fn main() {
    let mut stack = vec![1, 2, 3, 4, 5];

    while let Some(top) = stack.pop() {
        print!("{top} "); // 5 4 3 2 1
    }
    println!();
    // loop stops when pop() returns None (stack is empty)
}
```

Equivalent `loop`:
```rust
loop {
    match stack.pop() {
        Some(top) => print!("{top} "),
        None      => break,
    }
}
```

### Another common use — draining a channel

```rust
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();
    tx.send(1).unwrap();
    tx.send(2).unwrap();
    tx.send(3).unwrap();
    drop(tx); // close the channel

    while let Ok(msg) = rx.recv() {
        println!("received: {msg}");
    }
}
```

### `while let` with an iterator

```rust
fn main() {
    let mut iter = vec!["a", "b", "c"].into_iter();

    while let Some(item) = iter.next() {
        println!("{item}");
    }
}
```

Though in practice, a `for` loop is cleaner for iterators — `while let` shines when you're manually managing state.

---

## 6. `let else` — Destructure or Diverge

`let else` was stabilised in Rust 1.65. It's the most unique of the three:

```rust
let pattern = value else {
    // must diverge: return, break, continue, panic!, or loop forever
};
```

It says: **"destructure this value, or escape"**. The binding is available in the code *after* the `let else`, not inside the `else` block.

```rust
fn describe_coin(coin: Option<u32>) {
    let Some(value) = coin else {
        println!("No coin!");
        return;             // must diverge
    };

    // 'value' is available here — we know it's Some
    println!("Coin value: {value} cents");
}

fn main() {
    describe_coin(Some(25));
    describe_coin(None);
}
```

### The key difference from `if let`

With `if let`, the binding lives **inside** the `if` block:
```rust
if let Some(v) = coin {
    // v available here
}
// v NOT available here
```

With `let else`, the binding lives **after** the statement:
```rust
let Some(v) = coin else { return; };
// v available HERE and below — in the normal flow
```

### Real-world example — parsing input

```rust
fn parse_age(input: &str) -> u32 {
    let Ok(n) = input.trim().parse::<u32>() else {
        println!("Invalid age: '{input}'");
        return 0;
    };

    // n is valid u32 — use it directly
    if n > 150 {
        println!("Suspiciously old!");
    }
    n
}

fn main() {
    println!("{}", parse_age("25"));      // 25
    println!("{}", parse_age("hello"));   // Invalid age, returns 0
    println!("{}", parse_age("  30  ")); // 30
}
```

### Early-return pattern — replacing deep nesting

Before `let else` you'd write:
```rust
fn process(data: Option<Vec<i32>>) -> String {
    if let Some(v) = data {
        if !v.is_empty() {
            format!("first = {}", v[0])
        } else {
            String::from("empty vec")
        }
    } else {
        String::from("no data")
    }
}
```

With `let else` — flat, readable:
```rust
fn process(data: Option<Vec<i32>>) -> String {
    let Some(v) = data else { return String::from("no data"); };
    let Some(&first) = v.first() else { return String::from("empty vec"); };
    format!("first = {first}")
}
```

---

## 7. Nested `if let`

`if let` can be nested, though `let else` is often cleaner for deeply nested cases:

```rust
#[derive(Debug)]
struct Config { server: Option<ServerConfig> }

#[derive(Debug)]
struct ServerConfig { host: Option<String>, port: u16 }

fn main() {
    let cfg = Config {
        server: Some(ServerConfig {
            host: Some(String::from("localhost")),
            port: 8080,
        }),
    };

    // Nested if let
    if let Some(server) = &cfg.server {
        if let Some(host) = &server.host {
            println!("Connecting to {host}:{}", server.port);
        }
    }

    // Flatter with let else
    let Some(server) = &cfg.server else { return; };
    let Some(host)   = &server.host else { return; };
    println!("Connecting to {host}:{}", server.port);
}
```

---

## 8. Comparing All Three Syntaxes

Same goal — handle `Option<i32>`, print if Some:

```rust
let val: Option<i32> = Some(42);

// match — verbose but exhaustive
match val {
    Some(n) => println!("got {n}"),
    None    => {}
}

// if let — concise for one pattern
if let Some(n) = val {
    println!("got {n}");
}

// if let + else — concise with fallback
if let Some(n) = val {
    println!("got {n}");
} else {
    println!("nothing");
}

// let else — binds for use below, or exits
let Some(n) = val else { return; };
println!("got {n}");
// n available here too
```

---

## 9. When to Use `match` vs `if let`

| Situation | Best choice |
|-----------|------------|
| Handle all variants differently | `match` |
| Only care about one variant | `if let` |
| Loop until pattern stops matching | `while let` |
| Destructure or bail out early | `let else` |
| Multiple patterns with OR | `match` (cleaner) |
| Guard conditions per variant | `match` (cleaner) |
| Simple "is it Some/Ok?" check | `if let` or `.is_some()` |

---

## 10. Common Patterns & Idioms

### Unwrap-or-return (the most common `let else` use)

```rust
fn process(s: &str) -> Option<u32> {
    let Ok(n) = s.parse::<u32>() else { return None; };
    Some(n * 2)
}
```

### Consuming from a stack/queue

```rust
let mut queue = std::collections::VecDeque::from([1, 2, 3]);
while let Some(front) = queue.pop_front() {
    println!("{front}");
}
```

### Skipping None values

```rust
let items = vec![Some(1), None, Some(3), None, Some(5)];
for item in &items {
    let Some(v) = item else { continue; };
    println!("{v}");  // prints 1, 3, 5
}
```

### Extracting from a Result

```rust
fn read_port(env_var: &str) -> u16 {
    let Ok(val) = std::env::var(env_var) else {
        return 3000; // default port
    };
    let Ok(port) = val.parse::<u16>() else {
        return 3000;
    };
    port
}
```

---

## 11. Summary Cheat Sheet

```
IF LET
────────────────────────────────────────────────────────────
if let Pattern = value {
    // runs if value matches Pattern
    // inner bindings available here
} else {
    // runs if value doesn't match
}

Binding lives INSIDE the if block only.
Use when: one pattern to handle, rest is a fallback or ignored.

WHILE LET
────────────────────────────────────────────────────────────
while let Pattern = expression {
    // runs each iteration while expression matches Pattern
}

Stops looping when the pattern no longer matches.
Use when: looping until a value changes state (e.g. None).

LET ELSE
────────────────────────────────────────────────────────────
let Pattern = value else {
    // MUST diverge: return / break / continue / panic!()
};
// Pattern bindings available HERE (after the statement)

Binding lives AFTER the statement in normal flow.
Use when: destructuring at the start of a function, early-return on failure.

CHOOSE BY SITUATION
────────────────────────────────────────────────────────────
Many variants differently    → match
One variant, ignore rest     → if let
One variant, else fallback   → if let / else
Loop while matches           → while let
Destructure or bail early    → let else
```

---

## What's Next?

**Lesson 17 — Tuple Structs & Unit Structs** — Creating new distinct types with tuple-struct syntax and zero-sized marker types.

## Further Reading
- [The Rust Book — Ch 6.3: Concise Control Flow with if let](https://doc.rust-lang.org/book/ch06-03-if-let.html)
- [Rust Reference — if let expressions](https://doc.rust-lang.org/reference/expressions/if-expr.html#if-let-expressions)
- [let-else RFC](https://rust-lang.github.io/rfcs/3137-let-else.html)
