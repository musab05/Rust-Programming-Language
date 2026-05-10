# ✅ Lesson 90 — Answers: cargo-expand & cargo-asm (B6)

---

## Section A

### A1
| Tool | Shows | Level |
|---|---|---|
| `cargo expand` | Macro-expanded **Rust code** | Source level |
| `cargo asm` | Generated **assembly instructions** | Machine level |

`expand` answers "what code do macros generate?" while `asm` answers "what instructions does the compiler emit?"

### A2
`cargo expand` uses the compiler's internal `-Zunpretty=expanded` flag, which is an unstable feature only available on nightly. Stable Rust doesn't expose macro expansion output.

---

## Section B

### A3
```rust
#[derive(Debug, Clone, PartialEq)]
struct Point { x: f64, y: f64 }
```

- **Debug** generates `impl Debug` with `debug_struct_field2_finish(f, "Point", "x", &self.x, "y", &self.y)`
- **Clone** generates `impl Clone` calling `Clone::clone()` on each field
- **PartialEq** generates `impl PartialEq` comparing `self.x == other.x && self.y == other.y`

### A4
```rust
pub fn sum_iter(data: &[i32]) -> i32 { data.iter().sum() }
pub fn sum_loop(data: &[i32]) -> i32 { let mut s = 0; for &x in data { s += x; } s }
```

Both produce similar assembly because Rust iterators are **zero-cost abstractions**. The iterator chain is desugared, inlined, and optimized by LLVM to the same loop-and-accumulate pattern.

---

## Section C

### A5
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | Shows the Rust code after all macros are expanded |
| 2 | **False** | It shows **assembly** (x86, ARM, etc.), not LLVM IR |
| 3 | **True** | Uses unstable compiler flags only available on nightly |
| 4 | **False** | With optimization, they typically produce **identical** assembly |
| 5 | **True** | Godbolt supports Rust with side-by-side source ↔ assembly |
| 6 | **True** | The presence of that call means the bounds check is still there |

---

## 🏆 Lesson 90 Complete!

**Next up:** [Lesson 91 — Cross-Compilation](../lesson_91_cross_compile/lesson_91_cross_compile.md) 🦀
