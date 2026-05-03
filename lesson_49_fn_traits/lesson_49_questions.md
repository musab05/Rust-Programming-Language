# 🧪 Lesson 49 — Questions: Fn, FnMut, FnOnce (CL2)

> **Lesson:** [lesson_49_fn_traits.md](./lesson_49_fn_traits.md)  
> **Answers:** [lesson_49_answers.md](./lesson_49_answers.md)

---

## Section A — Which Trait?

### Q1
For each closure, identify which trait(s) it implements:
```rust
let a = || println!("hello");
let mut count = 0;
let b = || { count += 1; };
let name = String::from("Alice");
let c = || drop(name);
```

### Q2 — Compile or not?
```rust
fn call_twice(f: impl FnOnce()) {
    f();
    f();
}
fn main() { call_twice(|| println!("hi")); }
```

### Q3 — Compile or not?
```rust
fn call_twice(f: impl Fn()) {
    f();
    f();
}
fn main() { call_twice(|| println!("hi")); }
```

---

## Section B — Write It Yourself

### Q4 — Accept each trait (Roadmap Practice Task)
Write three functions:
1. `run_once(f: impl FnOnce() -> String) -> String`
2. `run_mut(f: impl FnMut(i32) -> i32, values: &[i32]) -> Vec<i32>`
3. `run_pure(f: impl Fn(i32) -> i32, x: i32) -> i32` that applies f twice

Demonstrate each with an appropriate closure.

### Q5 — Returning closures
Write `make_multiplier(factor: i32) -> impl Fn(i32) -> i32` that returns a closure.

### Q6 — Event handler
Build a simple event system using `Vec<Box<dyn Fn(&str)>>`. Add handlers and trigger them.

---

## Section C — Deep Understanding

### Q7 — True or False?
1. `FnOnce` is the most restrictive closure trait.
2. A closure that implements `Fn` also implements `FnMut` and `FnOnce`.
3. `mut f: impl FnMut()` requires the `mut` because `FnMut::call_mut` takes `&mut self`.
4. You can return a closure using `-> Fn(i32) -> i32` syntax.
5. `Box<dyn Fn(T)>` allows storing closures in collections.

### Q8
Why should you prefer `FnOnce` as a parameter trait when the closure is only called once?

---

*Three traits, one rule: use the most permissive trait that works! 🦀*
