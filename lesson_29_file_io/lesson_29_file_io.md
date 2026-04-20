# 📘 Lesson 29 — File I/O & std::io (RW2)

> **Series:** Rust From Zero · Beginner Level  
> **Roadmap ID:** RW2 · Category: 🌍 Real World  
> **Previous:** [Lesson 28 — Clippy & rustfmt](../lesson_28_clippy_rustfmt/lesson_28_clippy_rustfmt.md)  
> **Next:** [Lesson 30 — Lifetimes Basics](../lesson_30_lifetimes/lesson_30_lifetimes.md)  
> **Practice:** [Questions](./lesson_29_questions.md) · [Answers](./lesson_29_answers.md)  
> **Practice Task:** CSV parser that reads and aggregates data

---

## Table of Contents

1. [Reading Files](#1-reading-files)
2. [Writing Files](#2-writing-files)
3. [Appending to Files](#3-appending-to-files)
4. [Buffered I/O](#4-buffered-io)
5. [Reading Line by Line](#5-reading-line-by-line)
6. [stdin & stdout](#6-stdin--stdout)
7. [File Metadata & Paths](#7-file-metadata--paths)
8. [Directory Operations](#8-directory-operations)
9. [Error Handling for I/O](#9-error-handling-for-io)
10. [Real-World Example: CSV Parser](#10-real-world-example-csv-parser)
11. [Summary Cheat Sheet](#11-summary-cheat-sheet)

---

## 1. Reading Files

### The simple way — `fs::read_to_string`

```rust
use std::fs;

fn main() {
    match fs::read_to_string("hello.txt") {
        Ok(contents) => println!("{contents}"),
        Err(e) => println!("Error: {e}"),
    }
}
```

### Read as bytes — `fs::read`

```rust
use std::fs;

fn main() {
    match fs::read("image.png") {
        Ok(bytes) => println!("Read {} bytes", bytes.len()),
        Err(e) => println!("Error: {e}"),
    }
}
```

### Using `File::open` for more control

```rust
use std::fs::File;
use std::io::Read;

fn main() -> std::io::Result<()> {
    let mut file = File::open("hello.txt")?;
    let mut contents = String::new();
    file.read_to_string(&mut contents)?;
    println!("{contents}");
    Ok(())
}
```

---

## 2. Writing Files

### The simple way — `fs::write`

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    fs::write("output.txt", "Hello, Rust!\nSecond line.")?;
    println!("File written!");
    Ok(())
}
```

### Using `File::create` for more control

```rust
use std::fs::File;
use std::io::Write;

fn main() -> std::io::Result<()> {
    let mut file = File::create("output.txt")?;
    writeln!(file, "Name: Alice")?;
    writeln!(file, "Age: 30")?;
    writeln!(file, "Language: Rust")?;
    println!("File written!");
    Ok(())
}
```

> `File::create` **overwrites** the file if it exists.

### Using `write!` and `writeln!`

```rust
use std::fs::File;
use std::io::Write;

fn main() -> std::io::Result<()> {
    let mut file = File::create("report.txt")?;

    writeln!(file, "=== Report ===")?;
    for i in 1..=5 {
        writeln!(file, "Item {i}: value = {}", i * 10)?;
    }
    write!(file, "Total: {}", (1..=5).map(|i| i * 10).sum::<i32>())?;

    println!("Report written!");
    Ok(())
}
```

---

## 3. Appending to Files

Use `OpenOptions` to append instead of overwriting:

```rust
use std::fs::OpenOptions;
use std::io::Write;

fn main() -> std::io::Result<()> {
    let mut file = OpenOptions::new()
        .create(true)   // create if doesn't exist
        .append(true)   // append instead of overwrite
        .open("log.txt")?;

    writeln!(file, "[{}] Application started", "2024-01-15 10:30:00")?;
    writeln!(file, "[{}] Processing data...", "2024-01-15 10:30:01")?;

    println!("Log entries appended!");
    Ok(())
}
```

### `OpenOptions` builder methods:

| Method | Effect |
|---|---|
| `.read(true)` | Open for reading |
| `.write(true)` | Open for writing |
| `.append(true)` | Append instead of overwrite |
| `.create(true)` | Create file if it doesn't exist |
| `.truncate(true)` | Clear file contents on open |
| `.create_new(true)` | Fail if file already exists |

---

## 4. Buffered I/O

Direct file I/O makes a system call for every read/write. **Buffered I/O** batches operations for better performance.

### `BufReader` — buffered reading

```rust
use std::fs::File;
use std::io::{BufRead, BufReader};

fn main() -> std::io::Result<()> {
    let file = File::open("data.txt")?;
    let reader = BufReader::new(file);

    for line in reader.lines() {
        let line = line?;
        println!("{line}");
    }

    Ok(())
}
```

### `BufWriter` — buffered writing

```rust
use std::fs::File;
use std::io::{BufWriter, Write};

fn main() -> std::io::Result<()> {
    let file = File::create("big_output.txt")?;
    let mut writer = BufWriter::new(file);

    for i in 0..100_000 {
        writeln!(writer, "Line {i}: {}", i * i)?;
    }

    // Flush is called automatically when writer is dropped,
    // but you can call it explicitly:
    writer.flush()?;

    println!("Wrote 100,000 lines!");
    Ok(())
}
```

### When to use buffered I/O:
- **Always for line-by-line reading** — `BufReader` enables `.lines()`
- **Writing many small writes** — `BufWriter` batches them
- **Large files** — reduces system call overhead
- `fs::read_to_string` / `fs::write` are already efficient for simple cases

---

## 5. Reading Line by Line

```rust
use std::fs::File;
use std::io::{BufRead, BufReader};

fn main() -> std::io::Result<()> {
    let file = File::open("data.txt")?;
    let reader = BufReader::new(file);

    for (line_num, line) in reader.lines().enumerate() {
        let line = line?;
        println!("{:4}: {line}", line_num + 1);
    }

    Ok(())
}
```

### Processing with filtering:

```rust
use std::fs::File;
use std::io::{BufRead, BufReader};

fn main() -> std::io::Result<()> {
    let file = File::open("Cargo.toml")?;
    let reader = BufReader::new(file);

    let non_empty_lines: Vec<String> = reader
        .lines()
        .filter_map(|line| line.ok())
        .filter(|line| !line.trim().is_empty())
        .collect();

    println!("Non-empty lines: {}", non_empty_lines.len());
    for line in &non_empty_lines[..5.min(non_empty_lines.len())] {
        println!("  {line}");
    }

    Ok(())
}
```

---

## 6. stdin & stdout

### Reading from stdin

```rust
use std::io;

fn main() {
    println!("What's your name?");

    let mut input = String::new();
    io::stdin()
        .read_line(&mut input)
        .expect("Failed to read line");

    let name = input.trim();
    println!("Hello, {name}!");
}
```

### Reading multiple values:

```rust
use std::io::{self, BufRead};

fn main() {
    println!("Enter numbers (one per line, empty line to stop):");

    let stdin = io::stdin();
    let mut numbers: Vec<f64> = Vec::new();

    for line in stdin.lock().lines() {
        let line = line.expect("Failed to read");
        let trimmed = line.trim();
        if trimmed.is_empty() {
            break;
        }
        match trimmed.parse::<f64>() {
            Ok(n) => numbers.push(n),
            Err(_) => println!("  '{trimmed}' is not a number, skipping"),
        }
    }

    if numbers.is_empty() {
        println!("No numbers entered.");
    } else {
        let sum: f64 = numbers.iter().sum();
        let avg = sum / numbers.len() as f64;
        println!("Count: {}, Sum: {sum:.2}, Average: {avg:.2}", numbers.len());
    }
}
```

### Writing to stdout/stderr:

```rust
use std::io::{self, Write};

fn main() {
    // print! and println! go to stdout
    println!("This goes to stdout");

    // eprintln! goes to stderr
    eprintln!("This goes to stderr");

    // Manual stdout write (for performance in loops)
    let stdout = io::stdout();
    let mut handle = stdout.lock();
    for i in 0..5 {
        writeln!(handle, "Line {i}").unwrap();
    }
}
```

---

## 7. File Metadata & Paths

```rust
use std::fs;
use std::path::Path;

fn main() -> std::io::Result<()> {
    let path = Path::new("Cargo.toml");

    // Check existence
    println!("Exists: {}", path.exists());
    println!("Is file: {}", path.is_file());
    println!("Is dir: {}", path.is_dir());

    // File metadata
    let metadata = fs::metadata(path)?;
    println!("Size: {} bytes", metadata.len());
    println!("Read-only: {}", metadata.permissions().readonly());
    println!("Modified: {:?}", metadata.modified()?);

    // Path manipulation
    let full = Path::new("/home/user/project/src/main.rs");
    println!("File name: {:?}", full.file_name());        // Some("main.rs")
    println!("Extension: {:?}", full.extension());         // Some("rs")
    println!("Parent: {:?}", full.parent());               // Some("/home/user/project/src")
    println!("Stem: {:?}", full.file_stem());              // Some("main")

    // Build paths
    let mut path = std::path::PathBuf::from("/home/user");
    path.push("project");
    path.push("src");
    path.push("main.rs");
    println!("Built: {}", path.display());  // /home/user/project/src/main.rs

    Ok(())
}
```

---

## 8. Directory Operations

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    // Create directory
    fs::create_dir("new_dir")?;
    // Create nested directories
    fs::create_dir_all("parent/child/grandchild")?;

    // List directory contents
    for entry in fs::read_dir(".")? {
        let entry = entry?;
        let path = entry.path();
        let file_type = if path.is_dir() { "DIR " } else { "FILE" };
        println!("{file_type} {}", path.display());
    }

    // Remove file
    // fs::remove_file("temp.txt")?;

    // Remove directory (must be empty)
    // fs::remove_dir("empty_dir")?;

    // Remove directory and all contents
    // fs::remove_dir_all("some_dir")?;

    // Copy and rename
    // fs::copy("src.txt", "dst.txt")?;
    // fs::rename("old.txt", "new.txt")?;

    // Cleanup
    fs::remove_dir_all("new_dir")?;
    fs::remove_dir_all("parent")?;

    Ok(())
}
```

---

## 9. Error Handling for I/O

All I/O operations return `io::Result<T>` which is `Result<T, io::Error>`:

```rust
use std::fs::File;
use std::io::{self, ErrorKind};

fn main() {
    match File::open("nonexistent.txt") {
        Ok(file) => println!("Opened: {file:?}"),
        Err(e) => {
            match e.kind() {
                ErrorKind::NotFound => println!("File not found!"),
                ErrorKind::PermissionDenied => println!("No permission!"),
                _ => println!("Other error: {e}"),
            }
        }
    }
}
```

### Common `ErrorKind` variants:

| Variant | Meaning |
|---|---|
| `NotFound` | File/directory doesn't exist |
| `PermissionDenied` | No access permission |
| `AlreadyExists` | File already exists (with `create_new`) |
| `InvalidInput` | Invalid argument |
| `UnexpectedEof` | Unexpected end of file |
| `WouldBlock` | Operation would block (non-blocking I/O) |

---

## 10. Real-World Example: CSV Parser

This is the roadmap practice task:

```rust
use std::collections::HashMap;
use std::error::Error;
use std::fs::File;
use std::io::{BufRead, BufReader, Write};

#[derive(Debug)]
struct Record {
    name: String,
    department: String,
    salary: f64,
}

fn parse_csv(path: &str) -> Result<Vec<Record>, Box<dyn Error>> {
    let file = File::open(path)?;
    let reader = BufReader::new(file);
    let mut records = Vec::new();

    for (i, line) in reader.lines().enumerate() {
        let line = line?;
        if i == 0 { continue; }  // skip header

        let fields: Vec<&str> = line.split(',').collect();
        if fields.len() != 3 {
            eprintln!("Warning: skipping malformed line {}: '{line}'", i + 1);
            continue;
        }

        let salary: f64 = fields[2].trim().parse()?;
        records.push(Record {
            name: fields[0].trim().to_string(),
            department: fields[1].trim().to_string(),
            salary,
        });
    }

    Ok(records)
}

fn aggregate(records: &[Record]) {
    // Total and average salary
    let total: f64 = records.iter().map(|r| r.salary).sum();
    let avg = total / records.len() as f64;
    println!("Total salary:   ${total:.2}");
    println!("Average salary: ${avg:.2}");
    println!("Employees:      {}\n", records.len());

    // Per-department stats
    let mut dept_totals: HashMap<&str, (f64, usize)> = HashMap::new();
    for r in records {
        let entry = dept_totals.entry(&r.department).or_insert((0.0, 0));
        entry.0 += r.salary;
        entry.1 += 1;
    }

    println!("=== By Department ===");
    let mut depts: Vec<(&str, f64, usize)> = dept_totals
        .iter()
        .map(|(dept, (total, count))| (*dept, *total, *count))
        .collect();
    depts.sort_by(|a, b| b.1.partial_cmp(&a.1).unwrap());

    for (dept, total, count) in &depts {
        let avg = total / *count as f64;
        println!("  {dept}: {count} employees, total ${total:.2}, avg ${avg:.2}");
    }

    // Highest paid
    if let Some(top) = records.iter().max_by(|a, b| a.salary.partial_cmp(&b.salary).unwrap()) {
        println!("\nHighest paid: {} (${:.2})", top.name, top.salary);
    }
}

fn create_sample_csv(path: &str) -> std::io::Result<()> {
    let mut file = File::create(path)?;
    writeln!(file, "name,department,salary")?;
    writeln!(file, "Alice,Engineering,95000")?;
    writeln!(file, "Bob,Marketing,72000")?;
    writeln!(file, "Cara,Engineering,88000")?;
    writeln!(file, "Dave,Sales,65000")?;
    writeln!(file, "Eve,Marketing,78000")?;
    writeln!(file, "Frank,Engineering,102000")?;
    writeln!(file, "Grace,Sales,71000")?;
    writeln!(file, "Hank,Marketing,69000")?;
    Ok(())
}

fn main() -> Result<(), Box<dyn Error>> {
    let csv_path = "employees.csv";

    // Create sample data
    create_sample_csv(csv_path)?;
    println!("Created {csv_path}\n");

    // Parse and aggregate
    let records = parse_csv(csv_path)?;
    aggregate(&records);

    // Cleanup
    std::fs::remove_file(csv_path)?;

    Ok(())
}
```

---

## 11. Summary Cheat Sheet

```
READING
────────────────────────────────────────────────────────────
fs::read_to_string(path)       → Result<String>  (entire file)
fs::read(path)                 → Result<Vec<u8>> (bytes)
File::open(path)               → Result<File>    (handle)
BufReader::new(file).lines()   → Iterator<Result<String>>

WRITING
────────────────────────────────────────────────────────────
fs::write(path, data)          overwrite entire file
File::create(path)             create/overwrite, returns File
writeln!(file, "text")         write line to file
BufWriter::new(file)           buffered writing

APPENDING
────────────────────────────────────────────────────────────
OpenOptions::new().append(true).create(true).open(path)

STDIN / STDOUT
────────────────────────────────────────────────────────────
io::stdin().read_line(&mut s)  read one line
stdin.lock().lines()           iterate lines
println!() / print!()         stdout
eprintln!() / eprint!()       stderr

PATHS
────────────────────────────────────────────────────────────
Path::new("file.txt")         immutable path reference
PathBuf::from("/dir")         owned, mutable path
path.exists() / is_file() / is_dir()
path.file_name() / extension() / parent()

DIRECTORIES
────────────────────────────────────────────────────────────
fs::create_dir(path)          create one directory
fs::create_dir_all(path)      create nested directories
fs::read_dir(path)            list directory entries
fs::remove_file(path)         delete a file
fs::remove_dir_all(path)      delete dir recursively
fs::copy(src, dst)            copy a file
fs::rename(old, new)          rename/move

ERROR HANDLING
────────────────────────────────────────────────────────────
io::Result<T> = Result<T, io::Error>
error.kind()  → ErrorKind (NotFound, PermissionDenied, etc.)
```

---

## What's Next?

**Lesson 30 — Lifetimes Basics** — The final piece of the ownership puzzle. Learn lifetime annotations `'a`, why they're needed, and the elision rules.

## Further Reading
- [The Rust Book — Appendix: std::io](https://doc.rust-lang.org/std/io/index.html)
- [std::fs module](https://doc.rust-lang.org/std/fs/index.html)
- [std::path module](https://doc.rust-lang.org/std/path/index.html)

---

*I/O is where programs meet the real world! 🦀*
