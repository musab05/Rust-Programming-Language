# 📘 Lesson 68 — Design Patterns: State & Strategy (DP2)

> **Series:** Rust From Zero · Advanced Level  
> **Roadmap ID:** DP2 · Category: 🏗 Design Patterns  
> **Previous:** [Lesson 67 — Builder Pattern](../lesson_67_builder_pattern/lesson_67_builder_pattern.md)  
> **Next:** [Lesson 69 — Testing: Unit & Integration](../lesson_69_testing/lesson_69_testing.md)  
> **Practice:** [Questions](./lesson_68_questions.md) · [Answers](./lesson_68_answers.md)  
> **Practice Task:** Model a traffic light state machine and a pluggable compression strategy

---

## Table of Contents

1. [State Pattern with Enums](#1-state-pattern-with-enums)
2. [State Pattern with Trait Objects](#2-state-pattern-with-trait-objects)
3. [State Pattern with Types (Type-State)](#3-state-pattern-with-types-type-state)
4. [Strategy Pattern](#4-strategy-pattern)
5. [Strategy with Closures](#5-strategy-with-closures)
6. [Comparing Approaches](#6-comparing-approaches)
7. [Real-World Example: Payment Processor](#7-real-world-example-payment-processor)
8. [Real-World Example: Sorting Strategy](#8-real-world-example-sorting-strategy)
9. [Summary Cheat Sheet](#9-summary-cheat-sheet)

---

## 1. State Pattern with Enums

The most idiomatic Rust approach — use enums for states:

```rust
#[derive(Debug)]
enum TrafficLight {
    Red,
    Yellow,
    Green,
}

impl TrafficLight {
    fn next(self) -> Self {
        match self {
            TrafficLight::Red => TrafficLight::Green,
            TrafficLight::Green => TrafficLight::Yellow,
            TrafficLight::Yellow => TrafficLight::Red,
        }
    }

    fn duration_secs(&self) -> u32 {
        match self {
            TrafficLight::Red => 30,
            TrafficLight::Green => 25,
            TrafficLight::Yellow => 5,
        }
    }

    fn action(&self) -> &str {
        match self {
            TrafficLight::Red => "STOP",
            TrafficLight::Green => "GO",
            TrafficLight::Yellow => "CAUTION",
        }
    }
}

fn main() {
    let mut light = TrafficLight::Red;
    for _ in 0..6 {
        println!("{:?}: {} ({}s)", light, light.action(), light.duration_secs());
        light = light.next();
    }
}
```

### Enum state with data:

```rust
#[derive(Debug)]
enum Order {
    Pending { items: Vec<String> },
    Processing { items: Vec<String>, worker: String },
    Shipped { tracking: String },
    Delivered { signature: String },
    Cancelled { reason: String },
}

impl Order {
    fn process(self, worker: &str) -> Self {
        match self {
            Order::Pending { items } => Order::Processing {
                items, worker: worker.into(),
            },
            other => { println!("Can't process from {:?}", other); other }
        }
    }

    fn ship(self, tracking: &str) -> Self {
        match self {
            Order::Processing { .. } => Order::Shipped { tracking: tracking.into() },
            other => { println!("Can't ship from {:?}", other); other }
        }
    }

    fn deliver(self, signature: &str) -> Self {
        match self {
            Order::Shipped { .. } => Order::Delivered { signature: signature.into() },
            other => { println!("Can't deliver from {:?}", other); other }
        }
    }

    fn cancel(self, reason: &str) -> Self {
        match self {
            Order::Delivered { .. } => { println!("Can't cancel delivered"); self }
            _ => Order::Cancelled { reason: reason.into() },
        }
    }
}

fn main() {
    let order = Order::Pending { items: vec!["Book".into(), "Pen".into()] };
    let order = order.process("Alice");
    let order = order.ship("TRACK-123");
    let order = order.deliver("John Doe");
    println!("Final: {:?}", order);
}
```

---

## 2. State Pattern with Trait Objects

When states need dynamic dispatch:

```rust
trait State {
    fn name(&self) -> &str;
    fn enter(&self) { println!("→ Entering {}", self.name()); }
    fn handle(&self, event: &str) -> Box<dyn State>;
}

struct Idle;
struct Active;
struct Paused;

impl State for Idle {
    fn name(&self) -> &str { "Idle" }
    fn handle(&self, event: &str) -> Box<dyn State> {
        match event {
            "start" => { println!("Starting!"); Box::new(Active) }
            _ => { println!("Idle: ignoring '{event}'"); Box::new(Idle) }
        }
    }
}

impl State for Active {
    fn name(&self) -> &str { "Active" }
    fn handle(&self, event: &str) -> Box<dyn State> {
        match event {
            "pause" => { println!("Pausing..."); Box::new(Paused) }
            "stop" => { println!("Stopping!"); Box::new(Idle) }
            _ => { println!("Active: processing '{event}'"); Box::new(Active) }
        }
    }
}

impl State for Paused {
    fn name(&self) -> &str { "Paused" }
    fn handle(&self, event: &str) -> Box<dyn State> {
        match event {
            "resume" => { println!("Resuming!"); Box::new(Active) }
            "stop" => { println!("Stopping from pause"); Box::new(Idle) }
            _ => { println!("Paused: ignoring '{event}'"); Box::new(Paused) }
        }
    }
}

struct Machine {
    state: Box<dyn State>,
}

impl Machine {
    fn new() -> Self { let s = Box::new(Idle); s.enter(); Machine { state: s } }

    fn send(&mut self, event: &str) {
        let new_state = self.state.handle(event);
        if new_state.name() != self.state.name() {
            new_state.enter();
        }
        self.state = new_state;
    }
}

fn main() {
    let mut m = Machine::new();
    m.send("start");
    m.send("data");
    m.send("pause");
    m.send("resume");
    m.send("stop");
}
```

---

## 3. State Pattern with Types (Type-State)

Compile-time state enforcement (from Lesson 66):

```rust
use std::marker::PhantomData;

struct Locked;
struct Unlocked;

struct Door<S> { _state: PhantomData<S> }

impl Door<Locked> {
    fn new() -> Self { println!("🚪 Door created (locked)"); Door { _state: PhantomData } }
    fn unlock(self) -> Door<Unlocked> { println!("🔓 Unlocked"); Door { _state: PhantomData } }
}

impl Door<Unlocked> {
    fn open(&self) { println!("🚪 Opening door"); }
    fn lock(self) -> Door<Locked> { println!("🔒 Locked"); Door { _state: PhantomData } }
}

fn main() {
    let door = Door::<Locked>::new();
    // door.open();        // ❌ can't open locked door
    let door = door.unlock();
    door.open();           // ✅
    let _door = door.lock();
}
```

---

## 4. Strategy Pattern

Swap algorithms at runtime using trait objects:

```rust
trait Compressor {
    fn compress(&self, data: &[u8]) -> Vec<u8>;
    fn name(&self) -> &str;
}

struct NoCompression;
impl Compressor for NoCompression {
    fn compress(&self, data: &[u8]) -> Vec<u8> { data.to_vec() }
    fn name(&self) -> &str { "none" }
}

struct RunLength;
impl Compressor for RunLength {
    fn compress(&self, data: &[u8]) -> Vec<u8> {
        let mut result = vec![];
        let mut i = 0;
        while i < data.len() {
            let byte = data[i];
            let mut count: u8 = 1;
            while i + count as usize < data.len() && data[i + count as usize] == byte && count < 255 {
                count += 1;
            }
            result.push(count);
            result.push(byte);
            i += count as usize;
        }
        result
    }
    fn name(&self) -> &str { "RLE" }
}

struct FileWriter {
    compressor: Box<dyn Compressor>,
}

impl FileWriter {
    fn new(compressor: Box<dyn Compressor>) -> Self {
        FileWriter { compressor }
    }

    fn write(&self, data: &[u8]) {
        let compressed = self.compressor.compress(data);
        println!("[{}] {} bytes → {} bytes",
            self.compressor.name(), data.len(), compressed.len());
    }

    fn set_compressor(&mut self, compressor: Box<dyn Compressor>) {
        self.compressor = compressor;
    }
}

fn main() {
    let mut writer = FileWriter::new(Box::new(NoCompression));
    writer.write(b"AAAAAABBBCC");

    writer.set_compressor(Box::new(RunLength));
    writer.write(b"AAAAAABBBCC");
}
```

---

## 5. Strategy with Closures

Lighter weight — use closures instead of trait objects:

```rust
struct Validator {
    rules: Vec<Box<dyn Fn(&str) -> Result<(), String>>>,
}

impl Validator {
    fn new() -> Self { Validator { rules: vec![] } }

    fn add_rule<F: Fn(&str) -> Result<(), String> + 'static>(&mut self, rule: F) {
        self.rules.push(Box::new(rule));
    }

    fn validate(&self, input: &str) -> Result<(), Vec<String>> {
        let errors: Vec<String> = self.rules.iter()
            .filter_map(|rule| rule(input).err())
            .collect();
        if errors.is_empty() { Ok(()) } else { Err(errors) }
    }
}

fn main() {
    let mut v = Validator::new();

    v.add_rule(|s| if s.len() >= 3 { Ok(()) } else { Err("Too short".into()) });
    v.add_rule(|s| if s.chars().any(|c| c.is_uppercase()) { Ok(()) } else { Err("Need uppercase".into()) });
    v.add_rule(|s| if s.chars().any(|c| c.is_numeric()) { Ok(()) } else { Err("Need digit".into()) });

    match v.validate("Hi1") { Ok(()) => println!("✅ Valid"), Err(e) => println!("❌ {e:?}") }
    match v.validate("ab") { Ok(()) => println!("✅ Valid"), Err(e) => println!("❌ {e:?}") }
}
```

---

## 6. Comparing Approaches

| Approach | When to Use |
|---|---|
| **Enum state** | Fixed set of states known at compile time |
| **Trait object state** | Open set of states, plugins |
| **Type-state** | Must prevent invalid transitions at compile time |
| **Trait strategy** | Swappable algorithms, open for extension |
| **Closure strategy** | Simple strategies, inline logic |

---

## 7. Real-World Example: Payment Processor

```rust
trait PaymentGateway {
    fn charge(&self, amount: f64) -> Result<String, String>;
    fn name(&self) -> &str;
}

struct Stripe;
impl PaymentGateway for Stripe {
    fn charge(&self, amount: f64) -> Result<String, String> {
        Ok(format!("stripe_txn_{:.0}", amount * 100.0))
    }
    fn name(&self) -> &str { "Stripe" }
}

struct PayPal;
impl PaymentGateway for PayPal {
    fn charge(&self, amount: f64) -> Result<String, String> {
        if amount > 10000.0 { Err("PayPal limit exceeded".into()) }
        else { Ok(format!("pp_{:.0}", amount * 100.0)) }
    }
    fn name(&self) -> &str { "PayPal" }
}

struct Checkout {
    gateway: Box<dyn PaymentGateway>,
}

impl Checkout {
    fn new(gateway: Box<dyn PaymentGateway>) -> Self { Checkout { gateway } }

    fn process(&self, amount: f64) {
        println!("Processing ${amount:.2} via {}...", self.gateway.name());
        match self.gateway.charge(amount) {
            Ok(id) => println!("  ✅ Success: {id}"),
            Err(e) => println!("  ❌ Failed: {e}"),
        }
    }
}

fn main() {
    let checkout = Checkout::new(Box::new(Stripe));
    checkout.process(49.99);

    let checkout = Checkout::new(Box::new(PayPal));
    checkout.process(29.99);
}
```

---

## 8. Real-World Example: Sorting Strategy

```rust
fn main() {
    let mut data = vec![5, 3, 8, 1, 9, 2, 7];

    // Strategy as closure
    let strategies: Vec<(&str, Box<dyn Fn(&mut Vec<i32>)>)> = vec![
        ("Ascending", Box::new(|v: &mut Vec<i32>| v.sort())),
        ("Descending", Box::new(|v: &mut Vec<i32>| v.sort_by(|a, b| b.cmp(a)))),
        ("By last digit", Box::new(|v: &mut Vec<i32>| v.sort_by_key(|x| x % 10))),
    ];

    for (name, sort_fn) in &strategies {
        let mut copy = data.clone();
        sort_fn(&mut copy);
        println!("{name:15} → {:?}", copy);
    }
}
```

---

## 9. Summary Cheat Sheet

```
STATE PATTERN
────────────────────────────────────────────────────────────
Enum state     → match on variants, fixed states
Trait state    → Box<dyn State>, open for extension
Type-state     → PhantomData<S>, compile-time safety

STRATEGY PATTERN
────────────────────────────────────────────────────────────
Trait strategy → Box<dyn Algorithm>, swappable at runtime
Closure strategy → Box<dyn Fn(...)>, lightweight

CHOOSING
────────────────────────────────────────────────────────────
Known states + data      → enum
Unknown/extensible states → trait objects
Must enforce transitions  → type-state (PhantomData)
Swappable behavior        → strategy (trait or closure)
```

---

## What's Next?

**Lesson 69 — Testing: Unit & Integration** — Write robust tests. `#[test]`, `#[cfg(test)]`, assert macros, integration tests, and test organization.

## Further Reading
- [Rust Design Patterns — State](https://rust-unofficial.github.io/patterns/patterns/behavioural/strategy.html)
- [The Rust Book — Ch 17.3: State Pattern](https://doc.rust-lang.org/book/ch17-03-oo-design-patterns.html)

---

*State & Strategy: polymorphism done the Rust way! 🦀*
