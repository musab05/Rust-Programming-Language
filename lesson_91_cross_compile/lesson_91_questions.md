# 🧪 Lesson 91 — Questions: Cross-Compilation (B7)

> **Lesson:** [lesson_91_cross_compile.md](./lesson_91_cross_compile.md)  
> **Answers:** [lesson_91_answers.md](./lesson_91_answers.md)

---

## Section A — Conceptual

### Q1
What is a target triple? Break down `aarch64-unknown-linux-gnu`.

### Q2
Why is `x86_64-unknown-linux-musl` useful for deployment?

---

## Section B — Write It Yourself

### Q3 — Cross-platform config (Roadmap Practice Task)
Write a function that returns the platform's config directory using `#[cfg(target_os)]` for Windows, Linux, and macOS.

### Q4
List the commands to cross-compile a pure Rust CLI for Linux ARM64 from a Windows host.

---

## Section C — True or False?

### Q5
1. `rustup target add` installs the standard library for a target.
2. `cross` uses Docker to provide cross-compilation toolchains.
3. Static musl binaries require glibc on the target machine.
4. `#[cfg(target_os = "linux")]` removes code at compile time for non-Linux builds.
5. `cross test` can run tests for foreign architectures using QEMU.
6. All Rust crates can be cross-compiled without modification.

---

*Cross-compile: one codebase, every platform! 🦀*
