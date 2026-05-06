# 📘 Lesson 60 — Async I/O (AS3)

> **Series:** Rust From Zero · Advanced Level  
> **Roadmap ID:** AS3 · Category: 🌐 Async  
> **Previous:** [Lesson 59 — Tokio Runtime](../lesson_59_tokio/lesson_59_tokio.md)  
> **Next:** [Lesson 61 — Unsafe Rust](../lesson_61_unsafe/lesson_61_unsafe.md)  
> **Practice:** [Questions](./lesson_60_questions.md) · [Answers](./lesson_60_answers.md)  
> **Practice Task:** Build a TCP echo server with async I/O

---

## Table of Contents

1. [Async I/O in Rust](#1-async-io-in-rust)
2. [Async File I/O](#2-async-file-io)
3. [TCP — Async Networking](#3-tcp--async-networking)
4. [TCP Echo Server](#4-tcp-echo-server)
5. [Handling Multiple Connections](#5-handling-multiple-connections)
6. [AsyncRead and AsyncWrite](#6-asyncread-and-asyncwrite)
7. [Buffered I/O](#7-buffered-io)
8. [HTTP with reqwest](#8-http-with-reqwest)
9. [Real-World Example: Chat Server](#9-real-world-example-chat-server)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. Async I/O in Rust

Synchronous I/O blocks the thread. Async I/O yields to the runtime while waiting:

```
Sync I/O:
Thread: [read(file1)████████][read(file2)████████]  16ms total

Async I/O:
Task 1: [read(file1)···await···████]                 8ms total
Task 2:         [read(file2)···await···████]
Both run on ONE thread!
```

---

## 2. Async File I/O

```rust
use tokio::fs;
use tokio::io::{self, AsyncWriteExt, AsyncReadExt};

#[tokio::main]
async fn main() -> io::Result<()> {
    // Write a file
    fs::write("hello.txt", "Hello, async Rust!").await?;
    println!("File written");

    // Read a file
    let contents = fs::read_to_string("hello.txt").await?;
    println!("Contents: {contents}");

    // Append to a file
    let mut file = fs::OpenOptions::new()
        .append(true)
        .open("hello.txt")
        .await?;
    file.write_all(b"\nAppended line").await?;
    println!("Appended");

    // Read as bytes
    let bytes = fs::read("hello.txt").await?;
    println!("Size: {} bytes", bytes.len());

    // Read directory
    let mut entries = fs::read_dir(".").await?;
    while let Some(entry) = entries.next_entry().await? {
        println!("  {:?}", entry.file_name());
    }

    // Cleanup
    fs::remove_file("hello.txt").await?;

    Ok(())
}
```

---

## 3. TCP — Async Networking

### TCP Client:

```rust
use tokio::net::TcpStream;
use tokio::io::{AsyncWriteExt, AsyncReadExt};

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Connect to a server
    let mut stream = TcpStream::connect("127.0.0.1:8080").await?;
    println!("Connected to server");

    // Send data
    stream.write_all(b"Hello, server!").await?;

    // Read response
    let mut buf = vec![0u8; 1024];
    let n = stream.read(&mut buf).await?;
    println!("Response: {}", String::from_utf8_lossy(&buf[..n]));

    Ok(())
}
```

### TCP Listener:

```rust
use tokio::net::TcpListener;
use tokio::io::{AsyncReadExt, AsyncWriteExt};

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let listener = TcpListener::bind("127.0.0.1:8080").await?;
    println!("Server listening on :8080");

    loop {
        let (mut socket, addr) = listener.accept().await?;
        println!("Client connected: {addr}");

        // Handle each connection in a separate task
        tokio::spawn(async move {
            let mut buf = vec![0u8; 1024];
            match socket.read(&mut buf).await {
                Ok(n) if n > 0 => {
                    let msg = String::from_utf8_lossy(&buf[..n]);
                    println!("Received from {addr}: {msg}");
                    socket.write_all(b"Message received!").await.unwrap();
                }
                _ => println!("Client {addr} disconnected"),
            }
        });
    }
}
```

---

## 4. TCP Echo Server

A classic — sends back whatever the client sends:

```rust
use tokio::net::TcpListener;
use tokio::io::{AsyncReadExt, AsyncWriteExt};

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let listener = TcpListener::bind("127.0.0.1:8080").await?;
    println!("🔊 Echo server on :8080");

    loop {
        let (mut socket, addr) = listener.accept().await?;
        println!("📥 Client: {addr}");

        tokio::spawn(async move {
            let mut buf = vec![0u8; 1024];

            loop {
                match socket.read(&mut buf).await {
                    Ok(0) => {
                        println!("📤 {addr} disconnected");
                        break;
                    }
                    Ok(n) => {
                        // Echo back what we received
                        if socket.write_all(&buf[..n]).await.is_err() {
                            break;
                        }
                    }
                    Err(e) => {
                        eprintln!("Error from {addr}: {e}");
                        break;
                    }
                }
            }
        });
    }
}
```

---

## 5. Handling Multiple Connections

Using `tokio::io::copy` for the echo pattern:

```rust
use tokio::net::TcpListener;
use tokio::io;

#[tokio::main]
async fn main() -> io::Result<()> {
    let listener = TcpListener::bind("127.0.0.1:8080").await?;
    println!("Echo server on :8080");

    loop {
        let (socket, addr) = listener.accept().await?;
        println!("New connection: {addr}");

        tokio::spawn(async move {
            let (mut reader, mut writer) = io::split(socket);
            match io::copy(&mut reader, &mut writer).await {
                Ok(bytes) => println!("{addr} echoed {bytes} bytes"),
                Err(e) => eprintln!("{addr} error: {e}"),
            }
        });
    }
}
```

---

## 6. AsyncRead and AsyncWrite

Tokio's async versions of `std::io::Read/Write`:

```rust
use tokio::io::{self, AsyncReadExt, AsyncWriteExt, AsyncBufReadExt, BufReader};
use tokio::net::TcpStream;

async fn process_stream(stream: TcpStream) -> io::Result<()> {
    let (reader, mut writer) = io::split(stream);
    let mut reader = BufReader::new(reader);
    let mut line = String::new();

    // Read line by line
    while reader.read_line(&mut line).await? > 0 {
        let response = format!("Echo: {}", line.trim());
        writer.write_all(response.as_bytes()).await?;
        writer.write_all(b"\n").await?;
        line.clear();
    }

    Ok(())
}
```

---

## 7. Buffered I/O

```rust
use tokio::io::{self, AsyncBufReadExt, BufReader, BufWriter, AsyncWriteExt};
use tokio::fs::File;

#[tokio::main]
async fn main() -> io::Result<()> {
    // Buffered reader — read line by line
    let file = File::open("Cargo.toml").await?;
    let reader = BufReader::new(file);
    let mut lines = reader.lines();

    println!("First 5 lines of Cargo.toml:");
    let mut count = 0;
    while let Some(line) = lines.next_line().await? {
        println!("  {line}");
        count += 1;
        if count >= 5 { break; }
    }

    // Buffered writer — batch writes
    let file = File::create("output.txt").await?;
    let mut writer = BufWriter::new(file);

    for i in 0..100 {
        writer.write_all(format!("Line {i}\n").as_bytes()).await?;
    }
    writer.flush().await?;  // ensure all data is written
    println!("Wrote 100 lines to output.txt");

    tokio::fs::remove_file("output.txt").await?;

    Ok(())
}
```

---

## 8. HTTP with reqwest

For real HTTP requests, use the `reqwest` crate:

```toml
[dependencies]
tokio = { version = "1", features = ["full"] }
reqwest = { version = "0.12", features = ["json"] }
serde = { version = "1", features = ["derive"] }
serde_json = "1"
```

```rust
use serde::Deserialize;

#[derive(Debug, Deserialize)]
struct Post {
    id: u32,
    title: String,
    body: String,
}

#[tokio::main]
async fn main() -> Result<(), reqwest::Error> {
    // Simple GET
    let body = reqwest::get("https://httpbin.org/get")
        .await?
        .text()
        .await?;
    println!("Response length: {} chars", body.len());

    // JSON deserialization
    let post: Post = reqwest::get("https://jsonplaceholder.typicode.com/posts/1")
        .await?
        .json()
        .await?;
    println!("Post: {} - {}", post.id, post.title);

    // Concurrent requests
    let urls = vec![
        "https://jsonplaceholder.typicode.com/posts/1",
        "https://jsonplaceholder.typicode.com/posts/2",
        "https://jsonplaceholder.typicode.com/posts/3",
    ];

    let mut handles = vec![];
    for url in urls {
        handles.push(tokio::spawn(async move {
            reqwest::get(url).await?.json::<Post>().await
        }));
    }

    for handle in handles {
        match handle.await.unwrap() {
            Ok(post) => println!("  #{}: {}", post.id, post.title),
            Err(e) => eprintln!("  Error: {e}"),
        }
    }

    Ok(())
}
```

---

## 9. Real-World Example: Chat Server

```rust
use tokio::net::TcpListener;
use tokio::io::{AsyncBufReadExt, AsyncWriteExt, BufReader};
use tokio::sync::broadcast;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let listener = TcpListener::bind("127.0.0.1:8080").await?;
    let (tx, _) = broadcast::channel::<String>(100);

    println!("💬 Chat server on :8080");

    loop {
        let (socket, addr) = listener.accept().await?;
        let tx = tx.clone();
        let mut rx = tx.subscribe();

        tokio::spawn(async move {
            let (reader, mut writer) = socket.into_split();
            let mut reader = BufReader::new(reader);
            let mut line = String::new();

            writer.write_all(format!("Welcome {addr}! Type messages:\n").as_bytes())
                .await.unwrap();

            loop {
                tokio::select! {
                    // Read from this client
                    result = reader.read_line(&mut line) => {
                        match result {
                            Ok(0) => {
                                tx.send(format!("{addr} left the chat")).ok();
                                break;
                            }
                            Ok(_) => {
                                let msg = format!("[{addr}] {}", line.trim());
                                tx.send(msg).ok();
                                line.clear();
                            }
                            Err(_) => break,
                        }
                    }
                    // Write broadcasts from other clients
                    result = rx.recv() => {
                        if let Ok(msg) = result {
                            let formatted = format!("{msg}\n");
                            if writer.write_all(formatted.as_bytes()).await.is_err() {
                                break;
                            }
                        }
                    }
                }
            }
        });
    }
}
```

---

## 10. Summary Cheat Sheet

```
ASYNC FILE I/O
────────────────────────────────────────────────────────────
tokio::fs::read_to_string(path)    read file
tokio::fs::write(path, data)       write file
tokio::fs::read_dir(path)          list directory

ASYNC NETWORKING
────────────────────────────────────────────────────────────
TcpListener::bind(addr)            listen for connections
listener.accept()                  accept connection
TcpStream::connect(addr)           connect to server

READING & WRITING
────────────────────────────────────────────────────────────
stream.read(&mut buf)              read bytes
stream.write_all(bytes)            write all bytes
io::copy(&mut reader, &mut writer) pipe data
BufReader + .lines()               line-by-line reading

HTTP (reqwest)
────────────────────────────────────────────────────────────
reqwest::get(url).await?            GET request
.text().await?                      as String
.json::<T>().await?                 deserialize JSON

PATTERNS
────────────────────────────────────────────────────────────
Echo server     read → write back
Chat server     broadcast channel + select!
HTTP fetcher    concurrent reqwest tasks
```

---

## 🎉 You've Completed 60 Lessons!

Congratulations on reaching the **Advanced** tier! You now have:

- 🧠 **Ownership & Lifetimes**: Full mastery
- 📚 **Collections & Iterators**: Fluent pipeline processing
- ⚠ **Error Handling**: Custom errors, anyhow, thiserror
- 🔷 **Traits & Generics**: Associated types, operator overloading
- 📦 **Modules**: Workspaces, publishing
- 🔒 **Closures**: Fn traits, HOFs, composition
- 📌 **Smart Pointers**: Box, Rc, Arc, RefCell
- ⚡ **Concurrency**: Threads, channels, Mutex, atomics
- 🌐 **Async**: Futures, Tokio, async I/O

**Coming next**: Unsafe Rust, Macros, FFI, and advanced patterns! 🦀

## Further Reading
- [Tokio Tutorial — I/O](https://tokio.rs/tokio/tutorial/io)
- [reqwest docs](https://docs.rs/reqwest/)
- [Tokio Mini-Redis Tutorial](https://tokio.rs/tokio/tutorial/setup)

---

*Async I/O: reading and writing without blocking — the foundation of modern Rust servers! 🦀*
