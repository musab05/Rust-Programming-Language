# 🧪 Lesson 57 — Questions: Send & Sync (CC4)

> **Lesson:** [lesson_57_send_sync.md](./lesson_57_send_sync.md)  
> **Answers:** [lesson_57_answers.md](./lesson_57_answers.md)

---

## Section A — Send or Not?

### Q1
For each type, state whether it implements `Send`, `Sync`, both, or neither:
1. `i32`
2. `String`
3. `Rc<String>`
4. `Arc<String>`
5. `RefCell<i32>`
6. `Mutex<Vec<u8>>`
7. `Cell<bool>`
8. `*mut u8`

---

## Section B — Write It Yourself

### Q2 — Thread-safe wrapper (Roadmap Practice Task)
You have a single-threaded `Cache` using `Rc<RefCell<HashMap>>`. Rewrite it as a thread-safe version using `Arc<Mutex<HashMap>>`. Demonstrate sharing between 3 threads.

### Q3 — Compile-time assertions
Write a function `assert_thread_safe<T: Send + Sync>()` and call it with various types. Which calls compile and which don't?

---

## Section C — True or False?

### Q4
1. `Send` means a value can be shared via `&T` across threads.
2. If all fields of a struct are `Send`, the struct is automatically `Send`.
3. `Arc<T>` requires `T: Send + Sync` to be useful across threads.
4. `Mutex<T>` makes `T` accessible from multiple threads even if `T` is not `Sync`.
5. You can opt out of `Send` using `PhantomData<*const ()>`.
6. `unsafe impl Send` is needed when the compiler can't verify thread safety.

### Q5
Explain in your own words why `Rc<T>` is neither `Send` nor `Sync`, while `Arc<T>` is both.

---

*Send & Sync: the invisible guardians of thread safety! 🦀*
