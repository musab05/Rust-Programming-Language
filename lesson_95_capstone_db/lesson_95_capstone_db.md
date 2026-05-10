# 📘 Lesson 95 — Capstone: CLI SQL Database (RW7)

> **Series:** Rust From Zero · Advanced Level (Capstone Project)  
> **Roadmap ID:** RW7 · Category: 🌍 Real World  
> **Previous:** [Lesson 94 — Database with SQLx](../lesson_94_sqlx/lesson_94_sqlx.md)  
> **Next:** [Lesson 96 — Inline Assembly](../lesson_96_inline_asm/lesson_96_inline_asm.md)  
> **Practice:** [Questions](./lesson_95_questions.md) · [Answers](./lesson_95_answers.md)  
> **Practice Task:** Extend the database with WHERE clause support

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Architecture](#2-architecture)
3. [The Tokenizer (Lexer)](#3-the-tokenizer-lexer)
4. [The Parser](#4-the-parser)
5. [The Storage Engine](#5-the-storage-engine)
6. [The Query Executor](#6-the-query-executor)
7. [The REPL](#7-the-repl)
8. [Putting It All Together](#8-putting-it-all-together)
9. [Extensions & Challenges](#9-extensions--challenges)
10. [Summary](#10-summary)

---

## 1. Project Overview

Build a **minimal SQL database** from scratch:

```
$ cargo run
minidb> CREATE TABLE users (id, name, email);
Table 'users' created (3 columns)

minidb> INSERT INTO users VALUES (1, 'Alice', 'alice@example.com');
Inserted 1 row

minidb> INSERT INTO users VALUES (2, 'Bob', 'bob@example.com');
Inserted 1 row

minidb> SELECT * FROM users;
| id | name  | email             |
|----|-------|-------------------|
| 1  | Alice | alice@example.com |
| 2  | Bob   | bob@example.com   |
(2 rows)

minidb> SELECT name, email FROM users;
| name  | email             |
|-------|-------------------|
| Alice | alice@example.com |
| Bob   | bob@example.com   |
(2 rows)

minidb> DELETE FROM users WHERE id = 1;
Deleted 1 row

minidb> .exit
Bye!
```

### Skills exercised:
- Parsing (tokenizer + parser)
- Data structures (HashMap, Vec)
- Enums and pattern matching
- Error handling (Result, custom errors)
- Traits and polymorphism
- REPL loop
- File I/O (optional persistence)

---

## 2. Architecture

```
User Input (SQL string)
      │
      ▼
┌─────────────┐
│  Tokenizer  │  "SELECT * FROM users" → [Select, Star, From, Ident("users")]
└──────┬──────┘
       ▼
┌─────────────┐
│   Parser    │  Tokens → Statement::Select { table, columns, where_clause }
└──────┬──────┘
       ▼
┌─────────────┐
│  Executor   │  Statement → query the storage engine → results
└──────┬──────┘
       ▼
┌─────────────┐
│  Storage    │  HashMap<String, Table> → rows in memory
└─────────────┘
```

---

## 3. The Tokenizer (Lexer)

```rust
#[derive(Debug, Clone, PartialEq)]
enum Token {
    // Keywords
    Select, From, Where, Insert, Into, Values,
    Create, Table, Delete, Update, Set,
    And, Or, Not,
    // Symbols
    Star, Comma, Semicolon,
    LParen, RParen,
    Eq, NotEq, Lt, Gt, LtEq, GtEq,
    // Literals
    Ident(String),
    StringLit(String),
    NumberLit(f64),
    // End
    Eof,
}

struct Tokenizer {
    input: Vec<char>,
    pos: usize,
}

impl Tokenizer {
    fn new(input: &str) -> Self {
        Tokenizer { input: input.chars().collect(), pos: 0 }
    }

    fn peek(&self) -> Option<char> { self.input.get(self.pos).copied() }
    fn advance(&mut self) -> Option<char> {
        let ch = self.input.get(self.pos).copied();
        self.pos += 1;
        ch
    }

    fn skip_whitespace(&mut self) {
        while self.peek().map_or(false, |c| c.is_whitespace()) { self.advance(); }
    }

    fn read_ident(&mut self) -> String {
        let mut s = String::new();
        while self.peek().map_or(false, |c| c.is_alphanumeric() || c == '_') {
            s.push(self.advance().unwrap());
        }
        s
    }

    fn read_number(&mut self) -> f64 {
        let mut s = String::new();
        while self.peek().map_or(false, |c| c.is_ascii_digit() || c == '.') {
            s.push(self.advance().unwrap());
        }
        s.parse().unwrap_or(0.0)
    }

    fn read_string(&mut self) -> String {
        self.advance(); // skip opening quote
        let mut s = String::new();
        while self.peek().map_or(false, |c| c != '\'') {
            s.push(self.advance().unwrap());
        }
        self.advance(); // skip closing quote
        s
    }

    fn tokenize(&mut self) -> Vec<Token> {
        let mut tokens = vec![];
        loop {
            self.skip_whitespace();
            match self.peek() {
                None => { tokens.push(Token::Eof); break; }
                Some('*') => { self.advance(); tokens.push(Token::Star); }
                Some(',') => { self.advance(); tokens.push(Token::Comma); }
                Some(';') => { self.advance(); tokens.push(Token::Semicolon); }
                Some('(') => { self.advance(); tokens.push(Token::LParen); }
                Some(')') => { self.advance(); tokens.push(Token::RParen); }
                Some('=') => { self.advance(); tokens.push(Token::Eq); }
                Some('<') => {
                    self.advance();
                    if self.peek() == Some('=') { self.advance(); tokens.push(Token::LtEq); }
                    else { tokens.push(Token::Lt); }
                }
                Some('>') => {
                    self.advance();
                    if self.peek() == Some('=') { self.advance(); tokens.push(Token::GtEq); }
                    else { tokens.push(Token::Gt); }
                }
                Some('!') => {
                    self.advance();
                    if self.peek() == Some('=') { self.advance(); tokens.push(Token::NotEq); }
                }
                Some('\'') => { tokens.push(Token::StringLit(self.read_string())); }
                Some(c) if c.is_ascii_digit() => { tokens.push(Token::NumberLit(self.read_number())); }
                Some(c) if c.is_alphabetic() || c == '_' => {
                    let ident = self.read_ident();
                    let token = match ident.to_uppercase().as_str() {
                        "SELECT" => Token::Select,
                        "FROM" => Token::From,
                        "WHERE" => Token::Where,
                        "INSERT" => Token::Insert,
                        "INTO" => Token::Into,
                        "VALUES" => Token::Values,
                        "CREATE" => Token::Create,
                        "TABLE" => Token::Table,
                        "DELETE" => Token::Delete,
                        "UPDATE" => Token::Update,
                        "SET" => Token::Set,
                        "AND" => Token::And,
                        "OR" => Token::Or,
                        "NOT" => Token::Not,
                        _ => Token::Ident(ident),
                    };
                    tokens.push(token);
                }
                Some(c) => { self.advance(); } // skip unknown
            }
        }
        tokens
    }
}
```

---

## 4. The Parser

```rust
#[derive(Debug, Clone)]
enum Value {
    Text(String),
    Number(f64),
    Null,
}

impl std::fmt::Display for Value {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        match self {
            Value::Text(s) => write!(f, "{s}"),
            Value::Number(n) => {
                if *n == (*n as i64) as f64 { write!(f, "{}", *n as i64) }
                else { write!(f, "{n}") }
            }
            Value::Null => write!(f, "NULL"),
        }
    }
}

#[derive(Debug)]
enum WhereClause {
    Eq(String, Value),
    Gt(String, Value),
    Lt(String, Value),
}

#[derive(Debug)]
enum Statement {
    CreateTable { name: String, columns: Vec<String> },
    Insert { table: String, values: Vec<Value> },
    Select { table: String, columns: Vec<String>, where_clause: Option<WhereClause> },
    Delete { table: String, where_clause: Option<WhereClause> },
}

struct Parser {
    tokens: Vec<Token>,
    pos: usize,
}

impl Parser {
    fn new(tokens: Vec<Token>) -> Self { Parser { tokens, pos: 0 } }
    fn peek(&self) -> &Token { self.tokens.get(self.pos).unwrap_or(&Token::Eof) }
    fn advance(&mut self) -> Token { let t = self.tokens.get(self.pos).cloned().unwrap_or(Token::Eof); self.pos += 1; t }
    fn expect(&mut self, expected: &Token) -> Result<(), String> {
        let got = self.advance();
        if &got == expected { Ok(()) } else { Err(format!("Expected {expected:?}, got {got:?}")) }
    }

    fn parse(&mut self) -> Result<Statement, String> {
        match self.peek().clone() {
            Token::Create => self.parse_create(),
            Token::Insert => self.parse_insert(),
            Token::Select => self.parse_select(),
            Token::Delete => self.parse_delete(),
            other => Err(format!("Unexpected token: {other:?}")),
        }
    }

    fn parse_create(&mut self) -> Result<Statement, String> {
        self.advance(); // CREATE
        self.expect(&Token::Table)?;
        let name = match self.advance() { Token::Ident(n) => n, t => return Err(format!("Expected table name, got {t:?}")) };
        self.expect(&Token::LParen)?;
        let mut columns = vec![];
        loop {
            match self.advance() { Token::Ident(c) => columns.push(c), t => return Err(format!("Expected column, got {t:?}")) }
            if self.peek() == &Token::Comma { self.advance(); } else { break; }
        }
        self.expect(&Token::RParen)?;
        Ok(Statement::CreateTable { name, columns })
    }

    fn parse_insert(&mut self) -> Result<Statement, String> {
        self.advance(); // INSERT
        self.expect(&Token::Into)?;
        let table = match self.advance() { Token::Ident(n) => n, t => return Err(format!("Expected table, got {t:?}")) };
        self.expect(&Token::Values)?;
        self.expect(&Token::LParen)?;
        let mut values = vec![];
        loop {
            let val = match self.advance() {
                Token::StringLit(s) => Value::Text(s),
                Token::NumberLit(n) => Value::Number(n),
                Token::Ident(s) if s.to_uppercase() == "NULL" => Value::Null,
                Token::Ident(s) => Value::Text(s),
                t => return Err(format!("Expected value, got {t:?}")),
            };
            values.push(val);
            if self.peek() == &Token::Comma { self.advance(); } else { break; }
        }
        self.expect(&Token::RParen)?;
        Ok(Statement::Insert { table, values })
    }

    fn parse_select(&mut self) -> Result<Statement, String> {
        self.advance(); // SELECT
        let mut columns = vec![];
        if self.peek() == &Token::Star { self.advance(); columns.push("*".into()); }
        else {
            loop {
                match self.advance() { Token::Ident(c) => columns.push(c), t => return Err(format!("Expected column, got {t:?}")) }
                if self.peek() == &Token::Comma { self.advance(); } else { break; }
            }
        }
        self.expect(&Token::From)?;
        let table = match self.advance() { Token::Ident(n) => n, t => return Err(format!("Expected table, got {t:?}")) };
        let where_clause = if self.peek() == &Token::Where { self.advance(); Some(self.parse_where()?) } else { None };
        Ok(Statement::Select { table, columns, where_clause })
    }

    fn parse_delete(&mut self) -> Result<Statement, String> {
        self.advance(); // DELETE
        self.expect(&Token::From)?;
        let table = match self.advance() { Token::Ident(n) => n, t => return Err(format!("Expected table, got {t:?}")) };
        let where_clause = if self.peek() == &Token::Where { self.advance(); Some(self.parse_where()?) } else { None };
        Ok(Statement::Delete { table, where_clause })
    }

    fn parse_where(&mut self) -> Result<WhereClause, String> {
        let col = match self.advance() { Token::Ident(c) => c, t => return Err(format!("Expected column, got {t:?}")) };
        let op = self.advance();
        let val = match self.advance() {
            Token::StringLit(s) => Value::Text(s),
            Token::NumberLit(n) => Value::Number(n),
            Token::Ident(s) => Value::Text(s),
            t => return Err(format!("Expected value, got {t:?}")),
        };
        match op { Token::Eq => Ok(WhereClause::Eq(col, val)), Token::Gt => Ok(WhereClause::Gt(col, val)), Token::Lt => Ok(WhereClause::Lt(col, val)), _ => Err("Unsupported operator".into()) }
    }
}
```

---

## 5. The Storage Engine

```rust
use std::collections::HashMap;

#[derive(Debug)]
struct Table {
    columns: Vec<String>,
    rows: Vec<Vec<Value>>,
}

struct Database {
    tables: HashMap<String, Table>,
}

impl Database {
    fn new() -> Self { Database { tables: HashMap::new() } }

    fn create_table(&mut self, name: &str, columns: Vec<String>) -> Result<String, String> {
        if self.tables.contains_key(name) { return Err(format!("Table '{name}' already exists")); }
        let col_count = columns.len();
        self.tables.insert(name.to_string(), Table { columns, rows: vec![] });
        Ok(format!("Table '{name}' created ({col_count} columns)"))
    }

    fn insert(&mut self, table: &str, values: Vec<Value>) -> Result<String, String> {
        let tbl = self.tables.get_mut(table).ok_or(format!("Table '{table}' not found"))?;
        if values.len() != tbl.columns.len() {
            return Err(format!("Expected {} values, got {}", tbl.columns.len(), values.len()));
        }
        tbl.rows.push(values);
        Ok("Inserted 1 row".into())
    }

    fn select(&self, table: &str, columns: &[String], wh: &Option<WhereClause>) -> Result<(Vec<String>, Vec<Vec<Value>>), String> {
        let tbl = self.tables.get(table).ok_or(format!("Table '{table}' not found"))?;
        let col_indices: Vec<usize> = if columns.len() == 1 && columns[0] == "*" {
            (0..tbl.columns.len()).collect()
        } else {
            columns.iter().map(|c| tbl.columns.iter().position(|tc| tc == c).ok_or(format!("Column '{c}' not found"))).collect::<Result<Vec<_>, _>>()?
        };
        let col_names: Vec<String> = col_indices.iter().map(|&i| tbl.columns[i].clone()).collect();
        let mut result_rows = vec![];
        for row in &tbl.rows {
            if self.matches_where(row, &tbl.columns, wh)? {
                let selected: Vec<Value> = col_indices.iter().map(|&i| row[i].clone()).collect();
                result_rows.push(selected);
            }
        }
        Ok((col_names, result_rows))
    }

    fn delete(&mut self, table: &str, wh: &Option<WhereClause>) -> Result<String, String> {
        let tbl = self.tables.get_mut(table).ok_or(format!("Table '{table}' not found"))?;
        let columns = tbl.columns.clone();
        let before = tbl.rows.len();
        tbl.rows.retain(|row| !self.matches_where_inner(row, &columns, wh));
        let deleted = before - tbl.rows.len();
        Ok(format!("Deleted {deleted} row(s)"))
    }

    fn matches_where(&self, row: &[Value], columns: &[String], wh: &Option<WhereClause>) -> Result<bool, String> {
        Ok(self.matches_where_inner(row, columns, wh))
    }

    fn matches_where_inner(&self, row: &[Value], columns: &[String], wh: &Option<WhereClause>) -> bool {
        match wh {
            None => true,
            Some(WhereClause::Eq(col, val)) => {
                if let Some(idx) = columns.iter().position(|c| c == col) {
                    self.values_equal(&row[idx], val)
                } else { false }
            }
            Some(WhereClause::Gt(col, val)) => {
                if let Some(idx) = columns.iter().position(|c| c == col) {
                    self.value_gt(&row[idx], val)
                } else { false }
            }
            Some(WhereClause::Lt(col, val)) => {
                if let Some(idx) = columns.iter().position(|c| c == col) {
                    self.value_lt(&row[idx], val)
                } else { false }
            }
        }
    }

    fn values_equal(&self, a: &Value, b: &Value) -> bool {
        match (a, b) {
            (Value::Text(a), Value::Text(b)) => a == b,
            (Value::Number(a), Value::Number(b)) => (a - b).abs() < f64::EPSILON,
            (Value::Text(a), Value::Number(b)) => a.parse::<f64>().map_or(false, |n| (n - b).abs() < f64::EPSILON),
            (Value::Number(a), Value::Text(b)) => b.parse::<f64>().map_or(false, |n| (n - a).abs() < f64::EPSILON),
            _ => false,
        }
    }

    fn value_gt(&self, a: &Value, b: &Value) -> bool {
        match (a, b) { (Value::Number(a), Value::Number(b)) => a > b, _ => false }
    }

    fn value_lt(&self, a: &Value, b: &Value) -> bool {
        match (a, b) { (Value::Number(a), Value::Number(b)) => a < b, _ => false }
    }
}
```

---

## 6. The Query Executor

```rust
fn execute(db: &mut Database, stmt: Statement) -> Result<String, String> {
    match stmt {
        Statement::CreateTable { name, columns } => db.create_table(&name, columns),
        Statement::Insert { table, values } => db.insert(&table, values),
        Statement::Select { table, columns, where_clause } => {
            let (cols, rows) = db.select(&table, &columns, &where_clause)?;
            Ok(format_table(&cols, &rows))
        }
        Statement::Delete { table, where_clause } => db.delete(&table, &where_clause),
    }
}

fn format_table(columns: &[String], rows: &[Vec<Value>]) -> String {
    let mut widths: Vec<usize> = columns.iter().map(|c| c.len()).collect();
    for row in rows {
        for (i, val) in row.iter().enumerate() {
            widths[i] = widths[i].max(val.to_string().len());
        }
    }
    let mut out = String::new();
    // Header
    out.push('|');
    for (i, col) in columns.iter().enumerate() { out.push_str(&format!(" {:w$} |", col, w = widths[i])); }
    out.push('\n');
    // Separator
    out.push('|');
    for w in &widths { out.push_str(&format!("-{}-|", "-".repeat(*w))); }
    out.push('\n');
    // Rows
    for row in rows {
        out.push('|');
        for (i, val) in row.iter().enumerate() { out.push_str(&format!(" {:w$} |", val.to_string(), w = widths[i])); }
        out.push('\n');
    }
    out.push_str(&format!("({} rows)", rows.len()));
    out
}
```

---

## 7. The REPL

```rust
use std::io::{self, Write};

fn main() {
    let mut db = Database::new();
    println!("minidb v0.1.0 — Type SQL commands or .exit to quit\n");

    loop {
        print!("minidb> ");
        io::stdout().flush().unwrap();

        let mut input = String::new();
        if io::stdin().read_line(&mut input).is_err() { break; }
        let input = input.trim();

        if input.is_empty() { continue; }
        if input == ".exit" || input == ".quit" { println!("Bye!"); break; }
        if input == ".tables" {
            for name in db.tables.keys() { println!("  {name}"); }
            continue;
        }

        let mut tokenizer = Tokenizer::new(input);
        let tokens = tokenizer.tokenize();
        let mut parser = Parser::new(tokens);

        match parser.parse() {
            Ok(stmt) => match execute(&mut db, stmt) {
                Ok(msg) => println!("{msg}\n"),
                Err(e) => println!("Error: {e}\n"),
            },
            Err(e) => println!("Parse error: {e}\n"),
        }
    }
}
```

---

## 8. Putting It All Together

Create these files in your project:

```
minidb/
├── Cargo.toml
└── src/
    ├── main.rs       ← REPL + main
    ├── token.rs      ← Token enum + Tokenizer
    ├── parser.rs     ← Parser + Statement/Value enums
    ├── storage.rs    ← Database + Table
    └── executor.rs   ← execute() + format_table()
```

All the code sections above fit together into a working ~300-line SQL database.

---

## 9. Extensions & Challenges

Once the basics work, try adding:

| Extension | Difficulty | Skills |
|---|---|---|
| `UPDATE ... SET ... WHERE` | ⭐⭐ | Parser, executor |
| `ORDER BY column` | ⭐⭐ | Sorting Vec |
| `LIMIT n` | ⭐ | Iterator take |
| Persistence (save to file) | ⭐⭐ | Serde, File I/O |
| `COUNT(*)`, `SUM()`, `AVG()` | ⭐⭐⭐ | Aggregation |
| `JOIN` support | ⭐⭐⭐⭐ | Cartesian product, filtering |
| B-Tree index | ⭐⭐⭐⭐⭐ | Data structures |

---

## 10. Summary

This capstone brings together:

| Concept | Where Used |
|---|---|
| Enums + pattern matching | Token, Value, Statement, WhereClause |
| Structs | Table, Database, Tokenizer, Parser |
| HashMap | Tables storage |
| Vec | Rows, columns, tokens |
| Result + error handling | Every operation |
| Traits (Display) | Value formatting |
| String processing | Tokenizer |
| Iterators | Row filtering, column mapping |
| I/O | REPL stdin/stdout |
| Ownership + borrowing | &mut Database, &[Value] |

---

## What's Next?

**Lesson 96 — Inline Assembly** — Writing assembly inside Rust with `asm!()`.

## Further Reading
- [Build Your Own Database](https://cstack.github.io/db_tutorial/)
- [SQLite Architecture](https://www.sqlite.org/arch.html)

---

*Capstone complete: you built a SQL database in Rust! 🦀🏆*
