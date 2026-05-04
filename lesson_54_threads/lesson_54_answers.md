# ✅ Lesson 54 — Answers: Threads & spawn (CC1)

---

## Section A

### A1 — ❌ Won't compile
The closure borrows `name`, but the spawned thread might outlive `name`. Compiler error: `closure may outlive the current function`. Fix: add `move`.

### A2 — ✅ Compiles
`move` transfers ownership of `name` to the thread. `join()` waits for completion. Output: `Alice`.

### A3 — ❌ Won't compile
`Rc` is NOT `Send` — it can't be transferred between threads. Error: `Rc<i32> cannot be sent between threads safely`. Fix: use `Arc` instead.

---

## Section B

### A4
```rust
use std::thread;

fn main() {
    let texts = vec![
        "hello world".to_string(),
        "rust is great".to_string(),
        "fearless concurrency".to_string(),
        "threads are fun".to_string(),
        "parallel processing".to_string(),
        "zero cost abstractions".to_string(),
    ];

    let chunk_size = (texts.len() + 2) / 3;
    let mut handles = vec![];

    for (i, chunk) in texts.chunks(chunk_size).enumerate() {
        let chunk = chunk.to_vec();
        handles.push(thread::spawn(move || {
            let count: usize = chunk.iter().map(|s| s.len()).sum();
            println!("Thread {i}: {count} chars");
            count
        }));
    }

    let total: usize = handles.into_iter().map(|h| h.join().unwrap()).sum();
    println!("Total characters: {total}");
}
```

### A5
```rust
use std::thread;

fn main() {
    let data = vec![1, 2, 3, 4, 5, 6];

    thread::scope(|s| {
        s.spawn(|| println!("First half: {:?}", &data[..3]));
        s.spawn(|| println!("Second half: {:?}", &data[3..]));
    });

    println!("Full: {:?}", data);  // ✅ data still available
}
```

### A6
```rust
use std::thread;

fn main() {
    let handles: Vec<_> = (0..4)
        .map(|i| thread::spawn(move || i * i))
        .collect();

    let results: Vec<i32> = handles.into_iter()
        .map(|h| h.join().unwrap())
        .collect();

    println!("{:?}", results);  // [0, 1, 4, 9]
}
```

---

## Section C

### A7
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | When `main` returns, spawned threads are terminated |
| 2 | **True** | `join()` blocks until the thread finishes |
| 3 | **False** | `Rc` is not `Send` — use `Arc` for threads |
| 4 | **True** | Scoped threads auto-join at the end of the scope |
| 5 | **True** | `move` transfers ownership to the closure |
| 6 | **True** | `JoinHandle<T>` carries the return type of the closure |

### A8
`thread::spawn` creates a thread with a `'static` lifetime — it could run forever. The compiler can't guarantee borrowed data lives long enough, so `move` is required to transfer ownership.

`thread::scope` guarantees all threads complete before the scope exits. The compiler knows the data will outlive the threads, so borrowing is safe without `move`.

---

## 🏆 Lesson 54 Complete!

**Next up:** [Lesson 55 — Channels (mpsc)](../lesson_55_channels/lesson_55_channels.md) 🦀
