# 🧪 Lesson 68 — Questions: State & Strategy (DP2)

> **Lesson:** [lesson_68_state_strategy.md](./lesson_68_state_strategy.md)  
> **Answers:** [lesson_68_answers.md](./lesson_68_answers.md)

---

## Section A — Conceptual

### Q1
When would you use an enum-based state machine versus a type-state (PhantomData) approach?

### Q2
What is the Strategy pattern? How does it differ from the State pattern?

---

## Section B — Write It Yourself

### Q3 — Traffic light (Roadmap Practice Task)
Implement a `TrafficLight` enum with `Red`, `Yellow`, `Green`. Add `next()` for transitions and `wait_time()` for duration. Cycle through 9 transitions.

### Q4 — Pluggable formatter
Create a `Formatter` trait with `fn format(&self, text: &str) -> String`. Implement `UpperCase`, `LowerCase`, and `TitleCase` strategies. Build a `Printer` that uses the strategy.

### Q5 — Closure strategy
Build a `Pipeline` struct that stores a `Vec<Box<dyn Fn(String) -> String>>`. Implement `add_step()` and `execute()` that runs all steps in sequence.

---

## Section C — True or False?

### Q6
1. Enum-based state machines are checked at compile time for exhaustiveness.
2. Type-state pattern allows invalid state transitions at runtime.
3. Strategy pattern lets you swap algorithms at runtime.
4. Closures can be used as lightweight strategies.
5. Trait objects (`Box<dyn Trait>`) enable dynamic dispatch for strategies.

---

*Patterns: proven solutions for recurring problems! 🦀*
