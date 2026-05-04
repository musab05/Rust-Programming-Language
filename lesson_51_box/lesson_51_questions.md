# 🧪 Lesson 51 — Questions: Box (SP1)

> **Lesson:** [lesson_51_box.md](./lesson_51_box.md)  
> **Answers:** [lesson_51_answers.md](./lesson_51_answers.md)

---

## Section A — Predict: Compile or Not?

### Q1
```rust
enum List { Cons(i32, List), Nil }
fn main() { let _ = List::Nil; }
```

### Q2
```rust
enum List { Cons(i32, Box<List>), Nil }
fn main() {
    let l = List::Cons(1, Box::new(List::Cons(2, Box::new(List::Nil))));
}
```

### Q3
```rust
fn main() {
    let b = Box::new(42);
    println!("{}", b + 1);
}
```

---

## Section B — Write It Yourself

### Q4 — Binary tree (Roadmap Practice Task)
Create a recursive `Tree<T>` enum with `Empty`, `Leaf(T)`, and `Node { value, left, right }`. Implement:
1. `insert` for a binary search tree
2. `contains` to search for a value
3. `count` to count total nodes

### Q5 — Linked list
Build a singly-linked list using `Box`:
```rust
enum List<T> { Cons(T, Box<List<T>>), Nil }
```
Implement `push_front`, `to_vec`, and `Display`.

### Q6 — Trait object collection
Create a `Vec<Box<dyn Display>>` containing an `i32`, a `String`, and an `f64`. Print all elements.

---

## Section C — Deep Understanding

### Q7 — True or False?
1. `Box<T>` allocates T on the heap.
2. `Box<T>` can have multiple owners.
3. `Box<T>` implements `Deref`, so it auto-dereferences to `&T`.
4. Recursive types require `Box` (or another indirection) because the compiler needs a known size.
5. `drop(box_val)` frees the heap memory immediately.
6. `Box<dyn Trait>` is a fat pointer (data ptr + vtable ptr).

### Q8
Explain why `enum List { Cons(i32, List), Nil }` won't compile but `enum List { Cons(i32, Box<List>), Nil }` does. Draw the memory layout difference.

---

*Box: the simplest smart pointer, yet indispensable for recursive data! 🦀*
