# 📘 Lesson 78 — RAII & Guard Types (DP5)

> **Series:** Rust From Zero · Intermediate Level (Gap Fill)  
> **Roadmap ID:** DP5 · Category: 🏗 Design Patterns  
> **Previous:** [Lesson 77 — CI/CD with GitHub Actions](../lesson_77_ci_cd/lesson_77_ci_cd.md)  
> **Next:** [Lesson 79 — CLI Apps with clap](../lesson_79_clap/lesson_79_clap.md)  
> **Practice:** [Questions](./lesson_78_questions.md) · [Answers](./lesson_78_answers.md)  
> **Practice Task:** Implement a ScopeGuard that runs code on drop

---

## Table of Contents

1. [What Is RAII?](#1-what-is-raii)
2. [The Drop Trait](#2-the-drop-trait)
3. [Guard Types in std](#3-guard-types-in-std)
4. [Building a ScopeGuard](#4-building-a-scopeguard)
5. [File Handle RAII](#5-file-handle-raii)
6. [Timer Guard](#6-timer-guard)
7. [Transaction Guard](#7-transaction-guard)
8. [std::mem::drop and ManuallyDrop](#8-stdmemdrop-and-manuallydrop)
9. [Summary Cheat Sheet](#9-summary-cheat-sheet)

---

## 1. What Is RAII?

**Resource Acquisition Is Initialization** — tie resource lifetimes to object lifetimes:

```
┌─────────────────────────────────────────┐
│  Object created → resource acquired     │
│  Object used    → resource available    │
│  Object dropped → resource released     │
└─────────────────────────────────────────┘
```

```rust
fn main() {
    {
        let file = std::fs::File::create("temp.txt").unwrap();
        // file handle acquired
        // ... use file ...
    } // file goes out of scope → Drop::drop() called → file handle closed

    {
        let data = vec![1, 2, 3, 4, 5];
        // heap memory allocated
    } // Vec dropped → heap memory freed

    {
        let s = String::from("hello");
        // heap buffer allocated
    } // String dropped → heap buffer freed
}
```

**No manual cleanup needed** — Rust guarantees cleanup via Drop.

---

## 2. The Drop Trait

```rust
struct DatabaseConnection {
    url: String,
    connected: bool,
}

impl DatabaseConnection {
    fn new(url: &str) -> Self {
        println!("🔌 Connecting to {url}...");
        DatabaseConnection { url: url.to_string(), connected: true }
    }

    fn query(&self, sql: &str) -> Vec<String> {
        println!("  📋 Executing: {sql}");
        vec!["row1".into(), "row2".into()]
    }
}

impl Drop for DatabaseConnection {
    fn drop(&mut self) {
        println!("🔌 Disconnecting from {}...", self.url);
        self.connected = false;
        // In real code: close connection, release pool slot, etc.
    }
}

fn main() {
    println!("=== Start ===");
    {
        let db = DatabaseConnection::new("postgres://localhost/mydb");
        let rows = db.query("SELECT * FROM users");
        println!("  Got {} rows", rows.len());
    } // db.drop() called automatically here
    println!("=== End ===");
}
```

### Drop order:

```rust
struct A(i32);
struct B(i32);
impl Drop for A { fn drop(&mut self) { println!("Drop A({})", self.0); } }
impl Drop for B { fn drop(&mut self) { println!("Drop B({})", self.0); } }

fn main() {
    let a = A(1);
    let b = B(2);
    let c = A(3);
    // Drop order: REVERSE of creation → A(3), B(2), A(1)
}
```

---

## 3. Guard Types in std

### MutexGuard — releases lock on drop:

```rust
use std::sync::Mutex;

fn main() {
    let data = Mutex::new(vec![1, 2, 3]);

    {
        let mut guard = data.lock().unwrap();  // lock acquired
        guard.push(4);
        println!("Data: {:?}", *guard);
        // guard dropped here → lock released automatically
    }

    // Lock is free — another thread could acquire it
    println!("After scope: {:?}", data.lock().unwrap());
}
```

### Other std guard types:

| Guard | Acquires | Releases on Drop |
|---|---|---|
| `MutexGuard` | Mutex lock | Unlocks mutex |
| `RwLockReadGuard` | Read lock | Releases read lock |
| `RwLockWriteGuard` | Write lock | Releases write lock |
| `Ref` / `RefMut` | RefCell borrow | Releases borrow |
| `JoinHandle` | Thread | Detaches thread |

---

## 4. Building a ScopeGuard

Run arbitrary cleanup code when a scope exits:

```rust
struct ScopeGuard<F: FnOnce()> {
    callback: Option<F>,
}

impl<F: FnOnce()> ScopeGuard<F> {
    fn new(callback: F) -> Self {
        ScopeGuard { callback: Some(callback) }
    }

    /// Disarm the guard — cleanup will NOT run
    fn disarm(&mut self) {
        self.callback = None;
    }
}

impl<F: FnOnce()> Drop for ScopeGuard<F> {
    fn drop(&mut self) {
        if let Some(cb) = self.callback.take() {
            cb();
        }
    }
}

/// Helper function
fn on_scope_exit<F: FnOnce()>(f: F) -> ScopeGuard<F> {
    ScopeGuard::new(f)
}

fn main() {
    println!("=== Scope 1: Normal exit ===");
    {
        let _guard = on_scope_exit(|| println!("  Cleanup: scope 1 exited"));
        println!("  Working in scope 1...");
    }

    println!("\n=== Scope 2: Disarmed ===");
    {
        let mut guard = on_scope_exit(|| println!("  Cleanup: scope 2 exited"));
        println!("  Working in scope 2...");
        guard.disarm();  // prevent cleanup
        println!("  Guard disarmed — no cleanup will run");
    }

    println!("\n=== Scope 3: Temp file cleanup ===");
    {
        let path = "temp_work.txt";
        std::fs::write(path, "temporary data").unwrap();
        let _guard = on_scope_exit(move || {
            std::fs::remove_file(path).ok();
            println!("  Cleaned up {path}");
        });
        println!("  File exists: {}", std::path::Path::new(path).exists());
        // _guard drops → file deleted
    }
}
```

---

## 5. File Handle RAII

```rust
use std::io::{Write, BufWriter};
use std::fs::File;

struct LogFile {
    writer: BufWriter<File>,
    path: String,
    entries: usize,
}

impl LogFile {
    fn new(path: &str) -> std::io::Result<Self> {
        let file = File::create(path)?;
        println!("📝 Log opened: {path}");
        Ok(LogFile {
            writer: BufWriter::new(file),
            path: path.to_string(),
            entries: 0,
        })
    }

    fn log(&mut self, msg: &str) -> std::io::Result<()> {
        writeln!(self.writer, "[{}] {msg}", self.entries)?;
        self.entries += 1;
        Ok(())
    }
}

impl Drop for LogFile {
    fn drop(&mut self) {
        self.writer.flush().ok();
        println!("📝 Log closed: {} ({} entries)", self.path, self.entries);
    }
}

fn main() {
    let mut log = LogFile::new("app.log").unwrap();
    log.log("Application started").unwrap();
    log.log("Processing data").unwrap();
    log.log("Done").unwrap();
    // log.drop() called → buffer flushed, file closed
}
```

---

## 6. Timer Guard

```rust
use std::time::Instant;

struct Timer {
    label: String,
    start: Instant,
}

impl Timer {
    fn new(label: &str) -> Self {
        println!("⏱  [{label}] started");
        Timer { label: label.to_string(), start: Instant::now() }
    }
}

impl Drop for Timer {
    fn drop(&mut self) {
        let elapsed = self.start.elapsed();
        println!("⏱  [{}] completed in {:.2?}", self.label, elapsed);
    }
}

fn expensive_work() {
    let _timer = Timer::new("expensive_work");
    // simulate work
    let mut sum = 0u64;
    for i in 0..10_000_000 { sum += i; }
    println!("   Result: {sum}");
}

fn main() {
    let _total = Timer::new("total");
    expensive_work();
    {
        let _t = Timer::new("inner scope");
        std::thread::sleep(std::time::Duration::from_millis(50));
    }
}
```

---

## 7. Transaction Guard

```rust
struct Transaction {
    operations: Vec<String>,
    committed: bool,
}

impl Transaction {
    fn new() -> Self {
        println!("🔄 Transaction started");
        Transaction { operations: vec![], committed: false }
    }

    fn execute(&mut self, op: &str) {
        println!("  → {op}");
        self.operations.push(op.to_string());
    }

    fn commit(mut self) {
        println!("✅ Committed {} operations", self.operations.len());
        self.committed = true;
        // self is dropped after this, but committed=true prevents rollback
    }
}

impl Drop for Transaction {
    fn drop(&mut self) {
        if !self.committed {
            println!("⚠️  Rolling back {} operations!", self.operations.len());
            for op in self.operations.iter().rev() {
                println!("  ← UNDO: {op}");
            }
        }
    }
}

fn main() {
    // Committed transaction
    println!("=== Successful transaction ===");
    {
        let mut tx = Transaction::new();
        tx.execute("INSERT user Alice");
        tx.execute("UPDATE balance +100");
        tx.commit();
    }

    // Uncommitted transaction (simulating error)
    println!("\n=== Failed transaction ===");
    {
        let mut tx = Transaction::new();
        tx.execute("INSERT user Bob");
        tx.execute("UPDATE balance +200");
        // oops, no commit! → automatic rollback on drop
    }
}
```

---

## 8. std::mem::drop and ManuallyDrop

```rust
fn main() {
    let s = String::from("hello");

    // Drop early (before end of scope)
    drop(s);  // s is dropped NOW
    // println!("{s}");  // ❌ use after drop

    // ManuallyDrop — prevent automatic drop
    use std::mem::ManuallyDrop;
    let mut manual = ManuallyDrop::new(String::from("I control my destiny"));
    println!("{}", *manual);

    // Must manually drop (or leak memory)
    unsafe { ManuallyDrop::drop(&mut manual); }
}
```

---

## 9. Summary Cheat Sheet

```
RAII PRINCIPLE
────────────────────────────────────────────────────────────
Create object → acquire resource
Drop object   → release resource (automatic!)

DROP TRAIT
────────────────────────────────────────────────────────────
impl Drop for T { fn drop(&mut self) { cleanup(); } }
Drop order: reverse of creation order

GUARD PATTERN
────────────────────────────────────────────────────────────
Guard type holds a resource
Deref to access the resource
Drop releases the resource
Examples: MutexGuard, Ref, RefMut

SCOPE GUARD
────────────────────────────────────────────────────────────
Run cleanup on scope exit (success or panic)
Use .disarm() to cancel cleanup

EARLY DROP
────────────────────────────────────────────────────────────
drop(value)              drop before end of scope
ManuallyDrop::new(val)   prevent automatic drop
```

---

## What's Next?

**Lesson 79 — CLI Apps with clap** — Build full-featured command-line tools with argument parsing, subcommands, and validation.

## Further Reading
- [Rust Design Patterns — RAII Guards](https://rust-unofficial.github.io/patterns/patterns/behavioural/RAII.html)
- [std::ops::Drop](https://doc.rust-lang.org/std/ops/trait.Drop.html)

---

*RAII: acquire on create, release on drop — no leaks ever! 🦀*
