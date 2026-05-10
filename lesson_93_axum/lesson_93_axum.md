# 📘 Lesson 93 — Web Server with Axum (RW5)

> **Series:** Rust From Zero · Advanced Level (Gap Fill)  
> **Roadmap ID:** RW5 · Category: 🌍 Real World  
> **Previous:** [Lesson 92 — Observer / Event System](../lesson_92_observer/lesson_92_observer.md)  
> **Next:** [Lesson 94 — Database with SQLx](../lesson_94_sqlx/lesson_94_sqlx.md)  
> **Practice:** [Questions](./lesson_93_questions.md) · [Answers](./lesson_93_answers.md)  
> **Practice Task:** Build a REST API with CRUD endpoints for a todo app

---

## Table of Contents

1. [Why Axum?](#1-why-axum)
2. [Setup](#2-setup)
3. [Hello World Server](#3-hello-world-server)
4. [Routing](#4-routing)
5. [Extractors](#5-extractors)
6. [JSON Requests and Responses](#6-json-requests-and-responses)
7. [Shared State](#7-shared-state)
8. [Middleware and Layers](#8-middleware-and-layers)
9. [Error Handling](#9-error-handling)
10. [Full CRUD Example](#10-full-crud-example)
11. [Summary Cheat Sheet](#11-summary-cheat-sheet)

---

## 1. Why Axum?

Axum is the most popular async web framework in Rust, built by the Tokio team:

- Built on `tower` (middleware ecosystem)
- Type-safe extractors — no runtime failures
- Macro-free routing
- First-class async support
- Great performance

---

## 2. Setup

```toml
[dependencies]
axum = "0.7"
tokio = { version = "1", features = ["full"] }
serde = { version = "1", features = ["derive"] }
serde_json = "1"
tower-http = { version = "0.5", features = ["cors", "trace"] }
tracing = "0.1"
tracing-subscriber = "0.3"
```

---

## 3. Hello World Server

```rust
use axum::{routing::get, Router};

async fn hello() -> &'static str {
    "Hello, World! 🦀"
}

async fn health() -> &'static str {
    "OK"
}

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/", get(hello))
        .route("/health", get(health));

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    println!("🚀 Server running on http://localhost:3000");
    axum::serve(listener, app).await.unwrap();
}
```

```bash
$ cargo run
# Visit http://localhost:3000
# → "Hello, World! 🦀"
```

---

## 4. Routing

```rust
use axum::{routing::{get, post, put, delete}, Router};

async fn list_users() -> &'static str { "List all users" }
async fn get_user() -> &'static str { "Get one user" }
async fn create_user() -> &'static str { "Create user" }
async fn update_user() -> &'static str { "Update user" }
async fn delete_user() -> &'static str { "Delete user" }

fn user_routes() -> Router {
    Router::new()
        .route("/users", get(list_users).post(create_user))
        .route("/users/:id", get(get_user).put(update_user).delete(delete_user))
}

fn api_routes() -> Router {
    Router::new()
        .nest("/api/v1", user_routes())
        .route("/health", get(|| async { "OK" }))
}

#[tokio::main]
async fn main() {
    let app = api_routes();
    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    axum::serve(listener, app).await.unwrap();
}
```

---

## 5. Extractors

Extractors pull data from requests — type-safe at compile time:

```rust
use axum::{
    extract::{Path, Query, State},
    Json,
};
use serde::Deserialize;

// Path parameters: /users/42
async fn get_user(Path(id): Path<u32>) -> String {
    format!("User #{id}")
}

// Multiple path params: /users/42/posts/7
async fn get_post(Path((user_id, post_id)): Path<(u32, u32)>) -> String {
    format!("User #{user_id}, Post #{post_id}")
}

// Query parameters: /search?q=rust&page=2
#[derive(Deserialize)]
struct SearchParams {
    q: String,
    #[serde(default = "default_page")]
    page: u32,
}
fn default_page() -> u32 { 1 }

async fn search(Query(params): Query<SearchParams>) -> String {
    format!("Search: '{}' page {}", params.q, params.page)
}

// JSON body
#[derive(Deserialize)]
struct CreateUser {
    name: String,
    email: String,
}

async fn create_user(Json(payload): Json<CreateUser>) -> String {
    format!("Created: {} ({})", payload.name, payload.email)
}
```

---

## 6. JSON Requests and Responses

```rust
use axum::{Json, http::StatusCode};
use serde::{Serialize, Deserialize};

#[derive(Serialize)]
struct ApiResponse<T: Serialize> {
    success: bool,
    data: Option<T>,
    #[serde(skip_serializing_if = "Option::is_none")]
    error: Option<String>,
}

#[derive(Serialize, Deserialize)]
struct User {
    id: u32,
    name: String,
    email: String,
}

async fn get_user(axum::extract::Path(id): axum::extract::Path<u32>) -> Json<ApiResponse<User>> {
    let user = User {
        id,
        name: "Alice".into(),
        email: "alice@example.com".into(),
    };
    Json(ApiResponse { success: true, data: Some(user), error: None })
}

async fn create_user(
    Json(input): Json<User>,
) -> (StatusCode, Json<ApiResponse<User>>) {
    // In real app: save to database
    (
        StatusCode::CREATED,
        Json(ApiResponse { success: true, data: Some(input), error: None }),
    )
}
```

---

## 7. Shared State

```rust
use axum::{extract::State, routing::{get, post}, Json, Router};
use serde::{Serialize, Deserialize};
use std::sync::{Arc, Mutex};

#[derive(Clone, Serialize, Deserialize)]
struct Todo {
    id: u32,
    title: String,
    done: bool,
}

type AppState = Arc<Mutex<Vec<Todo>>>;

async fn list_todos(State(state): State<AppState>) -> Json<Vec<Todo>> {
    let todos = state.lock().unwrap();
    Json(todos.clone())
}

#[derive(Deserialize)]
struct NewTodo { title: String }

async fn add_todo(
    State(state): State<AppState>,
    Json(input): Json<NewTodo>,
) -> Json<Todo> {
    let mut todos = state.lock().unwrap();
    let todo = Todo {
        id: todos.len() as u32 + 1,
        title: input.title,
        done: false,
    };
    todos.push(todo.clone());
    Json(todo)
}

#[tokio::main]
async fn main() {
    let state: AppState = Arc::new(Mutex::new(vec![]));

    let app = Router::new()
        .route("/todos", get(list_todos).post(add_todo))
        .with_state(state);

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    println!("🚀 Todo API on http://localhost:3000/todos");
    axum::serve(listener, app).await.unwrap();
}
```

---

## 8. Middleware and Layers

```rust
use axum::{Router, routing::get, middleware, http::Request, extract::Next, response::Response};
use tower_http::cors::{CorsLayer, Any};
use std::time::Instant;

// Custom middleware — timing
async fn timing_middleware(
    req: Request<axum::body::Body>,
    next: Next,
) -> Response {
    let method = req.method().clone();
    let uri = req.uri().clone();
    let start = Instant::now();

    let response = next.run(req).await;

    println!("{method} {uri} → {} in {:?}", response.status(), start.elapsed());
    response
}

#[tokio::main]
async fn main() {
    let cors = CorsLayer::new()
        .allow_origin(Any)
        .allow_methods(Any)
        .allow_headers(Any);

    let app = Router::new()
        .route("/", get(|| async { "Hello!" }))
        .layer(middleware::from_fn(timing_middleware))
        .layer(cors);

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    axum::serve(listener, app).await.unwrap();
}
```

---

## 9. Error Handling

```rust
use axum::{http::StatusCode, response::IntoResponse, Json};
use serde_json::json;

enum AppError {
    NotFound(String),
    BadRequest(String),
    Internal(String),
}

impl IntoResponse for AppError {
    fn into_response(self) -> axum::response::Response {
        let (status, msg) = match self {
            AppError::NotFound(m) => (StatusCode::NOT_FOUND, m),
            AppError::BadRequest(m) => (StatusCode::BAD_REQUEST, m),
            AppError::Internal(m) => (StatusCode::INTERNAL_SERVER_ERROR, m),
        };
        (status, Json(json!({ "error": msg }))).into_response()
    }
}

async fn get_user(
    axum::extract::Path(id): axum::extract::Path<u32>,
) -> Result<Json<serde_json::Value>, AppError> {
    if id == 0 {
        return Err(AppError::BadRequest("ID must be > 0".into()));
    }
    if id > 100 {
        return Err(AppError::NotFound(format!("User {id} not found")));
    }
    Ok(Json(json!({ "id": id, "name": "Alice" })))
}
```

---

## 10. Full CRUD Example

```rust
use axum::{
    extract::{Path, State},
    http::StatusCode,
    routing::{get, post, put, delete},
    Json, Router,
};
use serde::{Serialize, Deserialize};
use std::sync::{Arc, Mutex};

#[derive(Clone, Serialize, Deserialize)]
struct Item { id: u32, name: String, done: bool }

#[derive(Deserialize)]
struct CreateItem { name: String }

type Db = Arc<Mutex<Vec<Item>>>;

async fn list(State(db): State<Db>) -> Json<Vec<Item>> {
    Json(db.lock().unwrap().clone())
}

async fn create(State(db): State<Db>, Json(input): Json<CreateItem>) -> (StatusCode, Json<Item>) {
    let mut items = db.lock().unwrap();
    let item = Item { id: items.len() as u32 + 1, name: input.name, done: false };
    items.push(item.clone());
    (StatusCode::CREATED, Json(item))
}

async fn update(State(db): State<Db>, Path(id): Path<u32>, Json(input): Json<CreateItem>) -> Result<Json<Item>, StatusCode> {
    let mut items = db.lock().unwrap();
    let item = items.iter_mut().find(|i| i.id == id).ok_or(StatusCode::NOT_FOUND)?;
    item.name = input.name;
    Ok(Json(item.clone()))
}

async fn remove(State(db): State<Db>, Path(id): Path<u32>) -> StatusCode {
    let mut items = db.lock().unwrap();
    if let Some(pos) = items.iter().position(|i| i.id == id) {
        items.remove(pos);
        StatusCode::NO_CONTENT
    } else { StatusCode::NOT_FOUND }
}

#[tokio::main]
async fn main() {
    let db: Db = Arc::new(Mutex::new(vec![]));
    let app = Router::new()
        .route("/items", get(list).post(create))
        .route("/items/:id", put(update).delete(remove))
        .with_state(db);

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    println!("🚀 CRUD API on http://localhost:3000/items");
    axum::serve(listener, app).await.unwrap();
}
```

---

## 11. Summary Cheat Sheet

```
SETUP
────────────────────────────────────────────────────────────
axum = "0.7"
tokio = { version = "1", features = ["full"] }

ROUTING
────────────────────────────────────────────────────────────
Router::new()
  .route("/path", get(handler).post(handler))
  .route("/path/:id", get(h).put(h).delete(h))
  .nest("/api", sub_router)

EXTRACTORS
────────────────────────────────────────────────────────────
Path(id): Path<u32>              URL param
Query(params): Query<T>          ?key=value
Json(body): Json<T>              JSON body
State(state): State<AppState>    shared state

RESPONSES
────────────────────────────────────────────────────────────
&str / String                    plain text
Json(value)                      JSON
(StatusCode, Json(v))            status + JSON
Result<T, AppError>              error handling

MIDDLEWARE
────────────────────────────────────────────────────────────
.layer(CorsLayer::new())
.layer(middleware::from_fn(my_fn))

STATE
────────────────────────────────────────────────────────────
Arc<Mutex<T>>                    shared mutable state
.with_state(state)               attach to router
```

---

## What's Next?

**Lesson 94 — Database with SQLx** — Compile-time verified SQL, connection pools, and migrations.

## Further Reading
- [Axum docs](https://docs.rs/axum/)
- [Axum examples](https://github.com/tokio-rs/axum/tree/main/examples)
- [Tower middleware](https://docs.rs/tower/)

---

*Axum: production-grade web servers, the Rust way! 🦀*
