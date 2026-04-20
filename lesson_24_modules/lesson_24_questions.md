# 🧪 Lesson 24 — Questions: Modules & mod (M1)

> **Lesson:** [lesson_24_modules.md](./lesson_24_modules.md)  
> **Answers:** [lesson_24_answers.md](./lesson_24_answers.md)

---

## Section A — Predict the Output

### Q1
```rust
mod greetings {
    pub fn hello() { println!("Hello!"); }
    fn secret() { println!("Secret!"); }
}

fn main() {
    greetings::hello();
}
```

### Q2
```rust
mod outer {
    pub mod inner {
        pub fn say() { println!("inner says hi"); }
    }
    pub fn call_inner() {
        inner::say();
        println!("outer called inner");
    }
}

fn main() {
    outer::call_inner();
    outer::inner::say();
}
```

### Q3
```rust
mod parent {
    pub fn greet() -> &'static str { "from parent" }

    pub mod child {
        pub fn greet() -> &'static str { "from child" }

        pub fn call_parent() -> &'static str {
            super::greet()
        }
    }
}

fn main() {
    println!("{}", parent::greet());
    println!("{}", parent::child::greet());
    println!("{}", parent::child::call_parent());
}
```

---

## Section B — Will It Compile?

### Q4
```rust
mod secret {
    fn hidden() { println!("hidden"); }
}

fn main() {
    secret::hidden();
}
```

### Q5
```rust
mod shapes {
    pub struct Circle {
        radius: f64,
    }
}

fn main() {
    let c = shapes::Circle { radius: 5.0 };
    println!("{}", c.radius);
}
```

### Q6
```rust
mod animals {
    pub enum Animal {
        Dog,
        Cat,
        Bird,
    }
}

fn main() {
    let pet = animals::Animal::Dog;
    println!("Got a pet!");
}
```

### Q7
```rust
mod a {
    pub mod b {
        pub fn hello() { println!("hello from b"); }
    }
}

fn main() {
    crate::a::b::hello();
    a::b::hello();
}
```

---

## Section C — Fix the Bugs

### Q8
```rust
mod math {
    fn add(a: i32, b: i32) -> i32 { a + b }

    pub fn double(x: i32) -> i32 {
        add(x, x)
    }
}

fn main() {
    println!("{}", math::add(3, 4));     // BUG: add is private
    println!("{}", math::double(5));
}
```

### Q9
```rust
mod config {
    pub struct Settings {
        pub debug: bool,
        log_level: String,
    }
}

fn main() {
    let s = config::Settings {
        debug: true,
        log_level: String::from("info"),   // BUG: log_level is private
    };
    println!("debug={}", s.debug);
}
```

### Q10
```rust
mod utils {
    pub mod formatting {
        pub fn bold(s: &str) -> String {
            format!("**{s}**")
        }
    }

    pub fn log(msg: &str) {
        let formatted = formatting::bold(msg);
        println!("[LOG] {formatted}");
    }
}

fn main() {
    utils::log("started");
    let b = formatting::bold("test");  // BUG: wrong path
    println!("{b}");
}
```

---

## Section D — Write It Yourself

### Q11 — Garden module (Roadmap Practice Task)
Create a module structure `garden::vegetables::asparagus` using inline modules:
- `garden` has a public function `water()` that prints "Watering the garden"
- `garden::vegetables` has a public function `list()` that returns a `Vec<&str>` of vegetable names
- `garden::vegetables` has a function `grow_asparagus()` that prints "Growing asparagus!"
- In `main`, call all three functions

### Q12 — Visibility practice
Create a module `bank` with:
- A public struct `Account` with a public `name: String` and private `balance: f64`
- A public constructor `Account::new(name, initial_balance)`
- A public method `deposit(&mut self, amount: f64)`
- A public method `get_balance(&self) -> f64`
- A private method `validate_amount(amount: f64) -> bool` (returns true if amount > 0)

In `main`, create an account, deposit money, and print the balance. Show that `balance` cannot be accessed directly.

### Q13 — Module with super
Create a module `game` with:
- A function `max_score() -> u32` that returns 1000
- A submodule `player` with a function `is_winner(score: u32) -> bool` that uses `super::max_score()` to check if the score meets the max

Test from `main`.

---

## Section E — Deep Understanding

### Q14 — True or False?
1. All items in a module are private by default.
2. A child module can access private items in its parent module.
3. `pub` on a struct makes all its fields public.
4. Enum variants are always public if the enum itself is `pub`.
5. `mod name;` in `main.rs` tells Rust to look for `name.rs` or `name/mod.rs`.
6. `super::` refers to the parent module.
7. `self::` refers to the child module.
8. `pub use` re-exports an item at the current module level.
9. You cannot nest modules more than 2 levels deep.
10. A private function can call another private function in the same module.

### Q15 — Design question
You're building a library with these components:
- User authentication (login, register, hash_password)
- Database operations (connect, query, disconnect)
- API endpoints (get_users, create_user, delete_user)

Sketch out a module structure. Which functions would be `pub`? Which would be private?

---

*Organise your code — your future self will thank you! 🦀*
