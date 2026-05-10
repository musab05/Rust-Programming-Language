# ✅ Lesson 93 — Answers: Web Server with Axum (RW5)

---

## Section A

### A1
An **extractor** is a type that Axum automatically populates from the incoming HTTP request. Built-in extractors:
1. `Path(val)` — URL path parameters
2. `Query(params)` — query string `?key=value`
3. `Json(body)` — JSON request body (deserializes via serde)
4. `State(state)` — shared application state

### A2
Axum handlers may run concurrently across threads. `Arc` provides shared ownership across tasks, and `Mutex` provides interior mutability with thread safety. Without them, you can't safely share mutable state between concurrent request handlers.

---

## Section B

### A3
```rust
use axum::{extract::{Path, State}, http::StatusCode, routing::{get, post, put, delete}, Json, Router};
use serde::{Serialize, Deserialize};
use std::sync::{Arc, Mutex};

#[derive(Clone, Serialize, Deserialize)]
struct Todo { id: u32, title: String, done: bool }
#[derive(Deserialize)]
struct Input { title: String }
type Db = Arc<Mutex<Vec<Todo>>>;

async fn list(State(db): State<Db>) -> Json<Vec<Todo>> { Json(db.lock().unwrap().clone()) }
async fn create(State(db): State<Db>, Json(i): Json<Input>) -> (StatusCode, Json<Todo>) {
    let mut todos = db.lock().unwrap();
    let t = Todo { id: todos.len() as u32 + 1, title: i.title, done: false };
    todos.push(t.clone());
    (StatusCode::CREATED, Json(t))
}
async fn update(State(db): State<Db>, Path(id): Path<u32>, Json(i): Json<Input>) -> Result<Json<Todo>, StatusCode> {
    let mut todos = db.lock().unwrap();
    let t = todos.iter_mut().find(|t| t.id == id).ok_or(StatusCode::NOT_FOUND)?;
    t.title = i.title; Ok(Json(t.clone()))
}
async fn remove(State(db): State<Db>, Path(id): Path<u32>) -> StatusCode {
    let mut todos = db.lock().unwrap();
    if let Some(p) = todos.iter().position(|t| t.id == id) { todos.remove(p); StatusCode::NO_CONTENT }
    else { StatusCode::NOT_FOUND }
}

#[tokio::main]
async fn main() {
    let db: Db = Arc::new(Mutex::new(vec![]));
    let app = Router::new()
        .route("/todos", get(list).post(create))
        .route("/todos/:id", put(update).delete(remove))
        .with_state(db);
    let l = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    axum::serve(l, app).await.unwrap();
}
```

### A4
```rust
use axum::{middleware, http::Request, extract::Next, response::Response};
use std::time::Instant;

async fn logger(req: Request<axum::body::Body>, next: Next) -> Response {
    let method = req.method().clone();
    let uri = req.uri().clone();
    let start = Instant::now();
    let resp = next.run(req).await;
    println!("{method} {uri} → {} [{:?}]", resp.status(), start.elapsed());
    resp
}
// Usage: .layer(middleware::from_fn(logger))
```

---

## Section C

### A5
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | Handlers are `async fn` returning something that implements `IntoResponse` |
| 2 | **True** | Path extractor parses URL segments into the specified type |
| 3 | **False** | `Json` as extractor requires `T: Deserialize`; as response requires `T: Serialize` |
| 4 | **True** | `.with_state(s)` makes `State(s)` available in handlers |
| 5 | **True** | Axum uses tower `Service` and `Layer` traits |
| 6 | **True** | `nest` prefixes all nested routes |

---

## 🏆 Lesson 93 Complete!

**Next up:** [Lesson 94 — Database with SQLx](../lesson_94_sqlx/lesson_94_sqlx.md) 🦀
