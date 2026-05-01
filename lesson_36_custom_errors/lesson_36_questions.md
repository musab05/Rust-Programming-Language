# 🧪 Lesson 36 — Questions: Custom Error Types (E4)

> **Lesson:** [lesson_36_custom_errors.md](./lesson_36_custom_errors.md)  
> **Answers:** [lesson_36_answers.md](./lesson_36_answers.md)

---

## Section A — Predict: Compile or Not?

### Q1
```rust
#[derive(Debug)]
enum MyError {
    NotFound(String),
}

fn fail() -> Result<(), MyError> {
    Err(MyError::NotFound("data.txt".into()))
}

fn main() {
    match fail() {
        Err(e) => println!("{e}"),
        Ok(()) => {}
    }
}
```

### Q2
```rust
use std::fmt;

#[derive(Debug)]
enum MyError { BadInput }

impl fmt::Display for MyError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "bad input")
    }
}

impl std::error::Error for MyError {}

fn main() {
    let e = MyError::BadInput;
    println!("{e}");
}
```

---

## Section B — Write It Yourself

### Q3 — Basic custom error
Create a `MathError` enum with variants:
- `DivisionByZero`
- `NegativeSquareRoot(f64)`
- `Overflow { operation: String, value: i64 }`

Implement `Display` and `Error`. Write a `safe_divide(a: f64, b: f64) -> Result<f64, MathError>` function.

### Q4 — From conversions
Create an `AppError` with `Io(std::io::Error)` and `Parse(std::num::ParseIntError)` variants. Implement `From` for both. Write a function that reads a file and parses its content as `i32`, using only `?`.

### Q5 — CLI parser (Roadmap Practice Task)
Build a simple key=value parser. Define `ParseError` with variants for:
- Missing `=` separator
- Empty key
- Empty value

Parse `vec!["name=Alice", "age=30", "bad_entry", "=nokey", "novalue="]` and collect successes/errors.

---

## Section C — Deep Understanding

### Q6 — True or False?
1. A custom error type must implement both `Display` and `Debug`.
2. `#[from]` in thiserror generates a `From` implementation.
3. `Error::source()` returns the human-readable error message.
4. You should use `String` as your error type in production code.
5. `?` can convert error types if a `From` implementation exists.

### Q7
When should you use `thiserror` vs implementing the traits manually? What are the tradeoffs?

---

*Descriptive errors turn debugging from guesswork into science! 🦀*
