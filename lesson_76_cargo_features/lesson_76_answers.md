# ✅ Lesson 76 — Answers: Cargo Features & Profiles (B1)

---

## Section A

### A1
- `#[cfg(feature = "x")]` — **compile-time**: removes the annotated code entirely if the feature is off. Code doesn't exist in the binary.
- `cfg!(feature = "x")` — **runtime bool**: returns `true`/`false`. Both branches are compiled, but the optimizer eliminates the dead branch.

### A2
`lto = true` enables **Link-Time Optimization** — the compiler optimizes across crate boundaries during linking.
- **Pro**: Faster, smaller binary (better inlining across crates)
- **Con**: Much slower compile times

---

## Section B

### A3
```toml
# Cargo.toml
[features]
default = []
logging = ["dep:log", "dep:env_logger"]

[dependencies]
log = { version = "0.4", optional = true }
env_logger = { version = "0.11", optional = true }
```

```rust
#[cfg(feature = "logging")]
fn init_log() { env_logger::init(); }

#[cfg(not(feature = "logging"))]
fn init_log() {}

fn main() {
    init_log();
    #[cfg(feature = "logging")]
    log::info!("Application started");
    println!("Running...");
}
```

### A4
```toml
[profile.dev]
opt-level = 1

[profile.release]
opt-level = 3
lto = true
codegen-units = 1
panic = "abort"
```

---

## Section C

### A5
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | `cargo build` defaults to the dev profile |
| 2 | **True** | `feat = ["dep:crate"]` includes the optional dependency |
| 3 | **False** | `--no-default-features` disables defaults; `--features` adds specific ones |
| 4 | **True** | `#[cfg]` is compile-time — code is not compiled at all |
| 5 | **True** | Max optimization = fast binary, slow compilation |
| 6 | **False** | Features are resolved at **compile time** |

---

## 🏆 Lesson 76 Complete!

**Next up:** [Lesson 77 — CI/CD with GitHub Actions](../lesson_77_ci_cd/lesson_77_ci_cd.md) 🦀
