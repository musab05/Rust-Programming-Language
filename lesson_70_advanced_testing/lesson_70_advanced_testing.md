# 📘 Lesson 70 — Testing: Mocking & Property-Based (TE2)

> **Series:** Rust From Zero · Advanced Level  
> **Roadmap ID:** TE2 · Category: 🧪 Testing  
> **Previous:** [Lesson 69 — Unit & Integration Tests](../lesson_69_testing/lesson_69_testing.md)  
> **Next:** [Lesson 71 — Weak\<T\>](../lesson_71_weak/lesson_71_weak.md)  
> **Practice:** [Questions](./lesson_70_questions.md) · [Answers](./lesson_70_answers.md)  
> **Practice Task:** Mock a database trait and write property-based tests

---

## Table of Contents

1. [Why Mocking?](#1-why-mocking)
2. [Manual Mocking with Traits](#2-manual-mocking-with-traits)
3. [mockall Crate](#3-mockall-crate)
4. [Mocking Patterns](#4-mocking-patterns)
5. [Property-Based Testing](#5-property-based-testing)
6. [proptest Crate](#6-proptest-crate)
7. [Custom Strategies](#7-custom-strategies)
8. [Test Fixtures](#8-test-fixtures)
9. [Doc Tests](#9-doc-tests)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. Why Mocking?

Replace real dependencies (database, API, filesystem) with controlled fakes:

```
Production:  App → Database → PostgreSQL
Testing:     App → MockDB   → in-memory HashMap
```

---

## 2. Manual Mocking with Traits

Define a trait for the dependency, then implement a mock:

```rust
trait UserRepository {
    fn find_by_id(&self, id: u32) -> Option<String>;
    fn save(&mut self, id: u32, name: &str) -> Result<(), String>;
}

// Real implementation
struct PostgresRepo { /* connection pool */ }
impl UserRepository for PostgresRepo {
    fn find_by_id(&self, _id: u32) -> Option<String> { todo!("query DB") }
    fn save(&mut self, _id: u32, _name: &str) -> Result<(), String> { todo!("insert") }
}

// Mock implementation
struct MockRepo {
    users: std::collections::HashMap<u32, String>,
}

impl MockRepo {
    fn new() -> Self { MockRepo { users: std::collections::HashMap::new() } }
    fn with_user(mut self, id: u32, name: &str) -> Self {
        self.users.insert(id, name.into()); self
    }
}

impl UserRepository for MockRepo {
    fn find_by_id(&self, id: u32) -> Option<String> { self.users.get(&id).cloned() }
    fn save(&mut self, id: u32, name: &str) -> Result<(), String> {
        self.users.insert(id, name.into()); Ok(())
    }
}

// Service that depends on the trait (not the concrete type)
struct UserService<R: UserRepository> { repo: R }

impl<R: UserRepository> UserService<R> {
    fn new(repo: R) -> Self { UserService { repo } }

    fn greet(&self, id: u32) -> String {
        match self.repo.find_by_id(id) {
            Some(name) => format!("Hello, {name}!"),
            None => "User not found".into(),
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_greet_found() {
        let repo = MockRepo::new().with_user(1, "Alice");
        let service = UserService::new(repo);
        assert_eq!(service.greet(1), "Hello, Alice!");
    }

    #[test]
    fn test_greet_not_found() {
        let repo = MockRepo::new();
        let service = UserService::new(repo);
        assert_eq!(service.greet(99), "User not found");
    }
}
```

---

## 3. mockall Crate

Auto-generate mocks from traits:

```toml
[dev-dependencies]
mockall = "0.13"
```

```rust
use mockall::automock;

#[automock]  // generates MockWeatherService
trait WeatherService {
    fn temperature(&self, city: &str) -> Result<f64, String>;
    fn humidity(&self, city: &str) -> Result<f64, String>;
}

struct WeatherApp<W: WeatherService> { service: W }

impl<W: WeatherService> WeatherApp<W> {
    fn report(&self, city: &str) -> String {
        match (self.service.temperature(city), self.service.humidity(city)) {
            (Ok(temp), Ok(hum)) => format!("{city}: {temp:.1}°C, {hum:.0}% humidity"),
            _ => format!("{city}: data unavailable"),
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_weather_report() {
        let mut mock = MockWeatherService::new();
        mock.expect_temperature()
            .with(mockall::predicate::eq("London"))
            .returning(|_| Ok(18.5));
        mock.expect_humidity()
            .with(mockall::predicate::eq("London"))
            .returning(|_| Ok(65.0));

        let app = WeatherApp { service: mock };
        assert_eq!(app.report("London"), "London: 18.5°C, 65% humidity");
    }

    #[test]
    fn test_weather_error() {
        let mut mock = MockWeatherService::new();
        mock.expect_temperature().returning(|_| Err("timeout".into()));
        mock.expect_humidity().returning(|_| Ok(50.0));

        let app = WeatherApp { service: mock };
        assert_eq!(app.report("Paris"), "Paris: data unavailable");
    }
}
```

---

## 4. Mocking Patterns

### Verify call count:

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_called_once() {
        let mut mock = MockWeatherService::new();
        mock.expect_temperature()
            .times(1)  // must be called exactly once
            .returning(|_| Ok(20.0));
        mock.expect_humidity()
            .times(1)
            .returning(|_| Ok(50.0));

        let app = WeatherApp { service: mock };
        app.report("Berlin");
        // mock is dropped here — verifies call counts
    }
}
```

### Sequence (ordered calls):

```rust
use mockall::Sequence;

#[test]
fn test_ordered_calls() {
    let mut mock = MockWeatherService::new();
    let mut seq = Sequence::new();

    mock.expect_temperature()
        .times(1)
        .in_sequence(&mut seq)
        .returning(|_| Ok(20.0));
    mock.expect_humidity()
        .times(1)
        .in_sequence(&mut seq)
        .returning(|_| Ok(50.0));

    let app = WeatherApp { service: mock };
    app.report("Rome");
}
```

---

## 5. Property-Based Testing

Instead of testing specific inputs, test **properties** that should hold for ALL inputs:

```rust
// Traditional test: specific inputs
#[test]
fn test_reverse() {
    assert_eq!(reverse("hello"), "olleh");
    assert_eq!(reverse(""), "");
}

// Property test: for ANY string, reverse(reverse(s)) == s
// Tests with hundreds of random inputs automatically!
```

---

## 6. proptest Crate

```toml
[dev-dependencies]
proptest = "1"
```

```rust
use proptest::prelude::*;

fn reverse(s: &str) -> String {
    s.chars().rev().collect()
}

fn sort_vec(v: &mut Vec<i32>) {
    v.sort();
}

proptest! {
    // Property: reversing twice gives back the original
    #[test]
    fn test_double_reverse(s in "\\PC*") {
        let result = reverse(&reverse(&s));
        assert_eq!(result, s);
    }

    // Property: sorted vec is always sorted
    #[test]
    fn test_sort_is_sorted(mut v in prop::collection::vec(any::<i32>(), 0..100)) {
        sort_vec(&mut v);
        for window in v.windows(2) {
            assert!(window[0] <= window[1]);
        }
    }

    // Property: sorted vec has same length
    #[test]
    fn test_sort_preserves_length(mut v in prop::collection::vec(any::<i32>(), 0..100)) {
        let original_len = v.len();
        sort_vec(&mut v);
        assert_eq!(v.len(), original_len);
    }

    // Property: addition is commutative
    #[test]
    fn test_add_commutative(a in any::<i32>(), b in any::<i32>()) {
        assert_eq!(a.wrapping_add(b), b.wrapping_add(a));
    }
}
```

### Constraining inputs:

```rust
proptest! {
    #[test]
    fn test_positive_sqrt(x in 0.0f64..1000.0) {
        let result = x.sqrt();
        assert!(result >= 0.0);
        assert!((result * result - x).abs() < 0.001);
    }

    #[test]
    fn test_valid_port(port in 1u16..=65535) {
        assert!(port > 0);
        let addr = format!("127.0.0.1:{port}");
        assert!(addr.contains(':'));
    }

    #[test]
    fn test_non_empty_string(s in "[a-zA-Z]{1,50}") {
        assert!(!s.is_empty());
        assert!(s.len() <= 50);
        assert!(s.chars().all(|c| c.is_alphabetic()));
    }
}
```

---

## 7. Custom Strategies

Generate complex test data:

```rust
use proptest::prelude::*;

#[derive(Debug, Clone)]
struct User { name: String, age: u8, email: String }

fn user_strategy() -> impl Strategy<Value = User> {
    (
        "[A-Z][a-z]{2,10}",           // name: capitalized
        18u8..=100,                     // age: adult
        "[a-z]{3,8}@[a-z]{3,5}\\.com", // email pattern
    ).prop_map(|(name, age, email)| User { name, age, email })
}

proptest! {
    #[test]
    fn test_user_valid(user in user_strategy()) {
        assert!(!user.name.is_empty());
        assert!(user.age >= 18);
        assert!(user.email.contains('@'));
    }
}
```

---

## 8. Test Fixtures

Reusable test setup:

```rust
struct TestContext {
    data: Vec<i32>,
    expected_sum: i32,
}

impl TestContext {
    fn small() -> Self { TestContext { data: vec![1, 2, 3], expected_sum: 6 } }
    fn large() -> Self { TestContext { data: (1..=100).collect(), expected_sum: 5050 } }
    fn empty() -> Self { TestContext { data: vec![], expected_sum: 0 } }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_sum_small() {
        let ctx = TestContext::small();
        assert_eq!(ctx.data.iter().sum::<i32>(), ctx.expected_sum);
    }

    #[test]
    fn test_sum_large() {
        let ctx = TestContext::large();
        assert_eq!(ctx.data.iter().sum::<i32>(), ctx.expected_sum);
    }

    #[test]
    fn test_sum_empty() {
        let ctx = TestContext::empty();
        assert_eq!(ctx.data.iter().sum::<i32>(), ctx.expected_sum);
    }
}
```

---

## 9. Doc Tests

Code examples in documentation are tested automatically:

```rust
/// Adds two numbers together.
///
/// # Examples
///
/// ```
/// let result = my_crate::add(2, 3);
/// assert_eq!(result, 5);
/// ```
///
/// ```
/// let result = my_crate::add(-1, 1);
/// assert_eq!(result, 0);
/// ```
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}

/// Divides two numbers.
///
/// # Errors
///
/// Returns `Err` if `b` is zero.
///
/// ```
/// assert_eq!(my_crate::divide(10, 2), Ok(5));
/// assert!(my_crate::divide(10, 0).is_err());
/// ```
pub fn divide(a: i32, b: i32) -> Result<i32, String> {
    if b == 0 { Err("division by zero".into()) } else { Ok(a / b) }
}
```

```bash
cargo test --doc  # run only doc tests
```

---

## 10. Summary Cheat Sheet

```
MANUAL MOCKING
────────────────────────────────────────────────────────────
trait Dependency { ... }
struct MockDep { ... }           hand-written mock
impl Dependency for MockDep      controlled responses

MOCKALL
────────────────────────────────────────────────────────────
#[automock] trait T { ... }      auto-generates MockT
mock.expect_method()             set expectations
    .returning(|_| value)        control return value
    .times(n)                    verify call count

PROPERTY-BASED (proptest)
────────────────────────────────────────────────────────────
proptest! {
    #[test]
    fn prop(x in any::<i32>()) { assert!(property(x)); }
}
Tests hundreds of random inputs automatically

DOC TESTS
────────────────────────────────────────────────────────────
/// ```
/// assert_eq!(my_fn(1), 2);
/// ```
cargo test --doc

TESTING PYRAMID
────────────────────────────────────────────────────────────
Doc tests        → API examples stay correct
Unit tests       → individual functions
Integration tests→ public API, cross-module
Property tests   → invariants hold for all inputs
```

---

## 🎉 70 Lessons Complete!

You've now covered an **exceptional breadth** of Rust:

| Phase | Lessons | Topics |
|-------|---------|--------|
| 🟢 Beginner | 1–30 | Ownership, structs, enums, pattern matching |
| 🟡 Intermediate | 31–50 | Iterators, errors, traits, generics, closures |
| 🔵 Advanced | 51–70 | Smart pointers, concurrency, async, macros, FFI, design patterns, testing |

**Coming next**: Serde, Web Development (Actix/Axum), Databases, and project-based learning!

## Further Reading
- [mockall docs](https://docs.rs/mockall/)
- [proptest Book](https://proptest-rs.github.io/proptest/intro.html)
- [Rust Book — Ch 11: Testing](https://doc.rust-lang.org/book/ch11-00-testing.html)

---

*Testing: confidence in every commit! 🦀*
