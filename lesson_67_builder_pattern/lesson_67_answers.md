# ✅ Lesson 67 — Answers: Builder Pattern (DP1)

---

## Section A

### A1
1. **Readability** — named setter methods are self-documenting vs positional args
2. **Optional fields** — only set what you need; defaults for the rest
3. **Validation** — `build()` can validate before construction
4. **Order-independent** — set fields in any order

### A2
- `self` (consuming) — each call takes ownership; the builder can't be reused. Simpler, no lifetime issues.
- `&mut self` (borrowing) — the builder is borrowed; can be reused or modified after `build()`. Useful for building multiple similar objects.

---

## Section B

### A3
```rust
#[derive(Debug)]
struct DatabaseConfig {
    host: String, port: u16, database: String,
    username: Option<String>, password: Option<String>,
    pool_size: usize, timeout_ms: u64,
}

struct DatabaseConfigBuilder {
    host: Option<String>, port: Option<u16>, database: Option<String>,
    username: Option<String>, password: Option<String>,
    pool_size: usize, timeout_ms: u64,
}

impl DatabaseConfigBuilder {
    fn new() -> Self {
        DatabaseConfigBuilder {
            host: None, port: None, database: None,
            username: None, password: None,
            pool_size: 5, timeout_ms: 5000,
        }
    }
    fn host(mut self, h: &str) -> Self { self.host = Some(h.into()); self }
    fn port(mut self, p: u16) -> Self { self.port = Some(p); self }
    fn database(mut self, d: &str) -> Self { self.database = Some(d.into()); self }
    fn username(mut self, u: &str) -> Self { self.username = Some(u.into()); self }
    fn password(mut self, p: &str) -> Self { self.password = Some(p.into()); self }
    fn pool_size(mut self, n: usize) -> Self { self.pool_size = n; self }
    fn timeout_ms(mut self, t: u64) -> Self { self.timeout_ms = t; self }

    fn build(self) -> Result<DatabaseConfig, String> {
        let host = self.host.ok_or("host is required")?;
        let port = self.port.ok_or("port is required")?;
        let database = self.database.ok_or("database is required")?;
        if port == 0 { return Err("port must be > 0".into()); }
        if self.pool_size == 0 { return Err("pool_size must be > 0".into()); }
        Ok(DatabaseConfig {
            host, port, database,
            username: self.username, password: self.password,
            pool_size: self.pool_size, timeout_ms: self.timeout_ms,
        })
    }
}

fn main() {
    let config = DatabaseConfigBuilder::new()
        .host("localhost").port(5432).database("myapp")
        .username("admin").pool_size(10)
        .build().unwrap();
    println!("{:#?}", config);
}
```

### A4
```rust
use std::marker::PhantomData;
struct Missing; struct Set;

struct UserBuilder<N, E> {
    name: String, email: String, age: u32,
    _n: PhantomData<N>, _e: PhantomData<E>,
}

impl UserBuilder<Missing, Missing> {
    fn new() -> Self {
        UserBuilder { name: String::new(), email: String::new(), age: 0,
            _n: PhantomData, _e: PhantomData }
    }
}

impl<E> UserBuilder<Missing, E> {
    fn name(self, n: &str) -> UserBuilder<Set, E> {
        UserBuilder { name: n.into(), email: self.email, age: self.age,
            _n: PhantomData, _e: PhantomData }
    }
}

impl<N> UserBuilder<N, Missing> {
    fn email(self, e: &str) -> UserBuilder<N, Set> {
        UserBuilder { name: self.name, email: e.into(), age: self.age,
            _n: PhantomData, _e: PhantomData }
    }
}

impl<N, E> UserBuilder<N, E> {
    fn age(mut self, a: u32) -> Self { self.age = a; self }
}

impl UserBuilder<Set, Set> {
    fn build(self) -> String { format!("{} <{}> age {}", self.name, self.email, self.age) }
}

fn main() {
    let user = UserBuilder::new().name("Alice").email("a@b.com").age(30).build();
    println!("{user}");
    // UserBuilder::new().name("Alice").build();  // ❌ compile error
}
```

---

## Section C

### A5
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | Builder shines with many optional/defaulted fields |
| 2 | **True** | `mut self` takes ownership; returns Self |
| 3 | **False** | Type-safe builders catch missing fields at **compile time** |
| 4 | **True** | `derive_builder` is a proc-macro that generates builder boilerplate |
| 5 | **True** | `build() -> Result<T, E>` enables validation |

---

## 🏆 Lesson 67 Complete!

**Next up:** [Lesson 68 — Design Patterns: State & Strategy](../lesson_68_state_strategy/lesson_68_state_strategy.md) 🦀
