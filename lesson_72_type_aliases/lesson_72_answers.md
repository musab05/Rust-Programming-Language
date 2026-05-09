# ✅ Lesson 72 — Answers: Type Aliases (AT1)

---

## Section A

### A1 — ✅ Compiles
`Score` is just `i32`. Mixing `Score` and `i32` is fine — they're the same type. Output: `30`.

### A2 — ✅ Compiles
The alias makes the closure type readable. Output: `10`.

---

## Section B

### A3
```rust
use std::collections::HashMap;

type DataMap = HashMap<String, Vec<(u32, String, bool)>>;
type Processor = Box<dyn Fn(&str) -> Result<String, String>>;

fn process(data: &DataMap, proc: &Processor) -> Vec<String> {
    let mut results = vec![];
    for key in data.keys() {
        match proc(key) {
            Ok(r) => results.push(r),
            Err(e) => results.push(format!("Error: {e}")),
        }
    }
    results
}

fn main() {
    let mut data: DataMap = HashMap::new();
    data.insert("users".into(), vec![(1, "Alice".into(), true)]);

    let upper: Processor = Box::new(|s| Ok(s.to_uppercase()));
    let results = process(&data, &upper);
    println!("{:?}", results);
}
```

### A4
```rust
#[derive(Debug)]
enum AppError { NotFound(String), InvalidInput(String), Internal(String) }
type AppResult<T> = Result<T, AppError>;

fn find(id: u32) -> AppResult<String> {
    if id == 0 { Err(AppError::NotFound("id=0".into())) }
    else { Ok(format!("item_{id}")) }
}

fn validate(s: &str) -> AppResult<u32> {
    s.parse().map_err(|_| AppError::InvalidInput(s.into()))
}

fn process(s: &str) -> AppResult<String> {
    let id = validate(s)?;
    find(id)
}

fn main() {
    println!("{:?}", process("5"));    // Ok("item_5")
    println!("{:?}", process("abc"));  // Err(InvalidInput("abc"))
}
```

---

## Section C

### A5
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **False** | Aliases are just alternative names — same type underneath |
| 2 | **True** | `Meters` IS `f64`, so assignment works freely |
| 3 | **True** | `std::io::Result<T>` is `Result<T, io::Error>` |
| 4 | **True** | e.g., `type StringMap<V> = HashMap<String, V>` |
| 5 | **False** | Aliases provide readability, NOT type safety — use newtypes for that |

---

## 🏆 Lesson 72 Complete!

**Next up:** [Lesson 73 — Newtype Pattern](../lesson_73_newtype/lesson_73_newtype.md) 🦀
