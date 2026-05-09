# 📘 Lesson 71 — Weak\<T\>: Breaking Reference Cycles (SP4)

> **Series:** Rust From Zero · Intermediate Level (Gap Fill)  
> **Roadmap ID:** SP4 · Category: 📌 Smart Pointers  
> **Previous:** [Lesson 70 — Advanced Testing](../lesson_70_advanced_testing/lesson_70_advanced_testing.md)  
> **Next:** [Lesson 72 — Type Aliases](../lesson_72_type_aliases/lesson_72_type_aliases.md)  
> **Practice:** [Questions](./lesson_71_questions.md) · [Answers](./lesson_71_answers.md)  
> **Practice Task:** Build a tree with parent back-references using Weak

---

## Table of Contents

1. [The Reference Cycle Problem](#1-the-reference-cycle-problem)
2. [What Is Weak\<T\>?](#2-what-is-weakt)
3. [Creating Weak References](#3-creating-weak-references)
4. [upgrade() — From Weak to Rc](#4-upgrade--from-weak-to-rc)
5. [Strong vs Weak Counts](#5-strong-vs-weak-counts)
6. [Breaking Cycles with Weak](#6-breaking-cycles-with-weak)
7. [Tree with Parent References](#7-tree-with-parent-references)
8. [Weak in Practice](#8-weak-in-practice)
9. [Summary Cheat Sheet](#9-summary-cheat-sheet)

---

## 1. The Reference Cycle Problem

`Rc<T>` can create cycles that **never get freed** — a memory leak:

```rust
use std::rc::Rc;
use std::cell::RefCell;

#[derive(Debug)]
struct Node {
    value: i32,
    next: Option<Rc<RefCell<Node>>>,
}

fn main() {
    let a = Rc::new(RefCell::new(Node { value: 1, next: None }));
    let b = Rc::new(RefCell::new(Node { value: 2, next: None }));

    // a → b
    a.borrow_mut().next = Some(Rc::clone(&b));
    // b → a  (CYCLE!)
    b.borrow_mut().next = Some(Rc::clone(&a));

    // Neither a nor b can be dropped!
    // strong_count never reaches 0 → memory leak
    println!("a strong: {}", Rc::strong_count(&a));  // 2
    println!("b strong: {}", Rc::strong_count(&b));  // 2
}
```

---

## 2. What Is Weak\<T\>?

`Weak<T>` is a **non-owning** reference. It doesn't prevent the value from being dropped:

```
Rc<T>   — strong reference → keeps value alive
Weak<T> — weak reference   → does NOT keep value alive
```

| | `Rc<T>` | `Weak<T>` |
|---|---|---|
| Keeps value alive | ✅ Yes | ❌ No |
| Can access value | ✅ Always | ⚠️ Only if alive (`upgrade()`) |
| Counts toward drop | ✅ Yes | ❌ No |
| Created via | `Rc::new()`, `Rc::clone()` | `Rc::downgrade()` |

---

## 3. Creating Weak References

```rust
use std::rc::{Rc, Weak};

fn main() {
    let strong: Rc<String> = Rc::new("hello".to_string());
    let weak: Weak<String> = Rc::downgrade(&strong);

    println!("Strong count: {}", Rc::strong_count(&strong));  // 1
    println!("Weak count: {}", Rc::weak_count(&strong));      // 1

    // Weak doesn't keep value alive
    println!("Weak is alive: {}", weak.upgrade().is_some());  // true

    drop(strong);  // value is dropped (strong count → 0)

    // Weak now points to nothing
    println!("Weak is alive: {}", weak.upgrade().is_some());  // false
}
```

---

## 4. upgrade() — From Weak to Rc

`upgrade()` returns `Option<Rc<T>>` — `Some` if the value is still alive, `None` if dropped:

```rust
use std::rc::{Rc, Weak};

fn try_use(weak: &Weak<String>) {
    match weak.upgrade() {
        Some(rc) => println!("Value: {rc}"),
        None => println!("Value has been dropped!"),
    }
}

fn main() {
    let data = Rc::new("important data".to_string());
    let weak = Rc::downgrade(&data);

    try_use(&weak);  // Value: important data

    drop(data);

    try_use(&weak);  // Value has been dropped!
}
```

---

## 5. Strong vs Weak Counts

```rust
use std::rc::Rc;

fn main() {
    let a = Rc::new(42);
    println!("After create:    strong={}, weak={}", Rc::strong_count(&a), Rc::weak_count(&a));

    let b = Rc::clone(&a);  // strong clone
    println!("After Rc::clone: strong={}, weak={}", Rc::strong_count(&a), Rc::weak_count(&a));

    let c = Rc::downgrade(&a);  // weak reference
    println!("After downgrade: strong={}, weak={}", Rc::strong_count(&a), Rc::weak_count(&a));

    let d = Rc::downgrade(&a);  // another weak
    println!("After downgrade: strong={}, weak={}", Rc::strong_count(&a), Rc::weak_count(&a));

    drop(b);
    println!("After drop(b):   strong={}, weak={}", Rc::strong_count(&a), Rc::weak_count(&a));

    // Output:
    // After create:    strong=1, weak=0
    // After Rc::clone: strong=2, weak=0
    // After downgrade: strong=2, weak=1
    // After downgrade: strong=2, weak=2
    // After drop(b):   strong=1, weak=2
}
```

**Rule:** Value is dropped when `strong_count` reaches 0. Weak count is irrelevant for drop.

---

## 6. Breaking Cycles with Weak

Use `Weak` for the "back edge" to prevent cycles:

```rust
use std::rc::{Rc, Weak};
use std::cell::RefCell;

#[derive(Debug)]
struct Node {
    value: i32,
    next: Option<Rc<RefCell<Node>>>,
    prev: Option<Weak<RefCell<Node>>>,  // Weak! won't create cycle
}

impl Node {
    fn new(value: i32) -> Rc<RefCell<Self>> {
        Rc::new(RefCell::new(Node { value, next: None, prev: None }))
    }
}

impl Drop for Node {
    fn drop(&mut self) { println!("  Dropping Node({})", self.value); }
}

fn main() {
    let a = Node::new(1);
    let b = Node::new(2);
    let c = Node::new(3);

    // Forward links: a → b → c (strong)
    a.borrow_mut().next = Some(Rc::clone(&b));
    b.borrow_mut().next = Some(Rc::clone(&c));

    // Back links: c → b → a (weak — NO cycle!)
    c.borrow_mut().prev = Some(Rc::downgrade(&b));
    b.borrow_mut().prev = Some(Rc::downgrade(&a));

    println!("a strong: {}", Rc::strong_count(&a));  // 1
    println!("b strong: {}", Rc::strong_count(&b));  // 2 (a.next + b itself)
    println!("c strong: {}", Rc::strong_count(&c));  // 2 (b.next + c itself)

    // Navigate backward from c
    if let Some(prev) = &c.borrow().prev {
        if let Some(node) = prev.upgrade() {
            println!("c's prev: {}", node.borrow().value);  // 2
        }
    }

    println!("\nDropping all...");
    // All nodes are dropped! No leak!
}
```

---

## 7. Tree with Parent References

The roadmap practice task:

```rust
use std::rc::{Rc, Weak};
use std::cell::RefCell;

#[derive(Debug)]
struct TreeNode {
    value: String,
    parent: RefCell<Weak<TreeNode>>,       // parent = Weak (back ref)
    children: RefCell<Vec<Rc<TreeNode>>>,   // children = Rc (forward ref)
}

impl TreeNode {
    fn new(value: &str) -> Rc<Self> {
        Rc::new(TreeNode {
            value: value.to_string(),
            parent: RefCell::new(Weak::new()),
            children: RefCell::new(vec![]),
        })
    }

    fn add_child(parent: &Rc<TreeNode>, child: &Rc<TreeNode>) {
        *child.parent.borrow_mut() = Rc::downgrade(parent);
        parent.children.borrow_mut().push(Rc::clone(child));
    }

    fn parent_name(&self) -> String {
        match self.parent.borrow().upgrade() {
            Some(p) => p.value.clone(),
            None => "(root)".to_string(),
        }
    }
}

impl Drop for TreeNode {
    fn drop(&mut self) { println!("  Dropping: {}", self.value); }
}

fn main() {
    let root = TreeNode::new("CEO");
    let eng = TreeNode::new("VP Engineering");
    let sales = TreeNode::new("VP Sales");
    let dev1 = TreeNode::new("Dev Team Lead");
    let dev2 = TreeNode::new("QA Lead");

    TreeNode::add_child(&root, &eng);
    TreeNode::add_child(&root, &sales);
    TreeNode::add_child(&eng, &dev1);
    TreeNode::add_child(&eng, &dev2);

    // Navigate tree
    println!("Tree structure:");
    println!("  {} (parent: {})", root.value, root.parent_name());
    for child in root.children.borrow().iter() {
        println!("    {} (parent: {})", child.value, child.parent_name());
        for grandchild in child.children.borrow().iter() {
            println!("      {} (parent: {})", grandchild.value, grandchild.parent_name());
        }
    }

    println!("\nStrong counts:");
    println!("  root: {}", Rc::strong_count(&root));
    println!("  eng:  {}", Rc::strong_count(&eng));
    println!("  dev1: {}", Rc::strong_count(&dev1));

    println!("\nDropping...");
    // All nodes drop cleanly — no cycles!
}
```

---

## 8. Weak in Practice

### Observer pattern (prevent dangling subscribers):

```rust
use std::rc::{Rc, Weak};
use std::cell::RefCell;

struct EventBus {
    listeners: RefCell<Vec<Weak<dyn Fn(&str)>>>,
}

impl EventBus {
    fn new() -> Self { EventBus { listeners: RefCell::new(vec![]) } }

    fn subscribe(&self, listener: &Rc<dyn Fn(&str)>) {
        self.listeners.borrow_mut().push(Rc::downgrade(listener));
    }

    fn emit(&self, event: &str) {
        let listeners = self.listeners.borrow();
        for weak in listeners.iter() {
            if let Some(listener) = weak.upgrade() {
                listener(event);
            }
        }
    }
}
```

### Cache (don't prevent cleanup):

```rust
use std::rc::{Rc, Weak};
use std::collections::HashMap;

struct Cache {
    entries: HashMap<String, Weak<String>>,
}

impl Cache {
    fn new() -> Self { Cache { entries: HashMap::new() } }

    fn get(&self, key: &str) -> Option<Rc<String>> {
        self.entries.get(key)?.upgrade()
    }

    fn store(&mut self, key: &str, value: &Rc<String>) {
        self.entries.insert(key.to_string(), Rc::downgrade(value));
    }
}
```

---

## 9. Summary Cheat Sheet

```
CREATING
────────────────────────────────────────────────────────────
Rc::downgrade(&rc)      → Weak<T>   (non-owning)
Weak::new()             → empty weak (always fails upgrade)

USING
────────────────────────────────────────────────────────────
weak.upgrade()          → Option<Rc<T>>  (Some if alive)
Rc::strong_count(&rc)   → number of Rc references
Rc::weak_count(&rc)     → number of Weak references

DROP RULE
────────────────────────────────────────────────────────────
strong_count → 0  ⇒  value DROPPED (weak count irrelevant)
After drop: upgrade() returns None

CYCLE PREVENTION
────────────────────────────────────────────────────────────
Forward refs (parent → child) → Rc<T>     (strong)
Back refs (child → parent)   → Weak<T>   (weak)

USE CASES
────────────────────────────────────────────────────────────
Tree parent pointers
Observer/listener cleanup
Cache entries
Doubly-linked structures
```

---

## What's Next?

**Lesson 72 — Type Aliases** — Simplify complex types with `type` aliases. Improve readability without runtime cost.

## Further Reading
- [The Rust Book — Ch 15.6: Reference Cycles](https://doc.rust-lang.org/book/ch15-06-reference-cycles.html)
- [std::rc::Weak](https://doc.rust-lang.org/std/rc/struct.Weak.html)

---

*Weak\<T\>: the polite reference that doesn't overstay its welcome! 🦀*
