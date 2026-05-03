# ✅ Lesson 43 — Answers: Standard Traits (T6)

---

## Section A

### A1
`Display` enables `{}`. `Debug` enables `{:?}`.

### A2
No, `Display` cannot be derived. The format is a design decision — the compiler can't guess whether you want `rgb(255,0,0)`, `#FF0000`, or `Color(red)`.

### A3
`Copy` requires `Clone` (it's a supertrait). You cannot have `Copy` without `Clone`. `Copy` means implicit bitwise copy; `Clone` means explicit `.clone()`.

### A4
`String` allocates on the heap. A bitwise copy would create two owners of the same heap memory — double free on drop. `String` must use `Clone` for explicit deep copies.

---

## Section B

### A5
```rust
use std::fmt;

#[derive(Debug, Clone, Copy, PartialEq, Eq)]
struct Color { r: u8, g: u8, b: u8 }

impl fmt::Display for Color {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "#{:02X}{:02X}{:02X}", self.r, self.g, self.b)
    }
}

impl Default for Color {
    fn default() -> Self { Color { r: 0, g: 0, b: 0 } }
}

impl From<(u8, u8, u8)> for Color {
    fn from((r, g, b): (u8, u8, u8)) -> Self { Color { r, g, b } }
}

fn main() {
    let red = Color { r: 255, g: 0, b: 0 };
    println!("{red}");  // #FF0000
    let blue: Color = (0, 0, 255).into();
    println!("{blue}");  // #0000FF
    println!("Default: {}", Color::default());  // #000000
    println!("Equal: {}", red == Color::from((255, 0, 0)));  // true
}
```

### A6
```rust
use std::fmt;

#[derive(Debug)]
struct Celsius(f64);
#[derive(Debug)]
struct Fahrenheit(f64);

impl fmt::Display for Celsius {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result { write!(f, "{:.1}°C", self.0) }
}
impl fmt::Display for Fahrenheit {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result { write!(f, "{:.1}°F", self.0) }
}

impl From<Celsius> for Fahrenheit {
    fn from(c: Celsius) -> Self { Fahrenheit(c.0 * 9.0 / 5.0 + 32.0) }
}
impl From<Fahrenheit> for Celsius {
    fn from(f: Fahrenheit) -> Self { Celsius((f.0 - 32.0) * 5.0 / 9.0) }
}

fn main() {
    let boiling = Celsius(100.0);
    let f: Fahrenheit = boiling.into();
    println!("{f}");  // 212.0°F
    let body = Fahrenheit(98.6);
    let c: Celsius = body.into();
    println!("{c}");  // 37.0°C
}
```

### A7
```rust
#[derive(Debug)]
struct DatabaseConfig {
    host: String, port: u16, database: String, pool_size: u32,
}

impl Default for DatabaseConfig {
    fn default() -> Self {
        DatabaseConfig {
            host: "localhost".into(),
            port: 5432,
            database: "app_production".into(),
            pool_size: 10,
        }
    }
}

fn main() {
    let config = DatabaseConfig { database: "my_app".into(), ..Default::default() };
    println!("{:?}", config);
}
```

---

## Section C

### A8
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **False** | `Eq` is a marker trait — no new methods, just guarantees reflexivity |
| 2 | **True** | The blanket impl `impl<A, B: From<A>> Into<B> for A` provides this |
| 3 | **True** | Derive macros require all fields to implement the trait |
| 4 | **False** | `f64` has `NaN != NaN`, violating reflexivity, so only `PartialEq` |
| 5 | **True** | `bool::default()` is `false` |
| 6 | **True** | The blanket impl `impl<T: Display> ToString for T` provides `.to_string()` |

### A9
```rust
use std::hash::{Hash, Hasher};

struct User { id: u64, email: String }

impl PartialEq for User {
    fn eq(&self, other: &Self) -> bool { self.id == other.id }
}
impl Eq for User {}

impl Hash for User {
    fn hash<H: Hasher>(&self, state: &mut H) { self.id.hash(state); }
}
```
Only `id` is used for equality and hashing — two users with the same `id` but different emails are considered equal. This is the **identity-based equality** pattern.

---

## 🏆 Lesson 43 Complete!

**Next up:** [Lesson 44 — Operator Overloading](../lesson_44_operator_overloading/lesson_44_operator_overloading.md) 🦀
