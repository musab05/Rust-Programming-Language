# 🧪 Lesson 23 — Questions: The ? Operator (E3)

> **Lesson:** [lesson_23_question_mark.md](./lesson_23_question_mark.md)  
> **Answers:** [lesson_23_answers.md](./lesson_23_answers.md)

---

## Section A — Predict the Output

### Q1
```rust
use std::num::ParseIntError;

fn double_parse(s: &str) -> Result<i32, ParseIntError> {
    let n = s.parse::<i32>()?;
    Ok(n * 2)
}

fn main() {
    println!("{:?}", double_parse("21"));
    println!("{:?}", double_parse("abc"));
}
```

### Q2
```rust
fn first_char(s: &str) -> Option<char> {
    let c = s.chars().next()?;
    Some(c)
}

fn main() {
    println!("{:?}", first_char("hello"));
    println!("{:?}", first_char(""));
}
```

### Q3
```rust
fn parse_add(a: &str, b: &str) -> Result<i32, String> {
    let x: i32 = a.parse().map_err(|e: std::num::ParseIntError| e.to_string())?;
    let y: i32 = b.parse().map_err(|e: std::num::ParseIntError| e.to_string())?;
    Ok(x + y)
}

fn main() {
    println!("{:?}", parse_add("10", "20"));
    println!("{:?}", parse_add("10", "abc"));
    println!("{:?}", parse_add("xyz", "20"));
}
```

---

## Section B — Will It Compile?

### Q4
```rust
fn get_value() -> Result<i32, String> {
    Ok(42)
}

fn main() {
    let val = get_value()?;
    println!("{val}");
}
```

### Q5
```rust
fn find_positive(numbers: &[i32]) -> Option<i32> {
    let first = numbers.first()?;
    if *first > 0 { Some(*first) } else { None }
}

fn main() {
    println!("{:?}", find_positive(&[5, -1, 3]));
    println!("{:?}", find_positive(&[-5, -1]));
    println!("{:?}", find_positive(&[]));
}
```

### Q6
```rust
fn mixed(s: &str) -> Result<usize, String> {
    let trimmed = s.trim();
    let first = trimmed.chars().next()?;  // ? on Option in Result function
    Ok(first as usize)
}
```

---

## Section C — Fix the Bugs

### Q7
```rust
fn read_number(s: &str) -> Result<i32, String> {
    let n = s.parse::<i32>()?;   // BUG: ParseIntError ≠ String
    Ok(n)
}
```

### Q8
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
        .ok_or("Subtraction would underflow")?;  // BUG: &str vs String

    Ok(result)
}
```

### Q9
```rust
use std::fs;

fn count_lines(path: &str) -> Result<usize, String> {
    let contents = fs::read_to_string(path)?;  // BUG: io::Error ≠ String
    Ok(contents.lines().count())
}
```

---

## Section D — Write It Yourself

### Q10 — Three-step pipeline (Roadmap Practice Task)
Write `fn process_input(input: &str) -> Result<String, String>` that chains 3 fallible operations using `?`:
1. Parse input as `f64` (fail if not a number)
2. Take the square root — fail if the number is negative
3. Format result to 2 decimal places

Test with: `"25"`, `"-4"`, `"hello"`, `"2"`

### Q11 — File statistics
Write `fn file_stats(path: &str) -> Result<(usize, usize, usize), Box<dyn std::error::Error>>` that returns `(lines, words, chars)` for a file. Use `?` for the file read. Test with a real file.

### Q12 — Option chain
Write `fn extract_domain(email: &str) -> Option<&str>` that:
1. Finds the `@` character position (use `.find('@')` which returns `Option`)
2. Returns the substring after `@`
3. Returns `None` if there's no `@`

Use `?` with Option. Test with: `"user@example.com"`, `"invalid"`, `""`, `"a@b"`

### Q13 — Custom error with From
Define an enum `ConfigError` with variants `FileError(io::Error)` and `ParseError(String)`. Implement `From<io::Error>` for it. Write a function that reads a config file and parses a port number, using `?` throughout.

---

## Section E — Deep Understanding

### Q14 — True or False?
1. `?` can only be used inside functions that return `Result`.
2. `?` on `Ok(v)` extracts `v` and continues execution.
3. `?` on `Err(e)` calls `return Err(e.into())`.
4. You can use `?` on both `Result` and `Option` in the same function.
5. `?` is syntactic sugar for a `match` expression.
6. `Box<dyn Error>` allows `?` to work with any error type.
7. `main()` can return `Result<(), E>` where `E: Debug`.
8. `.ok_or()` converts a `Result` to an `Option`.

### Q15 — Refactor challenge
Rewrite this function using `?` — remove all `match` blocks:
```rust
use std::fs;
use std::io;
use std::num::ParseIntError;

fn read_and_sum(path: &str) -> Result<i64, String> {
    let contents = match fs::read_to_string(path) {
        Ok(c) => c,
        Err(e) => return Err(format!("IO: {e}")),
    };

    let mut total: i64 = 0;
    for line in contents.lines() {
        let trimmed = line.trim();
        if trimmed.is_empty() { continue; }
        let n: i64 = match trimmed.parse() {
            Ok(n) => n,
            Err(e) => return Err(format!("Parse: {e}")),
        };
        total += n;
    }
    Ok(total)
}
```

---

*The `?` is the most elegant operator in Rust — use it generously! 🦀*
