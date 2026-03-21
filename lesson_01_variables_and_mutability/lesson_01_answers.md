# ✅ Lesson 01 — Answers: Variables & Mutability

> These are the full solutions with explanations.  
> Questions are in [`lesson_01_questions.md`](./lesson_01_questions.md)

---

## Section A — Conceptual Questions (Fix the Code)

---

### A1 — Fix the Mutation Error

**Problem:** `score` is declared without `mut`, so it cannot be reassigned.

**Buggy code:**

```rust
let score = 0;
score += 10; // ❌ cannot assign twice to immutable variable
```

**Fixed code:**

```rust
fn main() {
    let mut score = 0;  // ✅ add `mut`
    score += 10;
    score += 5;
    println!("Score: {score}"); // Score: 15
}
```

**Key takeaway:** In Rust, variables are immutable by default. If a value needs to change, explicitly opt-in with `mut`.

---

### A2 — Fix the Constant Declaration

**Problem:** Constants **require** an explicit type annotation. `const LIMIT = 500;` is invalid syntax.

**Fixed code:**

```rust
fn main() {
    const LIMIT: u32 = 500;  // ✅ type annotation required
    println!("Limit is: {LIMIT}"); // Limit is: 500
}
```

**Key takeaway:** Unlike `let` (where type annotation is optional), `const` always needs a type.

---

### A3 — Will This Compile?

**Answer: No, it will not compile.**

```rust
let mut x = 10;
x = "hello"; // ❌ expected integer, found `&str`
```

Even though `x` is mutable, `mut` only allows you to change the **value**, not the **type**. Once `x` is bound to an integer, it stays an integer for its entire lifetime.

**Compiler error:**

```
error[E0308]: mismatched types
  --> src/main.rs:3:9
   |
3  |     x = "hello";
   |         ^^^^^^^ expected integer, found `&str`
```

**Key takeaway:** `mut` ≠ dynamic typing. Rust is always statically typed.

---

### A4 — What Does This Print?

**Answer: `30`**

**Trace:**

```rust
let x = 5;          // x = 5
let x = x + 10;     // x = 5 + 10 = 15
let x = x * 2;      // x = 15 * 2 = 30
println!("{x}");    // 30
```

**Feature used:** **Shadowing** — each `let x = ...` creates a brand new variable that shadows the previous one.

---

### A5 — Shadowing Type Change

**Answer: It prints `2`**

```rust
let value = "42";       // &str, length is 2 characters
let value = value.len(); // usize, value is 2
println!("{value}");    // 2
```

`"42"` is a 2-character string, so `.len()` returns `2`.

**Would `mut` work here? No.**

```rust
let mut value = "42";
value = value.len(); // ❌ ERROR: expected `&str`, found `usize`
```

`mut` doesn't allow type changes. Shadowing is the correct tool when you're **transforming a value into a different type** while keeping the same conceptual name.

---

### A6 — Scope Puzzle

**Output:**

```
Inner: 11
Outer: 1
```

**Trace:**

```rust
let x = 1;               // outer x = 1

{
    let x = x + 10;      // inner x = 1 + 10 = 11 (shadows outer x)
    println!("Inner: {x}"); // prints 11
}                        // inner x is DROPPED here

println!("Outer: {x}");  // outer x = 1, still alive
```

The inner `let x` creates a separate variable. When the inner block ends, it's dropped. The outer `x` remains unchanged at `1`.

---

### A7 — Will This Compile?

**Answer: No.**

```rust
{
    let inner = 99;
}
println!("{inner}"); // ❌ cannot find value `inner` in this scope
```

`inner` was declared inside the block `{}` and goes out of scope when the block ends. It is no longer accessible after the closing brace.

**Compiler error:**

```
error[E0425]: cannot find value `inner` in this scope
```

---

## Section B — Write It Yourself

---

### A8 — Temperature Converter

```rust
fn main() {
    const FREEZING_POINT: f64 = 32.0;

    let celsius = 100.0_f64;
    let fahrenheit = (celsius * 9.0 / 5.0) + FREEZING_POINT;

    println!("{celsius}°C is {fahrenheit}°F");
    // Output: 100°C is 212°F
}
```

> Note: `FREEZING_POINT` isn't directly used in the formula above (the formula naturally produces 32 for 0°C), but including it demonstrates the intent. An alternative approach:

```rust
let fahrenheit = (celsius * 9.0 / 5.0) + FREEZING_POINT;
```

This is correct — at 100°C: `(100 × 9/5) + 32 = 180 + 32 = 212`.

---

### A9 — Mutable Counter

```rust
fn main() {
    let mut count = 0;

    count += 1;
    println!("Count: {count}"); // Count: 1

    count += 1;
    println!("Count: {count}"); // Count: 2

    count += 1;
    println!("Count: {count}"); // Count: 3

    count += 1;
    println!("Count: {count}"); // Count: 4

    count += 1;
    println!("Count: {count}"); // Count: 5
}
```

**Output:**

```
Count: 1
Count: 2
Count: 3
Count: 4
Count: 5
```

---

### A10 — Shadowing Pipeline

```rust
fn main() {
    let text = "  hello world  ";     // &str with whitespace
    let text = text.trim();           // &str without whitespace → "hello world"
    let text = text.len();            // usize: length of trimmed string

    println!("{text}"); // 11
}
```

`"hello world"` is 11 characters (`h-e-l-l-o-space-w-o-r-l-d`), so the output is `11`.

Each `let text = ...` creates a new variable of a potentially different type. This is only possible because of **shadowing**.

---

### A11 — User Age Validation

```rust
fn main() {
    let age: i32 = 15;
    let mut is_adult = false;

    if age >= 18 {
        is_adult = true;
    }

    println!("Is adult: {is_adult}"); // Is adult: false
}
```

Since `15 < 18`, `is_adult` stays `false`.

> **Bonus — more idiomatic Rust:** You can assign directly from an `if` expression:

```rust
fn main() {
    let age: i32 = 15;
    let is_adult = age >= 18; // evaluates to a bool directly
    println!("Is adult: {is_adult}"); // Is adult: false
}
```

---

### A12 — Constants in Calculation

```rust
fn main() {
    const PI: f64 = 3.14159265358979;
    const RADIUS_UNITS: &str = "cm";

    let radius: f64 = 7.0;
    let area = PI * radius * radius;
    let circumference = 2.0 * PI * radius;

    println!("Circle with radius {radius}{RADIUS_UNITS}:");
    println!("  Area:          {:.2} sq {RADIUS_UNITS}", area);
    println!("  Circumference: {:.2} {RADIUS_UNITS}", circumference);
}
```

**Output:**

```
Circle with radius 7cm:
  Area:          153.94 sq cm
  Circumference: 43.98 cm
```

---

### A13 — Swap Without a Third Variable (Using Shadowing)

```rust
fn main() {
    let a = 100;
    let b = 200;

    // Shadow b first using a's value, then shadow a using b's original value
    let a_original = a;
    let a = b;         // a is now 200
    let b = a_original; // b is now 100

    println!("a = {a}"); // a = 200
    println!("b = {b}"); // b = 100
}
```

> **Note:** The cleanest way to swap in Rust is actually using a tuple, which you'll learn about in the Data Types lesson:

```rust
let (a, b) = (b, a);
```

But using shadowing is a valid exercise in understanding how it creates new bindings.

---

## Section C — Deeper Thinking

---

### A14 — `const` vs `let` Decision

| Value                  | Choice    | Reason                                                      |
| ---------------------- | --------- | ----------------------------------------------------------- |
| π (3.14159...)         | `const`   | Mathematical constant, never changes, known at compile time |
| Loop counter           | `let mut` | Changes every iteration, runtime value                      |
| Max login attempts (5) | `const`   | Fixed policy value, meaningful name, compile-time known     |
| User's name from input | `let`     | Runtime value, not known at compile time                    |
| Speed of light         | `const`   | Physical constant, never changes, compile-time known        |
| Running total          | `let mut` | Accumulates at runtime, needs to change                     |

---

### A15 — Predict the Output

**Output:**

```
Block x: 53
Final x: 6
```

**Step-by-step trace:**

```rust
let x = 2;              // x = 2

let x = x * 3;          // x = 2 * 3 = 6

{
    let x = x + 100;    // inner x = 6 + 100 = 106
    let x = x / 2;      // inner x = 106 / 2 = 53
    println!("Block x: {x}"); // prints 53
}                       // inner x dropped

println!("Final x: {x}"); // outer x is still 6
```

The two inner `let x` statements only exist inside the block. When the block ends, the outer `x` (which is `6`) is still in scope.

---

### A16 — Debug This Program

**Bugs found:**

1. **`const width = 10;`** — Constants require type annotations → `const WIDTH: i32 = 10;`
2. **`const height = 5;`** — Same issue → `const HEIGHT: i32 = 5;`
3. **Naming convention** — Constants should be `SCREAMING_SNAKE_CASE`
4. **`let perimeter = 2 * width + height;`** — Formula is wrong. Perimeter of rectangle = `2 * (width + height)` → `2 * (width + height)`
5. **`area = area * 2;`** — `area` is a `let` binding (immutable). Must be `let mut area` or use shadowing

**Fixed version:**

```rust
fn main() {
    const WIDTH: i32 = 10;
    const HEIGHT: i32 = 5;

    let mut area = WIDTH * HEIGHT;
    let perimeter = 2 * (WIDTH + HEIGHT); // ✅ correct formula

    println!("Area: {area}");             // Area: 50
    println!("Perimeter: {perimeter}");   // Perimeter: 30

    area = area * 2;                      // ✅ now works with mut
    println!("Doubled area: {area}");     // Doubled area: 100
}
```

**Alternatively, using shadowing instead of `mut`:**

```rust
fn main() {
    const WIDTH: i32 = 10;
    const HEIGHT: i32 = 5;

    let area = WIDTH * HEIGHT;
    let perimeter = 2 * (WIDTH + HEIGHT);

    println!("Area: {area}");
    println!("Perimeter: {perimeter}");

    let area = area * 2; // ✅ shadowing
    println!("Doubled area: {area}");
}
```

---

### A17 — Shopping Cart

```rust
fn main() {
    const TAX_RATE: f64 = 0.08;

    let mut total: f64 = 0.0;

    // Add items
    total += 19.99; // Item 1: Rust Programming Book
    total += 14.99; // Item 2: Coffee mug
    total += 10.99; // Item 3: Sticker pack

    let subtotal = total; // save subtotal before tax
    let tax = subtotal * TAX_RATE;

    // Shadow total to include tax
    let total = subtotal + tax;

    println!("Subtotal: ${:.2}", subtotal); // Subtotal: $45.97
    println!("Tax (8%): ${:.2}", tax);      // Tax (8%):  $3.68
    println!("Total:    ${:.2}", total);    // Total:     $49.65
}
```

**Output:**

```
Subtotal: $45.97
Tax (8%): $3.68
Total:    $49.65
```

> `{:.2}` in the format string means "print with 2 decimal places".

---

### A18 — Underscore Variables

```rust
fn main() {
    let x = 5;     // ⚠️ Warning if never used — but we use it below
    let _y = 10;   // ✅ No warning — underscore prefix suppresses it
    let _ = 15;    // ✅ No warning — `_` is a discard pattern

    println!("{x}"); // x is used, so no warning
}
```

**Answers:**

1. **Which produce warnings?**
   - `x` — would produce a warning if never used. Here it's used in `println!`, so no warning.
   - `_y` — no warning, the underscore prefix signals "intentionally unused"
   - `_` — no warning, it's a special discard pattern

2. **Difference between `_y` and `_`:**
   - `_y` is a named variable that is bound to the value `10`. It just won't produce a warning. The value `10` is kept in memory until `_y` goes out of scope.
   - `_` is a **wildcard/discard pattern**. The value is immediately dropped — it's not stored in any named binding. It's used to explicitly throw away a value.

3. **Can you use them later?**
   - `_y` — **Yes**, you can reference `_y` in expressions later: `println!("{_y}")` works fine.
   - `_` — **No**, `_` is not a variable name. You cannot reference it later. Each `let _ = ...` is completely independent.

---

## 🏆 Lesson Complete!

You've practiced:

- ✅ Declaring immutable and mutable variables
- ✅ Understanding `const` and when to use it
- ✅ Applying shadowing for type transformations
- ✅ Understanding variable scope
- ✅ Recognizing and suppressing unused variable warnings
- ✅ Debugging common Rust variable errors

**Move on to** [`lesson_02_scalar_types.md`](../lesson_02_scalar_types/lesson_02_scalar_types.md) when you're ready! 🦀
