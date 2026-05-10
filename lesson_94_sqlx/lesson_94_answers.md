# ✅ Lesson 94 — Answers: Database with SQLx (RW6)

---

## Section A

### A1
- `sqlx::query!()` checks the SQL **at compile time** against an actual database (set via `DATABASE_URL`). Typos in column names or type mismatches become compile errors.
- `sqlx::query()` is a runtime-only query — SQL errors only surface when the code runs.

### A2
The transaction is **automatically rolled back** when dropped without `.commit()`. This is RAII in action — the `Transaction` type implements `Drop` which sends a ROLLBACK to the database.

---

## Section B

### A3
```rust
use sqlx::SqlitePool;

#[derive(Debug, sqlx::FromRow)]
struct User { id: i64, name: String, email: String }

async fn create_user(pool: &SqlitePool, name: &str, email: &str) -> Result<i64, sqlx::Error> {
    let r = sqlx::query("INSERT INTO users (name,email) VALUES (?,?)").bind(name).bind(email).execute(pool).await?;
    Ok(r.last_insert_rowid())
}
async fn get_user(pool: &SqlitePool, id: i64) -> Result<Option<User>, sqlx::Error> {
    sqlx::query_as::<_,User>("SELECT id,name,email FROM users WHERE id=?").bind(id).fetch_optional(pool).await
}
async fn list_users(pool: &SqlitePool) -> Result<Vec<User>, sqlx::Error> {
    sqlx::query_as::<_,User>("SELECT id,name,email FROM users").fetch_all(pool).await
}
async fn update_user(pool: &SqlitePool, id: i64, name: &str) -> Result<bool, sqlx::Error> {
    let r = sqlx::query("UPDATE users SET name=? WHERE id=?").bind(name).bind(id).execute(pool).await?;
    Ok(r.rows_affected() > 0)
}
async fn delete_user(pool: &SqlitePool, id: i64) -> Result<bool, sqlx::Error> {
    let r = sqlx::query("DELETE FROM users WHERE id=?").bind(id).execute(pool).await?;
    Ok(r.rows_affected() > 0)
}
```

### A4
```rust
use axum::{extract::State, http::StatusCode, routing::{get, post}, Json, Router};
use sqlx::SqlitePool;
use serde::{Serialize, Deserialize};

#[derive(Serialize, sqlx::FromRow)]
struct User { id: i64, name: String, email: String }
#[derive(Deserialize)]
struct NewUser { name: String, email: String }

async fn list(State(p): State<SqlitePool>) -> Result<Json<Vec<User>>, StatusCode> {
    sqlx::query_as::<_,User>("SELECT id,name,email FROM users").fetch_all(&p).await.map(Json).map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)
}
async fn create(State(p): State<SqlitePool>, Json(i): Json<NewUser>) -> Result<(StatusCode, Json<User>), StatusCode> {
    let r = sqlx::query("INSERT INTO users (name,email) VALUES (?,?)").bind(&i.name).bind(&i.email).execute(&p).await.map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?;
    Ok((StatusCode::CREATED, Json(User { id: r.last_insert_rowid(), name: i.name, email: i.email })))
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let pool = SqlitePool::connect("sqlite:app.db?mode=rwc").await?;
    sqlx::query("CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY AUTOINCREMENT, name TEXT NOT NULL, email TEXT NOT NULL)").execute(&pool).await?;
    let app = Router::new().route("/users", get(list).post(create)).with_state(pool);
    let l = tokio::net::TcpListener::bind("0.0.0.0:3000").await?;
    axum::serve(l, app).await?; Ok(())
}
```

---

## Section C

### A5
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | Verifies SQL against a real database at compile time |
| 2 | **False** | SQLx is NOT an ORM — it's a SQL toolkit (raw SQL, not query builders) |
| 3 | **True** | `FromRow` maps column names to struct fields |
| 4 | **True** | `begin()` returns a `Transaction` with RAII rollback |
| 5 | **True** | All three databases are supported |
| 6 | **False** | Pools are dropped automatically; close is handled by Drop |

---

## 🏆 Lesson 94 Complete!

**Next up:** [Lesson 95 — Capstone: CLI SQL Database](../lesson_95_capstone_db/lesson_95_capstone_db.md) 🦀
