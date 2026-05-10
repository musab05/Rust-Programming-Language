# ✅ Lesson 85 — Answers: Never Type (!) (AT3)

---

## Section A

### A1 — ✅ Compiles. `x` is `i32`.
The condition is `true`, so `x = 42`. The `else` branch returns `!` (from `panic!`), which coerces to `i32`. Output: `42`.

### A2 — ✅ Compiles (but never runs the assignment).
`foo()` returns `!`, which coerces to `String`. The code compiles, but `foo()` loops forever so the assignment to `x` never actually executes.

---

## Section B

### A3
```rust
fn fatal(msg: &str) -> ! {
    eprintln!("FATAL ERROR: {msg}");
    std::process::exit(1);
}

fn read_config(path: &str) -> String {
    match std::fs::read_to_string(path) {
        Ok(content) => content,             // String
        Err(e) => fatal(&format!("{e}")),    // ! → String
    }
}

fn main() {
    let config = read_config("app.toml");
    println!("{config}");
}
```

### A4
| Expression | Type | Practical Use |
|---|---|---|
| `panic!("msg")` | `!` | Abort on unrecoverable errors |
| `todo!("msg")` | `!` | Placeholder during development |
| `loop { }` | `!` | Server event loops, daemons |
| `std::process::exit(n)` | `!` | Fatal error handlers |
| `continue` | `!` | Filter in loops: `let x = if cond { val } else { continue };` |

---

## Section C

### A5
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | `!` is a subtype of every type — it coerces to anything |
| 2 | **True** | An infinite loop without break never produces a value |
| 3 | **True** | Because `!` coerces to any type |
| 4 | **True** | `continue` jumps to next iteration — never produces a value |
| 5 | **False** | `todo!()` returns `!` (it panics, never completing) |
| 6 | **True** | `exit()` terminates the process — never returns |

---

## 🏆 Lesson 85 Complete!

**Next up:** [Lesson 86 — Dynamically Sized Types](../lesson_86_dst/lesson_86_dst.md) 🦀
