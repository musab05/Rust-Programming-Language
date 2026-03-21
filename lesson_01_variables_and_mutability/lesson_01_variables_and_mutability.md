# 📘 Lesson 01 — Variables & Mutability in Rust

> **Series:** Learning Rust from Scratch  
> **Difficulty:** Beginner  
> **Files in this lesson:**
> - [`lesson_01_variables_and_mutability.md`](./lesson_01_variables_and_mutability.md) ← You are here
> - [`lesson_01_questions.md`](./lesson_01_questions.md) — Practice problems
> - [`lesson_01_answers.md`](./lesson_01_answers.md) — Solutions & explanations

---

## Table of Contents

1. [What is a Variable?](#1-what-is-a-variable)
2. [Declaring Variables with `let`](#2-declaring-variables-with-let)
3. [Immutability by Default](#3-immutability-by-default)
4. [Making Variables Mutable with `mut`](#4-making-variables-mutable-with-mut)
5. [Why Immutability by Default?](#5-why-immutability-by-default)
6. [Constants (`const`)](#6-constants-const)
7. [Shadowing](#7-shadowing)
8. [Shadowing vs Mutability — Key Differences](#8-shadowing-vs-mutability--key-differences)
9. [Type Inference vs Explicit Types](#9-type-inference-vs-explicit-types)
10. [Variable Scope](#10-variable-scope)
11. [Unused Variables — Compiler Warnings](#11-unused-variables--compiler-warnings)
12. [Summary Cheat Sheet](#12-summary-cheat-sheet)

---

## 1. What is a Variable?

A **variable** is a named storage location in memory that holds a value. In Rust, every variable has:

- A **name** (identifier)
- A **type** (what kind of data it stores)
- A **value** (the actual data)
- A **scope** (where in the code it is accessible)

```rust
let age = 25; // name: age | type: i32 (inferred) | value: 25
```

Rust is a **statically typed** language, meaning the type of every variable must be known at **compile time**. However, Rust's powerful **type inference** often means you don't need to write the type yourself — the compiler figures it out.

---

## 2. Declaring Variables with `let`

In Rust, you declare a variable using the `let` keyword:

```rust
fn main() {
    let x = 5;
    println!("The value of x is: {x}");
}
```

**Syntax breakdown:**
```
let  <name>  :  <type>  =  <value>  ;
 ↑      ↑         ↑          ↑
keyword name   optional    required
              type hint
```

You can optionally annotate the type explicitly:

```rust
let x: i32 = 5;       // explicitly i32
let name: &str = "Rustacean"; // explicitly a string slice
let pi: f64 = 3.14159;
```

Or let Rust infer it:

```rust
let x = 5;       // Rust infers i32
let name = "Rustacean"; // Rust infers &str
let pi = 3.14159; // Rust infers f64
```

---

## 3. Immutability by Default

Here is the **single most important concept** in this lesson:

> **In Rust, variables are immutable by default.**

This means once you assign a value to a variable, you **cannot change it** unless you explicitly opt into mutability.

```rust
fn main() {
    let x = 5;
    x = 6; // ❌ COMPILE ERROR!
}
```

**The compiler error:**
```
error[E0384]: cannot assign twice to immutable variable `x`
 --> src/main.rs:3:5
  |
2 |     let x = 5;
  |         -
  |         |
  |         first assignment to `x`
  |         help: consider making this binding mutable: `mut x`
3 |     x = 6;
  |     ^^^^^ cannot assign twice to immutable variable
```

Notice how Rust's error messages are incredibly helpful — it even tells you **exactly how to fix it** (`mut x`).

---

## 4. Making Variables Mutable with `mut`

To allow a variable to change its value, use the `mut` keyword:

```rust
fn main() {
    let mut x = 5;
    println!("x is: {x}");

    x = 6; // ✅ This is fine now
    println!("x is now: {x}");
}
```

**Output:**
```
x is: 5
x is now: 6
```

`mut` signals to both the compiler and other developers: *"This value is intentionally designed to change."*

### Mutability example — a counter:

```rust
fn main() {
    let mut count = 0;

    count += 1;
    count += 1;
    count += 1;

    println!("Final count: {count}"); // Final count: 3
}
```

### ⚠️ Important: `mut` doesn't change the type

You can change the **value** of a mutable variable, but NOT its **type**:

```rust
fn main() {
    let mut x = 5;
    x = 10;      // ✅ Same type (i32)
    x = "hello"; // ❌ COMPILE ERROR — can't change type!
}
```

---

## 5. Why Immutability by Default?

This design decision might feel unusual coming from other languages. Here's why Rust does it:

### 🔒 Safety
Immutable values **cannot be accidentally changed**. In large programs, this prevents entire classes of bugs where a value is unexpectedly modified far away from where it was defined.

### 🔄 Concurrency
When multiple threads share data, mutable shared state is a common source of **race conditions**. Immutability makes concurrent code much safer.

### 🧠 Clarity
When you see `mut`, you immediately know: *"this value is going to change somewhere."* When you don't see `mut`, you can trust the value never changes. This makes code easier to reason about.

### ⚡ Optimization
The compiler can make more aggressive optimizations when it knows values won't change.

---

## 6. Constants (`const`)

Constants are similar to immutable variables but have key differences:

```rust
const MAX_SCORE: u32 = 100_000;
const PI: f64 = 3.14159265358979;
```

### Differences between `let` and `const`:

| Feature | `let` | `const` |
|---|---|---|
| Keyword | `let` | `const` |
| Mutable? | Yes (with `mut`) | **Never** |
| Type annotation | Optional | **Required** |
| Where declared | Inside functions | Anywhere (global or local) |
| Value | Runtime or compile-time | **Must be compile-time** |
| Naming convention | `snake_case` | `SCREAMING_SNAKE_CASE` |
| Shadowing | Yes | No |

### Example — using constants:

```rust
const SECONDS_PER_MINUTE: u32 = 60;
const MINUTES_PER_HOUR: u32 = 60;
const HOURS_PER_DAY: u32 = 24;

fn main() {
    let seconds_in_a_day = SECONDS_PER_MINUTE * MINUTES_PER_HOUR * HOURS_PER_DAY;
    println!("Seconds in a day: {seconds_in_a_day}"); // 86400
}
```

### When to use `const` vs `let`:

- Use `const` for **fixed values that never change** and are meaningful throughout your program (like limits, math constants, config values)
- Use `let` for **local variables** inside functions

---

## 7. Shadowing

**Shadowing** is when you declare a new variable with the **same name** as an existing variable. The new variable *shadows* (hides) the old one:

```rust
fn main() {
    let x = 5;

    let x = x + 1; // shadows the previous x (x is now 6)

    {
        let x = x * 2; // shadows x in this inner scope (x is 12 here)
        println!("Inner x: {x}"); // Inner x: 12
    }

    println!("Outer x: {x}"); // Outer x: 6
}
```

Each `let x = ...` creates a **brand new variable** — it just happens to have the same name.

### Shadowing with a type change

This is a powerful feature — unlike `mut`, shadowing **allows you to change the type**:

```rust
fn main() {
    // First, spaces is a string of spaces (from user input simulation)
    let spaces = "   ";
    println!("spaces as &str: '{spaces}'");

    // Now we shadow it with a number — the length of the string
    let spaces = spaces.len();
    println!("spaces as usize: {spaces}"); // spaces as usize: 3
}
```

This is idiomatic Rust — instead of naming things `spaces_str` and `spaces_count`, you can shadow and reuse the name as the concept transforms.

Compare with `mut` — this would **fail**:

```rust
fn main() {
    let mut spaces = "   ";
    spaces = spaces.len(); // ❌ ERROR: expected &str, found usize
}
```

---

## 8. Shadowing vs Mutability — Key Differences

This is a common point of confusion. Here's a clear breakdown:

```rust
fn main() {
    // --- MUTATION ---
    let mut x = 5;
    x = 10;         // Same variable, new value, SAME type required
    println!("{x}"); // 10

    // --- SHADOWING ---
    let y = 5;
    let y = y + 1;  // NEW variable named y, old y is gone
    let y = y * 2;  // Another NEW variable
    println!("{y}"); // 12
}
```

| | Mutation (`mut`) | Shadowing (`let`) |
|---|---|---|
| Same variable? | ✅ Yes | ❌ No, new variable |
| Can change type? | ❌ No | ✅ Yes |
| Memory | Same memory location | New memory location |
| Old value | Overwritten | Inaccessible (but still in memory until scope ends) |
| Use case | Values that genuinely need to change | Transforming or reinterpreting a value |

---

## 9. Type Inference vs Explicit Types

Rust can often figure out the type from context:

```rust
fn main() {
    let x = 42;          // inferred as i32
    let y = 42u64;       // explicitly u64 via suffix
    let z: u8 = 42;      // explicitly u8 via annotation
    let w = 3.14;        // inferred as f64
    let flag = true;     // inferred as bool
    let letter = 'R';    // inferred as char
}
```

### When you MUST annotate the type

Sometimes the compiler can't figure it out — usually when there are multiple possible types:

```rust
fn main() {
    // Parsing a string into a number — must annotate!
    let guess: u32 = "42".parse().expect("Not a number!");
    
    // Without annotation:
    // let guess = "42".parse().expect("Not a number!"); // ❌ ERROR
    // error[E0283]: type annotations needed
}
```

### Common numeric types in Rust:

| Type | Description | Range |
|------|-------------|-------|
| `i8` | 8-bit signed | -128 to 127 |
| `i32` | 32-bit signed (default int) | -2.1B to 2.1B |
| `i64` | 64-bit signed | Very large |
| `u8` | 8-bit unsigned | 0 to 255 |
| `u32` | 32-bit unsigned | 0 to 4.3B |
| `u64` | 64-bit unsigned | Very large |
| `f32` | 32-bit float | ~7 decimal digits |
| `f64` | 64-bit float (default float) | ~15 decimal digits |
| `usize` | Platform-size unsigned | Used for indexing |

---

## 10. Variable Scope

A variable's **scope** is the region of code where it is valid and accessible. In Rust, scope is defined by **curly braces `{}`**:

```rust
fn main() {
    // `x` does not exist here yet

    let x = 10; // x comes into scope

    {
        let y = 20; // y comes into scope
        println!("x = {x}, y = {y}"); // both accessible
    } // y goes out of scope and is DROPPED here

    println!("x = {x}"); // ✅ x still accessible
    // println!("y = {y}"); // ❌ ERROR: y is not in scope
}
```

When a variable goes out of scope, Rust automatically calls its **drop** function and frees the memory. This is the foundation of Rust's ownership system (a more advanced topic).

### Scope in practice:

```rust
fn main() {
    let result;

    {
        let a = 10;
        let b = 20;
        result = a + b; // result is defined in outer scope, assigned here
    }
    // a and b are dropped here, but result persists

    println!("Result: {result}"); // ✅ Works! result is 30
}
```

---

## 11. Unused Variables — Compiler Warnings

Rust will **warn you** about variables you declare but never use:

```rust
fn main() {
    let x = 5; // ⚠️ warning: unused variable `x`
}
```

**Warning:**
```
warning: unused variable: `x`
 --> src/main.rs:2:9
  |
2 |     let x = 5;
  |         ^ help: if this is intentional, prefix it with an underscore: `_x`
```

To silence this warning for an intentionally unused variable, prefix the name with an underscore `_`:

```rust
fn main() {
    let _x = 5; // ✅ No warning — the underscore signals "intentionally unused"
    let _ = 5;  // ✅ `_` alone is a special "discard" pattern (value is dropped immediately)
}
```

---

## 12. Summary Cheat Sheet

```rust
// ──────────────────────────────────────────────
// BASIC VARIABLE DECLARATION
// ──────────────────────────────────────────────
let x = 5;              // immutable, type inferred
let y: i32 = 5;         // immutable, type explicit

// ──────────────────────────────────────────────
// MUTABLE VARIABLES
// ──────────────────────────────────────────────
let mut count = 0;      // mutable
count += 1;             // ✅ allowed

// ──────────────────────────────────────────────
// CONSTANTS
// ──────────────────────────────────────────────
const MAX: u32 = 100;   // type annotation REQUIRED
                        // value must be compile-time known

// ──────────────────────────────────────────────
// SHADOWING
// ──────────────────────────────────────────────
let x = 5;
let x = x + 1;          // shadows previous x
let x = "hello";        // can even change the type!

// ──────────────────────────────────────────────
// SCOPE
// ──────────────────────────────────────────────
{
    let inner = 42;     // only lives inside these braces
}
// inner is dropped here

// ──────────────────────────────────────────────
// UNUSED VARIABLE SUPPRESSION
// ──────────────────────────────────────────────
let _unused = 99;       // no compiler warning
```

---

## 🔗 What's Next?

After mastering variables and mutability, the next logical topics are:

- **Lesson 02 — Data Types - Scalar**
- **Lesson 03 — Data Types - Compund**
- **Lesson 04 — Functions**
- **Lesson 05 — Control Flow**

---

## 📚 Further Reading

- [The Rust Book — Chapter 3.1: Variables and Mutability](https://doc.rust-lang.org/book/ch03-01-variables-and-mutability.html)
- [Rust by Example — Variable Bindings](https://doc.rust-lang.org/rust-by-example/variable_bindings.html)
- [Rustlings Exercises](https://github.com/rust-lang/rustlings) — interactive exercises

---

*Happy coding, Rustacean! 🦀*
