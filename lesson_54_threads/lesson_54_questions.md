# 🧪 Lesson 54 — Questions: Threads & spawn (CC1)

> **Lesson:** [lesson_54_threads.md](./lesson_54_threads.md)  
> **Answers:** [lesson_54_answers.md](./lesson_54_answers.md)

---

## Section A — Compile or Not?

### Q1
```rust
use std::thread;
fn main() {
    let name = String::from("Alice");
    thread::spawn(|| println!("{name}"));
}
```

### Q2
```rust
use std::thread;
fn main() {
    let name = String::from("Alice");
    let handle = thread::spawn(move || println!("{name}"));
    handle.join().unwrap();
}
```

### Q3
```rust
use std::thread;
use std::rc::Rc;
fn main() {
    let data = Rc::new(42);
    thread::spawn(move || println!("{data}"));
}
```

---

## Section B — Write It Yourself

### Q4 — Parallel word count (Roadmap Practice Task)
Split a list of strings across 3 threads. Each thread counts the total characters in its assigned strings. Sum the results on the main thread.

### Q5 — Scoped threads
Use `thread::scope` to let two threads borrow different slices of the same `Vec`, then print the full vec after both complete.

### Q6 — Return values
Spawn 4 threads that each compute the square of their index (0²=0, 1²=1, 2²=4, 3²=9). Collect all results into a `Vec<i32>` using `join()`.

---

## Section C — True or False?

### Q7
1. Spawned threads are killed when the main thread exits.
2. `join()` blocks the calling thread until the spawned thread finishes.
3. `Rc<T>` can be sent between threads.
4. Scoped threads (`thread::scope`) automatically join when the scope ends.
5. `move` transfers ownership of captured variables to the thread's closure.
6. `thread::spawn` returns a `JoinHandle<T>` where T is the closure's return type.

### Q8
Why does `thread::spawn` require `move` for closures that capture variables, while scoped threads don't?

---

*Threads: fearless concurrency starts here! 🦀*
