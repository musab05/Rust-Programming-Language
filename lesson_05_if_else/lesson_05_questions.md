# 🧪 Lesson 05 — Practice Questions: Control Flow — if / else

> **Instructions:** Work through every question before checking answers.
> Answers are in [`lesson_05_answers.md`](./lesson_05_answers.md)

---

## Section A — Predict the Output

Read each snippet carefully and write the exact output before running it.

---

### Q1 — Basic if / else

```rust
fn main() {
    let x = 42;

    if x > 100 {
        println!("large");
    } else if x > 50 {
        println!("medium");
    } else if x > 0 {
        println!("small");
    } else {
        println!("non-positive");
    }
}
```

**What is the output?**

---

### Q2 — if as an Expression

```rust
fn main() {
    let score = 85;

    let grade = if score >= 90 {
        'A'
    } else if score >= 80 {
        'B'
    } else if score >= 70 {
        'C'
    } else {
        'F'
    };

    println!("Grade: {grade}");
    println!("Pass: {}", if score >= 60 { "Yes" } else { "No" });
}
```

**What are the two output lines?**

---

### Q3 — Boolean Operators

```rust
fn main() {
    let a = true;
    let b = false;
    let c = true;

    println!("{}", a && b);
    println!("{}", a || b);
    println!("{}", !b);
    println!("{}", a && b || c);
    println!("{}", a && (b || c));
    println!("{}", !(a && c) || b);
}
```

**What are the six output lines?**

---

### Q4 — Short-Circuit Evaluation

```rust
fn side_effect(label: &str, val: bool) -> bool {
    println!("evaluated: {label}");
    val
}

fn main() {
    println!("--- Test 1 ---");
    let _ = side_effect("A", false) && side_effect("B", true);

    println!("--- Test 2 ---");
    let _ = side_effect("C", true) || side_effect("D", false);

    println!("--- Test 3 ---");
    let _ = side_effect("E", true) && side_effect("F", false);
}
```

**What is the full output? Explain which calls are skipped and why.**

---

### Q5 — Nested if and Scope

```rust
fn main() {
    let x = 15;
    let y = 8;

    if x > 10 {
        if y > 10 {
            println!("both big");
        } else {
            println!("x big, y small");
        }
    } else {
        println!("x small");
    }

    let result = if x > y {
        x - y
    } else {
        y - x
    };
    println!("difference: {result}");
}
```

**What are the two output lines?**

---

### Q6 — if let with Option

```rust
fn main() {
    let values: [Option<i32>; 4] = [Some(10), None, Some(-5), None];

    for v in values {
        if let Some(n) = v {
            if n > 0 {
                println!("positive: {n}");
            } else {
                println!("non-positive: {n}");
            }
        } else {
            println!("nothing");
        }
    }
}
```

**What are the four output lines?**

---

### Q7 — Guard Clauses

```rust
fn check(n: i32) -> &'static str {
    if n < 0    { return "negative"; }
    if n == 0   { return "zero"; }
    if n > 1000 { return "huge"; }
    if n % 2 == 0 { return "even"; }
    "odd"
}

fn main() {
    println!("{}", check(-10));
    println!("{}", check(0));
    println!("{}", check(2000));
    println!("{}", check(42));
    println!("{}", check(7));
}
```

**What are the five output lines?**

---

### Q8 — Tricky Order of Conditions

```rust
fn classify(n: i32) -> &'static str {
    if n > 0  { "positive" }
    else if n > 50 { "large" }      // ← notice the order
    else if n < 0  { "negative" }
    else           { "zero" }
}

fn main() {
    println!("{}", classify(100));
    println!("{}", classify(50));
    println!("{}", classify(-5));
    println!("{}", classify(0));
}
```

**What are the four outputs? Is there a bug? If so, what is it?**

---

### Q9 — if let with else if let

```rust
fn describe(val: Option<i32>) {
    if let Some(n) = val {
        if n > 0 {
            println!("positive Some: {n}");
        } else if n == 0 {
            println!("zero Some");
        } else {
            println!("negative Some: {n}");
        }
    } else {
        println!("None");
    }
}

fn main() {
    describe(Some(5));
    describe(Some(0));
    describe(Some(-3));
    describe(None);
}
```

**What are the four output lines?**

---

### Q10 — Complex Boolean Expression

```rust
fn can_rent_car(age: u32, has_license: bool, years_driving: u32) -> bool {
    age >= 21 && has_license && years_driving >= 1
}

fn main() {
    println!("{}", can_rent_car(25, true, 3));
    println!("{}", can_rent_car(19, true, 2));
    println!("{}", can_rent_car(22, false, 5));
    println!("{}", can_rent_car(21, true, 0));
    println!("{}", can_rent_car(21, true, 1));
}
```

**What are the five output lines?**

---

## Section B — Fix the Bugs

Find every error, explain it clearly, and write the corrected version.

---

### Q11 — Non-Boolean Condition

```rust
fn main() {
    let count = 5;
    if count {
        println!("has items");
    }
}
```

---

### Q12 — Missing Braces

```rust
fn main() {
    let x = 10;
    if x > 5
        println!("x is greater than 5");
}
```

---

### Q13 — Mismatched Types in if Expression

```rust
fn main() {
    let condition = true;
    let value = if condition {
        42
    } else {
        "forty-two"
    };
    println!("{value}");
}
```

---

### Q14 — Semicolon Ruins the if Expression

```rust
fn main() {
    let n = 7;
    let label = if n % 2 == 0 {
        "even";
    } else {
        "odd";
    };
    println!("{label}");
}
```

**Explain precisely what goes wrong and fix it.**

---

### Q15 — Unreachable Branch

```rust
fn rate(score: u32) -> &'static str {
    if score >= 0  { return "any score"; }
    if score >= 50 { return "pass"; }
    if score >= 80 { return "good"; }
    "fail"
}

fn main() {
    println!("{}", rate(90));
    println!("{}", rate(60));
    println!("{}", rate(30));
}
```

**What is wrong? Fix the function so it returns the right label for each score.**

---

### Q16 — Assignment Inside Condition

```rust
fn main() {
    let mut x = 0;
    if x = 5 {
        println!("x is 5");
    }
}
```

---

### Q17 — if let Type Mismatch

```rust
fn main() {
    let value: Option<&str> = Some("hello");
    if let Some(n) = value {
        let upper = n.to_uppercase();
        println!("{}", upper + 1);
    }
}
```

---

### Q18 — Logic Error in Range Check

```rust
fn is_teen(age: u32) -> bool {
    if age > 12 || age < 20 {
        true
    } else {
        false
    }
}

fn main() {
    println!("{}", is_teen(15)); // should be true
    println!("{}", is_teen(5));  // should be false
    println!("{}", is_teen(25)); // should be false
}
```

**This compiles and runs but gives wrong answers. Identify the logic error, fix it, and simplify the function.**

---

## Section C — Write It Yourself

---

### Q19 — Number Classifier

Write a function `classify_number(n: i32) -> &'static str` that returns:
- `"large positive"` if n > 1000
- `"positive"` if n > 0
- `"zero"` if n == 0
- `"negative"` if n < 0
- `"large negative"` if n < -1000

> **Note:** Think carefully about order — `n < -1000` is also `n < 0`!

In `main`, test with: `-2000`, `-50`, `0`, `500`, `5000`.

---

### Q20 — Season Detector

Write a function `season(month: u32) -> &'static str` that:
- Returns `"Spring"` for months 3, 4, 5
- Returns `"Summer"` for months 6, 7, 8
- Returns `"Autumn"` for months 9, 10, 11
- Returns `"Winter"` for months 12, 1, 2
- Returns `"Invalid"` for anything else

Write two versions:
- One using `else if` chains
- One using `(start..=end).contains(&month)` for range checks

In `main`, test all 12 months and the invalid cases `0` and `13`.

---

### Q21 — Triangle Validator and Classifier

Write two functions:
1. `is_valid_triangle(a: f64, b: f64, c: f64) -> bool`
   - A triangle is valid if each side is less than the sum of the other two
2. `classify_triangle(a: f64, b: f64, c: f64) -> &'static str`
   - `"equilateral"` — all three sides equal
   - `"isosceles"` — exactly two sides equal
   - `"scalene"` — all sides different
   - `"invalid"` — not a valid triangle

In `main`, test with:
- `(3.0, 4.0, 5.0)` → valid scalene
- `(5.0, 5.0, 5.0)` → equilateral
- `(3.0, 3.0, 5.0)` → isosceles
- `(1.0, 2.0, 10.0)` → invalid

---

### Q22 — Password Strength Checker

Write a function `password_strength(password: &str) -> &'static str` that returns:
- `"weak"` if length < 6
- `"moderate"` if length 6–11 and contains at least one digit
- `"strong"` if length >= 12 and contains at least one digit and one uppercase letter
- `"moderate"` if length >= 12 but missing a digit or uppercase
- `"moderate"` if length 6–11 but no digit

> **Hints:**
> - `password.len()` gives the length
> - `password.chars().any(|c| c.is_ascii_digit())` checks for a digit
> - `password.chars().any(|c| c.is_uppercase())` checks for uppercase

In `main`, test with: `"abc"`, `"hello1"`, `"hello"`, `"MySecurePassword1"`, `"mysecurepassword1"`, `"MySecurePassword"`

---

### Q23 — Leap Year Checker

A year is a leap year if:
- It is divisible by 4, **AND**
- If it is divisible by 100, it must also be divisible by 400

Write:
1. `is_leap_year(year: u32) -> bool`
2. `days_in_february(year: u32) -> u32` — returns 28 or 29

In `main`, test: `1900`, `2000`, `2023`, `2024`, `1800`, `2100`, `2400`

Expected results:
- 1900 → not leap (divisible by 100 but not 400)
- 2000 → leap (divisible by 400)
- 2024 → leap (divisible by 4, not by 100)
- 2100 → not leap

---

### Q24 — Shipping Cost Calculator

Write a function `shipping_cost(weight_kg: f64, is_express: bool, is_international: bool) -> f64` with this pricing:

**Base rates:**
- 0–1 kg: $5.00
- 1–5 kg: $10.00
- 5–20 kg: $20.00
- Over 20 kg: $50.00

**Multipliers (applied to base rate):**
- Express shipping: ×2.0
- International: ×1.5
- Express AND International: ×3.0 (not 2.0 × 1.5, but a flat 3x)

**Guard:** negative weight returns `0.0`

In `main`, test these orders and print each cost formatted as `$XX.XX`:
- 0.5 kg, standard, domestic
- 3.0 kg, express, domestic
- 15.0 kg, standard, international
- 25.0 kg, express, international

---

### Q25 — Number Guessing Feedback

Write a function `guess_feedback(secret: i32, guess: i32) -> &'static str` that returns:
- `"Correct!"` if guess == secret
- `"Very cold"` if |guess - secret| > 100
- `"Cold"` if |guess - secret| > 50
- `"Warm"` if |guess - secret| > 10
- `"Hot"` if |guess - secret| > 0
- (Covered by "Correct!" already)

Write `main` to test with `secret = 42` and guesses: `42`, `200`, `80`, `55`, `44`.

---

### Q26 — ATM Withdrawal Validator

Write a function `can_withdraw(balance: f64, amount: f64, daily_limit: f64, withdrawn_today: f64) -> &'static str` that returns:

- `"Invalid amount"` if amount <= 0
- `"Insufficient funds"` if amount > balance
- `"Daily limit exceeded"` if withdrawn_today + amount > daily_limit
- `"Must withdraw in multiples of 10"` if amount % 10.0 != 0.0
- `"Approved"` otherwise

In `main`, test:
- balance=500, amount=200, limit=1000, withdrawn=0 → Approved
- balance=500, amount=600, limit=1000, withdrawn=0 → Insufficient funds
- balance=500, amount=150, limit=1000, withdrawn=0 → Must withdraw in multiples of 10
- balance=1000, amount=200, limit=500, withdrawn=400 → Daily limit exceeded
- balance=500, amount=0, limit=1000, withdrawn=0 → Invalid amount

---

## Section D — Deep Understanding

---

### Q27 — Predict the Complex Output

Trace every step carefully before running.

```rust
fn process(x: i32, y: i32) -> String {
    let category = if x > 0 && y > 0 {
        "both positive"
    } else if x > 0 || y > 0 {
        "one positive"
    } else if x == 0 || y == 0 {
        "has zero"
    } else {
        "both negative"
    };

    let magnitude = if x.abs() + y.abs() > 10 {
        "large"
    } else {
        "small"
    };

    format!("{category}, {magnitude}")
}

fn main() {
    println!("{}", process(3, 4));
    println!("{}", process(-1, 5));
    println!("{}", process(0, -3));
    println!("{}", process(-5, -8));
}
```

---

### Q28 — True or False?

For each statement, answer True or False with a brief explanation.

1. In Rust, `if 1 { println!("yes"); }` is valid code.
2. The `else` clause is required whenever you write an `if`.
3. Both arms of an `if` expression used as a value must have the same type.
4. `&&` always evaluates both its left and right operands.
5. `if let Some(x) = val { }` is equivalent to checking `val.is_some()` and then unwrapping.
6. Variables declared inside an `if` block are visible after the block ends.
7. `if` expressions can be used directly as function arguments.
8. Braces `{}` are optional in Rust `if` statements for single-line bodies.
9. `!true && false` evaluates to `false`.
10. An `else if` chain checks every condition even after one matches.
11. The `if` condition in `if let Some(x) = val` must be a `bool`.
12. You can nest `if` expressions inside other `if` expressions arbitrarily deep.

---

### Q29 — What's the Most Idiomatic Fix?

For each function, write the most idiomatic Rust version:

**Version A:**
```rust
fn is_positive(n: i32) -> bool {
    if n > 0 {
        return true;
    } else {
        return false;
    }
}
```

**Version B:**
```rust
fn max_of_two(a: i32, b: i32) -> i32 {
    let result;
    if a > b {
        result = a;
    } else {
        result = b;
    }
    return result;
}
```

**Version C:**
```rust
fn describe(n: i32) -> String {
    let s;
    if n > 0 {
        s = String::from("positive");
    } else if n < 0 {
        s = String::from("negative");
    } else {
        s = String::from("zero");
    }
    s
}
```

**Version D:**
```rust
fn check_access(is_admin: bool, is_owner: bool, resource_public: bool) -> bool {
    if is_admin == true {
        return true;
    }
    if is_owner == true {
        return true;
    }
    if resource_public == true {
        return true;
    }
    return false;
}
```

---

### Q30 — The Big Debug Challenge

The program below has **10 bugs**. Find every one, explain it, and provide the fully corrected program.

```rust
fn classify_bmi(bmi: f64) -> &str {
    if bmi < 0 {
        return "invalid"
    }
    else if bmi < 18.5 { "underweight" }
    else if bmi < 25.0 { "normal" }
    else if bmi < 30.0 { "overweight" }
    else               { "obese" }
}

fn can_donate_blood(age: u32, weight_kg: f64, is_healthy: bool) -> &'static str {
    if age < 17 || age > 65 { return "age restriction" }
    if weight_kg < 50       { return "weight too low" }
    if !is_healthy           { return "health restriction" }
    "eligible"
}

fn main() {
    let bmi = 22.5;
    let result = if bmi > 0 {
        classify_bmi(bmi)
    } else {
        classify_bmi(-1.0)
    }

    println!("BMI category: {result}");

    let age = 25;
    let weight = 65.5;
    let healthy = true;

    if can_donate_blood(age, weight, healthy) {
        println!("Can donate!");
    }

    let score = 75;
    let message = if score >= 60 { "pass" } else { "fail" };
    println!("Result: " + message);
}
```

**List all 10 bugs with explanations, then show the corrected program.**

---

### Q31 — Design Challenge: When to Use What?

For each scenario, decide whether to use:
- `if` / `else`
- `if let`
- `match` (preview — use `if` chains for now, but note where `match` would be better)
- Boolean expression only (no `if` at all)

Justify your choice and write a short code example for each.

1. Check if a `Vec<i32>` is not empty before processing it
2. Extract the inner value from `Option<String>` if it exists
3. Determine if a number is in the range 1–100 inclusive
4. Choose a greeting based on the time of day (morning/afternoon/evening/night)
5. Set a boolean flag `is_valid` based on whether all three conditions (age >= 18, non-empty name, valid email) are true
6. Handle exactly 5 distinct string values: `"north"`, `"south"`, `"east"`, `"west"`, and anything else

---

*"Control flow is the heartbeat of your program — make it clear, make it simple, make it right." 🦀*
