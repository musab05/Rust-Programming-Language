# 📘 Lesson 09 — Move vs Copy in Rust

> **Series:** Learning Rust from Scratch
> **Difficulty:** Intermediate
> **Prerequisite:** [Lesson 08 — Ownership Rules](../lesson_08_ownership.md)
>
> **Files in this lesson:**
> - [`lesson_09_move_copy.md`](./lesson_09_move_copy.md) ← You are here
> - [`lesson_09_questions.md`](./lesson_09_questions.md) — Practice problems
> - [`lesson_09_answers.md`](./lesson_09_answers.md) — Solutions

---

## Table of Contents

1. [The Core Distinction](#1-the-core-distinction)
2. [The `Copy` Trait — Cheap Bitwise Duplication](#2-the-copy-trait--cheap-bitwise-duplication)
   - 2.1 [Which Types Are Copy?](#21-which-types-are-copy)
   - 2.2 [Deriving Copy](#22-deriving-copy)
   - 2.3 [Why Not Make Everything Copy?](#23-why-not-make-everything-copy)
3. [Move Semantics — Transfer of Ownership](#3-move-semantics--transfer-of-ownership)
   - 3.1 [Move on Assignment](#31-move-on-assignment)
   - 3.2 [Move into Functions](#32-move-into-functions)
   - 3.3 [Move out of Functions](#33-move-out-of-functions)
   - 3.4 [Partial Moves](#34-partial-moves)
4. [`.clone()` — Explicit Deep Copy](#4-clone--explicit-deep-copy)
   - 4.1 [When to Clone](#41-when-to-clone)
   - 4.2 [Clone vs Copy](#42-clone-vs-copy)
   - 4.3 [Deriving Clone](#43-deriving-clone)
5. [The `Copy` + `Clone` Relationship](#5-the-copy--clone-relationship)
6. [`std::mem::copy` and `std::mem::swap`](#6-stdmemcopy-and-stdmemswap)
7. [Move Semantics in Match and Destructuring](#7-move-semantics-in-match-and-destructuring)
8. [Practical Decision Guide](#8-practical-decision-guide)
9. [Summary Cheat Sheet](#9-summary-cheat-sheet)

---

## 1. The Core Distinction

When you assign a value or pass it to a function, one of two things happens:

```
Copy  → a bitwise duplicate is made; BOTH variables are valid
Move  → ownership transfers; the ORIGINAL variable becomes invalid
```

```rust
fn main() {
    // Copy — both valid after assignment
    let a: i32 = 42;
    let b = a;
    println!("{a}, {b}"); // ✅ both work

    // Move — original invalid after assignment
    let s1 = String::from("hello");
    let s2 = s1;
    // println!("{s1}"); // ❌ s1 moved
    println!("{s2}");    // ✅
}
```

The difference is entirely determined by whether the type implements the `Copy` trait.

---

## 2. The `Copy` Trait — Cheap Bitwise Duplication

A type implements `Copy` when it is safe and cheap to duplicate it byte-by-byte. Copying means both variables end up with independent, identical data — no sharing, no ownership conflict.

### 2.1 Which Types Are Copy?

**Always Copy:**
- All integer types: `i8`, `i16`, `i32`, `i64`, `i128`, `isize`, `u8`, `u16`, `u32`, `u64`, `u128`, `usize`
- Both float types: `f32`, `f64`
- `bool`
- `char`
- The unit type: `()`
- Raw pointers: `*const T`, `*mut T`

**Copy if their elements/fields are Copy:**
- Tuples: `(i32, f64)` is Copy; `(i32, String)` is NOT
- Arrays: `[i32; 5]` is Copy; `[String; 5]` is NOT
- References: `&T` is always Copy (but `&mut T` is NOT)

**Never Copy (own heap memory):**
- `String`
- `Vec<T>`
- `Box<T>`
- `HashMap<K, V>`
- `File`, `TcpStream`, and most I/O types

```rust
fn main() {
    // All of these COPY — both variables valid after assignment
    let a: i32   = 1;    let _b = a;   println!("{a}");
    let c: f64   = 1.0;  let _d = c;   println!("{c}");
    let e: bool  = true; let _f = e;   println!("{e}");
    let g: char  = 'x';  let _h = g;   println!("{g}");
    let i: (i32, f64) = (1, 2.0); let _j = i; println!("{i:?}");
    let k: [u8; 3] = [1,2,3]; let _l = k; println!("{k:?}");
    let m: &str = "hi"; let _n = m;  println!("{m}"); // &str is Copy!
}
```

### 2.2 Deriving Copy

For your own types, you can opt into `Copy` with `#[derive(Copy, Clone)]`:

```rust
#[derive(Debug, Copy, Clone)]
struct Point {
    x: f64,
    y: f64,
}

fn main() {
    let p1 = Point { x: 1.0, y: 2.0 };
    let p2 = p1; // COPY — p1 still valid
    println!("{p1:?}"); // ✅
    println!("{p2:?}"); // ✅
}
```

**Rules for deriving Copy:**
1. All fields must implement `Copy`
2. `Clone` must also be implemented (Copy requires Clone)
3. The type must not implement `Drop` (a type managing resources shouldn't be cheaply copyable)

### 2.3 Why Not Make Everything Copy?

For types like `String` or `Vec`, copying would mean:
- Allocating new heap memory
- Copying potentially megabytes of data
- Tracking two independent owners

This would be **silently expensive** — a seemingly innocent `let b = a` could copy gigabytes. Rust separates cheap implicit copies (`Copy`) from expensive explicit copies (`.clone()`).

---

## 3. Move Semantics — Transfer of Ownership

### 3.1 Move on Assignment

```rust
fn main() {
    let s1 = String::from("hello");

    // This is a MOVE, not a copy:
    // - s1's stack data (ptr, len, cap) is copied to s2
    // - BUT the heap allocation is not duplicated
    // - s1 is marked as invalid
    let s2 = s1;

    // Rust tracks this at compile time — no runtime cost
    // println!("{s1}"); // ❌ compile error
    println!("{s2}");   // ✅
}
```

Move is not literally erasing data — the bits still exist in memory briefly. But Rust **statically prevents** you from using `s1` again, so it will never double-free the heap memory.

### 3.2 Move into Functions

Passing a non-Copy type to a function moves ownership:

```rust
fn process(v: Vec<i32>) -> usize {
    v.len()
} // v is dropped here (unless returned)

fn main() {
    let data = vec![1, 2, 3, 4, 5];
    let count = process(data);    // data MOVED into process
    // println!("{data:?}");      // ❌ data was moved
    println!("count: {count}");
}
```

For Copy types, the function receives a copy — original is fine:

```rust
fn square(n: i32) -> i32 { n * n }

fn main() {
    let x = 5;
    let result = square(x); // x is COPIED into square
    println!("{x}");        // ✅ x still valid
    println!("{result}");
}
```

### 3.3 Move out of Functions

A function can transfer ownership back to the caller by returning:

```rust
fn create() -> String {
    let s = String::from("created");
    s // ownership moves OUT to caller
}

fn transform(mut s: String) -> String {
    s.push_str(" transformed");
    s // return to pass ownership back
}

fn main() {
    let s1 = create();            // s1 owns the String
    let s2 = transform(s1);       // s1 moves in, s2 gets it back
    println!("{s2}");             // created transformed
}
```

### 3.4 Partial Moves

You can move one field out of a struct, but then the whole struct (or the moved field) cannot be used:

```rust
#[derive(Debug)]
struct Person {
    name: String,
    age: u32,
}

fn main() {
    let person = Person { name: String::from("Alice"), age: 30 };

    let name = person.name;   // name MOVED out of person
    // println!("{person:?}");// ❌ person partially moved
    println!("{name}");       // ✅ name is its own owner now
    println!("{}", person.age); // ✅ age is Copy — still accessible
}
```

---

## 4. `.clone()` — Explicit Deep Copy

`.clone()` creates an independent deep copy of a value — both the original and the clone are fully owned, independent values:

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1.clone(); // creates a NEW heap allocation with same content

    println!("{s1}"); // ✅ s1 owns its copy
    println!("{s2}"); // ✅ s2 owns its copy

    // Modifying one doesn't affect the other:
    let mut s3 = s1.clone();
    s3.push_str(" world");
    println!("{s1}"); // hello — unchanged
    println!("{s3}"); // hello world
}
```

### 4.1 When to Clone

Use `.clone()` when:
- You need two independent owners of the same data
- You can't use references (next lesson)
- Performance is not critical

Avoid `.clone()` when:
- You're cloning just to work around a compiler error — usually a sign you need a reference instead
- The data is very large — cloning copies all bytes
- You're in a tight performance-sensitive loop

```rust
// ❌ Cloning unnecessarily
fn get_length(s: String) -> usize {
    s.len()
}
fn main() {
    let s = String::from("hello");
    let len = get_length(s.clone()); // wasteful clone
    println!("{s}: {len}");
}

// ✅ Use a reference instead
fn get_length(s: &String) -> usize { s.len() }
fn main() {
    let s = String::from("hello");
    let len = get_length(&s); // no clone needed
    println!("{s}: {len}");
}
```

### 4.2 Clone vs Copy

| Aspect | Copy | Clone |
|--------|------|-------|
| How triggered | Implicit (automatic) | Explicit (`.clone()`) |
| Cost | Cheap (stack bitwise copy) | Can be expensive (deep heap copy) |
| Visibility | Silent | Visible in source code |
| Requirement | All fields must be Copy | Any type can implement it |

### 4.3 Deriving Clone

```rust
#[derive(Debug, Clone)]
struct Config {
    host: String,
    port: u16,
    debug: bool,
}

fn main() {
    let cfg1 = Config { host: String::from("localhost"), port: 8080, debug: true };
    let cfg2 = cfg1.clone(); // deep copy — both have independent Strings
    println!("{cfg1:?}");
    println!("{cfg2:?}");
}
```

---

## 5. The `Copy` + `Clone` Relationship

Every `Copy` type must also implement `Clone`. This makes sense: if you can cheaply copy bits, you can certainly clone.

```rust
// Copy: copy is implicit when assigning
// Clone: .clone() is explicit — calls Copy's bitwise duplication for Copy types

let x: i32 = 5;
let y = x;          // Copy: implicit
let z = x.clone();  // Clone: explicit — for i32, same as copy
println!("{x} {y} {z}");
```

A type can implement `Clone` without `Copy`. But a `Copy` type MUST implement `Clone`. The compiler enforces this.

---

## 6. `std::mem::copy` and `std::mem::swap`

```rust
fn main() {
    // mem::swap — exchange values of two mutable variables
    let mut a = String::from("first");
    let mut b = String::from("second");
    std::mem::swap(&mut a, &mut b);
    println!("{a}"); // second
    println!("{b}"); // first

    // mem::replace — replace a value and return the old one
    let old = std::mem::replace(&mut a, String::from("third"));
    println!("{a}");   // third
    println!("{old}"); // second

    // mem::take — move out of a mutable reference (replaces with Default)
    let mut s = String::from("hello");
    let taken = std::mem::take(&mut s);
    println!("{s}");     // "" (empty — default for String)
    println!("{taken}"); // hello
}
```

These utilities are useful when you need to move values out of mutable references — a common pattern in more advanced Rust code.

---

## 7. Move Semantics in Match and Destructuring

```rust
#[derive(Debug)]
struct Wrapper(String);

fn main() {
    let w = Wrapper(String::from("hello"));

    // Match moves out of w (unless you use ref)
    match w {
        Wrapper(s) => println!("Got: {s}"), // s owns the String
    }
    // println!("{w:?}"); // ❌ w was moved into the match

    // Use ref to borrow instead of move
    let w2 = Wrapper(String::from("world"));
    match &w2 {
        Wrapper(s) => println!("Got: {s}"), // s is &String
    }
    println!("{w2:?}"); // ✅ w2 not moved

    // Destructuring also moves
    let pair = (String::from("a"), String::from("b"));
    let (first, second) = pair; // both moved out of pair
    println!("{first} {second}"); // ✅
    // println!("{pair:?}"); // ❌ pair moved
}
```

---

## 8. Practical Decision Guide

```
When assigning or passing a value, ask:

Does the type implement Copy?
├── YES → implicit copy; original still valid; no action needed
└── NO (move type) →
    Do you need the original after this point?
    ├── NO → let it move; most efficient
    └── YES →
        Is a reference sufficient?
        ├── YES → pass &value or &mut value (next lesson)
        └── NO (need ownership) → .clone() — but consider if design is right
```

---

## 9. Summary Cheat Sheet

```rust
// ── COPY TYPES (implicit, cheap, silent) ───────
let a: i32 = 5;
let b = a;        // COPY — both a and b valid
println!("{a}");  // ✅

// Copy types: i8..i128, u8..u128, f32, f64,
//             bool, char, (), &T, *const T, *mut T
//             tuples/arrays if ALL elements are Copy

// ── MOVE TYPES (explicit in code, transfers ownership) ─
let s1 = String::from("hi");
let s2 = s1;       // MOVE — s1 invalid
// s1 invalid now   // ❌
println!("{s2}");  // ✅

// Move types: String, Vec<T>, Box<T>, HashMap, etc.

// ── CLONE (explicit deep copy) ──────────────────
let s3 = s2.clone();   // new heap allocation
println!("{s2}");      // ✅ still valid
println!("{s3}");      // ✅ independent copy

// ── DERIVE Copy/Clone ────────────────────────────
#[derive(Debug, Copy, Clone)]  // opt-in for your types
struct MyPoint { x: i32, y: i32 } // only if all fields are Copy

#[derive(Debug, Clone)]         // Clone without Copy
struct Config { host: String }  // has non-Copy field

// ── mem utilities ────────────────────────────────
std::mem::swap(&mut a, &mut b);   // exchange values
std::mem::replace(&mut a, new);   // replace, return old
std::mem::take(&mut a);           // move out, leave Default
```

---

## 📚 Further Reading

- [The Rust Book — Chapter 4.1: What is Ownership?](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html)
- [Rust Reference — Copy types](https://doc.rust-lang.org/reference/special-types-and-traits.html#copy)
- [std::clone::Clone](https://doc.rust-lang.org/std/clone/trait.Clone.html)
- [std::marker::Copy](https://doc.rust-lang.org/std/marker/trait.Copy.html)

---

*"Copy is automatic silence. Clone is deliberate speech. Move is honest transfer." 🦀*
