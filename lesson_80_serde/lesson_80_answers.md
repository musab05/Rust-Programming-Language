# ✅ Lesson 80 — Answers: Serialisation with Serde (RW3)

---

## Section A

### A1
```
{"x":1,"Y":2}
```
The `y` field is renamed to `"Y"` in JSON output via `#[serde(rename = "Y")]`.

---

## Section B

### A2
```rust
use serde::{Serialize, Deserialize};

#[derive(Debug, Serialize, Deserialize)]
struct Config {
    server: Server,
    #[serde(default)]
    logging: Logging,
}

#[derive(Debug, Serialize, Deserialize)]
struct Server { host: String, port: u16 }

#[derive(Debug, Serialize, Deserialize)]
struct Logging {
    #[serde(default = "default_level")]
    level: String,
    #[serde(default)]
    file: Option<String>,
}

impl Default for Logging {
    fn default() -> Self { Logging { level: "info".into(), file: None } }
}

fn default_level() -> String { "info".into() }

fn main() {
    let toml_str = r#"
[server]
host = "0.0.0.0"
port = 3000
"#;
    let config: Config = toml::from_str(toml_str).unwrap();
    println!("{:#?}", config);
    // logging uses defaults: level="info", file=None
}
```

### A3
```rust
use serde::{Serialize, Deserialize};

#[derive(Debug, Serialize, Deserialize)]
struct ApiResponse<T: serde::Serialize> {
    success: bool,
    #[serde(skip_serializing_if = "Option::is_none")]
    data: Option<T>,
    #[serde(skip_serializing_if = "Option::is_none")]
    error: Option<String>,
}

fn main() {
    let ok: ApiResponse<Vec<String>> = ApiResponse {
        success: true,
        data: Some(vec!["Alice".into()]),
        error: None,
    };
    println!("{}", serde_json::to_string_pretty(&ok).unwrap());
    // "error" field is omitted
}
```

### A4
```rust
use serde::{Serialize, Deserialize};

#[derive(Debug, Serialize, Deserialize)]
#[serde(tag = "type")]
enum Message {
    #[serde(rename = "text")]
    Text { content: String },
    #[serde(rename = "image")]
    Image { url: String },
    #[serde(rename = "ping")]
    Ping,
}

fn main() {
    let msgs = vec![
        Message::Text { content: "hello".into() },
        Message::Image { url: "https://img.jpg".into() },
        Message::Ping,
    ];
    for m in &msgs {
        println!("{}", serde_json::to_string(m).unwrap());
    }
    // {"type":"text","content":"hello"}
    // {"type":"image","url":"https://img.jpg"}
    // {"type":"ping"}
}
```

---

## Section C

### A5
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | Derive macros generate code at compile time |
| 2 | **True** | `Value` is a dynamic enum representing any JSON value |
| 3 | **True** | Missing fields get `Default::default()` or custom fn |
| 4 | **False** | Serde supports JSON, TOML, YAML, bincode, CSV, and more |
| 5 | **True** | `flatten` inlines nested struct fields into parent JSON object |
| 6 | **True** | The predicate skips serializing when the value is `None` |

---

## 🏆 Lesson 80 Complete!

**Next up:** [Lesson 81 — HTTP Client with reqwest](../lesson_81_reqwest/lesson_81_reqwest.md) 🦀
