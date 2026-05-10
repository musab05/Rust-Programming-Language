# 📘 Lesson 81 — HTTP Client with reqwest (RW4)

> **Series:** Rust From Zero · Intermediate Level (Gap Fill)  
> **Roadmap ID:** RW4 · Category: 🌍 Real World  
> **Previous:** [Lesson 80 — Serialisation with Serde](../lesson_80_serde/lesson_80_serde.md)  
> **Next:** [Lesson 82 — Trait Objects & Object Safety](../lesson_82_trait_objects/lesson_82_trait_objects.md)  
> **Practice:** [Questions](./lesson_81_questions.md) · [Answers](./lesson_81_answers.md)  
> **Practice Task:** Weather app hitting a public API

---

## Table of Contents

1. [What Is reqwest?](#1-what-is-reqwest)
2. [Setup](#2-setup)
3. [Simple GET Request](#3-simple-get-request)
4. [JSON Responses](#4-json-responses)
5. [POST Requests](#5-post-requests)
6. [Headers and Query Parameters](#6-headers-and-query-parameters)
7. [Error Handling](#7-error-handling)
8. [Client Reuse and Timeouts](#8-client-reuse-and-timeouts)
9. [Real-World Example: Weather App](#9-real-world-example-weather-app)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. What Is reqwest?

`reqwest` is the most popular HTTP client crate — think of it as `requests` for Python, but async-native:

```
reqwest = async-first HTTP client
         + JSON support (via serde)
         + TLS built-in
         + connection pooling
```

---

## 2. Setup

```toml
[dependencies]
reqwest = { version = "0.12", features = ["json"] }
tokio = { version = "1", features = ["full"] }
serde = { version = "1", features = ["derive"] }
serde_json = "1"
```

---

## 3. Simple GET Request

```rust
#[tokio::main]
async fn main() -> Result<(), reqwest::Error> {
    // Simple text response
    let body = reqwest::get("https://httpbin.org/get")
        .await?
        .text()
        .await?;

    println!("Response:\n{body}");
    Ok(())
}
```

### Blocking (non-async) client:

```toml
# Cargo.toml — add blocking feature
reqwest = { version = "0.12", features = ["json", "blocking"] }
```

```rust
fn main() -> Result<(), reqwest::Error> {
    let body = reqwest::blocking::get("https://httpbin.org/get")?
        .text()?;
    println!("{body}");
    Ok(())
}
```

---

## 4. JSON Responses

```rust
use serde::Deserialize;

#[derive(Debug, Deserialize)]
struct Todo {
    #[serde(rename = "userId")]
    user_id: u32,
    id: u32,
    title: String,
    completed: bool,
}

#[tokio::main]
async fn main() -> Result<(), reqwest::Error> {
    // Fetch a single todo
    let todo: Todo = reqwest::get("https://jsonplaceholder.typicode.com/todos/1")
        .await?
        .json()
        .await?;

    println!("{:#?}", todo);

    // Fetch multiple todos
    let todos: Vec<Todo> = reqwest::get("https://jsonplaceholder.typicode.com/todos?_limit=5")
        .await?
        .json()
        .await?;

    for t in &todos {
        let status = if t.completed { "✅" } else { "⬜" };
        println!("{status} [{}] {}", t.id, t.title);
    }

    Ok(())
}
```

---

## 5. POST Requests

```rust
use serde::{Serialize, Deserialize};

#[derive(Debug, Serialize)]
struct NewPost {
    title: String,
    body: String,
    #[serde(rename = "userId")]
    user_id: u32,
}

#[derive(Debug, Deserialize)]
struct CreatedPost {
    id: u32,
    title: String,
}

#[tokio::main]
async fn main() -> Result<(), reqwest::Error> {
    let client = reqwest::Client::new();

    // POST with JSON body
    let new_post = NewPost {
        title: "Hello from Rust".into(),
        body: "This post was created with reqwest!".into(),
        user_id: 1,
    };

    let response: CreatedPost = client
        .post("https://jsonplaceholder.typicode.com/posts")
        .json(&new_post)
        .send()
        .await?
        .json()
        .await?;

    println!("Created post #{}: {}", response.id, response.title);

    // POST with form data
    let params = [("username", "alice"), ("password", "secret")];
    let resp = client
        .post("https://httpbin.org/post")
        .form(&params)
        .send()
        .await?;

    println!("Form POST status: {}", resp.status());

    Ok(())
}
```

---

## 6. Headers and Query Parameters

```rust
use reqwest::header::{self, HeaderMap, HeaderValue};

#[tokio::main]
async fn main() -> Result<(), reqwest::Error> {
    let client = reqwest::Client::new();

    // Custom headers
    let mut headers = HeaderMap::new();
    headers.insert(header::ACCEPT, HeaderValue::from_static("application/json"));
    headers.insert("X-Custom-Header", HeaderValue::from_static("rust-client"));

    let resp = client
        .get("https://httpbin.org/headers")
        .headers(headers)
        .bearer_auth("my-secret-token")  // Authorization: Bearer ...
        .send()
        .await?;

    println!("Headers response: {}", resp.text().await?);

    // Query parameters
    let resp = client
        .get("https://httpbin.org/get")
        .query(&[("search", "rust"), ("page", "1"), ("limit", "10")])
        .send()
        .await?;

    println!("Query response: {}", resp.text().await?);

    Ok(())
}
```

---

## 7. Error Handling

```rust
use reqwest::StatusCode;

#[tokio::main]
async fn main() {
    match fetch_data().await {
        Ok(data) => println!("Data: {data}"),
        Err(e) => {
            if e.is_timeout() { eprintln!("⏱ Request timed out"); }
            else if e.is_connect() { eprintln!("🔌 Connection failed"); }
            else if e.is_status() {
                if let Some(status) = e.status() {
                    match status {
                        StatusCode::NOT_FOUND => eprintln!("404: Not found"),
                        StatusCode::UNAUTHORIZED => eprintln!("401: Unauthorized"),
                        StatusCode::INTERNAL_SERVER_ERROR => eprintln!("500: Server error"),
                        _ => eprintln!("HTTP {status}"),
                    }
                }
            } else {
                eprintln!("Other error: {e}");
            }
        }
    }
}

async fn fetch_data() -> Result<String, reqwest::Error> {
    let resp = reqwest::get("https://httpbin.org/status/404").await?;
    // error_for_status() converts 4xx/5xx into Err
    let resp = resp.error_for_status()?;
    resp.text().await
}
```

---

## 8. Client Reuse and Timeouts

```rust
use std::time::Duration;

#[tokio::main]
async fn main() -> Result<(), reqwest::Error> {
    // Reusable client with configuration
    let client = reqwest::Client::builder()
        .timeout(Duration::from_secs(10))
        .connect_timeout(Duration::from_secs(5))
        .user_agent("my-rust-app/1.0")
        .default_headers({
            let mut h = reqwest::header::HeaderMap::new();
            h.insert("Accept", "application/json".parse().unwrap());
            h
        })
        .build()?;

    // Reuse client for multiple requests (connection pooling)
    let urls = vec![
        "https://jsonplaceholder.typicode.com/todos/1",
        "https://jsonplaceholder.typicode.com/todos/2",
        "https://jsonplaceholder.typicode.com/todos/3",
    ];

    for url in urls {
        let resp = client.get(url).send().await?;
        println!("[{}] {}", resp.status(), url);
    }

    Ok(())
}
```

### Concurrent requests:

```rust
use futures::future::join_all;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = reqwest::Client::new();
    let urls: Vec<String> = (1..=5)
        .map(|i| format!("https://jsonplaceholder.typicode.com/todos/{i}"))
        .collect();

    let futures: Vec<_> = urls.iter()
        .map(|url| {
            let client = client.clone();
            let url = url.clone();
            async move {
                let resp = client.get(&url).send().await?;
                let text = resp.text().await?;
                Ok::<_, reqwest::Error>((url, text.len()))
            }
        })
        .collect();

    let results = join_all(futures).await;
    for result in results {
        match result {
            Ok((url, len)) => println!("✅ {url} → {len} bytes"),
            Err(e) => println!("❌ Error: {e}"),
        }
    }

    Ok(())
}
```

---

## 9. Real-World Example: Weather App

```rust
use serde::Deserialize;

#[derive(Debug, Deserialize)]
struct IpInfo {
    ip: String,
    city: Option<String>,
    region: Option<String>,
    country: Option<String>,
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = reqwest::Client::builder()
        .timeout(std::time::Duration::from_secs(10))
        .build()?;

    // Step 1: Get location from IP
    println!("🌍 Getting your location...");
    let info: IpInfo = client
        .get("https://ipinfo.io/json")
        .send().await?
        .json().await?;

    println!("  IP: {}", info.ip);
    println!("  Location: {}, {}",
        info.city.as_deref().unwrap_or("Unknown"),
        info.country.as_deref().unwrap_or("Unknown"));

    // Step 2: Fetch weather (using a free API)
    println!("\n🌤 Fetching weather...");
    let weather_url = format!(
        "https://wttr.in/{}?format=j1",
        info.city.as_deref().unwrap_or("London")
    );

    let resp = client.get(&weather_url).send().await?;
    if resp.status().is_success() {
        let body: serde_json::Value = resp.json().await?;
        if let Some(current) = body.get("current_condition")
            .and_then(|v| v.as_array())
            .and_then(|a| a.first())
        {
            println!("  Temperature: {}°C", current["temp_C"]);
            println!("  Feels like:  {}°C", current["FeelsLikeC"]);
            println!("  Humidity:    {}%", current["humidity"]);
        }
    } else {
        println!("  Could not fetch weather (status: {})", resp.status());
    }

    Ok(())
}
```

---

## 10. Summary Cheat Sheet

```
SETUP
────────────────────────────────────────────────────────────
reqwest = { version = "0.12", features = ["json"] }
tokio = { version = "1", features = ["full"] }

GET
────────────────────────────────────────────────────────────
reqwest::get(url).await?.text().await?
reqwest::get(url).await?.json::<T>().await?

POST
────────────────────────────────────────────────────────────
client.post(url).json(&data).send().await?
client.post(url).form(&params).send().await?

CLIENT
────────────────────────────────────────────────────────────
Client::builder().timeout(dur).build()?
client.get(url).headers(h).bearer_auth(tok).query(&q)

ERROR HANDLING
────────────────────────────────────────────────────────────
resp.error_for_status()?    4xx/5xx → Error
e.is_timeout()              timeout check
e.status()                  HTTP status code

CONCURRENT
────────────────────────────────────────────────────────────
join_all(futures).await      parallel requests
```

---

## What's Next?

**Lesson 82 — Trait Objects & Object Safety** — Deep dive into `dyn Trait`, vtables, object-safe rules, and plugin architectures.

## Further Reading
- [reqwest docs](https://docs.rs/reqwest/)
- [reqwest examples](https://github.com/seanmonstar/reqwest/tree/master/examples)

---

*reqwest: the internet at your fingertips! 🦀*
