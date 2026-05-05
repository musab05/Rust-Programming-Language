# ✅ Lesson 57 — Answers: Send & Sync (CC4)

---

## Section A

### A1

| Type | Send | Sync |
|---|---|---|
| `i32` | ✅ | ✅ |
| `String` | ✅ | ✅ |
| `Rc<String>` | ❌ | ❌ |
| `Arc<String>` | ✅ | ✅ |
| `RefCell<i32>` | ✅ | ❌ |
| `Mutex<Vec<u8>>` | ✅ | ✅ |
| `Cell<bool>` | ✅ | ❌ |
| `*mut u8` | ❌ | ❌ |

Key reasons:
- `Rc`: non-atomic ref count → data race if shared
- `RefCell`/`Cell`: interior mutability without sync → `&RefCell` shared across threads = race
- `Mutex`: wraps data with a lock, making it safe → provides Sync even if inner T isn't
- `*mut`: raw pointers give no guarantees

---

## Section B

### A2
```rust
use std::sync::{Arc, Mutex};
use std::collections::HashMap;
use std::thread;

struct Cache {
    data: Arc<Mutex<HashMap<String, String>>>,
}

impl Cache {
    fn new() -> Self { Cache { data: Arc::new(Mutex::new(HashMap::new())) } }

    fn get(&self, key: &str) -> Option<String> {
        self.data.lock().unwrap().get(key).cloned()
    }

    fn set(&self, key: &str, val: &str) {
        self.data.lock().unwrap().insert(key.to_string(), val.to_string());
    }

    fn share(&self) -> Cache {
        Cache { data: Arc::clone(&self.data) }
    }
}

fn main() {
    let cache = Cache::new();
    let mut handles = vec![];

    for i in 0..3 {
        let c = cache.share();
        handles.push(thread::spawn(move || {
            c.set(&format!("key_{i}"), &format!("value_{i}"));
            println!("Thread {i}: wrote key_{i}");
        }));
    }

    for h in handles { h.join().unwrap(); }

    for i in 0..3 {
        println!("key_{i} = {:?}", cache.get(&format!("key_{i}")));
    }
}
```

### A3
```rust
fn assert_thread_safe<T: Send + Sync>() {}

fn main() {
    assert_thread_safe::<i32>();                // ✅
    assert_thread_safe::<String>();             // ✅
    assert_thread_safe::<Vec<u8>>();            // ✅
    assert_thread_safe::<std::sync::Arc<i32>>();// ✅
    assert_thread_safe::<std::sync::Mutex<String>>(); // ✅

    // These would NOT compile:
    // assert_thread_safe::<std::rc::Rc<i32>>();       // ❌
    // assert_thread_safe::<std::cell::RefCell<i32>>(); // ❌ (not Sync)
    // assert_thread_safe::<std::cell::Cell<i32>>();    // ❌ (not Sync)
}
```

---

## Section C

### A4
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **False** | `Send` means a value can be **moved** to another thread; `Sync` is for sharing via `&T` |
| 2 | **True** | Auto-traits propagate from fields |
| 3 | **True** | `Arc` needs `T: Send` to be `Send`, and `T: Sync` to be `Sync` |
| 4 | **True** | `Mutex<T>` is `Sync` for any `T: Send` — the lock ensures exclusive access |
| 5 | **True** | `PhantomData<*const ()>` poisons the struct with a non-Send type |
| 6 | **True** | It bypasses the compiler's auto-derivation when you manually guarantee safety |

### A5
`Rc` uses a **non-atomic** reference counter. If two threads clone/drop an `Rc` simultaneously, the counter could be read and written at the same time — a data race. There's no lock or atomic operation protecting it.

`Arc` uses **atomic** operations (`fetch_add`, `fetch_sub`) for its reference counter. These are hardware-level operations that are guaranteed to be indivisible even when multiple CPU cores access them simultaneously. No data race is possible.

---

## 🏆 Lesson 57 Complete!

**Next up:** [Lesson 58 — Async/Await Basics](../lesson_58_async_await/lesson_58_async_await.md) 🦀
