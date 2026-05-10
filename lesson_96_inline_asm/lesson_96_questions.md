# 🧪 Lesson 96 — Questions & ✅ Answers: Inline Assembly (U5)

> **Lesson:** [lesson_96_inline_asm.md](./lesson_96_inline_asm.md)

---

## Q1 — Why does `asm!` require `unsafe`?
**A:** The compiler cannot verify assembly correctness — wrong instructions can corrupt memory, registers, or crash.

## Q2 — Write RDTSC (Roadmap Practice Task)
```rust
use std::arch::asm;
fn rdtsc() -> u64 {
    let lo: u32; let hi: u32;
    unsafe { asm!("rdtsc", out("eax") lo, out("edx") hi); }
    ((hi as u64) << 32) | lo as u64
}
fn main() { let t1 = rdtsc(); let t2 = rdtsc(); println!("Delta: {}", t2 - t1); }
```

## Q3 — True or False?
| # | Statement | Answer |
|---|-----------|--------|
| 1 | `asm!` can only be used in `unsafe` blocks | **True** |
| 2 | `out(reg)` places a value INTO a register | **False** — it reads FROM a register into a variable |
| 3 | `options(nomem)` means the asm doesn't access memory | **True** |
| 4 | Inline assembly is portable across architectures | **False** — it's platform-specific |

---

**Next:** [Lesson 97 — Proc Macros](../lesson_97_proc_macros/lesson_97_proc_macros.md) 🦀
