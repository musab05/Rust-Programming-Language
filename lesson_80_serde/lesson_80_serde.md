# 📘 Lesson 80 — Serialisation with Serde (RW3)

> **Series:** Rust From Zero · Intermediate Level (Gap Fill)  
> **Roadmap ID:** RW3 · Category: 🌍 Real World  
> **Previous:** [Lesson 79 — CLI Apps with clap](../lesson_79_clap/lesson_79_clap.md)  
> **Next:** [Lesson 81 — HTTP Client with reqwest](../lesson_81_reqwest/lesson_81_reqwest.md)  
> **Practice:** [Questions](./lesson_80_questions.md) · [Answers](./lesson_80_answers.md)  
> **Practice Task:** Config file reader with typed struct + serde

---

## Table of Contents

1. [What Is Serde?](#1-what-is-serde)
2. [Setup](#2-setup)
3. [Derive Serialize and Deserialize](#3-derive-serialize-and-deserialize)
4. [JSON](#4-json)
5. [TOML](#5-toml)
6. [Serde Attributes](#6-serde-attributes)
7. [Enums with Serde](#7-enums-with-serde)
8. [Custom Serialisation](#8-custom-serialisation)
9. [Real-World Example: Config File](#9-real-world-example-config-file)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. What Is Serde?

**Ser**ialize + **De**serialize — convert Rust structs to/from data formats:

```
Rust struct ←→ JSON
Rust struct ←→ TOML
Rust struct ←→ YAML
Rust struct ←→ MessagePack
Rust struct ←→ CSV
Rust struct ←→ Binary (bincode)
```

---

## 2. Setup

```toml
[dependencies]
serde = { version = "1", features = ["derive"] }
serde_json = "1"        # for JSON
toml = "0.8"             # for TOML
# serde_yaml = "0.9"     # for YAML
```

---

## 3. Derive Serialize and Deserialize

```rust
use serde::{Serialize, Deserialize};

#[derive(Debug, Serialize, Deserialize)]
struct User {
    name: String,
    age: u32,
    email: String,
    active: bool,
}

fn main() {
    let user = User {
        name: "Alice".into(),
        age: 30,
        email: "alice@example.com".into(),
        active: true,
    };

    // Serialize to JSON
    let json = serde_json::to_string_pretty(&user).unwrap();
    println!("JSON:\n{json}");

    // Deserialize from JSON
    let parsed: User = serde_json::from_str(&json).unwrap();
    println!("\nParsed: {:?}", parsed);
}
```

Output:
```json
{
  "name": "Alice",
  "age": 30,
  "email": "alice@example.com",
  "active": true
}
```

---

## 4. JSON

```rust
use serde::{Serialize, Deserialize};
use serde_json::{json, Value};

#[derive(Debug, Serialize, Deserialize)]
struct Post {
    title: String,
    body: String,
    tags: Vec<String>,
    published: bool,
}

fn main() {
    // Serialize
    let post = Post {
        title: "Rust is Great".into(),
        body: "Let me tell you why...".into(),
        tags: vec!["rust".into(), "programming".into()],
        published: true,
    };
    let json = serde_json::to_string(&post).unwrap();
    println!("Compact: {json}");

    // Deserialize
    let raw = r#"{"title":"Hello","body":"World","tags":["test"],"published":false}"#;
    let p: Post = serde_json::from_str(raw).unwrap();
    println!("Parsed: {:?}", p);

    // Untyped JSON (serde_json::Value)
    let v: Value = serde_json::from_str(raw).unwrap();
    println!("Title: {}", v["title"]);

    // Build JSON dynamically
    let dynamic = json!({
        "name": "Bob",
        "scores": [95, 87, 92],
        "metadata": { "level": "advanced" }
    });
    println!("Dynamic: {}", serde_json::to_string_pretty(&dynamic).unwrap());
}
```

---

## 5. TOML

```rust
use serde::{Serialize, Deserialize};

#[derive(Debug, Serialize, Deserialize)]
struct Config {
    server: ServerConfig,
    database: DatabaseConfig,
}

#[derive(Debug, Serialize, Deserialize)]
struct ServerConfig {
    host: String,
    port: u16,
    workers: usize,
}

#[derive(Debug, Serialize, Deserialize)]
struct DatabaseConfig {
    url: String,
    pool_size: u32,
}

fn main() {
    let toml_str = r#"
[server]
host = "0.0.0.0"
port = 8080
workers = 4

[database]
url = "postgres://localhost/mydb"
pool_size = 10
"#;

    // Deserialize from TOML
    let config: Config = toml::from_str(toml_str).unwrap();
    println!("{:#?}", config);

    // Serialize to TOML
    let output = toml::to_string_pretty(&config).unwrap();
    println!("TOML:\n{output}");
}
```

---

## 6. Serde Attributes

```rust
use serde::{Serialize, Deserialize};

#[derive(Debug, Serialize, Deserialize)]
struct ApiResponse {
    // Rename field in JSON
    #[serde(rename = "userId")]
    user_id: u32,

    // Use a default value if missing
    #[serde(default)]
    verified: bool,

    // Skip field during serialization
    #[serde(skip_serializing)]
    internal_token: String,

    // Skip if None
    #[serde(skip_serializing_if = "Option::is_none")]
    nickname: Option<String>,

    // Custom default
    #[serde(default = "default_role")]
    role: String,

    // Flatten nested struct
    #[serde(flatten)]
    metadata: Metadata,
}

#[derive(Debug, Serialize, Deserialize)]
struct Metadata {
    created_at: String,
    version: u32,
}

fn default_role() -> String { "user".to_string() }

fn main() {
    let json = r#"{
        "userId": 42,
        "internal_token": "secret",
        "created_at": "2024-01-01",
        "version": 1
    }"#;

    let resp: ApiResponse = serde_json::from_str(json).unwrap();
    println!("{:#?}", resp);
    // verified defaults to false, role defaults to "user", nickname is None

    let output = serde_json::to_string_pretty(&resp).unwrap();
    println!("\nSerialized:\n{output}");
    // internal_token is skipped, nickname is skipped (None)
}
```

---

## 7. Enums with Serde

```rust
use serde::{Serialize, Deserialize};

// External tagging (default)
#[derive(Debug, Serialize, Deserialize)]
enum Shape {
    Circle { radius: f64 },
    Rectangle { width: f64, height: f64 },
    Point,
}

// Internal tagging
#[derive(Debug, Serialize, Deserialize)]
#[serde(tag = "type")]
enum Event {
    #[serde(rename = "click")]
    Click { x: i32, y: i32 },
    #[serde(rename = "key")]
    KeyPress { key: String },
}

// Adjacent tagging
#[derive(Debug, Serialize, Deserialize)]
#[serde(tag = "t", content = "c")]
enum Message {
    Text(String),
    Image { url: String, width: u32 },
}

fn main() {
    // External: {"Circle":{"radius":5.0}}
    let s = Shape::Circle { radius: 5.0 };
    println!("External: {}", serde_json::to_string(&s).unwrap());

    // Internal: {"type":"click","x":10,"y":20}
    let e = Event::Click { x: 10, y: 20 };
    println!("Internal: {}", serde_json::to_string(&e).unwrap());

    // Adjacent: {"t":"Text","c":"hello"}
    let m = Message::Text("hello".into());
    println!("Adjacent: {}", serde_json::to_string(&m).unwrap());
}
```

---

## 8. Custom Serialisation

```rust
use serde::{Serialize, Deserialize, Serializer, Deserializer};

#[derive(Debug)]
struct Color { r: u8, g: u8, b: u8 }

impl Serialize for Color {
    fn serialize<S: Serializer>(&self, serializer: S) -> Result<S::Ok, S::Error> {
        let hex = format!("#{:02x}{:02x}{:02x}", self.r, self.g, self.b);
        serializer.serialize_str(&hex)
    }
}

impl<'de> Deserialize<'de> for Color {
    fn deserialize<D: Deserializer<'de>>(deserializer: D) -> Result<Self, D::Error> {
        let s: String = String::deserialize(deserializer)?;
        let s = s.trim_start_matches('#');
        if s.len() != 6 { return Err(serde::de::Error::custom("invalid hex color")); }
        let r = u8::from_str_radix(&s[0..2], 16).map_err(serde::de::Error::custom)?;
        let g = u8::from_str_radix(&s[2..4], 16).map_err(serde::de::Error::custom)?;
        let b = u8::from_str_radix(&s[4..6], 16).map_err(serde::de::Error::custom)?;
        Ok(Color { r, g, b })
    }
}

fn main() {
    let c = Color { r: 255, g: 128, b: 0 };
    let json = serde_json::to_string(&c).unwrap();
    println!("Serialized: {json}");  // "#ff8000"

    let parsed: Color = serde_json::from_str(&json).unwrap();
    println!("Parsed: {:?}", parsed);
}
```

---

## 9. Real-World Example: Config File

```rust
use serde::{Serialize, Deserialize};
use std::fs;

#[derive(Debug, Serialize, Deserialize)]
struct AppConfig {
    #[serde(default = "default_app_name")]
    app_name: String,
    server: Server,
    #[serde(default)]
    features: Features,
}

#[derive(Debug, Serialize, Deserialize)]
struct Server {
    host: String,
    port: u16,
    #[serde(default = "default_workers")]
    workers: usize,
}

#[derive(Debug, Default, Serialize, Deserialize)]
struct Features {
    #[serde(default)]
    logging: bool,
    #[serde(default)]
    metrics: bool,
}

fn default_app_name() -> String { "MyApp".into() }
fn default_workers() -> usize { 4 }

fn main() {
    // Create a default config file if it doesn't exist
    let config_path = "config.toml";
    if !std::path::Path::new(config_path).exists() {
        let default_config = AppConfig {
            app_name: "MyApp".into(),
            server: Server { host: "127.0.0.1".into(), port: 3000, workers: 4 },
            features: Features { logging: true, metrics: false },
        };
        let toml = toml::to_string_pretty(&default_config).unwrap();
        fs::write(config_path, &toml).unwrap();
        println!("Created default {config_path}:\n{toml}");
    }

    // Read and parse config
    let content = fs::read_to_string(config_path).unwrap();
    let config: AppConfig = toml::from_str(&content).unwrap();
    println!("Loaded: {:#?}", config);
}
```

---

## 10. Summary Cheat Sheet

```
SETUP
────────────────────────────────────────────────────────────
serde = { version = "1", features = ["derive"] }
serde_json = "1"   /  toml = "0.8"

DERIVE
────────────────────────────────────────────────────────────
#[derive(Serialize, Deserialize)]

SERIALIZE / DESERIALIZE
────────────────────────────────────────────────────────────
serde_json::to_string(&val)     → JSON string
serde_json::from_str(json)      → Rust struct
toml::to_string_pretty(&val)    → TOML string
toml::from_str(toml)            → Rust struct

ATTRIBUTES
────────────────────────────────────────────────────────────
#[serde(rename = "name")]       rename field
#[serde(default)]               use Default if missing
#[serde(skip_serializing)]      omit from output
#[serde(skip_serializing_if)]   conditional skip
#[serde(flatten)]               flatten nested struct
#[serde(tag = "type")]          enum tagging strategy

FORMATS
────────────────────────────────────────────────────────────
serde_json   JSON
toml         TOML
serde_yaml   YAML
bincode      Binary
csv          CSV
```

---

## What's Next?

**Lesson 81 — HTTP Client with reqwest** — Make HTTP requests, handle JSON APIs, and manage async networking.

## Further Reading
- [Serde docs](https://serde.rs/)
- [serde_json](https://docs.rs/serde_json/)
- [toml crate](https://docs.rs/toml/)

---

*Serde: the universal translator between Rust and the data world! 🦀*
