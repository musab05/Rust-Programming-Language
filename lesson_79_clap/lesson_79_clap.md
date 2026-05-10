# 📘 Lesson 79 — CLI Apps with clap (RW1)

> **Series:** Rust From Zero · Intermediate Level (Gap Fill)  
> **Roadmap ID:** RW1 · Category: 🌍 Real World  
> **Previous:** [Lesson 78 — RAII & Guard Types](../lesson_78_raii_guards/lesson_78_raii_guards.md)  
> **Next:** [Lesson 80 — Serialisation with Serde](../lesson_80_serde/lesson_80_serde.md)  
> **Practice:** [Questions](./lesson_79_questions.md) · [Answers](./lesson_79_answers.md)  
> **Practice Task:** Build a full-featured todo CLI app

---

## Table of Contents

1. [Why clap?](#1-why-clap)
2. [Setup](#2-setup)
3. [Basic Derive API](#3-basic-derive-api)
4. [Positional and Optional Arguments](#4-positional-and-optional-arguments)
5. [Flags and Defaults](#5-flags-and-defaults)
6. [Subcommands](#6-subcommands)
7. [Validation](#7-validation)
8. [Help Generation](#8-help-generation)
9. [Real-World Example: Todo CLI](#9-real-world-example-todo-cli)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. Why clap?

`clap` is the most popular argument parsing crate in Rust:

```bash
# Without clap — manual parsing, no help, no validation:
let args: Vec<String> = std::env::args().collect();
let filename = &args[1];  // panics if missing

# With clap — automatic help, validation, subcommands:
$ my_app --help
$ my_app add "Buy groceries" --priority high
$ my_app list --filter done
```

---

## 2. Setup

```toml
# Cargo.toml
[dependencies]
clap = { version = "4", features = ["derive"] }
```

---

## 3. Basic Derive API

```rust
use clap::Parser;

/// A simple greeting application
#[derive(Parser, Debug)]
#[command(name = "greeter", version, about = "Greets people")]
struct Args {
    /// Name of the person to greet
    name: String,

    /// Number of times to greet
    #[arg(short, long, default_value_t = 1)]
    count: u32,
}

fn main() {
    let args = Args::parse();

    for _ in 0..args.count {
        println!("Hello, {}!", args.name);
    }
}
```

```bash
$ cargo run -- Alice
Hello, Alice!

$ cargo run -- Alice --count 3
Hello, Alice!
Hello, Alice!
Hello, Alice!

$ cargo run -- --help
Greets people

Usage: greeter [OPTIONS] <NAME>

Arguments:
  <NAME>  Name of the person to greet

Options:
  -c, --count <COUNT>  Number of times to greet [default: 1]
  -h, --help           Print help
  -V, --version        Print version
```

---

## 4. Positional and Optional Arguments

```rust
use clap::Parser;

#[derive(Parser, Debug)]
struct Args {
    /// Input file (required, positional)
    input: String,

    /// Output file (optional, positional)
    output: Option<String>,

    /// Verbose mode
    #[arg(short, long)]
    verbose: bool,

    /// Log level
    #[arg(short, long, default_value = "info")]
    level: String,
}

fn main() {
    let args = Args::parse();
    println!("Input: {}", args.input);
    println!("Output: {:?}", args.output.unwrap_or("stdout".into()));
    println!("Verbose: {}", args.verbose);
    println!("Level: {}", args.level);
}
```

```bash
$ my_app data.csv
$ my_app data.csv output.json --verbose --level debug
$ my_app data.csv -v -l warn
```

---

## 5. Flags and Defaults

```rust
use clap::{Parser, ValueEnum};

#[derive(Debug, Clone, ValueEnum)]
enum Format {
    Json,
    Csv,
    Table,
}

#[derive(Parser, Debug)]
struct Args {
    /// Output format
    #[arg(short, long, value_enum, default_value_t = Format::Table)]
    format: Format,

    /// Maximum results
    #[arg(short, long, default_value_t = 10)]
    max: usize,

    /// Dry run mode
    #[arg(long)]
    dry_run: bool,

    /// Verbosity (-v, -vv, -vvv)
    #[arg(short, long, action = clap::ArgAction::Count)]
    verbose: u8,
}

fn main() {
    let args = Args::parse();
    println!("Format: {:?}", args.format);
    println!("Max: {}", args.max);
    println!("Dry run: {}", args.dry_run);
    println!("Verbosity: {}", args.verbose);
}
```

---

## 6. Subcommands

```rust
use clap::{Parser, Subcommand};

#[derive(Parser, Debug)]
#[command(name = "task", about = "Task manager CLI")]
struct Cli {
    #[command(subcommand)]
    command: Commands,
}

#[derive(Subcommand, Debug)]
enum Commands {
    /// Add a new task
    Add {
        /// Task description
        description: String,

        /// Priority (1-5)
        #[arg(short, long, default_value_t = 3)]
        priority: u8,
    },

    /// List all tasks
    List {
        /// Filter by status
        #[arg(short, long)]
        filter: Option<String>,
    },

    /// Complete a task
    Done {
        /// Task ID
        id: u32,
    },

    /// Remove a task
    Remove {
        /// Task ID
        id: u32,

        /// Skip confirmation
        #[arg(short, long)]
        force: bool,
    },
}

fn main() {
    let cli = Cli::parse();

    match cli.command {
        Commands::Add { description, priority } => {
            println!("Adding task: '{description}' (priority: {priority})");
        }
        Commands::List { filter } => {
            println!("Listing tasks (filter: {:?})", filter);
        }
        Commands::Done { id } => {
            println!("Marking task {id} as done");
        }
        Commands::Remove { id, force } => {
            if force { println!("Force removing task {id}"); }
            else { println!("Remove task {id}? (use --force to skip)"); }
        }
    }
}
```

```bash
$ task add "Buy milk" --priority 1
$ task list --filter pending
$ task done 3
$ task remove 5 --force
```

---

## 7. Validation

```rust
use clap::Parser;

#[derive(Parser, Debug)]
struct Args {
    /// Port number (1-65535)
    #[arg(short, long, value_parser = clap::value_parser!(u16).range(1..))]
    port: u16,

    /// Email address
    #[arg(short, long, value_parser = validate_email)]
    email: String,
}

fn validate_email(s: &str) -> Result<String, String> {
    if s.contains('@') && s.contains('.') {
        Ok(s.to_string())
    } else {
        Err(format!("'{s}' is not a valid email address"))
    }
}

fn main() {
    let args = Args::parse();
    println!("Port: {}, Email: {}", args.port, args.email);
}
```

```bash
$ my_app --port 0
error: 0 is not in 1..=65535

$ my_app --port 8080 --email "invalid"
error: 'invalid' is not a valid email address
```

---

## 8. Help Generation

clap auto-generates `--help` from doc comments:

```rust
/// My awesome application
///
/// This app does amazing things with your data.
/// It supports multiple formats and is very fast.
#[derive(Parser)]
#[command(
    name = "awesome",
    version = "1.0.0",
    author = "Your Name",
    about = "Does awesome things",
    long_about = "A detailed description of what this tool does..."
)]
struct Args {
    /// The input file to process
    #[arg(help = "Path to the input file")]
    input: String,
}
```

---

## 9. Real-World Example: Todo CLI

```rust
use clap::{Parser, Subcommand};

#[derive(Parser)]
#[command(name = "todo", version, about = "A simple todo manager")]
struct Cli {
    #[command(subcommand)]
    cmd: Cmd,
}

#[derive(Subcommand)]
enum Cmd {
    /// Add a new todo item
    Add {
        /// Todo description
        text: String,
    },
    /// List all todos
    List,
    /// Mark a todo as complete
    Done { id: usize },
}

fn main() {
    let cli = Cli::parse();
    // In a real app, you'd persist to a file (see Lesson 80 — Serde)
    match cli.cmd {
        Cmd::Add { text } => println!("✅ Added: {text}"),
        Cmd::List => println!("📋 (no items yet — implement persistence!)"),
        Cmd::Done { id } => println!("✔️  Completed item #{id}"),
    }
}
```

---

## 10. Summary Cheat Sheet

```
SETUP
────────────────────────────────────────────────────────────
clap = { version = "4", features = ["derive"] }

BASIC
────────────────────────────────────────────────────────────
#[derive(Parser)]
struct Args { name: String }
let args = Args::parse();

ARGUMENTS
────────────────────────────────────────────────────────────
name: String               positional, required
name: Option<String>       positional, optional
#[arg(short, long)]        --name or -n flag
#[arg(default_value_t)]    default value

SUBCOMMANDS
────────────────────────────────────────────────────────────
#[derive(Subcommand)]
enum Commands { Add { text: String }, List, Remove { id: u32 } }

ENUMS
────────────────────────────────────────────────────────────
#[derive(ValueEnum)]       --format json|csv|table

VALIDATION
────────────────────────────────────────────────────────────
#[arg(value_parser = my_fn)]  custom validation
value_parser!(u16).range(1..) range check
```

---

## What's Next?

**Lesson 80 — Serialisation with Serde** — `#[derive(Serialize, Deserialize)]`, JSON, TOML, and custom formats.

## Further Reading
- [clap docs](https://docs.rs/clap/)
- [clap derive tutorial](https://docs.rs/clap/latest/clap/_derive/_tutorial/index.html)

---

*clap: professional CLIs in minutes! 🦀*
