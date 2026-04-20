# 🧪 Lesson 29 — Questions: File I/O & std::io (RW2)

> **Lesson:** [lesson_29_file_io.md](./lesson_29_file_io.md)  
> **Answers:** [lesson_29_answers.md](./lesson_29_answers.md)

---

## Section A — Predict the Output

### Q1
```rust
use std::fs;
fn main() {
    fs::write("test.txt", "Hello\nWorld\nRust").unwrap();
    let contents = fs::read_to_string("test.txt").unwrap();
    println!("{}", contents.lines().count());
    println!("{}", contents.len());
    fs::remove_file("test.txt").unwrap();
}
```

### Q2
```rust
use std::path::Path;
fn main() {
    let p = Path::new("/home/user/project/src/main.rs");
    println!("{:?}", p.file_name().unwrap());
    println!("{:?}", p.extension().unwrap());
    println!("{:?}", p.parent().unwrap());
    println!("{}", p.exists());
}
```

---

## Section B — Will It Compile?

### Q3
```rust
use std::fs::File;
fn main() {
    let file = File::open("data.txt")?;
    println!("Opened!");
}
```

### Q4
```rust
use std::fs;
use std::io::{BufRead, BufReader};
fn main() -> std::io::Result<()> {
    fs::write("temp.txt", "a\nb\nc")?;
    let file = fs::File::open("temp.txt")?;
    let reader = BufReader::new(file);
    for line in reader.lines() {
        println!("{}", line?);
    }
    fs::remove_file("temp.txt")?;
    Ok(())
}
```

---

## Section C — Fix the Bugs

### Q5
```rust
use std::fs::File;
use std::io::Write;

fn main() {
    let file = File::create("output.txt").unwrap();
    writeln!(file, "Hello").unwrap();  // BUG: file is not mutable
}
```

### Q6
```rust
use std::fs;

fn main() -> std::io::Result<()> {
    let contents = fs::read_to_string("missing_file.txt");  // BUG: no error handling
    println!("{contents}");
    Ok(())
}
```

---

## Section D — Write It Yourself

### Q7 — Line counter
Write a program that:
1. Takes a file path (hardcoded or from args)
2. Counts and prints: total lines, non-empty lines, lines containing a keyword
3. Handles the case when the file doesn't exist

### Q8 — Simple logger
Write functions:
- `fn log_message(path: &str, message: &str) -> io::Result<()>` — appends a timestamped message to a log file
- Call it 3 times with different messages
- Read back and print the log file

### Q9 — CSV aggregator (Roadmap Practice Task)
Write a program that:
1. Creates a CSV file with columns: `product,category,price,quantity`
2. Reads and parses the CSV
3. Prints: total revenue (price × quantity), revenue per category, most expensive product

### Q10 — File copy with progress
Write `fn copy_file(src: &str, dst: &str) -> io::Result<u64>` that:
- Opens the source file
- Creates the destination file
- Copies contents using buffered I/O
- Returns the number of bytes copied
- Handles errors gracefully

---

## Section E — Deep Understanding

### Q11 — True or False?
1. `fs::read_to_string` loads the entire file into memory.
2. `BufReader` makes reading faster by reducing system calls.
3. `File::create` fails if the file already exists.
4. `writeln!` can write to any type implementing `Write`.
5. `fs::read_dir` returns files in alphabetical order.
6. `OpenOptions::new().append(true)` preserves existing file contents.
7. `io::stdin().read_line()` includes the newline character.
8. `Path` is an owned type and `PathBuf` is a borrowed type.

### Q12 — Compare approaches
When would you use each?
1. `fs::read_to_string` vs `BufReader::new(file).lines()`
2. `fs::write` vs `File::create` + `write!`
3. `print!` vs writing to `io::stdout().lock()`

---

*Files are the bridge between your program and the world! 🦀*
