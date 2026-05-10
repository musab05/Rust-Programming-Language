# 🧪 Lesson 87 — Questions: PhantomData (AT5)

> **Lesson:** [lesson_87_phantomdata.md](./lesson_87_phantomdata.md)  
> **Answers:** [lesson_87_answers.md](./lesson_87_answers.md)

---

## Section A — Compile or Not?

### Q1
```rust
struct Wrapper<T> { value: u64 }
fn main() { let w: Wrapper<String> = Wrapper { value: 42 }; }
```

---

## Section B — Write It Yourself

### Q2 — Type-safe units (Roadmap Practice Task)
Create `Quantity<Unit>` with PhantomData. Define `Meters`, `Kilograms`, `Seconds` unit markers. Implement `Add` for same-unit quantities. Show that adding different units fails to compile.

### Q3 — Type-state
Build a `Connection<State>` with states `Disconnected`, `Connected`, `Authenticated`. Ensure you can only `query()` when Authenticated.

---

## Section C — True or False?

### Q4
1. `PhantomData<T>` occupies zero bytes at runtime.
2. Unused type parameters cause a compile error.
3. `PhantomData<T>` makes the compiler act as if the struct owns a `T`.
4. PhantomData can be used to bind a lifetime to a struct.
5. Type-state pattern relies on PhantomData for compile-time state checks.
6. `PhantomData<T>` requires `T: Sized`.

---

*PhantomData: zero-cost, maximum type safety! 🦀*
