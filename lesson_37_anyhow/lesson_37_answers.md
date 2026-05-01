# ✅ Lesson 37 — Answers: anyhow & error-stack (E5)

---

## Section A

### A1
`anyhow::Result<T>` is `Result<T, anyhow::Error>`. Key differences from `Box<dyn Error>`:
- Built-in **context** chaining via `.context()` / `.with_context()`
- Automatic **backtrace** capture (when `RUST_BACKTRACE=1`)
- **Downcasting** support to recover original error types
- Helper macros: `anyhow!()`, `bail!()`, `ensure!()`
- Better error chain formatting with `.chain()`

### A2
- `.context("msg")` — evaluates the string immediately (eager), fine for static messages
- `.with_context(|| format!("msg {}", var))` — evaluates the closure only when an error occurs (lazy), better for expensive formatting with runtime values

---

## Section B

### A3
```rust
use anyhow::{Context, Result};

fn process() -> Result<()> {
    let config = std::fs::read_to_string("config.txt")
        .context("failed to read config file 'config.txt'")?;
    let port: u16 = config.trim().parse()
        .with_context(|| format!("failed to parse port from config: '{}'", config.trim()))?;
    let _addr = format!("0.0.0.0:{port}");
    println!("Server address: {_addr}");
    Ok(())
}

fn main() {
    if let Err(e) = process() {
        eprintln!("Error: {e:?}");
    }
}
```

### A4
```rust
use anyhow::{ensure, Result};

fn validate_email(email: &str) -> Result<()> {
    let at_count = email.matches('@').count();
    ensure!(at_count == 1, "email must contain exactly one '@', found {at_count}");

    let parts: Vec<&str> = email.splitn(2, '@').collect();
    let (local, domain) = (parts[0], parts[1]);

    ensure!(!local.is_empty(), "email must have content before '@'");
    ensure!(!domain.is_empty(), "email must have content after '@'");
    ensure!(domain.contains('.'), "domain must contain a '.': got '{domain}'");

    Ok(())
}

fn main() {
    let tests = vec!["user@example.com", "@bad.com", "noat", "user@nodot", "multi@@at"];
    for email in tests {
        match validate_email(email) {
            Ok(()) => println!("  ✅ {email}"),
            Err(e) => println!("  ❌ {email}: {e}"),
        }
    }
}
```

### A5
```rust
use anyhow::Result;

#[derive(Debug, thiserror::Error)]
enum ApiError {
    #[error("rate limited")]
    RateLimited,
    #[error("not found: {0}")]
    NotFound(String),
}

fn api_call() -> Result<()> {
    // Wrap typed error in anyhow
    Err(ApiError::NotFound("user/42".into()).into())
}

fn main() {
    if let Err(e) = api_call() {
        // Downcast to recover original type
        if let Some(api_err) = e.downcast_ref::<ApiError>() {
            match api_err {
                ApiError::RateLimited => println!("Wait and retry"),
                ApiError::NotFound(r) => println!("Resource missing: {r}"),
            }
        }
    }
}
```

---

## Section C

### A6
| Scenario | Tool | Reason |
|---|---|---|
| 1. Public CSV library | **thiserror** | Callers need typed errors to pattern match |
| 2. CLI tool | **anyhow** | Application code; just report and exit |
| 3. Learning exercise | **manual** | Understand Display, Error, From traits |
| 4. Internal module | **thiserror** | Other modules may match on errors |
| 5. One-off script | **anyhow** | Maximum convenience, minimal code |

### A7
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | `downcast_ref` can recover the original error type |
| 2 | **True** | `bail!` is syntactic sugar for `return Err(anyhow!(...))` |
| 3 | **False** | Libraries should use `thiserror` for typed errors; `anyhow` is for applications |
| 4 | **False** | `.context()` wraps the error, preserving the original in the chain |
| 5 | **False** | `ensure!` returns `Err`, it does not panic |

---

## 🏆 Lesson 37 Complete!

✅ anyhow::Result and anyhow::Error  
✅ Context and with_context  
✅ bail! and ensure! macros  
✅ Downcasting errors  
✅ anyhow vs thiserror decision framework  
✅ error-stack overview  

**Next up:** [Lesson 38 — Traits: Definition & Implementation](../lesson_38_traits/lesson_38_traits.md) 🦀
