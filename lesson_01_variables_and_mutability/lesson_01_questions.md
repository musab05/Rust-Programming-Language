# 🧪 Lesson 01 — Practice Questions: Variables & Mutability

> **Instructions:** Try to solve each problem yourself before looking at the answers.  
> Answers are in [`lesson_01_answers.md`](./lesson_01_answers.md)

---

## Section A — Conceptual Questions (Fix the Code)

These snippets have bugs. Identify the problem and fix them.

---

### Q1 — Fix the Mutation Error

```rust
fn main() {
    let score = 0;
    score += 10;
    score += 5;
    println!("Score: {score}");
}
```

**What's wrong, and how do you fix it?**

---

### Q2 — Fix the Constant Declaration

```rust
fn main() {
    const LIMIT = 500;
    println!("Limit is: {LIMIT}");
}
```

**This won't compile. Why? Fix it.**

---

### Q3 — Will This Compile?

```rust
fn main() {
    let mut x = 10;
    x = "hello";
    println!("{x}");
}
```

**Yes or no? Explain why.**

---

### Q4 — What Does This Print?

```rust
fn main() {
    let x = 5;
    let x = x + 10;
    let x = x * 2;
    println!("{x}");
}
```

**What is the output? Which Rust feature is being used?**

---

### Q5 — Shadowing Type Change

```rust
fn main() {
    let value = "42";
    let value = value.len();
    println!("{value}");
}
```

**What does this print? Would this work with `mut` instead of shadowing? Explain.**

---

### Q6 — Scope Puzzle

```rust
fn main() {
    let x = 1;
    {
        let x = x + 10;
        println!("Inner: {x}");
    }
    println!("Outer: {x}");
}
```

**What are the two printed values? Explain what happens to the inner `x`.**

---

### Q7 — Will This Compile?

```rust
fn main() {
    {
        let inner = 99;
    }
    println!("{inner}");
}
```

**Yes or no? Explain.**

---

## Section B — Write It Yourself

Write complete, working Rust programs for each of the following.

---

### Q8 — Temperature Converter

Write a program that:
- Declares a constant `FREEZING_POINT` with the value `32.0` (as `f64`)
- Declares a variable `celsius` with the value `100.0`
- Converts `celsius` to Fahrenheit using the formula: `F = (C × 9/5) + 32`
- Prints both values in a sentence like: `"100°C is 212°F"`

---

### Q9 — Mutable Counter

Write a program that:
- Starts a mutable variable `count` at `0`
- Increments it by `1` five times
- After each increment, prints the current value
- Final output should show `count` going from `1` to `5`

---

### Q10 — Shadowing Pipeline

Write a program that uses **shadowing (not mutation)** to transform a value through 3 steps:
1. Start with a `&str` value: `"  hello world  "`
2. Shadow it with the trimmed version (use `.trim()`)
3. Shadow it again with the length of the trimmed string
4. Print the final numeric value

> **Hint:** `.trim()` removes leading/trailing whitespace from a `&str`

---

### Q11 — User Age Validation

Write a program that:
- Declares a variable `age: i32` with value `15`
- Uses a mutable boolean `is_adult` initialized to `false`
- If `age >= 18`, set `is_adult` to `true`
- Print: `"Is adult: true"` or `"Is adult: false"` based on the value

---

### Q12 — Constants in Calculation

Write a program using **at least two constants** to calculate something meaningful.  
For example: calculate the area of a circle given a constant `PI` and a variable `radius`.

Requirements:
- At least 2 `const` values
- At least 1 `let` variable
- Print a descriptive result

---

### Q13 — Swap Without a Third Variable (Using Shadowing)

Given:
```rust
let a = 100;
let b = 200;
```

Using only **shadowing**, make it so that at the end of your program:
- A variable named `a` holds `200`
- A variable named `b` holds `100`
- Print both

> **Note:** Shadowing creates a new binding — think carefully about how to use this.

---

## Section C — Deeper Thinking

---

### Q14 — `const` vs `let` Decision

For each of the following, decide: should it be `const` or `let`? Why?

1. The value of π (3.14159...)
2. A loop counter that increments each iteration
3. The maximum allowed login attempts (e.g., 5)
4. A user's name read from input
5. The speed of light in m/s (299,792,458)
6. A running total being accumulated

---

### Q15 — Predict the Output

What does this program print? Trace through it step by step.

```rust
fn main() {
    let x = 2;

    let x = x * 3;

    {
        let x = x + 100;
        let x = x / 2;
        println!("Block x: {x}");
    }

    println!("Final x: {x}");
}
```

---

### Q16 — Debug This Program

The following program is supposed to calculate the area and perimeter of a rectangle, then double the area. It has **multiple bugs**. Find and fix all of them.

```rust
fn main() {
    const width = 10;
    const height = 5;

    let area = width * height;
    let perimeter = 2 * width + height;

    println!("Area: {area}");
    println!("Perimeter: {perimeter}");

    area = area * 2;
    println!("Doubled area: {area}");
}
```

**List every bug you find, explain each one, then write the corrected version.**

---

### Q17 — Write a Mini Program: Shopping Cart

Write a Rust program simulating a simple shopping cart:

- Define a constant `TAX_RATE: f64 = 0.08` (8%)
- Start with a mutable `total: f64 = 0.0`
- Add three item prices of your choice (using `+=`)
- Calculate the tax using shadowing: shadow `total` to include tax
- Print the subtotal (before tax) and the final total (after tax)

Example output format:
```
Subtotal: $45.97
Tax (8%): $3.68
Total: $49.65
```

---

### Q18 — Underscore Variables

```rust
fn main() {
    let x = 5;
    let _y = 10;
    let _ = 15;

    println!("{x}");
}
```

1. Which variables will produce a compiler warning and which won't? Why?
2. What is the difference between `_y` and `_`?
3. Can you use `_y` in an expression later? Can you use `_`?

---

*Good luck! Remember — the Rust compiler is your friend, not your enemy. Read its error messages carefully.* 🦀
