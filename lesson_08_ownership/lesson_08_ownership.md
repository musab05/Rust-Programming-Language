# 📘 Lesson 08 — Ownership Rules in Rust

> **Series:** Learning Rust from Scratch
> **Difficulty:** Intermediate (most important Rust concept)
> **Prerequisite:** [Lesson 07 — Comments & Documentation](../lesson_07/lesson_07_comments.md)
>
> **Files in this lesson:**
> - [`lesson_08_ownership.md`](./lesson_08_ownership.md) ← You are here
> - [`lesson_08_questions.md`](./lesson_08_questions.md) — Practice problems
> - [`lesson_08_answers.md`](./lesson_08_answers.md) — Solutions & explanations

---

## Table of Contents

1. [The Memory Problem Rust Solves](#1-the-memory-problem-rust-solves)
2. [The Stack and The Heap](#2-the-stack-and-the-heap)
3. [Ownership — The Three Rules](#3-ownership--the-three-rules)
4. [Rule 1 — Each Value Has One Owner](#4-rule-1--each-value-has-one-owner)
5. [Rule 2 — Only One Owner at a Time](#5-rule-2--only-one-owner-at-a-time)
6. [Rule 3 — Value is Dropped When Owner Goes Out of Scope](#6-rule-3--value-is-dropped-when-owner-goes-out-scope)
7. [Scope and Lifetime of a Variable](#7-scope-and-lifetime-of-a-variable)
8. [The `drop` Function](#8-the-drop-function)
9. [Ownership and Functions](#9-ownership-and-functions)
10. [Ownership and Return Values](#10-ownership-and-return-values)
11. [Why Ownership Matters — What It Prevents](#11-why-ownership-matters--what-it-prevents)
12. [Common Ownership Errors and How to Fix Them](#12-common-ownership-errors-and-how-to-fix-them)
13. [Summary Cheat Sheet](#13-summary-cheat-sheet)

---

## 1. The Memory Problem Rust Solves

Every program uses memory. The two hardest problems in systems programming are:

**1. Memory leaks** — allocating memory and never freeing it (the program grows without bound).
**2. Use-after-free / double-free** — freeing memory and then using it (or freeing it twice), causing crashes or security vulnerabilities.

Languages solve this differently:

| Approach | Languages | Trade-offs |
|----------|-----------|------------|
| **Manual memory management** | C, C++ | Fast, but programmers make mistakes |
| **Garbage collector (GC)** | Java, Python, Go | Safe, but runtime pauses, less control |
| **Ownership system** | **Rust** | Safe AND fast — compiler enforces rules |

Rust's **ownership system** is a set of rules the compiler checks at **compile time**. If you break a rule, the code doesn't compile. If it compiles, it's memory-safe — no GC needed at runtime.

---

## 2. The Stack and The Heap

Understanding where data lives in memory is essential to understanding ownership.

### The Stack

The **stack** is fast, ordered memory that operates like a stack of plates:
- Data is added (pushed) and removed (popped) in **Last In, First Out** order
- All data on the stack must have a **known, fixed size** at compile time
- Allocation and deallocation are **instant** — just move a pointer

```
Stack:
┌─────────────┐
│  frame: fn  │  ← current function's local variables
│  x: i32 = 5│
│  y: bool    │
├─────────────┤
│  frame: main│  ← caller's frame
│  ...        │
└─────────────┘
```

Types that live on the stack: `i32`, `f64`, `bool`, `char`, fixed-size arrays, tuples of stack types.

### The Heap

The **heap** is a large pool of memory for data whose size isn't known at compile time or needs to grow:
- Allocation requires **finding a free block**, recording it — slower
- Must be **explicitly freed** (in Rust, ownership handles this)
- Data can be any size and can grow

```
Heap (unordered):
┌────────────────────────────────────────┐
│  [String data "hello world"]           │
│  [Vec data 1,2,3,4,5]                  │
│  [free space]                          │
│  [another allocation...]               │
└────────────────────────────────────────┘
```

Types that live on the heap: `String`, `Vec<T>`, `Box<T>`, `HashMap<K,V>`.

**Why does this matter for ownership?**

Stack data is trivially copied — it's just bytes. Heap data involves a **pointer** on the stack pointing to data on the heap. When you "copy" heap data, do you copy the pointer (shallow) or the data (deep)? Ownership rules make this explicit and safe.

---

## 3. Ownership — The Three Rules

Rust's ownership system is built on exactly **three rules**:

```
╔══════════════════════════════════════════════════════════╗
║  OWNERSHIP RULES                                         ║
║                                                          ║
║  1. Each value in Rust has exactly one owner.            ║
║                                                          ║
║  2. There can only be one owner at a time.               ║
║                                                          ║
║  3. When the owner goes out of scope, the value          ║
║     is dropped (memory is freed).                        ║
╚══════════════════════════════════════════════════════════╝
```

These three rules, enforced at compile time, eliminate entire categories of memory bugs. Let's examine each in detail.

---

## 4. Rule 1 — Each Value Has One Owner

Every value has exactly **one** binding (variable) that owns it at any moment:

```rust
fn main() {
    let s = String::from("hello"); // s OWNS the String "hello"
    //  ^
    //  This variable is the owner
    //  Exactly one owner — always
}
```

When `s` is created, it becomes the sole owner of the `String` heap allocation.

---

## 5. Rule 2 — Only One Owner at a Time

When you assign a heap value to another variable, **ownership moves**. The original variable is no longer valid:

```rust
fn main() {
    let s1 = String::from("hello"); // s1 owns the String
    let s2 = s1;                    // ownership MOVES to s2

    // s1 is now invalid — Rust moved ownership to s2
    println!("{s1}"); // ❌ COMPILE ERROR: value borrowed after move
    println!("{s2}"); // ✅ Fine — s2 is the owner now
}
```

**Why does Rust do this?**

If both `s1` and `s2` pointed to the same heap data, and both tried to free it when they went out of scope, you'd have a **double-free** — a serious memory safety bug. By making `s1` invalid after the move, only `s2` can free the memory.

```
Before move:          After let s2 = s1:
s1 → [heap: "hello"]  s1 → [INVALID]
                       s2 → [heap: "hello"]
```

This is called a **move** in Rust. We cover moves vs copies in depth in Lesson 09.

---

## 6. Rule 3 — Value is Dropped When Owner Goes Out of Scope

When an owner goes out of scope (reaches the closing `}`), Rust automatically calls `drop` on the value, freeing its memory:

```rust
fn main() {
    {
        let s = String::from("hello"); // s comes into scope
        // s is valid here
        println!("{s}");
    } // ← s goes out of scope HERE → String is dropped, heap memory freed

    // s is not accessible here — and the memory is already freed
}
```

This happens **automatically**, with no garbage collector and no explicit `free()` call. The compiler knows exactly when each variable goes out of scope and inserts the cleanup code.

---

## 7. Scope and Lifetime of a Variable

A variable is valid from the point it is declared until the end of the scope (block) it's in:

```rust
fn main() {                        // scope begins
    // `s` does not exist yet

    let s = String::from("hello"); // s comes into scope
    //                             // s is valid from here...

    println!("{s}");               // use s

}                                  // scope ends → s is dropped
```

Scopes nest — inner blocks create inner scopes:

```rust
fn main() {
    let outer = String::from("outer");   // owned by main scope

    {
        let inner = String::from("inner"); // owned by this inner scope
        println!("{outer}");               // ✅ outer visible here
        println!("{inner}");               // ✅ inner visible here
    }   // inner is dropped HERE

    println!("{outer}");  // ✅ outer still alive
    // println!("{inner}");  // ❌ inner already dropped
}   // outer is dropped HERE
```

**Shadowing creates a new variable in the same scope:**

```rust
fn main() {
    let s = String::from("first");
    let s = String::from("second"); // shadows the first s
    // first String is dropped HERE (when shadowed)
    println!("{s}"); // "second"
}   // second String is dropped here
```

---

## 8. The `drop` Function

When a value goes out of scope, Rust calls a special method called `drop` (defined in the `Drop` trait). For heap-allocated types, `drop` frees the heap memory.

You can implement `drop` for your own types to run custom cleanup:

```rust
struct Resource {
    name: String,
}

impl Drop for Resource {
    fn drop(&mut self) {
        println!("Dropping resource: {}", self.name);
    }
}

fn main() {
    let r1 = Resource { name: String::from("database connection") };
    let r2 = Resource { name: String::from("file handle") };
    println!("Resources created");
    // r2 is dropped first (reverse order of creation), then r1
}
```

**Output:**
```
Resources created
Dropping resource: file handle
Dropping resource: database connection
```

Note: **Drops happen in reverse order** of creation — the last variable created is dropped first (LIFO, like a stack).

**You cannot call `drop` directly** — use `std::mem::drop(value)` if you need early cleanup:

```rust
fn main() {
    let s = String::from("hello");
    drop(s); // explicitly drop s before the scope ends
    // s is now invalid and freed
    println!("Memory already freed");
}
```

---

## 9. Ownership and Functions

Passing a value to a function **moves** ownership into the function (for heap types):

```rust
fn takes_ownership(s: String) {
    println!("{s}");
} // s goes out of scope here → String is dropped

fn makes_copy(n: i32) {
    println!("{n}");
} // n is a copy — original is unaffected

fn main() {
    let s = String::from("hello");
    takes_ownership(s);         // ownership MOVES into the function
    // println!("{s}");         // ❌ ERROR: s was moved, it's invalid here

    let x = 5;
    makes_copy(x);              // i32 is Copy — x is still valid
    println!("{x}");            // ✅ Fine — x was copied, not moved
}
```

**The key insight:** after `takes_ownership(s)`, the String is the function's responsibility. When the function returns, the String is dropped. The caller no longer has access.

```
main's stack:       function's stack:
  s: [ptr → heap]    s: [ptr → heap]   ← ownership transferred
                       (same heap data)
  After return:
  s: [INVALID]       String: DROPPED
```

---

## 10. Ownership and Return Values

Functions can **return ownership** back to the caller:

```rust
fn gives_ownership() -> String {
    let s = String::from("yours now");
    s  // s is returned — ownership moves to the caller
}

fn takes_and_gives_back(s: String) -> String {
    println!("Got: {s}");
    s  // return it — ownership moves back to caller
}

fn main() {
    let s1 = gives_ownership();        // s1 owns the new String
    println!("{s1}");                   // "yours now"

    let s2 = String::from("hello");
    let s3 = takes_and_gives_back(s2); // s2 moves in, s3 gets it back
    // s2 is invalid now
    println!("{s3}");                   // "hello"
}
```

**Returning multiple values using tuples** (common pattern before references):

```rust
fn process(s: String) -> (String, usize) {
    let length = s.len();
    (s, length) // return both the String (ownership) and the length
}

fn main() {
    let s = String::from("hello world");
    let (s, len) = process(s);  // get ownership back + the length
    println!("'{s}' has {len} characters");
}
```

> This is verbose. The better solution is **borrowing** (covered in Lesson 10) — you can give a function read access to data without transferring ownership.

---

## 11. Why Ownership Matters — What It Prevents

### Prevents Use-After-Free

```rust
// In C — dangerous:
// char* s = malloc(6);
// free(s);
// printf("%s", s); // ← use-after-free: undefined behavior!

// In Rust — caught at compile time:
fn main() {
    let s = String::from("hello");
    drop(s);
    println!("{s}"); // ❌ COMPILE ERROR — caught before the program runs
}
```

### Prevents Double-Free

```rust
// In C — dangerous:
// char* a = malloc(6);
// char* b = a;
// free(a);
// free(b); // ← double-free: corruption!

// In Rust — prevented by the move semantics:
fn main() {
    let a = String::from("hello");
    let b = a;              // ownership moves to b
    // a is now invalid — only b can free the memory
}   // only b's drop runs — no double-free possible
```

### Prevents Memory Leaks

```rust
fn main() {
    for _ in 0..1_000_000 {
        let s = String::from("temporary"); // allocated each iteration
    }   // dropped here — memory freed. No leak!
}
```

Without Rust's ownership (and with a GC), you'd rely on the GC to collect. With manual C memory management, forgetting to `free` would leak all those allocations.

---

## 12. Common Ownership Errors and How to Fix Them

### Error 1 — Use after move

```rust
// ❌ Error
let s1 = String::from("hello");
let s2 = s1;
println!("{s1}"); // error: value borrowed after move

// ✅ Fix A: Clone the data (expensive — copies heap data)
let s1 = String::from("hello");
let s2 = s1.clone(); // deep copy — s1 and s2 own separate copies
println!("{s1}"); // ✅ s1 is still valid
println!("{s2}"); // ✅ s2 has its own copy

// ✅ Fix B: Use a reference (borrow — covered in Lesson 10)
let s1 = String::from("hello");
let s2 = &s1; // borrow s1 — s1 still owns the data
println!("{s1}"); // ✅ s1 still owns it
println!("{s2}"); // ✅ s2 just borrows it
```

### Error 2 — Moving out of a function parameter

```rust
// ❌ Problem: take ownership then try to use after
fn print_and_use(s: String) {
    println!("{s}");
} // s dropped here

fn main() {
    let name = String::from("Alice");
    print_and_use(name);
    println!("{name}"); // ❌ name was moved into print_and_use
}

// ✅ Fix: accept a reference instead
fn print_it(s: &String) {   // borrows, doesn't take ownership
    println!("{s}");
}

fn main() {
    let name = String::from("Alice");
    print_it(&name);        // lend a reference
    println!("{name}");     // ✅ still valid
}
```

### Error 3 — Returning a reference to a local variable

```rust
// ❌ Dangling reference — would be caught at compile time
fn make_string() -> &String {    // error: missing lifetime
    let s = String::from("hi");
    &s  // ❌ s will be dropped when function returns — dangling pointer!
}

// ✅ Fix: return the owned String
fn make_string() -> String {
    let s = String::from("hi");
    s  // move ownership to caller — s is NOT dropped here
}
```

---

## 13. Summary Cheat Sheet

```
╔══════════════════════════════════════════════════════╗
║  THE THREE OWNERSHIP RULES                           ║
║  1. Each value has exactly ONE owner                 ║
║  2. Only ONE owner at a time                         ║
║  3. Owner out of scope → value DROPPED               ║
╚══════════════════════════════════════════════════════╝

STACK types (i32, bool, f64, char, fixed arrays):
  → Copied automatically — no ownership transfer
  → No heap allocation — just stack bytes

HEAP types (String, Vec<T>, Box<T>, HashMap...):
  → Assignment MOVES ownership
  → Passing to function MOVES ownership
  → Returning from function MOVES ownership back
  → Going out of scope calls DROP

KEY OPERATIONS:
  let s2 = s1;            → ownership MOVES from s1 to s2
  let s2 = s1.clone();    → deep copy, both valid
  let s2 = &s1;           → borrow (s1 still owns it)
  drop(s);                → explicit early drop
  fn f(s: String)         → takes ownership
  fn f(s: &String)        → borrows (no ownership transfer)
  fn f() -> String        → gives ownership to caller

WHAT OWNERSHIP PREVENTS:
  ✅ No use-after-free
  ✅ No double-free
  ✅ No memory leaks
  ✅ No null pointer dereferences (ownership guarantees validity)
  All checked at COMPILE TIME — zero runtime cost
```

---

## 🔗 What's Next?

- **Lesson 09 — Move vs Copy** (why some types copy and others move)
- **Lesson 10 — References & Borrowing** (use data without transferring ownership)

---

## 📚 Further Reading

- [The Rust Book — Chapter 4.1: What Is Ownership?](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html)
- [Rust by Example — Ownership](https://doc.rust-lang.org/rust-by-example/scope/move.html)
- [Rust Reference — Ownership](https://doc.rust-lang.org/reference/memory-model.html)

---

*"Ownership is the idea that transformed systems programming. Once it clicks, you'll wonder how you ever wrote code without it." 🦀*
