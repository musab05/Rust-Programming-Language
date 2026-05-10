# ✅ Lesson 91 — Answers: Cross-Compilation (B7)

---

## Section A

### A1
`aarch64-unknown-linux-gnu`:
- `aarch64` — Architecture: 64-bit ARM
- `unknown` — Vendor: not specified
- `linux` — OS: Linux kernel
- `gnu` — ABI: GNU libc (glibc)

### A2
`musl` produces **fully static** binaries with no runtime dependencies. The binary runs on any Linux distribution regardless of glibc version. Perfect for Docker containers, minimal images, and deployment to unknown environments.

---

## Section B

### A3
```rust
fn config_dir() -> String {
    #[cfg(target_os = "windows")]
    { std::env::var("APPDATA").unwrap_or("C:\\Users".into()) }
    #[cfg(target_os = "linux")]
    { format!("{}/.config", std::env::var("HOME").unwrap_or("/tmp".into())) }
    #[cfg(target_os = "macos")]
    { format!("{}/Library/Application Support", std::env::var("HOME").unwrap_or("/tmp".into())) }
}
```

### A4
```bash
rustup target add aarch64-unknown-linux-gnu
cargo install cross
cross build --release --target aarch64-unknown-linux-gnu
# Binary at: target/aarch64-unknown-linux-gnu/release/my_app
```

---

## Section C

### A5
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | It downloads the pre-compiled std for that target |
| 2 | **True** | cross runs builds inside Docker with all tools pre-installed |
| 3 | **False** | musl binaries are **statically linked** — no glibc needed |
| 4 | **True** | `#[cfg]` is compile-time — code is excluded entirely |
| 5 | **True** | cross uses QEMU emulation inside Docker for foreign arch tests |
| 6 | **False** | Crates with C dependencies may need linkers or cross-compilers |

---

## 🏆 Lesson 91 Complete!

**Next up:** [Lesson 92 — Observer / Event System](../lesson_92_observer/lesson_92_observer.md) 🦀
