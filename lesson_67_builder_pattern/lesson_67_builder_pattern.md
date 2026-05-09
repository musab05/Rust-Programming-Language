# 📘 Lesson 67 — Design Patterns: Builder (DP1)

> **Series:** Rust From Zero · Advanced Level  
> **Roadmap ID:** DP1 · Category: 🏗 Design Patterns  
> **Previous:** [Lesson 66 — Advanced Traits](../lesson_66_advanced_traits/lesson_66_advanced_traits.md)  
> **Next:** [Lesson 68 — Design Patterns: State & Strategy](../lesson_68_state_strategy/lesson_68_state_strategy.md)  
> **Practice:** [Questions](./lesson_67_questions.md) · [Answers](./lesson_67_answers.md)  
> **Practice Task:** Implement a fluent Builder for a complex configuration struct

---

## Table of Contents

1. [Why Builders?](#1-why-builders)
2. [Basic Builder](#2-basic-builder)
3. [Fluent Builder (Method Chaining)](#3-fluent-builder-method-chaining)
4. [Builder with Validation](#4-builder-with-validation)
5. [Builder with Required vs Optional Fields](#5-builder-with-required-vs-optional-fields)
6. [Type-Safe Builder](#6-type-safe-builder)
7. [derive_builder Crate](#7-derive_builder-crate)
8. [Real-World Example: HTTP Request Builder](#8-real-world-example-http-request-builder)
9. [Summary Cheat Sheet](#9-summary-cheat-sheet)

---

## 1. Why Builders?

When a struct has many fields, especially optional ones, constructors become unwieldy:

```rust
// ❌ Too many parameters — easy to mix up
let server = Server::new("0.0.0.0", 8080, 4, 1024, true, false, None, Some(30));

// ✅ Builder — clear and flexible
let server = Server::builder()
    .host("0.0.0.0")
    .port(8080)
    .workers(4)
    .max_connections(1024)
    .debug(true)
    .build()?;
```

---

## 2. Basic Builder

```rust
struct Config {
    host: String,
    port: u16,
    debug: bool,
    max_retries: u32,
}

struct ConfigBuilder {
    host: String,
    port: u16,
    debug: bool,
    max_retries: u32,
}

impl ConfigBuilder {
    fn new() -> Self {
        ConfigBuilder {
            host: "localhost".into(),
            port: 8080,
            debug: false,
            max_retries: 3,
        }
    }

    fn host(mut self, host: &str) -> Self { self.host = host.into(); self }
    fn port(mut self, port: u16) -> Self { self.port = port; self }
    fn debug(mut self, debug: bool) -> Self { self.debug = debug; self }
    fn max_retries(mut self, n: u32) -> Self { self.max_retries = n; self }

    fn build(self) -> Config {
        Config {
            host: self.host,
            port: self.port,
            debug: self.debug,
            max_retries: self.max_retries,
        }
    }
}

impl Config {
    fn builder() -> ConfigBuilder { ConfigBuilder::new() }
}

fn main() {
    let config = Config::builder()
        .host("0.0.0.0")
        .port(3000)
        .debug(true)
        .build();

    println!("{}:{} debug={} retries={}",
        config.host, config.port, config.debug, config.max_retries);
}
```

---

## 3. Fluent Builder (Method Chaining)

Using `&mut self` instead of `self` — allows reuse:

```rust
struct QueryBuilder {
    table: String,
    conditions: Vec<String>,
    limit: Option<usize>,
    order_by: Option<String>,
}

impl QueryBuilder {
    fn new(table: &str) -> Self {
        QueryBuilder {
            table: table.into(),
            conditions: vec![],
            limit: None,
            order_by: None,
        }
    }

    fn where_clause(&mut self, condition: &str) -> &mut Self {
        self.conditions.push(condition.into());
        self
    }

    fn limit(&mut self, n: usize) -> &mut Self {
        self.limit = Some(n);
        self
    }

    fn order_by(&mut self, column: &str) -> &mut Self {
        self.order_by = Some(column.into());
        self
    }

    fn build(&self) -> String {
        let mut query = format!("SELECT * FROM {}", self.table);
        if !self.conditions.is_empty() {
            query += &format!(" WHERE {}", self.conditions.join(" AND "));
        }
        if let Some(ref col) = self.order_by {
            query += &format!(" ORDER BY {col}");
        }
        if let Some(n) = self.limit {
            query += &format!(" LIMIT {n}");
        }
        query
    }
}

fn main() {
    let query = QueryBuilder::new("users")
        .where_clause("age > 18")
        .where_clause("active = true")
        .order_by("name")
        .limit(10)
        .build();

    println!("{query}");
    // SELECT * FROM users WHERE age > 18 AND active = true ORDER BY name LIMIT 10
}
```

---

## 4. Builder with Validation

Return `Result` from `build()`:

```rust
#[derive(Debug)]
struct Server {
    host: String,
    port: u16,
    workers: usize,
}

struct ServerBuilder {
    host: Option<String>,
    port: Option<u16>,
    workers: usize,
}

impl ServerBuilder {
    fn new() -> Self {
        ServerBuilder { host: None, port: None, workers: num_cpus() }
    }

    fn host(mut self, h: &str) -> Self { self.host = Some(h.into()); self }
    fn port(mut self, p: u16) -> Self { self.port = Some(p); self }
    fn workers(mut self, w: usize) -> Self { self.workers = w; self }

    fn build(self) -> Result<Server, String> {
        let host = self.host.ok_or("Host is required")?;
        let port = self.port.ok_or("Port is required")?;
        if port == 0 { return Err("Port cannot be 0".into()); }
        if self.workers == 0 { return Err("Workers must be > 0".into()); }
        Ok(Server { host, port, workers: self.workers })
    }
}

fn num_cpus() -> usize { 4 }

fn main() {
    let server = ServerBuilder::new()
        .host("0.0.0.0")
        .port(8080)
        .build()
        .unwrap();
    println!("{:?}", server);

    let err = ServerBuilder::new().port(8080).build();
    println!("{:?}", err);  // Err("Host is required")
}
```

---

## 5. Builder with Required vs Optional Fields

```rust
#[derive(Debug)]
struct Email {
    from: String,
    to: Vec<String>,
    subject: String,
    body: String,
    cc: Vec<String>,
    bcc: Vec<String>,
    reply_to: Option<String>,
    priority: u8,
}

struct EmailBuilder {
    from: String,
    to: Vec<String>,
    subject: String,
    body: String,
    cc: Vec<String>,
    bcc: Vec<String>,
    reply_to: Option<String>,
    priority: u8,
}

impl EmailBuilder {
    // Required fields in constructor
    fn new(from: &str, to: &str, subject: &str) -> Self {
        EmailBuilder {
            from: from.into(),
            to: vec![to.into()],
            subject: subject.into(),
            body: String::new(),
            cc: vec![],
            bcc: vec![],
            reply_to: None,
            priority: 3,
        }
    }

    // Optional fields
    fn body(mut self, b: &str) -> Self { self.body = b.into(); self }
    fn cc(mut self, addr: &str) -> Self { self.cc.push(addr.into()); self }
    fn bcc(mut self, addr: &str) -> Self { self.bcc.push(addr.into()); self }
    fn reply_to(mut self, addr: &str) -> Self { self.reply_to = Some(addr.into()); self }
    fn priority(mut self, p: u8) -> Self { self.priority = p; self }
    fn add_to(mut self, addr: &str) -> Self { self.to.push(addr.into()); self }

    fn build(self) -> Email {
        Email {
            from: self.from, to: self.to, subject: self.subject,
            body: self.body, cc: self.cc, bcc: self.bcc,
            reply_to: self.reply_to, priority: self.priority,
        }
    }
}

fn main() {
    let email = EmailBuilder::new("me@example.com", "you@example.com", "Hello!")
        .body("How are you?")
        .cc("manager@example.com")
        .priority(1)
        .build();

    println!("{:#?}", email);
}
```

---

## 6. Type-Safe Builder

Use the type system to enforce required fields at compile time:

```rust
use std::marker::PhantomData;

struct Missing;
struct Provided;

struct RequestBuilder<Host, Path> {
    host: String,
    path: String,
    timeout: u64,
    _h: PhantomData<Host>,
    _p: PhantomData<Path>,
}

impl RequestBuilder<Missing, Missing> {
    fn new() -> Self {
        RequestBuilder {
            host: String::new(), path: String::new(), timeout: 30,
            _h: PhantomData, _p: PhantomData,
        }
    }
}

impl<P> RequestBuilder<Missing, P> {
    fn host(self, host: &str) -> RequestBuilder<Provided, P> {
        RequestBuilder {
            host: host.into(), path: self.path, timeout: self.timeout,
            _h: PhantomData, _p: PhantomData,
        }
    }
}

impl<H> RequestBuilder<H, Missing> {
    fn path(self, path: &str) -> RequestBuilder<H, Provided> {
        RequestBuilder {
            host: self.host, path: path.into(), timeout: self.timeout,
            _h: PhantomData, _p: PhantomData,
        }
    }
}

impl<H, P> RequestBuilder<H, P> {
    fn timeout(mut self, t: u64) -> Self { self.timeout = t; self }
}

// build() only available when BOTH host and path are provided
impl RequestBuilder<Provided, Provided> {
    fn build(self) -> String {
        format!("https://{}{}?timeout={}", self.host, self.path, self.timeout)
    }
}

fn main() {
    let url = RequestBuilder::new()
        .host("api.example.com")
        .path("/users")
        .timeout(10)
        .build();
    println!("{url}");

    // ❌ Won't compile — missing path:
    // RequestBuilder::new().host("x").build();
}
```

---

## 7. derive_builder Crate

Automate builder generation:

```toml
[dependencies]
derive_builder = "0.20"
```

```rust
use derive_builder::Builder;

#[derive(Builder, Debug)]
struct Server {
    host: String,
    port: u16,
    #[builder(default = "4")]
    workers: usize,
    #[builder(default = "false")]
    debug: bool,
}

fn main() {
    let server = ServerBuilder::default()
        .host("0.0.0.0".into())
        .port(8080)
        .debug(true)
        .build()
        .unwrap();
    println!("{:?}", server);
}
```

---

## 8. Real-World Example: HTTP Request Builder

```rust
use std::collections::HashMap;

#[derive(Debug)]
struct HttpRequest {
    method: String,
    url: String,
    headers: HashMap<String, String>,
    body: Option<String>,
    timeout_ms: u64,
}

struct HttpRequestBuilder {
    method: String,
    url: String,
    headers: HashMap<String, String>,
    body: Option<String>,
    timeout_ms: u64,
}

impl HttpRequestBuilder {
    fn get(url: &str) -> Self { Self::new("GET", url) }
    fn post(url: &str) -> Self { Self::new("POST", url) }

    fn new(method: &str, url: &str) -> Self {
        HttpRequestBuilder {
            method: method.into(), url: url.into(),
            headers: HashMap::new(), body: None, timeout_ms: 5000,
        }
    }

    fn header(mut self, key: &str, value: &str) -> Self {
        self.headers.insert(key.into(), value.into()); self
    }

    fn json(mut self, body: &str) -> Self {
        self.body = Some(body.into());
        self.headers.insert("Content-Type".into(), "application/json".into());
        self
    }

    fn timeout(mut self, ms: u64) -> Self { self.timeout_ms = ms; self }

    fn build(self) -> HttpRequest {
        HttpRequest {
            method: self.method, url: self.url,
            headers: self.headers, body: self.body, timeout_ms: self.timeout_ms,
        }
    }
}

fn main() {
    let req = HttpRequestBuilder::post("https://api.example.com/users")
        .header("Authorization", "Bearer token123")
        .json(r#"{"name": "Alice", "age": 30}"#)
        .timeout(3000)
        .build();

    println!("{:#?}", req);
}
```

---

## 9. Summary Cheat Sheet

```
BASIC BUILDER
────────────────────────────────────────────────────────────
struct FooBuilder { ... }
fn field(mut self, v: T) -> Self { self.x = v; self }
fn build(self) -> Foo { ... }

WITH VALIDATION
────────────────────────────────────────────────────────────
fn build(self) -> Result<Foo, Error> { ... }

REQUIRED + OPTIONAL
────────────────────────────────────────────────────────────
Required → in new() constructor
Optional → setter methods with defaults

TYPE-SAFE BUILDER
────────────────────────────────────────────────────────────
PhantomData<Missing/Provided> as type params
build() only on Builder<Provided, Provided>

WHEN TO USE
────────────────────────────────────────────────────────────
Many optional fields
Complex construction logic
Validation needed
Configuration objects
```

---

## What's Next?

**Lesson 68 — Design Patterns: State & Strategy** — Runtime polymorphism patterns. Model state machines and interchangeable algorithms.

## Further Reading
- [Rust Design Patterns — Builder](https://rust-unofficial.github.io/patterns/patterns/creational/builder.html)
- [derive_builder](https://docs.rs/derive_builder/)

---

*Builder pattern: constructing complexity with clarity! 🦀*
