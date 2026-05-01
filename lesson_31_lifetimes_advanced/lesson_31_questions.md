# 🧪 Lesson 31 — Questions: Lifetimes in Structs & Advanced (O6)

> **Lesson:** [lesson_31_lifetimes_advanced.md](./lesson_31_lifetimes_advanced.md)  
> **Answers:** [lesson_31_answers.md](./lesson_31_answers.md)

---

## Section A — Predict: Compile or Not?

### Q1
```rust
struct Holder<'a> {
    data: &'a str,
}

fn main() {
    let h;
    {
        let s = String::from("hello");
        h = Holder { data: &s };
    }
    println!("{}", h.data);
}
```

### Q2
```rust
struct Holder<'a> {
    data: &'a str,
}

fn main() {
    let s = String::from("hello");
    let h = Holder { data: &s };
    println!("{}", h.data);
}
```

### Q3
```rust
struct Pair<'a, 'b> {
    first: &'a str,
    second: &'b str,
}

fn main() {
    let a = String::from("alpha");
    let result;
    {
        let b = String::from("beta");
        let p = Pair { first: &a, second: &b };
        result = p.first;
    }
    println!("{result}");
}
```

### Q4
```rust
struct Wrapper<'a> {
    value: &'a i32,
}

impl<'a> Wrapper<'a> {
    fn get(&self) -> &i32 {
        self.value
    }
}

fn main() {
    let x = 10;
    let w = Wrapper { value: &x };
    println!("{}", w.get());
}
```

### Q5
```rust
fn make_holder<'a>() -> Holder<'a> {
    let s = String::from("local");
    Holder { data: &s }
}

struct Holder<'a> {
    data: &'a str,
}
```

---

## Section B — Fix the Errors

### Q6
```rust
struct Name {
    value: &str,
}

fn main() {
    let n = Name { value: "Alice" };
    println!("{}", n.value);
}
```

### Q7
```rust
struct TextBlock<'a> {
    content: &'a str,
}

fn create_block() -> TextBlock {
    let text = String::from("hello");
    TextBlock { content: &text }
}
```

### Q8
```rust
struct Config<'a> {
    name: &'a str,
}

impl Config {
    fn new(name: &str) -> Config {
        Config { name }
    }
}
```

---

## Section C — Write It Yourself

### Q9 — Excerpt struct (Roadmap Practice Task)
Build a struct `Excerpt<'a>` that holds a `&'a str` field called `text`. Implement:
1. `fn new(text: &'a str) -> Excerpt<'a>` constructor
2. `fn word_count(&self) -> usize` that counts words
3. `fn first_word(&self) -> &str` that returns the first word

Test with a `String` and a string literal.

### Q10 — Key-Value pair
Create `struct KeyValue<'a>` with `key: &'a str` and `value: &'a str`. Implement:
1. `fn new(key: &'a str, value: &'a str) -> Self`
2. `fn format(&self) -> String` returning `"key=value"`
3. A `main` that creates several `KeyValue` instances and prints them

### Q11 — Independent lifetimes
Create `struct Comparison<'a, 'b>` with two fields of different lifetimes. Demonstrate that one reference can outlive the other by extracting a value in an inner scope.

---

## Section D — Deep Understanding

### Q12 — True or False?
1. A struct with a reference field must always have a lifetime parameter.
2. `impl<'a> Foo<'a>` means all methods of `Foo` return references with lifetime `'a`.
3. `T: 'static` means `T` must be a string literal.
4. `'a: 'b` means lifetime `'a` is shorter than `'b`.
5. If a struct owns all its data (e.g., `String` fields), it doesn't need lifetime annotations.
6. Elision Rule 3 applies to methods with `&self`, assigning self's lifetime to the output.
7. `Box::leak` is the only way to get a `'static` reference.

### Q13 — Explain
Why does this pattern work?
```rust
struct Config<'a> {
    name: &'a str,
}

fn main() {
    let config = Config { name: "MyApp" };
    println!("{}", config.name);
}
```
Why don't we need a separate `String` variable?

### Q14 — Design decision
You're building a logging system. Each log entry has a `level` (like "INFO", "ERROR"), a `message`, and a `timestamp`. Should `LogEntry` use references (`&str`) or owned data (`String`) for its fields? Justify your answer considering:
1. Log entries stored in a `Vec<LogEntry>`
2. Log entries that might outlive the source data
3. Performance considerations

---

*Lifetimes in structs: the compiler's promise that your data outlives your references! 🦀*
