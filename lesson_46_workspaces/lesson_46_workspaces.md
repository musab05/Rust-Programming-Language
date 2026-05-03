# 📘 Lesson 46 — Workspaces (M4)

> **Series:** Rust From Zero · Intermediate Level  
> **Roadmap ID:** M4 · Category: 📦 Modules & Crates  
> **Previous:** [Lesson 45 — Associated Types](../lesson_45_associated_types/lesson_45_associated_types.md)  
> **Next:** [Lesson 47 — Publishing to crates.io](../lesson_47_publishing/lesson_47_publishing.md)  
> **Practice:** [Questions](./lesson_46_questions.md) · [Answers](./lesson_46_answers.md)  
> **Practice Task:** Create a 3-crate workspace with shared dependencies

---

## Table of Contents

1. [What Are Workspaces?](#1-what-are-workspaces)
2. [Creating a Workspace](#2-creating-a-workspace)
3. [Adding Crates to a Workspace](#3-adding-crates-to-a-workspace)
4. [Dependencies Between Workspace Crates](#4-dependencies-between-workspace-crates)
5. [Shared Dependencies](#5-shared-dependencies)
6. [Running and Building](#6-running-and-building)
7. [Testing in Workspaces](#7-testing-in-workspaces)
8. [Workspace Configuration](#8-workspace-configuration)
9. [Real-World Example](#9-real-world-example)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. What Are Workspaces?

A **workspace** is a set of related crates that share:
- A single `Cargo.lock` (consistent dependency versions)
- A single `target/` directory (faster builds)
- Common configuration

Perfect for:
- Splitting a large project into logical crates
- Building a library + CLI tool
- Sharing code between a backend and CLI

---

## 2. Creating a Workspace

### Directory structure:

```
my_project/
├── Cargo.toml          ← workspace root
├── Cargo.lock          ← shared lock file
├── target/             ← shared build output
├── core/               ← library crate
│   ├── Cargo.toml
│   └── src/
│       └── lib.rs
├── cli/                ← binary crate
│   ├── Cargo.toml
│   └── src/
│       └── main.rs
└── api/                ← another binary crate
    ├── Cargo.toml
    └── src/
        └── main.rs
```

### Root `Cargo.toml`:

```toml
[workspace]
members = [
    "core",
    "cli",
    "api",
]

# Shared dependency versions (Rust 1.64+)
[workspace.dependencies]
serde = { version = "1", features = ["derive"] }
anyhow = "1"
```

The root `Cargo.toml` has **no** `[package]` section — it only defines the workspace.

---

## 3. Adding Crates to a Workspace

### Library crate (`core/Cargo.toml`):

```toml
[package]
name = "my_core"
version = "0.1.0"
edition = "2021"

[dependencies]
serde = { workspace = true }   # uses version from workspace root
```

### Binary crate (`cli/Cargo.toml`):

```toml
[package]
name = "my_cli"
version = "0.1.0"
edition = "2021"

[dependencies]
my_core = { path = "../core" }  # local dependency
anyhow = { workspace = true }
```

### Creating with cargo:

```bash
# Create workspace root
mkdir my_project && cd my_project

# Create member crates
cargo new core --lib
cargo new cli
cargo new api
```

Then create the root `Cargo.toml` with `[workspace]`.

---

## 4. Dependencies Between Workspace Crates

Workspace crates can depend on each other using `path`:

```toml
# In cli/Cargo.toml
[dependencies]
my_core = { path = "../core" }
```

```rust
// cli/src/main.rs
use my_core::some_function;

fn main() {
    some_function();
}
```

```rust
// core/src/lib.rs
pub fn some_function() {
    println!("Hello from core!");
}
```

---

## 5. Shared Dependencies

Use `[workspace.dependencies]` to ensure all crates use the same version:

```toml
# Root Cargo.toml
[workspace]
members = ["core", "cli", "api"]

[workspace.dependencies]
serde = { version = "1.0", features = ["derive"] }
tokio = { version = "1", features = ["full"] }
anyhow = "1"
log = "0.4"
```

```toml
# core/Cargo.toml
[dependencies]
serde = { workspace = true }
log = { workspace = true }

# cli/Cargo.toml
[dependencies]
my_core = { path = "../core" }
anyhow = { workspace = true }
tokio = { workspace = true }
```

Benefits:
- **One `Cargo.lock`** — all crates use the same dependency versions
- **One `target/`** — shared compilation, faster builds
- **Easy updates** — change version once, affects all crates

---

## 6. Running and Building

```bash
# Build everything
cargo build

# Build a specific crate
cargo build -p my_cli

# Run a specific binary
cargo run -p my_cli

# Run with arguments
cargo run -p my_cli -- --verbose

# Check all crates
cargo check

# Check a specific crate
cargo check -p my_core
```

---

## 7. Testing in Workspaces

```bash
# Test everything
cargo test

# Test a specific crate
cargo test -p my_core

# Test a specific test
cargo test -p my_core -- test_name
```

### Integration tests between crates:

```rust
// core/src/lib.rs
pub fn add(a: i32, b: i32) -> i32 { a + b }

// core/tests/integration.rs  (integration test)
use my_core::add;

#[test]
fn test_add() {
    assert_eq!(add(2, 3), 5);
}
```

---

## 8. Workspace Configuration

### Shared settings:

```toml
[workspace]
members = ["core", "cli", "api"]
resolver = "2"  # recommended for new projects

[workspace.package]
version = "0.1.0"
edition = "2021"
authors = ["Your Name <you@example.com>"]
license = "MIT"

[workspace.dependencies]
serde = { version = "1", features = ["derive"] }
```

### Using workspace package metadata:

```toml
# core/Cargo.toml
[package]
name = "my_core"
version.workspace = true     # inherits from workspace
edition.workspace = true
authors.workspace = true
```

### Exclude members:

```toml
[workspace]
members = ["core", "cli"]
exclude = ["experiments"]  # don't include this directory
```

---

## 9. Real-World Example

A typical web application workspace:

```
webapp/
├── Cargo.toml
├── domain/          ← business logic (pure Rust, no framework)
│   ├── Cargo.toml
│   └── src/lib.rs
├── db/              ← database layer
│   ├── Cargo.toml
│   └── src/lib.rs
├── api/             ← HTTP API server
│   ├── Cargo.toml
│   └── src/main.rs
└── cli/             ← CLI management tool
    ├── Cargo.toml
    └── src/main.rs
```

```toml
# Root Cargo.toml
[workspace]
members = ["domain", "db", "api", "cli"]

[workspace.dependencies]
serde = { version = "1", features = ["derive"] }
anyhow = "1"
```

```toml
# domain/Cargo.toml — no external deps, pure logic
[package]
name = "domain"
version = "0.1.0"
edition = "2021"

[dependencies]
serde = { workspace = true }
```

```toml
# db/Cargo.toml — depends on domain
[package]
name = "db"
version = "0.1.0"
edition = "2021"

[dependencies]
domain = { path = "../domain" }
```

```toml
# api/Cargo.toml — depends on domain + db
[package]
name = "api"
version = "0.1.0"
edition = "2021"

[dependencies]
domain = { path = "../domain" }
db = { path = "../db" }
anyhow = { workspace = true }
```

---

## 10. Summary Cheat Sheet

```
WORKSPACE STRUCTURE
────────────────────────────────────────────────────────────
root/
├── Cargo.toml         [workspace] definition
├── Cargo.lock         shared across all crates
├── target/            shared build directory
└── crate_a/           individual crate

ROOT CARGO.TOML
────────────────────────────────────────────────────────────
[workspace]
members = ["crate_a", "crate_b"]

[workspace.dependencies]
serde = "1"            shared versions

MEMBER CARGO.TOML
────────────────────────────────────────────────────────────
[dependencies]
serde = { workspace = true }          use workspace version
sibling = { path = "../sibling" }     local dependency

COMMANDS
────────────────────────────────────────────────────────────
cargo build              build all
cargo build -p name      build one crate
cargo run -p name        run one binary
cargo test               test all
cargo test -p name       test one crate

BENEFITS
────────────────────────────────────────────────────────────
One Cargo.lock           consistent dependency versions
One target/              faster incremental builds
Shared config            DRY package metadata
```

---

## What's Next?

**Lesson 47 — Publishing to crates.io** — Share your Rust libraries with the world. Learn documentation, versioning, and the publishing workflow.

## Further Reading
- [Cargo Book — Workspaces](https://doc.rust-lang.org/cargo/reference/workspaces.html)
- [The Rust Book — Ch 14.3: Cargo Workspaces](https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html)

---

*Workspaces: organize large projects like a pro! 🦀*
