# ✅ Lesson 81 — Answers: HTTP Client with reqwest (RW4)

---

## Section A

### A1
`Client` maintains a **connection pool**. Reusing it avoids re-establishing TCP/TLS connections for each request, significantly improving performance. It also applies shared configuration (timeouts, headers, user-agent).

### A2
`error_for_status()` checks the HTTP status code. If it's 4xx or 5xx, it converts the response into a `reqwest::Error`. Otherwise, it returns the response unchanged. This prevents silently processing error responses.

---

## Section B

### A3
```rust
use serde::Deserialize;

#[derive(Debug, Deserialize)]
struct User {
    id: u32,
    name: String,
    email: String,
    phone: String,
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let users: Vec<User> = reqwest::get("https://jsonplaceholder.typicode.com/users")
        .await?
        .json()
        .await?;

    for u in &users {
        println!("#{}: {} ({}) ☎ {}", u.id, u.name, u.email, u.phone);
    }
    Ok(())
}
```

### A4
```rust
use futures::future::join_all;
use std::time::Instant;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = reqwest::Client::new();
    let urls: Vec<String> = (1..=5)
        .map(|i| format!("https://jsonplaceholder.typicode.com/posts/{i}"))
        .collect();

    let start = Instant::now();
    let futs: Vec<_> = urls.iter().map(|url| {
        let c = client.clone();
        let u = url.clone();
        async move { c.get(&u).send().await?.text().await }
    }).collect();

    let results = join_all(futs).await;
    for (url, res) in urls.iter().zip(results.iter()) {
        match res {
            Ok(body) => println!("✅ {url} → {} bytes", body.len()),
            Err(e) => println!("❌ {url} → {e}"),
        }
    }
    println!("Total: {:?}", start.elapsed());
    Ok(())
}
```

---

## Section C

### A5
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | Convenience functions create a one-shot client internally |
| 2 | **True** | `.json::<T>()` uses serde to deserialize the body |
| 3 | **False** | reqwest is **async by default**; blocking requires the `blocking` feature |
| 4 | **True** | Timeout applies to each request made with that client |
| 5 | **True** | Adds `Authorization: Bearer <token>` header |
| 6 | **True** | `.json(&data)` serializes with serde and sets Content-Type |

---

## 🏆 Lesson 81 Complete!

**Next up:** [Lesson 82 — Trait Objects & Object Safety](../lesson_82_trait_objects/lesson_82_trait_objects.md) 🦀
