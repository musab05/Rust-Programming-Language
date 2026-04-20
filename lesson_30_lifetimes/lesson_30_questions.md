# 🧪 Lesson 30 — Questions: Lifetimes Basics (O5)

> **Lesson:** [lesson_30_lifetimes.md](./lesson_30_lifetimes.md)  
> **Answers:** [lesson_30_answers.md](./lesson_30_answers.md)

---

## Section A — Predict: Compile or Not?

### Q1
```rust
fn main() {
    let r;
    {
        let x = 5;
        r = &x;
    }
    println!("{r}");
}
```

### Q2
```rust
fn main() {
    let x = 5;
    let r = &x;
    println!("{r}");
}
```

### Q3
```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}

fn main() {
    let s1 = String::from("hello");
    let result;
    {
        let s2 = String::from("hi");
        result = longest(s1.as_str(), s2.as_str());
    }
    println!("{result}");
}
```

### Q4
```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}

fn main() {
    let s1 = String::from("hello world");
    let result;
    {
        let s2 = String::from("hi");
        result = longest(s1.as_str(), s2.as_str());
        println!("{result}");
    }
}
```

### Q5
```rust
fn first<'a>(x: &'a str, _y: &str) -> &'a str {
    x
}

fn main() {
    let s1 = String::from("hello");
    let result;
    {
        let s2 = String::from("world");
        result = first(s1.as_str(), s2.as_str());
    }
    println!("{result}");
}
```

---

## Section B — Does It Need Lifetime Annotations?

For each function, determine: does the compiler need explicit lifetime annotations, or can elision handle it?

### Q6
```rust
fn first_word(s: &str) -> &str {
    &s[..s.find(' ').unwrap_or(s.len())]
}
```

### Q7
```rust
fn longer(a: &str, b: &str) -> &str {
    if a.len() > b.len() { a } else { b }
}
```

### Q8
```rust
fn count_chars(s: &str) -> usize {
    s.chars().count()
}
```

### Q9
```rust
fn identity(s: &str) -> &str {
    s
}
```

### Q10
```rust
fn make_greeting(name: &str) -> String {
    format!("Hello, {name}!")
}
```

---

## Section C — Fix the Lifetime Errors

### Q11
```rust
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() { x } else { y }
}
```

### Q12
```rust
fn get_str() -> &str {
    let s = String::from("hello");
    &s
}
```

### Q13
```rust
fn pick_first(a: &str, b: &str) -> &str {
    a
}
```

---

## Section D — Write It Yourself

### Q14 — Longest of two strings (Roadmap Practice Task)
Write `fn longest<'a>(x: &'a str, y: &'a str) -> &'a str` that returns whichever is longer. Test with:
1. Both strings in the same scope
2. One string in an inner scope (show it works when used within the inner scope)
3. Show that trying to use the result outside the inner scope fails (as a comment)

### Q15 — First non-empty
Write `fn first_non_empty<'a>(a: &'a str, b: &'a str) -> &'a str` that returns the first non-empty string, or `b` if both are empty. Test with multiple cases.

### Q16 — Trim and return
Write `fn trim_start_matches_char<'a>(s: &'a str, c: char) -> &'a str` that returns a slice of `s` with leading occurrences of `c` removed. (Hint: use `s.trim_start_matches(c)` which returns a `&str` from the same data.)

---

## Section E — Deep Understanding

### Q17 — True or False?
1. Every reference in Rust has a lifetime.
2. Lifetime annotations change how long a reference lives.
3. `'static` means the data is allocated on the heap.
4. If a function has one reference input and one reference output, lifetime elision will annotate them with the same lifetime.
5. You can return a reference to a local variable from a function.
6. String literals (`"hello"`) have `'static` lifetime.
7. `fn foo<'a, 'b>(x: &'a str, y: &'b str) -> &'a str` means the output can only come from `x`.
8. Lifetime annotations are required on all functions that take references.

### Q18 — Explain the error
Why does this code fail? What is the compiler trying to protect you from?
```rust
fn main() {
    let result;
    {
        let data = vec![1, 2, 3];
        result = &data[0];
    }
    println!("{result}");
}
```

### Q19 — Elision rules
Apply the three elision rules to determine the full annotated signatures:
1. `fn first(s: &str) -> &str`
2. `fn skip(s: &str, n: usize) -> &str`
3. `fn combine(a: &str, b: &str) -> &str`

Which of these will fail (require manual annotation)?

---

*Lifetimes: the compiler's guarantee that your references always point to valid data! 🦀*
