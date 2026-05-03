# 🧪 Lesson 50 — Questions: Higher-Order Functions (CL3)

> **Lesson:** [lesson_50_higher_order.md](./lesson_50_higher_order.md)  
> **Answers:** [lesson_50_answers.md](./lesson_50_answers.md)

---

## Section A — Predict the Output

### Q1
```rust
fn apply(x: i32, f: impl Fn(i32) -> i32) -> i32 { f(x) }
fn main() { println!("{}", apply(5, |x| x * x + 1)); }
```

### Q2
```rust
fn make_adder(n: i32) -> impl Fn(i32) -> i32 {
    move |x| x + n
}
fn main() {
    let f = make_adder(10);
    println!("{} {}", f(5), f(20));
}
```

### Q3
```rust
fn twice(f: impl Fn(i32) -> i32, x: i32) -> i32 { f(f(x)) }
fn main() { println!("{}", twice(|x| x + 3, 10)); }
```

---

## Section B — Write It Yourself

### Q4 — compose function
Write `fn compose<A, B, C>(f: impl Fn(A) -> B, g: impl Fn(B) -> C) -> impl Fn(A) -> C`. Test by composing `double` and `to_string`.

### Q5 — Pipeline combinator (Roadmap Practice Task)
Build a `Pipeline<T>` struct with `.pipe(f)` and `.finish()` methods. Process a list of numbers: sort → filter > 3 → square each → sum.

### Q6 — Function registry
Build a `HashMap<String, Box<dyn Fn(f64) -> f64>>` containing math functions (`sqrt`, `abs`, `negate`). Look up and apply them by name.

---

## Section C — Deep Understanding

### Q7 — True or False?
1. `map`, `filter`, and `fold` are higher-order functions.
2. Rust supports automatic currying like Haskell.
3. `impl Fn(T) -> U` in return position requires `move` if the closure captures variables.
4. Functional and imperative styles produce the same result but may differ in readability.
5. A function that returns a closure is a higher-order function.

### Q8
Compare these two approaches for summing squares of even numbers from 1 to 100. Which is more idiomatic in Rust?

```rust
// Version A
let mut sum = 0;
for i in 1..=100 {
    if i % 2 == 0 { sum += i * i; }
}

// Version B
let sum: i32 = (1..=100).filter(|x| x % 2 == 0).map(|x| x * x).sum();
```

---

*50 lessons complete — you're a Rust intermediate now! 🦀🎉*
