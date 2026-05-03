# ✅ Lesson 47 — Answers: Publishing to crates.io (M5)

---

## Section A

### A1
Required fields: `name`, `version`, `description`, and `license` (or `license-file`).

### A2
- `///` — documents the NEXT item (function, struct, enum, etc.)
- `//!` — documents the ENCLOSING item (module, crate). Used at the top of `lib.rs` for crate-level documentation.

### A3
It validates the crate for publishing (checks metadata, runs tests, packages the crate) but does NOT actually upload it to crates.io. Useful for catching issues before publishing.

---

## Section B

### A4
```rust
/// Checks if a string is a palindrome (reads the same forwards and backwards).
///
/// The comparison is case-insensitive and ignores non-alphanumeric characters.
///
/// # Arguments
///
/// * `s` - The string to check
///
/// # Returns
///
/// `true` if the string is a palindrome, `false` otherwise.
///
/// # Examples
///
/// ```
/// use my_crate::is_palindrome;
///
/// assert!(is_palindrome("racecar"));
/// assert!(is_palindrome("A man a plan a canal Panama"));
/// assert!(!is_palindrome("hello"));
/// assert!(is_palindrome(""));
/// ```
pub fn is_palindrome(s: &str) -> bool {
    let cleaned: String = s.chars()
        .filter(|c| c.is_alphanumeric())
        .map(|c| c.to_lowercase().next().unwrap())
        .collect();
    let reversed: String = cleaned.chars().rev().collect();
    cleaned == reversed
}
```

### A5
```toml
[package]
name = "text-utils"
version = "0.1.0"
edition = "2021"
authors = ["Your Name <you@example.com>"]
description = "A collection of text processing utilities for Rust"
license = "MIT OR Apache-2.0"
repository = "https://github.com/yourname/text-utils"
homepage = "https://github.com/yourname/text-utils"
documentation = "https://docs.rs/text-utils"
readme = "README.md"
keywords = ["text", "string", "processing", "utils"]
categories = ["text-processing"]
```

---

## Section C

### A6
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | `cargo test` runs doc tests in `///` blocks by default |
| 2 | **False** | Yank prevents NEW projects from depending on it; existing builds still work |
| 3 | **True** | Pre-1.0, any minor version bump can include breaking changes |
| 4 | **True** | crates.io requires GitHub OAuth for authentication |
| 5 | **True** | `cargo doc --open` builds docs and opens in default browser |
| 6 | **True** | `examples/` directory + `cargo run --example name` is the standard pattern |

---

## 🏆 Lesson 47 Complete!

**Next up:** [Lesson 48 — Closures: Syntax & Captures](../lesson_48_closures/lesson_48_closures.md) 🦀
