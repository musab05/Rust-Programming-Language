# 🧪 Lesson 84 — Questions: Async Streams & Channels (CC8)

> **Lesson:** [lesson_84_async_streams.md](./lesson_84_async_streams.md)  
> **Answers:** [lesson_84_answers.md](./lesson_84_answers.md)

---

## Section A — Conceptual

### Q1
What's the difference between `mpsc`, `broadcast`, and `watch` channels?

### Q2
When would you use `oneshot` instead of `mpsc`?

---

## Section B — Write It Yourself

### Q3 — Log aggregator (Roadmap Practice Task)
Create 3 async producer tasks that send log messages through an `mpsc` channel. Write a consumer that prints all logs with timestamps.

### Q4 — Broadcast events
Create a broadcast channel. Spawn 2 subscribers. Publish 5 events. Each subscriber should print all events.

---

## Section C — True or False?

### Q5
1. `mpsc::channel` supports multiple senders and a single receiver.
2. `broadcast` delivers each message to exactly one subscriber.
3. `watch` only stores the latest value — subscribers may miss intermediate values.
4. `oneshot::channel` can send multiple values.
5. Streams in Rust are the async equivalent of iterators.
6. Dropping all senders causes `rx.recv().await` to return `None`.

---

*Async channels: the nervous system of concurrent Rust! 🦀*
