# ✅ Lesson 08 — Answers: Ownership Rules

> Questions are in [`lesson_08_questions.md`](./lesson_08_questions.md)

---

## Section A — Predict: Compile or Error?

### A1 — Error
`s` is moved into `t`. Using `s` after the move is a compile error:
`error[E0382]: borrow of moved value: 's'`

### A2 — Compiles ✅
`i32` implements the `Copy` trait. Assigning `x` to `y` copies the value — both `x` and `y` are valid independently.

### A3 — Error
`s` is moved into `eat()`. Trying to use `s` after the call is a compile error.

### A4 — Compiles ✅
`s.clone()` creates a deep copy. `eat()` receives and drops the clone. The original `s` is unaffected and can still be used.

### A5 — Compiles ✅
`give()` creates a `String`, moves it out as its return value. `g` in `main` takes ownership. No double-drop — ownership transferred cleanly.

### A6 — Error
`bad()` returns a reference to a local variable `s`. When `bad()` returns, `s` is dropped — the reference would be dangling. Rust catches this:
`error[E0106]: missing lifetime specifier` / "returns a reference to data owned by the current function".

### A7 — Compiles ✅
`.clone()` makes a deep copy on the heap. `s1` and `s2` own separate `String` values — both valid.

### A8 — Error
`s` is declared inside the inner block and is dropped at `}`. Accessing `s` outside that block:
`error[E0425]: cannot find value 's' in this scope`

---

## Section B — Trace the Ownership

### A9 — Drop Order
**Output:**
```
Created all three
Dropping: C
Dropping: B
Dropping: A
```

Variables are dropped in **reverse order of creation** (LIFO — last in, first out). `c` was created last, so it's dropped first, then `b`, then `a`. This mirrors the stack discipline.

---

### A10 — Move Through Function

```
(a) s is MOVED into the + operator (String's add consumes self)
    The string "hello world" is constructed and owned by `result`
(b) `result` (the String "hello world") is returned — ownership moves to the caller
(c) s1 owns the String "hello" on the heap
(d) s1 is NO LONGER VALID — ownership moved into process()
    s2 owns the returned String "hello world"
(e) prints "hello world"
```

---

### A11 — Scope Lifetimes

- **`b` dropped at line 7** (end of inner block)
- **`c` dropped at line 7** (end of inner block, before `b` — reverse order: c then b)
- **`d` dropped at line 10** (end of `main`)
- **`a` dropped at line 10** (end of `main`, after `d` — reverse order: d then a)

Full drop sequence at line 7: `c`, then `b`.
Full drop sequence at line 10: `d`, then `a`.

---

## Section C — Fix the Errors

### A12 — Use after move

```rust
// ✅ Fix (a): clone before first call
fn display(s: String) { println!("Displaying: {s}"); }
fn main() {
    let message = String::from("Hello, ownership!");
    display(message.clone()); // clone — original unaffected
    display(message);         // now move the original
}

// ✅ Fix (b): change signature to borrow — better!
fn display(s: &str) { println!("Displaying: {s}"); }
fn main() {
    let message = String::from("Hello, ownership!");
    display(&message); // borrow
    display(&message); // borrow again — message still owned by main
}
```

---

### A13 — Dangling return

**Problem:** `first_word` creates a local `String`, takes a reference to it, and tries to return that reference. But `word` (the String) is dropped when the function returns — the reference would point to freed memory.

```rust
// ✅ Fix: return an owned String
fn first_word(sentence: &str) -> String {
    String::from("hello") // own it and return it
}

// ✅ Even better for real use: return a &str slice of the input
fn first_word(sentence: &str) -> &str {
    sentence.split_whitespace().next().unwrap_or("")
}

fn main() {
    let result = first_word("hello world");
    println!("{result}"); // "hello"
}
```

---

### A14 — Partial move

**What happens:** `pair.0` is moved out of the tuple into `first`. Now `pair.0` is invalid (moved). `pair.1` is still valid (not moved). Trying to use `pair.0` after moving it is a compile error.

```rust
// ✅ Fixed using clone
fn main() {
    let pair = (String::from("hello"), String::from("world"));
    let first = pair.0.clone(); // clone, don't move
    println!("{}", pair.0);    // ✅ pair.0 is still valid
    println!("{}", pair.1);    // ✅ fine
    println!("{first}");       // ✅ the clone
}
```

---

### A15 — Multiple ownership errors

**Problem 1:** `summarise` takes `data: Vec<i32>` — moves the Vec. The caller can't use `numbers` again.
**Problem 2:** (Actually none in the function itself — the bugs are at the call site)
**Problem 3:** After `summarise(numbers)`, `numbers` is moved. `println!("{:?}", numbers)` fails.

```rust
// ✅ Fixed: borrow instead of taking ownership
fn summarise(data: &[i32]) -> String {  // borrow a slice
    let sum: i32 = data.iter().sum();
    let count = data.len();
    let avg = sum as f64 / count as f64;
    format!("sum={sum} count={count} avg={avg:.1}")
}

fn main() {
    let numbers = vec![1, 2, 3, 4, 5];
    let summary = summarise(&numbers);  // lend a slice reference
    println!("{summary}");
    println!("Original: {:?}", numbers); // ✅ numbers still valid
}
```

---

## Section D — Write It Yourself

### A16 — Ownership chain

```rust
fn clean(s: String) -> String {
    s.trim().to_string()
}

fn shout(s: String) -> String {
    s.to_uppercase()
}

fn main() {
    let raw = String::from("  hello, rust!  ");
    println!("raw created");

    let cleaned = clean(raw);
    // raw is no longer valid here — moved into clean()
    println!("cleaned: '{cleaned}'");

    let shouted = shout(cleaned);
    // cleaned is no longer valid — moved into shout()
    println!("shouted: '{shouted}'");

    // shouted is still valid here
}
```

**Output:**
```
raw created
cleaned: 'hello, rust!'
shouted: 'HELLO, RUST!'
```

---

### A17 — Custom Drop

```rust
struct TempFile {
    path: String,
}

impl Drop for TempFile {
    fn drop(&mut self) {
        println!("Deleted temp file: {}", self.path);
    }
}

fn main() {
    let file1 = TempFile { path: String::from("/tmp/file1.txt") };
    let file2 = TempFile { path: String::from("/tmp/file2.txt") };

    println!("Both files created");

    drop(file1); // explicitly drop file1 early
    println!("After explicit drop of file1");

    // file2 is dropped automatically here at end of scope
}
```

**Output:**
```
Both files created
Deleted temp file: /tmp/file1.txt
After explicit drop of file1
Deleted temp file: /tmp/file2.txt
```

---

### A18 — Ownership with Collections

```rust
fn take_largest(v: Vec<i32>) -> (i32, Vec<i32>) {
    let largest = *v.iter().max().unwrap();
    (largest, v) // return both — move Vec back to caller
}

fn main() {
    let numbers = vec![3, 1, 4, 1, 5, 9, 2, 6];
    let (max, numbers) = take_largest(numbers); // shadow with returned Vec
    println!("Largest: {max}");
    println!("Original Vec: {:?}", numbers); // ✅ accessible again
}
```

**Output:**
```
Largest: 9
Original Vec: [3, 1, 4, 1, 5, 9, 2, 6]
```

---

## Section E — Deep Understanding

### A19 — True or False?

| # | Statement | Answer | Explanation |
|---|---|---|---|
| 1 | A value can have two owners | **False** | Only ONE owner at a time — rule 2 |
| 2 | Both vars own data after String assign | **False** | The first var is invalidated (moved) |
| 3 | `i32` assignment moves ownership | **False** | `i32` is `Copy` — it's copied, not moved |
| 4 | `.clone()` creates deep heap copy | **True** | Full independent copy of heap data |
| 5 | Value dropped even if moved from | **False** | The MOVED-TO owner drops it, not the original |
| 6 | `drop(x)` frees memory immediately | **True** | Yes — calls destructor and frees heap immediately |
| 7 | Variables dropped in creation order | **False** | Dropped in REVERSE order (LIFO) |
| 8 | Returning always copies | **False** | Ownership MOVES out — no copy |
| 9 | Ownership enforced by GC at runtime | **False** | Enforced by the COMPILER at compile time — zero runtime cost |
| 10 | `fn eat(s: String)` frees the String | **True** | When `eat` returns, `s` goes out of scope and is dropped |

---

### A20 — Big Debug Challenge

**Bug 1:** `greet(name)` moves `name`. Calling `greet(name)` a second time fails — `name` was already moved.
```rust
// Fix: clone for the first call, or change greet to borrow
fn greet(name: &str) { println!("Hello, {name}!"); }
greet(&name); greet(&name);
```

**Bug 2:** `get_length(s)` takes ownership of `s`. After the call, `s` is moved — can't use it.
```rust
fn get_length(s: &str) -> usize { s.len() }
let len = get_length(&s);
println!("{s} has {len} chars"); // ✅
```

**Bug 3:** `let b = a` moves `a`. Using `a` after the move is an error.
```rust
let a = String::from("owned");
let b = a.clone(); // or let b = a; and don't use a
println!("{b}");
```

**Bug 4:** `r = &temp` stores a reference to `temp`. `temp` is dropped at `}`. `r` would be dangling — caught at compile time.
```rust
// Fix: either return owned value or restructure scope
let temp = String::from("temporary");
let r = &temp; // keep temp and r in same scope
println!("{r}");
```

**Bug 5:** NOT a bug. `v[0]` for `Vec<i32>` copies the `i32` (it's `Copy`). `first = 1` is a copy. `drop(v)` drops the Vec. `first` is still valid — it's an independent copy on the stack.

**Bug 6:** `let s2 = s1` moves `s1`. Then `let s3 = s1` tries to use the already-moved `s1`.
```rust
let s1 = String::from("hello");
let s2 = s1.clone();
let s3 = s1; // move s1 into s3
println!("{s2} {s3}");
```

**Corrected program:**
```rust
fn greet(name: &str) {
    println!("Hello, {name}!");
}

fn get_length(s: &str) -> usize {
    s.len()
}

fn main() {
    let name = String::from("Alice");
    greet(&name);
    greet(&name);          // ✅ borrows — name still valid

    let s = String::from("measure me");
    let len = get_length(&s);
    println!("{s} has {len} chars");  // ✅

    let a = String::from("owned");
    let b = a.clone();
    let _ = a;             // or just don't use a after b
    println!("{b}");

    let temp = String::from("temporary");
    let r = &temp;         // r and temp in same scope
    println!("{r}");

    let v = vec![1, 2, 3];
    let first = v[0];      // copies i32 — fine
    drop(v);
    println!("{first}");   // ✅ fine — first is a stack copy

    let s1 = String::from("hello");
    let s2 = s1.clone();
    let s3 = s1;
    println!("{s2} {s3}"); // ✅
}
```

---

## 🏆 Lesson Complete!

You've mastered Rust's foundational memory model:
- ✅ Why Rust needs ownership (no GC, no manual memory, no bugs)
- ✅ Stack vs Heap — what lives where and why it matters
- ✅ The three ownership rules and what each prevents
- ✅ Move semantics — why assignment invalidates the source
- ✅ Scope and drop — automatic memory management at zero cost
- ✅ Custom Drop — running code when values are cleaned up
- ✅ Ownership through functions — moving in, moving back
- ✅ Common errors and how to fix them

**Up next:** [Lesson 09 — Move vs Copy](../lesson_09_move_copy/lesson_09_move_copy.md) 🦀
