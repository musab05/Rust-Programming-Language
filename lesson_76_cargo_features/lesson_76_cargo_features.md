# 📘 Lesson 76 — Cargo Features & Profiles (B1)

> **Series:** Rust From Zero · Intermediate Level (Gap Fill)  
> **Roadmap ID:** B1 · Category: 🔧 Tooling  
> **Previous:** [Lesson 75 — Zero-Cost Abstractions](../lesson_75_zero_cost/lesson_75_zero_cost.md)  
> **Next:** [Lesson 77 — CI/CD with GitHub Actions](../lesson_77_ci_cd/lesson_77_ci_cd.md)  
> **Practice:** [Questions](./lesson_76_questions.md) · [Answers](./lesson_76_answers.md)  
> **Practice Task:** Add a 'logging' feature flag; disable in release

---

## Table of Contents

1. [Cargo Profiles](#1-cargo-profiles)
2. [Profile Settings](#2-profile-settings)
3. [What Are Features?](#3-what-are-features)
4. [Defining Features](#4-defining-features)
5. [Using Features in Code](#5-using-features-in-code)
6. [Optional Dependencies as Features](#6-optional-dependencies-as-features)
7. [Feature Combinations](#7-feature-combinations)
8. [Default Features](#8-default-features)
9. [Real-World Example](#9-real-world-example)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. Cargo Profiles

Profiles control how your code is compiled:

```bash
cargo build           # uses [profile.dev]    — fast compile, slow code
cargo build --release  # uses [profile.release] — slow compile, fast code
cargo test            # uses [profile.test]   — like dev + test harness
cargo bench           # uses [profile.bench]  — like release + test harness
```

| Profile | Optimization | Debug Info | Assertions | Use Case |
|---|---|---|---|---|
| `dev` | 0 (none) | ✅ Full | ✅ On | Development |
| `release` | 3 (max) | ❌ Off | ❌ Off | Production |
| `test` | 0 | ✅ Full | ✅ On | Testing |
| `bench` | 3 | ❌ Off | ❌ Off | Benchmarking |

---

## 2. Profile Settings

Customize in `Cargo.toml`:

```toml
[profile.dev]
opt-level = 0          # 0=none, 1=basic, 2=more, 3=max, "s"=size, "z"=min-size
debug = true           # include debug symbols
overflow-checks = true # check integer overflow
incremental = true     # faster recompilation

[profile.release]
opt-level = 3          # maximum optimization
debug = false          # no debug symbols
lto = true             # link-time optimization (slower compile, faster binary)
codegen-units = 1      # single codegen unit (better optimization)
panic = "abort"        # don't unwind — smaller binary
strip = true           # strip symbols from binary
```

### Common customizations:

```toml
# Fast dev builds with some optimization (good for game dev)
[profile.dev]
opt-level = 1

# Release with debug symbols (for profiling)
[profile.release]
debug = true

# Dev dependencies at higher optimization (e.g., image processing)
[profile.dev.package."image"]
opt-level = 3
```

---

## 3. What Are Features?

Features are **compile-time flags** that enable/disable functionality:

```bash
# Build with default features
cargo build

# Build with a specific feature
cargo build --features "logging"

# Build with multiple features
cargo build --features "logging,json"

# Build with NO default features
cargo build --no-default-features

# Build with no defaults + specific features
cargo build --no-default-features --features "minimal"
```

---

## 4. Defining Features

```toml
# Cargo.toml
[package]
name = "my_app"
version = "0.1.0"

[features]
default = ["json"]        # enabled by default
logging = []              # empty — just a flag
json = ["dep:serde_json"] # enables an optional dependency
full = ["logging", "json"] # combines features

[dependencies]
serde = "1"
serde_json = { version = "1", optional = true }  # only included if "json" feature
log = { version = "0.4", optional = true }
env_logger = { version = "0.11", optional = true }
```

---

## 5. Using Features in Code

```rust
// Conditionally compile code based on features
#[cfg(feature = "logging")]
fn init_logging() {
    env_logger::init();
    log::info!("Logging initialized");
}

#[cfg(not(feature = "logging"))]
fn init_logging() {
    // no-op when logging is disabled
}

fn process(data: &str) -> String {
    #[cfg(feature = "logging")]
    log::debug!("Processing: {data}");

    let result = data.to_uppercase();

    #[cfg(feature = "json")]
    {
        let json = serde_json::json!({ "input": data, "output": &result });
        println!("JSON: {json}");
    }

    result
}

fn main() {
    init_logging();
    let result = process("hello");
    println!("Result: {result}");
}
```

### Using `cfg!` macro for runtime checks:

```rust
fn main() {
    if cfg!(feature = "logging") {
        println!("Logging feature is enabled");
    } else {
        println!("Logging feature is disabled");
    }

    // cfg! returns bool — code is still compiled (but optimizer removes dead branch)
    // #[cfg(...)] removes code entirely at compile time
}
```

---

## 6. Optional Dependencies as Features

```toml
[dependencies]
reqwest = { version = "0.12", optional = true }
tokio = { version = "1", optional = true, features = ["full"] }

[features]
default = []
http = ["dep:reqwest", "dep:tokio"]
```

```rust
#[cfg(feature = "http")]
pub async fn fetch(url: &str) -> Result<String, reqwest::Error> {
    reqwest::get(url).await?.text().await
}

#[cfg(not(feature = "http"))]
pub fn fetch(_url: &str) -> Result<String, String> {
    Err("HTTP feature not enabled. Build with --features http".into())
}
```

---

## 7. Feature Combinations

```toml
[features]
default = ["std"]
std = []              # standard library support
alloc = []            # alloc crate only (no_std + alloc)
serde = ["dep:serde"] # serialization support
full = ["std", "serde"]
```

```rust
#[cfg(feature = "std")]
use std::collections::HashMap;

#[cfg(all(feature = "alloc", not(feature = "std")))]
extern crate alloc;

// Require multiple features
#[cfg(all(feature = "serde", feature = "std"))]
fn serialize_to_file() { /* ... */ }

// Any of multiple features
#[cfg(any(feature = "json", feature = "yaml"))]
fn parse_config() { /* ... */ }
```

---

## 8. Default Features

```toml
[features]
default = ["colors", "unicode"]
colors = []
unicode = []
minimal = []

# Users can disable defaults:
# cargo build --no-default-features --features "minimal"
```

```toml
# In a dependency — disable its default features
[dependencies]
tokio = { version = "1", default-features = false, features = ["rt", "macros"] }
```

---

## 9. Real-World Example

```toml
[package]
name = "my_logger"

[features]
default = ["colors"]
colors = ["dep:colored"]
json_output = ["dep:serde", "dep:serde_json"]
file_output = []

[dependencies]
colored = { version = "2", optional = true }
serde = { version = "1", optional = true, features = ["derive"] }
serde_json = { version = "1", optional = true }
```

```rust
use std::fmt;

pub enum Level { Info, Warn, Error }

impl fmt::Display for Level {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        let s = match self {
            Level::Info => "INFO",
            Level::Warn => "WARN",
            Level::Error => "ERROR",
        };

        #[cfg(feature = "colors")]
        {
            use colored::Colorize;
            let colored = match self {
                Level::Info => s.green(),
                Level::Warn => s.yellow(),
                Level::Error => s.red(),
            };
            return write!(f, "{colored}");
        }

        #[cfg(not(feature = "colors"))]
        write!(f, "{s}")
    }
}

pub fn log(level: Level, msg: &str) {
    let line = format!("[{}] {msg}", level);

    #[cfg(feature = "json_output")]
    {
        let json = serde_json::json!({
            "level": format!("{level}"),
            "message": msg,
        });
        println!("{json}");
        return;
    }

    println!("{line}");

    #[cfg(feature = "file_output")]
    {
        use std::io::Write;
        if let Ok(mut f) = std::fs::OpenOptions::new()
            .append(true).create(true).open("app.log")
        {
            writeln!(f, "{line}").ok();
        }
    }
}
```

---

## 10. Summary Cheat Sheet

```
PROFILES
────────────────────────────────────────────────────────────
[profile.dev]          opt-level = 0   (fast compile)
[profile.release]      opt-level = 3   (fast code)
lto = true             link-time optimization
codegen-units = 1      better optimization
panic = "abort"        smaller binary

FEATURES
────────────────────────────────────────────────────────────
[features]
default = ["feat_a"]         enabled by default
feat_a = []                  just a flag
feat_b = ["dep:some_crate"]  enables optional dep

IN CODE
────────────────────────────────────────────────────────────
#[cfg(feature = "name")]       compile-time conditional
#[cfg(not(feature = "name"))]  inverse
cfg!(feature = "name")         runtime bool (dead code eliminated)

CARGO COMMANDS
────────────────────────────────────────────────────────────
cargo build --features "a,b"
cargo build --no-default-features
cargo build --release
```

---

## What's Next?

**Lesson 77 — CI/CD with GitHub Actions** — Automate testing, linting, and deployment for Rust projects.

## Further Reading
- [Cargo Book — Profiles](https://doc.rust-lang.org/cargo/reference/profiles.html)
- [Cargo Book — Features](https://doc.rust-lang.org/cargo/reference/features.html)

---

*Features & Profiles: compile exactly what you need! 🦀*
