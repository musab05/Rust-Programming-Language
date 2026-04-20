# ✅ Lesson 24 — Answers: Modules & mod (M1)

---

## Section A

### A1
```
Hello!
```
`hello()` is `pub` so it's callable. `secret()` is private but isn't called, so no error.

### A2
```
inner says hi
outer called inner
inner says hi
```
`call_inner()` calls `inner::say()` then prints its own message. Then `main` calls `inner::say()` directly.

### A3
```
from parent
from child
from parent
```
Two different `greet()` functions at different module levels. `call_parent()` uses `super::greet()` to reach the parent's version.

---

## Section B

### A4 — ❌ No
`hidden()` is private (no `pub`). Can't access from `main`. Fix: add `pub` → `pub fn hidden()`.

### A5 — ❌ No
`radius` is private (no `pub` on the field). Can't construct `Circle` from outside the module since you can't set `radius`. Fix: either make `radius` pub, or add a constructor `pub fn new(r: f64) -> Circle`.

### A6 — ✅ Yes
Enum variants are automatically public when the enum is `pub`. `Animal::Dog` is accessible. Output: `Got a pet!`

### A7 — ✅ Yes
Both absolute (`crate::a::b::hello()`) and relative (`a::b::hello()`) paths work. Output:
```
hello from b
hello from b
```

---

## Section C

### A8
```rust
mod math {
    pub fn add(a: i32, b: i32) -> i32 { a + b }  // fix: make pub

    pub fn double(x: i32) -> i32 {
        add(x, x)
    }
}

fn main() {
    println!("{}", math::add(3, 4));     // ✅ now accessible
    println!("{}", math::double(5));
}
```

### A9
```rust
mod config {
    pub struct Settings {
        pub debug: bool,
        log_level: String,   // stays private
    }

    impl Settings {
        // fix: provide a constructor since log_level is private
        pub fn new(debug: bool, log_level: &str) -> Settings {
            Settings {
                debug,
                log_level: log_level.to_string(),
            }
        }

        pub fn log_level(&self) -> &str {
            &self.log_level
        }
    }
}

fn main() {
    let s = config::Settings::new(true, "info");
    println!("debug={}, level={}", s.debug, s.log_level());
}
```

### A10
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
    let b = utils::formatting::bold("test");  // fix: full path from crate root
    println!("{b}");
}
```

---

## Section D

### A11 — Garden module
```rust
mod garden {
    pub fn water() {
        println!("💧 Watering the garden");
    }

    pub mod vegetables {
        pub fn list() -> Vec<&'static str> {
            vec!["asparagus", "carrot", "broccoli", "spinach"]
        }

        pub fn grow_asparagus() {
            println!("🌱 Growing asparagus!");
        }
    }
}

fn main() {
    garden::water();
    garden::vegetables::grow_asparagus();
    let veggies = garden::vegetables::list();
    println!("Vegetables: {:?}", veggies);
}
```
**Output:**
```
💧 Watering the garden
🌱 Growing asparagus!
Vegetables: ["asparagus", "carrot", "broccoli", "spinach"]
```

### A12 — Visibility practice
```rust
mod bank {
    pub struct Account {
        pub name: String,
        balance: f64,  // private
    }

    impl Account {
        pub fn new(name: &str, initial_balance: f64) -> Account {
            let balance = if validate_amount(initial_balance) {
                initial_balance
            } else {
                0.0
            };
            Account {
                name: name.to_string(),
                balance,
            }
        }

        pub fn deposit(&mut self, amount: f64) {
            if validate_amount(amount) {
                self.balance += amount;
            }
        }

        pub fn get_balance(&self) -> f64 {
            self.balance
        }
    }

    fn validate_amount(amount: f64) -> bool {
        amount > 0.0
    }
}

fn main() {
    let mut account = bank::Account::new("Alice", 100.0);
    println!("Name: {}", account.name);         // ✅ public
    // println!("{}", account.balance);           // ❌ private
    println!("Balance: ${:.2}", account.get_balance());  // ✅

    account.deposit(50.0);
    account.deposit(-10.0);  // ignored — invalid amount
    println!("After deposits: ${:.2}", account.get_balance());
}
```
**Output:**
```
Name: Alice
Balance: $100.00
After deposits: $150.00
```

### A13 — Module with super
```rust
mod game {
    pub fn max_score() -> u32 {
        1000
    }

    pub mod player {
        pub fn is_winner(score: u32) -> bool {
            score >= super::max_score()
        }
    }
}

fn main() {
    println!("Max score: {}", game::max_score());
    println!("Score 500 wins? {}", game::player::is_winner(500));   // false
    println!("Score 1000 wins? {}", game::player::is_winner(1000)); // true
    println!("Score 1500 wins? {}", game::player::is_winner(1500)); // true
}
```

---

## Section E

### A14 — True or False?
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | Rust defaults to private — you must explicitly add `pub` |
| 2 | **False** | Children can access parent items, but siblings and parents cannot access private child items |
| 3 | **False** | `pub` on a struct only makes the struct visible; each field needs its own `pub` |
| 4 | **True** | Unlike structs, enum variants are automatically public when the enum is `pub` |
| 5 | **True** | `mod name;` is a declaration — Rust looks for `name.rs` or `name/mod.rs` |
| 6 | **True** | `super::` navigates to the parent module |
| 7 | **False** | `self::` refers to the current module, not a child |
| 8 | **True** | `pub use` makes an item from a submodule available at the current level |
| 9 | **False** | There's no limit on module nesting depth |
| 10 | **True** | Private functions are accessible to everything within the same module |

### A15 — Design question
```rust
// Suggested module structure:

mod auth {
    pub fn login(username: &str, password: &str) -> bool { ... }
    pub fn register(username: &str, password: &str) -> Result<(), String> { ... }
    fn hash_password(password: &str) -> String { ... }  // private — internal detail
}

mod database {
    pub fn connect(url: &str) -> Result<Connection, Error> { ... }
    pub fn query(conn: &Connection, sql: &str) -> Result<Vec<Row>, Error> { ... }
    pub fn disconnect(conn: Connection) { ... }

    pub struct Connection { /* fields private */ }
    struct Row { /* private internal type */ }
}

mod api {
    pub fn get_users() -> Vec<User> { ... }
    pub fn create_user(name: &str) -> Result<User, Error> { ... }
    pub fn delete_user(id: u32) -> Result<(), Error> { ... }
}
```

**Public:** Functions that form the API boundary (`login`, `register`, `connect`, `get_users`, etc.)  
**Private:** Implementation details (`hash_password`, internal types, helper functions)

---

## 🏆 Lesson 24 Complete!

✅ `mod name { ... }` — inline module definition  
✅ `mod name;` — module from separate file  
✅ `pub` — make items visible outside the module  
✅ Paths: `crate::`, `super::`, `self::`, relative  
✅ Struct field visibility is per-field  
✅ Enum variants are always public if the enum is public  
✅ `pub use` for re-exporting  
✅ Modern file layout without `mod.rs`  

**Up next: Lesson 25 — Paths & use 🦀**
