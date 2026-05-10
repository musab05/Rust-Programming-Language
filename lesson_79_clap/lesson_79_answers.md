# ✅ Lesson 79 — Answers: CLI Apps with clap (RW1)

---

## Section A

### A1
- **Positional**: order matters, no `--` prefix. E.g., `my_app input.txt output.txt`
- **Flag/option**: named with `--` prefix, order-independent. E.g., `my_app --port 8080 --verbose`

---

## Section B

### A2
```rust
use clap::{Parser, Subcommand, ValueEnum};

#[derive(Parser)]
#[command(name = "todo", about = "Todo manager")]
struct Cli { #[command(subcommand)] cmd: Cmd }

#[derive(Clone, ValueEnum, Debug)]
enum Filter { All, Done, Pending }

#[derive(Subcommand)]
enum Cmd {
    Add { text: String, #[arg(short, long, default_value_t = 3)] priority: u8 },
    List { #[arg(short, long, value_enum, default_value_t = Filter::All)] filter: Filter },
    Done { id: usize },
}

fn main() {
    let cli = Cli::parse();
    match cli.cmd {
        Cmd::Add { text, priority } => println!("Added: {text} (p{priority})"),
        Cmd::List { filter } => println!("Listing: {:?}", filter),
        Cmd::Done { id } => println!("Done: #{id}"),
    }
}
```

### A3
```rust
use clap::Parser;

#[derive(Parser)]
struct Args {
    #[arg(long, value_parser = clap::value_parser!(u16).range(1024..=65535))]
    port: u16,
    #[arg(long, value_parser = validate_email)]
    email: String,
}

fn validate_email(s: &str) -> Result<String, String> {
    if s.contains('@') { Ok(s.into()) } else { Err("Must contain @".into()) }
}

fn main() { let args = Args::parse(); println!("{} {}", args.port, args.email); }
```

---

## Section C

### A4
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | Derive macro generates parsing code from struct definition |
| 2 | **True** | `short` creates `-n`, `long` creates `--name` |
| 3 | **True** | `Option<T>` means the argument can be omitted |
| 4 | **True** | Doc comments (`///`) become help text |
| 5 | **True** | Subcommands are modeled as a `#[derive(Subcommand)]` enum |
| 6 | **True** | `ValueEnum` maps enum variants to string choices |

---

## 🏆 Lesson 79 Complete!

**Next up:** [Lesson 80 — Serialisation with Serde](../lesson_80_serde/lesson_80_serde.md) 🦀
