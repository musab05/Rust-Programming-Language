# 🧪 Lesson 45 — Questions: Associated Types (T8)

> **Lesson:** [lesson_45_associated_types.md](./lesson_45_associated_types.md)  
> **Answers:** [lesson_45_answers.md](./lesson_45_answers.md)

---

## Section A — Conceptual

### Q1
What is the key difference between a trait with an associated type (`type Item`) and a trait with a generic parameter (`Trait<T>`)?

### Q2
Why does `Iterator` use `type Item` instead of `Iterator<T>`?

---

## Section B — Write It Yourself

### Q3 — Container trait
Define a `Container` trait with an associated `Item` type. Add methods `add(&mut self, item: Self::Item)` and `get(&self, idx: usize) -> Option<&Self::Item>`. Implement for a `NumberList` struct.

### Q4 — Graph trait (Roadmap Practice Task)
Define a `Graph` trait with associated `Node` and `Weight` types. Implement for a simple `SocialNetwork` where nodes are usernames (String) and weights are friendship strength (u32).

### Q5 — Fully qualified syntax
Create a struct that implements two traits, both having a `name()` method. Demonstrate calling each version.

---

## Section C — True or False?

### Q6
1. A type can implement a trait with associated types only once.
2. Associated types can have trait bounds (e.g., `type Item: Display`).
3. `Self::Item` refers to the associated type chosen by the implementor.
4. `<Type as Trait>::method()` is the fully qualified syntax.
5. `Add::Output` is an example of an associated type.

---

*Associated types: the trait says what, the implementor says which type! 🦀*
