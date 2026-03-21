# 📘 Lesson 02 — Data Types: Scalar Types in Rust

> **Series:** Learning Rust from Scratch
> **Difficulty:** Beginner → Intermediate
> **Prerequisite:** [Lesson 01 — Variables & Mutability](../lesson_01/lesson_01_variables_and_mutability.md)
>
> **Files in this lesson:**
> - [`lesson_02_scalar_types.md`](./lesson_02_scalar_types.md) ← You are here
> - [`lesson_02_questions.md`](./lesson_02_questions.md) — Practice problems
> - [`lesson_02_answers.md`](./lesson_02_answers.md) — Solutions & explanations

---

## Table of Contents

1. [What Are Data Types?](#1-what-are-data-types)
2. [Scalar vs Compound Types](#2-scalar-vs-compound-types)
3. [Integer Types](#3-integer-types)
   - 3.1 [Signed vs Unsigned](#31-signed-vs-unsigned)
   - 3.2 [Integer Sizes](#32-integer-sizes)
   - 3.3 [Integer Literals](#33-integer-literals)
   - 3.4 [Integer Overflow](#34-integer-overflow)
   - 3.5 [Integer Methods](#35-integer-methods)
4. [Floating-Point Types](#4-floating-point-types)
   - 4.1 [f32 vs f64](#41-f32-vs-f64)
   - 4.2 [Floating-Point Precision](#42-floating-point-precision)
   - 4.3 [Special Float Values](#43-special-float-values)
   - 4.4 [Float Methods](#44-float-methods)
5. [Numeric Operations](#5-numeric-operations)
6. [The Boolean Type](#6-the-boolean-type)
   - 6.1 [Boolean Literals](#61-boolean-literals)
   - 6.2 [Comparison Operators](#62-comparison-operators)
   - 6.3 [Logical Operators](#63-logical-operators)
   - 6.4 [Boolean in Control Flow](#64-boolean-in-control-flow)
7. [The Character Type](#7-the-character-type)
   - 7.1 [char vs String](#71-char-vs-string)
   - 7.2 [Unicode Scalars](#72-unicode-scalars)
   - 7.3 [char Methods](#73-char-methods)
8. [Type Casting](#8-type-casting)
9. [Type Aliases](#9-type-aliases)
10. [Choosing the Right Type](#10-choosing-the-right-type)
11. [Summary Cheat Sheet](#11-summary-cheat-sheet)

---

## 1. What Are Data Types?

Every value in Rust has a **data type**. The type tells the compiler:

- How much **memory** to allocate for the value
- What **operations** are valid on the value
- How to **interpret** the raw bytes stored in memory

```rust
let x: i32 = 42;
//     ^^^
//     This is the type — it tells Rust:
//     - Allocate 4 bytes (32 bits)
//     - Treat it as a signed integer
//     - Allow arithmetic, comparison, etc.
```

Rust is a **statically typed** language — every variable's type must be known at **compile time**. This means type errors are caught before your program ever runs.

### Why does this matter?

```rust
let a: u8 = 200;
let b: u8 = 100;
let c = a + b; // 200 + 100 = 300 — but u8 max is 255!
               // Rust catches this as a potential overflow
```

Understanding types isn't just academic — it directly affects the **correctness** and **performance** of your programs.

---

## 2. Scalar vs Compound Types

Rust's type system is split into two broad categories:

```
Rust Types
├── Scalar (single value)
│   ├── Integer     (i8, i16, i32, i64, i128, isize, u8, u16, u32, u64, u128, usize)
│   ├── Float       (f32, f64)
│   ├── Boolean     (bool)
│   └── Character   (char)
│
└── Compound (multiple values)
    ├── Tuple       (fixed-size, mixed types)
    └── Array       (fixed-size, same type)
```

**This lesson covers Scalar types only.**  
Compound types are covered in Lesson 03.

A **scalar** type represents a **single, indivisible value** — just like a scalar in mathematics.

---

## 3. Integer Types

An integer is a **whole number** — no decimal point, no fractional part.

### 3.1 Signed vs Unsigned

**Signed integers** can be **positive or negative** (they carry a sign).  
**Unsigned integers** can only be **zero or positive**.

```rust
let signed: i32 = -42;    // ✅ negative allowed
let unsigned: u32 = 42;   // ✅ only zero and positive
let bad: u32 = -1;        // ❌ COMPILE ERROR
```

The naming convention:
- `i` prefix → **signed** integer (`i`nteger)
- `u` prefix → **unsigned** integer (`u`nsigned)

### 3.2 Integer Sizes

The number after the prefix is the **bit width** — how many bits of memory are used.

| Type | Bits | Bytes | Min Value | Max Value |
|------|------|-------|-----------|-----------|
| `i8` | 8 | 1 | -128 | 127 |
| `i16` | 16 | 2 | -32,768 | 32,767 |
| `i32` | 32 | 4 | -2,147,483,648 | 2,147,483,647 |
| `i64` | 64 | 8 | -9.2 × 10¹⁸ | 9.2 × 10¹⁸ |
| `i128` | 128 | 16 | -1.7 × 10³⁸ | 1.7 × 10³⁸ |
| `isize` | arch | arch | platform-dependent | platform-dependent |
| `u8` | 8 | 1 | 0 | 255 |
| `u16` | 16 | 2 | 0 | 65,535 |
| `u32` | 32 | 4 | 0 | 4,294,967,295 |
| `u64` | 64 | 8 | 0 | 1.8 × 10¹⁹ |
| `u128` | 128 | 16 | 0 | 3.4 × 10³⁸ |
| `usize` | arch | arch | 0 | platform-dependent |

> **`isize` and `usize`:** These are special — their size depends on your computer's architecture. On a 64-bit machine, they are 64-bit. On a 32-bit machine, they are 32-bit. They are primarily used for **indexing into collections**.

**The formula for ranges:**
- Signed `iN`: from **-(2^(N-1))** to **2^(N-1) - 1**
- Unsigned `uN`: from **0** to **2^N - 1**

```rust
// Examples
let a: i8  = 127;   // max i8
let b: u8  = 255;   // max u8
let c: i32 = -2_147_483_648; // min i32 (underscores for readability)
```

**Default integer type is `i32`** — it's the most performant on most 64-bit systems and is the type Rust uses when it infers an integer without other context.

```rust
let x = 42; // Rust infers i32
```

### 3.3 Integer Literals

Rust supports multiple ways to write integer literals:

```rust
// Decimal (base 10) — the normal way
let a = 1_000_000; // underscores are visual separators, ignored by compiler

// Hexadecimal (base 16) — prefix 0x
let b = 0xFF;       // = 255 in decimal
let c = 0xDEAD_BEEF; // visual separator works here too

// Octal (base 8) — prefix 0o
let d = 0o77;       // = 63 in decimal

// Binary (base 2) — prefix 0b
let e = 0b1111_0000; // = 240 in decimal

// Byte (u8 only) — prefix b and single quote
let f: u8 = b'A';   // = 65 (ASCII value of 'A')
```

**Type suffixes** — you can embed the type directly in the literal:

```rust
let g = 42u8;   // u8 with value 42
let h = 100i64; // i64 with value 100
let i = 255_u32; // u32 with value 255 (underscore before suffix is fine)
```

### 3.4 Integer Overflow

What happens when a value exceeds its type's maximum?

```rust
let x: u8 = 255;
let y = x + 1; // u8 can't hold 256!
```

**In debug mode** (default `cargo build` / `cargo run`):
Rust **panics** (crashes with an error message) — this is intentional to catch bugs early.
```
thread 'main' panicked at 'attempt to add with overflow'
```

**In release mode** (`cargo build --release`):
Rust performs **two's complement wrapping** — the value wraps around.
```
255u8 + 1 = 0   (wraps back to minimum)
```

**Explicit overflow handling methods:**

```rust
let x: u8 = 250;

// wrapping_add — always wraps (no panic)
let a = x.wrapping_add(10);   // 250 + 10 = 260, wraps to 4

// checked_add — returns Option<T>, None if overflow
let b = x.checked_add(10);    // Some(260)? No → None (overflow)
let c = x.checked_add(4);     // Some(254)

// saturating_add — clamps to the boundary
let d = x.saturating_add(10); // 255 (clamped to max, not overflowed)

// overflowing_add — returns (value, did_overflow)
let (val, overflowed) = x.overflowing_add(10); // (4, true)
```

These methods give you **explicit, intentional control** over edge cases.

### 3.5 Integer Methods

```rust
fn main() {
    let x: i32 = -42;
    let y: i32 = 17;

    println!("{}", x.abs());          // 42 — absolute value
    println!("{}", x.pow(2));         // 1764 — raise to power
    println!("{}", y.min(10));        // 10 — smaller of the two
    println!("{}", y.max(100));       // 100 — larger of the two
    println!("{}", y.clamp(0, 10));   // 10 — clamp within range [0, 10]
    println!("{}", i32::MAX);         // 2147483647 — type's max value
    println!("{}", i32::MIN);         // -2147483648 — type's min value
    println!("{}", u8::MAX);          // 255

    // Integer division (truncates toward zero)
    println!("{}", 17 / 5);  // 3 (not 3.4)
    println!("{}", -17 / 5); // -3 (truncates toward zero, not -4)

    // Remainder / modulo
    println!("{}", 17 % 5);  // 2
    println!("{}", -17 % 5); // -2 (sign follows dividend in Rust)
}
```

---

## 4. Floating-Point Types

Floating-point types represent **numbers with decimal points** (real numbers). Rust follows the **IEEE 754** standard for floating-point arithmetic.

### 4.1 f32 vs f64

| Type | Bits | Bytes | Precision | Use Case |
|------|------|-------|-----------|----------|
| `f32` | 32 | 4 | ~7 significant decimal digits | Memory-constrained, GPU, graphics |
| `f64` | 64 | 8 | ~15 significant decimal digits | General use — **default** |

```rust
let x = 3.14;       // f64 — Rust's default for floats
let y: f32 = 3.14;  // f32 — explicitly 32-bit
let z: f64 = 3.14;  // f64 — explicitly 64-bit
```

**Default float type is `f64`** — it's more precise and on modern 64-bit CPUs, roughly the same speed as `f32`. Use `f64` unless you have a specific reason for `f32`.

### 4.2 Floating-Point Precision

This is one of the most important (and often misunderstood) aspects of floats:

```rust
fn main() {
    let x: f64 = 0.1 + 0.2;
    println!("{x}");         // 0.30000000000000004 — NOT 0.3!
    println!("{x:.1}");      // 0.3 (rounded to 1 decimal for display)

    // NEVER compare floats with == for equality!
    println!("{}", 0.1 + 0.2 == 0.3); // false ← infamous float trap
}
```

**Why does this happen?**  
Computers store numbers in binary (base 2). The fraction `0.1` cannot be represented *exactly* in binary — just like `1/3` cannot be written exactly in decimal (0.333...). The stored value is a *very close approximation*.

**The right way to compare floats:**

```rust
fn main() {
    let a = 0.1_f64 + 0.2;
    let b = 0.3_f64;

    let epsilon = 1e-10; // a tiny tolerance value
    let are_equal = (a - b).abs() < epsilon;

    println!("Are equal (within tolerance): {are_equal}"); // true
}
```

### 4.3 Special Float Values

IEEE 754 defines several special float values:

```rust
fn main() {
    // Infinity
    let pos_inf: f64 = f64::INFINITY;
    let neg_inf: f64 = f64::NEG_INFINITY;
    println!("{pos_inf}");               // inf
    println!("{neg_inf}");               // -inf
    println!("{}", 1.0_f64 / 0.0);      // inf (note: integer 1/0 panics!)

    // NaN — Not a Number
    let nan: f64 = f64::NAN;
    println!("{nan}");                   // NaN
    println!("{}", 0.0_f64 / 0.0);      // NaN

    // NaN is NEVER equal to anything — not even itself!
    println!("{}", nan == nan);          // false ← critical gotcha
    println!("{}", nan.is_nan());        // true  ← correct way to check

    // Checking for infinity
    println!("{}", pos_inf.is_infinite()); // true
    println!("{}", pos_inf.is_finite());   // false
    println!("{}", (1.5_f64).is_finite()); // true

    // Constants
    println!("{}", f64::MAX);            // 1.7976931348623157e308
    println!("{}", f64::MIN_POSITIVE);   // 5e-324 (smallest positive f64)
}
```

### 4.4 Float Methods

```rust
fn main() {
    let x: f64 = 2.0;
    let y: f64 = -3.7;

    println!("{}", x.sqrt());        // 1.4142... — square root
    println!("{}", x.cbrt());        // 1.2599... — cube root
    println!("{}", x.powi(3));       // 8.0 — integer exponent
    println!("{}", x.powf(0.5));     // 1.4142... — float exponent (same as sqrt)
    println!("{}", x.exp());         // 7.389... — e^x
    println!("{}", x.ln());          // 0.693... — natural log
    println!("{}", x.log2());        // 1.0 — log base 2
    println!("{}", x.log10());       // 0.301... — log base 10
    println!("{}", y.abs());         // 3.7 — absolute value
    println!("{}", y.ceil());        // -3.0 — round UP toward +∞
    println!("{}", y.floor());       // -4.0 — round DOWN toward -∞
    println!("{}", y.round());       // -4.0 — round to nearest integer
    println!("{}", y.trunc());       // -3.0 — truncate toward zero
    println!("{}", y.fract());       // -0.7 — fractional part
    println!("{}", x.min(y));        // -3.7
    println!("{}", x.max(y));        // 2.0

    // Trigonometry (in radians)
    let angle = std::f64::consts::PI / 4.0; // 45 degrees in radians
    println!("{:.4}", angle.sin());  // 0.7071
    println!("{:.4}", angle.cos());  // 0.7071
    println!("{:.4}", angle.tan());  // 1.0000
}
```

---

## 5. Numeric Operations

Rust supports all standard arithmetic operators:

```rust
fn main() {
    // Integer arithmetic
    let sum        = 5 + 3;    // 8
    let difference = 10 - 4;   // 6
    let product    = 3 * 7;    // 21
    let quotient   = 10 / 3;   // 3  (integer division — truncates)
    let remainder  = 10 % 3;   // 1

    // Float arithmetic
    let fsum        = 5.0 + 3.0;   // 8.0
    let fdifference = 10.0 - 4.0;  // 6.0
    let fproduct    = 3.0 * 7.0;   // 21.0
    let fquotient   = 10.0 / 3.0;  // 3.333...
    let fremainder  = 10.0 % 3.0;  // 1.0

    // ❌ CANNOT mix integer and float in operations
    // let bad = 5 + 3.0; // ERROR: mismatched types

    // ✅ Must cast explicitly
    let good = 5 as f64 + 3.0; // 8.0

    println!("{sum} {difference} {product} {quotient} {remainder}");
    println!("{fsum} {fdifference} {fproduct} {fquotient} {fremainder}");
    println!("{good}");
}
```

### Compound assignment operators

```rust
fn main() {
    let mut x = 10;
    x += 5;  // x = x + 5  → 15
    x -= 3;  // x = x - 3  → 12
    x *= 2;  // x = x * 2  → 24
    x /= 4;  // x = x / 4  → 6
    x %= 4;  // x = x % 4  → 2
    println!("{x}"); // 2
}
```

### Operator precedence (highest to lowest)

```
1.  Unary: -x, !x
2.  as (casting)
3.  * / %
4.  + -
5.  << >>  (bitwise shift)
6.  &      (bitwise AND)
7.  ^      (bitwise XOR)
8.  |      (bitwise OR)
9.  == != < > <= >=
10. &&     (logical AND)
11. ||     (logical OR)
12. .. ..= (range)
13. =, +=, -=, etc.
```

Use parentheses when in doubt — it's always clearer:
```rust
let result = (2 + 3) * 4;  // 20, not 14
```

---

## 6. The Boolean Type

The `bool` type has exactly **two possible values**: `true` and `false`.

```rust
let is_running: bool = true;
let has_error: bool = false;
let is_valid = true; // type inferred as bool
```

Booleans occupy **1 byte** in memory (even though they only need 1 bit — Rust aligns to byte boundaries for performance).

### 6.1 Boolean Literals

```rust
let t = true;
let f = false;
```

That's it — no `1`/`0`, no `"true"`/`"false"` strings. Just the keywords `true` and `false`.

### 6.2 Comparison Operators

These evaluate to a `bool`:

```rust
fn main() {
    let a = 10;
    let b = 20;

    println!("{}", a == b);  // false — equal to
    println!("{}", a != b);  // true  — not equal to
    println!("{}", a < b);   // true  — less than
    println!("{}", a > b);   // false — greater than
    println!("{}", a <= b);  // true  — less than or equal to
    println!("{}", a >= b);  // false — greater than or equal to
}
```

### 6.3 Logical Operators

```rust
fn main() {
    let sunny = true;
    let warm = false;
    let windy = true;

    // AND — both must be true
    println!("{}", sunny && warm);  // false

    // OR — at least one must be true
    println!("{}", sunny || warm);  // true

    // NOT — flips the boolean
    println!("{}", !sunny);         // false
    println!("{}", !warm);          // true

    // Combining
    println!("{}", sunny && !windy); // false (sunny=true, !windy=false)
    println!("{}", (sunny || warm) && !windy); // false
}
```

### Short-circuit evaluation

Rust uses **short-circuit evaluation** for `&&` and `||`:
- `&&`: if the left side is `false`, the right side is **never evaluated**
- `||`: if the left side is `true`, the right side is **never evaluated**

```rust
fn expensive_check() -> bool {
    println!("expensive_check was called");
    true
}

fn main() {
    let x = false && expensive_check(); // expensive_check is NEVER called
    let y = true  || expensive_check(); // expensive_check is NEVER called
    println!("{x} {y}"); // false true
}
```

This matters for performance and for avoiding side effects in conditions.

### 6.4 Boolean in Control Flow

Booleans are the backbone of control flow:

```rust
fn main() {
    let temperature = 38;
    let is_fever = temperature > 37;

    if is_fever {
        println!("You have a fever!");
    } else {
        println!("Temperature is normal.");
    }

    // if as an expression (returns a value)
    let status = if is_fever { "sick" } else { "healthy" };
    println!("Status: {status}");
}
```

> **Note:** Unlike C, C++, or JavaScript, Rust does **not** allow integers where booleans are expected. `if 1 { ... }` is a **compile error** in Rust.

---

## 7. The Character Type

The `char` type represents a **single Unicode scalar value**.

```rust
let letter: char = 'a';
let digit: char = '7';
let emoji: char = '😀';
let chinese: char = '中';
let arabic: char = 'أ';
```

Notice: `char` uses **single quotes** `'`, while string literals use **double quotes** `"`.

A `char` in Rust is **4 bytes** (32 bits) — large enough to hold any Unicode scalar value.

### 7.1 char vs String

```rust
let c: char = 'A';    // a single character — 4 bytes
let s: &str = "A";    // a string (even with 1 char) — different type!

// They are NOT interchangeable
// let bad: char = "A"; // ❌ ERROR
```

| Feature | `char` | `&str` / `String` |
|---------|--------|------------------|
| Quotes | Single `'A'` | Double `"A"` |
| Size | Always 4 bytes | Variable |
| Contains | Exactly 1 Unicode scalar | Zero or more Unicode scalars |
| Usage | Individual characters | Text / words / sentences |

### 7.2 Unicode Scalars

A **Unicode scalar value** is any Unicode code point except surrogate pairs (U+0000 to U+D7FF and U+E000 to U+10FFFF). Rust's `char` covers the full range.

```rust
fn main() {
    // Unicode escape inside char
    let heart  = '\u{2764}';  // ❤
    let thumbs = '\u{1F44D}'; // 👍
    let pi_sym = '\u{03C0}';  // π

    println!("{heart} {thumbs} {pi_sym}");

    // Get the numeric code point
    let a = 'A';
    println!("{}", a as u32); // 65 — ASCII / Unicode code point

    // Create a char from a code point
    let b = char::from(66u8); // 'B'
    println!("{b}");
}
```

### 7.3 char Methods

```rust
fn main() {
    let c = 'R';
    let d = '5';
    let e = ' ';
    let f = 'é';

    // Classification
    println!("{}", c.is_alphabetic());    // true
    println!("{}", d.is_numeric());       // true
    println!("{}", d.is_alphanumeric());  // true
    println!("{}", e.is_whitespace());    // true
    println!("{}", c.is_uppercase());     // true
    println!("{}", c.is_lowercase());     // false
    println!("{}", c.is_ascii());         // true
    println!("{}", f.is_ascii());         // false (é is not ASCII)

    // Case conversion
    println!("{}", c.to_lowercase().next().unwrap()); // 'r'
    println!("{}", 'r'.to_uppercase().next().unwrap()); // 'R'

    // Digit value
    println!("{:?}", '7'.to_digit(10)); // Some(7) — base 10
    println!("{:?}", 'F'.to_digit(16)); // Some(15) — base 16 (hex)
    println!("{:?}", 'G'.to_digit(16)); // None — G is not a hex digit

    // Convert to string
    let s = c.to_string();
    println!("{s}"); // "R"

    // Length in UTF-8 bytes (not always 1!)
    println!("{}", 'A'.len_utf8()); // 1
    println!("{}", 'é'.len_utf8()); // 2
    println!("{}", '中'.len_utf8()); // 3
    println!("{}", '😀'.len_utf8()); // 4
}
```

---

## 8. Type Casting

Rust does **not** perform implicit type conversions. You must be **explicit** about converting between numeric types using the `as` keyword.

```rust
fn main() {
    let x: i32 = 42;

    // Cast to other numeric types
    let as_f64 = x as f64;    // 42.0
    let as_u8  = x as u8;     // 42
    let as_i64 = x as i64;    // 42

    println!("{as_f64} {as_u8} {as_i64}");
}
```

### Casting can be lossy — know the rules

```rust
fn main() {
    // Float → Integer: truncates (removes decimal, doesn't round)
    let f: f64 = 3.99;
    let i = f as i32; // 3 (NOT 4 — truncation, not rounding)
    println!("{i}");

    // Float → Integer: saturates at boundaries
    let big: f64 = 1_000_000.0;
    let small = big as u8; // 255 (saturates at u8::MAX, since Rust 1.45)
    println!("{small}");

    // Integer → smaller integer: truncates high bits
    let large: i32 = 300;
    let byte = large as u8; // 44 — 300 in binary is 100101100, u8 takes low 8 bits: 00101100 = 44
    println!("{byte}");

    // Signed → Unsigned (same size): reinterprets bits
    let neg: i8 = -1;
    let pos = neg as u8; // 255 — bit pattern of -1 in i8 is 11111111 = 255 as u8
    println!("{pos}");

    // char ↔ integer
    let c = 'A';
    let n = c as u32;        // 65
    let back = n as u8 as char; // 'A'
    println!("{n} {back}");
}
```

### Safer alternatives to `as`

For numeric conversions that might fail, use `From`/`Into` or `TryFrom`/`TryInto`:

```rust
fn main() {
    // From/Into — infallible (compile error if conversion can lose data)
    let x: i32 = 42;
    let y: i64 = i64::from(x);  // i32 → i64 is always safe
    let z: i64 = x.into();      // same thing using .into()

    // TryFrom/TryInto — fallible (returns Result)
    let big: i32 = 1000;
    let result = u8::try_from(big); // Err — 1000 doesn't fit in u8
    println!("{:?}", result);       // Err(TryFromIntError(()))

    let small: i32 = 100;
    let result2 = u8::try_from(small); // Ok — 100 fits in u8
    println!("{:?}", result2);         // Ok(100)
}
```

---

## 9. Type Aliases

You can create an **alias** (alternative name) for an existing type using `type`:

```rust
type Meters = f64;
type Kilograms = f64;
type UserId = u64;

fn calculate_bmi(weight: Kilograms, height: Meters) -> f64 {
    weight / (height * height)
}

fn main() {
    let weight: Kilograms = 70.0;
    let height: Meters = 1.75;
    let bmi = calculate_bmi(weight, height);
    println!("BMI: {:.1}", bmi); // BMI: 22.9

    let user: UserId = 100_000;
    println!("User ID: {user}");
}
```

> **Important:** Type aliases are just **alternate names** — `Meters` and `f64` are the **same type**. The compiler won't prevent you from mixing them. For true type safety (preventing `Meters` from being used where `Kilograms` is expected), you need **newtype wrappers** (an advanced topic).

---

## 10. Choosing the Right Type

Here's practical guidance for picking types in real code:

### For integers:
- **Default to `i32`** — it's the most common, idiomatic, and performant
- Use **`u8`** for byte data, color channels (0–255), small counts
- Use **`u32`** for non-negative values that won't overflow 4 billion
- Use **`i64` / `u64`** for large numbers (file sizes, timestamps, counts)
- Use **`usize`** for indexing into arrays/vectors
- Use **`isize`** for pointer-sized differences
- Use **`u128` / `i128`** sparingly — for cryptography or very large numbers

### For floats:
- **Default to `f64`** — more precise, same speed as f32 on most hardware
- Use **`f32`** for large arrays of floats (graphics, ML) where memory matters

### For booleans:
- Always use `bool` for true/false values — never use integers for this in Rust

### For characters:
- Use `char` when you genuinely need to work with a **single** Unicode character
- Use `&str` or `String` for text (even single-character strings if it's in a text context)

```rust
// Clear, idiomatic type choices:
let pixel_red: u8 = 255;         // color channel (0-255)
let user_count: u32 = 1_500_000; // non-negative count
let temperature: f64 = 36.6;     // measurement
let is_logged_in: bool = true;   // flag
let grade: char = 'A';           // single character
let index: usize = 0;            // array index
```

---

## 11. Summary Cheat Sheet

```rust
// ─────────────────────────────────────────────────────
// INTEGERS
// ─────────────────────────────────────────────────────
let a: i32  = -42;         // signed 32-bit (DEFAULT integer)
let b: u8   = 255;         // unsigned 8-bit  (0 to 255)
let c: i64  = 9_000_000;   // large signed, underscores for readability
let d: usize = 0;          // platform-sized (for indexing)

// Literals
let hex  = 0xFF;           // 255
let bin  = 0b1010;         // 10
let oct  = 0o17;           // 15
let byte: u8 = b'A';       // 65

// Overflow handling
let w = 250u8.wrapping_add(10);     // 4   (wraps)
let s = 250u8.saturating_add(10);  // 255 (clamps)
let c = 250u8.checked_add(10);     // None

// ─────────────────────────────────────────────────────
// FLOATS
// ─────────────────────────────────────────────────────
let x: f64 = 3.14;         // 64-bit (DEFAULT float)
let y: f32 = 3.14;         // 32-bit

// Never compare with ==, use epsilon
let close = (a - b).abs() < 1e-10;

// Special values
let inf = f64::INFINITY;
let nan = f64::NAN;
nan.is_nan();              // true (ONLY way to check NaN)

// ─────────────────────────────────────────────────────
// BOOLEANS
// ─────────────────────────────────────────────────────
let flag: bool = true;
let other = false;
let result = flag && !other;  // true
let either = flag || other;   // true

// ─────────────────────────────────────────────────────
// CHARACTERS
// ─────────────────────────────────────────────────────
let ch: char = 'R';        // single quotes
let emoji = '🦀';          // full Unicode support (4 bytes)
ch.is_alphabetic();        // true
ch.is_uppercase();         // true
'7'.to_digit(10);          // Some(7)

// ─────────────────────────────────────────────────────
// TYPE CASTING
// ─────────────────────────────────────────────────────
let n: i32 = 42;
let f = n as f64;          // 42.0
let b = 3.9_f64 as i32;    // 3 (truncates)
let safe = i64::from(n);   // infallible widening
let try_it = u8::try_from(n); // Ok(42) or Err if doesn't fit
```

---

## 🔗 What's Next?

- **Lesson 03 — Data Types: Compound** (tuples and arrays)
- **Lesson 04 — Functions** (parameters, return types, expressions)
- **Lesson 05 — Control Flow** (if, loop, while, for)

---

## 📚 Further Reading

- [The Rust Book — Chapter 3.2: Data Types](https://doc.rust-lang.org/book/ch03-02-data-types.html)
- [Rust by Example — Primitives](https://doc.rust-lang.org/rust-by-example/primitives.html)
- [Rust Reference — Numeric Types](https://doc.rust-lang.org/reference/types/numeric.html)
- [IEEE 754 Floating Point](https://en.wikipedia.org/wiki/IEEE_754) — understanding float behavior

---

*Keep going, Rustacean! 🦀 Types are the foundation of everything.*
