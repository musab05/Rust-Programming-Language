# ✅ Lesson 02 — Answers: Scalar Types

> Full solutions with explanations.
> Questions are in [`lesson_02_questions.md`](./lesson_02_questions.md)

---

## Section A — Predict the Output

---

### A1 — Integer Arithmetic

**Output:**

```
22
12
85
3
2
```

**Explanation:**

- `17 + 5 = 22`
- `17 - 5 = 12`
- `17 * 5 = 85`
- `17 / 5 = 3` — integer division **truncates** toward zero (drops the `.4`)
- `17 % 5 = 2` — remainder: `17 = 5×3 + 2`

The key thing to remember: integer division in Rust is **not** float division. `17 / 5` is `3`, not `3.4`.

---

### A2 — Float Precision Trap

**Output:**

```
0.30000000000000004
false
0.3
```

**Explanation:**

- Line 1: `0.1 + 0.2` does **not** produce exactly `0.3` in IEEE 754 binary floating point. The stored result is `0.30000000000000004`.
- Line 2: Because the actual value is `0.30000000000000004`, it is **not equal** to `0.3`. This is the famous float equality trap.
- Line 3: `{:.1}` rounds for **display** only. The underlying value is still `0.30000000000000004`, but it prints as `0.3`.

**Lesson:** Never use `==` to compare floats. Use an epsilon/tolerance check instead.

---

### A3 — Shadowing Across Types

**Output:**

```
4.2
```

**Trace:**

```
let value = 42          → i32: 42
let value = value as f64 → f64: 42.0
let value = value / 10.0 → f64: 42.0 / 10.0 = 4.2
```

Shadowing allowed the type to change from `i32` → `f64`. This is not possible with `mut`.

---

### A4 — Boolean Logic

**Output:**

```
false
true
false
true
true
false
```

**Trace:**

- `a && b` → `true && false` → `false`
- `a || b` → `true || false` → `true`
- `!a` → `!true` → `false`
- `a && b || c` → `(true && false) || true` → `false || true` → `true` (`&&` has higher precedence than `||`)
- `a && (b || c)` → `true && (false || true)` → `true && true` → `true`
- `!(a && c)` → `!(true && true)` → `!true` → `false`

---

### A5 — Char Arithmetic

**Output:**

```
65
B
a
```

**Trace:**

- `'A' as u32` → `65` (ASCII code for 'A')
- `65 + 1 = 66`, cast back: `66 as u8 as char` → `'B'` (ASCII 66 = 'B')
- `'A'.to_lowercase().next().unwrap()` → `'a'`

The `.to_lowercase()` method returns an iterator (because some Unicode characters lowercase to multiple chars), so we call `.next().unwrap()` to get the first (and only) char.

---

### A6 — Overflow Methods

**Output:**

```
4
255
None
Ok(254)
4 true
```

**Trace with `x = 250u8`:**

- `wrapping_add(10)`: `250 + 10 = 260`. `260 - 256 = 4`. Wraps around to `4`.
- `saturating_add(10)`: `250 + 10 = 260 > 255`. Clamps to `255` (maximum u8).
- `checked_add(10)`: Would overflow, returns `None`.
- `checked_add(4)`: `250 + 4 = 254`, fits in u8, returns `Ok(254)`.
- `overflowing_add(10)`: Returns `(4, true)` — the wrapped value AND a flag saying overflow occurred.

---

### A7 — Type Casting Rules

**Output:**

```
9
255
44
```

**Explanations:**

1. `9.99_f64 as i32` → `9`. Casting float to int **truncates** (cuts off decimal part). It does NOT round. `9.99` becomes `9`.

2. `-1_i8 as u8` → `255`. In two's complement, `-1` as an 8-bit signed integer is the bit pattern `11111111`. When reinterpreted as an unsigned 8-bit integer, that same pattern is `255`.

3. `300_i32 as u8` → `44`. Since 1.45, Rust saturates out-of-range float casts, but for **integer-to-integer** `as` casts, it still **truncates bits**. 300 in binary is `100101100`. Taking only the low 8 bits: `00101100` = 44. (256 + 44 = 300, so 300 mod 256 = 44.)

---

### A8 — Special Float Values

**Output:**

```
false
true
true
true
true
```

**Explanations:**

- `nan == nan` → `false`. NaN is **never equal to anything**, including itself. This is defined by the IEEE 754 standard.
- `nan.is_nan()` → `true`. The correct way to check for NaN.
- `inf > 1_000_000.0` → `true`. Infinity is larger than any finite number.
- `inf.is_infinite()` → `true`.
- `(1.0_f64).is_finite()` → `true`. Normal numbers are finite.

---

## Section B — Fix the Bugs

---

### A9 — Mixed Types

**Bug:** Rust does not allow adding `i32` and `f64` directly. There is no implicit type conversion.

```rust
// ❌ Original
let result = x + y; // error: mismatched types

// ✅ Fixed — cast the integer to f64
fn main() {
    let x: i32 = 10;
    let y: f64 = 3.0;
    let result = x as f64 + y; // 13.0
    println!("{result}");
}
```

**Output:** `13`

---

### A10 — Broken Boolean Check

**Bug:** In Rust, `if` conditions must be a `bool`. Unlike C/JavaScript, integers are NOT automatically truthy or falsy.

```rust
// ❌ Original
if score { ... } // error: expected `bool`, found integer

// ✅ Fixed
fn main() {
    let score = 85;
    if score > 0 {
        println!("Has a score");
    }
}
```

**Output:** `Has a score`

---

### A11 — Wrong Quotes

**Bug:** `char` literals use **single quotes** `'`. Double quotes `"` create a string slice `&str`, not a `char`.

```rust
// ❌ Original
let initial: char = "R"; // error: expected `char`, found `&str`

// ✅ Fixed
fn main() {
    let initial: char = 'R';
    println!("{initial}");
}
```

**Output:** `R`

---

### A12 — Const Without Type

**Bug:** Constants **require** an explicit type annotation. The type cannot be inferred.

```rust
// ❌ Original
const GRAVITY = 9.81; // error: missing type for `const` item

// ✅ Fixed
fn main() {
    const GRAVITY: f64 = 9.81;
    println!("{GRAVITY}");
}
```

**Output:** `9.81`

---

### A13 — Comparing Float with ==

**Fix:** Use an epsilon tolerance instead of `==`.

```rust
fn main() {
    let result = 0.1_f64 + 0.2_f64;
    let expected = 0.3_f64;

    let epsilon = 1e-10;
    if (result - expected).abs() < epsilon {
        println!("Correct!");
    } else {
        println!("Wrong!");
    }
}
```

**Output:** `Correct!`

**Why:** `(result - expected).abs()` gives the absolute difference. If it's smaller than our tiny epsilon (`0.0000000001`), we consider the values "equal enough" for practical purposes.

---

### A14 — Integer Overflow

**What happens in debug mode:** The program **panics** at runtime with:

```
thread 'main' panicked at 'attempt to add with overflow'
```

**Fixed versions:**

```rust
fn main() {
    let max: u8 = 255;

    // Method 1: wrapping_add — wraps around to 0
    let wrapped = max.wrapping_add(1);
    println!("Wrapping: {wrapped}"); // 0

    // Method 2: saturating_add — stays at the max (255)
    let saturated = max.saturating_add(1);
    println!("Saturating: {saturated}"); // 255

    // Method 3: checked_add — returns None if overflow
    match max.checked_add(1) {
        Some(val) => println!("Checked: {val}"),
        None => println!("Checked: overflow!"), // this runs
    }
}
```

---

## Section C — Write It Yourself

---

### A15 — Temperature Converter

```rust
fn main() {
    let celsius: f64 = 100.0;

    let fahrenheit = (celsius * 9.0 / 5.0) + 32.0;
    let kelvin = celsius + 273.15;

    println!("Celsius:    {:.2}°C", celsius);
    println!("Fahrenheit: {:.2}°F", fahrenheit);
    println!("Kelvin:     {:.2}K",  kelvin);
}
```

**Output:**

```
Celsius:    100.00°C
Fahrenheit: 212.00°F
Kelvin:     373.15K
```

---

### A16 — Digit Classifier

```rust
fn main() {
    let c: char = 'A'; // change this to test: '7', '!', ' ', 'z'

    println!("Character: '{c}'");
    println!("  is_alphabetic:   {}", c.is_alphabetic());
    println!("  is_numeric:      {}", c.is_numeric());
    println!("  is_alphanumeric: {}", c.is_alphanumeric());
    println!("  is_uppercase:    {}", c.is_uppercase());
    println!("  is_lowercase:    {}", c.is_lowercase());
    println!("  is_whitespace:   {}", c.is_whitespace());
}
```

**Output for `'A'`:**

```
Character: 'A'
  is_alphabetic:   true
  is_numeric:      false
  is_alphanumeric: true
  is_uppercase:    true
  is_lowercase:    false
  is_whitespace:   false
```

**Output for `'7'`:**

```
Character: '7'
  is_alphabetic:   false
  is_numeric:      true
  is_alphanumeric: true
  is_uppercase:    false
  is_lowercase:    false
  is_whitespace:   false
```

---

### A17 — Safe Integer Conversion

```rust
fn main() {
    for val in [-5_i32, 0, 100, 256, 1000, 42] {
        match u8::try_from(val) {
            Ok(v)  => println!("{val} fits in u8: {v}"),
            Err(_) => println!("{val} does not fit in u8"),
        }
    }
}
```

**Output:**

```
-5 does not fit in u8
0 fits in u8: 0
100 fits in u8: 100
256 does not fit in u8
1000 does not fit in u8
42 fits in u8: 42
```

**Why:** `u8` holds values `0..=255`. Any negative number or value above 255 causes `try_from` to return `Err`.

---

### A18 — Celsius Validity Checker

```rust
fn main() {
    let temp: f64 = 22.5; // try: -300.0, 100.0

    let is_below_absolute_zero = temp < -273.15;
    let is_water_freezing       = temp <= 0.0;
    let is_room_temp            = temp >= 18.0 && temp <= 26.0;
    let is_boiling              = temp >= 100.0;

    println!("Temperature: {temp}°C");
    println!("  Below absolute zero: {is_below_absolute_zero}");
    println!("  Water freezing:      {is_water_freezing}");
    println!("  Room temperature:    {is_room_temp}");
    println!("  Boiling:             {is_boiling}");
}
```

**Output for `22.5`:**

```
Temperature: 22.5°C
  Below absolute zero: false
  Water freezing:      false
  Room temperature:    true
  Boiling:             false
```

**Output for `-300.0`:**

```
Temperature: -300°C
  Below absolute zero: true
  Water freezing:      true
  Room temperature:    false
  Boiling:             false
```

**Output for `100.0`:**

```
Temperature: 100°C
  Below absolute zero: false
  Water freezing:      false
  Room temperature:    false
  Boiling:             true
```

---

### A19 — Bits and Bytes Visualizer

```rust
fn main() {
    let values: [u8; 4] = [42, 255, 0, 128];

    for n in values {
        println!("Dec: {n:3}  Hex: {:02X}  Bin: {:08b}  Oct: {:03o}", n, n, n);
    }
}
```

**Output:**

```
Dec:  42  Hex: 2A  Bin: 00101010  Oct: 052
Dec: 255  Hex: FF  Bin: 11111111  Oct: 377
Dec:   0  Hex: 00  Bin: 00000000  Oct: 000
Dec: 128  Hex: 80  Bin: 10000000  Oct: 200
```

**Format specifiers:**

- `{:3}` — right-align in 3-char field
- `{:02X}` — uppercase hex, 2 digits, zero-padded
- `{:08b}` — binary, 8 digits, zero-padded
- `{:03o}` — octal, 3 digits, zero-padded

---

### A20 — Circle Calculator

```rust
fn main() {
    const PI: f64 = std::f64::consts::PI;
    let radius: f64 = 5.0; // try: 1.0

    let area          = PI * radius * radius;
    let circumference = 2.0 * PI * radius;
    let diameter      = 2.0 * radius;

    let epsilon = 1e-10;
    let is_unit_circle = (radius - 1.0).abs() < epsilon;

    println!("Radius:        {radius:.4}");
    println!("Diameter:      {diameter:.4}");
    println!("Circumference: {circumference:.4}");
    println!("Area:          {area:.4}");
    println!("Unit circle:   {is_unit_circle}");
}
```

**Output for `radius = 5.0`:**

```
Radius:        5.0000
Diameter:      10.0000
Circumference: 31.4159
Area:          78.5398
Unit circle:   false
```

**Output for `radius = 1.0`:**

```
Radius:        1.0000
Diameter:      2.0000
Circumference: 6.2832
Area:          3.1416
Unit circle:   true
```

> Note: We use epsilon comparison for `is_unit_circle` — never `== 1.0` for floats.

---

### A21 — Char Cipher (ROT13)

```rust
fn main() {
    let input: char = 'R';

    let result: char = if input.is_ascii_uppercase() {
        ((input as u32 - 65 + 13) % 26 + 65) as u8 as char
    } else if input.is_ascii_lowercase() {
        ((input as u32 - 97 + 13) % 26 + 97) as u8 as char
    } else {
        input // non-letter characters are unchanged
    };

    println!("ROT13('{input}') = '{result}'");
}
```

**Output:**

```
ROT13('R') = 'E'
```

**How the formula works for `'R'` (uppercase):**

- `'R' as u32` = 82
- Subtract 'A' offset: `82 - 65 = 17` (0-based index: R is the 17th letter, 0-indexed)
- Add 13: `17 + 13 = 30`
- Wrap with mod 26: `30 % 26 = 4`
- Add 'A' back: `4 + 65 = 69`
- `69 as u8 as char` = `'E'`

**Verification:** R(17) + 13 = 30, 30 mod 26 = 4, which is 'E'. And ROT13('E') should give back 'R': E(4) + 13 = 17, which is 'R'. ✅

---

## Section D — Deep Understanding

---

### A22 — Size Reasoning

| Question                  | Answer  | Reasoning                                      |
| ------------------------- | ------- | ---------------------------------------------- |
| `u8::MAX`                 | `255`   | 2⁸ - 1 = 255                                   |
| `i8::MIN`                 | `-128`  | -(2⁷) = -128                                   |
| `u16::MAX`                | `65535` | 2¹⁶ - 1 = 65535                                |
| `200u8.wrapping_add(100)` | `44`    | 200 + 100 = 300; 300 - 256 = 44                |
| bytes in `char`           | `4`     | Rust `char` is always 4 bytes (Unicode scalar) |
| bytes in `bool`           | `1`     | 1 byte (aligned to byte boundary)              |
| Default integer           | `i32`   | Inferred when no context guides the type       |
| Default float             | `f64`   | More precise; same speed on modern CPUs        |

---

### A23 — True or False?

| #   | Statement                     | Answer    | Explanation                                                                  |
| --- | ----------------------------- | --------- | ---------------------------------------------------------------------------- |
| 1   | `let x: i32 = 3.0;`           | **False** | `3.0` is a float literal; you cannot assign a float to `i32` without casting |
| 2   | `let x = -1u8;`               | **False** | `-1` is negative; `u8` can only hold 0–255. Compile error                    |
| 3   | `'A' == "A"`                  | **False** | `char` and `&str` are different types; comparison won't compile              |
| 4   | `1.0_f32 == 1.0_f64`          | **False** | `f32` and `f64` are different types; won't compile without casting           |
| 5   | `let b: bool = 1;`            | **False** | Rust doesn't allow integers where `bool` is expected                         |
| 6   | `f64::NAN == f64::NAN`        | **False** | NaN is never equal to anything per IEEE 754                                  |
| 7   | `255u8 + 1` panics in debug   | **True**  | Integer overflow panics in debug mode                                        |
| 8   | `3.9_f64 as i32 == 4`         | **False** | `as` truncates (cuts off decimal), it does NOT round. Result is `3`          |
| 9   | `b'Z'` is valid `u8`          | **True**  | Byte literal syntax; `b'Z'` = 90 (ASCII for 'Z')                             |
| 10  | `'\u{1F600}'` is valid `char` | **True**  | U+1F600 is 😀 — a valid Unicode scalar value                                 |

---

### A24 — The Big Debug Challenge

**7 bugs identified:**

```rust
// BUGGY VERSION (annotated)
fn main() {
    const max_health = 100;           // BUG 1: const needs type annotation
                                      // BUG 2: const name should be SCREAMING_SNAKE_CASE

    let player_health: u8 = 150;      // BUG 3: u8 max is 255, 150 fits BUT
                                      // the intent of "max health = 100" suggests
                                      // starting at 100. Actually 150 DOES fit u8,
                                      // but health > max_health is semantically wrong.
                                      // Bigger issue below ↓

    let mut score = 0;                // OK (mut is unused but not a hard error)

    let is_alive = player_health > 0  // BUG 4: missing semicolon
    let has_full_health = player_health == max_health;

    let damage: i32 = 30;
    let remaining = player_health - damage; // BUG 5: type mismatch — u8 - i32

    let grade: char = "S";            // BUG 6: double quotes — should be 'S'

    println!("Alive: {is_alive}");
    println!("Full health: {has_full_health}");
    println!("Remaining HP: {remaining}");
    println!("Grade: {grade}");
    println!("Max: {max_health}");
                                      // BUG 7: `score` is declared mut but never mutated
                                      // (warning, not error — but still a bug in intent)
}
```

**Fully corrected version:**

```rust
fn main() {
    const MAX_HEALTH: u8 = 100;           // ✅ type annotation + SCREAMING_SNAKE_CASE

    let player_health: u8 = 100;          // ✅ start at max health, fits u8
    let mut score: u32 = 0;               // ✅ mut kept, used later if needed

    let is_alive = player_health > 0;     // ✅ semicolon added
    let has_full_health = player_health == MAX_HEALTH;

    let damage: u8 = 30;                  // ✅ match type with player_health
    let remaining = player_health - damage; // u8 - u8 = u8 ✅

    let grade: char = 'S';                // ✅ single quotes for char

    score += 1; // ✅ actually use the mutable variable

    println!("Alive: {is_alive}");
    println!("Full health: {has_full_health}");
    println!("Remaining HP: {remaining}");
    println!("Grade: {grade}");
    println!("Max: {MAX_HEALTH}");
    println!("Score: {score}");
}
```

**Output:**

```
Alive: true
Full health: true
Remaining HP: 70
Grade: S
Max: 100
Score: 1
```

---

### A25 — Design Thinking: Pick the Right Types

| #   | Variable                        | Best Type      | Justification                                                                          |
| --- | ------------------------------- | -------------- | -------------------------------------------------------------------------------------- |
| 1   | Person's age (0–150)            | `u8`           | Non-negative, max 150 fits in u8 (max 255)                                             |
| 2   | π for scientific computation    | `f64`          | Maximum precision for scientific work                                                  |
| 3   | User online flag                | `bool`         | A yes/no flag is exactly what `bool` is for                                            |
| 4   | Student letter grade            | `char`         | A single character value                                                               |
| 5   | Unix timestamp                  | `u64`          | Large non-negative integer; current timestamp ≈ 1.7 billion, grows over decades        |
| 6   | Pixel red channel               | `u8`           | Exactly 0–255, perfect fit for `u8`                                                    |
| 7   | Vec index                       | `usize`        | Rust's indexing type; required by the compiler for array/vec access                    |
| 8   | Bank account balance            | `f64` or `i64` | `f64` for dollars with cents. In real finance code, `i64` of cents avoids float issues |
| 9   | Efficiency ratio                | `f64`          | A ratio is a real number, needs decimal precision                                      |
| 10  | Warehouse item count (billions) | `u64`          | Billions exceed `u32::MAX` (≈4.3B); `u64` goes up to 18 quintillion                    |

---

## 🏆 Lesson Complete!

You've now practiced:

- ✅ All integer types, sizes, and overflow handling
- ✅ Float precision, special values (NaN, Infinity)
- ✅ Boolean logic and short-circuit evaluation
- ✅ `char` and Unicode scalar values
- ✅ Type casting with `as`, `from`, and `try_from`
- ✅ Choosing appropriate types for real scenarios

**Up next:** [Lesson 03 — Compound Types](../lesson_03_compound_types//lesson_03_compound_types.md) — Tuples and Arrays 🦀
