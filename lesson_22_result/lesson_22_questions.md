# 🧪 Lesson 22 — Questions: Result\<T,E\> (E2)

> **Lesson:** [lesson_22_result.md](./lesson_22_result.md)  
> **Answers:** [lesson_22_answers.md](./lesson_22_answers.md)

---

## Section A — Predict the Output

### Q1
```rust
fn main() {
    let a: Result<i32, &str> = Ok(10);
    let b: Result<i32, &str> = Err("oops");

    println!("{}", a.unwrap_or(0));
    println!("{}", b.unwrap_or(0));
    println!("{}", a.is_ok());
    println!("{}", b.is_err());
}
```

### Q2
```rust
fn main() {
    let x: Result<i32, &str> = Ok(5);
    let y = x.map(|v| v * 3).map(|v| v + 1);
    println!("{:?}", y);

    let z: Result<i32, &str> = Err("fail");
    let w = z.map(|v| v * 3).map(|v| v + 1);
    println!("{:?}", w);
}
```

### Q3
```rust
fn half(x: i32) -> Result<i32, String> {
    if x % 2 == 0 {
        Ok(x / 2)
    } else {
        Err(format!("{x} is not even"))
    }
}

fn main() {
    let result = Ok(20)
        .and_then(half)
        .and_then(half);
    println!("{:?}", result);

    let result2 = Ok(15)
        .and_then(half);
    println!("{:?}", result2);
}
```

### Q4
```rust
fn main() {
    let results: Vec<Result<i32, &str>> = vec![Ok(1), Err("a"), Ok(3), Err("b")];
    let oks: Vec<i32> = results.iter().filter_map(|r| r.as_ref().ok().copied()).collect();
    let errs: Vec<&&str> = results.iter().filter_map(|r| r.as_ref().err()).collect();
    println!("{:?}", oks);
    println!("{:?}", errs);
}
```

---

## Section B — Will It Compile?

### Q5
```rust
fn main() {
    let result: Result<i32, String> = Ok(42);
    let val = result.unwrap();
    let val2 = result.unwrap();  // using result again
    println!("{val} {val2}");
}
```

### Q6
```rust
fn get_number() -> Result<i32, String> {
    Ok(42)
}

fn main() {
    let num = get_number();
    println!("{num}");  // printing Result directly
}
```

### Q7
```rust
fn main() {
    let opt: Option<i32> = Some(10);
    let res: Result<i32, &str> = opt.ok_or("was None");
    println!("{:?}", res);

    let res2: Result<i32, &str> = Ok(99);
    let opt2: Option<i32> = res2.ok();
    println!("{:?}", opt2);
}
```

---

## Section C — Fix the Bugs

### Q8 — Handle the error properly
```rust
use std::fs;

fn main() {
    let contents = fs::read_to_string("nonexistent.txt");
    println!("Contents: {contents}");  // BUG: can't print Result with {}
}
```

### Q9 — Fix the combinator chain
```rust
fn main() {
    let result: Result<&str, &str> = Ok("42");
    let parsed = result.map(|s| s.parse::<i32>());  // BUG: this creates Result<Result<...>>
    println!("{:?}", parsed);
}
```

### Q10 — Type mismatch
```rust
fn validate_age(input: &str) -> Result<u32, String> {
    let age = input.parse::<u32>().unwrap();  // BUG: panics instead of returning Err
    if age < 18 {
        Err(String::from("Must be 18 or older"))
    } else {
        Ok(age)
    }
}

fn main() {
    println!("{:?}", validate_age("25"));
    println!("{:?}", validate_age("abc"));
    println!("{:?}", validate_age("15"));
}
```

---

## Section D — Write It Yourself

### Q11 — Safe string-to-number converter
Write `fn parse_positive(s: &str) -> Result<u32, String>` that:
- Returns `Err` if the string can't be parsed as `u32`
- Returns `Err` if the number is `0`
- Returns `Ok(n)` for valid positive numbers

Test with: `"42"`, `"0"`, `"-5"`, `"hello"`, `"1000000"`

### Q12 — File reader (Roadmap Practice Task)
Write `fn read_file_safe(path: &str) -> Result<String, String>` that:
- Reads a file using `std::fs::read_to_string`
- Returns `Ok(contents)` on success
- Returns `Err(message)` with a user-friendly error message on failure
- The error message should include the file path

In `main`, try to read 2 files — one that exists (`Cargo.toml` or any file you know exists) and one that doesn't.

### Q13 — Grade calculator
Write a program that:
1. Has `fn parse_grade(s: &str) -> Result<f64, String>` — parses a string to f64, returns error if not a number or not between 0.0 and 100.0
2. Has `fn letter_grade(score: f64) -> &'static str` — returns "A" (90+), "B" (80+), "C" (70+), "D" (60+), or "F"
3. In `main`, process a list of grade strings: `["95", "abc", "82", "101", "73", "-5", "60"]`
4. For valid grades, print the letter grade. For invalid, print the error.

### Q14 — Result chain
Write a pipeline that takes a string input and:
1. Parses it as `i32` (might fail)
2. Checks it's positive (might fail)
3. Checks it's less than 1000 (might fail)
4. Returns the value doubled

Use `and_then` to chain the steps. Test with: `"42"`, `"-5"`, `"2000"`, `"hello"`.

---

## Section E — Deep Understanding

### Q15 — True or False?
1. `Result<T, E>` has exactly two variants: `Ok(T)` and `Err(E)`.
2. `.map()` on an `Err` variant will execute the closure.
3. `.and_then()` is useful when the transformation itself can fail.
4. `result.ok()` converts `Result<T,E>` to `Option<E>`.
5. `.unwrap_or_default()` requires `T: Default`.
6. `is_ok()` consumes the `Result`.
7. You can use `match` on a `Result` just like on an `Option`.
8. The `?` operator can only be used with `Result`, not `Option`.
9. `Result<(), E>` is valid — it means "success with no value, or error".
10. `map_err` transforms the `Ok` value of a `Result`.

### Q16 — Compare approaches
For each scenario, which approach would you use and why?
1. Parsing user input that might be invalid
2. Looking up a config value that must exist for the program to run
3. Reading a file that might not exist
4. Accessing an element in a vec by index
5. A function that can never fail

### Q17 — Big debug challenge
```rust
fn parse_pair(s: &str) -> Result<(i32, i32), String> {
    let parts: Vec<&str> = s.split(',').collect();
    if parts.len() != 2 {
        return Err(format!("Expected 2 parts, got {}", parts.len()));
    }

    let a = parts[0].parse::<i32>().unwrap();  // Bug 1: should not unwrap
    let b = parts[1].parse::<i32>().unwrap();  // Bug 2: should not unwrap

    Ok((a, b))
}

fn add_pairs(s1: &str, s2: &str) -> Result<(i32, i32), String> {
    let (a1, b1) = parse_pair(s1).unwrap();  // Bug 3: should handle error
    let (a2, b2) = parse_pair(s2).unwrap();  // Bug 4: should handle error
    Ok((a1 + a2, b1 + b2))
}

fn main() {
    println!("{:?}", add_pairs("1,2", "3,4"));
    println!("{:?}", add_pairs("1,abc", "3,4"));   // Bug 5: will panic
    println!("{:?}", add_pairs("1,2,3", "3,4"));   // has 3 parts
}
```
Fix all 5 bugs so the program handles all error cases gracefully.

---

*Errors are values in Rust — treat them as such! 🦀*
