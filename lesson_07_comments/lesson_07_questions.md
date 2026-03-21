# 🧪 Lesson 07 — Practice Questions: Comments & Documentation

> Answers are in [`lesson_07_answers.md`](./lesson_07_answers.md)

---

## Section A — Identify and Analyze

### Q1 — Comment Types
For each comment below, identify the type (`//`, `/* */`, `///`, `//!`) and state its purpose:

```rust
//! # Auth Module
//! Handles user authentication for the application.

/// Hashes a plaintext password using bcrypt.
///
/// # Arguments
/// * `password` - The plaintext password to hash.
///
/// # Examples
/// ```
/// let hash = hash_password("secret");
/// assert!(hash.len() > 0);
/// ```
pub fn hash_password(password: &str) -> String {
    // TODO: integrate real bcrypt library
    /* placeholder — returns input for now */
    password.to_string()
}
```

---

### Q2 — Good vs Bad Comments
For each comment, say whether it is GOOD or BAD and explain why. Then rewrite the bad ones.

```rust
// A: add 1 to index
let index = index + 1;

// B: offset by 1 because the legacy API uses 1-based indexing
let index = index + 1;

// C: loop through items
for item in &cart { process(item); }

// D: apply 15% senior discount for customers aged 65+
let price = price * 0.85;

// E: returns true
fn is_active(user: &User) -> bool { user.active }

// F: returns true if the user account is active and not suspended
fn is_active(user: &User) -> bool { user.active }
```

---

### Q3 — Doc Comment Structure
The following doc comment has several problems. List every issue and write a corrected version.

```rust
// This function adds two numbers together and returns the result
// arguments: a (first number), b (second number)
// returns: the sum
// example: add(2, 3) = 5
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

---

### Q4 — Doc Test Analysis
Which of these doc test code blocks will compile and pass? Which will fail? Which won't compile? Explain each.

**Block A:**
```rust
/// ```
/// let x = 5;
/// assert_eq!(x * 2, 10);
/// ```
```

**Block B:**
```rust
/// ```
/// let result = 1 / 0;
/// ```
```

**Block C:**
```rust
/// ```should_panic
/// let v: Vec<i32> = vec![];
/// let _ = v[5];
/// ```
```

**Block D:**
```rust
/// ```no_run
/// let conn = connect_to_database("postgres://localhost/mydb");
/// ```
```

**Block E:**
```rust
/// ```ignore
/// this is not valid rust code at all !!!
/// ```
```

---

## Section B — Write the Documentation

### Q5 — Document this function

Write complete `///` documentation for this function following Rust conventions:

```rust
pub fn clamp(value: f64, min: f64, max: f64) -> f64 {
    if value < min { min }
    else if value > max { max }
    else { value }
}
```

Your doc comment must include: summary, description, Arguments section, Returns section, Examples section (with assertions), and a Panics section (note: it doesn't panic, but what about if min > max?).

---

### Q6 — Document this struct

Write complete `///` documentation for the struct and all its fields, plus the two methods:

```rust
pub struct BankAccount {
    pub owner: String,
    balance: f64,
    pub is_frozen: bool,
}

impl BankAccount {
    pub fn deposit(&mut self, amount: f64) -> Result<f64, String> {
        if self.is_frozen { return Err("Account is frozen".to_string()); }
        if amount <= 0.0  { return Err("Amount must be positive".to_string()); }
        self.balance += amount;
        Ok(self.balance)
    }

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

### Q7 — Document this enum

Write `///` documentation for the enum, all variants, and the `is_terminal` method:

```rust
pub enum TaskStatus {
    Pending,
    InProgress { progress_pct: u8 },
    Completed,
    Failed(String),
    Cancelled,
}

impl TaskStatus {
    pub fn is_terminal(&self) -> bool {
        matches!(self, TaskStatus::Completed | TaskStatus::Failed(_) | TaskStatus::Cancelled)
    }
}
```

---

### Q8 — Module-level documentation

Write `//!` inner documentation for a module called `geometry` that:
- Has a one-sentence summary
- Has a longer description explaining it provides 2D geometry primitives
- Has a Quick Start section with a usage example
- Lists what is included: `Point`, `Circle`, `Rectangle`
- Notes that all coordinates use `f64`

---

## Section C — Fix the Bugs

### Q9 — Wrong comment syntax for documentation

```rust
// Returns the length of a string in characters.
//
// # Examples
// ```
// assert_eq!(char_count("hello"), 5);
// ```
pub fn char_count(s: &str) -> usize {
    s.chars().count()
}
```

---

### Q10 — Doc test that's wrong

```rust
/// Reverses a string.
///
/// # Examples
///
/// ```
/// let reversed = reverse_str("hello");
/// assert_eq!(reversed, "hello"); // should be "olleh"
/// ```
pub fn reverse_str(s: &str) -> String {
    s.chars().rev().collect()
}
```

---

### Q11 — Inner vs outer confusion

```rust
pub mod utils {
    /// # Utils Module
    ///
    /// Provides utility functions for string processing.

    pub fn trim_and_upper(s: &str) -> String {
        s.trim().to_uppercase()
    }
}
```

---

### Q12 — Missing sections

This doc comment is for a function that can panic and returns a Result. What critical sections are missing?

```rust
/// Reads a configuration file and returns parsed settings.
///
/// # Examples
///
/// ```no_run
/// let config = read_config("config.toml").unwrap();
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

### Q13 — True or False?

1. `///` comments are compiled into the final binary.
2. Doc test examples in `# Examples` are run by `cargo test`.
3. `//!` documents the item that comes after it.
4. You can use full Markdown (tables, bold, links) in `///` comments.
5. A ````no_run` block is compiled but not executed.
6. `// TODO:` comments cause a compiler warning.
7. `cargo doc --open` generates docs and opens them in a browser.
8. Block comments `/* */` can be nested in Rust.
9. If a doc test fails, `cargo test` reports a failure.
10. Private functions can also have `///` doc comments.

---

### Q14 — Design Challenge

You're building a `math` module for a library. Write a complete, properly documented Rust snippet that includes:

1. `//!` module-level documentation with a summary, description, and example
2. A documented `pub const` for Euler's number `e ≈ 2.718281828`
3. A fully documented `pub fn is_prime(n: u64) -> bool` with all relevant sections including a doc test
4. A fully documented `pub fn gcd(a: u64, b: u64) -> u64` (greatest common divisor)
5. At least one `// line comment` inside a function body explaining a non-obvious step

---

### Q15 — Big Documentation Review

The following code has **6 documentation problems**. Find each one, explain it, and write the fully corrected version.

```rust
//! A utility library for text processing.

/// Counts words in a string
/// arguments: text - the input string
/// note: splits on whitespace
pub fn count_words(text: &str) -> usize {
    text.split_whitespace().count()
}

/// ```
/// let n = count_words("hello world");
/// assert_eq!(n, 3);
/// ```
///
/// Splits the text on whitespace and counts parts.
pub fn count_sentences(text: &str) -> usize {
    text.split('.').filter(|s| !s.trim().is_empty()).count()
}

/* This function is currently unused
pub fn deprecated_counter(text: &str) -> usize {
    text.len()
}
*/

//! End of file.
```

---

*"Good documentation is a gift to your future self." 🦀*
