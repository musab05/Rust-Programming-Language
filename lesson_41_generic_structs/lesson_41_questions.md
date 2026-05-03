# 🧪 Lesson 41 — Questions: Generics in Structs & Enums (T4)

> **Lesson:** [lesson_41_generic_structs.md](./lesson_41_generic_structs.md)  
> **Answers:** [lesson_41_answers.md](./lesson_41_answers.md)

---

## Section A — Predict: Compile or Not?

### Q1
```rust
struct Point<T> { x: T, y: T }
fn main() {
    let p = Point { x: 5, y: 10.0 };
}
```

### Q2
```rust
struct Point<T, U> { x: T, y: U }
fn main() {
    let p = Point { x: 5, y: 10.0 };
    println!("{}, {}", p.x, p.y);
}
```

### Q3
```rust
struct Wrapper<T> { value: T }
impl Wrapper<i32> {
    fn is_positive(&self) -> bool { self.value > 0 }
}
fn main() {
    let w = Wrapper { value: -5 };
    println!("{}", w.is_positive());
    let w2 = Wrapper { value: "hello" };
    // println!("{}", w2.is_positive());
}
```

### Q4
```rust
#[derive(Debug)]
enum Maybe<T> { Just(T), Nothing }
fn main() {
    let a: Maybe<i32> = Maybe::Just(42);
    let b: Maybe<String> = Maybe::Nothing;
    println!("{:?} {:?}", a, b);
}
```

---

## Section B — Write It Yourself

### Q5 — Pair with cmp_display (Roadmap Practice Task)
Create `Pair<T>` with `first` and `second` fields. Implement:
1. `new(first: T, second: T)` for all T
2. `cmp_display(&self)` only when `T: Display + PartialOrd` — prints which is larger

### Q6 — Generic enum
Create `enum Response<T, E>` with `Success(T)`, `Failure(E)`, and `Pending`. Implement a `describe` method that works when both T and E implement `Display`.

### Q7 — Mixup method
Create `Point<T, U>` and implement a `mixup` method that takes another `Point<V, W>` and returns `Point<T, W>`. Demonstrate with concrete types.

---

## Section C — Deep Understanding

### Q8 — True or False?
1. A struct with `struct Foo<T> { x: T, y: T }` requires both fields to be the same type.
2. `impl Foo<f64>` adds methods only for `Foo<f64>`, not `Foo<i32>`.
3. A method can introduce type parameters that the struct doesn't have.
4. `Option<T>` and `Result<T, E>` are examples of generic enums.
5. Default type parameters make the type parameter optional when using the struct.

### Q9
Explain why `impl<T> Point<T>` needs the `<T>` after `impl`. What would happen without it?

---

*Generics in data structures: flexibility meets type safety! 🦀*
