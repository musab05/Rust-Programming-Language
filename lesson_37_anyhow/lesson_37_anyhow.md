# 📘 Lesson 37 — anyhow & error-stack (E5)

> **Series:** Rust From Zero · Intermediate Level  
> **Roadmap ID:** E5 · Category: ⚠ Error Handling  
> **Previous:** [Lesson 36 — Custom Error Types](../lesson_36_custom_errors/lesson_36_custom_errors.md)  
> **Next:** [Lesson 38 — Traits: Definition & Implementation](../lesson_38_traits/lesson_38_traits.md)  
> **Practice:** [Questions](./lesson_37_questions.md) · [Answers](./lesson_37_answers.md)  
> **Practice Task:** Rewrite a multi-error app using anyhow

---

## Table of Contents

1. [The Problem with Box<dyn Error>](#1-the-problem-with-boxdyn-error)
2. [anyhow — Dynamic Error Handling](#2-anyhow--dynamic-error-handling)
3. [anyhow::Context — Adding Context](#3-anhowcontext--adding-context)
4. [anyhow::bail! and ensure!](#4-anyhowbail-and-ensure)
5. [Downcasting anyhow Errors](#5-downcasting-anyhow-errors)
6. [anyhow vs thiserror — When to Use Each](#6-anyhow-vs-thiserror--when-to-use-each)
7. [error-stack Overview](#7-error-stack-overview)
8. [Real-World Example](#8-real-world-example)
9. [Summary Cheat Sheet](#9-summary-cheat-sheet)

---

## 1. The Problem with Box<dyn Error>

```rust
use std::error::Error;

// Works but is verbose and loses type information
fn do_stuff() -> Result<(), Box<dyn Error>> {
    let content = std::fs::read_to_string("config.txt")?;
    let number: i32 = content.trim().parse()?;
    println!("Got: {number}");
    Ok(())
}
```

Problems:
- Verbose return type
- No built-in context ("which file failed?")
- Backtrace not included by default

---

## 2. anyhow — Dynamic Error Handling

`anyhow` provides `anyhow::Result<T>` as a drop-in replacement:

```rust
// Cargo.toml: anyhow = "1"
use anyhow::Result;

fn do_stuff() -> Result<()> {
    let content = std::fs::read_to_string("config.txt")?;
    let number: i32 = content.trim().parse()?;
    println!("Got: {number}");
    Ok(())
}

fn main() {
    if let Err(e) = do_stuff() {
        // anyhow prints the full error chain
        println!("Error: {e}");
        println!("\nFull chain:");
        for cause in e.chain() {
            println!("  Caused by: {cause}");
        }
    }
}
```

### Creating ad-hoc errors:

```rust
use anyhow::{anyhow, Result};

fn validate_age(age: i32) -> Result<()> {
    if age < 0 {
        return Err(anyhow!("age cannot be negative: {age}"));
    }
    if age > 150 {
        return Err(anyhow!("age is unrealistic: {age}"));
    }
    Ok(())
}
```

---

## 3. anyhow::Context — Adding Context

The killer feature — add human-readable context to any error:

```rust
use anyhow::{Context, Result};

fn read_config(path: &str) -> Result<String> {
    std::fs::read_to_string(path)
        .with_context(|| format!("failed to read config file '{path}'"))
}

fn parse_port(config: &str) -> Result<u16> {
    config.trim().parse::<u16>()
        .with_context(|| format!("failed to parse port from '{config}'"))
}

fn setup_server() -> Result<()> {
    let config = read_config("server.conf")
        .context("setting up server")?;
    let port = parse_port(&config)
        .context("configuring port")?;
    println!("Server on port {port}");
    Ok(())
}

fn main() {
    if let Err(e) = setup_server() {
        // Prints the full chain:
        // Error: setting up server
        //
        // Caused by:
        //     0: failed to read config file 'server.conf'
        //     1: No such file or directory
        println!("Error: {e:?}");
    }
}
```

### `.context()` vs `.with_context()`:

```rust
// .context("static string") — evaluated always
result.context("failed to read file")?;

// .with_context(|| format!(...)) — evaluated only on error (lazy)
result.with_context(|| format!("failed to read '{path}'"))?;
```

---

## 4. anyhow::bail! and ensure!

Shortcuts for returning errors:

```rust
use anyhow::{bail, ensure, Result};

fn validate_username(name: &str) -> Result<()> {
    // bail! — return Err immediately
    if name.is_empty() {
        bail!("username cannot be empty");
    }

    // ensure! — like assert! but returns Err instead of panicking
    ensure!(name.len() >= 3, "username must be at least 3 characters, got {}", name.len());
    ensure!(name.len() <= 20, "username too long: {} characters", name.len());
    ensure!(
        name.chars().all(|c| c.is_alphanumeric() || c == '_'),
        "username contains invalid characters: '{name}'"
    );

    Ok(())
}

fn main() {
    let tests = vec!["", "ab", "alice", "valid_user_123", "bad char!", "a".repeat(25).as_str()];
    // Note: we'll use owned strings for the long one
    for name in &["", "ab", "alice", "valid_user_123", "bad char!"] {
        match validate_username(name) {
            Ok(()) => println!("  ✅ '{name}' is valid"),
            Err(e) => println!("  ❌ '{name}': {e}"),
        }
    }
}
```

---

## 5. Downcasting anyhow Errors

You can recover typed errors from `anyhow::Error`:

```rust
use anyhow::{anyhow, Result};

#[derive(Debug, thiserror::Error)]
enum AppError {
    #[error("rate limited")]
    RateLimited,
    #[error("not found: {0}")]
    NotFound(String),
}

fn do_request() -> Result<()> {
    Err(AppError::RateLimited.into())
}

fn main() {
    if let Err(e) = do_request() {
        // Downcast to check specific error type
        if let Some(app_err) = e.downcast_ref::<AppError>() {
            match app_err {
                AppError::RateLimited => println!("Retrying after delay..."),
                AppError::NotFound(name) => println!("Resource '{name}' missing"),
            }
        } else {
            println!("Unknown error: {e}");
        }
    }
}
```

---

## 6. anyhow vs thiserror — When to Use Each

| | **thiserror** | **anyhow** |
|---|---|---|
| **Use in** | Libraries | Applications |
| **Error type** | Concrete enum | Dynamic `anyhow::Error` |
| **Pattern matching** | ✅ Full match | Requires downcasting |
| **Context** | Manual | `.context()` built-in |
| **Boilerplate** | Medium (derive macro) | Minimal |
| **Backtraces** | Manual | Automatic |

### The golden rule:

> **Libraries** → `thiserror` (callers need typed errors)  
> **Applications** → `anyhow` (you just want to report & exit)  
> **Both** → `thiserror` for the error types, `anyhow` in `main()`

```rust
// In your library crate:
#[derive(Debug, thiserror::Error)]
pub enum DbError {
    #[error("connection failed: {0}")]
    Connection(String),
    #[error("query failed: {0}")]
    Query(String),
}

// In your application's main:
use anyhow::{Context, Result};

fn main() -> Result<()> {
    let db = connect_db()
        .context("initializing database")?;
    Ok(())
}
```

---

## 7. error-stack Overview

`error-stack` (by Hash) provides richer error reporting:

```rust
// Cargo.toml: error-stack = "0.4"

// error-stack provides:
// - Automatic backtraces
// - Attachments (extra context as arbitrary types)
// - Printable reports with full chain
// - Type-safe context changes

// Basic idea:
// use error_stack::{Report, ResultExt};
//
// fn read_file(path: &str) -> Result<String, Report<io::Error>> {
//     std::fs::read_to_string(path)
//         .map_err(Report::from)
//         .attach_printable(format!("reading config from {path}"))
// }
```

### When to use error-stack:
- You need rich diagnostic reports
- You want typed context attachments
- You're building a complex application with deep error chains

For most projects, **anyhow is sufficient**.

---

## 8. Real-World Example

Roadmap practice task — rewriting with anyhow:

```rust
use anyhow::{bail, ensure, Context, Result};
use std::collections::HashMap;

fn read_config(path: &str) -> Result<String> {
    std::fs::read_to_string(path)
        .with_context(|| format!("failed to read config from '{path}'"))
}

fn parse_config(content: &str) -> Result<HashMap<String, String>> {
    let mut config = HashMap::new();

    for (i, line) in content.lines().enumerate() {
        let line = line.trim();
        if line.is_empty() || line.starts_with('#') {
            continue;
        }

        let parts: Vec<&str> = line.splitn(2, '=').collect();
        ensure!(
            parts.len() == 2,
            "invalid config at line {}: expected key=value, got '{line}'",
            i + 1
        );

        let key = parts[0].trim();
        let value = parts[1].trim();
        ensure!(!key.is_empty(), "empty key at line {}", i + 1);

        config.insert(key.to_string(), value.to_string());
    }

    Ok(config)
}

fn get_port(config: &HashMap<String, String>) -> Result<u16> {
    let port_str = config.get("port")
        .context("missing 'port' in configuration")?;
    port_str.parse::<u16>()
        .with_context(|| format!("invalid port value: '{port_str}'"))
}

fn run_app() -> Result<()> {
    let content = read_config("app.conf")?;
    let config = parse_config(&content)
        .context("parsing configuration")?;
    let port = get_port(&config)
        .context("reading server port")?;
    println!("Starting server on port {port}");
    Ok(())
}

fn main() {
    if let Err(e) = run_app() {
        eprintln!("Application error: {e:?}");
        std::process::exit(1);
    }
}
```

---

## 9. Summary Cheat Sheet

```
anyhow BASICS
────────────────────────────────────────────────────────────
use anyhow::Result;            anyhow::Result<T> = Result<T, anyhow::Error>
anyhow!("message")             create ad-hoc error
bail!("message")               return Err(anyhow!(...))
ensure!(cond, "msg")           return Err if condition false

CONTEXT
────────────────────────────────────────────────────────────
.context("msg")?               add context (eager)
.with_context(|| msg)?         add context (lazy)
e.chain()                      iterate error chain

DOWNCASTING
────────────────────────────────────────────────────────────
e.downcast_ref::<MyError>()    try to get concrete error type
e.downcast::<MyError>()        consume and convert

WHEN TO USE WHAT
────────────────────────────────────────────────────────────
Library code  → thiserror (typed errors for callers)
App code      → anyhow (easy context, backtraces)
Complex apps  → error-stack (rich diagnostics)
Learning      → manual Error impl (understand the traits)
```

---

## What's Next?

**Lesson 38 — Traits: Definition & Implementation** — Enter the world of Rust's most powerful abstraction. Define behavior that types can share, and build flexible, reusable code.

## Further Reading
- [anyhow crate](https://docs.rs/anyhow)
- [thiserror crate](https://docs.rs/thiserror)
- [error-stack crate](https://docs.rs/error-stack)
- [Rust Error Handling Best Practices](https://www.lpalmieri.com/posts/error-handling-rust/)

---

*anyhow: because sometimes you just want to say "something went wrong"! 🦀*
