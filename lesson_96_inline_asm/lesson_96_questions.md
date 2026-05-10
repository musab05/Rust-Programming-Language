# 🧪 Lesson 96 — Questions: Inline Assembly (U5)

> **Lesson:** [lesson_96_inline_asm.md](./lesson_96_inline_asm.md)  
> **Answers:** [lesson_96_answers.md](./lesson_96_answers.md)

---

## Section A — Conceptual

### Q1
Why does `asm!` require an `unsafe` block? What specific guarantees can the compiler NOT verify?

### Q2
Explain the difference between `in(reg)`, `out(reg)`, and `inout(reg)` operand types. When would you use each?

### Q3
What happens if you forget to declare a clobbered register or use the wrong `options(...)` flag?

---

## Section B — Write It Yourself

### Q4 — RDTSC timestamp (Roadmap Practice Task)
Write a function `rdtsc() -> u64` that reads the CPU timestamp counter using the `rdtsc` instruction. Split the result from `eax` (low 32 bits) and `edx` (high 32 bits) and combine them. Use it to measure the cycle cost of a `nop`.

### Q5 — CPUID vendor string
Write a function `cpu_vendor() -> [u8; 12]` that calls `cpuid` with `eax = 0` and extracts the vendor string from `ebx`, `edx`, `ecx` (note the non-obvious ordering).

### Q6 — Safe wrapper
Write a safe public function `add_asm(a: u64, b: u64) -> u64` that uses `asm!` internally with the `add` instruction. The function signature should be safe even though the implementation uses `unsafe`.

---

## Section C — True or False?

### Q7
1. `asm!` can be used in safe Rust without an `unsafe` block.
2. `out(reg) val` writes a Rust variable's value INTO a CPU register.
3. `options(nomem)` promises the assembly doesn't read or write memory.
4. Inline assembly is portable across x86 and ARM without changes.
5. `options(pure, nomem)` lets the compiler optimize away redundant calls.
6. You can use named registers like `in("rax")` for specific register constraints.

### Q8
Explain what `options(preserves_flags)` does and when you would use it. What could go wrong if you claim `preserves_flags` but your assembly actually modifies the flags register?

---

*When you need the metal! 🦀*
