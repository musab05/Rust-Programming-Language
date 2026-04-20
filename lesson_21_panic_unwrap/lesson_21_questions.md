# 🧪 Lesson 21 — Questions: panic! & unwrap (E1)

> **Lesson:** [lesson_21_panic_unwrap.md](./lesson_21_panic_unwrap.md)  
> **Answers:** [lesson_21_answers.md](./lesson_21_answers.md)

---

## Section A — Predict the Output

### Q1
```rust
fn main() {
    let v = vec![10, 20, 30];
    println!("{}", v.get(1).unwrap());
    println!("{}", v.get(5).unwrap_or(&0));
    println!("{:?}", v.get(99));
}
```

### Q2
```rust
fn main() {
    let x: Option<i32> = Some(42);
    let y: Option<i32> = None;

    println!("{}", x.unwrap_or(0));
    println!("{}", y.unwrap_or(0));
    println!("{}", x.expect("x was None"));
}
```

### Q3
```rust
fn main() {
    let results: Vec<Result<i32, &str>> = vec![
        Ok(1),
        Err("bad"),
        Ok(3),
    ];

    for r in &results {
        match r {
            Ok(v) => println!("ok: {v}"),
            Err(e) => println!("err: {e}"),
        }
    }
}
```

### Q4
```rust
fn main() {
    let a: i32 = "123".parse().unwrap();
    let b: f64 = "3.14".parse().unwrap();
    let c: i32 = "99".parse().unwrap_or(-1);
    let d: i32 = "abc".parse().unwrap_or(-1);
    println!("{a} {b} {c} {d}");
}
```

---

## Section B — Will It Compile? Will It Panic?

### Q5
```rust
fn main() {
    let v: Vec<i32> = vec![];
    let first = v[0];
    println!("{first}");
}
```
**Compile? Run successfully?**

### Q6
```rust
fn main() {
    let s = String::from("hello");
    let c = s.chars().nth(10).unwrap();
    println!("{c}");
}
```
**Compile? Run successfully?**

### Q7
```rust
fn main() {
    assert_eq!(2 + 2, 4);
    assert_ne!("rust", "python");
    assert!(true);
    println!("All assertions passed!");
}
```
**Compile? Run successfully?**

### Q8
```rust
fn main() {
    let x: Result<i32, &str> = Err("oops");
    let val = x.unwrap_or_else(|e| {
        println!("Error occurred: {e}");
        -1
    });
    println!("val = {val}");
}
```
**Compile? Run successfully? What does it print?**

---

## Section C — Fix the Bugs

### Q9 — Remove all panics
This program panics. Rewrite it so it handles every case gracefully without panicking:
```rust
fn main() {
    let numbers = vec![10, 20, 30];

    let first = numbers[0];          // could panic on empty vec
    let tenth = numbers[9];          // panics — out of bounds
    let parsed: i32 = "abc".parse().unwrap();  // panics — bad parse

    println!("{first} {tenth} {parsed}");
}
```

### Q10 — Use expect instead of unwrap
Replace every `unwrap()` with a meaningful `expect()` message:
```rust
use std::collections::HashMap;

fn main() {
    let mut scores: HashMap<&str, i32> = HashMap::new();
    scores.insert("Alice", 95);

    let alice_score = scores.get("Alice").unwrap();
    let parsed_score: i32 = "88".parse().unwrap();
    let env_val = std::env::var("MY_APP_CONFIG").unwrap();

    println!("{alice_score} {parsed_score} {env_val}");
}
```

### Q11 — Fix the assertion
```rust
fn double(x: i32) -> i32 {
    x + x
}

fn main() {
    assert_eq!(double(5), 11, "double(5) should be 10");
    assert_eq!(double(0), 0);
    assert_eq!(double(-3), -6);
    println!("All tests passed!");
}
```

---

## Section D — Write It Yourself

### Q12 — Safe division function
Write a function `fn safe_divide(a: f64, b: f64) -> Option<f64>` that:
- Returns `None` if `b` is `0.0`
- Returns `Some(a / b)` otherwise

Test with multiple cases including division by zero. Use `unwrap_or` and `match` to handle the result in `main`.

### Q13 — Panic-free Vec accessor
Write a function `fn get_or_default(v: &[i32], index: usize, default: i32) -> i32` that:
- Returns the value at `index` if it exists
- Returns `default` if the index is out of bounds
- **Never panics**

Test with an empty vec, a vec with 3 elements, and an index of 100.

### Q14 — Intentional panic reader (Roadmap Practice Task)
Write a program that intentionally triggers 3 different panics (one at a time). For each:
1. Write the panicking code
2. Comment it out (so the program compiles)
3. Add a comment describing what the panic message would say
4. Write the **safe alternative** that doesn't panic

The 3 panics should be from different sources (e.g., index out of bounds, unwrap on None, integer overflow).

### Q15 — Validate function inputs with assert
Write a function `fn create_percentage(value: f64) -> f64` that:
- Panics with a descriptive message if `value < 0.0` or `value > 100.0`
- Returns the value otherwise

Also write `fn safe_create_percentage(value: f64) -> Option<f64>` that returns `None` instead of panicking. Test both versions.

---

## Section E — Deep Understanding

### Q16 — True or False?
1. `unwrap()` panics if called on `Some(value)`.
2. `expect("msg")` provides a custom message when panicking.
3. `panic!` always terminates the entire program — it can never be caught.
4. `assert_eq!(a, b)` panics if `a != b`.
5. `debug_assert!` is checked in both debug and release builds.
6. `v.get(i)` returns `Option<&T>` and never panics.
7. `v[i]` returns `Option<&T>` and never panics.
8. `.unwrap_or(default)` panics if the value is `None`.
9. Setting `panic = "abort"` in Cargo.toml prevents stack unwinding.
10. `catch_unwind` is the recommended way to handle errors in Rust.

### Q17 — Explain the difference
Explain the difference between each pair. When would you use one vs the other?
1. `unwrap()` vs `expect()`
2. `unwrap()` vs `unwrap_or()`
3. `assert!()` vs `debug_assert!()`
4. `panic!()` vs returning `Result::Err`
5. `v[i]` vs `v.get(i)`

### Q18 — Big debug challenge (5 bugs)
```rust
fn lookup_score(name: &str) -> i32 {
    let scores = vec![("Alice", 95), ("Bob", 80), ("Cara", 88)];

    for (n, s) in scores {
        if n == name {
            return s;
        }
    }

    panic!("Student not found");  // Bug 1: should not panic for missing student
}

fn main() {
    let name = "Dave";
    let score = lookup_score(name);  // Bug 2: will panic

    let numbers = vec![10, 20, 30];
    let sum = numbers[0] + numbers[1] + numbers[3];  // Bug 3: index 3 out of bounds

    let input = "not_a_number";
    let parsed: i32 = input.parse().unwrap();  // Bug 4: will panic

    assert_eq!(2 + 2, 5);  // Bug 5: wrong assertion
    println!("Score: {score}, Sum: {sum}, Parsed: {parsed}");
}
```
Find and fix all 5 bugs so the program compiles and runs without any panics.

---

*Remember: `panic!` is your last resort, not your first tool. The compiler is your friend — let it help you handle errors gracefully.* 🦀
