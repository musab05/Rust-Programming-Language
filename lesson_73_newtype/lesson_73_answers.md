# ✅ Lesson 73 — Answers: Newtype Pattern (AT2)

---

## Section A

### A1 — ❌ Won't compile
`Meters` and `Seconds` are distinct types. You can't assign one to the other. Error: `expected Seconds, found Meters`.

### A2 — ✅ Compiles
Type aliases are the SAME type. `Meters` and `Seconds` are both `f64`, so assignment is fine.

---

## Section B

### A3
```rust
#[derive(Debug, Clone)]
struct Email(String);

impl Email {
    fn new(s: &str) -> Result<Self, String> {
        let s = s.trim();
        if !s.contains('@') { return Err("Missing '@'".into()); }
        let parts: Vec<&str> = s.split('@').collect();
        if parts.len() != 2 || parts[0].is_empty() || parts[1].is_empty() {
            return Err("Invalid format".into());
        }
        if !parts[1].contains('.') { return Err("Missing domain".into()); }
        Ok(Email(s.to_lowercase()))
    }
    fn as_str(&self) -> &str { &self.0 }
    fn domain(&self) -> &str { self.0.split('@').nth(1).unwrap_or("") }
}

impl std::fmt::Display for Email {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result { write!(f, "{}", self.0) }
}

fn main() {
    let e = Email::new("Alice@Example.COM").unwrap();
    println!("{e}");                       // alice@example.com
    println!("Domain: {}", e.domain());   // example.com
}
```

### A4
```rust
#[derive(Debug, Clone, Copy)]
struct Celsius(f64);
#[derive(Debug, Clone, Copy)]
struct Fahrenheit(f64);

impl From<Celsius> for Fahrenheit {
    fn from(c: Celsius) -> Self { Fahrenheit(c.0 * 9.0 / 5.0 + 32.0) }
}
impl From<Fahrenheit> for Celsius {
    fn from(f: Fahrenheit) -> Self { Celsius((f.0 - 32.0) * 5.0 / 9.0) }
}

fn heat_to(temp: Celsius) { println!("Heating to {:.1}°C", temp.0); }

fn main() {
    let c = Celsius(100.0);
    let f: Fahrenheit = c.into();
    println!("{:.1}°C = {:.1}°F", c.0, f.0);
    heat_to(c);
    // heat_to(f);  // ❌ compile error
}
```

### A5
```rust
struct IntList(Vec<i32>);
impl std::fmt::Display for IntList {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        let s: Vec<String> = self.0.iter().map(|n| n.to_string()).collect();
        write!(f, "{{{}}}", s.join(", "))
    }
}
fn main() {
    let list = IntList(vec![1, 2, 3]);
    println!("{list}");  // {1, 2, 3}
}
```

---

## Section C

### A6
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **False** | Newtypes are zero-cost — optimized away by the compiler |
| 2 | **True** | It's a new struct, distinct from bare `f64` |
| 3 | **True** | You can implement foreign traits on your newtype |
| 4 | **True** | `repr(transparent)` guarantees identical layout |
| 5 | **True** | Deref coercion lets methods of the inner type be called |

---

## 🏆 Lesson 73 Complete!

**Next up:** [Lesson 74 — Stack vs Heap](../lesson_74_stack_heap/lesson_74_stack_heap.md) 🦀
