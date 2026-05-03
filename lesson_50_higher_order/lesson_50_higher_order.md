# 📘 Lesson 50 — Higher-Order Functions (CL3)

> **Series:** Rust From Zero · Intermediate Level  
> **Roadmap ID:** CL3 · Category: 🔒 Closures  
> **Previous:** [Lesson 49 — Fn, FnMut, FnOnce](../lesson_49_fn_traits/lesson_49_fn_traits.md)  
> **Next:** Lesson 51 — Smart Pointers: Box *(coming soon)*  
> **Practice:** [Questions](./lesson_50_questions.md) · [Answers](./lesson_50_answers.md)  
> **Practice Task:** Build a function pipeline combinator

---

## Table of Contents

1. [What Are Higher-Order Functions?](#1-what-are-higher-order-functions)
2. [Functions That Take Functions](#2-functions-that-take-functions)
3. [Functions That Return Functions](#3-functions-that-return-functions)
4. [Function Composition](#4-function-composition)
5. [Iterator Methods as HOFs](#5-iterator-methods-as-hofs)
6. [Currying and Partial Application](#6-currying-and-partial-application)
7. [Functional Pipelines](#7-functional-pipelines)
8. [Imperative vs Functional Style](#8-imperative-vs-functional-style)
9. [Real-World Example: Pipeline Combinator](#9-real-world-example-pipeline-combinator)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. What Are Higher-Order Functions?

A **higher-order function** (HOF) is a function that:
- Takes one or more functions as arguments, OR
- Returns a function

You've already used many: `map`, `filter`, `fold`, `for_each`.

```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5];

    // map — takes a function, returns transformed iterator
    let doubled: Vec<i32> = numbers.iter().map(|x| x * 2).collect();

    // filter — takes a predicate function
    let evens: Vec<&i32> = numbers.iter().filter(|&&x| x % 2 == 0).collect();

    // fold — takes an accumulator function
    let sum: i32 = numbers.iter().fold(0, |acc, x| acc + x);

    println!("Doubled: {:?}", doubled);
    println!("Evens: {:?}", evens);
    println!("Sum: {sum}");
}
```

---

## 2. Functions That Take Functions

```rust
fn apply<F, T, R>(value: T, f: F) -> R
where
    F: Fn(T) -> R,
{
    f(value)
}

fn apply_n<F>(mut value: i32, f: F, n: usize) -> i32
where
    F: Fn(i32) -> i32,
{
    for _ in 0..n {
        value = f(value);
    }
    value
}

fn apply_all<F>(items: &[i32], f: F) -> Vec<i32>
where
    F: Fn(i32) -> i32,
{
    items.iter().map(|&x| f(x)).collect()
}

fn main() {
    println!("{}", apply(5, |x| x * x));        // 25
    println!("{}", apply("hi", |s| s.len()));    // 2

    println!("{}", apply_n(2, |x| x * 2, 5));   // 64 (2→4→8→16→32→64)

    let nums = vec![1, 2, 3, 4, 5];
    let squares = apply_all(&nums, |x| x * x);
    println!("{:?}", squares);  // [1, 4, 9, 16, 25]
}
```

### Accepting multiple closures:

```rust
fn branch<F, G, T>(condition: bool, value: T, on_true: F, on_false: G) -> T
where
    F: FnOnce(T) -> T,
    G: FnOnce(T) -> T,
{
    if condition { on_true(value) } else { on_false(value) }
}

fn main() {
    let result = branch(true, 10, |x| x * 2, |x| x + 100);
    println!("{result}");  // 20

    let result = branch(false, 10, |x| x * 2, |x| x + 100);
    println!("{result}");  // 110
}
```

---

## 3. Functions That Return Functions

```rust
fn make_adder(n: i32) -> impl Fn(i32) -> i32 {
    move |x| x + n
}

fn make_greeter(greeting: String) -> impl Fn(&str) -> String {
    move |name| format!("{greeting}, {name}!")
}

fn make_validator(min: i32, max: i32) -> impl Fn(i32) -> bool {
    move |x| x >= min && x <= max
}

fn main() {
    let add_5 = make_adder(5);
    let add_10 = make_adder(10);
    println!("{}, {}", add_5(3), add_10(3));  // 8, 13

    let hello = make_greeter("Hello".into());
    let hi = make_greeter("Hi".into());
    println!("{}", hello("Alice"));  // Hello, Alice!
    println!("{}", hi("Bob"));       // Hi, Bob!

    let valid_age = make_validator(0, 150);
    println!("{}", valid_age(25));    // true
    println!("{}", valid_age(-5));    // false
}
```

---

## 4. Function Composition

Combine two functions into one:

```rust
fn compose<A, B, C, F, G>(f: F, g: G) -> impl Fn(A) -> C
where
    F: Fn(A) -> B,
    G: Fn(B) -> C,
{
    move |x| g(f(x))
}

fn main() {
    let double = |x: i32| x * 2;
    let add_one = |x: i32| x + 1;
    let to_string = |x: i32| format!("Result: {x}");

    // double then add_one
    let double_then_add = compose(double, add_one);
    println!("{}", double_then_add(5));  // 11

    // chain three: double → add_one → to_string
    let pipeline = compose(compose(double, add_one), to_string);
    println!("{}", pipeline(5));  // Result: 11
}
```

---

## 5. Iterator Methods as HOFs

The iterator API is a masterclass in higher-order functions:

```rust
fn main() {
    let words = vec!["hello", "world", "foo", "bar", "rust"];

    // Chain of HOFs — each takes a closure
    let result: String = words.iter()
        .filter(|w| w.len() > 3)          // HOF: takes predicate
        .map(|w| w.to_uppercase())         // HOF: takes transform
        .enumerate()                        // HOF: adds index
        .map(|(i, w)| format!("{}. {w}", i + 1))  // HOF: formats
        .collect::<Vec<_>>()
        .join(", ");

    println!("{result}");  // 1. HELLO, 2. WORLD, 3. RUST

    // Functional sum of squares of even numbers
    let sum_of_even_squares: i32 = (1..=10)
        .filter(|x| x % 2 == 0)
        .map(|x| x * x)
        .sum();

    println!("Sum: {sum_of_even_squares}");  // 220
}
```

---

## 6. Currying and Partial Application

Rust doesn't have automatic currying, but you can simulate it:

```rust
// Partial application via closure
fn multiply(a: i32) -> impl Fn(i32) -> i32 {
    move |b| a * b
}

// "Curried" add
fn add(a: i32) -> impl Fn(i32) -> i32 {
    move |b| a + b
}

fn main() {
    let double = multiply(2);
    let triple = multiply(3);

    println!("{}", double(5));   // 10
    println!("{}", triple(5));   // 15

    // Use in iterator chains
    let numbers = vec![1, 2, 3, 4, 5];
    let doubled: Vec<i32> = numbers.iter().map(|&x| double(x)).collect();
    let offset: Vec<i32> = numbers.iter().map(|&x| add(100)(x)).collect();

    println!("{:?}", doubled);  // [2, 4, 6, 8, 10]
    println!("{:?}", offset);   // [101, 102, 103, 104, 105]
}
```

---

## 7. Functional Pipelines

Build complex transformations from simple, reusable parts:

```rust
fn main() {
    // Data processing pipeline
    let raw_data = vec![
        " Alice, 85 ",
        " Bob, 92 ",
        " Charlie, 78 ",
        " Diana, 95 ",
        " Eve, 65 ",
        " bad data ",
    ];

    let honor_roll: Vec<String> = raw_data.iter()
        .map(|s| s.trim())                              // clean whitespace
        .filter_map(|s| {                               // parse or skip
            let parts: Vec<&str> = s.split(',').collect();
            if parts.len() == 2 {
                let name = parts[0].trim();
                let score: i32 = parts[1].trim().parse().ok()?;
                Some((name.to_string(), score))
            } else {
                None
            }
        })
        .filter(|(_, score)| *score >= 80)              // grade filter
        .map(|(name, score)| format!("{name}: {score}")) // format
        .collect();

    println!("Honor Roll:");
    honor_roll.iter().for_each(|s| println!("  ⭐ {s}"));
}
```

---

## 8. Imperative vs Functional Style

```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

    // Imperative style
    let mut sum_imperative = 0;
    for &n in &numbers {
        if n % 2 == 0 {
            sum_imperative += n * n;
        }
    }

    // Functional style
    let sum_functional: i32 = numbers.iter()
        .filter(|&&n| n % 2 == 0)
        .map(|&n| n * n)
        .sum();

    assert_eq!(sum_imperative, sum_functional);  // both 220
    println!("Sum of even squares: {sum_functional}");

    // Both are valid! Functional is often more readable for data pipelines.
    // Imperative may be clearer for complex control flow.
}
```

---

## 9. Real-World Example: Pipeline Combinator

The roadmap practice task:

```rust
struct Pipeline<T> {
    value: T,
}

impl<T> Pipeline<T> {
    fn new(value: T) -> Self {
        Pipeline { value }
    }

    fn pipe<U, F>(self, f: F) -> Pipeline<U>
    where
        F: FnOnce(T) -> U,
    {
        Pipeline { value: f(self.value) }
    }

    fn tap<F>(self, f: F) -> Self
    where
        F: FnOnce(&T),
    {
        f(&self.value);
        self
    }

    fn finish(self) -> T {
        self.value
    }
}

fn main() {
    let result = Pipeline::new(vec![3, 1, 4, 1, 5, 9, 2, 6])
        .tap(|v| println!("Input: {:?}", v))
        .pipe(|mut v| { v.sort(); v })
        .tap(|v| println!("Sorted: {:?}", v))
        .pipe(|v| v.into_iter().filter(|&x| x > 2).collect::<Vec<_>>())
        .tap(|v| println!("Filtered: {:?}", v))
        .pipe(|v| v.iter().sum::<i32>())
        .tap(|s| println!("Sum: {s}"))
        .finish();

    println!("\nFinal result: {result}");
}
```

### Function registry pattern:

```rust
use std::collections::HashMap;

type MathFn = Box<dyn Fn(f64, f64) -> f64>;

fn build_calculator() -> HashMap<String, MathFn> {
    let mut ops: HashMap<String, MathFn> = HashMap::new();
    ops.insert("add".into(), Box::new(|a, b| a + b));
    ops.insert("sub".into(), Box::new(|a, b| a - b));
    ops.insert("mul".into(), Box::new(|a, b| a * b));
    ops.insert("div".into(), Box::new(|a, b| if b != 0.0 { a / b } else { f64::NAN }));
    ops.insert("pow".into(), Box::new(|a, b| a.powf(b)));
    ops
}

fn main() {
    let calc = build_calculator();

    let ops = vec![("add", 10.0, 5.0), ("mul", 3.0, 7.0), ("pow", 2.0, 10.0)];
    for (op, a, b) in ops {
        if let Some(f) = calc.get(op) {
            println!("{op}({a}, {b}) = {}", f(a, b));
        }
    }
}
```

---

## 10. Summary Cheat Sheet

```
HIGHER-ORDER FUNCTIONS
────────────────────────────────────────────────────────────
Takes function    fn apply(f: impl Fn(T) -> U, x: T) -> U
Returns function  fn make(n: i32) -> impl Fn(i32) -> i32

COMPOSITION
────────────────────────────────────────────────────────────
fn compose(f, g) → |x| g(f(x))
Chain: compose(compose(f, g), h)

PARTIAL APPLICATION
────────────────────────────────────────────────────────────
fn mul(a: i32) -> impl Fn(i32) -> i32 { move |b| a * b }
let double = mul(2);

ITERATOR HOFs
────────────────────────────────────────────────────────────
.map(f)        transform each element
.filter(f)     keep matching elements
.fold(init, f) accumulate to single value
.for_each(f)   execute side effect
.find(f)       first match

PIPELINE PATTERN
────────────────────────────────────────────────────────────
Pipeline::new(data)
    .pipe(transform1)
    .pipe(transform2)
    .finish()

IMPERATIVE vs FUNCTIONAL
────────────────────────────────────────────────────────────
Both valid! Use functional for data transforms.
Use imperative for complex control flow.
```

---

## 🎉 You've Completed 50 Lessons!

Congratulations! You've built a strong intermediate foundation in Rust:

- 🧠 **Ownership**: Lifetimes, borrows, advanced patterns
- 📚 **Collections**: Iterators, adaptors, collecting
- ⚠ **Error Handling**: Custom errors, anyhow, thiserror
- 🔷 **Traits**: Bounds, generics, associated types, operator overloading
- 📦 **Modules**: Workspaces, publishing
- 🔒 **Closures**: Fn/FnMut/FnOnce, higher-order functions

**Coming next**: Smart Pointers, Concurrency, and Async Rust! 🦀

## Further Reading
- [Rust by Example — Higher Order Functions](https://doc.rust-lang.org/rust-by-example/fn/hof.html)
- [Functional Programming in Rust](https://doc.rust-lang.org/book/ch13-00-functional-features.html)

---

*Higher-order functions: compose small pieces into powerful pipelines! 🦀*
