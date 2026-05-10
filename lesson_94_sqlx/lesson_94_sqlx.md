# 📘 Lesson 94 — Database with SQLx (RW6)

> **Series:** Rust From Zero · Advanced Level (Gap Fill)  
> **Roadmap ID:** RW6 · Category: 🌍 Real World  
> **Previous:** [Lesson 93 — Web Server with Axum](../lesson_93_axum/lesson_93_axum.md)  
> **Next:** [Lesson 95 — Capstone: CLI SQL Database](../lesson_95_capstone_db/lesson_95_capstone_db.md)  
> **Practice:** [Questions](./lesson_94_questions.md) · [Answers](./lesson_94_answers.md)  
> **Practice Task:** Build a user repository with compile-time checked queries

---

## Table of Contents

1. [What Is SQLx?](#1-what-is-sqlx)
2. [Setup](#2-setup)
3. [Connecting to a Database](#3-connecting-to-a-database)
4. [Compile-Time Checked Queries](#4-compile-time-checked-queries)
5. [CRUD Operations](#5-crud-operations)
6. [Migrations](#6-migrations)
7. [Connection Pools](#7-connection-pools)
8. [Transactions](#8-transactions)
9. [Integrating with Axum](#9-integrating-with-axum)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. What Is SQLx?

SQLx is an **async, compile-time checked** SQL toolkit:

| Feature | SQLx | Other ORMs |
|---|---|---|
| SQL queries | ✅ Raw SQL | Generated SQL |
| Compile-time check | ✅ `sqlx::query!` | ❌ Runtime errors |
| Async | ✅ Native | Varies |
| ORM layer | ❌ Not an ORM | ✅ Full ORM |
| Databases | Postgres, MySQL, SQLite | Varies |

---

## 2. Setup

```toml
[dependencies]
sqlx = { version = "0.8", features = ["runtime-tokio", "sqlite", "macros"] }
tokio = { version = "1", features = ["full"] }
serde = { version = "1", features = ["derive"] }
```

For PostgreSQL:
```toml
sqlx = { version = "0.8", features = ["runtime-tokio", "postgres", "macros"] }
```

```bash
# Install SQLx CLI for migrations
cargo install sqlx-cli --features sqlite

# Set database URL
# SQLite:
export DATABASE_URL="sqlite:my_app.db"

# PostgreSQL:
# export DATABASE_URL="postgres://user:pass@localhost/mydb"
```

---

## 3. Connecting to a Database

```rust
use sqlx::sqlite::SqlitePoolOptions;

#[tokio::main]
async fn main() -> Result<(), sqlx::Error> {
    // SQLite — creates file if it doesn't exist
    let pool = SqlitePoolOptions::new()
        .max_connections(5)
        .connect("sqlite:my_app.db?mode=rwc")
        .await?;

    // Create table
    sqlx::query(
        "CREATE TABLE IF NOT EXISTS users (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            email TEXT NOT NULL UNIQUE,
            created_at DATETIME DEFAULT CURRENT_TIMESTAMP
        )"
    )
    .execute(&pool)
    .await?;

    println!("✅ Database connected and table created");
    Ok(())
}
```

---

## 4. Compile-Time Checked Queries

The `query!` macro verifies SQL at compile time against your database:

```rust
use sqlx::FromRow;

#[derive(Debug, FromRow)]
struct User {
    id: i64,
    name: String,
    email: String,
}

async fn demo(pool: &sqlx::SqlitePool) -> Result<(), sqlx::Error> {
    // Compile-time checked — if column doesn't exist, won't compile!
    let users = sqlx::query_as!(User, "SELECT id, name, email FROM users")
        .fetch_all(pool)
        .await?;

    for user in &users {
        println!("#{}: {} ({})", user.id, user.name, user.email);
    }

    // query! returns anonymous structs
    let row = sqlx::query!("SELECT COUNT(*) as count FROM users")
        .fetch_one(pool)
        .await?;
    println!("Total users: {}", row.count);

    Ok(())
}
```

### Without compile-time checking (runtime):

```rust
// query_as — maps to struct at runtime (no compile-time check)
let users = sqlx::query_as::<_, User>("SELECT id, name, email FROM users")
    .fetch_all(pool)
    .await?;

// query — returns Row objects
let row = sqlx::query("SELECT name FROM users WHERE id = ?")
    .bind(1)
    .fetch_one(pool)
    .await?;

let name: String = row.get("name");
```

---

## 5. CRUD Operations

```rust
use sqlx::SqlitePool;

#[derive(Debug, sqlx::FromRow, serde::Serialize)]
struct User {
    id: i64,
    name: String,
    email: String,
}

// CREATE
async fn create_user(pool: &SqlitePool, name: &str, email: &str) -> Result<i64, sqlx::Error> {
    let result = sqlx::query("INSERT INTO users (name, email) VALUES (?, ?)")
        .bind(name)
        .bind(email)
        .execute(pool)
        .await?;

    Ok(result.last_insert_rowid())
}

// READ (one)
async fn get_user(pool: &SqlitePool, id: i64) -> Result<Option<User>, sqlx::Error> {
    sqlx::query_as::<_, User>("SELECT id, name, email FROM users WHERE id = ?")
        .bind(id)
        .fetch_optional(pool)
        .await
}

// READ (all)
async fn list_users(pool: &SqlitePool) -> Result<Vec<User>, sqlx::Error> {
    sqlx::query_as::<_, User>("SELECT id, name, email FROM users ORDER BY id")
        .fetch_all(pool)
        .await
}

// UPDATE
async fn update_user(pool: &SqlitePool, id: i64, name: &str) -> Result<bool, sqlx::Error> {
    let result = sqlx::query("UPDATE users SET name = ? WHERE id = ?")
        .bind(name)
        .bind(id)
        .execute(pool)
        .await?;

    Ok(result.rows_affected() > 0)
}

// DELETE
async fn delete_user(pool: &SqlitePool, id: i64) -> Result<bool, sqlx::Error> {
    let result = sqlx::query("DELETE FROM users WHERE id = ?")
        .bind(id)
        .execute(pool)
        .await?;

    Ok(result.rows_affected() > 0)
}

#[tokio::main]
async fn main() -> Result<(), sqlx::Error> {
    let pool = SqlitePool::connect("sqlite:demo.db?mode=rwc").await?;
    sqlx::query("CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY AUTOINCREMENT, name TEXT NOT NULL, email TEXT NOT NULL UNIQUE)")
        .execute(&pool).await?;

    let id = create_user(&pool, "Alice", "alice@example.com").await?;
    println!("Created user #{id}");

    if let Some(user) = get_user(&pool, id).await? {
        println!("Found: {:?}", user);
    }

    update_user(&pool, id, "Alice Updated").await?;

    for user in list_users(&pool).await? {
        println!("  #{}: {}", user.id, user.name);
    }

    delete_user(&pool, id).await?;
    println!("Deleted user #{id}");

    Ok(())
}
```

---

## 6. Migrations

```bash
# Create migrations directory
sqlx migrate add create_users

# This creates: migrations/20240101000000_create_users.sql
```

```sql
-- migrations/20240101000000_create_users.sql
CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    email TEXT NOT NULL UNIQUE,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

```bash
# Run migrations
sqlx migrate run

# Revert last migration
sqlx migrate revert
```

### Run migrations from code:

```rust
#[tokio::main]
async fn main() -> Result<(), sqlx::Error> {
    let pool = sqlx::SqlitePool::connect("sqlite:app.db?mode=rwc").await?;

    // Run embedded migrations at startup
    sqlx::migrate!("./migrations")
        .run(&pool)
        .await?;

    println!("✅ Migrations complete");
    Ok(())
}
```

---

## 7. Connection Pools

```rust
use sqlx::sqlite::SqlitePoolOptions;
use std::time::Duration;

#[tokio::main]
async fn main() -> Result<(), sqlx::Error> {
    let pool = SqlitePoolOptions::new()
        .max_connections(10)               // max pool size
        .min_connections(2)                // keep 2 alive
        .acquire_timeout(Duration::from_secs(5))  // wait for connection
        .idle_timeout(Duration::from_secs(600))   // close idle after 10min
        .connect("sqlite:app.db?mode=rwc")
        .await?;

    // Pool is Clone — share across handlers
    let pool2 = pool.clone();
    tokio::spawn(async move {
        let count: (i64,) = sqlx::query_as("SELECT COUNT(*) FROM users")
            .fetch_one(&pool2).await.unwrap();
        println!("Users: {}", count.0);
    });

    Ok(())
}
```

---

## 8. Transactions

```rust
async fn transfer(
    pool: &sqlx::SqlitePool,
    from: i64,
    to: i64,
    amount: f64,
) -> Result<(), sqlx::Error> {
    let mut tx = pool.begin().await?;

    sqlx::query("UPDATE accounts SET balance = balance - ? WHERE id = ?")
        .bind(amount).bind(from)
        .execute(&mut *tx).await?;

    sqlx::query("UPDATE accounts SET balance = balance + ? WHERE id = ?")
        .bind(amount).bind(to)
        .execute(&mut *tx).await?;

    // Commit — if any query failed, we never get here (auto-rollback on drop)
    tx.commit().await?;
    println!("✅ Transferred {amount} from #{from} to #{to}");
    Ok(())
}
```

---

## 9. Integrating with Axum

```rust
use axum::{extract::{Path, State}, http::StatusCode, routing::{get, post}, Json, Router};
use sqlx::SqlitePool;
use serde::{Serialize, Deserialize};

#[derive(Serialize, sqlx::FromRow)]
struct User { id: i64, name: String, email: String }

#[derive(Deserialize)]
struct NewUser { name: String, email: String }

async fn list(State(pool): State<SqlitePool>) -> Result<Json<Vec<User>>, StatusCode> {
    sqlx::query_as::<_, User>("SELECT id, name, email FROM users")
        .fetch_all(&pool).await
        .map(Json)
        .map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)
}

async fn create(State(pool): State<SqlitePool>, Json(input): Json<NewUser>) -> Result<(StatusCode, Json<User>), StatusCode> {
    let result = sqlx::query("INSERT INTO users (name, email) VALUES (?, ?)")
        .bind(&input.name).bind(&input.email)
        .execute(&pool).await
        .map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?;

    let user = User { id: result.last_insert_rowid(), name: input.name, email: input.email };
    Ok((StatusCode::CREATED, Json(user)))
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let pool = SqlitePool::connect("sqlite:app.db?mode=rwc").await?;
    sqlx::query("CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY AUTOINCREMENT, name TEXT NOT NULL, email TEXT NOT NULL UNIQUE)")
        .execute(&pool).await?;

    let app = Router::new()
        .route("/users", get(list).post(create))
        .with_state(pool);

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await?;
    println!("🚀 API at http://localhost:3000/users");
    axum::serve(listener, app).await?;
    Ok(())
}
```

---

## 10. Summary Cheat Sheet

```
SETUP
────────────────────────────────────────────────────────────
sqlx = { version = "0.8", features = ["runtime-tokio", "sqlite", "macros"] }
cargo install sqlx-cli --features sqlite

CONNECT
────────────────────────────────────────────────────────────
SqlitePool::connect("sqlite:app.db?mode=rwc").await?
PgPool::connect("postgres://user:pass@host/db").await?

QUERIES
────────────────────────────────────────────────────────────
sqlx::query("SQL").bind(val).execute(&pool)        execute
sqlx::query("SQL").bind(val).fetch_one(&pool)      one row
sqlx::query("SQL").bind(val).fetch_all(&pool)      all rows
sqlx::query("SQL").bind(val).fetch_optional(&pool) Option<Row>
sqlx::query_as::<_, T>("SQL").fetch_all(&pool)     typed

COMPILE-TIME
────────────────────────────────────────────────────────────
sqlx::query!("SELECT ...")           anonymous struct
sqlx::query_as!(Type, "SELECT ...")  named struct

MIGRATIONS
────────────────────────────────────────────────────────────
sqlx migrate add name               create migration
sqlx migrate run                     apply migrations
sqlx::migrate!("./migrations").run(&pool)   from code

TRANSACTIONS
────────────────────────────────────────────────────────────
let mut tx = pool.begin().await?;
sqlx::query("...").execute(&mut *tx).await?;
tx.commit().await?;
```

---

## What's Next?

**Lesson 95 — Capstone: CLI SQL Database** — Build a simple SQL database engine from scratch as a capstone project.

## Further Reading
- [SQLx docs](https://docs.rs/sqlx/)
- [SQLx GitHub](https://github.com/launchbadge/sqlx)

---

*SQLx: SQL that the compiler verifies for you! 🦀*
