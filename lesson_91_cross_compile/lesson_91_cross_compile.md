# 📘 Lesson 91 — Cross-Compilation (B7)

> **Series:** Rust From Zero · Advanced Level (Gap Fill)  
> **Roadmap ID:** B7 · Category: 🔧 Tooling  
> **Previous:** [Lesson 90 — cargo-expand & cargo-asm](../lesson_90_cargo_tools/lesson_90_cargo_tools.md)  
> **Next:** [Lesson 92 — Observer / Event System](../lesson_92_observer/lesson_92_observer.md)  
> **Practice:** [Questions](./lesson_91_questions.md) · [Answers](./lesson_91_answers.md)  
> **Practice Task:** Cross-compile a CLI tool for Linux from Windows/macOS

---

## Table of Contents

1. [What Is Cross-Compilation?](#1-what-is-cross-compilation)
2. [Targets and Triples](#2-targets-and-triples)
3. [Adding Targets](#3-adding-targets)
4. [Simple Cross-Compilation](#4-simple-cross-compilation)
5. [Cross-Compiling with cross](#5-cross-compiling-with-cross)
6. [Conditional Compilation](#6-conditional-compilation)
7. [Common Target Triples](#7-common-target-triples)
8. [Linking and C Dependencies](#8-linking-and-c-dependencies)
9. [Summary Cheat Sheet](#9-summary-cheat-sheet)

---

## 1. What Is Cross-Compilation?

Building a binary on one platform that runs on a **different** platform:

```
Your machine (host)         Target machine
┌──────────────┐            ┌──────────────┐
│ Windows x64  │ ──build──▶ │ Linux ARM64  │
│              │            │ (Raspberry Pi)│
└──────────────┘            └──────────────┘
```

Rust makes this remarkably easy — LLVM supports many backends.

---

## 2. Targets and Triples

A target triple describes: `arch-vendor-os-abi`

```
x86_64-unknown-linux-gnu
  │       │       │    │
  │       │       │    └─ ABI (GNU libc)
  │       │       └────── OS (Linux)
  │       └────────────── Vendor (unknown)
  └────────────────────── Architecture (x86 64-bit)
```

```bash
# See your current target
rustc --print cfg | grep target

# List ALL available targets
rustup target list

# List installed targets
rustup target list --installed
```

---

## 3. Adding Targets

```bash
# Add Linux target (from Windows/macOS)
rustup target add x86_64-unknown-linux-gnu
rustup target add x86_64-unknown-linux-musl

# Add Windows target (from Linux/macOS)
rustup target add x86_64-pc-windows-msvc

# Add macOS target (from Linux/Windows)
rustup target add x86_64-apple-darwin
rustup target add aarch64-apple-darwin

# Add ARM targets (for Raspberry Pi, embedded)
rustup target add aarch64-unknown-linux-gnu
rustup target add armv7-unknown-linux-gnueabihf

# Add WebAssembly
rustup target add wasm32-unknown-unknown
rustup target add wasm32-wasi
```

---

## 4. Simple Cross-Compilation

### Pure Rust projects (no C dependencies):

```bash
# Build for Linux (musl — static linking, no libc dependency)
cargo build --release --target x86_64-unknown-linux-musl

# Build for WebAssembly
cargo build --release --target wasm32-unknown-unknown

# Output location:
# target/x86_64-unknown-linux-musl/release/my_app
# target/wasm32-unknown-unknown/release/my_app.wasm
```

### Set default target in `.cargo/config.toml`:

```toml
# .cargo/config.toml
[build]
target = "x86_64-unknown-linux-musl"

# Or specify linker for specific targets
[target.aarch64-unknown-linux-gnu]
linker = "aarch64-linux-gnu-gcc"
```

---

## 5. Cross-Compiling with cross

`cross` uses Docker containers with pre-configured toolchains:

```bash
# Install cross
cargo install cross

# Cross-compile (uses Docker — no manual toolchain setup!)
cross build --release --target aarch64-unknown-linux-gnu
cross build --release --target x86_64-unknown-linux-gnu
cross build --release --target armv7-unknown-linux-gnueabihf

# Run tests on target (via QEMU in Docker)
cross test --target aarch64-unknown-linux-gnu
```

**Advantages of `cross`:**
- No manual linker/toolchain installation
- Handles C dependencies automatically
- Supports running tests via QEMU emulation
- Works identically on all host platforms

---

## 6. Conditional Compilation

Write platform-specific code:

```rust
fn get_config_dir() -> String {
    #[cfg(target_os = "windows")]
    { std::env::var("APPDATA").unwrap_or_else(|_| "C:\\Users".into()) }

    #[cfg(target_os = "linux")]
    { std::env::var("HOME").map(|h| format!("{h}/.config")).unwrap_or("/tmp".into()) }

    #[cfg(target_os = "macos")]
    { std::env::var("HOME").map(|h| format!("{h}/Library/Application Support")).unwrap_or("/tmp".into()) }
}

fn main() {
    println!("Config dir: {}", get_config_dir());

    // Runtime checks
    println!("OS:   {}", std::env::consts::OS);         // "windows", "linux", "macos"
    println!("Arch: {}", std::env::consts::ARCH);       // "x86_64", "aarch64"
    println!("Exe:  {}", std::env::consts::EXE_SUFFIX); // ".exe" or ""
}
```

### Available cfg predicates:

```rust
#[cfg(target_os = "linux")]
#[cfg(target_os = "windows")]
#[cfg(target_os = "macos")]
#[cfg(target_arch = "x86_64")]
#[cfg(target_arch = "aarch64")]
#[cfg(target_family = "unix")]     // linux + macOS
#[cfg(target_family = "windows")]
#[cfg(target_env = "gnu")]         // GNU libc
#[cfg(target_env = "musl")]        // musl libc (static)
#[cfg(target_pointer_width = "64")]
```

---

## 7. Common Target Triples

| Triple | Platform | Notes |
|---|---|---|
| `x86_64-unknown-linux-gnu` | Linux x64 (glibc) | Most common Linux |
| `x86_64-unknown-linux-musl` | Linux x64 (static) | No libc dependency |
| `aarch64-unknown-linux-gnu` | Linux ARM64 | Raspberry Pi 4, AWS Graviton |
| `armv7-unknown-linux-gnueabihf` | Linux ARMv7 | Raspberry Pi 3 |
| `x86_64-pc-windows-msvc` | Windows x64 | Requires MSVC |
| `x86_64-pc-windows-gnu` | Windows x64 (MinGW) | Uses GNU toolchain |
| `x86_64-apple-darwin` | macOS Intel | |
| `aarch64-apple-darwin` | macOS Apple Silicon | M1/M2/M3 |
| `wasm32-unknown-unknown` | WebAssembly | Browser/Node |
| `wasm32-wasi` | WebAssembly (WASI) | Server-side Wasm |

---

## 8. Linking and C Dependencies

### Static linking with musl:

```bash
# Produces a fully static binary — no runtime dependencies
cargo build --release --target x86_64-unknown-linux-musl

# Check: no dynamic dependencies
# ldd target/x86_64-unknown-linux-musl/release/my_app
# → "not a dynamic executable" (fully static!)
```

### When you have C dependencies:

```toml
# .cargo/config.toml
[target.aarch64-unknown-linux-gnu]
linker = "aarch64-linux-gnu-gcc"
```

```bash
# Install cross-compilation toolchain (Ubuntu/Debian)
sudo apt install gcc-aarch64-linux-gnu

# Or just use `cross` — it handles everything:
cross build --release --target aarch64-unknown-linux-gnu
```

---

## 9. Summary Cheat Sheet

```
ADDING TARGETS
────────────────────────────────────────────────────────────
rustup target add <triple>
rustup target list --installed

BUILDING
────────────────────────────────────────────────────────────
cargo build --target <triple>
cross build --target <triple>     (Docker-based)

CONDITIONAL COMPILATION
────────────────────────────────────────────────────────────
#[cfg(target_os = "linux")]
#[cfg(target_arch = "x86_64")]
#[cfg(target_family = "unix")]
std::env::consts::OS              runtime OS string

STATIC LINKING
────────────────────────────────────────────────────────────
x86_64-unknown-linux-musl         fully static binary
No glibc dependency, runs anywhere

CONFIG
────────────────────────────────────────────────────────────
.cargo/config.toml
[target.aarch64-unknown-linux-gnu]
linker = "aarch64-linux-gnu-gcc"

CROSS (DOCKER)
────────────────────────────────────────────────────────────
cargo install cross
cross build --target <triple>     handles everything
cross test --target <triple>      runs tests via QEMU
```

---

## What's Next?

**Lesson 92 — Observer / Event System** — Publish-subscribe pattern in Rust with trait objects and callbacks.

## Further Reading
- [Rust Cross-Compilation Guide](https://rust-lang.github.io/rustup/cross-compilation.html)
- [cross](https://github.com/cross-rs/cross)

---

*Cross-compile: write once, build for everywhere! 🦀*
