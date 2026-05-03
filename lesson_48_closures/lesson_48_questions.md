# 🧪 Lesson 48 — Questions: Closures (CL1)

> **Lesson:** [lesson_48_closures.md](./lesson_48_closures.md)  
> **Answers:** [lesson_48_answers.md](./lesson_48_answers.md)

---

## Section A — Predict: Compile or Not?

### Q1
```rust
fn main() {
    let x = 10;
    let add_x = |n| n + x;
    println!("{}", add_x(5));
    println!("x = {x}");
}
```

### Q2
```rust
fn main() {
    let name = String::from("Alice");
    let greet = move || println!("Hello, {name}!");
    greet();
    println!("{name}");
}
```

### Q3
```rust
fn main() {
    let mut v = vec![1, 2, 3];
    let mut push = || v.push(4);
    push();
    push();
    println!("{:?}", v);
}
```

---

## Section B — Write It Yourself

### Q4 — Configurable filter (Roadmap Practice Task)
Write a `filter_by` function that takes a slice and a closure predicate. Use it to:
1. Filter even numbers from `[1..=10]`
2. Filter words longer than 4 characters
3. Filter using a captured variable (threshold)

### Q5 — Counter closure
Create a closure that captures a mutable counter variable and increments it each time it's called, returning the new count.

### Q6 — move with threads
Spawn a thread that prints a Vec. Explain why `move` is necessary.

---

## Section C — Deep Understanding

### Q7 — True or False?
1. Closures can capture variables; regular functions cannot.
2. `move` always transfers ownership of captured variables.
3. A closure's types are inferred from the first call only.
4. `move` with `Copy` types copies the value into the closure.
5. Every closure has a unique anonymous type.
6. You can name a closure's type directly (e.g., `let x: ClosureName = ...`).

### Q8
Explain the three automatic capture modes. What determines which one Rust uses?

---

*Closures: anonymous functions with superpowers! 🦀*
