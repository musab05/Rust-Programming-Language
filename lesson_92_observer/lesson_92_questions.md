# 🧪 Lesson 92 — Questions: Observer / Event System (DP4)

> **Lesson:** [lesson_92_observer.md](./lesson_92_observer.md)  
> **Answers:** [lesson_92_answers.md](./lesson_92_answers.md)

---

## Section A — Conceptual

### Q1
What are the advantages of the Observer pattern over direct function calls?

---

## Section B — Write It Yourself

### Q2 — Typed event bus (Roadmap Practice Task)
Build an event bus that supports typed events using `TypeId` and `Any`. Subscribe to `LoginEvent` and `ErrorEvent` separately.

### Q3 — Closure-based emitter
Create an `EventEmitter` using `HashMap<String, Vec<Box<dyn Fn(&str)>>>`. Demonstrate `.on()` and `.emit()`.

---

## Section C — True or False?

### Q4
1. The Observer pattern decouples publishers from subscribers.
2. `broadcast::channel` delivers each message to exactly one subscriber.
3. `TypeId::of::<T>()` uniquely identifies a Rust type at runtime.
4. Closure-based observers are more flexible than trait-based.
5. The Observer pattern is unsuitable for async Rust.
6. `Any::downcast_ref` safely converts a trait object to a concrete type.

---

*Events: decouple, subscribe, react! 🦀*
