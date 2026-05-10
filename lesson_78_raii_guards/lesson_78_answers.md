# ✅ Lesson 78 — Answers: RAII & Guard Types (DP5)

---

## Section A

### A1
```
Drop 3
Drop 2
Drop 1
```
Rust drops in **reverse creation order** — last created, first dropped.

---

## Section B

### A2
```rust
struct ScopeGuard<F: FnOnce()> { cb: Option<F> }
impl<F: FnOnce()> ScopeGuard<F> {
    fn new(f: F) -> Self { ScopeGuard { cb: Some(f) } }
    fn disarm(&mut self) { self.cb = None; }
}
impl<F: FnOnce()> Drop for ScopeGuard<F> {
    fn drop(&mut self) { if let Some(f) = self.cb.take() { f(); } }
}

fn main() {
    { let _g = ScopeGuard::new(|| println!("Cleanup ran!")); }
    // Output: Cleanup ran!
    { let mut g = ScopeGuard::new(|| println!("Won't run"));
      g.disarm(); }
    // No output — disarmed
}
```

### A3
```rust
use std::time::Instant;
struct Timer { label: String, start: Instant }
impl Timer {
    fn new(label: &str) -> Self { Timer { label: label.into(), start: Instant::now() } }
}
impl Drop for Timer {
    fn drop(&mut self) { println!("[{}] {:.2?}", self.label, self.start.elapsed()); }
}
fn main() {
    let _t = Timer::new("main");
    std::thread::sleep(std::time::Duration::from_millis(100));
}
```

### A4
```rust
struct Transaction { ops: Vec<String>, committed: bool }
impl Transaction {
    fn new() -> Self { Transaction { ops: vec![], committed: false } }
    fn execute(&mut self, op: &str) { self.ops.push(op.into()); }
    fn commit(mut self) { self.committed = true; println!("Committed"); }
}
impl Drop for Transaction {
    fn drop(&mut self) {
        if !self.committed {
            println!("Rolling back {} ops", self.ops.len());
        }
    }
}
fn main() {
    { let mut tx = Transaction::new(); tx.execute("INSERT"); tx.commit(); }
    { let mut tx = Transaction::new(); tx.execute("INSERT"); }  // auto-rollback
}
```

---

## Section C

### A5
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | LIFO: last variable created is dropped first |
| 2 | **True** | MutexGuard's Drop impl unlocks the mutex |
| 3 | **False** | Can't call `.drop()` directly — use `drop(val)` or `std::mem::drop` |
| 4 | **True** | `drop()` takes ownership and drops immediately |
| 5 | **True** | Drop runs during stack unwinding on panic (unless `panic=abort`) |
| 6 | **True** | ManuallyDrop wraps a value and suppresses automatic Drop |

---

## 🏆 Lesson 78 Complete!

**Next up:** [Lesson 79 — CLI Apps with clap](../lesson_79_clap/lesson_79_clap.md) 🦀
