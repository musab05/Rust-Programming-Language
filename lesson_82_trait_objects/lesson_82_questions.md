# 🧪 Lesson 82 — Questions: Trait Objects & Object Safety (T9)

> **Lesson:** [lesson_82_trait_objects.md](./lesson_82_trait_objects.md)  
> **Answers:** [lesson_82_answers.md](./lesson_82_answers.md)

---

## Section A — Object Safe or Not?

### Q1
Which traits are object-safe?
```rust
trait A { fn name(&self) -> &str; }
trait B { fn clone(&self) -> Self; }
trait C { fn process<T>(&self, item: T); }
trait D { fn draw(&self); fn resize(&mut self, factor: f64); }
```

---

## Section B — Write It Yourself

### Q2 — Plugin system (Roadmap Practice Task)
Design a `Plugin` trait with `name()`, `version()`, and `execute()`. Create 3 different plugins. Build a `PluginManager` that stores `Vec<Box<dyn Plugin>>` and runs all plugins.

### Q3 — Heterogeneous collection
Create a `Shape` trait. Implement it for `Circle`, `Square`, `Triangle`. Store all three in a `Vec<Box<dyn Shape>>` and calculate total area.

---

## Section C — True or False?

### Q4
1. `dyn Trait` is a fat pointer containing a data pointer and vtable pointer.
2. A trait with `fn clone(&self) -> Self` is object-safe.
3. `Box<dyn Trait>` owns the trait object on the heap.
4. Static dispatch (`impl Trait`) is faster than dynamic dispatch (`dyn Trait`).
5. Object safety is checked at compile time.
6. You can have `Vec<Box<dyn TraitA + TraitB>>` with multiple unrelated traits.

---

*Trait objects: polymorphism when you need it! 🦀*
