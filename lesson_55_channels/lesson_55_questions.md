# 🧪 Lesson 55 — Questions: Channels (CC2)

> **Lesson:** [lesson_55_channels.md](./lesson_55_channels.md)  
> **Answers:** [lesson_55_answers.md](./lesson_55_answers.md)

---

## Section A — Predict: What Happens?

### Q1
```rust
use std::sync::mpsc;
fn main() {
    let (tx, rx) = mpsc::channel();
    tx.send(42).unwrap();
    println!("{}", rx.recv().unwrap());
}
```

### Q2
```rust
use std::sync::mpsc;
use std::thread;
fn main() {
    let (tx, rx) = mpsc::channel();
    thread::spawn(move || {
        let msg = String::from("hello");
        tx.send(msg).unwrap();
        println!("{msg}");
    });
    println!("{}", rx.recv().unwrap());
}
```

### Q3
```rust
use std::sync::mpsc;
fn main() {
    let (tx, rx) = mpsc::channel::<i32>();
    drop(tx);
    println!("{:?}", rx.recv());
}
```

---

## Section B — Write It Yourself

### Q4 — Multi-producer logging (Roadmap Practice Task)
Create 3 producer threads that each send 3 log messages (with their thread ID). A single consumer prints all messages. Ensure the channel closes when all producers finish.

### Q5 — Worker pool
Send 10 tasks (numbers) through a channel. Have 3 worker threads each receive tasks and compute their squares. Collect results.

### Q6 — Ping-pong
Create two threads that pass a message back and forth 5 times using two channels.

---

## Section C — True or False?

### Q7
1. `mpsc` stands for "multiple producer, single consumer."
2. `send()` copies the value to the receiver.
3. `rx.recv()` blocks until a message arrives or the channel closes.
4. Dropping all senders closes the channel.
5. You can have multiple receivers with `mpsc`.
6. `sync_channel(0)` means every `send` blocks until a `recv` is ready.
7. Iterating over `rx` ends when the channel closes.

### Q8
Explain the difference between `mpsc::channel()` and `mpsc::sync_channel(n)`. When would you use each?

---

*Channels: threads talking to each other, safely! 🦀*
