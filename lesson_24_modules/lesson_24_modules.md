# 📘 Lesson 24 — Modules & mod (M1)

> **Series:** Rust From Zero · Beginner Level  
> **Roadmap ID:** M1 · Category: 📁 Modules  
> **Previous:** [Lesson 23 — The ? Operator](../lesson_23_question_mark/lesson_23_question_mark.md)  
> **Next:** [Lesson 25 — Paths & use](../lesson_25_paths_use/lesson_25_paths_use.md)  
> **Practice:** [Questions](./lesson_24_questions.md) · [Answers](./lesson_24_answers.md)  
> **Practice Task:** Organise a library into `garden::vegetables::asparagus`

---

## Table of Contents

1. [Why Modules?](#1-why-modules)
2. [Defining Modules with `mod`](#2-defining-modules-with-mod)
3. [Module Tree & Nesting](#3-module-tree--nesting)
4. [Visibility: `pub` Keyword](#4-visibility-pub-keyword)
5. [Accessing Items: Paths](#5-accessing-items-paths)
6. [`super` and `self`](#6-super-and-self)
7. [Modules in Separate Files](#7-modules-in-separate-files)
8. [Re-exporting with `pub use`](#8-re-exporting-with-pub-use)
9. [`pub` on Struct Fields](#9-pub-on-struct-fields)
10. [Module Organisation Patterns](#10-module-organisation-patterns)
11. [Summary Cheat Sheet](#11-summary-cheat-sheet)

---

## 1. Why Modules?

As programs grow, you need to organise code into logical units. Modules let you:

- **Group** related functions, structs, and enums together
- **Control visibility** — decide what's public API vs private implementation
- **Avoid name clashes** — two modules can have functions with the same name
- **Navigate** large codebases with clear structure

```
my_project/
├── src/
│   ├── main.rs          (crate root)
│   ├── garden/
│   │   ├── mod.rs       (garden module)
│   │   └── vegetables.rs
│   └── utils.rs
```

---

## 2. Defining Modules with `mod`

Use the `mod` keyword to define a module **inline**:

```rust
mod math {
    pub fn add(a: i32, b: i32) -> i32 {
        a + b
    }

    pub fn multiply(a: i32, b: i32) -> i32 {
        a * b
    }

    fn secret_formula(x: i32) -> i32 {
        x * 42  // private — only accessible within this module
    }
}

fn main() {
    println!("{}", math::add(3, 4));       // 7
    println!("{}", math::multiply(3, 4));  // 12
    // println!("{}", math::secret_formula(5)); // ❌ ERROR: private
}
```

---

## 3. Module Tree & Nesting

Modules can be nested to create a hierarchy:

```rust
mod garden {
    pub mod vegetables {
        pub fn grow_asparagus() {
            println!("🌱 Growing asparagus!");
        }

        pub fn grow_carrot() {
            println!("🥕 Growing carrot!");
        }
    }

    pub mod fruits {
        pub fn grow_apple() {
            println!("🍎 Growing apple!");
        }
    }

    pub fn water_all() {
        println!("💧 Watering the garden...");
        // Can access child modules
        vegetables::grow_asparagus();
        fruits::grow_apple();
    }
}

fn main() {
    garden::vegetables::grow_asparagus();  // full path
    garden::fruits::grow_apple();
    garden::water_all();
}
```

### The module tree:
```
crate (root)
└── garden
    ├── vegetables
    │   ├── grow_asparagus()
    │   └── grow_carrot()
    ├── fruits
    │   └── grow_apple()
    └── water_all()
```

---

## 4. Visibility: `pub` Keyword

By default, everything in Rust is **private**. Use `pub` to make items accessible:

```rust
mod backend {
    pub fn public_api() {
        println!("This is public");
        private_helper();  // ✅ can call private fn from same module
    }

    fn private_helper() {
        println!("This is private — internal only");
    }

    pub struct User {
        pub name: String,    // public field
        password: String,    // private field!
    }

    impl User {
        pub fn new(name: &str, password: &str) -> User {
            User {
                name: name.to_string(),
                password: password.to_string(),
            }
        }

        pub fn check_password(&self, attempt: &str) -> bool {
            self.password == attempt
        }
    }
}

fn main() {
    backend::public_api();
    // backend::private_helper(); // ❌ ERROR: private

    let user = backend::User::new("Alice", "secret123");
    println!("Name: {}", user.name);           // ✅ public field
    // println!("Pass: {}", user.password);     // ❌ private field
    println!("Auth: {}", user.check_password("secret123")); // ✅
}
```

### Visibility rules:
- **Child modules** can access everything in parent modules
- **Parent modules** cannot access private items in child modules
- **Siblings** cannot access each other's private items
- `pub` makes an item visible to the parent (and therefore to the outside world)

---

## 5. Accessing Items: Paths

Two kinds of paths:

```rust
mod a {
    pub mod b {
        pub fn hello() {
            println!("Hello from a::b!");
        }
    }
}

fn main() {
    // Absolute path (from crate root)
    crate::a::b::hello();

    // Relative path (from current scope)
    a::b::hello();
}
```

| Path type | Starts with | Example |
|---|---|---|
| Absolute | `crate::` | `crate::garden::vegetables::grow()` |
| Relative | module name | `garden::vegetables::grow()` |
| Parent | `super::` | `super::helper()` |
| Current | `self::` | `self::helper()` |

---

## 6. `super` and `self`

### `super` — access parent module

```rust
mod audio {
    pub fn master_volume() -> u8 {
        100
    }

    pub mod effects {
        pub fn reverb(input: f32) -> f32 {
            let vol = super::master_volume();  // go up to parent (audio)
            input * (vol as f32 / 100.0)
        }
    }
}

fn main() {
    println!("{}", audio::effects::reverb(0.5));  // 0.5
}
```

### `self` — explicitly refer to current module

```rust
mod utils {
    fn helper() -> i32 { 42 }

    pub fn do_work() -> i32 {
        self::helper()  // same as just helper(), but explicit
    }
}
```

### `super` in tests (common pattern):

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;  // import everything from parent module

    #[test]
    fn test_add() {
        assert_eq!(add(2, 3), 5);
    }
}
```

---

## 7. Modules in Separate Files

For real projects, you move modules into their own files.

### Style 1: One file per module

```
src/
├── main.rs
├── garden.rs        ← mod garden
└── utils.rs         ← mod utils
```

**`src/main.rs`:**
```rust
mod garden;  // tells Rust to look for garden.rs (or garden/mod.rs)
mod utils;

fn main() {
    garden::grow();
    utils::log("Program started");
}
```

**`src/garden.rs`:**
```rust
pub fn grow() {
    println!("🌱 Growing!");
}
```

**`src/utils.rs`:**
```rust
pub fn log(message: &str) {
    println!("[LOG] {message}");
}
```

### Style 2: Directory with `mod.rs` (for nested modules)

```
src/
├── main.rs
└── garden/
    ├── mod.rs           ← mod garden
    └── vegetables.rs    ← mod vegetables (submodule)
```

**`src/main.rs`:**
```rust
mod garden;

fn main() {
    garden::vegetables::grow_asparagus();
}
```

**`src/garden/mod.rs`:**
```rust
pub mod vegetables;  // declares the submodule
```

**`src/garden/vegetables.rs`:**
```rust
pub fn grow_asparagus() {
    println!("🌱 Growing asparagus!");
}
```

### Style 3: Modern style (Rust 2018+) — no `mod.rs` needed

```
src/
├── main.rs
├── garden.rs            ← mod garden (declares submodules)
└── garden/
    └── vegetables.rs    ← submodule
```

**`src/garden.rs`:**
```rust
pub mod vegetables;
```

This is the **preferred modern style** — it avoids the confusing `mod.rs` files.

---

## 8. Re-exporting with `pub use`

`pub use` re-exports an item, making it available at a higher level:

```rust
mod shapes {
    mod circle {
        pub fn area(radius: f64) -> f64 {
            std::f64::consts::PI * radius * radius
        }
    }

    // Re-export so users don't need to know about the internal structure
    pub use circle::area as circle_area;
}

fn main() {
    // Without pub use: shapes::circle::area(5.0) — but circle is private!
    // With pub use:
    println!("{:.2}", shapes::circle_area(5.0)); // 78.54
}
```

This is powerful for creating clean public APIs while keeping internal structure flexible.

---

## 9. `pub` on Struct Fields

Structs have **per-field** visibility:

```rust
mod database {
    pub struct Connection {
        pub host: String,       // public — users can read/set
        pub port: u16,          // public
        password: String,       // PRIVATE — internal only
    }

    impl Connection {
        // Constructor needed because password is private
        pub fn new(host: &str, port: u16, password: &str) -> Connection {
            Connection {
                host: host.to_string(),
                port,
                password: password.to_string(),
            }
        }

        pub fn connect(&self) {
            println!("Connecting to {}:{} with password [hidden]", self.host, self.port);
        }
    }
}

fn main() {
    let conn = database::Connection::new("localhost", 5432, "secret");
    println!("Host: {}", conn.host);    // ✅
    // println!("Pass: {}", conn.password); // ❌ private
    conn.connect();
}
```

### Enums: all variants are public if the enum is public

```rust
mod shapes {
    pub enum Color {
        Red,       // automatically public
        Green,     // automatically public
        Blue,      // automatically public
    }
}

fn main() {
    let c = shapes::Color::Red;  // ✅ all variants accessible
}
```

---

## 10. Module Organisation Patterns

### Pattern 1: Feature-based modules
```
src/
├── main.rs
├── auth.rs           (login, register, tokens)
├── database.rs       (connections, queries)
└── api.rs            (routes, handlers)
```

### Pattern 2: Layer-based modules
```
src/
├── main.rs
├── models.rs         (data structures)
├── services.rs       (business logic)
└── handlers.rs       (I/O layer)
```

### Pattern 3: Mixed approach (real projects)
```
src/
├── main.rs
├── lib.rs            (library root)
├── config.rs
├── error.rs
├── models/
│   ├── mod.rs
│   ├── user.rs
│   └── post.rs
└── services/
    ├── mod.rs
    ├── auth.rs
    └── storage.rs
```

---

## 11. Summary Cheat Sheet

```
DEFINING MODULES
────────────────────────────────────────────────────────────
mod name { ... }              inline module
mod name;                     module from file (name.rs or name/mod.rs)

VISIBILITY
────────────────────────────────────────────────────────────
fn private_fn()               private (default)
pub fn public_fn()            public
pub(crate) fn crate_fn()      visible within crate only
pub(super) fn parent_fn()     visible to parent module only

PATHS
────────────────────────────────────────────────────────────
crate::module::item           absolute path
module::item                  relative path
super::item                   parent module
self::item                    current module

RE-EXPORTING
────────────────────────────────────────────────────────────
pub use inner::function;      re-export to this level
pub use inner::function as alias;  re-export with rename

FILE LAYOUT (modern style)
────────────────────────────────────────────────────────────
src/main.rs                   mod garden;
src/garden.rs                 pub mod vegetables;
src/garden/vegetables.rs      pub fn grow() { ... }

STRUCT VISIBILITY
────────────────────────────────────────────────────────────
pub struct S { pub x: i32, y: i32 }
  → x is public, y is private
  → constructor needed (can't create S from outside with private fields)

ENUM VISIBILITY
────────────────────────────────────────────────────────────
pub enum E { A, B, C }
  → all variants are public automatically
```

---

## What's Next?

**Lesson 25 — Paths & use** — Bring items into scope with `use`, rename with `as`, and use nested paths for clean imports.

## Further Reading
- [The Rust Book — Ch 7.2: Defining Modules](https://doc.rust-lang.org/book/ch07-02-defining-modules-to-control-scope-and-privacy.html)
- [Rust by Example — Modules](https://doc.rust-lang.org/rust-by-example/mod.html)

---

*A well-organised codebase is a joy to work with! 🦀*
