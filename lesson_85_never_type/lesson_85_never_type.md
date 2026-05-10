# 📘 Lesson 85 — Never Type (!) (AT3)

> **Series:** Rust From Zero · Advanced Level (Gap Fill)  
> **Roadmap ID:** AT3 · Category: 🔷 Advanced Types  
> **Previous:** [Lesson 84 — Async Streams & Channels](../lesson_84_async_streams/lesson_84_async_streams.md)  
> **Next:** [Lesson 86 — Dynamically Sized Types](../lesson_86_dst/lesson_86_dst.md)  
> **Practice:** [Questions](./lesson_85_questions.md) · [Answers](./lesson_85_answers.md)  
> **Practice Task:** Write a function that returns ! and explain when useful

---

## Table of Contents

1. [What Is the Never Type?](#1-what-is-the-never-type)
2. [Diverging Functions](#2-diverging-functions)
3. [Where ! Appears](#3-where--appears)
4. [! in Match Arms](#4--in-match-arms)
5. [! with loop](#5--with-loop)
6. [! with continue and break](#6--with-continue-and-break)
7. [! as a Return Type](#7--as-a-return-type)
8. [Practical Uses](#8-practical-uses)
9. [Summary Cheat Sheet](#9-summary-cheat-sheet)

---

## 1. What Is the Never Type?

`!` is the type for expressions that **never produce a value** — they diverge:

```rust
// This function NEVER returns
fn infinite_loop() -> ! {
    loop {
        // runs forever
    }
}

// This function NEVER returns (always panics)
fn always_panics() -> ! {
    panic!("I never return!");
}

// This function NEVER returns (always exits)
fn always_exits() -> ! {
    std::process::exit(1);
}
```

**Key property:** `!` can coerce into ANY type. This is why `panic!()` works everywhere.

---

## 2. Diverging Functions

A function is "diverging" if it never returns normally:

```rust
fn divide(a: i32, b: i32) -> i32 {
    if b == 0 {
        panic!("division by zero");  // panic! returns !
        // ! coerces to i32, so the match arm types are consistent
    }
    a / b
}

fn main() {
    println!("{}", divide(10, 2));  // 5
    // println!("{}", divide(10, 0));  // panics
}
```

### Common diverging expressions:

```rust
fn demo() {
    let _: ! = panic!("crash");         // panic! → !
    let _: ! = todo!("not yet");        // todo! → !
    let _: ! = unimplemented!();        // unimplemented! → !
    let _: ! = unreachable!();          // unreachable! → !
    let _: ! = loop {};                 // infinite loop → !
    let _: ! = std::process::exit(0);   // exit → !
    let _: ! = std::process::abort();   // abort → !
}
```

---

## 3. Where ! Appears

### panic! in if/else:

```rust
fn get_value(opt: Option<i32>) -> i32 {
    // Both arms must return the same type
    // panic! returns !, which coerces to i32 — so this works
    if let Some(val) = opt {
        val        // i32
    } else {
        panic!("no value!")  // ! coerces to i32
    }
}
```

### unwrap() uses !:

```rust
// This is essentially how unwrap() works:
fn my_unwrap<T>(opt: Option<T>) -> T {
    match opt {
        Some(val) => val,     // T
        None => panic!("called unwrap on None"),  // ! coerces to T
    }
}
```

---

## 4. ! in Match Arms

```rust
fn describe(x: i32) -> &'static str {
    match x {
        0 => "zero",
        1..=100 => "small",
        _ => {
            if x < 0 {
                "negative"
            } else {
                panic!("too large!")  // ! coerces to &str
            }
        }
    }
}

fn main() {
    println!("{}", describe(42));
    println!("{}", describe(-5));
}
```

### Pattern matching with Result:

```rust
fn parse_or_die(s: &str) -> i32 {
    match s.parse::<i32>() {
        Ok(n) => n,                              // i32
        Err(e) => panic!("Parse error: {e}"),    // ! → i32
    }
}

// More idiomatic: unwrap_or_else
fn parse_or_default(s: &str) -> i32 {
    s.parse().unwrap_or_else(|_| {
        eprintln!("Warning: '{s}' is not a number, using 0");
        0
    })
}
```

---

## 5. ! with loop

An infinite `loop` without `break` has type `!`:

```rust
// Server that runs forever
fn run_server() -> ! {
    loop {
        // accept connections
        // handle requests
        println!("Handling request...");
        std::thread::sleep(std::time::Duration::from_secs(1));
    }
}

// loop with break has a concrete type (the break value)
fn find_answer() -> i32 {
    let mut x = 0;
    loop {
        x += 1;
        if x * x > 100 {
            break x;  // loop produces i32 (not !)
        }
    }
}

fn main() {
    println!("Answer: {}", find_answer());  // 11
    // run_server();  // would run forever
}
```

---

## 6. ! with continue and break

`continue` and `break` also have type `!`:

```rust
fn first_positive(numbers: &[i32]) -> Option<i32> {
    for &n in numbers {
        let positive = if n > 0 {
            n                // i32
        } else {
            continue         // ! coerces to i32
        };
        return Some(positive);
    }
    None
}

fn main() {
    let nums = vec![-3, -1, 0, 5, 7];
    println!("{:?}", first_positive(&nums));  // Some(5)
}
```

---

## 7. ! as a Return Type

```rust
// Error handler that logs and exits
fn fatal_error(msg: &str) -> ! {
    eprintln!("FATAL: {msg}");
    std::process::exit(1);
}

// Type-safe usage — the compiler knows this never returns
fn load_config(path: &str) -> String {
    match std::fs::read_to_string(path) {
        Ok(content) => content,
        Err(e) => fatal_error(&format!("Cannot read {path}: {e}")),
        // ! coerces to String — the compiler accepts this
    }
}

fn main() {
    let config = load_config("config.toml");
    println!("Config loaded: {} bytes", config.len());
}
```

---

## 8. Practical Uses

### 1. Error handling shortcuts:

```rust
fn must_parse(s: &str) -> u32 {
    s.parse().unwrap_or_else(|_| panic!("'{s}' is not a u32"))
}
```

### 2. Infinite event loops:

```rust
async fn event_loop() -> ! {
    loop {
        // process events
        tokio::time::sleep(tokio::time::Duration::from_millis(100)).await;
    }
}
```

### 3. Placeholder during development:

```rust
fn complex_algorithm() -> Vec<i32> {
    todo!("implement sorting algorithm")  // ! coerces to Vec<i32>
}
```

### 4. Enum variants that can never be constructed:

```rust
enum MyResult<T> {
    Ok(T),
    Err(!),  // This variant can never exist (unstable feature)
}
// If stabilized, MyResult<T>::Err can never be constructed
// So unwrap() would be infallible!
```

---

## 9. Summary Cheat Sheet

```
THE NEVER TYPE (!)
────────────────────────────────────────────────────────────
!                      type that never produces a value
fn foo() -> !          function NEVER returns

DIVERGING EXPRESSIONS
────────────────────────────────────────────────────────────
panic!(msg)            always panics           → !
todo!(msg)             placeholder panic       → !
unimplemented!()       not implemented panic   → !
unreachable!()         should-not-reach panic  → !
loop { }               infinite loop           → !
std::process::exit(n)  process termination     → !
continue               skip to next iteration  → !
break                  exit loop               → !

KEY PROPERTY
────────────────────────────────────────────────────────────
! coerces to ANY type
This is why panic!() works in any context:
  let x: i32 = if cond { 42 } else { panic!() };

COMMON PATTERN
────────────────────────────────────────────────────────────
fn bail(msg: &str) -> ! {
    eprintln!("{msg}");
    std::process::exit(1);
}
```

---

## What's Next?

**Lesson 86 — Dynamically Sized Types (DST)** — Why `str` and `[T]` must live behind pointers, and how `Sized` bounds work.

## Further Reading
- [The Rust Reference — Never Type](https://doc.rust-lang.org/reference/types/never.html)
- [Rust Book — Ch 19.4: Advanced Types](https://doc.rust-lang.org/book/ch19-04-advanced-types.html)

---

*Never type: when functions have no way back! 🦀*
