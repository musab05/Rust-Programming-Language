# 📘 Lesson 05 — Control Flow: if / else in Rust

> **Series:** Learning Rust from Scratch
> **Difficulty:** Beginner → Intermediate
> **Prerequisite:** [Lesson 04 — Functions](../lesson_04/lesson_04_functions.md)
>
> **Files in this lesson:**
> - [`lesson_05_if_else.md`](./lesson_05_if_else.md) ← You are here
> - [`lesson_05_questions.md`](./lesson_05_questions.md) — Practice problems
> - [`lesson_05_answers.md`](./lesson_05_answers.md) — Solutions & explanations

---

## Table of Contents

1. [What is Control Flow?](#1-what-is-control-flow)
2. [The Basic `if` Expression](#2-the-basic-if-expression)
   - 2.1 [Syntax and Structure](#21-syntax-and-structure)
   - 2.2 [The Condition Must Be a Boolean](#22-the-condition-must-be-a-boolean)
   - 2.3 [The `if` Body is a Block](#23-the-if-body-is-a-block)
3. [The `else` Clause](#3-the-else-clause)
4. [The `else if` Ladder](#4-the-else-if-ladder)
   - 4.1 [Writing an `else if` Chain](#41-writing-an-else-if-chain)
   - 4.2 [Order Matters in `else if` Chains](#42-order-matters-in-else-if-chains)
   - 4.3 [When to Stop Using `else if`](#43-when-to-stop-using-else-if)
5. [`if` as an Expression — Returning Values](#5-if-as-an-expression--returning-values)
   - 5.1 [Assigning from `if`](#51-assigning-from-if)
   - 5.2 [Arms Must Have the Same Type](#52-arms-must-have-the-same-type)
   - 5.3 [`if` Expressions in Function Returns](#53-if-expressions-in-function-returns)
   - 5.4 [`if` Expressions as Function Arguments](#54-if-expressions-as-function-arguments)
6. [Nested `if` Statements](#6-nested-if-statements)
   - 6.1 [Basic Nesting](#61-basic-nesting)
   - 6.2 [Flattening Nested Ifs — Guard Clauses](#62-flattening-nested-ifs--guard-clauses)
7. [Boolean Logic in Conditions](#7-boolean-logic-in-conditions)
   - 7.1 [Logical AND — `&&`](#71-logical-and--)
   - 7.2 [Logical OR — `||`](#72-logical-or--)
   - 7.3 [Logical NOT — `!`](#73-logical-not--)
   - 7.4 [Short-Circuit Evaluation](#74-short-circuit-evaluation)
   - 7.5 [Combining Operators — Precedence](#75-combining-operators--precedence)
8. [Comparing Values in Conditions](#8-comparing-values-in-conditions)
   - 8.1 [Comparison Operators](#81-comparison-operators)
   - 8.2 [Comparing Floats](#82-comparing-floats)
   - 8.3 [Comparing Strings](#83-comparing-strings)
   - 8.4 [Checking Ranges with `&&`](#84-checking-ranges-with-)
9. [`if let` — Conditional Destructuring](#9-if-let--conditional-destructuring)
   - 9.1 [Basic `if let`](#91-basic-if-let)
   - 9.2 [`if let` with `else`](#92-if-let-with-else)
   - 9.3 [When to Use `if let` vs `match`](#93-when-to-use-if-let-vs-match)
10. [Common Patterns and Pitfalls](#10-common-patterns-and-pitfalls)
    - 10.1 [Ternary-style `if` expression](#101-ternary-style-if-expression)
    - 10.2 [The Dangling Else Problem — Does Rust Have It?](#102-the-dangling-else-problem--does-rust-have-it)
    - 10.3 [Returning `bool` from a Function — Don't Use `if`](#103-returning-bool-from-a-function--dont-use-if)
    - 10.4 [Shadowing Inside `if` Blocks](#104-shadowing-inside-if-blocks)
    - 10.5 [Using `if` with `Option` and `Result`](#105-using-if-with-option-and-result)
11. [Real-World Examples](#11-real-world-examples)
12. [Summary Cheat Sheet](#12-summary-cheat-sheet)

---

## 1. What is Control Flow?

By default, a Rust program executes statements **top to bottom**, one line at a time. **Control flow** is the ability to change this — to make decisions, repeat actions, or skip sections of code.

```
Normal execution:          With control flow:
─────────────────          ──────────────────────────────
Statement 1            →   Statement 1
Statement 2            →   if (condition) {
Statement 3            →       Statement 2  ← runs only when true
Statement 4            →   } else {
                       →       Statement 3  ← runs only when false
                       →   }
                       →   Statement 4  ← always runs
```

Rust has several control flow constructs:

| Construct | Purpose |
|-----------|---------|
| `if` / `else` | Branch based on a boolean condition |
| `match` | Pattern matching (next lesson) |
| `loop` | Repeat forever until explicitly stopped |
| `while` | Repeat while a condition is true |
| `for` | Iterate over a collection or range |

**This lesson covers `if` / `else` in full depth.**

---

## 2. The Basic `if` Expression

### 2.1 Syntax and Structure

```
if <condition> {
    // body — runs when condition is true
}
```

```rust
fn main() {
    let temperature = 35;

    if temperature > 30 {
        println!("It's hot outside!");  // only runs if temperature > 30
    }

    println!("Program continues here regardless."); // always runs
}
```

**Output:**
```
It's hot outside!
Program continues here regardless.
```

If `temperature` were `25`, the `if` block would be **skipped entirely** and only the last line would print.

### 2.2 The Condition Must Be a Boolean

In Rust, the condition in an `if` statement **must** evaluate to a `bool`. There is no implicit conversion from integers, pointers, or other types — unlike C, C++, JavaScript, or Python.

```rust
fn main() {
    let x = 5;

    // ✅ These are valid — all produce bool
    if x > 0           { println!("positive"); }
    if x != 0          { println!("non-zero"); }
    if x == 5          { println!("equals five"); }
    if true            { println!("always"); }

    // ❌ These are INVALID in Rust
    // if x { ... }           // error: expected `bool`, found integer
    // if 1 { ... }           // error: expected `bool`, found integer
    // if "hello" { ... }     // error: expected `bool`, found `&str`
    // if Some(5) { ... }     // error: expected `bool`, found `Option<i32>`
}
```

This is a **deliberate design choice** — it prevents bugs where `if (x = 5)` accidentally assigns instead of comparing (a common C bug). In Rust, assignment inside a condition is a compile error.

```rust
let mut x = 0;
// if x = 5 { ... } // ❌ Does not compile — cannot assign in a condition
if x == 5 { ... }   // ✅ Comparison
```

### 2.3 The `if` Body is a Block

The body of an `if` is always a **block** `{ }` — curly braces are **not optional** in Rust, even for single-line bodies:

```rust
// ❌ This does NOT compile
if x > 0
    println!("positive"); // error: expected `{`, found identifier

// ✅ This is correct
if x > 0 {
    println!("positive");
}
```

This eliminates an entire class of bugs found in other languages (like accidentally indenting code thinking it's inside an `if` when it isn't).

---

## 3. The `else` Clause

The `else` clause provides a block that runs when the `if` condition is `false`:

```rust
fn main() {
    let number = 7;

    if number % 2 == 0 {
        println!("{number} is even");
    } else {
        println!("{number} is odd");
    }
}
```

**Output:** `7 is odd`

### Without `else` — sometimes that's fine

You don't always need `else`. If you only care about one case, just use a bare `if`:

```rust
fn main() {
    let score = 95;

    if score == 100 {
        println!("Perfect score! 🎉");
    }
    // If score isn't 100, nothing special happens — that's fine
    println!("Your score: {score}");
}
```

### `else` runs exactly when `if` doesn't

The `if` and `else` blocks are **mutually exclusive** — exactly one of them will run for any given execution:

```rust
fn main() {
    let is_raining = true;

    if is_raining {
        println!("Bring an umbrella.");
    } else {
        println!("Leave the umbrella at home.");
    }
    // Exactly one of these two lines prints — never both, never neither
}
```

---

## 4. The `else if` Ladder

### 4.1 Writing an `else if` Chain

When you have more than two cases, chain `else if` clauses:

```rust
fn main() {
    let score = 78;

    if score >= 90 {
        println!("Grade: A");
    } else if score >= 80 {
        println!("Grade: B");
    } else if score >= 70 {
        println!("Grade: C");
    } else if score >= 60 {
        println!("Grade: D");
    } else {
        println!("Grade: F");
    }
}
```

**Output:** `Grade: C`

The conditions are checked **in order from top to bottom**. The first condition that evaluates to `true` wins, and all remaining branches are **skipped**.

### 4.2 Order Matters in `else if` Chains

The order of your conditions directly affects the result:

```rust
fn main() {
    let x = 15;

    // ✅ Correct order — most restrictive first
    if x > 20 {
        println!("large");
    } else if x > 10 {
        println!("medium");   // ← this runs for x=15
    } else {
        println!("small");
    }

    // ❌ Wrong order — the second condition is never reached
    if x > 10 {
        println!("big");      // ← this runs for x=15 (catches medium AND large!)
    } else if x > 20 {
        println!("very big"); // ← UNREACHABLE: if x > 20, it was already caught above
    } else {
        println!("tiny");
    }
}
```

**A trickier ordering bug:**

```rust
fn classify_age(age: u32) -> &'static str {
    // ❌ Wrong — `age >= 0` is always true for u32, catches everything!
    if age >= 0   { return "any age"; }   // every u32 is >= 0!
    if age >= 18  { return "adult"; }     // unreachable
    if age >= 65  { return "senior"; }    // unreachable
    "unknown"
}

fn classify_age_correct(age: u32) -> &'static str {
    // ✅ Correct — most restrictive first
    if age >= 65  { "senior" }
    else if age >= 18 { "adult" }
    else { "minor" }
}
```

### 4.3 When to Stop Using `else if`

Long `else if` chains become hard to read. Rust's `match` expression is almost always better for more than 2–3 branches. Here's a preview:

```rust
// ❌ Hard to read with many branches
if x == 1 { "one" }
else if x == 2 { "two" }
else if x == 3 { "three" }
else if x == 4 { "four" }
else { "other" }

// ✅ match is cleaner for this case
match x {
    1 => "one",
    2 => "two",
    3 => "three",
    4 => "four",
    _ => "other",
}
```

`match` is covered fully in the next lesson. For now: use `if`/`else if` for conditions with ranges or complex boolean expressions, and `match` for exact value comparisons.

---

## 5. `if` as an Expression — Returning Values

This is one of the most important features of Rust's `if`. Unlike in many languages where `if` is a **statement**, in Rust `if` is an **expression** — it produces a value.

### 5.1 Assigning from `if`

```rust
fn main() {
    let temperature = 28;

    // The entire if/else produces a value, assigned to `weather`
    let weather = if temperature > 25 {
        "hot"      // no semicolon — this is the arm's value
    } else {
        "cool"     // no semicolon — this is the arm's value
    };             // semicolon here ends the `let` statement

    println!("Weather: {weather}"); // Weather: hot
}
```

This is Rust's equivalent of the **ternary operator** (`condition ? value_a : value_b`) found in C, Java, and JavaScript — but cleaner and more powerful because it uses the same `if`/`else` syntax.

```rust
// JavaScript / C style (Rust does NOT have this):
// let weather = temperature > 25 ? "hot" : "cool";

// Rust style (clean, readable, same power):
let weather = if temperature > 25 { "hot" } else { "cool" };
```

### 5.2 Arms Must Have the Same Type

When `if` is used as an expression, **every arm must evaluate to the same type**. If they don't, the compiler rejects the code.

```rust
fn main() {
    let condition = true;

    // ✅ Both arms are &str
    let text = if condition { "yes" } else { "no" };

    // ✅ Both arms are i32
    let number = if condition { 42 } else { 0 };

    // ❌ Arms have different types — compile error
    let mixed = if condition {
        42         // i32
    } else {
        "hello"    // &str  ← ERROR: expected i32, found &str
    };
}
```

**The compiler error:**
```
error[E0308]: `if` and `else` have incompatible types
  --> src/main.rs:10:9
   |
7  |     let mixed = if condition {
   |                 ------------ `if` and `else` have incompatible types
8  |         42
   |         -- expected because of this
9  |     } else {
10 |         "hello"
   |         ^^^^^^^ expected integer, found `&str`
```

**Why?** Because `mixed` has exactly one type — and the compiler must know that type at compile time. If the two arms could produce different types, the compiler couldn't determine `mixed`'s type.

### 5.3 `if` Expressions in Function Returns

Since functions return the value of their last expression, and `if` is an expression, you can use `if`/`else` directly as a function's return value:

```rust
fn absolute_value(n: i32) -> i32 {
    if n < 0 { -n } else { n }
}

fn max(a: i32, b: i32) -> i32 {
    if a > b { a } else { b }
}

fn sign(n: i32) -> &'static str {
    if n > 0 {
        "positive"
    } else if n < 0 {
        "negative"
    } else {
        "zero"
    }
}

fn main() {
    println!("{}", absolute_value(-7)); // 7
    println!("{}", absolute_value(3));  // 3
    println!("{}", max(10, 25));        // 25
    println!("{}", sign(-5));           // negative
    println!("{}", sign(0));            // zero
}
```

### 5.4 `if` Expressions as Function Arguments

You can pass an `if` expression directly as a function argument:

```rust
fn main() {
    let score = 72;
    let passes = score >= 60;

    // if expression used directly as argument
    println!("Result: {}", if passes { "Pass" } else { "Fail" });

    let x = 10;
    let y = 20;

    // if as argument
    println!("Max: {}", if x > y { x } else { y });

    // Multiple if expressions in one call
    println!("{} + {} = {}",
        if x > 0 { "positive" } else { "negative" },
        if y > 0 { "positive" } else { "negative" },
        x + y
    );
}
```

---

## 6. Nested `if` Statements

### 6.1 Basic Nesting

`if` blocks can be nested inside other `if` blocks:

```rust
fn main() {
    let age = 20;
    let has_id = true;

    if age >= 18 {
        if has_id {
            println!("Entry granted.");
        } else {
            println!("You're old enough, but need to show ID.");
        }
    } else {
        println!("Entry denied — must be 18 or older.");
    }
}
```

This works, but nesting more than 2–3 levels deep quickly becomes hard to read.

```rust
// ❌ Deeply nested — hard to follow (also called "arrow code")
fn check(a: bool, b: bool, c: bool, d: bool) {
    if a {
        if b {
            if c {
                if d {
                    println!("All conditions met!");
                }
            }
        }
    }
}
```

### 6.2 Flattening Nested Ifs — Guard Clauses

The **guard clause** pattern rewrites nested `if`s as early returns, keeping the main logic at the same indentation level:

```rust
// ❌ Deeply nested version
fn process_user(name: &str, age: u32, is_active: bool) {
    if !name.is_empty() {
        if age >= 18 {
            if is_active {
                println!("Processing user: {name}");
                // ... main logic buried deep
            } else {
                println!("User is not active.");
            }
        } else {
            println!("User must be 18 or older.");
        }
    } else {
        println!("Name cannot be empty.");
    }
}

// ✅ Guard clause version — flat, readable
fn process_user_clean(name: &str, age: u32, is_active: bool) {
    if name.is_empty()  { println!("Name cannot be empty."); return; }
    if age < 18         { println!("User must be 18 or older."); return; }
    if !is_active       { println!("User is not active."); return; }

    // Main logic — always reachable here, no nesting
    println!("Processing user: {name}");
}

fn main() {
    process_user_clean("Alice", 25, true);   // Processing user: Alice
    process_user_clean("", 25, true);        // Name cannot be empty.
    process_user_clean("Bob", 15, true);     // User must be 18 or older.
    process_user_clean("Carol", 30, false);  // User is not active.
}
```

**The guard clause pattern is considered highly idiomatic in Rust.** Handle all invalid/edge cases first with early returns, then write the main logic cleanly at the base level.

---

## 7. Boolean Logic in Conditions

### 7.1 Logical AND — `&&`

`&&` is `true` only when **both** sides are `true`:

```rust
fn main() {
    let age = 25;
    let has_ticket = true;

    if age >= 18 && has_ticket {
        println!("Welcome to the event!");
    }

    // Truth table for &&:
    // true  && true  = true
    // true  && false = false
    // false && true  = false
    // false && false = false
}
```

### 7.2 Logical OR — `||`

`||` is `true` when **at least one** side is `true`:

```rust
fn main() {
    let is_weekend = false;
    let is_holiday = true;

    if is_weekend || is_holiday {
        println!("Enjoy your day off!"); // prints because is_holiday is true
    }

    // Truth table for ||:
    // true  || true  = true
    // true  || false = true
    // false || true  = true
    // false || false = false
}
```

### 7.3 Logical NOT — `!`

`!` inverts a boolean:

```rust
fn main() {
    let is_raining = false;

    if !is_raining {
        println!("Go for a walk!"); // prints because !false = true
    }

    let has_error = false;
    let success = !has_error; // true

    println!("Success: {success}"); // Success: true
}
```

### 7.4 Short-Circuit Evaluation

Rust evaluates `&&` and `||` with **short-circuit logic** — if the result can be determined from the left side alone, the right side is **never evaluated**:

```rust
fn expensive_check() -> bool {
    println!("expensive_check was called!"); // side effect we can observe
    true
}

fn main() {
    // Short-circuit AND: left is false → right is never evaluated
    let result = false && expensive_check();
    println!("{result}"); // false
    // "expensive_check was called!" is NOT printed

    // Short-circuit OR: left is true → right is never evaluated
    let result = true || expensive_check();
    println!("{result}"); // true
    // "expensive_check was called!" is NOT printed

    // No short-circuit: left is true for &&, so right IS evaluated
    let result = true && expensive_check();
    println!("{result}"); // true
    // "expensive_check was called!" IS printed
}
```

**Why this matters:**

1. **Performance** — skip expensive computations when the result is already known
2. **Safety** — prevents null/bounds checks on the right side when the left fails:

```rust
fn main() {
    let numbers = vec![1, 2, 3];
    let index = 10;

    // Safe: index < numbers.len() is checked first
    // If it's false, numbers[index] is NEVER evaluated (no panic!)
    if index < numbers.len() && numbers[index] > 2 {
        println!("Found a large number!");
    } else {
        println!("Index out of range or number not large enough.");
    }
}
```

### 7.5 Combining Operators — Precedence

When combining `&&`, `||`, and `!`, operator precedence determines grouping:

```
Precedence (highest to lowest):
1. !   (NOT)
2. &&  (AND)
3. ||  (OR)
```

```rust
fn main() {
    let a = true;
    let b = false;
    let c = true;

    // ! binds tightest
    println!("{}", !a && b);          // (!a) && b = false && false = false
    println!("{}", !a || c);          // (!a) || c = false || true  = true

    // && before ||
    println!("{}", a || b && c);      // a || (b && c) = true || false = true
    println!("{}", (a || b) && c);    // (a || b) && c = true && true  = true

    // Use parentheses for clarity when combining
    let is_valid = (a && !b) || (b && !c);
    println!("{is_valid}");           // (true && true) || (false && false) = true
}
```

**Best practice:** Use **parentheses** whenever you combine `&&` and `||` in the same expression — even if the precedence would give the right result. It makes the intent clear:

```rust
// ❌ Relying on precedence — easy to misread
if user_is_admin || user_is_owner && resource_is_public { ... }

// ✅ Parentheses make the grouping explicit and intention clear
if user_is_admin || (user_is_owner && resource_is_public) { ... }
```

---

## 8. Comparing Values in Conditions

### 8.1 Comparison Operators

```rust
fn main() {
    let a = 10;
    let b = 20;

    println!("{}", a == b);  // false — equal
    println!("{}", a != b);  // true  — not equal
    println!("{}", a <  b);  // true  — less than
    println!("{}", a >  b);  // false — greater than
    println!("{}", a <= b);  // true  — less than or equal
    println!("{}", a >= b);  // false — greater than or equal
}
```

### 8.2 Comparing Floats

Never use `==` to compare floats directly — use an epsilon tolerance:

```rust
fn main() {
    let result = 0.1 + 0.2;
    let expected = 0.3;

    // ❌ Wrong — may be false due to floating-point precision
    if result == expected {
        println!("Equal");
    }

    // ✅ Correct — compare within a tolerance
    let epsilon = 1e-10;
    if (result - expected).abs() < epsilon {
        println!("Close enough to equal"); // this runs
    }

    // For "greater than" and "less than", direct comparison is fine
    if result > 0.0 {
        println!("positive"); // this is fine
    }
}
```

### 8.3 Comparing Strings

```rust
fn main() {
    let greeting = "hello";

    // == compares content (not memory address)
    if greeting == "hello" {
        println!("Got hello!");
    }

    if greeting != "world" {
        println!("Not world"); // prints
    }

    // Lexicographic comparison (alphabetical order)
    if "apple" < "banana" {
        println!("apple comes first"); // prints
    }

    // Case-insensitive comparison
    let input = "HELLO";
    if input.to_lowercase() == "hello" {
        println!("Case-insensitive match"); // prints
    }

    // Checking string content
    let sentence = "Hello, Rustacean!";
    if sentence.starts_with("Hello") {
        println!("Starts with Hello"); // prints
    }
    if sentence.ends_with("!") {
        println!("Ends with !"); // prints
    }
    if sentence.contains("Rust") {
        println!("Contains Rust"); // prints
    }
    if sentence.is_empty() {
        println!("Empty string");
    } else {
        println!("Not empty"); // prints
    }
}
```

### 8.4 Checking Ranges with `&&`

Rust doesn't have a built-in range condition like Python's `0 < x < 10`. Use `&&` instead:

```rust
fn main() {
    let age = 25;
    let temperature = 22.5_f64;

    // ❌ Python style — does NOT work in Rust
    // if 18 < age < 65 { ... }

    // ✅ Rust style — explicit &&
    if age >= 18 && age < 65 {
        println!("Working age adult");
    }

    if temperature >= 18.0 && temperature <= 26.0 {
        println!("Comfortable room temperature");
    }

    // Using Rust's range contains method (alternative approach)
    if (18..65).contains(&age) {
        println!("Working age (using range)");
    }

    if (18.0..=26.0).contains(&temperature) {
        println!("Comfortable (using range)");
    }
}
```

---

## 9. `if let` — Conditional Destructuring

`if let` is a special syntax that combines a pattern match with a conditional. It's primarily used with `Option<T>` and `Result<T, E>` types.

### 9.1 Basic `if let`

```rust
fn main() {
    let maybe_number: Option<i32> = Some(42);

    // ❌ Verbose way — using match for a single case
    match maybe_number {
        Some(n) => println!("Got: {n}"),
        None    => {}  // nothing to do here
    }

    // ✅ Cleaner with if let — when you only care about one pattern
    if let Some(n) = maybe_number {
        println!("Got: {n}"); // n is bound here as 42
    }
    // if maybe_number is None, the block is simply skipped

    // Works with any pattern
    let pair = (true, 99);
    if let (true, x) = pair {
        println!("First is true, second is: {x}"); // x = 99
    }

    // Works with enum variants
    let value: Option<&str> = Some("hello");
    if let Some(text) = value {
        println!("Text: {text}"); // Text: hello
    }

    let nothing: Option<i32> = None;
    if let Some(n) = nothing {
        println!("Got: {n}"); // This block is skipped
    }
    println!("After the if let"); // Always runs
}
```

### 9.2 `if let` with `else`

```rust
fn main() {
    let config_value: Option<u32> = Some(8080);

    let port = if let Some(p) = config_value {
        p
    } else {
        3000 // default port
    };

    println!("Listening on port: {port}"); // 8080

    // Practical example — parsing
    let user_input = "42";
    if let Ok(number) = user_input.parse::<i32>() {
        println!("Parsed: {number}");
    } else {
        println!("Could not parse as number");
    }

    let bad_input = "abc";
    if let Ok(number) = bad_input.parse::<i32>() {
        println!("Parsed: {number}");
    } else {
        println!("Could not parse 'abc' as number"); // this runs
    }
}
```

### 9.3 When to Use `if let` vs `match`

```rust
fn main() {
    let score: Option<u32> = Some(85);

    // Use `if let` when:
    // — you only care about ONE pattern
    // — the else case does nothing or is simple
    if let Some(s) = score {
        println!("Score: {s}");
    }

    // Use `match` when:
    // — you need to handle MULTIPLE patterns
    // — you want exhaustive coverage
    match score {
        Some(s) if s >= 90 => println!("Excellent! {s}"),
        Some(s) if s >= 70 => println!("Good: {s}"),
        Some(s)            => println!("Needs work: {s}"),
        None               => println!("No score yet"),
    }
}
```

---

## 10. Common Patterns and Pitfalls

### 10.1 Ternary-style `if` expression

Rust's `if` expression replaces the `? :` ternary operator of other languages:

```rust
fn main() {
    let x = 10;

    // Rust — concise, on one line
    let label = if x > 0 { "positive" } else { "non-positive" };
    println!("{label}");

    // Can be used inline in any expression context
    let message = format!("x is {}", if x > 0 { "positive" } else { "non-positive" });
    println!("{message}");

    // Multi-line when the logic is complex — equally valid
    let grade = if x >= 90 {
        'A'
    } else if x >= 80 {
        'B'
    } else {
        'C'
    };
    println!("{grade}");
}
```

### 10.2 The Dangling Else Problem — Does Rust Have It?

In C/C++, there's a famous ambiguity:
```c
// C — which `if` does the `else` belong to?
if (a)
    if (b)
        do_something();
else
    do_other(); // belongs to if(b), not if(a) — surprising!
```

**Rust doesn't have this problem** because braces are mandatory:

```rust
fn main() {
    let a = true;
    let b = false;

    // Rust — braces make structure completely unambiguous
    if a {
        if b {
            println!("both");
        }
        // else here would belong to if b — structurally clear
    } else {
        println!("not a"); // this clearly belongs to if a
    }
}
```

Mandatory braces eliminate an entire category of ambiguity bugs.

### 10.3 Returning `bool` from a Function — Don't Use `if`

A very common beginner mistake is writing unnecessary `if/else` when returning a bool:

```rust
// ❌ Redundant — comparison already produces bool
fn is_even(n: i32) -> bool {
    if n % 2 == 0 {
        return true;
    } else {
        return false;
    }
}

// ❌ Slightly better but still redundant
fn is_even(n: i32) -> bool {
    if n % 2 == 0 { true } else { false }
}

// ✅ Idiomatic — the comparison IS the boolean
fn is_even(n: i32) -> bool {
    n % 2 == 0
}

// ✅ Also fine for more complex conditions
fn is_valid_score(score: i32) -> bool {
    score >= 0 && score <= 100
}

fn main() {
    println!("{}", is_even(4));          // true
    println!("{}", is_even(7));          // false
    println!("{}", is_valid_score(85));  // true
    println!("{}", is_valid_score(-1));  // false
}
```

### 10.4 Shadowing Inside `if` Blocks

Variables declared inside an `if` block are local to that block:

```rust
fn main() {
    let x = 10;

    if x > 5 {
        let message = "big";  // only exists in this block
        println!("{message}");
    }
    // println!("{message}"); // ❌ ERROR — message is out of scope

    // If you need the value outside, declare it before the if:
    let label;
    if x > 5 {
        label = "big";
    } else {
        label = "small";
    }
    println!("{label}"); // ✅ Works — label was declared in outer scope

    // Or more idiomatically, use if as an expression:
    let label = if x > 5 { "big" } else { "small" };
    println!("{label}"); // ✅ Cleaner
}
```

### 10.5 Using `if` with `Option` and `Result`

Before `if let`, you might try using `==` with `Option` — but that requires `None` comparisons:

```rust
fn main() {
    let maybe: Option<i32> = Some(5);

    // ❌ Clunky — checks the Option itself
    if maybe == Some(5) {
        println!("It's 5!"); // works but fragile
    }

    // ❌ Also works but verbose
    if maybe != None {
        println!("Has a value");
    }

    // ✅ Idiomatic — is_some() / is_none() for simple checks
    if maybe.is_some() {
        println!("Has a value");
    }
    if maybe.is_none() {
        println!("Is empty");
    }

    // ✅ Best — if let for when you need the inner value
    if let Some(n) = maybe {
        println!("Value is: {n}");
    }

    // Result works the same way
    let result: Result<i32, &str> = Ok(42);
    if result.is_ok() {
        println!("Success!");
    }
    if let Ok(value) = result {
        println!("Value: {value}");
    }
}
```

---

## 11. Real-World Examples

### Example 1 — Login Validator

```rust
fn validate_login(username: &str, password: &str) -> &'static str {
    // Guard clauses for each invalid case
    if username.is_empty() {
        return "Username cannot be empty";
    }
    if password.is_empty() {
        return "Password cannot be empty";
    }
    if username.len() < 3 {
        return "Username must be at least 3 characters";
    }
    if password.len() < 8 {
        return "Password must be at least 8 characters";
    }

    // All checks passed
    "Login valid"
}

fn main() {
    println!("{}", validate_login("", "password123"));       // Username cannot be empty
    println!("{}", validate_login("al", "password123"));     // Username must be at least 3...
    println!("{}", validate_login("alice", "short"));        // Password must be at least 8...
    println!("{}", validate_login("alice", "secure123"));    // Login valid
}
```

### Example 2 — BMI Calculator with Category

```rust
fn bmi_category(bmi: f64) -> &'static str {
    if bmi < 0.0 {
        "Invalid BMI"
    } else if bmi < 18.5 {
        "Underweight"
    } else if bmi < 25.0 {
        "Normal weight"
    } else if bmi < 30.0 {
        "Overweight"
    } else {
        "Obese"
    }
}

fn calculate_bmi(weight_kg: f64, height_m: f64) -> f64 {
    if height_m <= 0.0 {
        return -1.0; // invalid sentinel value
    }
    weight_kg / (height_m * height_m)
}

fn main() {
    let weight = 70.0;
    let height = 1.75;

    let bmi = calculate_bmi(weight, height);
    let category = bmi_category(bmi);

    println!("BMI: {:.1}", bmi);          // BMI: 22.9
    println!("Category: {category}");     // Category: Normal weight
}
```

### Example 3 — Traffic Light State Machine

```rust
fn next_light(current: &str) -> &'static str {
    if current == "red" {
        "green"
    } else if current == "green" {
        "yellow"
    } else if current == "yellow" {
        "red"
    } else {
        "unknown"
    }
}

fn can_go(light: &str) -> bool {
    light == "green"
}

fn main() {
    let mut light = "red";

    for _ in 0..6 {
        println!("Light: {light} — {}",
            if can_go(light) { "GO" } else { "STOP" }
        );
        light = next_light(light);
    }
}
```

**Output:**
```
Light: red — STOP
Light: green — GO
Light: yellow — STOP
Light: red — STOP
Light: green — GO
Light: yellow — STOP
```

### Example 4 — Safe Array Access with Bounds Check

```rust
fn safe_get(arr: &[i32], index: usize) -> Option<i32> {
    if index < arr.len() {
        Some(arr[index])
    } else {
        None
    }
}

fn main() {
    let data = [10, 20, 30, 40, 50];

    for i in [0, 2, 4, 7] {
        if let Some(value) = safe_get(&data, i) {
            println!("data[{i}] = {value}");
        } else {
            println!("data[{i}] = out of bounds");
        }
    }
}
```

**Output:**
```
data[0] = 10
data[2] = 30
data[4] = 50
data[7] = out of bounds
```

---

## 12. Summary Cheat Sheet

```rust
// ════════════════════════════════════════════════
// BASIC IF / ELSE
// ════════════════════════════════════════════════

// Simple if
if condition {
    // runs when true
}

// if / else
if condition {
    // true branch
} else {
    // false branch
}

// else if ladder
if condition_a {
    // ...
} else if condition_b {
    // ...
} else {
    // default
}

// ════════════════════════════════════════════════
// IF AS AN EXPRESSION (assigns a value)
// ════════════════════════════════════════════════

let x = if condition { value_a } else { value_b };
// both arms must have the same type!

// As return value of a function
fn sign(n: i32) -> &'static str {
    if n > 0 { "positive" } else if n < 0 { "negative" } else { "zero" }
}

// ════════════════════════════════════════════════
// BOOLEAN OPERATORS
// ════════════════════════════════════════════════

if a && b  { }   // AND  — both true
if a || b  { }   // OR   — at least one true
if !a      { }   // NOT  — flip
if (a && b) || c { }  // use parentheses when mixing && and ||

// ════════════════════════════════════════════════
// COMPARISON OPERATORS
// ════════════════════════════════════════════════

==  !=  <  >  <=  >=

// Range check
if x >= 0 && x < 100 { }         // manual range
if (0..100).contains(&x) { }     // using range

// Float comparison — always use epsilon!
if (a - b).abs() < 1e-10 { }     // not == for floats!

// ════════════════════════════════════════════════
// IF LET — PATTERN MATCHING IN A CONDITION
// ════════════════════════════════════════════════

if let Some(value) = option_var {
    // value is bound here
}

if let Some(value) = option_var {
    // use value
} else {
    // option_var was None
}

if let Ok(result) = result_var {
    // result is bound here
}

// ════════════════════════════════════════════════
// GUARD CLAUSE PATTERN
// ════════════════════════════════════════════════

fn process(x: i32) -> &'static str {
    if x < 0   { return "negative"; }     // guard 1
    if x == 0  { return "zero"; }         // guard 2
    if x > 100 { return "too large"; }    // guard 3

    "valid"  // main logic — clean, at base level
}

// ════════════════════════════════════════════════
// IDIOMATIC PATTERNS
// ════════════════════════════════════════════════

// ✅ Return bool directly (no if needed)
fn is_even(n: i32) -> bool { n % 2 == 0 }

// ✅ Ternary-style
let label = if x > 0 { "pos" } else { "non-pos" };

// ✅ Option checks
if value.is_some() { }
if value.is_none() { }
if let Some(v) = value { }

// ✅ Short-circuit for safety
if i < arr.len() && arr[i] > 0 { }
```

---

## 🔗 What's Next?

- **Lesson 06 — Control Flow: Loops** (`loop`, `while`, `for`, `break`, `continue`)
- **Lesson 07 — Control Flow: `match`** (Rust's most powerful branching construct)
- **Lesson 08 — Ownership** (Rust's memory model — the most unique Rust concept)

---

## 📚 Further Reading

- [The Rust Book — Chapter 3.5: Control Flow](https://doc.rust-lang.org/book/ch03-05-control-flow.html)
- [Rust by Example — if/else](https://doc.rust-lang.org/rust-by-example/flow_control/if_else.html)
- [Rust by Example — if let](https://doc.rust-lang.org/rust-by-example/flow_control/if_let.html)
- [Rust Reference — If expressions](https://doc.rust-lang.org/reference/expressions/if-expr.html)

---

*"Clear conditions make for clear code. Say what you mean, and mean what you say." 🦀*
