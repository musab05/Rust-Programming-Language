# ✅ Lesson 60 — Answers: Async I/O (AS3)

---

## Section A

### A1
`std::fs::read_to_string` **blocks** the current thread while reading. `tokio::fs::read_to_string` is async — it **yields** to the runtime while the OS reads the file, allowing other tasks to run on the same thread.

### A2
Each connection is handled in its own `tokio::spawn` task so the main loop can continue accepting new connections immediately. Without spawning, the server would handle one client at a time (blocking on each).

### A3
`tokio::io::split` splits a `TcpStream` into an `OwnedReadHalf` and `OwnedWriteHalf`, allowing separate tasks to read and write concurrently on the same connection (essential for the chat server pattern with `select!`).

---

## Section B

### A4
```rust
use tokio::net::TcpListener;
use tokio::io::{AsyncBufReadExt, AsyncWriteExt, BufReader};

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let listener = TcpListener::bind("127.0.0.1:8080").await?;
    println!("Echo server on :8080");

    loop {
        let (socket, addr) = listener.accept().await?;
        tokio::spawn(async move {
            let (reader, mut writer) = socket.into_split();
            let mut reader = BufReader::new(reader);
            let mut line = String::new();

            loop {
                line.clear();
                match reader.read_line(&mut line).await {
                    Ok(0) => break,
                    Ok(_) => {
                        let response = format!("Echo: {}", line);
                        writer.write_all(response.as_bytes()).await.unwrap();
                    }
                    Err(_) => break,
                }
            }
            println!("{addr} disconnected");
        });
    }
}
```

### A5
```rust
use tokio::fs;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Create test files
    fs::write("a.txt", "File A content").await?;
    fs::write("b.txt", "File B content").await?;
    fs::write("c.txt", "File C content").await?;

    // Read concurrently
    let (a, b, c) = tokio::join!(
        fs::read_to_string("a.txt"),
        fs::read_to_string("b.txt"),
        fs::read_to_string("c.txt"),
    );

    println!("A: {}", a?);
    println!("B: {}", b?);
    println!("C: {}", c?);

    // Cleanup
    fs::remove_file("a.txt").await?;
    fs::remove_file("b.txt").await?;
    fs::remove_file("c.txt").await?;
    Ok(())
}
```

### A6
```rust
// Cargo.toml: reqwest = { version = "0.12" }

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let urls = vec![
        "https://httpbin.org/get",
        "https://httpbin.org/status/200",
        "https://httpbin.org/ip",
    ];

    let mut handles = vec![];
    for url in urls {
        handles.push(tokio::spawn(async move {
            let resp = reqwest::get(url).await?;
            let status = resp.status();
            let body = resp.text().await?;
            Ok::<_, reqwest::Error>((url, status, body.len()))
        }));
    }

    for handle in handles {
        let (url, status, len) = handle.await??;
        println!("{url}: {status} ({len} bytes)");
    }
    Ok(())
}
```

---

## Section C

### A7
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | `tokio::fs` uses a thread pool internally to avoid blocking the async runtime |
| 2 | **False** | `TcpListener::accept()` is async — it yields, not blocks |
| 3 | **True** | `io::copy` asynchronously pipes all bytes from reader to writer |
| 4 | **True** | `BufReader` accumulates small reads into larger chunks |
| 5 | **False** | `tokio::spawn` creates a lightweight **task**, not a thread |
| 6 | **True** | Broadcast channels clone each message to all active subscribers |

### A8
The chat server uses **async tasks** instead of threads. Each client connection is a lightweight task (~hundreds of bytes) scheduled on a small thread pool (e.g., 4 threads). When a task waits for I/O (reading from a socket), it **yields** to the runtime, which runs another task on the same thread. This means 4 OS threads can handle thousands of connections efficiently, since most connections are idle (waiting for input) at any given time. The runtime only wakes a task when its socket has data to read.

---

## 🏆 Lesson 60 Complete — Congratulations! 🎉

✅ Async file I/O  
✅ TCP networking (client & server)  
✅ Echo server pattern  
✅ AsyncRead/AsyncWrite traits  
✅ Buffered I/O  
✅ HTTP with reqwest  
✅ Chat server with broadcast  

**You've completed 60 Rust lessons!** 🦀  
Next: Unsafe Rust, Macros, FFI, and beyond!
