# 🧪 Lesson 40 — Questions: Generics in Functions (T3)

> **Lesson:** [lesson_40_generics.md](./lesson_40_generics.md)  
> **Answers:** [lesson_40_answers.md](./lesson_40_answers.md)

---

## Section A — Predict the Output

### Q1
```rust
fn identity<T>(x: T) -> T { x }

fn main() {
    println!("{}", identity(42));
    println!("{}", identity("hello"));
}
```

### Q2
```rust
fn first<T>(a: T, _b: T) -> T { a }

fn main() {
    println!("{}", first(1, 2));
    // println!("{}", first(1, "two"));  // Does this compile?
}
```

### Q3
```rust
fn make_vec<T>(item: T, count: usize) -> Vec<T>
where T: Clone
{
    vec![item; count]
}

fn main() {
    let v = make_vec("hi", 3);
    println!("{:?}", v);
}
```

---

## Section B — Write It Yourself

### Q4 — Generic Stack (Roadmap Practice Task)
Build a `Stack<T>` with:
- `push(&mut self, item: T)`
- `pop(&mut self) -> Option<T>`
- `peek(&self) -> Option<&T>`
- `is_empty(&self) -> bool`

Test with both `i32` and `String` values.

### Q5 — Generic min
Write `fn min_of<T: PartialOrd>(a: T, b: T) -> T` that returns the smaller value.

### Q6 — Generic contains
Write `fn contains<T: PartialEq>(haystack: &[T], needle: &T) -> bool` without using the built-in `.contains()` method.

### Q7 — Generic transform
Write `fn transform<T, U, F>(items: Vec<T>, f: F) -> Vec<U> where F: Fn(T) -> U` that applies `f` to each item.

---

## Section C — Deep Understanding

### Q8 — True or False?
1. Generics have runtime overhead in Rust.
2. Monomorphization creates a separate function for each type used.
3. You can call generic functions without specifying the type if the compiler can infer it.
4. `fn f<T>(x: T)` and `fn f(x: impl ?Sized)` are equivalent.
5. The turbofish `::<>` is required on every generic function call.

### Q9
Explain why this doesn't compile and provide two different fixes:
```rust
fn largest<T>(list: &[T]) -> T {
    let mut max = list[0];
    for &item in &list[1..] {
        if item > max { max = item; }
    }
    max
}
```

### Q10 — Zero-cost abstraction
In your own words, explain what "zero-cost abstraction" means for Rust generics. How does it compare to Java/C# generics?

---

*Generic functions: one definition, infinite possibilities! 🦀*
