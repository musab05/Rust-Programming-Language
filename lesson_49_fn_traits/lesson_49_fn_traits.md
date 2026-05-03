# 📘 Lesson 49 — Fn, FnMut, FnOnce (CL2)

> **Series:** Rust From Zero · Intermediate Level  
> **Roadmap ID:** CL2 · Category: 🔒 Closures  
> **Previous:** [Lesson 48 — Closures: Syntax & Captures](../lesson_48_closures/lesson_48_closures.md)  
> **Next:** [Lesson 50 — Higher-Order Functions](../lesson_50_higher_order/lesson_50_higher_order.md)  
> **Practice:** [Questions](./lesson_49_questions.md) · [Answers](./lesson_49_answers.md)  
> **Practice Task:** Write functions accepting each Fn trait; explain why each is needed

---

## Table of Contents

1. [The Three Closure Traits](#1-the-three-closure-traits)
2. [Fn — Immutable Borrow](#2-fn--immutable-borrow)
3. [FnMut — Mutable Borrow](#3-fnmut--mutable-borrow)
4. [FnOnce — Takes Ownership](#4-fnonce--takes-ownership)
5. [The Trait Hierarchy](#5-the-trait-hierarchy)
6. [Choosing the Right Trait](#6-choosing-the-right-trait)
7. [Storing Closures](#7-storing-closures)
8. [Returning Closures](#8-returning-closures)
9. [Real-World Patterns](#9-real-world-patterns)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. The Three Closure Traits

Every closure in Rust implements one or more of these traits:

```rust
// Simplified definitions:
trait FnOnce<Args> {
    type Output;
    fn call_once(self, args: Args) -> Self::Output;
}

trait FnMut<Args>: FnOnce<Args> {
    fn call_mut(&mut self, args: Args) -> Self::Output;
}

trait Fn<Args>: FnMut<Args> {
    fn call(&self, args: Args) -> Self::Output;
}
```

| Trait | Receiver | Can call | Captures |
|---|---|---|---|
| `Fn` | `&self` | Unlimited | Immutable borrow |
| `FnMut` | `&mut self` | Unlimited | Mutable borrow |
| `FnOnce` | `self` | Once | Takes ownership |

---

## 2. Fn — Immutable Borrow

Closures that only **read** captured variables implement `Fn`:

```rust
fn call_twice(f: impl Fn()) {
    f();
    f();  // can call multiple times — Fn takes &self
}

fn apply<F: Fn(i32) -> i32>(x: i32, f: F) -> i32 {
    f(x)
}

fn main() {
    let message = "Hello";
    call_twice(|| println!("{message}"));  // borrows message by &str

    let factor = 3;
    let result = apply(10, |x| x * factor);  // borrows factor
    println!("{result}");  // 30
}
```

---

## 3. FnMut — Mutable Borrow

Closures that **modify** captured variables implement `FnMut`:

```rust
fn call_n_times(mut f: impl FnMut(), n: usize) {
    for _ in 0..n {
        f();  // needs &mut self
    }
}

fn main() {
    let mut count = 0;
    call_n_times(|| {
        count += 1;
        println!("Count: {count}");
    }, 3);
    // Count: 1
    // Count: 2
    // Count: 3
}
```

### Why `mut f`?

```rust
// FnMut::call_mut takes &mut self
// So we need mut access to the closure itself
fn do_it(mut f: impl FnMut()) {
    f();  // calls f.call_mut() — needs &mut f
}
```

---

## 4. FnOnce — Takes Ownership

Closures that **consume** captured variables implement only `FnOnce`:

```rust
fn call_once(f: impl FnOnce()) {
    f();
    // f();  // ❌ can't call again — f was consumed
}

fn consume_and_report(f: impl FnOnce() -> String) -> String {
    f()  // calls f.call_once() — consumes f
}

fn main() {
    let name = String::from("Alice");

    // This closure moves name — can only be called once
    let greet = || {
        let owned = name;  // takes ownership of name
        format!("Hello, {owned}!")
    };

    let result = consume_and_report(greet);
    println!("{result}");
    // greet();  // ❌ already consumed
}
```

### move closures and FnOnce:

```rust
fn main() {
    let data = vec![1, 2, 3];

    // move takes ownership, but if we only read, it's still Fn
    let read_data = move || println!("{:?}", data);
    read_data();
    read_data();  // ✅ Still Fn — move copies/moves in, but only reads

    let data2 = vec![4, 5, 6];
    // This closure consumes data2 — FnOnce only
    let consume = move || {
        drop(data2);  // consumes data2
    };
    consume();
    // consume();  // ❌ FnOnce — data2 was dropped
}
```

---

## 5. The Trait Hierarchy

```
FnOnce    ← every closure implements this (broadest)
  ↑
FnMut     ← closures that don't consume captures
  ↑
Fn        ← closures that don't mutate captures (narrowest)
```

- Every `Fn` is also `FnMut` and `FnOnce`
- Every `FnMut` is also `FnOnce`
- `FnOnce` is the most general — all closures implement it

```rust
fn takes_fn(f: impl Fn()) { f(); f(); }
fn takes_fn_mut(mut f: impl FnMut()) { f(); f(); }
fn takes_fn_once(f: impl FnOnce()) { f(); }

fn main() {
    let x = 42;
    let print_x = || println!("{x}");  // Fn (only reads x)

    takes_fn(print_x);        // ✅ Fn works everywhere
    takes_fn_mut(print_x);    // ✅ Fn is also FnMut
    takes_fn_once(print_x);   // ✅ Fn is also FnOnce
}
```

---

## 6. Choosing the Right Trait

When writing a function that accepts a closure, use the **most general** trait that works:

```rust
// ✅ Best practice: use the most permissive trait
// FnOnce > FnMut > Fn (from most to least permissive)

// If you only call the closure once → FnOnce
fn run_once(f: impl FnOnce() -> String) -> String { f() }

// If you call it multiple times and it might mutate → FnMut
fn run_many(mut f: impl FnMut() -> i32, n: usize) -> Vec<i32> {
    (0..n).map(|_| f()).collect()
}

// If you call it multiple times and it must NOT mutate → Fn
fn run_pure(f: impl Fn(i32) -> i32, x: i32) -> i32 { f(f(x)) }

fn main() {
    let s = run_once(|| String::from("hello"));
    println!("{s}");

    let mut counter = 0;
    let nums = run_many(|| { counter += 1; counter }, 5);
    println!("{:?}", nums);  // [1, 2, 3, 4, 5]

    let result = run_pure(|x| x * 2, 3);
    println!("{result}");  // 12
}
```

---

## 7. Storing Closures

### In structs with generics:

```rust
struct Callback<F: Fn(i32)> {
    func: F,
}

impl<F: Fn(i32)> Callback<F> {
    fn new(func: F) -> Self { Callback { func } }
    fn call(&self, value: i32) { (self.func)(value); }
}

fn main() {
    let cb = Callback::new(|x| println!("Got: {x}"));
    cb.call(42);
}
```

### In structs with trait objects (dynamic):

```rust
struct EventHandler {
    handlers: Vec<Box<dyn Fn(String)>>,
}

impl EventHandler {
    fn new() -> Self { EventHandler { handlers: vec![] } }

    fn on_event(&mut self, handler: impl Fn(String) + 'static) {
        self.handlers.push(Box::new(handler));
    }

    fn emit(&self, event: String) {
        for handler in &self.handlers {
            handler(event.clone());
        }
    }
}

fn main() {
    let mut eh = EventHandler::new();
    eh.on_event(|e| println!("Handler 1: {e}"));
    eh.on_event(|e| println!("Handler 2: {e}"));
    eh.emit("click".into());
}
```

---

## 8. Returning Closures

Closures have anonymous types — use `impl Fn` or `Box<dyn Fn>`:

```rust
// Using impl Fn (static dispatch)
fn make_adder(x: i32) -> impl Fn(i32) -> i32 {
    move |y| x + y
}

// Using Box<dyn Fn> (dynamic dispatch, needed for different return types)
fn make_operation(op: &str) -> Box<dyn Fn(i32, i32) -> i32> {
    match op {
        "add" => Box::new(|a, b| a + b),
        "mul" => Box::new(|a, b| a * b),
        _ => Box::new(|a, b| a - b),
    }
}

fn main() {
    let add_5 = make_adder(5);
    println!("{}", add_5(10));   // 15
    println!("{}", add_5(20));   // 25

    let op = make_operation("mul");
    println!("{}", op(4, 5));    // 20
}
```

---

## 9. Real-World Patterns

### Builder pattern with closures:

```rust
struct QueryBuilder {
    filters: Vec<Box<dyn Fn(&i32) -> bool>>,
}

impl QueryBuilder {
    fn new() -> Self { QueryBuilder { filters: vec![] } }

    fn where_clause(mut self, filter: impl Fn(&i32) -> bool + 'static) -> Self {
        self.filters.push(Box::new(filter));
        self
    }

    fn execute(&self, data: &[i32]) -> Vec<&i32> {
        data.iter()
            .filter(|item| self.filters.iter().all(|f| f(item)))
            .collect()
    }
}

fn main() {
    let data = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

    let results = QueryBuilder::new()
        .where_clause(|x| *x > 3)
        .where_clause(|x| *x < 8)
        .where_clause(|x| x % 2 == 0)
        .execute(&data);

    println!("{:?}", results);  // [4, 6]
}
```

---

## 10. Summary Cheat Sheet

```
THE THREE TRAITS
────────────────────────────────────────────────────────────
Fn        &self       reads captures     call many times
FnMut     &mut self   modifies captures  call many times
FnOnce    self        consumes captures  call once

HIERARCHY: Fn ⊂ FnMut ⊂ FnOnce
────────────────────────────────────────────────────────────
Every Fn is FnMut, every FnMut is FnOnce.

CHOOSING (as parameter)
────────────────────────────────────────────────────────────
Use FnOnce  → call once, maximum flexibility
Use FnMut   → call multiple times, allow mutation
Use Fn      → call multiple times, require purity

STORING CLOSURES
────────────────────────────────────────────────────────────
Struct<F: Fn(T)>            generic (static dispatch)
Box<dyn Fn(T)>              trait object (dynamic dispatch)

RETURNING CLOSURES
────────────────────────────────────────────────────────────
-> impl Fn(T) -> U          one concrete type
-> Box<dyn Fn(T) -> U>      multiple possible types
```

---

## What's Next?

**Lesson 50 — Higher-Order Functions** — Functions that take functions and return functions. Master `map`, `filter`, `fold`, composition, and functional programming patterns in Rust.

## Further Reading
- [The Rust Book — Ch 13.1: Closures](https://doc.rust-lang.org/book/ch13-01-closures.html)
- [Rust Reference — Closure Types](https://doc.rust-lang.org/reference/types/closure.html)

---

*Fn, FnMut, FnOnce: the closure contract system! 🦀*
