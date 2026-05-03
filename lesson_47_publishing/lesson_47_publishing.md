# 📘 Lesson 47 — Publishing to crates.io (M5)

> **Series:** Rust From Zero · Intermediate Level  
> **Roadmap ID:** M5 · Category: 📦 Modules & Crates  
> **Previous:** [Lesson 46 — Workspaces](../lesson_46_workspaces/lesson_46_workspaces.md)  
> **Next:** [Lesson 48 — Closures: Syntax & Captures](../lesson_48_closures/lesson_48_closures.md)  
> **Practice:** [Questions](./lesson_47_questions.md) · [Answers](./lesson_47_answers.md)  
> **Practice Task:** Prepare a crate for publishing with docs, metadata, and examples

---

## Table of Contents

1. [What Is crates.io?](#1-what-is-cratesio)
2. [Preparing Your Crate](#2-preparing-your-crate)
3. [Cargo.toml Metadata](#3-cargotoml-metadata)
4. [Documentation with rustdoc](#4-documentation-with-rustdoc)
5. [Documentation Tests](#5-documentation-tests)
6. [Adding Examples](#6-adding-examples)
7. [Publishing Workflow](#7-publishing-workflow)
8. [Versioning (SemVer)](#8-versioning-semver)
9. [Yanking and Updating](#9-yanking-and-updating)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. What Is crates.io?

**crates.io** is Rust's official package registry — like npm for JavaScript or PyPI for Python. Anyone can publish and share Rust libraries.

- Browse: [https://crates.io](https://crates.io)
- Docs: [https://docs.rs](https://docs.rs) (auto-generated)
- Currently 140,000+ crates

---

## 2. Preparing Your Crate

### Crate structure:

```
my_crate/
├── Cargo.toml
├── README.md
├── LICENSE
├── src/
│   └── lib.rs
├── examples/
│   └── basic.rs
└── tests/
    └── integration.rs
```

### Public API design:

```rust
// src/lib.rs

//! # My Crate
//!
//! `my_crate` provides utilities for working with temperatures.
//!
//! ## Quick Start
//!
//! ```
//! use my_crate::Celsius;
//!
//! let temp = Celsius::new(100.0);
//! let fahrenheit = temp.to_fahrenheit();
//! println!("{fahrenheit}°F");
//! ```

/// A temperature in Celsius.
///
/// # Examples
///
/// ```
/// use my_crate::Celsius;
///
/// let boiling = Celsius::new(100.0);
/// assert_eq!(boiling.value(), 100.0);
/// ```
#[derive(Debug, Clone, Copy, PartialEq)]
pub struct Celsius(f64);

impl Celsius {
    /// Creates a new `Celsius` temperature.
    pub fn new(value: f64) -> Self {
        Celsius(value)
    }

    /// Returns the temperature value.
    pub fn value(&self) -> f64 {
        self.0
    }

    /// Converts to Fahrenheit.
    ///
    /// # Examples
    ///
    /// ```
    /// use my_crate::Celsius;
    ///
    /// let boiling = Celsius::new(100.0);
    /// assert!((boiling.to_fahrenheit() - 212.0).abs() < 0.01);
    /// ```
    pub fn to_fahrenheit(&self) -> f64 {
        self.0 * 9.0 / 5.0 + 32.0
    }
}

impl std::fmt::Display for Celsius {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        write!(f, "{:.1}°C", self.0)
    }
}
```

---

## 3. Cargo.toml Metadata

```toml
[package]
name = "my-temperature"
version = "0.1.0"
edition = "2021"
authors = ["Your Name <you@example.com>"]
description = "A simple temperature conversion library"
license = "MIT OR Apache-2.0"
repository = "https://github.com/you/my-temperature"
homepage = "https://github.com/you/my-temperature"
documentation = "https://docs.rs/my-temperature"
readme = "README.md"
keywords = ["temperature", "conversion", "celsius", "fahrenheit"]
categories = ["science"]
exclude = ["tests/", ".github/"]

[dependencies]
# your dependencies here
```

### Required fields for publishing:
- `name` — unique on crates.io
- `version` — semantic version
- `description` — short summary
- `license` or `license-file` — open source license

---

## 4. Documentation with rustdoc

### Three types of doc comments:

```rust
//! Module-level documentation (at the top of lib.rs)
//! Describes the entire crate.

/// Item-level documentation
/// Describes a function, struct, enum, etc.
///
/// # Examples
///
/// ```
/// let x = my_function(5);
/// assert_eq!(x, 10);
/// ```
pub fn my_function(x: i32) -> i32 {
    x * 2
}
```

### Common doc sections:

```rust
/// Brief description in one line.
///
/// Longer description with more detail
/// spanning multiple lines.
///
/// # Arguments
///
/// * `name` - The name to greet
/// * `times` - Number of times to greet
///
/// # Returns
///
/// A formatted greeting string.
///
/// # Examples
///
/// ```
/// let greeting = my_crate::greet("Alice", 2);
/// assert!(greeting.contains("Alice"));
/// ```
///
/// # Panics
///
/// Panics if `times` is 0.
///
/// # Errors
///
/// Returns `Err` if the name is empty.
pub fn greet(name: &str, times: usize) -> Result<String, String> {
    if name.is_empty() {
        return Err("Name cannot be empty".into());
    }
    if times == 0 {
        panic!("Times must be > 0");
    }
    Ok(format!("Hello, {name}! ").repeat(times))
}
```

### Building docs:

```bash
# Build and open docs in browser
cargo doc --open

# Build docs for your crate only (not dependencies)
cargo doc --no-deps --open
```

---

## 5. Documentation Tests

Code in `/// ``` blocks is automatically compiled and tested:

```bash
cargo test --doc
```

### Controlling doc tests:

```rust
/// This example shows error handling.
///
/// ```
/// # // Lines starting with # are hidden in docs but still compiled
/// # fn main() -> Result<(), Box<dyn std::error::Error>> {
/// let value: i32 = "42".parse()?;
/// assert_eq!(value, 42);
/// # Ok(())
/// # }
/// ```

/// This example should panic.
///
/// ```should_panic
/// panic!("oh no!");
/// ```

/// This example won't compile (and that's expected).
///
/// ```compile_fail
/// let x: i32 = "not a number";
/// ```

/// This example is for illustration only — don't run it.
///
/// ```no_run
/// loop { /* infinite */ }
/// ```
```

---

## 6. Adding Examples

```rust
// examples/basic.rs
use my_temperature::Celsius;

fn main() {
    let temps = vec![0.0, 20.0, 37.0, 100.0];

    for t in temps {
        let c = Celsius::new(t);
        println!("{c} = {:.1}°F", c.to_fahrenheit());
    }
}
```

```bash
# Run an example
cargo run --example basic
```

---

## 7. Publishing Workflow

```bash
# 1. Create account on crates.io (via GitHub)
# 2. Get API token from https://crates.io/settings/tokens

# 3. Login
cargo login <your-api-token>

# 4. Dry-run (check without publishing)
cargo publish --dry-run

# 5. Publish!
cargo publish
```

### Pre-publish checklist:
- [ ] `cargo test` passes
- [ ] `cargo clippy` has no warnings
- [ ] `cargo doc` builds without errors
- [ ] `README.md` exists and is helpful
- [ ] `LICENSE` file present
- [ ] Version is appropriate
- [ ] All public items are documented

---

## 8. Versioning (SemVer)

Rust follows **Semantic Versioning**: `MAJOR.MINOR.PATCH`

| Version Bump | When | Example |
|---|---|---|
| **PATCH** (0.1.0 → 0.1.1) | Bug fixes, no API changes | Fix a calculation error |
| **MINOR** (0.1.0 → 0.2.0) | New features, backwards compatible | Add a new function |
| **MAJOR** (0.1.0 → 1.0.0) | Breaking API changes | Rename a public function |

### Pre-1.0 special rules:
- `0.x.y` is treated as unstable
- `0.1.x → 0.2.0` can have breaking changes
- `0.1.0 → 0.1.1` should be compatible

---

## 9. Yanking and Updating

```bash
# Yank a version (prevent new projects from using it)
cargo yank --vers 0.1.0

# Un-yank
cargo yank --vers 0.1.0 --undo

# Publish a new version
# 1. Update version in Cargo.toml
# 2. cargo publish
```

Yanking doesn't delete — existing projects can still build. It only prevents new `Cargo.lock` entries.

---

## 10. Summary Cheat Sheet

```
PREPARING
────────────────────────────────────────────────────────────
README.md, LICENSE, good docs
All pub items documented with ///
Doc tests in examples blocks

CARGO.TOML REQUIRED FIELDS
────────────────────────────────────────────────────────────
name, version, description, license/license-file

DOCUMENTATION
────────────────────────────────────────────────────────────
//!             module/crate-level docs
///             item-level docs
# Examples      code examples (auto-tested!)
cargo doc       build docs
cargo test --doc run doc tests

PUBLISHING
────────────────────────────────────────────────────────────
cargo login TOKEN
cargo publish --dry-run   validate
cargo publish             publish to crates.io

SEMVER
────────────────────────────────────────────────────────────
MAJOR.MINOR.PATCH
PATCH  → bug fix        (compatible)
MINOR  → new feature    (compatible)
MAJOR  → breaking change
```

---

## What's Next?

**Lesson 48 — Closures: Syntax & Captures** — Enter the world of anonymous functions. Learn closure syntax, capture modes, and how closures interact with ownership.

## Further Reading
- [Cargo Book — Publishing](https://doc.rust-lang.org/cargo/reference/publishing.html)
- [The Rust Book — Ch 14: Publishing](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html)
- [docs.rs](https://docs.rs) — auto-generated documentation for all crates

---

*Sharing code with the world — one `cargo publish` at a time! 🦀*
