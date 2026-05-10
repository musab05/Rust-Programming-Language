# ✅ Lesson 96 — Answers: Inline Assembly (U5)

---

## Section A

### A1
`asm!` requires `unsafe` because the compiler cannot verify:
- **Register correctness** — wrong registers can corrupt program state
- **Memory safety** — assembly can dereference arbitrary pointers
- **Undefined behavior** — incorrect instructions can crash, corrupt the stack, or violate ABI
- **Clobber correctness** — if you forget to declare modified registers, the compiler's register allocator will produce wrong code

The Rust compiler can verify Rust code's safety guarantees, but raw assembly is a black box.

### A2
| Operand | Direction | Usage |
|---------|-----------|-------|
| `in(reg) val` | Rust → CPU | Passes a Rust value INTO a register as input. The register is read-only. |
| `out(reg) val` | CPU → Rust | Reads a value FROM a register into a Rust variable. The register's initial value is undefined. |
| `inout(reg) val` | Both | The register starts with a Rust value and the assembly modifies it in-place. Use `inout(reg) a => b` to separate input/output variables. |

Use `in` when you only need to feed data to the assembly. Use `out` when you only need results. Use `inout` when the instruction modifies a register in place (e.g., `add` modifies the destination operand).

### A3
**Undeclared clobbers:** The compiler assumes the register is unchanged and may rely on its value — leading to silent data corruption or crashes. For example, if your assembly modifies `rcx` but you don't declare it, the compiler might have stored a local variable there.

**Wrong options:** If you claim `nomem` but the assembly reads memory, the compiler may reorder memory operations around your assembly block, causing reads of stale data or writes that get overwritten.

---

## Section B

### A4
```rust
use std::arch::asm;

fn rdtsc() -> u64 {
    let lo: u32;
    let hi: u32;
    unsafe {
        asm!(
            "rdtsc",
            out("eax") lo,
            out("edx") hi,
            options(nomem, nostack, preserves_flags),
        );
    }
    ((hi as u64) << 32) | (lo as u64)
}

fn main() {
    let t1 = rdtsc();
    unsafe { asm!("nop"); }
    let t2 = rdtsc();
    println!("nop cost: {} cycles", t2 - t1);

    // Measure a small loop
    let t1 = rdtsc();
    for _ in 0..1000 {
        unsafe { asm!("nop"); }
    }
    let t2 = rdtsc();
    println!("1000 nops: {} cycles ({:.1} per nop)", t2 - t1, (t2 - t1) as f64 / 1000.0);
}
```

### A5
```rust
use std::arch::asm;

fn cpu_vendor() -> [u8; 12] {
    let ebx: u32;
    let ecx: u32;
    let edx: u32;
    unsafe {
        asm!(
            "cpuid",
            inout("eax") 0u32 => _, // leaf 0, discard eax result
            out("ebx") ebx,
            out("ecx") ecx,
            out("edx") edx,
        );
    }
    // Vendor string order: EBX, EDX, ECX (not EBX, ECX, EDX!)
    let mut result = [0u8; 12];
    result[0..4].copy_from_slice(&ebx.to_le_bytes());
    result[4..8].copy_from_slice(&edx.to_le_bytes());
    result[8..12].copy_from_slice(&ecx.to_le_bytes());
    result
}

fn main() {
    let vendor = cpu_vendor();
    let vendor_str = core::str::from_utf8(&vendor).unwrap_or("???");
    println!("CPU Vendor: {vendor_str}");
    // Typically: "GenuineIntel" or "AuthenticAMD"
}
```

### A6
```rust
use std::arch::asm;

/// Adds two u64 values using inline assembly.
/// Safe wrapper around unsafe asm! — the invariants are trivially upheld
/// because `add` on two general-purpose registers cannot violate memory safety.
pub fn add_asm(a: u64, b: u64) -> u64 {
    let result: u64;
    unsafe {
        asm!(
            "add {0}, {1}",
            inout(reg) a => result,
            in(reg) b,
            options(nomem, nostack, pure),
        );
    }
    result
}

fn main() {
    assert_eq!(add_asm(3, 4), 7);
    assert_eq!(add_asm(100, 200), 300);
    assert_eq!(add_asm(0, 0), 0);
    println!("All add_asm tests passed!");
}
```
The function is marked safe because we maintain all invariants: the `add` instruction cannot corrupt memory, we correctly declare all operands, and `options(nomem, nostack, pure)` is truthful.

---

## Section C

### A7
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **False** | `asm!` always requires `unsafe` — the compiler can't verify assembly |
| 2 | **False** | `out(reg)` reads FROM a register into a Rust variable (output) |
| 3 | **True** | It tells the compiler this assembly doesn't touch memory |
| 4 | **False** | Inline assembly is architecture-specific; x86 and ARM have different instructions |
| 5 | **True** | `pure` + `nomem` means no side effects, so identical calls can be deduplicated |
| 6 | **True** | Named register constraints like `in("rax")` force a specific register |

### A8
`options(preserves_flags)` tells the compiler that the assembly does NOT modify the CPU flags register (carry, zero, overflow, etc.). The compiler can then keep condition codes across the `asm!` block instead of re-evaluating conditions.

**If you lie about it:** The compiler may rely on flag values from BEFORE the assembly block. If your assembly actually modifies flags (e.g., `add` sets the carry flag), subsequent conditional code could branch on stale flag values — causing silent logic errors that are extremely hard to debug.

---

## 🏆 Lesson 96 Complete!

**Next up:** [Lesson 97 — Attribute & Function-like Macros](../lesson_97_proc_macros/lesson_97_proc_macros.md) 🦀
