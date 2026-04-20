# 🧪 Lesson 16 — Questions: `if let` / `while let` / `let else` (S5)

> **Lesson:** [lesson_16_if_let.md](./lesson_16_if_let.md)  
> **Answers:** [lesson_16_answers.md](./lesson_16_answers.md)

---

## Section A — Predict the Output

### Q1
```rust
fn main() {
    let values = vec![Some(10), None, Some(30), None, Some(50)];
    for v in &values {
        if let Some(n) = v {
            print!("{n} ");
        }
    }
    println!();
}
```

### Q2
```rust
fn main() {
    let mut stack = vec!["a", "b", "c"];
    while let Some(top) = stack.pop() {
        print!("{top} ");
    }
    println!("done");
}
```

### Q3
```rust
fn describe(n: Option<i32>) -> &'static str {
    if let Some(x) = n {
        if x > 0 { "positive" } else { "non-positive" }
    } else {
        "none"
    }
}
fn main() {
    println!("{}", describe(Some(5)));
    println!("{}", describe(Some(-3)));
    println!("{}", describe(None));
}
```

### Q4
```rust
fn get_value(s: &str) -> u32 {
    let Ok(n) = s.parse::<u32>() else {
        return 0;
    };
    n * 10
}
fn main() {
    println!("{}", get_value("5"));
    println!("{}", get_value("hello"));
    println!("{}", get_value("0"));
}
```

### Q5
```rust
#[derive(Debug)]
enum Message { Move { x: i32, y: i32 }, Quit, Print(String) }

fn main() {
    let msgs = vec![
        Message::Move { x: 3, y: 4 },
        Message::Quit,
        Message::Print(String::from("hi")),
        Message::Move { x: 0, y: 0 },
    ];
    for m in &msgs {
        if let Message::Move { x, y } = m {
            println!("move ({x},{y})");
        }
    }
}
```

---

## Section B — Will It Compile?

### Q6
```rust
fn main() {
    let x: Option<i32> = Some(5);
    if let Some(n) = x {
        println!("{n}");
    }
    println!("{n}");  // use n outside the if let
}
```

### Q7
```rust
fn main() {
    let val: Option<i32> = None;
    let Some(n) = val else {
        println!("no value");
    };
    println!("{n}");
}
```

### Q8
```rust
fn first_word(s: Option<&str>) -> &str {
    let Some(text) = s else { return ""; };
    text.split_whitespace().next().unwrap_or("")
}
fn main() {
    println!("{}", first_word(Some("hello world")));
    println!("{}", first_word(None));
}
```

### Q9
```rust
fn main() {
    let mut v = vec![1, 2, 3];
    while let Some(x) = v.pop() {
        v.push(x + 10);  // push while popping
    }
    println!("done");
}
```

### Q10
```rust
fn main() {
    let pair = (Some(1), Some(2));
    if let (Some(a), Some(b)) = pair {
        println!("{}", a + b);
    }
}
```

---

## Section C — Fix the Bugs

### Q11 — Binding used outside scope
```rust
fn main() {
    let config: Option<String> = Some(String::from("dark_mode"));
    if let Some(theme) = config {
        println!("Theme: {theme}");
    }
    println!("Theme was: {theme}");  // BUG: theme out of scope
}
```
Fix without changing to a `match`.

### Q12 — let else missing diverge
```rust
fn process(val: Option<i32>) -> i32 {
    let Some(n) = val else {
        println!("no value");  // BUG: doesn't diverge
    };
    n * 2
}
fn main() { println!("{}", process(Some(5))); }
```

### Q13 — while let infinite loop
```rust
fn main() {
    let mut count = 0u32;
    while let x = Some(count) {   // BUG: pattern always matches
        println!("{count}");
        count += 1;
        if count > 3 { break; }
    }
}
```
Fix to only continue while `count <= 3` using `while let` correctly. Hint: change the value being matched.

### Q14 — Refactor match to if let
```rust
fn print_if_some(val: Option<&str>) {
    match val {
        Some(s) => println!("Value: {s}"),
        _       => {}   // do nothing
    }
}
```
Rewrite using `if let`.

---

## Section D — Write It Yourself

### Q15 — Roadmap Practice Task: Refactor match to if let
Take this code and refactor every `match` that has a trivial `_ => {}` arm to use `if let` instead. Keep `match` only where it handles multiple variants meaningfully.

```rust
#[derive(Debug)]
enum Event { Click(i32, i32), KeyPress(char), Resize(u32, u32), Quit }

fn handle_events(events: &[Event]) {
    for e in events {
        match e {
            Event::Click(x, y) => println!("click at ({x},{y})"),
            _ => {}
        }

        match e {
            Event::KeyPress(c) => println!("key: {c}"),
            _ => {}
        }

        match e {
            Event::Quit => println!("quitting"),
            _ => {}
        }

        match e {
            Event::Click(x, y) | Event::Resize(x, y) => {
                println!("coords: {x},{y}")
            }
            _ => {}
        }
    }
}

fn main() {
    handle_events(&[
        Event::Click(1, 2),
        Event::KeyPress('A'),
        Event::Resize(800, 600),
        Event::Quit,
    ]);
}
```

### Q16 — Stack processor with while let
Build a `fn process_stack(stack: &mut Vec<i32>) -> i32` that:
- Pops values one at a time using `while let`
- Skips negative values (continue)
- Stops early if it sees a value > 100 (break with that value as return)
- Returns the sum of all consumed positive values otherwise

### Q17 — Input parser with let else
Write `fn parse_record(line: &str) -> Option<(String, u32, f64)>` that:
- Splits the line on commas: `"Alice,30,9.5"`
- Extracts name (String), age (u32), score (f64)
- Returns `None` if anything fails to parse
- Use `let else` with `return None` for each failure point

Test with `"Alice,30,9.5"`, `"Bob,bad,7.0"`, `"short"`.

### Q18 — Config loader
```rust
struct Config { host: String, port: u16, debug: bool }
```
Write `fn load_config(host: Option<&str>, port: Option<u16>, debug: Option<bool>) -> Config` that:
- Uses `if let` to override each field if Some
- Defaults: host="localhost", port=3000, debug=false
- Returns the Config

### Q19 — Drain and transform
Write `fn drain_positive(source: &mut Vec<i32>) -> Vec<i32>` that:
- Uses `while let` to pop from source
- Collects only positive values into the result Vec
- Stops if it encounters a zero

### Q20 — Full program: Message processor
```rust
#[derive(Debug)]
enum Msg {
    Login { user: String, password: String },
    Post  { user: String, content: String },
    Logout(String),
    Ping,
}
```
Write a `fn process(msg: &Msg) -> String` that:
- Uses `if let` for `Login` — returns `"LOGIN: user"` or `"AUTH_FAIL"` if password is empty
- Uses `if let` for `Post` — returns `"POST by user: content"` (truncate content to 20 chars)
- Uses `if let` for `Logout` — returns `"LOGOUT: user"`
- Returns `"PONG"` for `Ping`

Also write `fn run_session(msgs: Vec<Msg>)` that uses `while let` on a VecDeque to process messages one at a time, stopping on `Logout`.

---

## Section E — Deep Understanding

### Q21 — True or False?
1. `if let Some(x) = val` — `x` is available after the closing `}`.
2. `while let` stops when the pattern does not match.
3. `let else` requires the else block to return, break, continue, or panic.
4. `if let` is always better than `match` for single-variant handling.
5. `let Some(x) = val else { return; };` — `x` is available on the next line.
6. You can use `if let` with struct variants: `if let Event::Click { x, y } = e`.
7. `while let Some(x) = iter.next()` is equivalent to a `for` loop over the iterator.
8. `let else` was available since Rust 1.0.
9. You can chain `if let` with `&&` conditions in Rust 1.64+.
10. `if let` is purely syntactic sugar — it compiles to the same code as `match`.

### Q22 — Refactoring challenge
Rewrite this deeply nested code using `let else` to make it flat:

```rust
fn get_port(config: Option<&str>) -> u16 {
    if let Some(s) = config {
        if let Ok(n) = s.parse::<u16>() {
            if n > 0 && n < 65535 {
                return n;
            }
        }
    }
    8080  // default
}
```

### Q23 — Spot the difference
What is different between these two, and when does it matter?

```rust
// Version A
fn handle_a(val: Option<String>) {
    if let Some(s) = val {
        println!("got: {s}");
    }
    // can still use val here? What happened to it?
}

// Version B
fn handle_b(val: Option<String>) {
    let Some(s) = val else { return; };
    println!("got: {s}");
    // can still use val here?
}
```
