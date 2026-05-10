# 📘 Lesson 77 — CI/CD with GitHub Actions (B8)

> **Series:** Rust From Zero · Intermediate Level (Gap Fill)  
> **Roadmap ID:** B8 · Category: 🔧 Tooling  
> **Previous:** [Lesson 76 — Cargo Features & Profiles](../lesson_76_cargo_features/lesson_76_cargo_features.md)  
> **Next:** [Lesson 78 — RAII & Guard Types](../lesson_78_raii_guards/lesson_78_raii_guards.md)  
> **Practice:** [Questions](./lesson_77_questions.md) · [Answers](./lesson_77_answers.md)  
> **Practice Task:** Set up CI that tests on stable + nightly + MSRV

---

## Table of Contents

1. [What Is CI/CD?](#1-what-is-cicd)
2. [GitHub Actions Basics](#2-github-actions-basics)
3. [Minimal Rust CI](#3-minimal-rust-ci)
4. [Testing on Multiple Toolchains](#4-testing-on-multiple-toolchains)
5. [Caching Dependencies](#5-caching-dependencies)
6. [Clippy and Rustfmt Checks](#6-clippy-and-rustfmt-checks)
7. [Full Production Workflow](#7-full-production-workflow)
8. [Cross-Platform Testing](#8-cross-platform-testing)
9. [Release Automation](#9-release-automation)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. What Is CI/CD?

```
Developer pushes code
        ↓
┌──── CI Pipeline ────┐
│ 1. Build            │
│ 2. Lint (clippy)    │  ← Continuous Integration
│ 3. Format (rustfmt) │
│ 4. Test             │
│ 5. Security audit   │
└─────────────────────┘
        ↓ (all pass)
┌──── CD Pipeline ────┐
│ 6. Build release    │  ← Continuous Deployment
│ 7. Publish / Deploy │
└─────────────────────┘
```

---

## 2. GitHub Actions Basics

Workflows live in `.github/workflows/`:

```
my_project/
├── .github/
│   └── workflows/
│       ├── ci.yml        ← runs on every push/PR
│       └── release.yml   ← runs on tags
├── src/
└── Cargo.toml
```

### Workflow anatomy:

```yaml
name: CI                         # workflow name
on: [push, pull_request]         # triggers

jobs:
  build:                         # job name
    runs-on: ubuntu-latest       # runner OS
    steps:
      - uses: actions/checkout@v4       # check out code
      - name: Build
        run: cargo build --verbose
      - name: Test
        run: cargo test --verbose
```

---

## 3. Minimal Rust CI

```yaml
# .github/workflows/ci.yml
name: Rust CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  CARGO_TERM_COLOR: always

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install Rust
        uses: dtolnay/rust-toolchain@stable

      - name: Build
        run: cargo build --verbose

      - name: Run tests
        run: cargo test --verbose
```

---

## 4. Testing on Multiple Toolchains

```yaml
name: Multi-Toolchain CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        toolchain:
          - stable
          - beta
          - nightly
          - "1.75.0"    # MSRV (Minimum Supported Rust Version)

    steps:
      - uses: actions/checkout@v4

      - name: Install Rust ${{ matrix.toolchain }}
        uses: dtolnay/rust-toolchain@master
        with:
          toolchain: ${{ matrix.toolchain }}

      - name: Build
        run: cargo build --verbose

      - name: Test
        run: cargo test --verbose
```

### Set MSRV in Cargo.toml:

```toml
[package]
name = "my_crate"
version = "0.1.0"
edition = "2021"
rust-version = "1.75.0"  # minimum supported Rust version
```

---

## 5. Caching Dependencies

Speed up CI by caching compiled dependencies:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable

      - name: Cache cargo registry and build
        uses: actions/cache@v4
        with:
          path: |
            ~/.cargo/registry
            ~/.cargo/git
            target
          key: ${{ runner.os }}-cargo-${{ hashFiles('**/Cargo.lock') }}
          restore-keys: |
            ${{ runner.os }}-cargo-

      - run: cargo test --verbose
```

Or use the dedicated Rust cache action:

```yaml
      - uses: Swatinem/rust-cache@v2  # automatic Rust-specific caching
```

---

## 6. Clippy and Rustfmt Checks

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
        with:
          components: clippy, rustfmt

      - name: Check formatting
        run: cargo fmt --all -- --check

      - name: Run clippy
        run: cargo clippy --all-targets --all-features -- -D warnings
```

### Security audit:

```yaml
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install cargo-audit
        run: cargo install cargo-audit
      - name: Security audit
        run: cargo audit
```

---

## 7. Full Production Workflow

```yaml
name: Rust CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  CARGO_TERM_COLOR: always

jobs:
  # Job 1: Format check
  fmt:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
        with:
          components: rustfmt
      - run: cargo fmt --all -- --check

  # Job 2: Clippy lint
  clippy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
        with:
          components: clippy
      - uses: Swatinem/rust-cache@v2
      - run: cargo clippy --all-targets --all-features -- -D warnings

  # Job 3: Test on multiple toolchains
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        toolchain: [stable, beta, nightly]
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@master
        with:
          toolchain: ${{ matrix.toolchain }}
      - uses: Swatinem/rust-cache@v2
      - run: cargo test --all-features --verbose

  # Job 4: MSRV check
  msrv:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@master
        with:
          toolchain: "1.75.0"
      - run: cargo build --verbose

  # Job 5: Documentation
  docs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
      - run: cargo doc --no-deps --all-features
        env:
          RUSTDOCFLAGS: "-D warnings"
```

---

## 8. Cross-Platform Testing

```yaml
jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
        toolchain: [stable]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@master
        with:
          toolchain: ${{ matrix.toolchain }}
      - uses: Swatinem/rust-cache@v2
      - run: cargo test --verbose
```

---

## 9. Release Automation

```yaml
name: Release

on:
  push:
    tags: ['v*']  # triggers on v1.0.0, v0.2.1, etc.

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable

      - name: Build release binary
        run: cargo build --release

      - name: Create GitHub Release
        uses: softprops/action-gh-release@v1
        with:
          files: target/release/my_app
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

## 10. Summary Cheat Sheet

```
WORKFLOW FILE
────────────────────────────────────────────────────────────
.github/workflows/ci.yml

TRIGGERS
────────────────────────────────────────────────────────────
on: [push, pull_request]
on: push: tags: ['v*']

KEY ACTIONS
────────────────────────────────────────────────────────────
actions/checkout@v4         clone the repo
dtolnay/rust-toolchain@     install Rust
Swatinem/rust-cache@v2      cache dependencies

CI STEPS
────────────────────────────────────────────────────────────
cargo fmt -- --check        formatting
cargo clippy -- -D warnings linting
cargo test --all-features   testing
cargo doc --no-deps         documentation
cargo audit                 security

MATRIX TESTING
────────────────────────────────────────────────────────────
strategy:
  matrix:
    toolchain: [stable, beta, nightly, "1.75.0"]
    os: [ubuntu-latest, macos-latest, windows-latest]
```

---

## What's Next?

**Lesson 78 — RAII & Guard Types** — The Drop trait, resource cleanup, and MutexGuard-style patterns.

## Further Reading
- [GitHub Actions Docs](https://docs.github.com/en/actions)
- [dtolnay/rust-toolchain](https://github.com/dtolnay/rust-toolchain)
- [Swatinem/rust-cache](https://github.com/Swatinem/rust-cache)

---

*CI/CD: every push is tested, every release is automated! 🦀*
