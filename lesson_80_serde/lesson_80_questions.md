# 🧪 Lesson 80 — Questions: Serialisation with Serde (RW3)

> **Lesson:** [lesson_80_serde.md](./lesson_80_serde.md)  
> **Answers:** [lesson_80_answers.md](./lesson_80_answers.md)

---

## Section A — Predict the Output

### Q1
```rust
use serde::{Serialize, Deserialize};
#[derive(Serialize, Deserialize, Debug)]
struct Point { x: i32, #[serde(rename = "Y")] y: i32 }
fn main() {
    let p = Point { x: 1, y: 2 };
    println!("{}", serde_json::to_string(&p).unwrap());
}
```

---

## Section B — Write It Yourself

### Q2 — Config reader (Roadmap Practice Task)
Create a TOML config file with `[server]` and `[logging]` sections. Write a Rust program that reads it into typed structs, with defaults for optional fields.

### Q3 — JSON API response
Create an `ApiResponse<T>` struct with `success: bool`, `data: Option<T>`, `error: Option<String>`. Serialize and deserialize it with `skip_serializing_if`.

### Q4 — Enum serialization
Create a `Message` enum with `Text(String)`, `Image { url: String }`, `Ping`. Serialize using internal tagging (`#[serde(tag = "type")]`).

---

## Section C — True or False?

### Q5
1. `#[derive(Serialize)]` generates serialization code at compile time.
2. `serde_json::Value` allows working with untyped JSON.
3. `#[serde(default)]` fills missing fields with `Default::default()`.
4. Serde only supports JSON format.
5. `#[serde(flatten)]` merges nested struct fields into the parent.
6. `#[serde(skip_serializing_if = "Option::is_none")]` omits `None` values.

---

*Serde: any struct, any format, any time! 🦀*
