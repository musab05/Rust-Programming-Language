# 📘 Lesson 92 — Observer / Event System (DP4)

> **Series:** Rust From Zero · Advanced Level (Gap Fill)  
> **Roadmap ID:** DP4 · Category: 🏗 Design Patterns  
> **Previous:** [Lesson 91 — Cross-Compilation](../lesson_91_cross_compile/lesson_91_cross_compile.md)  
> **Next:** [Lesson 93 — Web Server with Axum](../lesson_93_axum/lesson_93_axum.md)  
> **Practice:** [Questions](./lesson_92_questions.md) · [Answers](./lesson_92_answers.md)  
> **Practice Task:** Build an event bus supporting typed events

---

## Table of Contents

1. [The Observer Pattern](#1-the-observer-pattern)
2. [Trait-Based Observer](#2-trait-based-observer)
3. [Closure-Based Events](#3-closure-based-events)
4. [Typed Event Bus](#4-typed-event-bus)
5. [Async Observer](#5-async-observer)
6. [Channel-Based Pub/Sub](#6-channel-based-pubsub)
7. [Real-World Example: UI Events](#7-real-world-example-ui-events)
8. [Summary Cheat Sheet](#8-summary-cheat-sheet)

---

## 1. The Observer Pattern

```
Subject (Publisher)          Observers (Subscribers)
┌──────────────────┐
│ EventEmitter     │───notify──▶ Observer A
│                  │───notify──▶ Observer B
│ .subscribe(obs)  │───notify──▶ Observer C
│ .emit(event)     │
└──────────────────┘
```

When the subject's state changes, all registered observers are notified automatically.

---

## 2. Trait-Based Observer

```rust
// Event type
#[derive(Debug, Clone)]
enum Event {
    UserLoggedIn(String),
    UserLoggedOut(String),
    MessageSent { from: String, to: String, body: String },
}

// Observer trait
trait Observer {
    fn on_event(&self, event: &Event);
}

// Subject (event emitter)
struct EventEmitter {
    observers: Vec<Box<dyn Observer>>,
}

impl EventEmitter {
    fn new() -> Self { EventEmitter { observers: vec![] } }

    fn subscribe(&mut self, observer: Box<dyn Observer>) {
        self.observers.push(observer);
    }

    fn emit(&self, event: Event) {
        for observer in &self.observers {
            observer.on_event(&event);
        }
    }
}

// Concrete observers
struct Logger;
impl Observer for Logger {
    fn on_event(&self, event: &Event) {
        println!("📝 LOG: {event:?}");
    }
}

struct SecurityAudit;
impl Observer for SecurityAudit {
    fn on_event(&self, event: &Event) {
        match event {
            Event::UserLoggedIn(user) => println!("🔒 AUDIT: {user} logged in"),
            Event::UserLoggedOut(user) => println!("🔒 AUDIT: {user} logged out"),
            _ => {}
        }
    }
}

struct Analytics {
    event_count: std::cell::Cell<u32>,
}

impl Analytics {
    fn new() -> Self { Analytics { event_count: std::cell::Cell::new(0) } }
}

impl Observer for Analytics {
    fn on_event(&self, _event: &Event) {
        let count = self.event_count.get() + 1;
        self.event_count.set(count);
        println!("📊 ANALYTICS: {count} total events");
    }
}

fn main() {
    let mut emitter = EventEmitter::new();
    emitter.subscribe(Box::new(Logger));
    emitter.subscribe(Box::new(SecurityAudit));
    emitter.subscribe(Box::new(Analytics::new()));

    emitter.emit(Event::UserLoggedIn("alice".into()));
    println!();
    emitter.emit(Event::MessageSent {
        from: "alice".into(), to: "bob".into(), body: "hello".into(),
    });
    println!();
    emitter.emit(Event::UserLoggedOut("alice".into()));
}
```

---

## 3. Closure-Based Events

More flexible — no need to define observer structs:

```rust
type Callback = Box<dyn Fn(&str)>;

struct EventBus {
    listeners: std::collections::HashMap<String, Vec<Callback>>,
}

impl EventBus {
    fn new() -> Self {
        EventBus { listeners: std::collections::HashMap::new() }
    }

    fn on(&mut self, event: &str, callback: impl Fn(&str) + 'static) {
        self.listeners
            .entry(event.to_string())
            .or_default()
            .push(Box::new(callback));
    }

    fn emit(&self, event: &str, data: &str) {
        if let Some(callbacks) = self.listeners.get(event) {
            for cb in callbacks {
                cb(data);
            }
        }
    }
}

fn main() {
    let mut bus = EventBus::new();

    bus.on("login", |user| println!("👤 {user} logged in"));
    bus.on("login", |user| println!("📊 Analytics: login by {user}"));
    bus.on("error", |msg| eprintln!("❌ Error: {msg}"));

    bus.emit("login", "alice");
    bus.emit("error", "database timeout");
    bus.emit("logout", "alice");  // no listeners — nothing happens
}
```

---

## 4. Typed Event Bus

Type-safe events using generics:

```rust
use std::any::{Any, TypeId};
use std::collections::HashMap;

struct TypedBus {
    handlers: HashMap<TypeId, Vec<Box<dyn Fn(&dyn Any)>>>,
}

impl TypedBus {
    fn new() -> Self { TypedBus { handlers: HashMap::new() } }

    fn subscribe<E: 'static>(&mut self, handler: impl Fn(&E) + 'static) {
        let type_id = TypeId::of::<E>();
        let wrapper = move |event: &dyn Any| {
            if let Some(e) = event.downcast_ref::<E>() {
                handler(e);
            }
        };
        self.handlers.entry(type_id).or_default().push(Box::new(wrapper));
    }

    fn publish<E: 'static>(&self, event: E) {
        let type_id = TypeId::of::<E>();
        if let Some(handlers) = self.handlers.get(&type_id) {
            for handler in handlers {
                handler(&event);
            }
        }
    }
}

// Typed events
#[derive(Debug)]
struct LoginEvent { user: String }

#[derive(Debug)]
struct ErrorEvent { code: u32, message: String }

fn main() {
    let mut bus = TypedBus::new();

    bus.subscribe(|e: &LoginEvent| println!("👤 Login: {}", e.user));
    bus.subscribe(|e: &LoginEvent| println!("📊 Analytics: login"));
    bus.subscribe(|e: &ErrorEvent| eprintln!("❌ Error {}: {}", e.code, e.message));

    bus.publish(LoginEvent { user: "alice".into() });
    bus.publish(ErrorEvent { code: 500, message: "Server crash".into() });
}
```

---

## 5. Async Observer

Using tokio channels for async event handling:

```rust
use tokio::sync::broadcast;

#[derive(Debug, Clone)]
enum AppEvent {
    UserAction(String),
    SystemAlert(String),
}

async fn logger(mut rx: broadcast::Receiver<AppEvent>) {
    while let Ok(event) = rx.recv().await {
        println!("📝 LOG: {event:?}");
    }
}

async fn alerter(mut rx: broadcast::Receiver<AppEvent>) {
    while let Ok(event) = rx.recv().await {
        if let AppEvent::SystemAlert(msg) = event {
            println!("🚨 ALERT: {msg}");
        }
    }
}

#[tokio::main]
async fn main() {
    let (tx, _) = broadcast::channel::<AppEvent>(32);

    tokio::spawn(logger(tx.subscribe()));
    tokio::spawn(alerter(tx.subscribe()));

    tx.send(AppEvent::UserAction("clicked button".into())).unwrap();
    tx.send(AppEvent::SystemAlert("disk full".into())).unwrap();

    tokio::time::sleep(tokio::time::Duration::from_millis(100)).await;
}
```

---

## 6. Channel-Based Pub/Sub

Scalable event system using mpsc:

```rust
use tokio::sync::mpsc;
use std::collections::HashMap;

type Handler = Box<dyn Fn(String) + Send>;

struct PubSub {
    tx: mpsc::Sender<(String, String)>,
}

impl PubSub {
    fn new(mut handlers: HashMap<String, Vec<Handler>>) -> Self {
        let (tx, mut rx) = mpsc::channel::<(String, String)>(100);

        tokio::spawn(async move {
            while let Some((topic, data)) = rx.recv().await {
                if let Some(list) = handlers.get(&topic) {
                    for h in list { h(data.clone()); }
                }
            }
        });

        PubSub { tx }
    }

    async fn publish(&self, topic: &str, data: &str) {
        self.tx.send((topic.into(), data.into())).await.ok();
    }
}

#[tokio::main]
async fn main() {
    let mut handlers: HashMap<String, Vec<Handler>> = HashMap::new();
    handlers.entry("user".into()).or_default().push(Box::new(|d| println!("👤 {d}")));
    handlers.entry("error".into()).or_default().push(Box::new(|d| eprintln!("❌ {d}")));

    let ps = PubSub::new(handlers);
    ps.publish("user", "alice logged in").await;
    ps.publish("error", "timeout").await;

    tokio::time::sleep(tokio::time::Duration::from_millis(50)).await;
}
```

---

## 7. Real-World Example: UI Events

```rust
use std::collections::HashMap;

type Listener = Box<dyn Fn(&serde_json::Value)>;

struct Component {
    name: String,
    listeners: HashMap<String, Vec<Listener>>,
}

impl Component {
    fn new(name: &str) -> Self {
        Component { name: name.into(), listeners: HashMap::new() }
    }

    fn on(&mut self, event: &str, handler: impl Fn(&serde_json::Value) + 'static) {
        self.listeners.entry(event.into()).or_default().push(Box::new(handler));
    }

    fn emit(&self, event: &str, data: serde_json::Value) {
        println!("[{}] emitting '{event}'", self.name);
        if let Some(handlers) = self.listeners.get(event) {
            for h in handlers { h(&data); }
        }
    }
}

fn main() {
    let mut button = Component::new("SubmitButton");

    button.on("click", |data| println!("  Click handler: {data}"));
    button.on("click", |_| println!("  Analytics: button clicked"));
    button.on("hover", |_| println!("  Tooltip shown"));

    button.emit("click", serde_json::json!({"x": 100, "y": 200}));
    button.emit("hover", serde_json::json!(null));
}
```

---

## 8. Summary Cheat Sheet

```
OBSERVER PATTERN
────────────────────────────────────────────────────────────
Subject.subscribe(observer)   register
Subject.emit(event)           notify all

APPROACHES
────────────────────────────────────────────────────────────
Trait-based    trait Observer { fn on_event(&self, e: &Event) }
Closure-based  .on("event", |data| { ... })
Typed bus      TypeId → type-safe downcasting
Async          broadcast/mpsc channels

WHEN TO USE
────────────────────────────────────────────────────────────
Decoupled components      → observer pattern
UI events                  → closure-based
Typed domain events        → typed bus
Async microservices        → channel-based pub/sub
```

---

## What's Next?

**Lesson 93 — Web Server with Axum** — Build REST APIs with routing, middleware, extractors, and JSON responses.

## Further Reading
- [Rust Design Patterns — Observer](https://rust-unofficial.github.io/patterns/)
- [tokio::sync::broadcast](https://docs.rs/tokio/latest/tokio/sync/broadcast/)

---

*Observer pattern: loosely coupled, highly reactive! 🦀*
