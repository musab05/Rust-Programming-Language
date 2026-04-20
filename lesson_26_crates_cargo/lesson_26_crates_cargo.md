# 📘 Lesson 26 — Crates & Cargo (M3)

> **Series:** Rust From Zero · Beginner Level  
> **Roadmap ID:** M3 · Category: 📁 Modules  
> **Previous:** [Lesson 25 — Paths & use](../lesson_25_paths_use/lesson_25_paths_use.md)  
> **Next:** [Lesson 27 — Testing](../lesson_27_testing/lesson_27_testing.md)  
> **Practice:** [Questions](./lesson_26_questions.md) · [Answers](./lesson_26_answers.md)  
> **Practice Task:** Add `serde` + `serde_json`; serialize a struct to JSON

---

## Table of Contents

1. [What Is a Crate?](#1-what-is-a-crate)
2. [Binary vs Library Crates](#2-binary-vs-library-crates)
3. [Packages](#3-packages)
4. [Cargo.toml — The Manifest](#4-cargotoml--the-manifest)
5. [Adding Dependencies](#5-adding-dependencies)
6. [Cargo.lock](#6-cargolock)
7. [Creating a Library Crate](#7-creating-a-library-crate)
8. [The Crate Root](#8-the-crate-root)
9. [Essential Cargo Commands](#9-essential-cargo-commands)
10. [Features](#10-features)
11. [Publishing to crates.io](#11-publishing-to-cratesio)
12. [Summary Cheat Sheet](#12-summary-cheat-sheet)

---

## 1. What Is a Crate?

A **crate** is the smallest unit of compilation in Rust. It can be:
- A **binary** — an executable program
- A **library** — reusable code that other crates depend on

```
Crate
├── The unit of compilation
├── Has a "crate root" file (entry point)
└── Produces one binary OR one library
```

Every Rust program you've written so far has been a binary crate with `main.rs` as the crate root.

---

## 2. Binary vs Library Crates

| Feature | Binary Crate | Library Crate |
|---|---|---|
| Entry point | `src/main.rs` | `src/lib.rs` |
| Has `fn main()`? | ✅ Yes | ❌ No |
| Produces | An executable | A `.rlib` file |
| Created with | `cargo new myapp` | `cargo new mylib --lib` |
| Can be run? | ✅ `cargo run` | ❌ Not directly |
| Purpose | Programs | Reusable libraries |

### A package can have both:

```
my_project/
├── Cargo.toml
└── src/
    ├── main.rs      ← binary crate root
    └── lib.rs       ← library crate root
```

The binary crate can use the library crate:

```rust
// src/lib.rs
pub fn greet(name: &str) -> String {
    format!("Hello, {name}!")
}
```

```rust
// src/main.rs
use my_project::greet;  // use the library crate

fn main() {
    println!("{}", greet("Rustacean"));
}
```

---

## 3. Packages

A **package** is what Cargo creates and manages. It contains:
- A `Cargo.toml` file (the manifest)
- One or more crates

**Rules:**
- A package can contain **at most one** library crate
- A package can contain **any number** of binary crates
- A package must contain **at least one** crate

### Multiple binaries:

```
my_project/
├── Cargo.toml
└── src/
    ├── main.rs           ← default binary
    └── bin/
        ├── tool1.rs      ← additional binary
        └── tool2.rs      ← additional binary
```

```bash
cargo run                    # runs src/main.rs
cargo run --bin tool1        # runs src/bin/tool1.rs
cargo run --bin tool2        # runs src/bin/tool2.rs
```

---

## 4. Cargo.toml — The Manifest

Every Cargo project has a `Cargo.toml` at the root:

```toml
[package]
name = "my_project"        # crate name (used in use statements)
version = "0.1.0"          # semantic versioning
edition = "2021"           # Rust edition (2015, 2018, 2021, 2024)
authors = ["Your Name <you@example.com>"]
description = "A cool Rust project"
license = "MIT"

[dependencies]
serde = "1.0"              # external dependencies go here
rand = "0.8"

[dev-dependencies]
criterion = "0.5"          # only for tests and benchmarks

[build-dependencies]
cc = "1.0"                 # only for build scripts
```

### Semantic Versioning (SemVer):

```
major.minor.patch
  1   .  2  .  3
```

| Change type | Example | When |
|---|---|---|
| Patch (1.0.**1**) | Bug fixes | No API changes |
| Minor (1.**1**.0) | New features | Backwards compatible |
| Major (**2**.0.0) | Breaking changes | API incompatible |

### Version requirements in Cargo.toml:

```toml
[dependencies]
serde = "1.0"          # ^1.0 — any 1.x.x (>=1.0.0, <2.0.0)
rand = "0.8.5"         # ^0.8.5 — any 0.8.x (>=0.8.5, <0.9.0)
exact = "=1.2.3"       # exactly 1.2.3
range = ">=1.0, <1.5"  # range
```

---

## 5. Adding Dependencies

### Method 1: Edit `Cargo.toml` manually

```toml
[dependencies]
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
```

### Method 2: Use `cargo add` (recommended)

```bash
cargo add serde --features derive
cargo add serde_json
```

### Using your dependency:

```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
struct Person {
    name: String,
    age: u32,
    email: String,
}

fn main() {
    // Serialize to JSON
    let person = Person {
        name: String::from("Alice"),
        age: 30,
        email: String::from("alice@example.com"),
    };

    let json = serde_json::to_string_pretty(&person).unwrap();
    println!("Serialized:\n{json}");

    // Deserialize from JSON
    let json_str = r#"{"name":"Bob","age":25,"email":"bob@example.com"}"#;
    let bob: Person = serde_json::from_str(json_str).unwrap();
    println!("\nDeserialized: {bob:?}");
}
```

**Output:**
```
Serialized:
{
  "name": "Alice",
  "age": 30,
  "email": "alice@example.com"
}

Deserialized: Person { name: "Bob", age: 25, email: "bob@example.com" }
```

---

## 6. Cargo.lock

`Cargo.lock` records the **exact versions** of all dependencies used. It's auto-generated.

| | Cargo.toml | Cargo.lock |
|---|---|---|
| Written by | You | Cargo |
| Contains | Version requirements | Exact versions |
| Commit to git? | ✅ Always | ✅ For binaries, ❌ For libraries |
| Purpose | "What I want" | "What I got" |

```bash
cargo update           # update Cargo.lock to newest compatible versions
cargo update -p serde  # update only serde
```

---

## 7. Creating a Library Crate

```bash
cargo new my_math --lib
```

This creates:
```
my_math/
├── Cargo.toml
└── src/
    └── lib.rs
```

**`src/lib.rs`:**
```rust
/// Adds two numbers together.
///
/// # Examples
///
/// ```
/// assert_eq!(my_math::add(2, 3), 5);
/// ```
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}

/// Computes the factorial of n.
pub fn factorial(n: u64) -> u64 {
    (1..=n).product()
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_add() {
        assert_eq!(add(2, 3), 5);
        assert_eq!(add(-1, 1), 0);
    }

    #[test]
    fn test_factorial() {
        assert_eq!(factorial(0), 1);
        assert_eq!(factorial(5), 120);
    }
}
```

### Using your library from another project:

```toml
# In another project's Cargo.toml
[dependencies]
my_math = { path = "../my_math" }
```

```rust
use my_math::{add, factorial};

fn main() {
    println!("2 + 3 = {}", add(2, 3));
    println!("5! = {}", factorial(5));
}
```

---

## 8. The Crate Root

The **crate root** is the source file that Cargo passes to the compiler as the starting point:

| Crate type | Root file | What it contains |
|---|---|---|
| Binary | `src/main.rs` | `fn main()` + module declarations |
| Library | `src/lib.rs` | Public API + module declarations |

Everything in your crate is part of a module tree rooted at the crate root:

```
crate (root)
├── mod garden (src/garden.rs)
│   └── mod vegetables (src/garden/vegetables.rs)
├── mod utils (src/utils.rs)
└── fn main()
```

---

## 9. Essential Cargo Commands

```bash
# Creating
cargo new my_app           # new binary project
cargo new my_lib --lib     # new library project
cargo init                 # init in current directory

# Building
cargo build                # debug build → target/debug/
cargo build --release      # optimised build → target/release/
cargo check                # type-check without building (fast!)

# Running
cargo run                  # build and run
cargo run --release        # run optimised
cargo run -- arg1 arg2     # pass args to your program

# Testing
cargo test                 # run all tests
cargo test test_name       # run tests matching a name

# Dependencies
cargo add serde            # add a dependency
cargo add serde --features derive  # with features
cargo remove serde         # remove a dependency
cargo update               # update Cargo.lock

# Information
cargo doc --open           # generate and open documentation
cargo tree                 # show dependency tree
cargo clippy               # run the linter
cargo fmt                  # format code
```

---

## 10. Features

Features let you conditionally include functionality:

```toml
# Cargo.toml
[features]
default = ["json"]           # enabled by default
json = ["dep:serde_json"]    # opt-in feature
logging = []                 # feature flag, no extra deps

[dependencies]
serde = { version = "1.0", features = ["derive"] }
serde_json = { version = "1.0", optional = true }
```

```rust
// In your code
#[cfg(feature = "json")]
pub fn to_json<T: serde::Serialize>(value: &T) -> String {
    serde_json::to_string(value).unwrap()
}

#[cfg(feature = "logging")]
pub fn log(msg: &str) {
    println!("[LOG] {msg}");
}
```

```bash
cargo build                          # default features
cargo build --features "logging"     # add logging
cargo build --no-default-features    # disable defaults
```

---

## 11. Publishing to crates.io

The Rust community shares libraries on [crates.io](https://crates.io):

```bash
# 1. Login (one-time)
cargo login YOUR_API_TOKEN

# 2. Ensure Cargo.toml has required metadata
#    name, version, description, license

# 3. Check everything is ready
cargo publish --dry-run

# 4. Publish!
cargo publish
```

### Finding crates:
- [crates.io](https://crates.io) — official registry
- [lib.rs](https://lib.rs) — better search and categorisation
- `cargo search keyword` — search from command line

### Popular crates to know:

| Crate | Purpose |
|---|---|
| `serde` / `serde_json` | Serialization/deserialization |
| `rand` | Random number generation |
| `tokio` | Async runtime |
| `clap` | CLI argument parsing |
| `reqwest` | HTTP client |
| `anyhow` / `thiserror` | Error handling |
| `log` / `env_logger` | Logging |
| `regex` | Regular expressions |

---

## 12. Summary Cheat Sheet

```
CRATE TYPES
────────────────────────────────────────────────────────────
Binary crate     src/main.rs       produces executable
Library crate    src/lib.rs        produces .rlib

CARGO.TOML
────────────────────────────────────────────────────────────
[package]        name, version, edition
[dependencies]   runtime dependencies
[dev-dependencies] test/bench only
[features]       optional functionality

DEPENDENCY VERSIONS
────────────────────────────────────────────────────────────
"1.0"           ^1.0 — compatible updates (>=1.0.0, <2.0.0)
"=1.2.3"        exact version
">=1, <2"       range

CARGO COMMANDS
────────────────────────────────────────────────────────────
cargo new/init   create project
cargo build      compile
cargo run        compile + run
cargo check      type-check (fastest)
cargo test       run tests
cargo add        add dependency
cargo doc        generate docs
cargo tree       show dep tree
cargo clippy     lint
cargo fmt        format

FEATURES
────────────────────────────────────────────────────────────
#[cfg(feature = "json")]     conditional compilation
optional = true              optional dependency
```

---

## What's Next?

**Lesson 27 — Testing** — Write unit tests with `#[test]`, use `assert!` macros, run with `cargo test`, and test organisation patterns.

## Further Reading
- [The Rust Book — Ch 7: Managing Growing Projects](https://doc.rust-lang.org/book/ch07-00-managing-growing-projects-with-packages-crates-and-modules.html)
- [The Cargo Book](https://doc.rust-lang.org/cargo/)
- [crates.io](https://crates.io)

---

*Cargo is your best friend in the Rust ecosystem! 🦀*
