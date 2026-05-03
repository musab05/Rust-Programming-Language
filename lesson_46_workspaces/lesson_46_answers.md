# ✅ Lesson 46 — Answers: Workspaces (M4)

---

## Section A

### A1
All crates share: one **`Cargo.lock`** (consistent dependency versions), one **`target/`** directory (shared compilation artifacts), and optionally shared **dependency versions** and **package metadata** via `[workspace.dependencies]` / `[workspace.package]`.

### A2
`{ workspace = true }` tells Cargo to use the dependency version defined in the workspace root's `[workspace.dependencies]`. This ensures all crates use the same version.

### A3
Use a `path` dependency: `sibling_crate = { path = "../sibling_crate" }`.

---

## Section B

### A4
```
math_workspace/
├── Cargo.toml
├── math_core/
│   ├── Cargo.toml
│   └── src/lib.rs
├── math_cli/
│   ├── Cargo.toml
│   └── src/main.rs
└── math_api/
    ├── Cargo.toml
    └── src/main.rs
```

**Root `Cargo.toml`:**
```toml
[workspace]
members = ["math_core", "math_cli", "math_api"]

[workspace.dependencies]
serde = { version = "1", features = ["derive"] }
anyhow = "1"
```

**`math_core/Cargo.toml`:**
```toml
[package]
name = "math_core"
version = "0.1.0"
edition = "2021"

[dependencies]
serde = { workspace = true }
```

**`math_cli/Cargo.toml`:**
```toml
[package]
name = "math_cli"
version = "0.1.0"
edition = "2021"

[dependencies]
math_core = { path = "../math_core" }
anyhow = { workspace = true }
```

**`math_api/Cargo.toml`:**
```toml
[package]
name = "math_api"
version = "0.1.0"
edition = "2021"

[dependencies]
math_core = { path = "../math_core" }
serde = { workspace = true }
anyhow = { workspace = true }
```

### A5
```toml
[workspace]
members = ["lib", "server", "tools"]

[workspace.dependencies]
tokio = { version = "1", features = ["full"] }
```

---

## Section C

### A6
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **False** | Workspace root has `[workspace]`, not `[package]` |
| 2 | **True** | One lock file ensures consistent versions |
| 3 | **True** | `cargo run -p crate_name` runs a specific binary |
| 4 | **False** | All crates share ONE `target/` at the workspace root |
| 5 | **True** | `path` dependencies link to sibling crates |
| 6 | **False** | Members can be libraries, binaries, or both |

---

## 🏆 Lesson 46 Complete!

**Next up:** [Lesson 47 — Publishing to crates.io](../lesson_47_publishing/lesson_47_publishing.md) 🦀
