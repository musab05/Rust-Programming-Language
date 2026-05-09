# 📘 Lesson 66 — Advanced Traits & Type-Level Programming (AL2)

> **Series:** Rust From Zero · Advanced Level  
> **Roadmap ID:** AL2 · Category: 🧬 Advanced Patterns  
> **Previous:** [Lesson 65 — Advanced Lifetimes](../lesson_65_advanced_lifetimes/lesson_65_advanced_lifetimes.md)  
> **Next:** [Lesson 67 — Design Patterns: Builder](../lesson_67_builder_pattern/lesson_67_builder_pattern.md)  
> **Practice:** [Questions](./lesson_66_questions.md) · [Answers](./lesson_66_answers.md)  
> **Practice Task:** Implement the newtype pattern and blanket implementations

---

## Table of Contents

1. [Fully Qualified Syntax](#1-fully-qualified-syntax)
2. [Supertraits](#2-supertraits)
3. [Blanket Implementations](#3-blanket-implementations)
4. [The Newtype Pattern](#4-the-newtype-pattern)
5. [Marker Traits](#5-marker-traits)
6. [Sealed Traits](#6-sealed-traits)
7. [Extension Traits](#7-extension-traits)
8. [Type-State Pattern](#8-type-state-pattern)
9. [Summary Cheat Sheet](#9-summary-cheat-sheet)

---

## 1. Fully Qualified Syntax

When multiple traits define the same method:

```rust
trait Pilot {
    fn fly(&self) -> &str;
}

trait Wizard {
    fn fly(&self) -> &str;
}

struct Human;

impl Pilot for Human {
    fn fly(&self) -> &str { "This is your captain speaking" }
}

impl Wizard for Human {
    fn fly(&self) -> &str { "Up!" }
}

impl Human {
    fn fly(&self) -> &str { "waving arms furiously" }
}

fn main() {
    let person = Human;

    // Calls Human::fly (inherent method)
    println!("{}", person.fly());

    // Fully qualified — specify which trait
    println!("{}", Pilot::fly(&person));
    println!("{}", Wizard::fly(&person));

    // Fully qualified (verbose form)
    println!("{}", <Human as Pilot>::fly(&person));
}
```

### For associated functions (no `self`):

```rust
trait Animal {
    fn name() -> String;
}

struct Dog;

impl Dog {
    fn name() -> String { "Spot".into() }
}

impl Animal for Dog {
    fn name() -> String { "puppy".into() }
}

fn main() {
    println!("{}", Dog::name());              // Spot (inherent)
    println!("{}", <Dog as Animal>::name());   // puppy (trait)
}
```

---

## 2. Supertraits

A trait that requires another trait:

```rust
use std::fmt;

// Display is a SUPERTRAIT — OutlinePrint requires it
trait OutlinePrint: fmt::Display {
    fn outline_print(&self) {
        let output = self.to_string();  // can use Display methods!
        let len = output.len();
        println!("{}", "*".repeat(len + 4));
        println!("* {} *", output);
        println!("{}", "*".repeat(len + 4));
    }
}

struct Point { x: f64, y: f64 }

// Must implement Display first (supertrait requirement)
impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}

impl OutlinePrint for Point {}  // gets outline_print for free

fn main() {
    let p = Point { x: 1.0, y: 2.5 };
    p.outline_print();
    // ************
    // * (1, 2.5) *
    // ************
}
```

### Multiple supertraits:

```rust
trait Serializable: fmt::Display + fmt::Debug + Clone {
    fn serialize(&self) -> String {
        format!("{:?}", self)
    }
}
```

---

## 3. Blanket Implementations

Implement a trait for ALL types that satisfy a bound:

```rust
use std::fmt;

trait Printable {
    fn print(&self);
}

// Blanket impl: ANY type that implements Display gets Printable
impl<T: fmt::Display> Printable for T {
    fn print(&self) {
        println!("→ {self}");
    }
}

fn main() {
    42.print();                // → 42
    "hello".print();           // → hello
    3.14_f64.print();          // → 3.14
    String::from("hi").print(); // → hi
}
```

### The standard library uses blanket impls extensively:

```rust
// From std: if T implements Display, &T also implements Display
// impl<T: Display> Display for &T { ... }

// From std: Any T implements From<T> (identity conversion)
// impl<T> From<T> for T { fn from(t: T) -> T { t } }

// From std: if T: Into<U>, then U: From<T>
// impl<T, U> Into<U> for T where U: From<T> { ... }
```

---

## 4. The Newtype Pattern

Wrap a type in a tuple struct to add behavior:

```rust
use std::fmt;

// Can't implement Display for Vec<T> directly (orphan rule)
// But we CAN wrap it in a newtype:
struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

fn main() {
    let w = Wrapper(vec!["hello".into(), "world".into()]);
    println!("{w}");  // [hello, world]
}
```

### Newtype for type safety:

```rust
struct Meters(f64);
struct Kilometers(f64);
struct Miles(f64);

impl Meters {
    fn to_km(&self) -> Kilometers { Kilometers(self.0 / 1000.0) }
}

impl Kilometers {
    fn to_miles(&self) -> Miles { Miles(self.0 * 0.621371) }
}

fn drive(distance: Kilometers) {
    println!("Driving {:.1} km", distance.0);
}

fn main() {
    let d = Kilometers(42.0);
    drive(d);

    // drive(Meters(100.0));  // ❌ type error — can't pass Meters as Km
    let m = Meters(5000.0);
    drive(m.to_km());  // ✅ explicit conversion
}
```

### Transparent newtype with Deref:

```rust
use std::ops::Deref;

struct Email(String);

impl Email {
    fn new(s: &str) -> Result<Self, String> {
        if s.contains('@') { Ok(Email(s.to_string())) }
        else { Err("Invalid email".into()) }
    }
}

impl Deref for Email {
    type Target = String;
    fn deref(&self) -> &String { &self.0 }
}

fn main() {
    let email = Email::new("alice@example.com").unwrap();
    println!("Length: {}", email.len());       // Deref to String
    println!("Upper: {}", email.to_uppercase());
}
```

---

## 5. Marker Traits

Traits with no methods — they mark a type as having a property:

```rust
// Standard marker traits: Send, Sync, Copy, Sized, Unpin

// Custom marker trait
trait Validated {}

struct ValidatedEmail(String);
impl Validated for ValidatedEmail {}

struct RawInput(String);
// NOT Validated

fn send_email<T: Validated + AsRef<str>>(addr: &T) {
    println!("Sending to: {}", addr.as_ref());
}

impl AsRef<str> for ValidatedEmail {
    fn as_ref(&self) -> &str { &self.0 }
}

fn main() {
    let email = ValidatedEmail("alice@example.com".into());
    send_email(&email);  // ✅

    // let raw = RawInput("bad".into());
    // send_email(&raw);  // ❌ RawInput is not Validated
}
```

---

## 6. Sealed Traits

Prevent external crates from implementing your trait:

```rust
mod sealed {
    // Private module — external crates can't access this
    pub trait Sealed {}
}

// Public trait with private supertrait
pub trait MyProtocol: sealed::Sealed {
    fn process(&self) -> String;
}

// Only types in THIS crate can implement Sealed (and thus MyProtocol)
pub struct TypeA;
pub struct TypeB;

impl sealed::Sealed for TypeA {}
impl sealed::Sealed for TypeB {}

impl MyProtocol for TypeA {
    fn process(&self) -> String { "A processed".into() }
}

impl MyProtocol for TypeB {
    fn process(&self) -> String { "B processed".into() }
}

// External crate:
// impl MyProtocol for TheirType {}  // ❌ can't implement Sealed
```

---

## 7. Extension Traits

Add methods to types you don't own:

```rust
trait StringExt {
    fn is_blank(&self) -> bool;
    fn truncate_to(&self, max_len: usize) -> &str;
    fn word_count(&self) -> usize;
}

impl StringExt for str {
    fn is_blank(&self) -> bool {
        self.trim().is_empty()
    }

    fn truncate_to(&self, max_len: usize) -> &str {
        if self.len() <= max_len { self }
        else { &self[..max_len] }
    }

    fn word_count(&self) -> usize {
        self.split_whitespace().count()
    }
}

fn main() {
    let text = "Hello, beautiful world!";
    println!("Blank: {}", "  ".is_blank());         // true
    println!("Truncated: {}", text.truncate_to(5));  // Hello
    println!("Words: {}", text.word_count());        // 3
}
```

---

## 8. Type-State Pattern

Use the type system to enforce valid state transitions at compile time:

```rust
// States as zero-sized types
struct Draft;
struct Review;
struct Published;

struct Post<State> {
    title: String,
    content: String,
    _state: std::marker::PhantomData<State>,
}

impl Post<Draft> {
    fn new(title: &str) -> Self {
        Post { title: title.into(), content: String::new(), _state: std::marker::PhantomData }
    }

    fn write(&mut self, text: &str) { self.content.push_str(text); }

    fn submit(self) -> Post<Review> {
        println!("📝 Submitted for review");
        Post { title: self.title, content: self.content, _state: std::marker::PhantomData }
    }
}

impl Post<Review> {
    fn approve(self) -> Post<Published> {
        println!("✅ Approved!");
        Post { title: self.title, content: self.content, _state: std::marker::PhantomData }
    }

    fn reject(self) -> Post<Draft> {
        println!("❌ Rejected, back to draft");
        Post { title: self.title, content: self.content, _state: std::marker::PhantomData }
    }
}

impl Post<Published> {
    fn display(&self) {
        println!("📰 {} — {}", self.title, self.content);
    }
}

fn main() {
    let mut post = Post::<Draft>::new("Rust is Great");
    post.write("Rust provides memory safety...");

    let in_review = post.submit();
    // in_review.write("more");  // ❌ can't write in Review state
    // in_review.display();       // ❌ can't display in Review state

    let published = in_review.approve();
    published.display();  // ✅
    // published.approve();  // ❌ can't approve already Published
}
```

---

## 9. Summary Cheat Sheet

```
FULLY QUALIFIED SYNTAX
────────────────────────────────────────────────────────────
<Type as Trait>::method(&val)     disambiguate methods

SUPERTRAITS
────────────────────────────────────────────────────────────
trait A: B + C { }               A requires B and C

BLANKET IMPLEMENTATIONS
────────────────────────────────────────────────────────────
impl<T: Display> MyTrait for T   implement for ALL Display types

NEWTYPE PATTERN
────────────────────────────────────────────────────────────
struct Wrapper(Inner);           bypass orphan rule, add type safety
impl Deref for Wrapper           transparent access to inner type

MARKER TRAITS
────────────────────────────────────────────────────────────
trait Marker {}                  no methods, marks a property

SEALED TRAITS
────────────────────────────────────────────────────────────
Private supertrait               prevents external implementations

EXTENSION TRAITS
────────────────────────────────────────────────────────────
trait StrExt { fn my_method(); } add methods to foreign types
impl StrExt for str { ... }

TYPE-STATE
────────────────────────────────────────────────────────────
struct Obj<State>                compile-time state machine
fn transition(self) -> Obj<Next> consume state, return next
```

---

## What's Next?

**Lesson 67 — Design Patterns: Builder** — The Builder pattern for constructing complex objects step by step.

## Further Reading
- [The Rust Book — Ch 19.3: Advanced Traits](https://doc.rust-lang.org/book/ch19-03-advanced-traits.html)
- [Rust Design Patterns — Newtype](https://rust-unofficial.github.io/patterns/patterns/behavioural/newtype.html)

---

*Advanced traits: the type system is your ally! 🦀*
