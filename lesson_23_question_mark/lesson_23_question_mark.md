# 📘 Lesson 23 — The ? Operator (E3)

> **Series:** Rust From Zero · Beginner Level  
> **Roadmap ID:** E3 · Category: ⚠ Error Handling  
> **Previous:** [Lesson 22 — Result\<T,E\>](../lesson_22_result/lesson_22_result.md)  
> **Next:** [Lesson 24 — Modules & mod](../lesson_24_modules/lesson_24_modules.md)  
> **Practice:** [Questions](./lesson_23_questions.md) · [Answers](./lesson_23_answers.md)  
> **Practice Task:** Chain 3 fallible operations using only `?`

---

## Table of Contents

1. [The Problem: Verbose Error Handling](#1-the-problem-verbose-error-handling)
2. [Enter the `?` Operator](#2-enter-the--operator)
3. [How `?` Works Under the Hood](#3-how--works-under-the-hood)
4. [Chaining `?` for Clean Pipelines](#4-chaining--for-clean-pipelines)
5. [`?` with `Option<T>`](#5--with-optiont)
6. [The `From` Trait and Error Conversion](#6-the-from-trait-and-error-conversion)
7. [Using `?` in `main()`](#7-using--in-main)
8. [Real-World Example: Config File Reader](#8-real-world-example-config-file-reader)
9. [Common Mistakes with `?`](#9-common-mistakes-with-)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. The Problem: Verbose Error Handling

Without `?`, chaining multiple fallible operations leads to deeply nested `match` blocks:

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_verbose() -> Result<String, io::Error> {
    let file = File::open("username.txt");

    let mut file = match file {
        Ok(f) => f,
        Err(e) => return Err(e),    // early return on error
    };

    let mut contents = String::new();

    match file.read_to_string(&mut contents) {
        Ok(_) => {}
        Err(e) => return Err(e),    // early return on error
    }

    Ok(contents.trim().to_string())
}
```

This works, but it's **verbose** and **repetitive**. Every `match` follows the same pattern: unwrap `Ok` or return `Err`.

---

## 2. Enter the `?` Operator

The `?` operator replaces that entire pattern. Place it after any `Result` expression:

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username() -> Result<String, io::Error> {
    let mut file = File::open("username.txt")?;    // ? = return Err if Err
    let mut contents = String::new();
    file.read_to_string(&mut contents)?;            // ? again
    Ok(contents.trim().to_string())
}
```

**Same logic, 60% less code.**

### Even more concise with chaining:

```rust
use std::fs;
use std::io;

fn read_username() -> Result<String, io::Error> {
    let contents = fs::read_to_string("username.txt")?;
    Ok(contents.trim().to_string())
}
```

---

## 3. How `?` Works Under the Hood

The `?` operator on a `Result<T, E>` is equivalent to:

```rust
// expression?
// is equivalent to:
match expression {
    Ok(value) => value,           // unwrap and continue
    Err(e) => return Err(e.into()),  // convert error and return early
}
```

Key points:
1. On `Ok(v)` → extracts the value `v` and execution continues
2. On `Err(e)` → calls `.into()` on the error (for conversion), wraps it in `Err()`, and **returns early** from the function
3. **Can only be used in functions that return `Result` or `Option`**

---

## 4. Chaining `?` for Clean Pipelines

The roadmap practice task: chain 3 fallible operations using only `?`:

```rust
use std::num::ParseIntError;

fn parse_and_validate(input: &str) -> Result<i32, String> {
    // Operation 1: Trim and parse
    let number: i32 = input.trim()
        .parse()
        .map_err(|e: ParseIntError| format!("Parse error: {e}"))?;

    // Operation 2: Check range
    if number < 0 || number > 1000 {
        return Err(format!("{number} out of range (0-1000)"));
    }

    // Operation 3: Apply business logic
    let result = if number % 2 == 0 {
        Ok(number / 2)
    } else {
        Err(format!("{number} must be even"))
    }?;

    Ok(result)
}

fn main() {
    let tests = ["  42  ", "hello", "2000", "7", "100"];
    for t in tests {
        println!("'{t}' → {:?}", parse_and_validate(t));
    }
}
```

**Output:**
```
'  42  ' → Ok(21)
'hello' → Err("Parse error: invalid digit found in string")
'2000' → Err("2000 out of range (0-1000)")
'7' → Err("7 must be even")
'100' → Ok(50)
```

### A file processing pipeline:

```rust
use std::fs;
use std::io;

fn count_words_in_file(path: &str) -> Result<usize, io::Error> {
    let contents = fs::read_to_string(path)?;       // might fail: file not found
    let count = contents.split_whitespace().count();
    Ok(count)
}

fn get_longest_line(path: &str) -> Result<String, io::Error> {
    let contents = fs::read_to_string(path)?;
    let longest = contents
        .lines()
        .max_by_key(|line| line.len())
        .unwrap_or("")
        .to_string();
    Ok(longest)
}

fn main() {
    match count_words_in_file("Cargo.toml") {
        Ok(count) => println!("Word count: {count}"),
        Err(e) => println!("Error: {e}"),
    }

    match get_longest_line("Cargo.toml") {
        Ok(line) => println!("Longest line: '{line}'"),
        Err(e) => println!("Error: {e}"),
    }
}
```

---

## 5. `?` with `Option<T>`

The `?` operator also works with `Option`:

```rust
fn first_even(numbers: &[i32]) -> Option<i32> {
    let first = numbers.first()?;      // None → return None
    if first % 2 == 0 {
        Some(*first)
    } else {
        None
    }
}

fn get_middle_char(s: &str) -> Option<char> {
    let len = s.len();
    if len == 0 { return None; }
    let mid = len / 2;
    s.chars().nth(mid)                  // already returns Option
}

fn main() {
    println!("{:?}", first_even(&[4, 1, 6]));  // Some(4)
    println!("{:?}", first_even(&[3, 1, 6]));  // None
    println!("{:?}", first_even(&[]));          // None

    println!("{:?}", get_middle_char("hello")); // Some('l')
    println!("{:?}", get_middle_char(""));      // None
}
```

> ⚠️ You **cannot** mix `?` on `Result` and `?` on `Option` in the same function. The function must return either `Result` or `Option`.

---

## 6. The `From` Trait and Error Conversion

The `?` operator calls `.into()` on errors, which uses the `From` trait. This allows automatic error conversion:

```rust
use std::fs;
use std::io;
use std::num::ParseIntError;

#[derive(Debug)]
enum AppError {
    Io(io::Error),
    Parse(ParseIntError),
}

// Implement From for automatic conversion
impl From<io::Error> for AppError {
    fn from(e: io::Error) -> Self {
        AppError::Io(e)
    }
}

impl From<ParseIntError> for AppError {
    fn from(e: ParseIntError) -> Self {
        AppError::Parse(e)
    }
}

fn read_number_from_file(path: &str) -> Result<i32, AppError> {
    let contents = fs::read_to_string(path)?;  // io::Error → AppError::Io
    let number = contents.trim().parse()?;       // ParseIntError → AppError::Parse
    Ok(number)
}

fn main() {
    match read_number_from_file("number.txt") {
        Ok(n) => println!("Number: {n}"),
        Err(AppError::Io(e)) => println!("IO error: {e}"),
        Err(AppError::Parse(e)) => println!("Parse error: {e}"),
    }
}
```

The `?` automatically converts `io::Error` and `ParseIntError` into `AppError` because we implemented `From`.

### Using `Box<dyn Error>` for quick prototyping:

```rust
use std::error::Error;
use std::fs;

fn do_stuff() -> Result<i32, Box<dyn Error>> {
    let contents = fs::read_to_string("number.txt")?;   // any error type works
    let number: i32 = contents.trim().parse()?;           // any error type works
    Ok(number * 2)
}

fn main() {
    match do_stuff() {
        Ok(n) => println!("Result: {n}"),
        Err(e) => println!("Error: {e}"),
    }
}
```

`Box<dyn Error>` accepts any error type — great for quick scripts, but you lose specific error matching.

---

## 7. Using `?` in `main()`

By default `main()` returns `()`, so you can't use `?`. But you can change `main()` to return `Result`:

```rust
use std::fs;
use std::error::Error;

fn main() -> Result<(), Box<dyn Error>> {
    let contents = fs::read_to_string("Cargo.toml")?;
    let line_count = contents.lines().count();
    println!("Cargo.toml has {line_count} lines");
    Ok(())  // must return Ok(()) on success
}
```

If `main()` returns `Err`, Rust prints the error and exits with a non-zero code:
```
Error: No such file or directory (os error 2)
```

---

## 8. Real-World Example: Config File Reader

```rust
use std::collections::HashMap;
use std::fs;
use std::error::Error;

/// Reads a simple key=value config file
fn read_config(path: &str) -> Result<HashMap<String, String>, Box<dyn Error>> {
    let contents = fs::read_to_string(path)?;

    let mut config = HashMap::new();

    for (line_num, line) in contents.lines().enumerate() {
        let line = line.trim();

        // Skip empty lines and comments
        if line.is_empty() || line.starts_with('#') {
            continue;
        }

        // Split on first '='
        let (key, value) = line.split_once('=')
            .ok_or_else(|| format!("Line {}: missing '=' in '{line}'", line_num + 1))?;

        config.insert(
            key.trim().to_string(),
            value.trim().to_string(),
        );
    }

    Ok(config)
}

fn main() -> Result<(), Box<dyn Error>> {
    let config = read_config("app.conf")?;

    for (key, value) in &config {
        println!("{key} = {value}");
    }

    Ok(())
}
```

Notice how `?` is used on:
1. `fs::read_to_string()` → `io::Error` converted via `Box<dyn Error>`
2. `.ok_or_else()` → `String` converted via `Box<dyn Error>`

---

## 9. Common Mistakes with `?`

### Mistake 1: Using `?` in a function that returns `()`

```rust
// ❌ Won't compile — main returns ()
fn main() {
    let contents = std::fs::read_to_string("file.txt")?;
}
```

Fix: change the return type:
```rust
fn main() -> Result<(), std::io::Error> {
    let contents = std::fs::read_to_string("file.txt")?;
    Ok(())
}
```

### Mistake 2: Mixing `Option` and `Result` with `?`

```rust
// ❌ Won't compile — function returns Result but ? is on Option
fn first_line(path: &str) -> Result<String, std::io::Error> {
    let contents = std::fs::read_to_string(path)?;
    let line = contents.lines().next()?;  // ❌ Option, not Result!
    Ok(line.to_string())
}
```

Fix: convert `Option` to `Result`:
```rust
fn first_line(path: &str) -> Result<String, std::io::Error> {
    let contents = std::fs::read_to_string(path)?;
    let line = contents.lines().next()
        .ok_or_else(|| std::io::Error::new(
            std::io::ErrorKind::InvalidData,
            "File is empty"
        ))?;
    Ok(line.to_string())
}
```

### Mistake 3: Forgetting that `?` returns early

```rust
fn process() -> Result<i32, String> {
    let value = Err("fail")?;
    // This line never runs because ? returns early
    println!("This won't print");
    Ok(value)
}
```

---

## 10. Summary Cheat Sheet

```
THE ? OPERATOR
────────────────────────────────────────────────────────────
expression?
  On Ok(v)  → extracts v, continues
  On Err(e) → return Err(e.into())
  On Some(v)→ extracts v (if function returns Option)
  On None   → return None (if function returns Option)

REQUIREMENTS
────────────────────────────────────────────────────────────
Function must return Result<T, E> or Option<T>
Can't mix ? on Result and ? on Option in the same fn
Error type must implement From<SourceError> (or be the same)

ERROR CONVERSION
────────────────────────────────────────────────────────────
impl From<io::Error> for MyError { ... }
  → allows ? to auto-convert io::Error to MyError

Box<dyn Error>
  → accepts ANY error type (quick prototyping)

PATTERNS
────────────────────────────────────────────────────────────
let val = might_fail()?;              single ?
let val = a()?.method()?.field;       chained
let val = opt.ok_or("err msg")?;      Option → Result → ?

fn main() → Result
────────────────────────────────────────────────────────────
fn main() -> Result<(), Box<dyn Error>> {
    let x = fallible_op()?;
    Ok(())
}

VERSUS
────────────────────────────────────────────────────────────
match expr { Ok(v)=>v, Err(e)=>return Err(e) }  verbose
expr?                                             concise (same thing)
expr.unwrap()                                     panics on error
```

---

## What's Next?

**Lesson 24 — Modules & mod** — Organise your code into modules, control visibility with `pub`, and navigate with `use`, `super`, and `crate::`.

## Further Reading
- [The Rust Book — Ch 9.2: Propagating Errors](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html#a-shortcut-for-propagating-errors-the--operator)
- [The `?` operator in std](https://doc.rust-lang.org/std/ops/trait.Try.html)

---

*Let `?` do the heavy lifting — your code will thank you! 🦀*
