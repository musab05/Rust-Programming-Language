# ✅ Lesson 29 — Answers: File I/O & std::io (RW2)

---

## Section A

### A1
```
3
16
```
3 lines (`"Hello"`, `"World"`, `"Rust"`). Length is 16 bytes: `Hello\nWorld\nRust` = 5+1+5+1+4 = 16.

### A2
```
"main.rs"
"rs"
"/home/user/project/src"
false
```
Path components extracted. `exists()` is `false` because this path doesn't actually exist on the system.

---

## Section B

### A3 — ❌ No
`?` requires the function to return `Result`. `main()` returns `()`. Fix: `fn main() -> std::io::Result<()>` and add `Ok(())`.

### A4 — ✅ Yes
Compiles and runs. Creates temp file, reads line by line with BufReader, prints each line, then cleans up.

---

## Section C

### A5
```rust
use std::fs::File;
use std::io::Write;

fn main() {
    let mut file = File::create("output.txt").unwrap();  // fix: add mut
    writeln!(file, "Hello").unwrap();
}
```

### A6
```rust
use std::fs;

fn main() -> std::io::Result<()> {
    let contents = fs::read_to_string("missing_file.txt")?;  // fix: propagate error with ?
    println!("{contents}");
    Ok(())
}
```
Or handle with `match` for a user-friendly message.

---

## Section D

### A7 — Line counter
```rust
use std::fs::File;
use std::io::{BufRead, BufReader};

fn count_lines(path: &str, keyword: &str) {
    let file = match File::open(path) {
        Ok(f) => f,
        Err(e) => {
            println!("Error opening '{path}': {e}");
            return;
        }
    };

    let reader = BufReader::new(file);
    let mut total = 0;
    let mut non_empty = 0;
    let mut with_keyword = 0;

    for line in reader.lines() {
        let line = line.expect("Failed to read line");
        total += 1;
        if !line.trim().is_empty() {
            non_empty += 1;
        }
        if line.contains(keyword) {
            with_keyword += 1;
        }
    }

    println!("File: {path}");
    println!("  Total lines:     {total}");
    println!("  Non-empty lines: {non_empty}");
    println!("  Lines with '{keyword}': {with_keyword}");
}

fn main() {
    count_lines("Cargo.toml", "version");
    count_lines("nonexistent.txt", "test");
}
```

### A8 — Simple logger
```rust
use std::fs::{self, OpenOptions};
use std::io::{self, Write};

fn log_message(path: &str, message: &str) -> io::Result<()> {
    let mut file = OpenOptions::new()
        .create(true)
        .append(true)
        .open(path)?;

    let timestamp = "2024-01-15 10:30:00";  // simplified
    writeln!(file, "[{timestamp}] {message}")?;
    Ok(())
}

fn main() -> io::Result<()> {
    let log_path = "app.log";

    log_message(log_path, "Application started")?;
    log_message(log_path, "Processing data...")?;
    log_message(log_path, "Application stopped")?;

    // Read back
    let contents = fs::read_to_string(log_path)?;
    println!("=== Log Contents ===");
    println!("{contents}");

    fs::remove_file(log_path)?;
    Ok(())
}
```

### A9 — CSV aggregator
```rust
use std::collections::HashMap;
use std::fs::{self, File};
use std::io::{BufRead, BufReader, Write};

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let path = "products.csv";

    // Create sample data
    let mut f = File::create(path)?;
    writeln!(f, "product,category,price,quantity")?;
    writeln!(f, "Laptop,Electronics,999.99,50")?;
    writeln!(f, "Phone,Electronics,699.99,200")?;
    writeln!(f, "Desk,Furniture,249.99,100")?;
    writeln!(f, "Chair,Furniture,189.99,150")?;
    writeln!(f, "Headphones,Electronics,149.99,300")?;
    writeln!(f, "Bookshelf,Furniture,129.99,80")?;
    drop(f);

    // Parse CSV
    let file = File::open(path)?;
    let reader = BufReader::new(file);

    let mut total_revenue = 0.0f64;
    let mut category_revenue: HashMap<String, f64> = HashMap::new();
    let mut most_expensive: Option<(String, f64)> = None;

    for (i, line) in reader.lines().enumerate() {
        let line = line?;
        if i == 0 { continue; }

        let fields: Vec<&str> = line.split(',').collect();
        if fields.len() != 4 { continue; }

        let product = fields[0].trim();
        let category = fields[1].trim();
        let price: f64 = fields[2].trim().parse()?;
        let quantity: f64 = fields[3].trim().parse()?;

        let revenue = price * quantity;
        total_revenue += revenue;
        *category_revenue.entry(category.to_string()).or_insert(0.0) += revenue;

        match &most_expensive {
            None => most_expensive = Some((product.to_string(), price)),
            Some((_, p)) if price > *p => {
                most_expensive = Some((product.to_string(), price));
            }
            _ => {}
        }
    }

    println!("Total revenue: ${total_revenue:.2}");
    println!("\nRevenue by category:");
    let mut cats: Vec<_> = category_revenue.iter().collect();
    cats.sort_by(|a, b| b.1.partial_cmp(a.1).unwrap());
    for (cat, rev) in cats {
        println!("  {cat}: ${rev:.2}");
    }
    if let Some((name, price)) = most_expensive {
        println!("\nMost expensive: {name} (${price:.2})");
    }

    fs::remove_file(path)?;
    Ok(())
}
```

### A10 — File copy
```rust
use std::fs::File;
use std::io::{self, BufReader, BufWriter, Read, Write};

fn copy_file(src: &str, dst: &str) -> io::Result<u64> {
    let src_file = File::open(src)?;
    let dst_file = File::create(dst)?;

    let mut reader = BufReader::new(src_file);
    let mut writer = BufWriter::new(dst_file);

    let mut total = 0u64;
    let mut buffer = [0u8; 8192];

    loop {
        let bytes_read = reader.read(&mut buffer)?;
        if bytes_read == 0 { break; }
        writer.write_all(&buffer[..bytes_read])?;
        total += bytes_read as u64;
    }

    writer.flush()?;
    Ok(total)
}

fn main() {
    match copy_file("Cargo.toml", "Cargo_copy.toml") {
        Ok(bytes) => {
            println!("Copied {bytes} bytes");
            std::fs::remove_file("Cargo_copy.toml").ok();
        }
        Err(e) => println!("Copy failed: {e}"),
    }
}
```

---

## Section E

### A11 — True or False?
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | `read_to_string` reads the entire file into a `String` in memory |
| 2 | **True** | BufReader batches reads into a buffer, reducing syscall overhead |
| 3 | **False** | `File::create` overwrites if the file exists; use `create_new(true)` on OpenOptions to fail |
| 4 | **True** | `writeln!` works with `File`, `BufWriter`, `Vec<u8>`, or any `Write` implementor |
| 5 | **False** | `read_dir` order is filesystem-dependent, not guaranteed alphabetical |
| 6 | **True** | `append(true)` adds to the end without clearing existing content |
| 7 | **True** | `read_line` includes `\n`; use `.trim()` to remove it |
| 8 | **False** | It's the opposite: `Path` is borrowed (`&str`-like), `PathBuf` is owned (`String`-like) |

### A12 — Compare approaches
1. **`read_to_string` vs `BufReader.lines()`**: Use `read_to_string` for small files where you need the whole content. Use `BufReader` for large files or when processing line-by-line to avoid loading everything into memory.

2. **`fs::write` vs `File::create` + `write!`**: Use `fs::write` for simple one-shot writes. Use `File::create` + `write!` when you need multiple write operations or formatted output.

3. **`print!` vs `stdout().lock()`**: `print!` locks stdout for each call (fine for occasional output). Locking once with `stdout().lock()` is faster for loops with many writes.

---

## 🏆 Lesson 29 Complete!

✅ `fs::read_to_string` / `fs::write` for simple I/O  
✅ `File::open` / `File::create` for handle-based I/O  
✅ `BufReader` / `BufWriter` for performance  
✅ `OpenOptions` for append mode  
✅ Line-by-line reading with `.lines()`  
✅ stdin/stdout/stderr  
✅ `Path` / `PathBuf` for path manipulation  
✅ Directory operations  
✅ CSV parser — roadmap practice ✓  

**Up next: Lesson 30 — Lifetimes Basics 🦀**
