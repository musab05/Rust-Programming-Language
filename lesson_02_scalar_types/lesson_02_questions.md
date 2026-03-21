# 🧪 Lesson 02 — Practice Questions: Scalar Types

> **Instructions:** Try every question before checking answers.
> Answers are in [`lesson_02_answers.md`](./lesson_02_answers.md)

---

## Section A — Predict the Output

Read the code carefully and write what you think the output will be. Then run it to verify.

---

### Q1 — Integer Arithmetic

```rust
fn main() {
    let a: i32 = 17;
    let b: i32 = 5;

    println!("{}", a + b);
    println!("{}", a - b);
    println!("{}", a * b);
    println!("{}", a / b);
    println!("{}", a % b);
}
```

**What are the five output lines?**

---

### Q2 — Float Precision Trap

```rust
fn main() {
    let x = 0.1_f64 + 0.2_f64;
    println!("{}", x);
    println!("{}", x == 0.3_f64);
    println!("{:.1}", x);
}
```

**What are the three output lines? Which one surprises you?**

---

### Q3 — Shadowing Across Types

```rust
fn main() {
    let value = 42;
    let value = value as f64;
    let value = value / 10.0;
    println!("{value}");
}
```

**What is the output?**

---

### Q4 — Boolean Logic

```rust
fn main() {
    let a = true;
    let b = false;
    let c = true;

    println!("{}", a && b);
    println!("{}", a || b);
    println!("{}", !a);
    println!("{}", a && b || c);
    println!("{}", a && (b || c));
    println!("{}", !(a && c));
}
```

**What are the six output lines?**

---

### Q5 — Char Arithmetic

```rust
fn main() {
    let c = 'A';
    let n = c as u32;
    println!("{n}");

    let next = (n + 1) as u8 as char;
    println!("{next}");

    let lower = c.to_lowercase().next().unwrap();
    println!("{lower}");
}
```

**What are the three output lines?**

---

### Q6 — Overflow Methods

```rust
fn main() {
    let x: u8 = 250;

    println!("{}", x.wrapping_add(10));
    println!("{}", x.saturating_add(10));
    println!("{:?}", x.checked_add(10));
    println!("{:?}", x.checked_add(4));

    let (val, did_overflow) = x.overflowing_add(10);
    println!("{val} {did_overflow}");
}
```

**What are the five output lines?**

---

### Q7 — Type Casting Rules

```rust
fn main() {
    let f: f64 = 9.99;
    let i = f as i32;
    println!("{i}");

    let neg: i8 = -1;
    let u = neg as u8;
    println!("{u}");

    let big: i32 = 300;
    let small = big as u8;
    println!("{small}");
}
```

**What are the three output lines? Explain each.**

---

### Q8 — Special Float Values

```rust
fn main() {
    let nan = f64::NAN;
    let inf = f64::INFINITY;

    println!("{}", nan == nan);
    println!("{}", nan.is_nan());
    println!("{}", inf > 1_000_000.0);
    println!("{}", inf.is_infinite());
    println!("{}", (1.0_f64).is_finite());
}
```

**What are the five output lines?**

---

## Section B — Fix the Bugs

Each snippet has one or more errors. Find them, explain them, and fix them.

---

### Q9 — Mixed Types

```rust
fn main() {
    let x: i32 = 10;
    let y: f64 = 3.0;
    let result = x + y;
    println!("{result}");
}
```

---

### Q10 — Broken Boolean Check

```rust
fn main() {
    let score = 85;
    if score {
        println!("Has a score");
    }
}
```

---

### Q11 — Wrong Quotes

```rust
fn main() {
    let initial: char = "R";
    println!("{initial}");
}
```

---

### Q12 — Const Without Type

```rust
fn main() {
    const GRAVITY = 9.81;
    println!("{GRAVITY}");
}
```

---

### Q13 — Comparing Float with ==

The following code compiles but gives a logically wrong result. Fix it to work correctly:

```rust
fn main() {
    let result = 0.1_f64 + 0.2_f64;
    let expected = 0.3_f64;

    if result == expected {
        println!("Correct!");
    } else {
        println!("Wrong!"); // this always runs — fix the comparison
    }
}
```

---

### Q14 — Integer Overflow

```rust
fn main() {
    let max: u8 = 255;
    let overflowed = max + 1;
    println!("{overflowed}");
}
```

**What happens here in debug mode? Rewrite it using two different overflow-handling methods.**

---

## Section C — Write It Yourself

---

### Q15 — Temperature Converter (All Scales)

Write a program that:
- Takes a temperature value `celsius: f64 = 100.0`
- Converts to **Fahrenheit**: `F = (C × 9/5) + 32`
- Converts to **Kelvin**: `K = C + 273.15`
- Prints all three values with 2 decimal places, labelled clearly

---

### Q16 — Digit Classifier

Write a program that takes a `char` variable and:
- Prints whether it is: alphabetic, numeric, alphanumeric, uppercase, lowercase, or whitespace
- Works correctly for each of these test characters: `'A'`, `'7'`, `'!'`, `' '`, `'z'`

Run your program once for each character (you can hardcode them one at a time).

---

### Q17 — Safe Integer Conversion

Write a program that:
- Defines a list of `i32` values: `[-5, 0, 100, 256, 1000, 42]`
- For each value, attempts to convert it to `u8` using `try_from`
- Prints either `"X fits in u8: Y"` or `"X does not fit in u8"` for each

> **Hint:** Use `u8::try_from(value)` and match on `Ok(v)` / `Err(_)`.  
> You'll need a loop: `for val in [-5_i32, 0, 100, 256, 1000, 42] { ... }`

---

### Q18 — Celsius Validity Checker

Write a program that:
- Takes a temperature `temp: f64`
- Uses boolean expressions (no `if` needed, just `let` bindings) to determine:
  - `is_below_absolute_zero`: true if temp < -273.15
  - `is_water_freezing`: true if temp <= 0.0
  - `is_room_temp`: true if temp is between 18.0 and 26.0 (inclusive)
  - `is_boiling`: true if temp >= 100.0
- Prints all four boolean values

Test with: `temp = 22.5`, then `temp = -300.0`, then `temp = 100.0`

---

### Q19 — Bits and Bytes Visualizer

Write a program that takes a `u8` value and:
- Prints its decimal value
- Prints its hexadecimal representation using `{:X}` format
- Prints its binary representation using `{:08b}` format (8 digits, zero-padded)
- Prints its octal representation using `{:o}` format

Test with: `42`, `255`, `0`, `128`

Expected format for `42`:
```
Dec: 42  Hex: 2A  Bin: 00101010  Oct: 52
```

---

### Q20 — Circle Calculator

Write a program that:
- Defines `const PI: f64 = std::f64::consts::PI` (use Rust's built-in PI)
- Takes a `radius: f64`
- Calculates and prints:
  - Area: `π × r²` 
  - Circumference: `2 × π × r`
  - Diameter: `2 × r`
- Prints each result with **4 decimal places**
- Also checks and prints: `is_unit_circle: bool` (true if radius is approximately 1.0)

Test with radius = `5.0`, then radius = `1.0`

---

### Q21 — Char Cipher (ROT13)

ROT13 is a simple cipher that rotates each letter by 13 positions in the alphabet:
- 'A' → 'N', 'B' → 'O', ..., 'N' → 'A', 'Z' → 'M'
- Only affects letters; digits, spaces, punctuation are unchanged
- ROT13 applied twice returns the original letter

Write a function... wait, you haven't learned functions yet. Write the logic directly in `main` for a single character:
- Start with `let input: char = 'R'`
- Calculate its ROT13 value
- Print the result

> **Hint:** Get the ASCII code with `as u32`. 'A'=65, 'Z'=90, 'a'=97, 'z'=122.  
> For uppercase: `((c as u32 - 65 + 13) % 26 + 65) as u8 as char`

---

## Section D — Deep Understanding

---

### Q22 — Size Reasoning

Answer these without running code first. Then verify.

1. What is `u8::MAX`?
2. What is `i8::MIN`?
3. What is `u16::MAX`?
4. If `x: u8 = 200`, what is `x.wrapping_add(100)`?
5. How many bytes does a `char` take?
6. How many bytes does a `bool` take?
7. What is the default integer type in Rust?
8. What is the default float type in Rust?

---

### Q23 — True or False?

For each statement, say True or False and briefly explain why:

1. `let x: i32 = 3.0;` compiles successfully.
2. `let x = -1u8;` compiles successfully.
3. `'A' == "A"` compiles successfully.
4. `1.0_f32 == 1.0_f64` compiles successfully.
5. `let b: bool = 1;` compiles successfully.
6. `f64::NAN == f64::NAN` evaluates to `true`.
7. `255u8 + 1` panics in debug mode.
8. `3.9_f64 as i32` evaluates to `4`.
9. `b'Z'` is a valid `u8` literal.
10. `'\u{1F600}'` is a valid `char` literal.

---

### Q24 — The Big Debug Challenge

The following program has **7 bugs**. Find and fix all of them, then run the corrected program.

```rust
fn main() {
    // Bug hunt — find all 7 issues!

    const max_health = 100;

    let player_health: u8 = 150;  // players start at 150? suspicious...
    let mut score = 0;

    let is_alive = player_health > 0
    let has_full_health = player_health == max_health;

    let damage: i32 = 30;
    let remaining = player_health - damage;

    let grade: char = "S";

    println!("Alive: {is_alive}");
    println!("Full health: {has_full_health}");
    println!("Remaining HP: {remaining}");
    println!("Grade: {grade}");
    println!("Max: {max_health}");
}
```

**List each bug, explain it, and provide the fully corrected program.**

---

### Q25 — Design Thinking: Pick the Right Types

For each variable below, pick the **most appropriate Rust type** and briefly justify it:

1. The age of a person (0–150)
2. The exact value of π for scientific computation
3. A flag indicating whether a user is online
4. A student's letter grade ('A', 'B', 'C', 'D', 'F')
5. A Unix timestamp (seconds since 1970, large positive number)
6. A pixel's red channel value (0–255)
7. The index used to access an element in a Vec
8. A bank account balance in dollars (can be negative)
9. The ratio of two measurements (e.g., efficiency = output / input)
10. The number of items in a warehouse (could be in the billions)

---

*Take your time. Read errors carefully. The compiler is teaching you, not fighting you.* 🦀
