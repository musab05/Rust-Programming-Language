# ✅ Lesson 36 — Answers: Custom Error Types (E4)

---

## Section A

### A1 — ❌ Won't compile
`MyError` implements `Debug` but not `Display`. The `println!("{e}")` macro requires `Display`. Fix: implement `fmt::Display for MyError`.

### A2 — ✅ Compiles
Both `Display` and `Debug` are implemented, and `Error` is implemented. All requirements met.

---

## Section B

### A3 — MathError
```rust
use std::fmt;

#[derive(Debug)]
enum MathError {
    DivisionByZero,
    NegativeSquareRoot(f64),
    Overflow { operation: String, value: i64 },
}

impl fmt::Display for MathError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            MathError::DivisionByZero => write!(f, "division by zero"),
            MathError::NegativeSquareRoot(n) => {
                write!(f, "cannot take square root of negative number: {n}")
            }
            MathError::Overflow { operation, value } => {
                write!(f, "overflow in '{operation}' with value {value}")
            }
        }
    }
}

impl std::error::Error for MathError {}

fn safe_divide(a: f64, b: f64) -> Result<f64, MathError> {
    if b == 0.0 {
        Err(MathError::DivisionByZero)
    } else {
        Ok(a / b)
    }
}

fn main() {
    println!("{:?}", safe_divide(10.0, 3.0));  // Ok(3.333...)
    println!("{}", safe_divide(10.0, 0.0).unwrap_err());  // division by zero
}
```

### A4 — From conversions
```rust
use std::fmt;
use std::num::ParseIntError;

#[derive(Debug)]
enum AppError {
    Io(std::io::Error),
    Parse(ParseIntError),
}

impl fmt::Display for AppError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            AppError::Io(e) => write!(f, "I/O error: {e}"),
            AppError::Parse(e) => write!(f, "parse error: {e}"),
        }
    }
}

impl std::error::Error for AppError {}

impl From<std::io::Error> for AppError {
    fn from(e: std::io::Error) -> Self { AppError::Io(e) }
}

impl From<ParseIntError> for AppError {
    fn from(e: ParseIntError) -> Self { AppError::Parse(e) }
}

fn read_number(path: &str) -> Result<i32, AppError> {
    let content = std::fs::read_to_string(path)?;
    let number = content.trim().parse::<i32>()?;
    Ok(number)
}

fn main() {
    match read_number("number.txt") {
        Ok(n) => println!("Got: {n}"),
        Err(e) => println!("Error: {e}"),
    }
}
```

### A5 — Key=value parser
```rust
use std::fmt;

#[derive(Debug)]
enum ParseError {
    MissingSeparator(String),
    EmptyKey(String),
    EmptyValue(String),
}

impl fmt::Display for ParseError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            ParseError::MissingSeparator(s) => write!(f, "missing '=' in: '{s}'"),
            ParseError::EmptyKey(s) => write!(f, "empty key in: '{s}'"),
            ParseError::EmptyValue(s) => write!(f, "empty value in: '{s}'"),
        }
    }
}

impl std::error::Error for ParseError {}

fn parse_pair(input: &str) -> Result<(&str, &str), ParseError> {
    let parts: Vec<&str> = input.splitn(2, '=').collect();
    if parts.len() != 2 {
        return Err(ParseError::MissingSeparator(input.into()));
    }
    let (key, value) = (parts[0], parts[1]);
    if key.is_empty() {
        return Err(ParseError::EmptyKey(input.into()));
    }
    if value.is_empty() {
        return Err(ParseError::EmptyValue(input.into()));
    }
    Ok((key, value))
}

fn main() {
    let inputs = vec!["name=Alice", "age=30", "bad_entry", "=nokey", "novalue="];

    for input in inputs {
        match parse_pair(input) {
            Ok((k, v)) => println!("  ✅ {k} = {v}"),
            Err(e) => println!("  ❌ {e}"),
        }
    }
}
```

---

## Section C

### A6
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | `Error: Display + Debug` — both are required |
| 2 | **True** | `#[from]` generates `impl From<T> for MyError` |
| 3 | **False** | `source()` returns the underlying cause error, not the message |
| 4 | **False** | `String` errors lose type information; use enums for production |
| 5 | **True** | `?` uses `From::from()` to convert error types |

### A7
**Use `thiserror`** in most cases:
- Eliminates boilerplate for `Display`, `Error`, and `From`
- Less code to maintain, fewer bugs
- Standard in the Rust ecosystem for library crates

**Implement manually** when:
- You can't add dependencies (e.g., `no_std` environments)
- You need complex custom logic in `Display` or `source()`
- You're learning and want to understand the underlying mechanics

---

## 🏆 Lesson 36 Complete!

✅ The Error trait (Display + Debug)  
✅ Custom error enums with context  
✅ From conversions for ? operator  
✅ Error wrapping and source()  
✅ thiserror crate  

**Next up:** [Lesson 37 — anyhow & error-stack](../lesson_37_anyhow/lesson_37_anyhow.md) 🦀
