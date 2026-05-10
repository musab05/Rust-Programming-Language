# 🧪 Lesson 78 — Questions: RAII & Guard Types (DP5)

> **Lesson:** [lesson_78_raii_guards.md](./lesson_78_raii_guards.md)  
> **Answers:** [lesson_78_answers.md](./lesson_78_answers.md)

---

## Section A — Predict the Output

### Q1
```rust
struct G(i32);
impl Drop for G { fn drop(&mut self) { println!("Drop {}", self.0); } }
fn main() { let _a = G(1); let _b = G(2); let _c = G(3); }
```

---

## Section B — Write It Yourself

### Q2 — ScopeGuard (Roadmap Practice Task)
Implement a `ScopeGuard<F: FnOnce()>` with `new()`, `disarm()`, and `Drop`. Demonstrate it running cleanup and being disarmed.

### Q3 — Timer guard
Create a `Timer` struct that records `Instant::now()` on creation and prints elapsed time on drop. Use it to time a code block.

### Q4 — Transaction guard
Build a `Transaction` with `execute()`, `commit()`, and auto-rollback on drop if not committed.

---

## Section C — True or False?

### Q5
1. Rust drops values in reverse order of creation.
2. `MutexGuard` releases the lock when it goes out of scope.
3. You can call `.drop()` explicitly on any value.
4. `drop(val)` drops a value before the end of its scope.
5. RAII guarantees resources are released even if a panic occurs.
6. `ManuallyDrop` prevents the compiler from calling `Drop::drop`.

---

*RAII: deterministic cleanup, zero leaks! 🦀*
