# ✅ Lesson 77 — Answers: CI/CD with GitHub Actions (B8)

---

## Section A

### A1
- **CI (Continuous Integration)**: Automatically build, lint, and test code on every push/PR. Catches bugs early.
- **CD (Continuous Deployment/Delivery)**: Automatically build release artifacts and deploy or publish when tests pass (e.g., on tag push).

### A2
- **stable**: What most users run — must work.
- **beta**: Upcoming stable — catch regressions early.
- **nightly**: Bleeding edge — verify compatibility, test nightly features.
- **MSRV**: Minimum Supported Rust Version — ensure you don't accidentally use too-new features.

---

## Section B

### A3
```yaml
name: Rust CI
on: [push, pull_request]
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
        with: { components: "clippy, rustfmt" }
      - run: cargo fmt --all -- --check
      - run: cargo clippy --all-targets -- -D warnings

  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        toolchain: [stable, nightly, "1.75.0"]
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@master
        with: { toolchain: "${{ matrix.toolchain }}" }
      - uses: Swatinem/rust-cache@v2
      - run: cargo test --verbose
```

### A4
```yaml
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
      - uses: Swatinem/rust-cache@v2  # one line — handles everything
      - run: cargo test
```

---

## Section C

### A5
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | Workflow YAML files go in `.github/workflows/` |
| 2 | **True** | `-D warnings` promotes warnings to errors, failing CI |
| 3 | **False** | Matrix jobs run **in parallel** by default |
| 4 | **True** | It caches cargo registry, git checkout, and target dir |
| 5 | **True** | `on: push: tags: ['v*']` triggers on version tags |
| 6 | **False** | `--check` only reports issues — it doesn't modify files |

---

## 🏆 Lesson 77 Complete!

**Next up:** [Lesson 78 — RAII & Guard Types](../lesson_78_raii_guards/lesson_78_raii_guards.md) 🦀
