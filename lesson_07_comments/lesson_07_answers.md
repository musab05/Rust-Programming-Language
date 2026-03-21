# ✅ Lesson 07 — Answers: Comments & Documentation

> Questions are in [`lesson_07_questions.md`](./lesson_07_questions.md)

---

## Section A — Identify and Analyze

### A1 — Comment Types

| Comment | Type | Purpose |
|---------|------|---------|
| `//! # Auth Module ...` | `//!` inner doc comment | Documents the enclosing module itself — appears at the top of the file/module |
| `/// Hashes a plaintext password...` | `///` outer doc comment | Documents the `hash_password` function that immediately follows |
| `// TODO: integrate real bcrypt library` | `//` line comment | Developer note for future work — compiler ignores it |
| `/* placeholder — returns input for now */` | `/* */` block comment | Inline note explaining temporary behavior |

---

### A2 — Good vs Bad Comments

| | Verdict | Reason |
|--|---------|--------|
| A | ❌ **Bad** | Restates the code — anyone can see it adds 1 |
| B | ✅ **Good** | Explains WHY — domain knowledge about the legacy API |
| C | ❌ **Bad** | Restates the code — the loop is self-evident |
| D | ✅ **Good** | Explains the business rule (15%, seniors 65+) — not obvious from `0.85` |
| E | ❌ **Bad** | The doc comment says nothing the function name doesn't already say |
| F | ✅ **Good** | Clarifies the nuance: active AND not suspended |

**Rewritten bad comments:**
```rust
// A: Remove the comment entirely — code is self-explanatory

// C: Remove the comment entirely — or add WHY if there's a reason
// skip items marked as deleted
for item in &cart { process(item); }

// E — proper doc comment:
/// Returns `true` if the user account is active and not suspended.
fn is_active(user: &User) -> bool { user.active }
```

---

### A3 — Doc Comment Structure

**Problems:**
1. Uses `//` line comments instead of `///` doc comments — these won't appear in `cargo doc`
2. Arguments described in plain prose instead of the `# Arguments` section with `*` bullets
3. No blank lines between sections (needed for Markdown paragraphs)
4. Example is not in a code block — won't be compiled/run as a doc test
5. Missing a `# Returns` section
6. Summary line style not idiomatic

**Corrected version:**
```rust
/// Adds two integers and returns their sum.
///
/// # Arguments
///
/// * `a` - The first addend.
/// * `b` - The second addend.
///
/// # Returns
///
/// The sum of `a` and `b` as `i32`.
///
/// # Examples
///
/// ```
/// assert_eq!(add(2, 3), 5);
/// assert_eq!(add(-1, 1), 0);
/// assert_eq!(add(0, 0), 0);
/// ```
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

---

### A4 — Doc Test Analysis

| Block | Result | Reason |
|-------|--------|--------|
| A | ✅ Compiles and passes | Valid Rust, assertion is correct |
| B | ❌ Compiles but panics at runtime | `1 / 0` causes integer division-by-zero panic |
| C | ✅ Passes | Marked `should_panic` — the out-of-bounds access panics as expected |
| D | ✅ Compiles, not run | `no_run` — compiled for syntax check only, never executed |
| E | ✅ Ignored completely | `ignore` — not even compiled |

---

## Section B — Write the Documentation

### A5 — Document `clamp`

```rust
/// Clamps a value to the inclusive range `[min, max]`.
///
/// If `value` is less than `min`, returns `min`.
/// If `value` is greater than `max`, returns `max`.
/// Otherwise, returns `value` unchanged.
///
/// # Arguments
///
/// * `value` - The value to clamp.
/// * `min` - The lower bound of the range (inclusive).
/// * `max` - The upper bound of the range (inclusive).
///
/// # Returns
///
/// The clamped value within `[min, max]`.
///
/// # Examples
///
/// ```
/// // Value below the range
/// assert_eq!(clamp(-5.0, 0.0, 10.0), 0.0);
///
/// // Value above the range
/// assert_eq!(clamp(15.0, 0.0, 10.0), 10.0);
///
/// // Value within the range — unchanged
/// assert_eq!(clamp(7.0, 0.0, 10.0), 7.0);
///
/// // Value at the boundary — unchanged
/// assert_eq!(clamp(0.0, 0.0, 10.0), 0.0);
/// ```
///
/// # Panics
///
/// This function does not panic. However, if `min > max`, the result
/// is unspecified — either `min` or `max` may be returned depending
/// on `value`. Consider validating `min <= max` before calling.
pub fn clamp(value: f64, min: f64, max: f64) -> f64 {
    if value < min { min }
    else if value > max { max }
    else { value }
}
```

---

### A6 — Document `BankAccount`

```rust
/// A bank account with an owner, balance, and freeze status.
///
/// Accounts start unfrozen. A frozen account rejects all transactions.
/// The balance is private and can only be modified through `deposit`
/// and `withdraw`.
///
/// # Examples
///
/// ```
/// let mut account = BankAccount {
///     owner: String::from("Alice"),
///     balance: 0.0,     // Note: normally you'd use a constructor
///     is_frozen: false,
/// };
/// account.deposit(100.0).unwrap();
/// account.withdraw(30.0).unwrap();
/// ```
pub struct BankAccount {
    /// The full name of the account owner.
    pub owner: String,

    /// The current account balance in the account's currency.
    /// Always non-negative for valid accounts.
    balance: f64,

    /// Whether the account is currently frozen.
    /// Frozen accounts reject all deposits and withdrawals.
    pub is_frozen: bool,
}

impl BankAccount {
    /// Deposits an amount into the account.
    ///
    /// # Arguments
    ///
    /// * `amount` - The amount to deposit. Must be positive.
    ///
    /// # Returns
    ///
    /// `Ok(new_balance)` on success, or `Err(message)` if:
    /// - The account is frozen
    /// - `amount` is zero or negative
    ///
    /// # Errors
    ///
    /// Returns `Err("Account is frozen")` if `self.is_frozen` is true.
    /// Returns `Err("Amount must be positive")` if `amount <= 0.0`.
    ///
    /// # Examples
    ///
    /// ```
    /// // (abbreviated — in real code use a constructor)
    /// ```
    pub fn deposit(&mut self, amount: f64) -> Result<f64, String> {
        if self.is_frozen { return Err("Account is frozen".to_string()); }
        if amount <= 0.0  { return Err("Amount must be positive".to_string()); }
        self.balance += amount;
        Ok(self.balance)
    }

    /// Withdraws an amount from the account.
    ///
    /// # Arguments
    ///
    /// * `amount` - The amount to withdraw. Must be positive and ≤ balance.
    ///
    /// # Returns
    ///
    /// `Ok(new_balance)` on success, or `Err(message)` on failure.
    ///
    /// # Errors
    ///
    /// - `"Account is frozen"` if the account is frozen
    /// - `"Amount must be positive"` if `amount <= 0.0`
    /// - `"Insufficient funds"` if `amount > self.balance`
    pub fn withdraw(&mut self, amount: f64) -> Result<f64, String> {
        if self.is_frozen     { return Err("Account is frozen".to_string()); }
        if amount <= 0.0      { return Err("Amount must be positive".to_string()); }
        if amount > self.balance { return Err("Insufficient funds".to_string()); }
        self.balance -= amount;
        Ok(self.balance)
    }
}
```

---

### A7 — Document `TaskStatus`

```rust
/// The lifecycle status of a task in the task management system.
///
/// Tasks move through: `Pending` → `InProgress` → `Completed`
/// or can end in `Failed` or `Cancelled`.
#[derive(Debug, PartialEq)]
pub enum TaskStatus {
    /// The task has been created but not yet started.
    Pending,

    /// The task is actively being worked on.
    ///
    /// Contains the current completion percentage (0–100).
    InProgress { progress_pct: u8 },

    /// The task finished successfully.
    Completed,

    /// The task encountered an error and could not finish.
    ///
    /// Contains a human-readable error message describing what went wrong.
    Failed(String),

    /// The task was explicitly cancelled before completion.
    Cancelled,
}

impl TaskStatus {
    /// Returns `true` if this status represents a terminal state.
    ///
    /// Terminal states are those from which no further transitions occur:
    /// `Completed`, `Failed`, and `Cancelled`.
    ///
    /// # Examples
    ///
    /// ```
    /// assert!(TaskStatus::Completed.is_terminal());
    /// assert!(TaskStatus::Failed("err".to_string()).is_terminal());
    /// assert!(TaskStatus::Cancelled.is_terminal());
    ///
    /// assert!(!TaskStatus::Pending.is_terminal());
    /// assert!(!TaskStatus::InProgress { progress_pct: 50 }.is_terminal());
    /// ```
    pub fn is_terminal(&self) -> bool {
        matches!(self, TaskStatus::Completed | TaskStatus::Failed(_) | TaskStatus::Cancelled)
    }
}
```

---

### A8 — Module Documentation

```rust
//! # Geometry
//!
//! 2D geometry primitives for working with points, circles, and rectangles.
//!
//! This module provides efficient, floating-point based geometric types
//! suitable for game development, graphics, and mathematical computation.
//! All coordinates and dimensions use `f64` for maximum precision.
//!
//! ## Quick Start
//!
//! ```
//! use geometry::{Point, Circle};
//!
//! let center = Point { x: 0.0, y: 0.0 };
//! let circle = Circle { center, radius: 5.0 };
//! println!("Area: {:.2}", circle.area());
//! ```
//!
//! ## Included Types
//!
//! - [`Point`] — A 2D coordinate (x, y)
//! - [`Circle`] — A circle defined by center and radius
//! - [`Rectangle`] — An axis-aligned rectangle
```

---

## Section C — Fix the Bugs

### A9 — Wrong syntax for documentation

**Bug:** Uses `//` instead of `///`. Line comments are ignored by `cargo doc` — the function won't appear with any documentation in the generated HTML.

```rust
// ✅ Fixed — use /// for doc comments
/// Returns the length of a string in characters (not bytes).
///
/// This correctly handles multi-byte Unicode characters.
///
/// # Examples
///
/// ```
/// assert_eq!(char_count("hello"), 5);
/// assert_eq!(char_count("日本語"), 3); // 3 chars, 9 bytes
/// ```
pub fn char_count(s: &str) -> usize {
    s.chars().count()
}
```

---

### A10 — Doc test with wrong assertion

**Bug:** The assertion `assert_eq!(reversed, "hello")` is wrong — `reverse_str("hello")` returns `"olleh"`. The doc test will compile but **fail**, causing `cargo test --doc` to report an error.

```rust
// ✅ Fixed
/// Reverses a string.
///
/// # Examples
///
/// ```
/// let reversed = reverse_str("hello");
/// assert_eq!(reversed, "olleh"); // fixed: "olleh" not "hello"
///
/// assert_eq!(reverse_str(""), "");
/// assert_eq!(reverse_str("a"), "a");
/// ```
pub fn reverse_str(s: &str) -> String {
    s.chars().rev().collect()
}
```

---

### A11 — Inner vs outer confusion

**Bug:** `///` is an *outer* doc comment — it documents the next item. But there is no item immediately following these `///` lines (the next thing is the function `trim_and_upper`, but there's no blank line or separator issue — actually `///` inside the module body without an item following is the issue). The intent is to document the module itself, so `//!` (inner doc comment) should be used.

```rust
// ✅ Fixed — use //! inside the module to document it
pub mod utils {
    //! # Utils
    //!
    //! Provides utility functions for string processing.

    pub fn trim_and_upper(s: &str) -> String {
        s.trim().to_uppercase()
    }
}
```

---

### A12 — Missing sections

**Missing sections:**
1. `# Panics` — the function panics when `path.is_empty()`
2. `# Errors` — the function returns `Result` and can fail with `io::Error`
3. `# Arguments` — `path` parameter is not documented

```rust
// ✅ Complete documentation
/// Reads a configuration file and returns its contents as a string.
///
/// Reads the entire file at the given path and returns it as a `String`.
///
/// # Arguments
///
/// * `path` - The filesystem path to the configuration file. Must not be empty.
///
/// # Returns
///
/// `Ok(String)` containing the file contents on success.
///
/// # Errors
///
/// Returns `Err(io::Error)` if:
/// - The file does not exist
/// - Insufficient permissions to read the file
/// - Any other I/O error occurs
///
/// # Panics
///
/// Panics if `path` is an empty string. Use `is_empty()` to check before calling.
///
/// # Examples
///
/// ```no_run
/// let config = read_config("config.toml").unwrap();
/// println!("{config}");
/// ```
pub fn read_config(path: &str) -> Result<String, std::io::Error> {
    if path.is_empty() {
        panic!("Path cannot be empty");
    }
    std::fs::read_to_string(path)
}
```

---

## Section D — Deep Understanding

### A13 — True or False?

| # | Statement | Answer | Explanation |
|---|---|---|---|
| 1 | `///` compiled into binary | **False** | Doc comments are stripped by the compiler; they live only in doc artifacts |
| 2 | Examples run by `cargo test` | **True** | `cargo test --doc` (or plain `cargo test`) compiles and runs doc examples |
| 3 | `//!` documents item after it | **False** | `//!` documents the ENCLOSING item (the module it's inside), not the next item |
| 4 | Full Markdown in `///` | **True** | rustdoc renders full CommonMark Markdown including tables, bold, links |
| 5 | `no_run` compiled not run | **True** | Syntax-checked but never executed |
| 6 | `// TODO:` causes warning | **False** | It's just a comment — no compiler effect. Use `#[warn(dead_code)]` attributes for that |
| 7 | `cargo doc --open` opens browser | **True** | Generates HTML docs and opens your default browser |
| 8 | Block comments can be nested | **True** | Unlike C, Rust supports nested `/* /* */ */` |
| 9 | Failed doc test = `cargo test` failure | **True** | Doc tests are first-class tests — failures are reported and CI fails |
| 10 | Private functions can have `///` | **True** | They just won't appear in public docs unless you use `--document-private-items` |

---

### A14 — Design Challenge

```rust
//! # Math Utilities
//!
//! Provides common mathematical functions for number theory
//! and arithmetic operations.
//!
//! All functions operate on unsigned 64-bit integers unless noted.
//!
//! ## Example
//!
//! ```
//! assert!(is_prime(17));
//! assert_eq!(gcd(48, 18), 6);
//! ```

/// Euler's number `e`, the base of the natural logarithm.
///
/// Accurate to 9 decimal places.
pub const E: f64 = 2.718281828;

/// Returns `true` if `n` is a prime number.
///
/// A prime number is a natural number greater than 1 that has
/// no positive divisors other than 1 and itself.
///
/// Uses trial division up to √n for efficiency.
///
/// # Arguments
///
/// * `n` - The number to test for primality.
///
/// # Returns
///
/// `true` if `n` is prime, `false` otherwise.
///
/// # Panics
///
/// Does not panic for any input.
///
/// # Examples
///
/// ```
/// assert!(!is_prime(0));
/// assert!(!is_prime(1));
/// assert!(is_prime(2));
/// assert!(is_prime(17));
/// assert!(!is_prime(18));
/// ```
pub fn is_prime(n: u64) -> bool {
    if n < 2 { return false; }
    if n == 2 { return true; }
    if n % 2 == 0 { return false; }

    let mut i = 3;
    // Only check up to sqrt(n) — any factor larger than sqrt(n)
    // would have a corresponding factor smaller than sqrt(n)
    while i * i <= n {
        if n % i == 0 { return false; }
        i += 2;
    }
    true
}

/// Computes the greatest common divisor (GCD) of two integers.
///
/// Uses the Euclidean algorithm: repeatedly replaces the larger
/// number with the remainder when divided by the smaller, until
/// one number becomes zero.
///
/// # Arguments
///
/// * `a` - The first non-negative integer.
/// * `b` - The second non-negative integer.
///
/// # Returns
///
/// The GCD of `a` and `b`. Returns `a` if `b` is zero,
/// and `b` if `a` is zero.
///
/// # Examples
///
/// ```
/// assert_eq!(gcd(48, 18), 6);
/// assert_eq!(gcd(100, 75), 25);
/// assert_eq!(gcd(7, 0), 7);
/// assert_eq!(gcd(0, 0), 0);
/// ```
pub fn gcd(mut a: u64, mut b: u64) -> u64 {
    while b != 0 {
        (a, b) = (b, a % b);
    }
    a
}
```

---

### A15 — Big Documentation Review

**6 problems found:**

1. **`count_words` uses wrong format** — "arguments: ..." and "note: ..." are plain prose, not the standard `# Arguments` section with `*` bullets.

2. **`count_words` is missing a blank line** between the summary and the argument description — needed for proper Markdown rendering.

3. **`count_sentences` has the doc test ABOVE the description** — the `# Examples` section appears before the descriptive text, violating convention (summary → description → sections).

4. **`count_sentences` doc test is wrong** — `"hello world"` has no `.` so it returns 0, not 2. The assertion `assert_eq!(n, 3)` is wrong (it says count_words but is on count_sentences, and the sentence count of `"hello world"` is 0).

5. **`/* ... */` block comment for dead code** — should be removed entirely or the code should be deleted. Commented-out code is a code smell for production libraries.

6. **`//! End of file.` at the bottom** — inner doc comments must appear at the TOP of the module/file, before any items. Placing `//!` at the end is invalid or misleading.

**Corrected version:**
```rust
//! A utility library for text processing.
//!
//! Provides functions for analyzing and transforming string content.

/// Counts the number of whitespace-separated words in a string.
///
/// # Arguments
///
/// * `text` - The input string to count words in.
///
/// # Returns
///
/// The number of whitespace-separated tokens in `text`.
///
/// # Examples
///
/// ```
/// assert_eq!(count_words("hello world"), 2);
/// assert_eq!(count_words("  spaced  out  "), 2);
/// assert_eq!(count_words(""), 0);
/// ```
pub fn count_words(text: &str) -> usize {
    text.split_whitespace().count()
}

/// Counts the number of sentences in a string.
///
/// Splits on `.` and counts non-empty segments.
///
/// # Examples
///
/// ```
/// assert_eq!(count_sentences("Hello. World."), 2);
/// assert_eq!(count_sentences("No period"), 1);
/// ```
pub fn count_sentences(text: &str) -> usize {
    text.split('.').filter(|s| !s.trim().is_empty()).count()
}
```

---

## 🏆 Lesson Complete!

You've mastered:
- ✅ All four comment types: `//`, `/* */`, `///`, `//!`
- ✅ Writing effective, useful comments that explain WHY not WHAT
- ✅ Full `///` doc comment structure with all standard sections
- ✅ Markdown in documentation: bold, code, tables, links
- ✅ Doc tests — executable examples that are verified by `cargo test`
- ✅ `//!` for module and crate-level documentation
- ✅ Documenting functions, structs, fields, enums, variants, constants
- ✅ `cargo doc --open` and `cargo test --doc`
- ✅ What to comment and what not to

**Up next:** [Lesson 08 — Ownership Rules](../lesson_08_ownership/lesson_08_ownership.md) 🦀
