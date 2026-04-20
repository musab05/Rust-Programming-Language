# ✅ Lesson 16 — Answers: `if let` / `while let` / `let else` (S5)

---

## Section A — Predict the Output

### A1
```
10 30 50 
```
`if let Some(n)` fires for the three `Some` values, skips `None`.

### A2
```
c b a done
```
`pop()` returns elements last-in-first-out. Stops when pop returns `None`.

### A3
```
positive
non-positive
none
```

### A4
```
50
0
0
```
`"5".parse::<u32>()` = `Ok(5)` → 5×10 = 50. `"hello"` parse fails → returns 0. `"0"` → 0×10 = 0.

### A5
```
move (3,4)
move (0,0)
```
Only `Move` variants match `if let Message::Move { x, y }`. `Quit` and `Print` are skipped.

---

## Section B — Will It Compile?

### A6 — ❌ No
`n` is bound inside the `if let` block only. It's not accessible after the closing `}`. **Error:** cannot find value `n` in this scope.

### A7 — ❌ No
The `else` block of `let else` **must diverge** — it must `return`, `break`, `continue`, `panic!()`, or loop forever. Just `println!` doesn't diverge. **Error:** `else` block in `let...else` must be diverging.

### A8 — ✅ Yes
`let Some(text) = s else { return ""; };` — if `s` is `None`, returns `""`. Otherwise `text` holds the `&str`. Clean and correct. Output: `hello\n`.

### A9 — ❌ Infinite loop (compiles, but hangs)
`v.pop()` removes an element, then `v.push(x + 10)` adds one back. The Vec never becomes empty, so `pop()` never returns `None`. The `while let` loops forever. Fix: remove the push, or track a counter to stop.

### A10 — ✅ Yes
Tuple pattern `(Some(a), Some(b))` matches when both are `Some`. `pair` is moved in but `Copy` for `(Option<i32>, Option<i32>)` doesn't apply — however `i32` is `Copy` so `Option<i32>` is `Copy`. Output: `3`.

---

## Section C — Fix the Bugs

### A11 — Binding used outside scope
```rust
fn main() {
    let config: Option<String> = Some(String::from("dark_mode"));

    // Fix: use let else so the binding is available afterward
    let Some(theme) = config else {
        println!("No theme set");
        return;
    };
    println!("Theme: {theme}");
    println!("Theme was: {theme}");  // ✅ now in scope
}
```
Alternatively, clone the value before the `if let`:
```rust
fn main() {
    let config: Option<String> = Some(String::from("dark_mode"));
    let display = config.as_deref().unwrap_or("none").to_string();
    if let Some(theme) = &config {
        println!("Theme: {theme}");
    }
    println!("Theme was: {display}");
}
```

### A12 — let else missing diverge
```rust
fn process(val: Option<i32>) -> i32 {
    let Some(n) = val else {
        return 0;   // fix: must diverge
    };
    n * 2
}
fn main() { println!("{}", process(Some(5))); } // 10
```

### A13 — while let infinite loop
```rust
// Fix: match on a computed value that eventually becomes None
fn main() {
    let mut count = 0u32;
    while let Some(c) = if count <= 3 { Some(count) } else { None } {
        println!("{c}");
        count += 1;
    }
}
// Simpler approach — just use a regular while:
fn main() {
    let mut count = 0u32;
    while count <= 3 {
        println!("{count}");
        count += 1;
    }
}
```
The original bug: `while let x = Some(count)` — a bare name `x` is a binding pattern that always matches `Some(count)`. Since `Some(count)` always succeeds, the loop never exits via the pattern. The `break` was the only exit.

### A14 — Refactor match to if let
```rust
fn print_if_some(val: Option<&str>) {
    if let Some(s) = val {
        println!("Value: {s}");
    }
}
```

---

## Section D — Write It Yourself

### A15 — Roadmap Practice Task: Refactor to if let
```rust
#[derive(Debug)]
enum Event { Click(i32, i32), KeyPress(char), Resize(u32, u32), Quit }

fn handle_events(events: &[Event]) {
    for e in events {
        // ✅ single-variant matches → if let
        if let Event::Click(x, y) = e {
            println!("click at ({x},{y})");
        }

        if let Event::KeyPress(c) = e {
            println!("key: {c}");
        }

        if let Event::Quit = e {
            println!("quitting");
        }

        // ✅ OR pattern across variants — keep match (if let can't do this cleanly)
        match e {
            Event::Click(x, y) | Event::Resize(x, y) => println!("coords: {x},{y}"),
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
**Output:**
```
click at (1,2)
coords: 1,2
key: A
coords: 800,600
quitting
```

### A16 — Stack processor with while let
```rust
fn process_stack(stack: &mut Vec<i32>) -> i32 {
    let mut sum = 0;
    while let Some(val) = stack.pop() {
        if val < 0 { continue; }
        if val > 100 { return val; }
        sum += val;
    }
    sum
}

fn main() {
    let mut s1 = vec![10, -5, 20, 30];
    println!("{}", process_stack(&mut s1)); // 60

    let mut s2 = vec![5, 200, 10];
    println!("{}", process_stack(&mut s2)); // 200 (early return)

    let mut s3 = vec![-1, -2, -3];
    println!("{}", process_stack(&mut s3)); // 0 (all negative, skipped)
}
```

### A17 — Input parser with let else
```rust
fn parse_record(line: &str) -> Option<(String, u32, f64)> {
    let mut parts = line.split(',');

    let Some(name_str) = parts.next() else { return None; };
    let Some(age_str)  = parts.next() else { return None; };
    let Some(score_str) = parts.next() else { return None; };

    let name = name_str.trim().to_string();

    let Ok(age) = age_str.trim().parse::<u32>() else { return None; };
    let Ok(score) = score_str.trim().parse::<f64>() else { return None; };

    Some((name, age, score))
}

fn main() {
    println!("{:?}", parse_record("Alice,30,9.5"));    // Some(("Alice", 30, 9.5))
    println!("{:?}", parse_record("Bob,bad,7.0"));     // None
    println!("{:?}", parse_record("short"));            // None
}
```

### A18 — Config loader
```rust
struct Config { host: String, port: u16, debug: bool }

fn load_config(host: Option<&str>, port: Option<u16>, debug: Option<bool>) -> Config {
    let mut cfg = Config {
        host: String::from("localhost"),
        port: 3000,
        debug: false,
    };

    if let Some(h) = host  { cfg.host  = h.to_string(); }
    if let Some(p) = port  { cfg.port  = p; }
    if let Some(d) = debug { cfg.debug = d; }

    cfg
}

fn main() {
    let c = load_config(Some("example.com"), None, Some(true));
    println!("{}:{} debug={}", c.host, c.port, c.debug);
    // example.com:3000 debug=true

    let d = load_config(None, None, None);
    println!("{}:{} debug={}", d.host, d.port, d.debug);
    // localhost:3000 debug=false
}
```

### A19 — Drain and transform
```rust
fn drain_positive(source: &mut Vec<i32>) -> Vec<i32> {
    let mut result = Vec::new();
    while let Some(val) = source.pop() {
        if val == 0 { break; }
        if val > 0 { result.push(val); }
    }
    result
}

fn main() {
    let mut v = vec![1, -2, 3, 0, 5, 6];
    let pos = drain_positive(&mut v);
    println!("{:?}", pos);    // [6, 5] (popped from end, stops at 0)
    println!("{:?}", v);      // [1, -2, 3] (remaining)
}
```

### A20 — Message processor
```rust
use std::collections::VecDeque;

#[derive(Debug)]
enum Msg {
    Login { user: String, password: String },
    Post  { user: String, content: String },
    Logout(String),
    Ping,
}

fn process(msg: &Msg) -> String {
    if let Msg::Login { user, password } = msg {
        return if password.is_empty() {
            String::from("AUTH_FAIL")
        } else {
            format!("LOGIN: {user}")
        };
    }
    if let Msg::Post { user, content } = msg {
        let preview: String = content.chars().take(20).collect();
        return format!("POST by {user}: {preview}");
    }
    if let Msg::Logout(user) = msg {
        return format!("LOGOUT: {user}");
    }
    String::from("PONG")
}

fn run_session(msgs: Vec<Msg>) {
    let mut queue = VecDeque::from(msgs);
    while let Some(msg) = queue.pop_front() {
        let response = process(&msg);
        println!("{response}");
        if let Msg::Logout(_) = &msg { break; }
    }
}

fn main() {
    run_session(vec![
        Msg::Login  { user: "alice".into(), password: "s3cret".into() },
        Msg::Ping,
        Msg::Post   { user: "alice".into(), content: "Hello Rust world!".into() },
        Msg::Logout("alice".into()),
        Msg::Ping,  // not processed — session ended at Logout
    ]);
}
```
**Output:**
```
LOGIN: alice
PONG
POST by alice: Hello Rust world!
LOGOUT: alice
```

---

## Section E — Deep Understanding

### A21 — True or False?
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **False** | `if let` bindings are scoped to the `if` block only |
| 2 | **True** | `while let` continues while the pattern matches, stops on mismatch |
| 3 | **True** | The `else` block of `let else` must diverge — `return`, `break`, `continue`, or `panic!` |
| 4 | **False** | `match` is better when multiple variants need different handling, or when exhaustiveness matters |
| 5 | **True** | `let else` bindings live in the enclosing scope after the statement |
| 6 | **True** | `if let Event::Click { x, y } = e { ... }` works for struct-style enum variants |
| 7 | **True** | Both consume the iterator via `.next()` until `None`, but `for` is idiomatic |
| 8 | **False** | `let else` was stabilized in Rust **1.65** (November 2022) |
| 9 | **True** | `if let Some(x) = val && x > 10` — let-chain syntax from Rust 1.64+ (RFC 2497) |
| 10 | **True** | The compiler lowers `if let` to the same MIR/code as a `match` with a wildcard arm |

### A22 — Refactoring with let else
```rust
fn get_port(config: Option<&str>) -> u16 {
    let Some(s) = config else { return 8080; };
    let Ok(n)   = s.parse::<u16>() else { return 8080; };
    if n == 0 || n >= 65535 { return 8080; }
    n
}

fn main() {
    println!("{}", get_port(Some("9000")));  // 9000
    println!("{}", get_port(Some("bad")));   // 8080
    println!("{}", get_port(None));           // 8080
}
```
The flat version is easier to read: each `let else` handles one failure case, and at the end of all the guards, `n` is valid.

### A23 — Spot the difference
**Version A — `if let`:**
```rust
fn handle_a(val: Option<String>) {
    if let Some(s) = val {  // val is MOVED into the if let
        println!("got: {s}");
    }
    // val is no longer accessible here — it was moved
    // println!("{:?}", val); // ERROR: use of moved value
}
```

**Version B — `let else`:**
```rust
fn handle_b(val: Option<String>) {
    let Some(s) = val else { return; };  // val is MOVED here
    println!("got: {s}");
    // val is no longer accessible here either — same ownership situation
    // println!("{:?}", val); // ERROR: use of moved value
}
```

**The key difference:**
- In **A**: `s` is only available *inside* the `if` block. Code after the block can't use `s`.
- In **B**: `s` is available *after* the `let else` statement, in the rest of the function. This is the main advantage.

**When it matters:** If you need to use the destructured value for the rest of the function (not just a short block), `let else` is far cleaner. `if let` forces you to indent all the subsequent code inside the block.

---

## 🏆 Lesson 16 Complete!

✅ `if let` — concise single-pattern matching  
✅ `if let` + `else` — pattern with fallback  
✅ `if let` chains with `&&`  
✅ `while let` — loop while a pattern holds  
✅ `let else` — destructure or diverge (Rust 1.65+)  
✅ Binding scope difference: `if let` (inside) vs `let else` (after)  
✅ Choosing between `match`, `if let`, `while let`, `let else`  
✅ Refactored `match` to `if let` — roadmap practice ✓  

**Up next: [Lesson 17 — Tuple Structs & Unit Structs](../lesson_17_tuple_unit_structs/lesson_17_tuple_unit_structs.md) 🦀**
