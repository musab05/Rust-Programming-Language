# 📘 Lesson 36 — Custom Error Types (E4)

> **Series:** Rust From Zero · Intermediate Level  
> **Roadmap ID:** E4 · Category: ⚠ Error Handling  
> **Previous:** [Lesson 35 — Collecting & FromIterator](../lesson_35_collecting/lesson_35_collecting.md)  
> **Next:** [Lesson 37 — anyhow & error-stack](../lesson_37_anyhow/lesson_37_anyhow.md)  
> **Practice:** [Questions](./lesson_36_questions.md) · [Answers](./lesson_36_answers.md)  
> **Practice Task:** Build a CLI parser with a rich custom error enum

---

## Table of Contents

1. [Why Custom Errors?](#1-why-custom-errors)
2. [The Error Trait](#2-the-error-trait)
3. [Building a Custom Error Enum](#3-building-a-custom-error-enum)
4. [Implementing Display and Error](#4-implementing-display-and-error)
5. [From Conversions — Making ? Work](#5-from-conversions--making--work)
6. [Error Context and Wrapping](#6-error-context-and-wrapping)
7. [The thiserror Crate](#7-the-thiserror-crate)
8. [Real-World Example: CLI Parser](#8-real-world-example-cli-parser)
9. [Best Practices](#9-best-practices)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. Why Custom Errors?

Standard library errors (`io::Error`, `ParseIntError`) are great, but real applications have domain-specific failures:

```rust
// Instead of this vague approach:
fn parse_config(path: &str) -> Result<Config, Box<dyn std::error::Error>> {
    // What went wrong? File missing? Bad format? Missing field?
    todo!()
}

// Build descriptive errors:
enum ConfigError {
    FileNotFound(String),
    ParseError { line: usize, message: String },
    MissingField(String),
}
```

---

## 2. The Error Trait

The `std::error::Error` trait requires `Display` + `Debug`:

```rust
use std::fmt;

// The trait definition (simplified):
// pub trait Error: Display + Debug {
//     fn source(&self) -> Option<&(dyn Error + 'static)> { None }
// }
```

You need to implement:
1. `Debug` — usually via `#[derive(Debug)]`
2. `Display` — human-readable error message
3. `Error` — the trait itself (can be empty if no source error)

---

## 3. Building a Custom Error Enum

```rust
use std::fmt;
use std::num::ParseIntError;

#[derive(Debug)]
enum AppError {
    NotFound(String),
    ParseError(ParseIntError),
    InvalidInput { field: String, reason: String },
    Unauthorized,
}

impl fmt::Display for AppError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            AppError::NotFound(name) => write!(f, "not found: {name}"),
            AppError::ParseError(e) => write!(f, "parse error: {e}"),
            AppError::InvalidInput { field, reason } => {
                write!(f, "invalid input for '{field}': {reason}")
            }
            AppError::Unauthorized => write!(f, "unauthorized access"),
        }
    }
}

impl std::error::Error for AppError {
    fn source(&self) -> Option<&(dyn std::error::Error + 'static)> {
        match self {
            AppError::ParseError(e) => Some(e),
            _ => None,
        }
    }
}

fn main() {
    let err = AppError::InvalidInput {
        field: "age".into(),
        reason: "must be positive".into(),
    };
    println!("Error: {err}");
    println!("Debug: {err:?}");
}
```

---

## 4. Implementing Display and Error

### Display — the user-facing message:

```rust
use std::fmt;

#[derive(Debug)]
enum DbError {
    ConnectionFailed(String),
    QueryFailed { query: String, message: String },
    Timeout(u64),
}

impl fmt::Display for DbError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            DbError::ConnectionFailed(host) => {
                write!(f, "failed to connect to database at '{host}'")
            }
            DbError::QueryFailed { query, message } => {
                write!(f, "query failed: {message}\n  query: {query}")
            }
            DbError::Timeout(seconds) => {
                write!(f, "database operation timed out after {seconds}s")
            }
        }
    }
}

impl std::error::Error for DbError {}
```

---

## 5. From Conversions — Making ? Work

Implement `From` to enable the `?` operator for automatic error conversion:

```rust
use std::num::ParseIntError;
use std::io;
use std::fmt;

#[derive(Debug)]
enum AppError {
    Io(io::Error),
    Parse(ParseIntError),
    Custom(String),
}

impl fmt::Display for AppError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            AppError::Io(e) => write!(f, "I/O error: {e}"),
            AppError::Parse(e) => write!(f, "parse error: {e}"),
            AppError::Custom(msg) => write!(f, "{msg}"),
        }
    }
}

impl std::error::Error for AppError {}

// These From impls make ? work!
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

// Now ? converts automatically:
fn read_number(path: &str) -> Result<i32, AppError> {
    let content = std::fs::read_to_string(path)?;  // io::Error → AppError::Io
    let number = content.trim().parse::<i32>()?;     // ParseIntError → AppError::Parse
    Ok(number)
}

fn main() {
    match read_number("number.txt") {
        Ok(n) => println!("Got: {n}"),
        Err(e) => println!("Error: {e}"),
    }
}
```

---

## 6. Error Context and Wrapping

Add context to errors for better debugging:

```rust
use std::fmt;

#[derive(Debug)]
enum ConfigError {
    ReadError { path: String, source: std::io::Error },
    ParseError { path: String, line: usize, message: String },
    MissingField { path: String, field: String },
}

impl fmt::Display for ConfigError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            ConfigError::ReadError { path, source } => {
                write!(f, "failed to read config '{path}': {source}")
            }
            ConfigError::ParseError { path, line, message } => {
                write!(f, "parse error in '{path}' at line {line}: {message}")
            }
            ConfigError::MissingField { path, field } => {
                write!(f, "missing required field '{field}' in '{path}'")
            }
        }
    }
}

impl std::error::Error for ConfigError {
    fn source(&self) -> Option<&(dyn std::error::Error + 'static)> {
        match self {
            ConfigError::ReadError { source, .. } => Some(source),
            _ => None,
        }
    }
}

fn read_config(path: &str) -> Result<String, ConfigError> {
    std::fs::read_to_string(path).map_err(|e| ConfigError::ReadError {
        path: path.to_string(),
        source: e,
    })
}
```

---

## 7. The thiserror Crate

`thiserror` eliminates boilerplate with derive macros:

```rust
// Add to Cargo.toml:
// [dependencies]
// thiserror = "1"

use thiserror::Error;

#[derive(Debug, Error)]
enum AppError {
    #[error("file not found: {0}")]
    NotFound(String),

    #[error("I/O error: {0}")]
    Io(#[from] std::io::Error),       // auto-generates From impl!

    #[error("parse error: {0}")]
    Parse(#[from] std::num::ParseIntError),  // auto From!

    #[error("invalid input for '{field}': {reason}")]
    InvalidInput { field: String, reason: String },

    #[error("unauthorized")]
    Unauthorized,
}

// That's it! No manual Display, Error, or From implementations needed.

fn read_number(path: &str) -> Result<i32, AppError> {
    let content = std::fs::read_to_string(path)?;  // ? works via #[from]
    let number = content.trim().parse::<i32>()?;     // ? works via #[from]
    Ok(number)
}
```

### thiserror features:

| Attribute | Effect |
|---|---|
| `#[error("message")]` | Generates `Display` implementation |
| `#[from]` | Generates `From` implementation for `?` |
| `#[source]` | Links to the underlying error via `Error::source()` |
| `{0}`, `{field}` | Interpolates fields into the error message |

---

## 8. Real-World Example: CLI Parser

The roadmap practice task:

```rust
use std::fmt;
use std::num::ParseIntError;

#[derive(Debug)]
enum CliError {
    MissingArgument(String),
    InvalidFlag(String),
    ParseError { flag: String, source: ParseIntError },
    TooManyArguments { expected: usize, got: usize },
    HelpRequested,
}

impl fmt::Display for CliError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            CliError::MissingArgument(name) => {
                write!(f, "missing required argument: --{name}")
            }
            CliError::InvalidFlag(flag) => {
                write!(f, "unknown flag: {flag}")
            }
            CliError::ParseError { flag, source } => {
                write!(f, "invalid value for --{flag}: {source}")
            }
            CliError::TooManyArguments { expected, got } => {
                write!(f, "too many arguments: expected {expected}, got {got}")
            }
            CliError::HelpRequested => {
                write!(f, "Usage: app [--name NAME] [--count N] [--verbose]")
            }
        }
    }
}

impl std::error::Error for CliError {}

struct CliArgs {
    name: String,
    count: u32,
    verbose: bool,
}

fn parse_args(args: &[String]) -> Result<CliArgs, CliError> {
    let mut name = None;
    let mut count = 1u32;
    let mut verbose = false;
    let mut i = 0;

    while i < args.len() {
        match args[i].as_str() {
            "--help" | "-h" => return Err(CliError::HelpRequested),
            "--name" => {
                i += 1;
                name = Some(args.get(i)
                    .ok_or_else(|| CliError::MissingArgument("name".into()))?
                    .clone());
            }
            "--count" => {
                i += 1;
                let val = args.get(i)
                    .ok_or_else(|| CliError::MissingArgument("count".into()))?;
                count = val.parse().map_err(|e| CliError::ParseError {
                    flag: "count".into(),
                    source: e,
                })?;
            }
            "--verbose" | "-v" => verbose = true,
            other => return Err(CliError::InvalidFlag(other.into())),
        }
        i += 1;
    }

    Ok(CliArgs {
        name: name.ok_or_else(|| CliError::MissingArgument("name".into()))?,
        count,
        verbose,
    })
}

fn main() {
    // Simulate different argument scenarios
    let test_cases = vec![
        vec!["--name", "Alice", "--count", "3", "--verbose"],
        vec!["--name", "Bob"],
        vec!["--count", "5"],         // missing name
        vec!["--name", "Eve", "--count", "abc"],  // bad count
        vec!["--unknown"],            // invalid flag
        vec!["--help"],               // help
    ];

    for args in test_cases {
        let args: Vec<String> = args.into_iter().map(String::from).collect();
        print!("Args {:?} → ", args);
        match parse_args(&args) {
            Ok(cli) => println!("OK: name={}, count={}, verbose={}",
                                cli.name, cli.count, cli.verbose),
            Err(e) => println!("Error: {e}"),
        }
    }
}
```

---

## 9. Best Practices

1. **Use enums, not strings** — Each variant describes a specific failure mode
2. **Include context** — File paths, line numbers, field names in error variants
3. **Implement `From` for `?`** — Let the error propagation be automatic
4. **Use `thiserror` for libraries** — Reduces boilerplate, keeps errors typed
5. **Use `source()` for chaining** — Link to underlying errors for debugging
6. **Keep error messages user-friendly** — Display is for humans, Debug is for developers

---

## 10. Summary Cheat Sheet

```
MANUAL CUSTOM ERRORS
────────────────────────────────────────────────────────────
1. #[derive(Debug)] on your error enum
2. impl Display — human-readable messages
3. impl Error   — can be empty or implement source()
4. impl From<OtherError> — for each error ? should convert

thiserror CRATE
────────────────────────────────────────────────────────────
#[derive(Debug, Error)]
enum MyError {
    #[error("message {0}")]     Display auto-generated
    Variant(Type),

    #[error("msg: {0}")]
    WithFrom(#[from] OtherErr), From auto-generated

    #[error("ctx")]
    WithSource {
        #[source] inner: Err,   source() auto-generated
        context: String,
    },
}

DESIGN PATTERNS
────────────────────────────────────────────────────────────
One error enum per module/crate
Variants for each failure mode
Include context (paths, names, line numbers)
impl From for automatic ? conversion
```

---

## What's Next?

**Lesson 37 — anyhow & error-stack** — Dynamic error handling for applications. Learn `anyhow::Result`, `.context()`, and when to use `anyhow` vs `thiserror`.

## Further Reading
- [The Rust Book — Error Handling](https://doc.rust-lang.org/book/ch09-00-error-handling.html)
- [thiserror crate](https://docs.rs/thiserror)
- [Error Handling in Rust (blog)](https://blog.burntsushi.net/rust-error-handling/)

---

*Custom errors: making failures as descriptive as your successes! 🦀*
