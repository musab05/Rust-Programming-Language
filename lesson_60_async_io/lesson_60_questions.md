# 🧪 Lesson 60 — Questions: Async I/O (AS3)

> **Lesson:** [lesson_60_async_io.md](./lesson_60_async_io.md)  
> **Answers:** [lesson_60_answers.md](./lesson_60_answers.md)

---

## Section A — Conceptual

### Q1
What is the difference between `std::fs::read_to_string` and `tokio::fs::read_to_string`?

### Q2
Why does the echo server use `tokio::spawn` for each connection?

### Q3
What is `tokio::io::split` used for?

---

## Section B — Write It Yourself

### Q4 — TCP echo server (Roadmap Practice Task)
Write a basic TCP echo server that accepts connections and echoes back each line the client sends, prefixed with "Echo: ". Use `BufReader` for line-by-line reading.

### Q5 — Concurrent file reader
Write an async function that reads 3 files concurrently using `tokio::join!` and prints their contents.

### Q6 — Async HTTP fetcher
Using `reqwest`, fetch 3 URLs concurrently and print the status code and body length of each.

---

## Section C — True or False?

### Q7
1. `tokio::fs` functions are non-blocking alternatives to `std::fs`.
2. `TcpListener::accept()` blocks the OS thread until a connection arrives.
3. `io::copy` pipes data from a reader to a writer asynchronously.
4. `BufReader` reduces the number of system calls by reading in chunks.
5. Each `tokio::spawn` creates a new OS thread.
6. `broadcast::channel` allows multiple consumers to receive the same messages.

### Q8
Explain how a chat server can handle thousands of simultaneous clients with just a few OS threads.

---

*60 lessons complete — you're an advanced Rustacean now! 🦀🎉*
