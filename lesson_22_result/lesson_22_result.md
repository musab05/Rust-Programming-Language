# 📘 Lesson 22 — Result\<T,E\> (E2)

> **Series:** Rust From Zero · Beginner Level  
> **Roadmap ID:** E2 · Category: ⚠ Error Handling  
> **Previous:** [Lesson 21 — panic! & unwrap](../lesson_21_panic_unwrap/lesson_21_panic_unwrap.md)  
> **Next:** [Lesson 23 — The ? Operator](../lesson_23_question_mark/lesson_23_question_mark.md)  
> **Practice:** [Questions](./lesson_22_questions.md) · [Answers](./lesson_22_answers.md)  
> **Practice Task:** File reader that returns `Result<String, io::Error>`

---

## Table of Contents

1. [What Is `Result<T, E>`?](#1-what-is-resultt-e)
2. [Creating Results](#2-creating-results)
3. [Handling Results with `match`](#3-handling-results-with-match)
4. [The `unwrap` and `expect` Shortcuts](#4-the-unwrap-and-expect-shortcuts)
5. [Combinators: `map`, `and_then`, `or_else`](#5-combinators-map-and_then-or_else)
6. [`unwrap_or` Family](#6-unwrap_or-family)
7. [`is_ok()` and `is_err()`](#7-is_ok-and-is_err)
8. [Functions That Return Result](#8-functions-that-return-result)
9. [Real-World Example: File I/O](#9-real-world-example-file-io)
10. [Converting Between `Option` and `Result`](#10-converting-between-option-and-result)
11. [Common Error Types in std](#11-common-error-types-in-std)
12. [Summary Cheat Sheet](#12-summary-cheat-sheet)

---

## 1. What Is `Result<T, E>`?

`Result<T, E>` is Rust's primary mechanism for **recoverable errors**. It's an enum defined in the standard library:

```rust
enum Result<T, E> {
    Ok(T),    // success — contains the value
    Err(E),   // failure — contains the error
}
```

- `T` = the type of the **success** value
- `E` = the type of the **error** value

```rust
fn main() {
    let success: Result<i32, String> = Ok(42);
    let failure: Result<i32, String> = Err(String::from("something went wrong"));

    println!("{:?}", success); // Ok(42)
    println!("{:?}", failure); // Err("something went wrong")
}
```

### Why not just use exceptions?

Rust has **no exceptions**. Instead:
- Errors that can be recovered from use `Result<T, E>`
- Errors that cannot be recovered from use `panic!`
- The compiler **forces** you to handle `Result` — you can't accidentally ignore errors

---

## 2. Creating Results

```rust
fn main() {
    // Explicit type annotation
    let a: Result<i32, &str> = Ok(42);
    let b: Result<i32, &str> = Err("failed");

    // Type inferred from usage
    let c = Ok::<i32, String>(100);
    let d = Err::<i32, String>(String::from("oops"));

    // From standard library functions
    let parsed: Result<i32, _> = "42".parse();         // Ok(42)
    let bad_parse: Result<i32, _> = "hello".parse();   // Err(ParseIntError)

    println!("{:?}", parsed);      // Ok(42)
    println!("{:?}", bad_parse);   // Err(ParseIntError { kind: InvalidDigit })
}
```

### Many std functions return `Result`:

```rust
use std::fs;
use std::net::TcpStream;

fn main() {
    // File operations → Result<String, io::Error>
    let content = fs::read_to_string("hello.txt");

    // Network → Result<TcpStream, io::Error>
    let stream = TcpStream::connect("127.0.0.1:8080");

    // Parsing → Result<T, ParseError>
    let num: Result<i32, _> = "42".parse();
}
```

---

## 3. Handling Results with `match`

The most explicit way to handle a `Result`:

```rust
use std::fs;

fn main() {
    let result = fs::read_to_string("hello.txt");

    match result {
        Ok(contents) => println!("File contents:\n{contents}"),
        Err(error) => println!("Failed to read file: {error}"),
    }
}
```

### Nested matching for different error kinds:

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let file = File::open("hello.txt");

    let file = match file {
        Ok(f) => f,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => {
                println!("File not found — creating it...");
                File::create("hello.txt").expect("Failed to create file")
            }
            ErrorKind::PermissionDenied => {
                panic!("Permission denied!");
            }
            other => {
                panic!("Unexpected error: {other:?}");
            }
        },
    };

    println!("File opened successfully: {file:?}");
}
```

---

## 4. The `unwrap` and `expect` Shortcuts

As covered in Lesson 21, these extract the `Ok` value or panic:

```rust
fn main() {
    // unwrap — panics with generic message on Err
    let val: i32 = "42".parse().unwrap();    // 42

    // expect — panics with YOUR message on Err
    let val: i32 = "42".parse()
        .expect("Failed to parse number");    // 42

    println!("{val}");
}
```

> Remember: prefer `expect` over `unwrap` in real code for better error messages.

---

## 5. Combinators: `map`, `and_then`, `or_else`

Combinators let you transform `Result` values without verbose `match` blocks.

### `map` — transform the `Ok` value

```rust
fn main() {
    let result: Result<i32, &str> = Ok(5);
    let doubled = result.map(|v| v * 2);
    println!("{:?}", doubled); // Ok(10)

    let err_result: Result<i32, &str> = Err("fail");
    let doubled = err_result.map(|v| v * 2);
    println!("{:?}", doubled); // Err("fail") — map skips Err
}
```

### `map_err` — transform the `Err` value

```rust
fn main() {
    let result: Result<i32, &str> = Err("bad input");
    let mapped = result.map_err(|e| format!("Error: {e}"));
    println!("{:?}", mapped); // Err("Error: bad input")
}
```

### `and_then` — chain operations that might fail

```rust
fn parse_and_double(s: &str) -> Result<i32, String> {
    s.parse::<i32>()
        .map_err(|e| e.to_string())
        .and_then(|n| {
            if n > 1000 {
                Err(String::from("Number too large"))
            } else {
                Ok(n * 2)
            }
        })
}

fn main() {
    println!("{:?}", parse_and_double("42"));    // Ok(84)
    println!("{:?}", parse_and_double("abc"));   // Err("invalid digit found in string")
    println!("{:?}", parse_and_double("9999"));  // Err("Number too large")
}
```

### `or_else` — provide a fallback on error

```rust
fn main() {
    let result: Result<i32, &str> = Err("primary failed");
    let fallback = result.or_else(|_| Ok::<i32, &str>(42));
    println!("{:?}", fallback); // Ok(42)
}
```

### Chaining combinators:

```rust
fn main() {
    let result: Result<i32, String> = "  42  "
        .trim()
        .parse::<i32>()
        .map_err(|e| e.to_string())
        .map(|n| n * 2)
        .map(|n| n + 10);

    println!("{:?}", result); // Ok(94)
}
```

---

## 6. `unwrap_or` Family

```rust
fn main() {
    let ok: Result<i32, &str> = Ok(42);
    let err: Result<i32, &str> = Err("fail");

    // unwrap_or — provide a default value
    println!("{}", ok.unwrap_or(0));    // 42
    println!("{}", err.unwrap_or(0));   // 0

    // unwrap_or_else — compute default lazily
    println!("{}", err.unwrap_or_else(|e| {
        println!("Error: {e}");
        -1
    }));
    // Prints: Error: fail
    // Then: -1

    // unwrap_or_default — use Default::default()
    let err2: Result<i32, &str> = Err("fail");
    println!("{}", err2.unwrap_or_default()); // 0 (default for i32)

    let err3: Result<String, &str> = Err("fail");
    println!("'{}'", err3.unwrap_or_default()); // '' (default for String is empty)
}
```

---

## 7. `is_ok()` and `is_err()`

Quick boolean checks without extracting the value:

```rust
fn main() {
    let ok: Result<i32, &str> = Ok(42);
    let err: Result<i32, &str> = Err("fail");

    println!("{}", ok.is_ok());   // true
    println!("{}", ok.is_err());  // false
    println!("{}", err.is_ok());  // false
    println!("{}", err.is_err()); // true

    // Useful in conditionals
    let result: Result<i32, _> = "42".parse();
    if result.is_ok() {
        println!("Parse succeeded!");
    }

    // Filter a collection
    let results = vec![Ok(1), Err("a"), Ok(3), Err("b"), Ok(5)];
    let successes: Vec<i32> = results.into_iter()
        .filter_map(|r| r.ok())  // .ok() converts Result → Option
        .collect();
    println!("{:?}", successes); // [1, 3, 5]
}
```

---

## 8. Functions That Return Result

Writing your own fallible functions:

```rust
#[derive(Debug)]
enum MathError {
    DivisionByZero,
    NegativeSquareRoot,
}

fn divide(a: f64, b: f64) -> Result<f64, MathError> {
    if b == 0.0 {
        Err(MathError::DivisionByZero)
    } else {
        Ok(a / b)
    }
}

fn sqrt(x: f64) -> Result<f64, MathError> {
    if x < 0.0 {
        Err(MathError::NegativeSquareRoot)
    } else {
        Ok(x.sqrt())
    }
}

fn main() {
    match divide(10.0, 3.0) {
        Ok(result) => println!("10 / 3 = {result:.4}"),
        Err(e) => println!("Error: {e:?}"),
    }

    match divide(10.0, 0.0) {
        Ok(result) => println!("10 / 0 = {result}"),
        Err(e) => println!("Error: {e:?}"),
    }

    match sqrt(-4.0) {
        Ok(result) => println!("sqrt(-4) = {result}"),
        Err(e) => println!("Error: {e:?}"),
    }
}
```

**Output:**
```
10 / 3 = 3.3333
Error: DivisionByZero
Error: NegativeSquareRoot
```

---

## 9. Real-World Example: File I/O

This is the roadmap practice task — a function that reads a file and returns `Result`:

```rust
use std::fs;
use std::io;

fn read_file(path: &str) -> Result<String, io::Error> {
    fs::read_to_string(path)
}

fn read_first_line(path: &str) -> Result<String, io::Error> {
    let contents = fs::read_to_string(path)?;
    let first_line = contents
        .lines()
        .next()
        .unwrap_or("")
        .to_string();
    Ok(first_line)
}

fn main() {
    // Try reading an existing file
    match read_file("Cargo.toml") {
        Ok(contents) => {
            let line_count = contents.lines().count();
            println!("Cargo.toml has {line_count} lines");
        }
        Err(e) => println!("Could not read Cargo.toml: {e}"),
    }

    // Try reading a non-existent file
    match read_file("does_not_exist.txt") {
        Ok(contents) => println!("Contents: {contents}"),
        Err(e) => println!("Error: {e}"),
    }

    // Read first line
    match read_first_line("Cargo.toml") {
        Ok(line) => println!("First line: {line}"),
        Err(e) => println!("Error: {e}"),
    }
}
```

---

## 10. Converting Between `Option` and `Result`

```rust
fn main() {
    // Option → Result
    let opt: Option<i32> = Some(42);
    let res: Result<i32, &str> = opt.ok_or("value was None");
    println!("{:?}", res); // Ok(42)

    let none: Option<i32> = None;
    let res: Result<i32, &str> = none.ok_or("value was None");
    println!("{:?}", res); // Err("value was None")

    // ok_or_else — lazily compute the error
    let none2: Option<i32> = None;
    let res = none2.ok_or_else(|| format!("Missing value at {}", "runtime"));
    println!("{:?}", res); // Err("Missing value at runtime")

    // Result → Option
    let ok: Result<i32, &str> = Ok(42);
    let opt: Option<i32> = ok.ok();       // Some(42)
    println!("{:?}", opt);

    let err: Result<i32, &str> = Err("fail");
    let opt: Option<i32> = err.ok();      // None (error is discarded)
    println!("{:?}", opt);

    // Get the error as Option
    let err: Result<i32, &str> = Err("fail");
    let opt: Option<&str> = err.err();    // Some("fail")
    println!("{:?}", opt);
}
```

---

## 11. Common Error Types in std

| Function / Module | Error Type | Example |
|---|---|---|
| `str::parse::<T>()` | `T::Err` (e.g., `ParseIntError`) | `"abc".parse::<i32>()` |
| `fs::read_to_string()` | `io::Error` | File not found |
| `File::open()` | `io::Error` | Permission denied |
| `TcpStream::connect()` | `io::Error` | Connection refused |
| `env::var()` | `env::VarError` | Variable not set |
| `String::from_utf8()` | `FromUtf8Error` | Invalid UTF-8 bytes |

All these error types implement `std::fmt::Display` and `std::fmt::Debug`, so you can always print them.

---

## 12. Summary Cheat Sheet

```
RESULT BASICS
────────────────────────────────────────────────────────────
Ok(value)                  success variant
Err(error)                 failure variant
Result<T, E>               T = success type, E = error type

EXTRACTING VALUES
────────────────────────────────────────────────────────────
match result { Ok(v) => ..., Err(e) => ... }    explicit
result.unwrap()            Ok→v, Err→PANIC
result.expect("msg")       Ok→v, Err→PANIC with msg
result.unwrap_or(default)  Ok→v, Err→default
result.unwrap_or_else(f)   Ok→v, Err→f(e)
result.unwrap_or_default() Ok→v, Err→Default::default()

COMBINATORS
────────────────────────────────────────────────────────────
result.map(|v| ...)        transform Ok value
result.map_err(|e| ...)    transform Err value
result.and_then(|v| ...)   chain fallible operations
result.or_else(|e| ...)    fallback on error

CHECKING
────────────────────────────────────────────────────────────
result.is_ok()             → bool
result.is_err()            → bool

CONVERTING
────────────────────────────────────────────────────────────
option.ok_or(err)          Option → Result
option.ok_or_else(|| err)  Option → Result (lazy)
result.ok()                Result → Option (discard error)
result.err()               Result → Option (discard success)

COMMON PATTERN
────────────────────────────────────────────────────────────
fn do_thing() -> Result<Value, Error> {
    let x = might_fail()?;   // ? = early return on Err
    let y = also_might_fail()?;
    Ok(compute(x, y))
}
```

---

## What's Next?

**Lesson 23 — The ? Operator** — The most ergonomic way to propagate errors. Learn how `?` replaces verbose `match` blocks and how `From` trait conversions make it work across error types.

## Further Reading
- [The Rust Book — Ch 9.2: Recoverable Errors with Result](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html)
- [std::result module](https://doc.rust-lang.org/std/result/index.html)

---

*Handle your errors gracefully — your future self will thank you! 🦀*
