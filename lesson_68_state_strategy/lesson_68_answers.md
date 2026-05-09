# ✅ Lesson 68 — Answers: State & Strategy (DP2)

---

## Section A

### A1
- **Enum state** — when states are known at compile time and you want exhaustive `match` checking. Easier to implement, more idiomatic Rust. Transitions checked at runtime.
- **Type-state** — when you need the compiler to prevent invalid transitions entirely. More boilerplate but zero runtime cost. Use for safety-critical state machines (connections, auth, workflows).

### A2
- **State** — object changes behavior based on its current state. The state itself determines what happens. Focus: "what state am I in?"
- **Strategy** — object delegates behavior to a pluggable algorithm. The caller chooses which algorithm. Focus: "how should I do this?"

---

## Section B

### A3
```rust
#[derive(Debug)]
enum TrafficLight { Red, Yellow, Green }

impl TrafficLight {
    fn next(self) -> Self {
        match self {
            Self::Red => Self::Green,
            Self::Green => Self::Yellow,
            Self::Yellow => Self::Red,
        }
    }
    fn wait_time(&self) -> u32 {
        match self { Self::Red => 30, Self::Green => 25, Self::Yellow => 5 }
    }
}

fn main() {
    let mut light = TrafficLight::Red;
    for _ in 0..9 {
        println!("{:?} — wait {}s", light, light.wait_time());
        light = light.next();
    }
}
```

### A4
```rust
trait Formatter {
    fn format(&self, text: &str) -> String;
}

struct UpperCase;
impl Formatter for UpperCase {
    fn format(&self, text: &str) -> String { text.to_uppercase() }
}

struct LowerCase;
impl Formatter for LowerCase {
    fn format(&self, text: &str) -> String { text.to_lowercase() }
}

struct TitleCase;
impl Formatter for TitleCase {
    fn format(&self, text: &str) -> String {
        text.split_whitespace()
            .map(|w| { let mut c = w.chars(); match c.next() {
                Some(f) => f.to_uppercase().to_string() + &c.as_str().to_lowercase(),
                None => String::new(),
            }}).collect::<Vec<_>>().join(" ")
    }
}

struct Printer { formatter: Box<dyn Formatter> }
impl Printer {
    fn new(f: Box<dyn Formatter>) -> Self { Printer { formatter: f } }
    fn print(&self, text: &str) { println!("{}", self.formatter.format(text)); }
}

fn main() {
    Printer::new(Box::new(UpperCase)).print("hello world");
    Printer::new(Box::new(TitleCase)).print("hello world");
}
```

### A5
```rust
struct Pipeline {
    steps: Vec<Box<dyn Fn(String) -> String>>,
}

impl Pipeline {
    fn new() -> Self { Pipeline { steps: vec![] } }
    fn add_step<F: Fn(String) -> String + 'static>(&mut self, f: F) { self.steps.push(Box::new(f)); }
    fn execute(&self, input: &str) -> String {
        let mut result = input.to_string();
        for step in &self.steps { result = step(result); }
        result
    }
}

fn main() {
    let mut p = Pipeline::new();
    p.add_step(|s| s.trim().to_string());
    p.add_step(|s| s.to_uppercase());
    p.add_step(|s| format!("[{s}]"));
    println!("{}", p.execute("  hello world  "));  // [HELLO WORLD]
}
```

---

## Section C

### A6
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | Rust's `match` on enums enforces exhaustive handling |
| 2 | **False** | Type-state prevents invalid transitions at **compile time** |
| 3 | **True** | That's the core idea — plug in different algorithms |
| 4 | **True** | Closures are a lightweight alternative to trait implementations |
| 5 | **True** | `Box<dyn Trait>` enables runtime polymorphism |

---

## 🏆 Lesson 68 Complete!

**Next up:** [Lesson 69 — Testing: Unit & Integration](../lesson_69_testing/lesson_69_testing.md) 🦀
