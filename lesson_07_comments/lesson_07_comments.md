# 📘 Lesson 07 — Comments & Documentation in Rust

> **Series:** Learning Rust from Scratch
> **Difficulty:** Beginner
> **Prerequisite:** [Lesson 06 — Loops](../lesson_06/lesson_06_loops.md)
>
> **Files in this lesson:**
> - [`lesson_07_comments.md`](./lesson_07_comments.md) ← You are here
> - [`lesson_07_questions.md`](./lesson_07_questions.md) — Practice problems
> - [`lesson_07_answers.md`](./lesson_07_answers.md) — Solutions & explanations

---

## Table of Contents

1. [Why Comments Matter](#1-why-comments-matter)
2. [Line Comments — `//`](#2-line-comments--)
3. [Block Comments — `/* */`](#3-block-comments----)
4. [Documentation Comments — `///`](#4-documentation-comments---)
   - 4.1 [Basic `///` Syntax](#41-basic--syntax)
   - 4.2 [Standard Documentation Sections](#42-standard-documentation-sections)
   - 4.3 [Markdown in Doc Comments](#43-markdown-in-doc-comments)
   - 4.4 [Doc Tests — Executable Examples](#44-doc-tests--executable-examples)
5. [Inner Doc Comments — `//!`](#5-inner-doc-comments---)
6. [Documenting Different Items](#6-documenting-different-items)
   - 6.1 [Functions](#61-functions)
   - 6.2 [Structs and Fields](#62-structs-and-fields)
   - 6.3 [Enums and Variants](#63-enums-and-variants)
   - 6.4 [Constants and Type Aliases](#64-constants-and-type-aliases)
   - 6.5 [Modules](#65-modules)
7. [`cargo doc` — Generating HTML Documentation](#7-cargo-doc--generating-html-documentation)
8. [Best Practices — What to Comment and What Not To](#8-best-practices--what-to-comment-and-what-not-to)
9. [Summary Cheat Sheet](#9-summary-cheat-sheet)

---

## 1. Why Comments Matter

Code is written once but read many times — by your future self, teammates, and users of your library. Comments bridge the gap between **what** the code does (which the code itself shows) and **why** it does it (which only you know).

```rust
// ❌ Useless comment — just restates the code
let x = x + 1; // add 1 to x

// ✅ Useful comment — explains WHY
let x = x + 1; // offset by 1 because the API is 1-indexed but our array is 0-indexed
```

Rust has **four** types of comments, each with a specific purpose:

| Type | Syntax | Purpose |
|------|--------|---------|
| Line comment | `//` | Inline explanation of code |
| Block comment | `/* */` | Temporarily disable code; multi-line notes |
| Doc comment (outer) | `///` | Documents the item that follows it |
| Doc comment (inner) | `//!` | Documents the enclosing item (module/crate) |

---

## 2. Line Comments — `//`

The most common comment. Everything after `//` on a line is ignored by the compiler.

```rust
fn main() {
    // This is a full-line comment
    let x = 5; // This is an inline comment

    // Multi-line thoughts can use
    // consecutive line comments
    // like this

    let result = x * 2; // double x for the display scale factor
}
```

**Best practices for line comments:**
- Place them **on the line above** or **at the end of** the line they describe
- Keep them **short** — one or two sentences max
- Explain **why**, not **what** (the code shows what)
- Update them when the code changes — outdated comments are worse than none

```rust
fn calculate_tax(amount: f64) -> f64 {
    // Tax rate is 8.5% per regional regulation §42-B
    // This rate is reviewed annually — last updated 2024
    amount * 0.085
}
```

---

## 3. Block Comments — `/* */`

Block comments span multiple lines with `/*` to open and `*/` to close.

```rust
fn main() {
    /* This is a block comment.
       It can span multiple lines.
       The compiler ignores everything inside. */
    let x = 5;

    let y = /* even inline */ 10;

    /*
     * Some developers style them
     * with leading asterisks for readability
     */
}
```

**Main use case — commenting out code temporarily:**

```rust
fn main() {
    let result = compute_v2(42);

    /* Old implementation — keeping for reference
    let result = compute_v1(42);
    if result > 100 {
        log_warning("high value");
    }
    */

    println!("{result}");
}
```

**Block comments can be nested in Rust** (unlike C):

```rust
/* outer comment
    /* inner comment — valid in Rust! */
   still in outer comment
*/
```

> **Tip:** In practice, use `//` for almost everything. Block comments are mainly for temporarily disabling large sections of code. For production code, prefer removing dead code over commenting it out.

---

## 4. Documentation Comments — `///`

`///` is Rust's **doc comment** — it generates HTML documentation from your comments using `cargo doc`. These are not just notes for developers reading the source; they become the official API documentation for your library or application.

### 4.1 Basic `///` Syntax

A `///` comment documents the **item immediately below it**:

```rust
/// Returns the square of a number.
fn square(n: i32) -> i32 {
    n * n
}

/// The maximum number of retry attempts allowed.
const MAX_RETRIES: u32 = 5;

/// A 2D point with floating-point coordinates.
struct Point {
    /// The horizontal coordinate.
    x: f64,
    /// The vertical coordinate.
    y: f64,
}
```

Multiple `///` lines form a single doc comment:

```rust
/// Converts a temperature in Celsius to Fahrenheit.
///
/// Uses the standard conversion formula.
fn to_fahrenheit(celsius: f64) -> f64 {
    celsius * 9.0 / 5.0 + 32.0
}
```

### 4.2 Standard Documentation Sections

The Rust community has standardized doc comment sections using Markdown headings with `#`:

```rust
/// Divides two floating-point numbers safely.
///
/// Returns `None` if the divisor is zero to avoid division by zero.
///
/// # Arguments
///
/// * `dividend` - The number to be divided (numerator).
/// * `divisor` - The number to divide by (denominator).
///
/// # Returns
///
/// `Some(f64)` with the result, or `None` if `divisor` is zero.
///
/// # Examples
///
/// ```
/// let result = safe_divide(10.0, 2.0);
/// assert_eq!(result, Some(5.0));
///
/// let bad = safe_divide(10.0, 0.0);
/// assert_eq!(bad, None);
/// ```
///
/// # Panics
///
/// This function never panics.
///
/// # Errors
///
/// Returns `None` (not an `Err`) for zero divisor.
///
/// # Safety
///
/// This function is safe to call in all contexts.
fn safe_divide(dividend: f64, divisor: f64) -> Option<f64> {
    if divisor == 0.0 {
        None
    } else {
        Some(dividend / divisor)
    }
}
```

**Common sections:**

| Section | Purpose |
|---------|---------|
| First line | Brief one-sentence summary (shown in search results) |
| Rest of first paragraph | Extended description |
| `# Arguments` | Describe each parameter |
| `# Returns` | What the function returns |
| `# Examples` | Code examples (run as tests!) |
| `# Panics` | When/why the function panics |
| `# Errors` | Error conditions for `Result`-returning functions |
| `# Safety` | Required invariants for `unsafe` functions |

### 4.3 Markdown in Doc Comments

Doc comments support full **CommonMark Markdown**:

```rust
/// Processes input data using the following algorithm:
///
/// 1. Parse the raw bytes
/// 2. Validate against the schema
/// 3. Transform to output format
///
/// ## Performance
///
/// This function runs in **O(n)** time and **O(1)** space.
///
/// ## Supported Formats
///
/// | Format | Extension | Supported |
/// |--------|-----------|-----------|
/// | JSON   | `.json`   | ✅ |
/// | XML    | `.xml`    | ✅ |
/// | TOML   | `.toml`   | ✅ |
/// | YAML   | `.yaml`   | ❌ |
///
/// See the [official docs](https://example.com) for more details.
///
/// ```
/// // Example with inline code
/// let data = b"hello";
/// process(data);
/// ```
fn process(data: &[u8]) {
    // implementation
}
```

You can use:
- **Bold**: `**text**`
- *Italic*: `*text*`
- `Inline code`: `` `code` ``
- Code blocks: ` ``` `
- Links: `[text](url)`
- Lists: `- item` or `1. item`
- Tables: `| col | col |`
- Headings: `# H1`, `## H2`

### 4.4 Doc Tests — Executable Examples

The code in `# Examples` sections is **automatically compiled and run** by `cargo test`. This ensures your examples are always correct and up-to-date.

```rust
/// Adds two integers and returns the result.
///
/// # Examples
///
/// ```
/// // This code is compiled and executed by `cargo test`!
/// let sum = add(2, 3);
/// assert_eq!(sum, 5);
///
/// let negative = add(-1, -1);
/// assert_eq!(negative, -2);
/// ```
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

Run doc tests with:
```bash
cargo test --doc
```

**Controlling doc test behavior:**

```rust
/// ```no_run
/// // Compiles but doesn't run (useful for network/IO examples)
/// let result = connect_to_server("localhost:8080");
/// ```

/// ```ignore
/// // Not compiled at all (use when example needs external state)
/// let x = GLOBAL_THING.get_value();
/// ```

/// ```should_panic
/// // Expected to panic — test passes if it does
/// let v: Vec<i32> = Vec::new();
/// let _ = v[0]; // panics!
/// ```
```

---

## 5. Inner Doc Comments — `//!`

`//!` documents the **enclosing item** — the module or crate that the comment is placed inside:

```rust
//! # My Awesome Library
//!
//! This crate provides utilities for processing data efficiently.
//!
//! ## Quick Start
//!
//! ```
//! use my_lib::process;
//! let result = process(42);
//! ```
//!
//! ## Features
//!
//! - Fast processing
//! - Memory-safe
//! - Zero dependencies

// The actual library code follows below...
pub fn process(x: i32) -> i32 {
    x * 2
}
```

`//!` is placed at the **top of a file** (before any items) and documents that entire file/module. It's the crate-level documentation users see on [docs.rs](https://docs.rs).

**Comparison:**

```rust
//! This documents the MODULE itself (must be inside the module)

/// This documents the FUNCTION that follows
fn my_function() {}
```

---

## 6. Documenting Different Items

### 6.1 Functions

```rust
/// Computes the nth Fibonacci number iteratively.
///
/// # Arguments
///
/// * `n` - The index in the Fibonacci sequence (0-indexed).
///         `fibonacci(0)` returns 0, `fibonacci(1)` returns 1.
///
/// # Returns
///
/// The nth Fibonacci number as `u64`.
///
/// # Panics
///
/// Does not panic for any valid `u32` input.
///
/// # Examples
///
/// ```
/// assert_eq!(fibonacci(0), 0);
/// assert_eq!(fibonacci(1), 1);
/// assert_eq!(fibonacci(10), 55);
/// ```
pub fn fibonacci(n: u32) -> u64 {
    if n == 0 { return 0; }
    let (mut a, mut b) = (0u64, 1u64);
    for _ in 1..n {
        (a, b) = (b, a + b);
    }
    b
}
```

### 6.2 Structs and Fields

```rust
/// Represents a player in the game.
///
/// A player has a name, health points, and a score.
/// Health ranges from 0 (dead) to 100 (full health).
///
/// # Examples
///
/// ```
/// let player = Player {
///     name: String::from("Alice"),
///     health: 100,
///     score: 0,
/// };
/// assert!(player.is_alive());
/// ```
pub struct Player {
    /// The player's display name.
    pub name: String,

    /// Current health points. Range: 0–100.
    /// When this reaches 0, the player is considered dead.
    pub health: u32,

    /// The player's current score. Increases with kills and objectives.
    pub score: u64,
}

impl Player {
    /// Returns `true` if the player has health greater than zero.
    pub fn is_alive(&self) -> bool {
        self.health > 0
    }
}
```

### 6.3 Enums and Variants

```rust
/// The cardinal directions of a compass.
///
/// Used for navigation and movement in grid-based systems.
#[derive(Debug, PartialEq)]
pub enum Direction {
    /// Move upward (decreasing row index).
    North,
    /// Move downward (increasing row index).
    South,
    /// Move rightward (increasing column index).
    East,
    /// Move leftward (decreasing column index).
    West,
}

impl Direction {
    /// Returns the opposite direction.
    ///
    /// # Examples
    ///
    /// ```
    /// use my_lib::Direction;
    /// assert_eq!(Direction::North.opposite(), Direction::South);
    /// ```
    pub fn opposite(&self) -> Direction {
        match self {
            Direction::North => Direction::South,
            Direction::South => Direction::North,
            Direction::East  => Direction::West,
            Direction::West  => Direction::East,
        }
    }
}
```

### 6.4 Constants and Type Aliases

```rust
/// The mathematical constant π (pi) to 15 decimal places.
///
/// Use `std::f64::consts::PI` for higher precision in production code.
pub const PI: f64 = 3.141592653589793;

/// The speed of light in meters per second.
///
/// Source: NIST CODATA 2018 recommended values.
pub const SPEED_OF_LIGHT: u64 = 299_792_458;

/// A result type for fallible operations in this crate.
///
/// `E` defaults to our crate's error type for convenience.
pub type Result<T> = std::result::Result<T, String>;

/// Weight measured in kilograms.
///
/// Note: This is a type alias — the compiler won't prevent mixing
/// `Kilograms` and `f64` values. Use a newtype for stronger guarantees.
pub type Kilograms = f64;
```

### 6.5 Modules

```rust
/// Utilities for mathematical operations.
///
/// This module provides common math functions not available
/// in the standard library, with a focus on performance.
pub mod math {
    //! # Math Utilities
    //!
    //! Functions in this module operate on `f64` values unless noted.
    //! All functions handle edge cases gracefully without panicking.

    /// Returns the greatest common divisor of two integers.
    ///
    /// Uses the Euclidean algorithm.
    ///
    /// # Examples
    /// ```
    /// assert_eq!(gcd(48, 18), 6);
    /// ```
    pub fn gcd(mut a: u64, mut b: u64) -> u64 {
        while b != 0 {
            (a, b) = (b, a % b);
        }
        a
    }
}
```

---

## 7. `cargo doc` — Generating HTML Documentation

Rust's toolchain can generate beautiful HTML documentation from your `///` and `//!` comments:

```bash
# Generate docs for your project
cargo doc

# Generate docs AND open them in your browser
cargo doc --open

# Generate docs including private items
cargo doc --document-private-items

# Run doc tests
cargo test --doc
```

The generated docs are placed in `target/doc/`. They include:
- A searchable index of all public items
- Your `///` comments rendered as HTML with syntax-highlighted code blocks
- Cross-links between items
- Doc test results

**Viewing documentation for dependencies:**
```bash
cargo doc --open  # also generates docs for all your dependencies
```

---

## 8. Best Practices — What to Comment and What Not To

### ✅ DO comment:

```rust
// WHY: explain reasoning, constraints, domain knowledge
let price = base * 1.085; // 8.5% tax per regional regulation §42-B

// WHAT non-obvious algorithms do
// Uses Floyd's cycle detection algorithm
let mut slow = head;
let mut fast = head;

// IMPORTANT warnings
// Safety: caller must ensure `ptr` is non-null and aligned
unsafe fn read_unchecked(ptr: *const i32) -> i32 { *ptr }

// TODO/FIXME for future work
// TODO: replace with a proper parser once the spec is finalized
let workaround = input.split(',').next().unwrap_or("");

// Public API — always document public functions
/// Sorts a slice in ascending order using quicksort.
pub fn sort(data: &mut [i32]) { /* ... */ }
```

### ❌ DON'T comment:

```rust
// i = 0 (the code is self-explanatory)
let i = 0;

// increment i
i += 1;

// check if name is empty
if name.is_empty() { ... }

// This function returns the sum of a and b
// (the name add(a, b) already says this)
fn add(a: i32, b: i32) -> i32 { a + b }
```

### Writing Comments That Age Well

```rust
// ❌ Fragile — describes implementation detail that might change
// This uses a hash map internally
fn get_user(id: u32) -> Option<User> { ... }

// ✅ Stable — describes the contract, not implementation
// Returns the user with the given ID, or None if not found.
fn get_user(id: u32) -> Option<User> { ... }

// ❌ Date-based comment — becomes misleading quickly
// Added 2023-01-15 for the new billing feature
fn calculate_invoice() { ... }

// ✅ Use version control (git blame) for history, not comments
// Calculates invoice total including all applicable taxes and fees
fn calculate_invoice() { ... }
```

---

## 9. Summary Cheat Sheet

```rust
// ════════════════════════════════════════════
// LINE COMMENT — explains nearby code
// ════════════════════════════════════════════
// Single line note
let x = 5; // inline note

// ════════════════════════════════════════════
// BLOCK COMMENT — disable code, long notes
// ════════════════════════════════════════════
/* temporarily disabled
   let old = compute_old();
*/
let y = /* inline block */ 10;

// ════════════════════════════════════════════
// DOC COMMENT — documents the next item
// ════════════════════════════════════════════
/// One-sentence summary.
///
/// Detailed description with **markdown** support.
///
/// # Arguments
/// * `x` - description of x
///
/// # Returns
/// What is returned and when.
///
/// # Examples
/// ```
/// let result = my_fn(5);
/// assert_eq!(result, 10);
/// ```
///
/// # Panics
/// When this function panics.
///
/// # Errors
/// Error conditions for Result-returning fns.
pub fn my_fn(x: i32) -> i32 { x * 2 }

// ════════════════════════════════════════════
// INNER DOC COMMENT — documents enclosing module
// ════════════════════════════════════════════
//! # Module Name
//!
//! Description of this module goes here.
//! Placed at the TOP of the file, before any items.

// ════════════════════════════════════════════
// DOC TEST CONTROL
// ════════════════════════════════════════════
/// ```              ← compiled and run
/// ```no_run        ← compiled but not run
/// ```ignore        ← not compiled
/// ```should_panic  ← expected to panic

// ════════════════════════════════════════════
// CARGO COMMANDS
// ════════════════════════════════════════════
// cargo doc          → generate docs
// cargo doc --open   → generate + open browser
// cargo test --doc   → run doc tests
```

---

## 🔗 What's Next?

- **Lesson 08 — Ownership Rules** (Rust's memory model — the most important concept)
- **Lesson 09 — Move vs Copy** (how values transfer between bindings)
- **Lesson 10 — References & Borrowing** (accessing data safely)

---

## 📚 Further Reading

- [The Rust Book — Chapter 14.2: Publishing to Crates.io (documentation)](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html)
- [Rust by Example — Documentation](https://doc.rust-lang.org/rust-by-example/meta/doc.html)
- [rustdoc Book](https://doc.rust-lang.org/rustdoc/)
- [API Guidelines — Documentation](https://rust-lang.github.io/api-guidelines/documentation.html)

---

*"Code tells you how. Comments tell you why. Docs tell others what they need to know." 🦀*
