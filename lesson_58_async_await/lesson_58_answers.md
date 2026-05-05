# ✅ Lesson 58 — Answers: Async/Await Basics (AS1)

---

## Section A

### A1
It returns `impl Future<Output = String>` — a **future** that, when polled to completion, yields a `String`. It does NOT return a `String` directly.

### A2
**False.** Rust futures are lazy. Calling `fetch()` creates a `Future` object but does NOT execute the body. The body runs only when the future is polled by a runtime (via `.await` or explicit polling).

### A3
`.await` can only be used inside:
- `async fn` functions
- `async { }` blocks

It cannot be used in regular (synchronous) functions.

---

## Section B

### A4
```rust
async fn fetch_user(id: u32) -> String {
    format!("User_{id}")
}

async fn fetch_profile(user: &str) -> String {
    format!("{user}_profile")
}

async fn fetch_avatar(profile: &str) -> String {
    format!("{profile}_avatar.png")
}

async fn get_avatar(user_id: u32) -> String {
    let user = fetch_user(user_id).await;
    let profile = fetch_profile(&user).await;
    let avatar = fetch_avatar(&profile).await;
    avatar
}

// A runtime (like tokio) would:
// 1. Poll get_avatar → it calls fetch_user and returns Pending
// 2. Poll fetch_user → Ready("User_1")
// 3. Resume get_avatar → calls fetch_profile, returns Pending
// 4. Poll fetch_profile → Ready("User_1_profile")
// 5. Resume get_avatar → calls fetch_avatar, returns Pending
// 6. Poll fetch_avatar → Ready("User_1_profile_avatar.png")
// 7. get_avatar returns Ready("User_1_profile_avatar.png")
```

### A5
```rust
async fn example() {
    let nums = vec![1, 2, 3, 4, 5];

    let sum_future = async move {
        nums.iter().sum::<i32>()
    };

    let sum = sum_future.await;
    println!("Sum: {sum}");
    // println!("{:?}", nums);  // ❌ nums moved into async block
}
```
`async move` is needed because the async block might be polled later (lazily), possibly after `nums` goes out of scope. `move` ensures the block owns `nums` so it's safe to use whenever polled.

---

## Section C

### A6
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **False** | Rust futures are **lazy** — they do nothing until polled |
| 2 | **True** | `.await` only works in async contexts |
| 3 | **False** | `std` provides `Future` trait but NO runtime |
| 4 | **False** | `Pending` means "not ready yet, poll again later" |
| 5 | **True** | Async tasks are lightweight (~hundreds of bytes each) |
| 6 | **True** | `Pin` prevents movement which could invalidate self-references |
| 7 | **False** | `.await` **yields** control to the runtime; it doesn't block the OS thread |

### A7
**Threads:** Each of the 10,000 connections gets its own OS thread. Each thread uses ~8KB of stack memory (80MB total). The OS scheduler must context-switch between 10,000 threads — significant overhead. Most threads are idle (waiting for I/O).

**Async:** All 10,000 connections are handled as async tasks on a small thread pool (e.g., 4-8 threads). Each task uses ~hundreds of bytes. When a task waits for I/O, the runtime switches to another task — no OS context switch needed. Much less memory and CPU overhead.

For I/O-bound work like HTTP connections, async is dramatically more efficient.

---

## 🏆 Lesson 58 Complete!

**Next up:** [Lesson 59 — Tokio Runtime](../lesson_59_tokio/lesson_59_tokio.md) 🦀
