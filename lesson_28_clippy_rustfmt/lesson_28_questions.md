# 🧪 Lesson 28 — Questions: Clippy & rustfmt (B5)

> **Lesson:** [lesson_28_clippy_rustfmt.md](./lesson_28_clippy_rustfmt.md)  
> **Answers:** [lesson_28_answers.md](./lesson_28_answers.md)

---

## Section A — Spot the Clippy Warning

For each snippet, identify what clippy would warn about and write the fix.

### Q1
```rust
fn is_adult(age: u32) -> bool {
    if age >= 18 { true } else { false }
}
```

### Q2
```rust
fn main() {
    let names = vec!["Alice", "Bob", "Cara"];
    if names.len() > 0 {
        println!("Have names");
    }
}
```

### Q3
```rust
fn add(a: i32, b: i32) -> i32 {
    return a + b;
}
```

### Q4
```rust
fn main() {
    let mut s = String::new();
    s.push_str("!");
}
```

### Q5
```rust
fn main() {
    let x = 42;
    let y = x.clone();  // x is i32 (Copy type)
    println!("{y}");
}
```

### Q6
```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5];
    let mut sum = 0;
    for item in &v {
        sum += item;
    }
    println!("{sum}");
}
```

---

## Section B — Fix All Clippy Warnings (Roadmap Practice Task)

### Q7 — Fix this entire program
```rust
fn get_status(code: i32) -> String {
    let result = if code == 200 {
        return String::from("OK");
    } else if code == 404 {
        return String::from("Not Found");
    } else if code == 500 {
        return String::from("Server Error");
    } else {
        return String::from("Unknown");
    };
    result
}

fn is_valid(s: &str) -> bool {
    if s.len() == 0 {
        return false;
    }
    if s.contains("@") { true } else { false }
}

fn process(items: &Vec<i32>) -> Vec<i32> {
    let mut result = Vec::new();
    for i in 0..items.len() {
        if items[i] > 0 {
            result.push(items[i] * 2);
        }
    }
    result
}

fn main() {
    let status = get_status(200);
    println!("{status}");

    let valid = is_valid("user@example.com");
    println!("Valid: {valid}");

    let nums = vec![1, -2, 3, -4, 5];
    let doubled = process(&nums);
    println!("{:?}", doubled);
}
```

---

## Section C — Formatting

### Q8 — Format this code
Manually rewrite this code as `cargo fmt` would format it:
```rust
fn main(){let x=5;let y=10;if x>3{println!("{}",x+y)}else{println!("small")}let v=vec![1,2,3];for i in &v{println!("{}",i)}}
```

### Q9 — rustfmt.toml
What would you put in `rustfmt.toml` to:
1. Set max line width to 80
2. Use 2 spaces for indentation
3. Merge imports by crate

---

## Section D — Deep Understanding

### Q10 — True or False?
1. `cargo fmt` modifies your source files in place.
2. `cargo clippy` can automatically fix all warnings.
3. `#[allow(clippy::lint_name)]` permanently silences a lint for the entire project.
4. Clippy warnings always indicate bugs.
5. `cargo fmt -- --check` exits with an error if formatting is wrong.
6. `clippy::pedantic` lints are enabled by default.
7. You can configure clippy in `Cargo.toml` under `[lints.clippy]`.
8. `_variable` prefix suppresses unused variable warnings.

### Q11 — When to suppress vs when to fix
For each clippy warning, would you fix it or suppress it? Explain why.
1. `clippy::needless_return` on a function with a complex match at the end
2. `clippy::too_many_arguments` on a function with 8 parameters
3. `clippy::cast_possible_truncation` where you've verified the range
4. `clippy::unwrap_used` in test code

---

*Keep your code clean and idiomatic! 🦀*
