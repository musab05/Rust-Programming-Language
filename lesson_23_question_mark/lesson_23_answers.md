# ✅ Lesson 23 — Answers: The ? Operator (E3)

---

## Section A

### A1
```
Ok(42)
Err(ParseIntError { kind: InvalidDigit })
```
`"21"` parses to 21, doubled = 42. `"abc"` fails parsing, `?` returns the error immediately.

### A2
```
Some('h')
None
```
`"hello".chars().next()` → `Some('h')`. `"".chars().next()` → `None`, `?` returns `None`.

### A3
```
Ok(30)
Err("invalid digit found in string")
Err("invalid digit found in string")
```
First: 10+20=30. Second: `b` fails. Third: `a` fails — `?` returns at the first failure.

---

## Section B

### A4 — ❌ No
`main()` returns `()` by default. `?` requires the function to return `Result` or `Option`. Fix:
```rust
fn main() -> Result<(), String> {
    let val = get_value()?;
    println!("{val}");
    Ok(())
}
```

### A5 — ✅ Yes
Output:
```
Some(5)
None
None
```
`[5,-1,3]`: first is 5, positive → `Some(5)`. `[-5,-1]`: first is -5, not positive → `None`. `[]`: `first()` returns `None`, `?` propagates it.

### A6 — ❌ No
`?` on `Option` (`chars().next()`) inside a function returning `Result`. Can't mix. Fix: use `.ok_or("empty string")?` to convert `Option` to `Result`.

---

## Section C

### A7
```rust
fn read_number(s: &str) -> Result<i32, String> {
    let n = s.parse::<i32>()
        .map_err(|e| e.to_string())?;   // fix: convert ParseIntError to String
    Ok(n)
}
```

### A8
```rust
fn process(input: &str) -> Result<i32, String> {
    let number = input.parse::<i32>()
        .map_err(|e| e.to_string())?;
    
    let halved = if number % 2 == 0 {
        number / 2
    } else {
        return Err(format!("{number} is odd"));
    };

    let result = halved.checked_sub(10)
        .ok_or_else(|| String::from("Subtraction would underflow"))?;  // fix: String, not &str

    Ok(result)
}
```

### A9
```rust
use std::fs;

fn count_lines(path: &str) -> Result<usize, String> {
    let contents = fs::read_to_string(path)
        .map_err(|e| format!("Failed to read '{path}': {e}"))?;  // fix: convert io::Error to String
    Ok(contents.lines().count())
}
```

---

## Section D

### A10 — Three-step pipeline
```rust
fn process_input(input: &str) -> Result<String, String> {
    // Step 1: Parse as f64
    let number: f64 = input.parse()
        .map_err(|e: std::num::ParseFloatError| format!("Not a number: {e}"))?;

    // Step 2: Square root (fail if negative)
    if number < 0.0 {
        return Err(format!("Cannot take square root of {number}"));
    }
    let sqrt = number.sqrt();

    // Step 3: Format to 2 decimal places
    Ok(format!("{sqrt:.2}"))
}

fn main() {
    for input in ["25", "-4", "hello", "2"] {
        println!("'{input}' → {:?}", process_input(input));
    }
}
```
**Output:**
```
'25' → Ok("5.00")
'-4' → Err("Cannot take square root of -4")
'hello' → Err("Not a number: invalid float literal")
'2' → Ok("1.41")
```

### A11 — File statistics
```rust
use std::error::Error;
use std::fs;

fn file_stats(path: &str) -> Result<(usize, usize, usize), Box<dyn Error>> {
    let contents = fs::read_to_string(path)?;
    let lines = contents.lines().count();
    let words = contents.split_whitespace().count();
    let chars = contents.chars().count();
    Ok((lines, words, chars))
}

fn main() {
    match file_stats("Cargo.toml") {
        Ok((l, w, c)) => println!("Lines: {l}, Words: {w}, Chars: {c}"),
        Err(e) => println!("Error: {e}"),
    }
}
```

### A12 — Option chain
```rust
fn extract_domain(email: &str) -> Option<&str> {
    let at_pos = email.find('@')?;
    Some(&email[at_pos + 1..])
}

fn main() {
    println!("{:?}", extract_domain("user@example.com")); // Some("example.com")
    println!("{:?}", extract_domain("invalid"));           // None
    println!("{:?}", extract_domain(""));                   // None
    println!("{:?}", extract_domain("a@b"));               // Some("b")
}
```

### A13 — Custom error with From
```rust
use std::fs;
use std::io;

#[derive(Debug)]
enum ConfigError {
    FileError(io::Error),
    ParseError(String),
}

impl From<io::Error> for ConfigError {
    fn from(e: io::Error) -> Self {
        ConfigError::FileError(e)
    }
}

fn read_port(path: &str) -> Result<u16, ConfigError> {
    let contents = fs::read_to_string(path)?;  // io::Error → ConfigError::FileError

    let port: u16 = contents.trim().parse()
        .map_err(|e: std::num::ParseIntError|
            ConfigError::ParseError(format!("Invalid port: {e}"))
        )?;

    Ok(port)
}

fn main() {
    match read_port("port.txt") {
        Ok(port) => println!("Port: {port}"),
        Err(ConfigError::FileError(e)) => println!("File error: {e}"),
        Err(ConfigError::ParseError(e)) => println!("Parse error: {e}"),
    }
}
```

---

## Section E

### A14 — True or False?
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **False** | `?` also works with `Option<T>` (function must return `Option`) |
| 2 | **True** | `Ok(v)?` extracts `v` and execution continues normally |
| 3 | **True** | `Err(e)?` calls `.into()` for conversion and returns early |
| 4 | **False** | You cannot mix `?` on `Result` and `Option` in the same function |
| 5 | **True** | `?` is equivalent to `match expr { Ok(v)=>v, Err(e)=>return Err(e.into()) }` |
| 6 | **True** | Any error type can be boxed as `Box<dyn Error>` for convenience |
| 7 | **True** | `main() -> Result<(), E>` where `E: Debug` is valid — errors are printed on exit |
| 8 | **False** | `.ok_or()` converts `Option` to `Result`, not the other way around |

### A15 — Refactor challenge
```rust
use std::fs;

fn read_and_sum(path: &str) -> Result<i64, String> {
    let contents = fs::read_to_string(path)
        .map_err(|e| format!("IO: {e}"))?;

    let mut total: i64 = 0;
    for line in contents.lines() {
        let trimmed = line.trim();
        if trimmed.is_empty() { continue; }
        let n: i64 = trimmed.parse()
            .map_err(|e: std::num::ParseIntError| format!("Parse: {e}"))?;
        total += n;
    }
    Ok(total)
}
```
Both `match` blocks replaced with `?` + `.map_err()`. Same behavior, much cleaner.

---

## 🏆 Lesson 23 Complete!

✅ `?` operator — concise error propagation  
✅ Works with both `Result<T,E>` and `Option<T>`  
✅ Requires function to return `Result` or `Option`  
✅ Auto-converts errors via `From` trait  
✅ `Box<dyn Error>` for accepting any error type  
✅ `main() -> Result<(), E>` for using `?` in main  
✅ Cannot mix `Result` and `Option` `?` in one function  

**Up next: Lesson 24 — Modules & mod 🦀**
