# ✅ Lesson 22 — Answers: Result\<T,E\> (E2)

---

## Section A

### A1
```
10
0
true
true
```
`a` is `Ok(10)` → unwrap_or returns 10. `b` is `Err` → unwrap_or returns 0. `a.is_ok()` = true. `b.is_err()` = true.

### A2
```
Ok(16)
Err("fail")
```
`Ok(5)` → map(×3) → `Ok(15)` → map(+1) → `Ok(16)`. `Err("fail")` → map skips both closures → `Err("fail")`.

### A3
```
Ok(5)
Err("15 is not even")
```
`Ok(20)` → half → `Ok(10)` → half → `Ok(5)`. `Ok(15)` → half → 15 is odd → `Err("15 is not even")`.

### A4
```
[1, 3]
["a", "b"]
```
`filter_map` with `.ok()` collects only `Ok` values: [1, 3]. `.err()` collects only `Err` values: ["a", "b"].

---

## Section B

### A5 — ❌ No
`Result` doesn't implement `Copy`. After `result.unwrap()`, `result` is moved. The second `result.unwrap()` fails: `use of moved value: result`. Fix: use `result.as_ref().unwrap()` or clone.

### A6 — ❌ No
`Result` does not implement `Display` (it implements `Debug`). `println!("{num}")` requires `Display`. Fix: use `{num:?}` for Debug format, or match and print the inner value.

### A7 — ✅ Yes
Both conversions are valid. `Some(10).ok_or("was None")` → `Ok(10)`. `Ok(99).ok()` → `Some(99)`. Output:
```
Ok(10)
Some(99)
```

---

## Section C

### A8 — Handle the error properly
```rust
use std::fs;

fn main() {
    match fs::read_to_string("nonexistent.txt") {
        Ok(contents) => println!("Contents: {contents}"),
        Err(e) => println!("Error reading file: {e}"),
    }
}
```

### A9 — Fix the combinator chain
```rust
fn main() {
    let result: Result<&str, &str> = Ok("42");
    // Fix: use and_then instead of map when the closure returns a Result
    let parsed = result.and_then(|s| {
        s.parse::<i32>().map_err(|e| "parse failed")
    });
    println!("{:?}", parsed); // Ok(42)
}
```
Bug: `.map()` wraps the return value in `Ok`, so `parse()` returning `Result` creates `Result<Result<i32, _>, _>`. Use `.and_then()` which flattens the nested Result.

### A10 — Type mismatch
```rust
fn validate_age(input: &str) -> Result<u32, String> {
    let age = input.parse::<u32>()
        .map_err(|e| format!("Invalid number: {e}"))?;  // fix: propagate error
    if age < 18 {
        Err(String::from("Must be 18 or older"))
    } else {
        Ok(age)
    }
}

fn main() {
    println!("{:?}", validate_age("25"));   // Ok(25)
    println!("{:?}", validate_age("abc"));  // Err("Invalid number: ...")
    println!("{:?}", validate_age("15"));   // Err("Must be 18 or older")
}
```

---

## Section D

### A11 — Safe string-to-number converter
```rust
fn parse_positive(s: &str) -> Result<u32, String> {
    let n = s.parse::<u32>()
        .map_err(|e| format!("Cannot parse '{s}': {e}"))?;
    if n == 0 {
        Err(String::from("Value must be positive (not zero)"))
    } else {
        Ok(n)
    }
}

fn main() {
    let tests = ["42", "0", "-5", "hello", "1000000"];
    for t in tests {
        match parse_positive(t) {
            Ok(n) => println!("'{t}' → Ok({n})"),
            Err(e) => println!("'{t}' → Err({e})"),
        }
    }
}
```
**Output:**
```
'42' → Ok(42)
'0' → Err(Value must be positive (not zero))
'-5' → Err(Cannot parse '-5': invalid digit found in string)
'hello' → Err(Cannot parse 'hello': invalid digit found in string)
'1000000' → Ok(1000000)
```

### A12 — File reader (Roadmap Practice Task)
```rust
use std::fs;

fn read_file_safe(path: &str) -> Result<String, String> {
    fs::read_to_string(path)
        .map_err(|e| format!("Failed to read '{path}': {e}"))
}

fn main() {
    // Try an existing file
    match read_file_safe("Cargo.toml") {
        Ok(contents) => {
            let lines = contents.lines().count();
            println!("✅ Cargo.toml: {lines} lines");
        }
        Err(e) => println!("❌ {e}"),
    }

    // Try a non-existent file
    match read_file_safe("does_not_exist.txt") {
        Ok(contents) => println!("Contents: {contents}"),
        Err(e) => println!("❌ {e}"),
    }
}
```

### A13 — Grade calculator
```rust
fn parse_grade(s: &str) -> Result<f64, String> {
    let score = s.parse::<f64>()
        .map_err(|e| format!("'{s}' is not a number: {e}"))?;
    if score < 0.0 || score > 100.0 {
        Err(format!("{score} is out of range (0-100)"))
    } else {
        Ok(score)
    }
}

fn letter_grade(score: f64) -> &'static str {
    if score >= 90.0 { "A" }
    else if score >= 80.0 { "B" }
    else if score >= 70.0 { "C" }
    else if score >= 60.0 { "D" }
    else { "F" }
}

fn main() {
    let inputs = ["95", "abc", "82", "101", "73", "-5", "60"];
    for input in inputs {
        match parse_grade(input) {
            Ok(score) => println!("{input} → {score} → {}", letter_grade(score)),
            Err(e) => println!("{input} → Error: {e}"),
        }
    }
}
```
**Output:**
```
95 → 95 → A
abc → Error: 'abc' is not a number: invalid float literal
82 → 82 → B
101 → Error: 101 is out of range (0-100)
73 → 73 → C
-5 → Error: -5 is out of range (0-100)
60 → 60 → D
```

### A14 — Result chain
```rust
fn process(input: &str) -> Result<i32, String> {
    input
        .parse::<i32>()
        .map_err(|e| format!("Parse error: {e}"))
        .and_then(|n| {
            if n <= 0 {
                Err(format!("{n} is not positive"))
            } else {
                Ok(n)
            }
        })
        .and_then(|n| {
            if n >= 1000 {
                Err(format!("{n} is too large (max 999)"))
            } else {
                Ok(n)
            }
        })
        .map(|n| n * 2)
}

fn main() {
    let tests = ["42", "-5", "2000", "hello"];
    for t in tests {
        println!("'{t}' → {:?}", process(t));
    }
}
```
**Output:**
```
'42' → Ok(84)
'-5' → Err("-5 is not positive")
'2000' → Err("2000 is too large (max 999)")
'hello' → Err("Parse error: invalid digit found in string")
```

---

## Section E

### A15 — True or False?
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | `Result<T, E>` is an enum with exactly `Ok(T)` and `Err(E)` |
| 2 | **False** | `.map()` on `Err` skips the closure entirely and returns `Err` unchanged |
| 3 | **True** | `and_then` accepts a closure that returns `Result`, chaining fallible ops |
| 4 | **False** | `.ok()` converts `Result<T,E>` to `Option<T>` (not `Option<E>`) |
| 5 | **True** | `unwrap_or_default()` calls `Default::default()`, so `T` must implement `Default` |
| 6 | **False** | `is_ok()` takes `&self` — it borrows, doesn't consume |
| 7 | **True** | `match` works on both `Result` and `Option` — they're both enums |
| 8 | **False** | `?` works with both `Result` and `Option` (since Rust 1.22 for Option in Try) |
| 9 | **True** | `Result<(), E>` means "no meaningful success value" — common for functions that do side effects |
| 10 | **False** | `map_err` transforms the `Err` value, not the `Ok` value — use `map` for `Ok` |

### A16 — Compare approaches
1. **Parsing user input** → `Result<T, E>` with `parse()` and proper error messages. User input is inherently fallible.
2. **Config value that must exist** → `expect("meaningful message")`. If the config is truly required, failing fast with context is appropriate.
3. **Reading a file** → `Result<T, io::Error>` and propagate or handle. File operations commonly fail and callers should decide how to handle it.
4. **Vec element by index** → `.get(i)` returning `Option<&T>`. Index might be out of bounds — use the safe accessor.
5. **Function that can never fail** → Just return `T` directly. No need for `Result` or `Option` if failure is impossible.

### A17 — Big debug challenge
```rust
fn parse_pair(s: &str) -> Result<(i32, i32), String> {
    let parts: Vec<&str> = s.split(',').collect();
    if parts.len() != 2 {
        return Err(format!("Expected 2 parts, got {}", parts.len()));
    }

    // Fix 1 & 2: propagate parse errors instead of unwrapping
    let a = parts[0].parse::<i32>()
        .map_err(|e| format!("Cannot parse '{}': {e}", parts[0]))?;
    let b = parts[1].parse::<i32>()
        .map_err(|e| format!("Cannot parse '{}': {e}", parts[1]))?;

    Ok((a, b))
}

fn add_pairs(s1: &str, s2: &str) -> Result<(i32, i32), String> {
    // Fix 3 & 4: propagate errors with ?
    let (a1, b1) = parse_pair(s1)?;
    let (a2, b2) = parse_pair(s2)?;
    Ok((a1 + a2, b1 + b2))
}

fn main() {
    // Fix 5: all cases handled gracefully
    println!("{:?}", add_pairs("1,2", "3,4"));     // Ok((4, 6))
    println!("{:?}", add_pairs("1,abc", "3,4"));   // Err("Cannot parse 'abc': ...")
    println!("{:?}", add_pairs("1,2,3", "3,4"));   // Err("Expected 2 parts, got 3")
}
```

---

## 🏆 Lesson 22 Complete!

✅ `Result<T, E>` — `Ok(T)` for success, `Err(E)` for failure  
✅ `match` for explicit error handling  
✅ `unwrap`, `expect` for quick extraction (panic on error)  
✅ `unwrap_or`, `unwrap_or_else`, `unwrap_or_default` for safe defaults  
✅ Combinators: `map`, `map_err`, `and_then`, `or_else`  
✅ `is_ok()`, `is_err()` for boolean checks  
✅ Converting between `Option` and `Result`  
✅ Writing functions that return `Result`  

**Up next: Lesson 23 — The ? Operator 🦀**
