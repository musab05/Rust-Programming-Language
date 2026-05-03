# 🧪 Lesson 42 — Questions: impl Trait & dyn Trait (T5)

> **Lesson:** [lesson_42_impl_dyn_trait.md](./lesson_42_impl_dyn_trait.md)  
> **Answers:** [lesson_42_answers.md](./lesson_42_answers.md)

---

## Section A — Predict: Compile or Not?

### Q1
```rust
trait Speak { fn speak(&self) -> String; }
struct Dog;
impl Speak for Dog { fn speak(&self) -> String { "Woof".into() } }
struct Cat;
impl Speak for Cat { fn speak(&self) -> String { "Meow".into() } }

fn make(cat: bool) -> impl Speak {
    if cat { Cat } else { Dog }
}
```

### Q2
```rust
trait Speak { fn speak(&self) -> String; }
struct Dog;
impl Speak for Dog { fn speak(&self) -> String { "Woof".into() } }
struct Cat;
impl Speak for Cat { fn speak(&self) -> String { "Meow".into() } }

fn make(cat: bool) -> Box<dyn Speak> {
    if cat { Box::new(Cat) } else { Box::new(Dog) }
}
```

### Q3
```rust
trait Factory { fn create() -> Self; }
fn use_it(f: &dyn Factory) { }
```

---

## Section B — Write It Yourself

### Q4 — Polymorphic collection (Roadmap Practice Task)
Create a `Summary` trait. Implement it for `Article`, `Tweet`, and `Video`. Create a `Vec<Box<dyn Summary>>` containing all three types, and iterate to print each summary.

### Q5 — Factory function
Write `fn create_shape(kind: &str) -> Box<dyn Shape>` that returns circles or rectangles based on the input string.

### Q6 — Plugin system
Design a `Plugin` trait with `name(&self) -> &str` and `execute(&self)`. Create two plugins and store them in a `Vec<Box<dyn Plugin>>`. Execute all plugins.

---

## Section C — Deep Understanding

### Q7 — True or False?
1. `dyn Trait` uses a vtable for method dispatch.
2. `impl Trait` in return position can return different concrete types.
3. A trait with a method returning `Self` is object-safe.
4. `Box<dyn Trait>` is a fat pointer containing data + vtable pointers.
5. Static dispatch is always preferable to dynamic dispatch.
6. `&dyn Trait` requires heap allocation.

### Q8
When would you choose `enum` dispatch over `dyn Trait`? Give a concrete example.

---

*Polymorphism in Rust: compile-time speed or runtime flexibility — your choice! 🦀*
