# 🧪 Lesson 08 — Practice Questions: Ownership Rules

> Answers are in [`lesson_08_answers.md`](./lesson_08_answers.md)

---

## Section A — Predict: Compile or Error?

For each snippet, state whether it **compiles** or produces a **compile error**. If it errors, name the ownership rule being violated.

### Q1
```rust
fn main() {
    let s = String::from("hello");
    let t = s;
    println!("{s}");
}
```

### Q2
```rust
fn main() {
    let x = 42;
    let y = x;
    println!("{x} {y}");
}
```

### Q3
```rust
fn eat(s: String) { println!("{s}"); }
fn main() {
    let s = String::from("hello");
    eat(s);
    println!("{s}");
}
```

### Q4
```rust
fn eat(s: String) { println!("{s}"); }
fn main() {
    let s = String::from("hello");
    eat(s.clone());
    println!("{s}");
}
```

### Q5
```rust
fn give() -> String {
    let s = String::from("gift");
    s
}
fn main() {
    let g = give();
    println!("{g}");
}
```

### Q6
```rust
fn bad() -> &String {
    let s = String::from("local");
    &s
}
fn main() {
    let r = bad();
    println!("{r}");
}
```

### Q7
```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1.clone();
    println!("{s1} {s2}");
}
```

### Q8
```rust
fn main() {
    {
        let s = String::from("inner");
    }
    println!("{s}");
}
```

---

## Section B — Trace the Ownership

### Q9 — Drop Order

```rust
struct Loud(String);
impl Drop for Loud {
    fn drop(&mut self) { println!("Dropping: {}", self.0); }
}

fn main() {
    let a = Loud(String::from("A"));
    let b = Loud(String::from("B"));
    let c = Loud(String::from("C"));
    println!("Created all three");
}
```

**In what order are A, B, C dropped? Why?**

---

### Q10 — Move Through Function

Trace what each variable owns (or doesn't) at each marked line:

```rust
fn process(s: String) -> String {
    let result = s + " world";  // (a) what happens to s here?
    result                       // (b) what is returned?
}

fn main() {
    let s1 = String::from("hello"); // (c) s1 owns?
    let s2 = process(s1);           // (d) s1 valid? s2 owns?
    println!("{s2}");               // (e) what prints?
}
```

---

### Q11 — Scope Lifetimes

Mark where each variable is dropped:

```rust
fn main() {                              // line 1
    let a = String::from("a");           // line 2
    {                                    // line 3
        let b = String::from("b");       // line 4
        let c = String::from("c");       // line 5
        println!("{a} {b} {c}");         // line 6
    }                                    // line 7
    let d = String::from("d");           // line 8
    println!("{a} {d}");                 // line 9
}                                        // line 10
```

**Where is each of `a`, `b`, `c`, `d` dropped?**

---

## Section C — Fix the Errors

### Q12 — Use after move

```rust
fn display(s: String) {
    println!("Displaying: {s}");
}

fn main() {
    let message = String::from("Hello, ownership!");
    display(message);
    display(message); // want to display it twice
}
```

**Fix it two ways: (a) clone, (b) change the function signature to borrow.**

---

### Q13 — Dangling return

```rust
fn first_word(sentence: &str) -> &String {
    let word = String::from("hello");
    &word
}

fn main() {
    let result = first_word("hello world");
    println!("{result}");
}
```

**What is the ownership problem? How do you fix it?**

---

### Q14 — Partial move

```rust
fn main() {
    let pair = (String::from("hello"), String::from("world"));
    let first = pair.0;  // move first element out
    println!("{}", pair.0); // try to use it
    println!("{}", pair.1); // still valid?
}
```

**What happens? Fix it using `.clone()`.**

---

### Q15 — Multiple ownership errors

The following function has 3 ownership-related problems. Find and fix all of them:

```rust
fn summarise(data: Vec<i32>) -> String {
    let sum: i32 = data.iter().sum();
    let count = data.len();
    let avg = sum as f64 / count as f64;
    let result = format!("sum={sum} count={count} avg={avg:.1}");
    result
}

fn main() {
    let numbers = vec![1, 2, 3, 4, 5];
    let summary = summarise(numbers);
    println!("{summary}");
    println!("Original: {:?}", numbers); // want to use numbers again
}
```

---

## Section D — Write It Yourself

### Q16 — Ownership chain

Write a program that:
1. Creates a `String` named `raw`
2. Passes it to a function `clean(s: String) -> String` that trims whitespace and returns the owned string
3. Passes the result to a function `shout(s: String) -> String` that uppercases and returns it
4. Prints the final result
5. Shows that each intermediate variable is no longer valid after being moved

---

### Q17 — Custom Drop

Write a struct `TempFile` with a field `path: String`. Implement `Drop` so that when a `TempFile` is dropped it prints `"Deleted temp file: {path}"`. In `main`:
- Create two TempFiles
- Use `std::mem::drop` to explicitly drop the first one early
- Show that the second is dropped automatically at end of scope

---

### Q18 — Ownership with Collections

Write a function `take_largest(v: Vec<i32>) -> (i32, Vec<i32>)` that:
- Takes ownership of the Vec
- Finds the largest value
- Returns BOTH the largest value AND the original Vec (so the caller gets ownership back)

In `main`, demonstrate that the original Vec is accessible again after the call.

---

## Section E — Deep Understanding

### Q19 — True or False?

1. A value can have two owners at the same time in Rust.
2. When you assign a `String` to another variable, both variables own the data.
3. When an `i32` is assigned to another variable, ownership is moved.
4. Calling `.clone()` on a String creates a deep copy on the heap.
5. A value is dropped when it goes out of scope, even if it was moved from.
6. `drop(x)` frees memory immediately, before the scope ends.
7. Variables are dropped in the order they are created.
8. Returning a value from a function always copies it.
9. The ownership system is enforced at runtime by the garbage collector.
10. A function that takes `s: String` is responsible for eventually freeing that String.

---

### Q20 — Big Debug Challenge (6 ownership bugs)

```rust
fn greet(name: String) {
    println!("Hello, {name}!");
}

fn get_length(s: String) -> usize {
    s.len()
}

fn main() {
    let name = String::from("Alice");

    greet(name);
    greet(name);           // Bug 1

    let s = String::from("measure me");
    let len = get_length(s);
    println!("{s} has {len} chars");  // Bug 2

    let a = String::from("owned");
    let b = a;
    println!("{a}");       // Bug 3

    let r;
    {
        let temp = String::from("temporary");
        r = &temp;         // Bug 4
    }
    println!("{r}");

    let v = vec![1, 2, 3];
    let first = v[0];
    drop(v);
    println!("{first}");   // Bug 5? Or is this fine?

    let s1 = String::from("hello");
    let s2 = s1;
    let s3 = s1;           // Bug 6
    println!("{s2} {s3}");
}
```

**Identify each bug, explain the ownership violation, and provide the corrected program.**

---

*"Ownership is not a limitation — it's a superpower. It makes you think about data flow, and data flow is everything." 🦀*
