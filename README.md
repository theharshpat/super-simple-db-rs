# super-simple-db-rs

A minimal database implementation in Rust, following Sciore's "Database Design and Implementation" (2nd Edition). Java code from the book is translated to idiomatic Rust.

## Goals

1. **Learn Rust** - through building a real system
2. **Learn DB internals** - by implementing from scratch
3. **Keep it simple** - minimal viable features, no over-engineering

## LLM Interaction Protocol

The LLM acts as a **tutor/co-learning buddy**, NOT a code generator.

### Workflow for Each Module

```
1. QUIZ      -> LLM quizzes on concepts before implementation
2. ASSESS    -> LLM identifies knowledge gaps from responses
3. EXPLAIN   -> LLM fills gaps with targeted explanations
4. DESIGN    -> LLM probes: "How would YOU design this?"
5. CRITIQUE  -> LLM presents alternatives, tradeoffs
6. IMPLEMENT -> ONLY when user says "implement" - small chunks, user takes back control
```

### Rules

- Never implement without being asked
- Quiz thoroughly before any implementation
- Explain Rust concepts as they arise (ownership, lifetimes, traits, etc.)
- Compare Rust approach vs Java (from book) when relevant
- Keep implementations minimal - no premature abstractions

## Learner Background

- Completed CMU 15-445 (Pavlo, 2025) - all 25 lectures
- Understands DB concepts: buffer pool, B+trees, query execution, concurrency control, recovery
- Rust level: can read most code, struggles with advanced concepts (lifetimes, async, macros)

## Error Handling

Use `thiserror` crate for custom error types:

```rust
#[derive(Debug, thiserror::Error)]
pub enum DbError {
    #[error("I/O error: {0}")]
    Io(#[from] std::io::Error),
    #[error("buffer pool exhausted")]
    BufferAbort,
    #[error("lock timeout - possible deadlock")]
    LockAbort,
    #[error("parse error: {0}")]
    Parse(String),
    // ... etc
}

pub type Result<T> = std::result::Result<T, DbError>;
```

## Testing Strategy

**Unit tests** (`#[cfg(test)]` inline in each module):
- Test individual structs/functions
- Can access private internals
- Example: test `Page::get_int`/`set_int` roundtrip

**Integration tests** (`/tests/` directory):
- Test multi-layer flows as black box
- Only access public API
- Example: full SQL flow (create table -> insert -> query -> verify)

```
tests/
├── file_test.rs       # FileMgr read/write blocks
├── buffer_test.rs     # BufferMgr pin/unpin/flush
├── tx_test.rs         # Transaction commit/rollback/recovery
├── record_test.rs     # TableScan iteration
├── sql_test.rs        # Full parse -> plan -> execute flow
└── recovery_test.rs   # Crash recovery scenarios
```

Clean up test databases in each test (use `tempdir` or delete after).

## Implementation Roadmap

Build order follows dependencies. Each module builds on previous ones.

### Phase 1: Storage Engine

| # | Module | Key Structs | Rust Concepts | Status |
|---|--------|-------------|---------------|--------|
| 1.1 | **File Manager** | `BlockId`, `Page`, `FileMgr` | `std::fs`, `Result`, error handling, byte slices | [ ] |
| 1.2 | **Log Manager** | `LogMgr`, `LogIterator` | Iterators, interior mutability | [ ] |
| 1.3 | **Buffer Manager** | `Buffer`, `BufferMgr` | `Rc<RefCell<>>` or `Arc<Mutex<>>`, ownership | [ ] |

### Phase 2: Transactions

| # | Module | Key Structs | Rust Concepts | Status |
|---|--------|-------------|---------------|--------|
| 2.1 | **Concurrency Mgr** | `LockTable`, `ConcurrencyMgr` | `HashMap`, `Condvar`, wait/notify | [ ] |
| 2.2 | **Recovery Mgr** | `LogRecord`, `RecoveryMgr` | Enums with data, pattern matching | [ ] |
| 2.3 | **Transaction** | `Transaction` | RAII, `Drop` trait, coordinating managers | [ ] |

### Phase 3: Record Management

| # | Module | Key Structs | Rust Concepts | Status |
|---|--------|-------------|---------------|--------|
| 3.1 | **Record Page** | `Schema`, `Layout`, `RecordPage` | Structs, enums, builder pattern | [ ] |
| 3.2 | **Table Scan** | `TableScan` | Trait implementation, mutable iteration | [ ] |

### Phase 4: Metadata

| # | Module | Key Structs | Rust Concepts | Status |
|---|--------|-------------|---------------|--------|
| 4.1 | **Table Manager** | `TableMgr` | Self-referential bootstrap problem | [ ] |
| 4.2 | **View Manager** | `ViewMgr` | String storage, simple CRUD | [ ] |
| 4.3 | **Stat Manager** | `StatMgr`, `StatInfo` | `HashMap`, lazy refresh | [ ] |
| 4.4 | **Index Manager** | `IndexMgr`, `IndexInfo` | Metadata for indexes | [ ] |
| 4.5 | **Metadata Manager** | `MetadataMgr` | Facade pattern | [ ] |

### Phase 5: Query Processing

| # | Module | Key Structs | Rust Concepts | Status |
|---|--------|-------------|---------------|--------|
| 5.1 | **Scan Interface** | `Scan`, `UpdateScan` traits | Trait objects, `Box<dyn Trait>` | [ ] |
| 5.2 | **Predicates** | `Constant`, `Expression`, `Term`, `Predicate` | Enums, recursive evaluation | [ ] |
| 5.3 | **Query Scans** | `SelectScan`, `ProjectScan`, `ProductScan` | Composition, trait delegation | [ ] |

### Phase 6: Parsing & Planning

| # | Module | Key Structs | Rust Concepts | Status |
|---|--------|-------------|---------------|--------|
| 6.1 | **Lexer** | `Lexer`, `Token` | `Peekable<Chars>`, string processing | [ ] |
| 6.2 | **Parser** | `Parser`, `*Data` structs | Recursive descent, `Result` chaining | [ ] |
| 6.3 | **Planner** | `Plan` trait, `QueryPlanner`, `UpdatePlanner` | Trait objects, strategy pattern | [ ] |

### Phase 7: Indexing

| # | Module | Key Structs | Rust Concepts | Status |
|---|--------|-------------|---------------|--------|
| 7.1 | **Index Interface** | `Index` trait | Trait definition | [ ] |
| 7.2 | **Hash Index** | `HashIndex` | Hashing, bucket files | [ ] |
| 7.3 | **B-Tree Index** | `BTreeLeaf`, `BTreeDir`, `BTreeIndex` | Recursive structures, tree traversal | [ ] |
| 7.4 | **Index Scans & Plans** | `IndexSelectScan`, `IndexJoinScan`, `IndexSelectPlan`, `IndexJoinPlan` | Integrating indexes into query processing | [ ] |

### Phase 8: Server

| # | Module | Key Structs | Rust Concepts | Status |
|---|--------|-------------|---------------|--------|
| 8.1 | **Embedded Server** | `SimpleDB` | Initialization, facade | [ ] |
| 8.2 | **Network Server** | `Server`, `Connection`, `RemoteStatement`, `RemoteResultSet` | `std::net`, TCP, threading | [ ] |

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              CLIENT LAYER                                   │
│  ┌─────────────────────────────┐    ┌─────────────────────────────────────┐ │
│  │     Embedded Driver         │    │        Network Driver               │ │
│  │  (direct SimpleDB access)   │    │   (TCP connection to server)        │ │
│  └──────────────┬──────────────┘    └──────────────────┬──────────────────┘ │
└─────────────────┼──────────────────────────────────────┼────────────────────┘
                  │                                      │
                  ▼                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                              SERVER LAYER (8.x)                             │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │  SimpleDB { file_mgr, log_mgr, buffer_mgr, metadata_mgr, planner }   │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                              QUERY LAYER                                    │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────────┐  │
│  │   Lexer (6.1)   │─▶│  Parser (6.2)   │─▶│      Planner (6.3)          │  │
│  │   SQL -> Tokens  │  │ Tokens -> AST    │  │  AST -> Plan -> Scan tree    │  │
│  └─────────────────┘  └─────────────────┘  └─────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                              ACCESS LAYER                                   │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │                         Scan Interface (5.1)                           │ │
│  │         TableScan, SelectScan, ProjectScan, ProductScan                │ │
│  │              IndexSelectScan, IndexJoinScan (7.4)                      │ │
│  └────────────────────────────────────────────────────────────────────────┘ │
│  ┌────────────────────────────┐  ┌────────────────────────────────────────┐ │
│  │    Predicates (5.2-5.3)    │  │           Index (7.1-7.3)              │ │
│  │  Constant, Term, Predicate │  │      HashIndex, BTreeIndex            │ │
│  └────────────────────────────┘  └────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                             STORAGE LAYER                                   │
│  ┌──────────────────────────┐  ┌────────────────────────────────────────┐   │
│  │   Record Mgmt (3.x)      │  │         Metadata Mgmt (4.x)            │   │
│  │  Schema, Layout,         │  │  TableMgr, ViewMgr, StatMgr,          │   │
│  │  RecordPage, TableScan   │  │  IndexMgr, MetadataMgr                │   │
│  └──────────────────────────┘  └────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           TRANSACTION LAYER (2.x)                           │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                        Transaction (2.3)                             │   │
│  │    Coordinates all operations, provides ACID guarantees              │   │
│  └───────────────────────────┬──────────────────────────────────────────┘   │
│                              │                                              │
│         ┌────────────────────┼────────────────────┐                         │
│         ▼                    ▼                    ▼                         │
│  ┌─────────────────┐  ┌─────────────┐  ┌───────────────────────┐            │
│  │ ConcurrencyMgr  │  │ RecoveryMgr │  │   BufferList          │            │
│  │     (2.1)       │  │    (2.2)    │  │ (pinned buffers)      │            │
│  │  S/X locking    │  │  WAL undo   │  │                       │            │
│  └────────┬────────┘  └──────┬──────┘  └───────────────────────┘            │
│           │                  │                                              │
│           ▼                  ▼                                              │
│    ┌─────────────┐    ┌─────────────┐                                       │
│    │  LockTable  │    │  LogRecord  │                                       │
│    │ (per block) │    │   (enum)    │                                       │
│    └─────────────┘    └─────────────┘                                       │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                              DISK LAYER                                     │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │                       Buffer Manager (1.3)                             │ │
│  │    Buffer pool: Vec<Buffer>, replacement policy, pin/unpin            │ │
│  └───────────────────────────────┬────────────────────────────────────────┘ │
│                                  │                                          │
│         ┌────────────────────────┴─────────────────────────┐                │
│         ▼                                                  ▼                │
│  ┌─────────────────────────────┐        ┌─────────────────────────────────┐ │
│  │     Log Manager (1.2)       │        │      File Manager (1.1)         │ │
│  │  Append-only WAL            │        │   BlockId, Page, read/write     │ │
│  │  LSN tracking               │        │   block-level disk I/O          │ │
│  └──────────────┬──────────────┘        └────────────────┬────────────────┘ │
│                 │                                        │                  │
│                 └──────────────────┬─────────────────────┘                  │
│                                    ▼                                        │
│                          ┌─────────────────┐                                │
│                          │   OS File I/O   │                                │
│                          │  (std::fs::File)│                                │
│                          └─────────────────┘                                │
└─────────────────────────────────────────────────────────────────────────────┘

Data Flow:
  SQL Query -> Lexer -> Parser -> Planner -> Scan Tree -> Execute via Transaction
                                              ↓
                              TableScan/IndexScan -> RecordPage -> Buffer -> Disk

Write Path:
  INSERT/UPDATE -> Transaction.set_int/set_string
                       ↓
              RecoveryMgr.log_set_* (WAL first!)
                       ↓
              Buffer.set_int/set_string (modify in memory)
                       ↓
              BufferMgr.flush (eventually write to disk)
```

---

## Module Specifications

### 1.1 File Manager
**Book**: Chapter 3

**Purpose**: Read/write fixed-size blocks to disk files. Foundation of everything.

**Core Types**:
```
BlockId { filename: String, blknum: i32 }       -- identifies a disk block
Page { data: Vec<u8> }                          -- in-memory byte buffer
FileMgr { db_directory, blocksize, is_new, open_files }
```

**Key Operations**:
```
FileMgr::new(db_path, blocksize) -> FileMgr     -- create/open database
FileMgr::read(&self, blk, page)                -- read block into page
FileMgr::write(&self, blk, page)               -- write page to block
FileMgr::append(&self, filename) -> BlockId     -- append new block to file
FileMgr::length(&self, filename) -> i32         -- number of blocks in file
FileMgr::is_new(&self) -> bool                  -- was DB newly created?
FileMgr::block_size(&self) -> i32

Page::new(blocksize)                           -- create empty page
Page::new_from_bytes(bytes)                    -- wrap existing bytes
Page::get_int(offset) -> i32                    -- read 4-byte int
Page::set_int(offset, val)                     -- write 4-byte int
Page::get_bytes(offset) -> Vec<u8>              -- read length-prefixed bytes
Page::set_bytes(offset, bytes)                 -- write length-prefixed bytes
Page::get_string(offset) -> String              -- read length-prefixed UTF-8
Page::set_string(offset, val)                  -- write length-prefixed UTF-8
Page::max_length(strlen) -> i32                 -- bytes needed for string
```

**Design Decisions**:
- Block size: 4096 bytes (typical, matches OS page)
- String encoding: 4-byte length prefix + UTF-8 bytes
- File tracking: `HashMap<String, File>` for open files
- Byte order: big-endian (Java default) or little-endian (native)?

**Rust Concepts**:
- `std::fs::File`, `std::io::{Read, Write, Seek}`
- `Result<T, E>` for error handling
- Byte slice manipulation: `&[u8]`, `Vec<u8>`
- `std::path::Path` for file paths
- Interior mutability for open_files map (need to modify in &self methods)

---

### 1.2 Log Manager
**Book**: Chapter 5 (section 5.3)

**Purpose**: Append-only log for Write-Ahead Logging (WAL). Guarantees durability.

**Core Types**:
```
LogMgr {
    file_mgr,
    logfile: String,
    logpage: Page,
    currentblk: BlockId,
    latest_lsn: i32,           -- increments with each append
    last_saved_lsn: i32        -- last LSN flushed to disk
}
LogIterator { file_mgr, blk, page, currentpos }
```

**Key Operations**:
```
LogMgr::new(file_mgr, logfile) -> LogMgr
LogMgr::append(&mut self, record: &[u8]) -> i32 (LSN)   -- add log record, return LSN
LogMgr::flush(&mut self, lsn)                          -- flush if lsn > last_saved_lsn
LogMgr::iterator(&self) -> LogIterator                  -- iterate newest to oldest

LogIterator implements Iterator<Item = Vec<u8>>
```

**Log Block Layout**:
```
┌─────────────────────────────────────────────────────────────┐
│ [boundary: i32] [rec_n] [rec_n-1] ... [rec_1] [free space] │
│       ↑            ↑                              ↑         │
│    offset to    records grow      ←────────── boundary      │
│    last record   right to left                              │
└─────────────────────────────────────────────────────────────┘
```
- Records stored back-to-front (newest at lower addresses)
- Boundary points to last record's start
- When record doesn't fit, flush current block, start new one

**Design Decisions**:
- Why back-to-front? Enables easy backward iteration (for recovery)
- Each record: 4-byte length + content bytes
- LSN: simple counter, not byte offset

**Rust Concepts**:
- Implementing `Iterator` trait
- Interior mutability (`RefCell` or `Mutex`) for thread safety
- Byte manipulation utilities

---

### 1.3 Buffer Manager
**Book**: Chapter 4

**Purpose**: Cache disk blocks in memory. Limited buffer pool (like CPU cache for disk).

**Core Types**:
```
Buffer {
    file_mgr,
    log_mgr,
    contents: Page,
    blk: Option<BlockId>,
    pins: i32,
    txnum: i32,           -- transaction that modified it (-1 if clean)
    lsn: i32              -- LSN of most recent modification (-1 if clean)
}
BufferMgr {
    buffer_pool: Vec<Buffer>,  -- fixed size pool
    num_available: i32,
    max_time: Duration         -- wait timeout for buffer
}
```

**Key Operations**:
```
Buffer::new(file_mgr, log_mgr) -> Buffer
Buffer::contents(&self) -> &Page
Buffer::contents_mut(&mut self) -> &mut Page
Buffer::block(&self) -> Option<&BlockId>
Buffer::set_modified(&mut self, txnum, lsn)    -- mark as dirty
Buffer::is_pinned(&self) -> bool

BufferMgr::new(file_mgr, log_mgr, num_buffs) -> BufferMgr
BufferMgr::available(&self) -> i32
BufferMgr::flush_all(&mut self, txnum)         -- flush all buffers modified by tx
BufferMgr::unpin(&mut self, buff)
BufferMgr::pin(&mut self, blk) -> Buffer ref    -- get buffer for block, wait if needed
```

**Pin/Unpin Protocol**:
1. `pin(blk)`: Find buffer for block (or allocate), increment pin count
2. Client uses buffer
3. `unpin(buff)`: Decrement pin count, buffer can be reused when pins=0

**Replacement Policy** (when no buffer has blk and all pinned):
- Naive: first unpinned buffer (SimpleDB default)
- LRU: least recently unpinned
- Clock: circular scan with "second chance" bit

**Design Decisions**:
- What happens when all buffers pinned? Wait with timeout, abort if exceeded
- Flush strategy: only flush when buffer reused or explicit flush_all
- Buffer identification: index in pool vs. reference/pointer

**Rust Concepts**:
- `Arc<Mutex<>>` for shared mutable state
- Condvar for waiting on available buffer
- Lifetime challenges: returning reference to buffer while holding lock
- Consider `Rc<RefCell<>>` for single-threaded version first

---

### 2.1 Concurrency Manager
**Book**: Chapter 5 (section 5.4)

**Purpose**: S/X locking to ensure serializability. Each transaction gets its own ConcurrencyMgr.

**Core Types**:
```
LockTable {
    locks: HashMap<BlockId, i32>   -- positive = S lock count, -1 = X lock
    -- needs Condvar for waiting
}
ConcurrencyMgr {
    lock_table: shared ref to LockTable,
    locks: HashMap<BlockId, String>   -- "S" or "X" for this tx's locks
}
```

**Key Operations**:
```
LockTable::new() -> LockTable
LockTable::slock(&self, blk)          -- acquire S lock, wait if X held
LockTable::xlock(&self, blk)          -- acquire X lock, wait if any lock held
LockTable::unlock(&self, blk)         -- release lock

ConcurrencyMgr::new(lock_table) -> ConcurrencyMgr
ConcurrencyMgr::slock(&mut self, blk) -- get S lock if don't already have one
ConcurrencyMgr::xlock(&mut self, blk) -- upgrade S to X, or get X
ConcurrencyMgr::release(&mut self)    -- release all locks held by this tx
```

**Lock Compatibility**:
```
             Existing Lock
             None    S      X
Requested S   ✓      ✓      wait
Requested X   ✓     wait    wait
```

**Deadlock Handling**:
- SimpleDB: Wait with timeout (10 seconds), abort if exceeded
- Alternative: Wait-die or wound-wait (not implemented in basic version)

**Design Decisions**:
- Lock granularity: block-level (simple) vs. record-level (more concurrency)
- Lock upgrade: S->X requires waiting for other S locks to release
- When to acquire locks? Strict 2PL: hold all until commit/abort

**Rust Concepts**:
- `Mutex<HashMap<...>>` + `Condvar`
- `wait_timeout` on Condvar
- Ownership of lock_table shared across transactions

---

### 2.2 Recovery Manager
**Book**: Chapter 5 (section 5.3)

**Purpose**: Enable rollback and crash recovery using log records.

**Core Types**:
```
LogRecord (enum):
    Checkpoint
    Start { txnum }
    Commit { txnum }
    Rollback { txnum }
    SetInt { txnum, blk, offset, old_val }
    SetString { txnum, blk, offset, old_val }

RecoveryMgr {
    log_mgr,
    buffer_mgr,
    tx: Transaction ref,
    txnum: i32
}
```

**Key Operations**:
```
RecoveryMgr::new(tx, txnum, log_mgr, buffer_mgr) -> RecoveryMgr
RecoveryMgr::commit(&mut self)          -- write COMMIT, flush log
RecoveryMgr::rollback(&mut self)        -- undo changes, write ROLLBACK
RecoveryMgr::recover(&mut self)         -- on startup: undo all uncommitted
RecoveryMgr::set_int(&mut self, buff, offset, new_val) -> i32 (LSN)
RecoveryMgr::set_string(&mut self, buff, offset, new_val) -> i32 (LSN)

LogRecord::write_to_log(log_mgr, ...) -> LSN   -- serialize and append
LogRecord::from_bytes(bytes) -> LogRecord      -- deserialize
LogRecord::undo(&self, tx)                    -- apply undo operation
```

**Log Record Format** (example for SetInt):
```
[record_type: i32][txnum: i32][filename_len: i32][filename][blknum: i32][offset: i32][old_val: i32]
```

**Recovery Algorithm** (undo-only):
```
1. Read log backwards
2. For each record:
   - COMMIT: add txnum to committed set
   - ROLLBACK: add txnum to committed set (already rolled back)
   - SetInt/SetString: if txnum not committed, call undo()
   - Start: stop when all uncommitted tx's have been fully undone
3. Write CHECKPOINT record
```

**Design Decisions**:
- Undo-only vs. undo-redo: SimpleDB uses undo-only (force policy: flush before commit)
- Old value vs. new value: old value for undo, no redo needed
- CHECKPOINT: quiescent (stop world) in SimpleDB

**Rust Concepts**:
- Enums with associated data
- `match` pattern matching
- Serialization/deserialization to bytes
- `impl` methods on enum variants

---

### 2.3 Transaction
**Book**: Chapter 5

**Purpose**: Coordinate buffer, concurrency, recovery managers. Provide ACID to higher layers.

**Core Types**:
```
Transaction {
    file_mgr,
    buffer_mgr,
    log_mgr,
    concurrency_mgr: ConcurrencyMgr,
    recovery_mgr: RecoveryMgr,
    txnum: i32,
    buffers: BufferList   -- tracks pinned buffers
}
BufferList {
    buffers: HashMap<BlockId, Buffer ref>,
    pins: Vec<BlockId>    -- for multiple pins of same block
}
```

**Key Operations**:
```
Transaction::new(file_mgr, log_mgr, buffer_mgr) -> Transaction
Transaction::commit(&mut self)
Transaction::rollback(&mut self)
Transaction::recover(&mut self)         -- called on startup

-- Buffer operations
Transaction::pin(&mut self, blk)
Transaction::unpin(&mut self, blk)

-- Read operations (acquire S lock)
Transaction::get_int(&mut self, blk, offset) -> i32
Transaction::get_string(&mut self, blk, offset) -> String

-- Write operations (acquire X lock, log before modify)
Transaction::set_int(&mut self, blk, offset, val, ok_to_log: bool)
Transaction::set_string(&mut self, blk, offset, val, ok_to_log: bool)

-- File operations
Transaction::size(&mut self, filename) -> i32
Transaction::append(&mut self, filename) -> BlockId
Transaction::block_size(&self) -> i32
```

**ACID Guarantees**:
- **Atomicity**: Rollback undoes all changes via log
- **Consistency**: Application-level (not DB's job in SimpleDB)
- **Isolation**: S/X locking with strict 2PL
- **Durability**: WAL + force commit (flush before commit returns)

**Design Decisions**:
- `ok_to_log` parameter: false during recovery (don't re-log undo operations)
- Transaction numbering: global counter, increment on each new tx
- Drop behavior: auto-rollback if not committed

**Rust Concepts**:
- RAII: implement `Drop` to auto-rollback
- Coordinating multiple managers (composition)
- Reference management: tx holds refs to managers

---

### 3.1 Record Page
**Book**: Chapter 6

**Purpose**: Store records in fixed-length slots within a block.

**Core Types**:
```
FieldType (enum): Integer, Varchar

FieldInfo { field_type, length }   -- length only for varchar

Schema {
    fields: Vec<String>,           -- field names in order
    info: HashMap<String, FieldInfo>
}

Layout {
    schema,
    offsets: HashMap<String, i32>,
    slot_size: i32
}

RecordPage {
    tx,
    blk: BlockId,
    layout
}
```

**Key Operations**:
```
-- Schema
Schema::new() -> Schema
Schema::add_int_field(&mut self, name)
Schema::add_string_field(&mut self, name, length)
Schema::add_field(&mut self, name, field_type, length)
Schema::add_all(&mut self, other_schema)
Schema::fields(&self) -> &Vec<String>
Schema::has_field(&self, name) -> bool
Schema::field_type(&self, name) -> FieldType
Schema::length(&self, name) -> i32

-- Layout
Layout::new(schema) -> Layout
Layout::schema(&self) -> &Schema
Layout::offset(&self, fieldname) -> i32
Layout::slot_size(&self) -> i32

-- RecordPage
RecordPage::new(tx, blk, layout) -> RecordPage
RecordPage::get_int(&mut self, slot, fieldname) -> i32
RecordPage::get_string(&mut self, slot, fieldname) -> String
RecordPage::set_int(&mut self, slot, fieldname, val)
RecordPage::set_string(&mut self, slot, fieldname, val)
RecordPage::delete(&mut self, slot)               -- mark EMPTY
RecordPage::format(&mut self)                     -- init all slots EMPTY
RecordPage::next_after(&mut self, slot) -> Option<i32>    -- next USED slot
RecordPage::insert_after(&mut self, slot) -> Option<i32>  -- next EMPTY, mark USED
```

**Slot Layout**:
```
┌──────────────────────────────────────────────────────┐
│ [flag: i32] [field1] [field2] ... [fieldN] [padding] │
│     0=EMPTY                                          │
│     1=USED                                           │
└──────────────────────────────────────────────────────┘
slot_size = 4 + sum(field sizes) aligned as needed
```

**Design Decisions**:
- Fixed-length slots (simplifies slot addressing)
- Varchar: allocate max length always (wastes space, but simple)
- Null handling: not supported in SimpleDB
- Field order: as added to schema

**Rust Concepts**:
- `enum` for FieldType
- `HashMap` for field info
- Builder pattern for Schema

---

### 3.2 Table Scan
**Book**: Chapter 6

**Purpose**: Iterate over all records in a table (across all blocks).

**Core Types**:
```
RID { blknum: i32, slot: i32 }    -- record identifier

TableScan {
    tx,
    layout,
    rp: Option<RecordPage>,       -- current record page
    filename: String,
    current_slot: i32
}
```

**Key Operations**:
```
TableScan::new(tx, tablename, layout) -> TableScan
TableScan::close(&mut self)

-- Implements Scan trait
TableScan::before_first(&mut self)           -- position before first record
TableScan::next(&mut self) -> bool            -- move to next record
TableScan::get_int(&self, fieldname) -> i32
TableScan::get_string(&self, fieldname) -> String
TableScan::get_val(&self, fieldname) -> Constant
TableScan::has_field(&self, fieldname) -> bool

-- Implements UpdateScan trait
TableScan::set_int(&mut self, fieldname, val)
TableScan::set_string(&mut self, fieldname, val)
TableScan::set_val(&mut self, fieldname, val)
TableScan::insert(&mut self)                 -- find empty slot, position there
TableScan::delete(&mut self)                 -- delete current record
TableScan::get_rid(&self) -> RID
TableScan::move_to_rid(&mut self, rid)
```

**Iteration Logic**:
```
next():
  slot = rp.next_after(current_slot)
  if slot found: current_slot = slot; return true
  if more blocks: move to next block, try again
  return false

insert():
  slot = rp.insert_after(current_slot)
  if slot found: current_slot = slot; return
  if more blocks: try next block
  else: append new block, format it, use slot 0
```

**Rust Concepts**:
- Trait implementation (Scan, UpdateScan)
- Optional fields (`Option<RecordPage>`)
- Mutable borrows across method calls

---

### 4.1 Table Manager
**Book**: Chapter 7

**Purpose**: Manage catalog of tables. Know what tables exist and their schemas.

**Core Types**:
```
TableMgr {
    tcat_layout: Layout,   -- layout of tblcat
    fcat_layout: Layout    -- layout of fldcat
}
```

**Catalog Tables**:
```
tblcat(tblname: varchar(16), slotsize: int)
fldcat(tblname: varchar(16), fldname: varchar(16), type: int, length: int, offset: int)
```

**Key Operations**:
```
TableMgr::new(is_new, tx) -> TableMgr
TableMgr::create_table(&self, tblname, schema, tx)
TableMgr::get_layout(&self, tblname, tx) -> Layout
```

**Bootstrap Problem**:
- tblcat and fldcat describe themselves
- If DB is new: create catalog tables first
- Constructor must build catalog layouts without reading catalogs

**Rust Concepts**:
- Initialization order matters
- Self-referential data bootstrapping

---

### 4.2 View Manager
**Book**: Chapter 7

**Purpose**: Store SQL view definitions.

**Core Types**:
```
ViewMgr { tbl_mgr }
```

**Catalog Table**:
```
viewcat(viewname: varchar(16), viewdef: varchar(100))
```

**Key Operations**:
```
ViewMgr::new(is_new, tbl_mgr, tx) -> ViewMgr
ViewMgr::create_view(&self, vname, vdef, tx)
ViewMgr::get_view_def(&self, vname, tx) -> Option<String>
```

---

### 4.3 Stat Manager
**Book**: Chapter 7

**Purpose**: Maintain table statistics for query optimizer.

**Core Types**:
```
StatInfo {
    num_blocks: i32,
    num_recs: i32
}
StatMgr {
    tbl_mgr,
    table_stats: HashMap<String, StatInfo>,
    num_calls: i32
}
```

**Key Operations**:
```
StatInfo::new(num_blocks, num_recs) -> StatInfo
StatInfo::blocks_accessed(&self) -> i32
StatInfo::records_output(&self) -> i32
StatInfo::distinct_values(&self, fldname) -> i32   -- estimate: num_recs/3

StatMgr::new(tbl_mgr, tx) -> StatMgr
StatMgr::get_stat_info(&mut self, tblname, layout, tx) -> StatInfo
```

**Refresh Strategy**:
- Recalculate all stats every 100 calls
- Initial calculation on first access per table

**Rust Concepts**:
- Interior mutability for lazy refresh
- HashMap caching

---

### 4.4 Index Manager
**Book**: Chapter 7

**Purpose**: Track which indexes exist on which tables/fields.

**Core Types**:
```
IndexInfo {
    idxname: String,
    fldname: String,
    tx,
    tbl_schema: Schema,
    idx_layout: Layout,
    stat_info: StatInfo
}
IndexMgr {
    layout,   -- layout of idxcat
    tbl_mgr,
    stat_mgr
}
```

**Catalog Table**:
```
idxcat(idxname: varchar(16), tblname: varchar(16), fldname: varchar(16))
```

**Key Operations**:
```
IndexMgr::new(is_new, tbl_mgr, stat_mgr, tx) -> IndexMgr
IndexMgr::create_index(&self, idxname, tblname, fldname, tx)
IndexMgr::get_index_info(&self, tblname, tx) -> HashMap<String, IndexInfo>

IndexInfo::open(&self) -> Box<dyn Index>
IndexInfo::blocks_accessed(&self) -> i32
IndexInfo::records_output(&self) -> i32
IndexInfo::distinct_values(&self, fldname) -> i32
```

---

### 4.5 Metadata Manager
**Book**: Chapter 7

**Purpose**: Facade over all metadata managers.

**Core Types**:
```
MetadataMgr {
    tbl_mgr,
    view_mgr,
    stat_mgr,
    idx_mgr
}
```

**Key Operations**:
```
MetadataMgr::new(is_new, tx) -> MetadataMgr
MetadataMgr::create_table(...)
MetadataMgr::get_layout(...)
MetadataMgr::create_view(...)
MetadataMgr::get_view_def(...)
MetadataMgr::create_index(...)
MetadataMgr::get_index_info(...)
MetadataMgr::get_stat_info(...)
```

---

### 5.1 Scan Interface
**Book**: Chapter 8

**Purpose**: Uniform interface for accessing query results. All query operators implement this.

**Core Types**:
```
Constant (enum):
    Int(i32)
    String(String)

trait Scan {
    fn before_first(&mut self);
    fn next(&mut self) -> bool;
    fn get_int(&self, fldname: &str) -> i32;
    fn get_string(&self, fldname: &str) -> String;
    fn get_val(&self, fldname: &str) -> Constant;
    fn has_field(&self, fldname: &str) -> bool;
    fn close(&mut self);
}

trait UpdateScan: Scan {
    fn set_int(&mut self, fldname: &str, val: i32);
    fn set_string(&mut self, fldname: &str, val: &str);
    fn set_val(&mut self, fldname: &str, val: Constant);
    fn insert(&mut self);
    fn delete(&mut self);
    fn get_rid(&self) -> RID;
    fn move_to_rid(&mut self, rid: RID);
}
```

**Usage Pattern**:
```rust
scan.before_first();
while scan.next() {
    let name = scan.get_string("name");
    let age = scan.get_int("age");
    // process record
}
scan.close();
```

**Rust Concepts**:
- Trait definition
- Trait objects (`Box<dyn Scan>`)
- Trait inheritance (UpdateScan extends Scan)

---

### 5.2 Predicates
**Book**: Chapter 8

**Purpose**: Represent WHERE clause conditions for selection.

**Core Types**:
```
Expression (enum):
    Const(Constant)
    Field(String)

Term {
    lhs: Expression,
    rhs: Expression
}

Predicate {
    terms: Vec<Term>   -- ANDed together
}
```

**Key Operations**:
```
-- Expression
Expression::is_constant(&self) -> bool
Expression::is_field_name(&self) -> bool
Expression::as_constant(&self) -> Option<&Constant>
Expression::as_field_name(&self) -> Option<&String>
Expression::evaluate(&self, scan) -> Constant
Expression::applies_to(&self, schema) -> bool

-- Term (equality: lhs = rhs)
Term::is_satisfied(&self, scan) -> bool
Term::reduction_factor(&self, plan) -> i32     -- selectivity estimate
Term::equates_with_constant(&self, fldname) -> Option<Constant>
Term::equates_with_field(&self, fldname) -> Option<String>
Term::applies_to(&self, schema) -> bool

-- Predicate (AND of terms)
Predicate::new() -> Predicate
Predicate::new_with_term(term) -> Predicate
Predicate::conjoin_with(&mut self, other: Predicate)
Predicate::is_satisfied(&self, scan) -> bool
Predicate::reduction_factor(&self, plan) -> i32
Predicate::select_sub_pred(&self, schema) -> Option<Predicate>  -- terms for this schema
Predicate::join_sub_pred(&self, sch1, sch2) -> Option<Predicate> -- terms joining schemas
Predicate::equates_with_constant(&self, fldname) -> Option<Constant>
Predicate::equates_with_field(&self, fldname) -> Option<String>
```

**Selectivity Estimation**:
- Term "F = c": 1 / distinct_values(F)
- Term "F1 = F2": max(distinct_values(F1), distinct_values(F2))
- Predicate: product of term reduction factors

**Rust Concepts**:
- Enums with variants
- Pattern matching
- Method chaining

---

### 5.3 Query Scans
**Book**: Chapter 8

**Purpose**: Implement relational algebra operators as Scan implementations.

**SelectScan**:
```
SelectScan { scan: Box<dyn Scan>, pred: Predicate }

next(): loop { if !scan.next() return false; if pred.is_satisfied(scan) return true }
```

**ProjectScan**:
```
ProjectScan { scan: Box<dyn Scan>, fields: Vec<String> }

has_field(f): fields.contains(f)
get_*(f): scan.get_*(f)   -- delegate
```

**ProductScan** (nested loop):
```
ProductScan { scan1, scan2, scan2_has_started }

before_first(): scan1.before_first(); scan2_has_started = false
next():
    if scan2_has_started && scan2.next(): return true
    if !scan1.next(): return false
    scan2.before_first(); scan2_has_started = true
    return scan2.next()
```

**Rust Concepts**:
- Composition with trait objects
- Delegation pattern
- Owned vs borrowed scans

---

### 6.1 Lexer
**Book**: Chapter 9

**Purpose**: Tokenize SQL string into tokens.

**Core Types**:
```
Token (enum):
    Keyword(String)     -- SELECT, FROM, WHERE, etc.
    Id(String)          -- identifiers
    IntConstant(i32)
    StringConstant(String)
    Symbol(char)        -- comma, parens, equals, etc.
    Eof
```

**Keywords**: SELECT, FROM, WHERE, AND, INSERT, INTO, VALUES, DELETE, UPDATE, SET, CREATE, TABLE, INT, VARCHAR, VIEW, AS, INDEX, ON

**Key Operations**:
```
Lexer::new(input: &str) -> Lexer

-- Matching (peek)
Lexer::match_delim(&self, d: char) -> bool
Lexer::match_int_constant(&self) -> bool
Lexer::match_string_constant(&self) -> bool
Lexer::match_keyword(&self, w: &str) -> bool
Lexer::match_id(&self) -> bool

-- Eating (consume)
Lexer::eat_delim(&mut self, d: char) -> Result<()>
Lexer::eat_int_constant(&mut self) -> Result<i32>
Lexer::eat_string_constant(&mut self) -> Result<String>
Lexer::eat_keyword(&mut self, w: &str) -> Result<()>
Lexer::eat_id(&mut self) -> Result<String>
```

**Design Decisions**:
- Case-insensitive keywords
- String constants: single-quoted
- Identifiers: start with letter, contain letters/digits/underscore

**Rust Concepts**:
- `Peekable<Chars>`
- `Result` for parse errors
- String manipulation

---

### 6.2 Parser
**Book**: Chapter 9

**Purpose**: Recursive descent parser. Produces AST / data structures for query execution.

**Core Types**:
```
QueryData { fields: Vec<String>, tables: Vec<String>, pred: Predicate }
InsertData { tblname: String, fields: Vec<String>, vals: Vec<Constant> }
DeleteData { tblname: String, pred: Predicate }
ModifyData { tblname: String, fldname: String, new_val: Expression, pred: Predicate }
CreateTableData { tblname: String, schema: Schema }
CreateViewData { viewname: String, query: QueryData }
CreateIndexData { idxname: String, tblname: String, fldname: String }
```

**Grammar** (simplified):
```
Query      := SELECT FieldList FROM TableList [ WHERE Predicate ]
Insert     := INSERT INTO Id ( FieldList ) VALUES ( ConstList )
Delete     := DELETE FROM Id [ WHERE Predicate ]
Modify     := UPDATE Id SET Field = Expression [ WHERE Predicate ]
CreateTable:= CREATE TABLE Id ( FieldDefs )
CreateView := CREATE VIEW Id AS Query
CreateIndex:= CREATE INDEX Id ON Id ( Field )
```

**Key Operations**:
```
Parser::new(input: &str) -> Parser
Parser::query(&mut self) -> Result<QueryData>
Parser::update_cmd(&mut self) -> Result<UpdateCmd enum>
```

**Rust Concepts**:
- Recursive descent structure
- Error propagation with `?`
- Enum for different command types

---

### 6.3 Planner
**Book**: Chapter 10

**Purpose**: Convert parsed SQL to executable scan tree.

**Core Types**:
```
trait Plan {
    fn open(&self) -> Box<dyn Scan>;
    fn blocks_accessed(&self) -> i32;
    fn records_output(&self) -> i32;
    fn distinct_values(&self, fldname: &str) -> i32;
    fn schema(&self) -> &Schema;
}

-- Basic plans
TablePlan { tx, tblname, layout, stat_info }
SelectPlan { plan: Box<dyn Plan>, pred }
ProjectPlan { plan: Box<dyn Plan>, schema }
ProductPlan { plan1, plan2, schema }

-- Planners
BasicQueryPlanner { mdm }
BasicUpdatePlanner { mdm }
Planner { query_planner, update_planner }
```

**Key Operations**:
```
Planner::new(query_planner, update_planner) -> Planner
Planner::create_query_plan(data, tx) -> Box<dyn Plan>
Planner::execute_update(data, tx) -> i32   -- returns affected rows

QueryPlanner::create_plan(data, tx) -> Box<dyn Plan>
UpdatePlanner::execute_insert(data, tx) -> i32
UpdatePlanner::execute_delete(data, tx) -> i32
UpdatePlanner::execute_modify(data, tx) -> i32
UpdatePlanner::execute_create_table(data, tx) -> i32
UpdatePlanner::execute_create_view(data, tx) -> i32
UpdatePlanner::execute_create_index(data, tx) -> i32
```

**Basic Query Planning Algorithm**:
```
1. For each table T in FROM: create TablePlan(T)
2. Take product of all TablePlans (left-associative)
3. Apply SelectPlan with WHERE predicate
4. Apply ProjectPlan with SELECT fields
```

**Note**: BasicQueryPlanner does NOT use indexes. It always does full table scans.
To leverage `IndexSelectPlan`/`IndexJoinPlan` from 7.4, you'd need an index-aware
planner (e.g., `HeuristicQueryPlanner` or `OptimizedQueryPlanner` - covered in
Sciore Chapter 14, stretch goal for this project).

**Rust Concepts**:
- Trait objects for polymorphism
- Box<dyn Plan> for owned trait objects
- Strategy pattern via traits

---

### 7.1 Index Interface
**Book**: Chapter 12

**Purpose**: Common interface for all index implementations.

**Core Types**:
```
trait Index {
    fn before_first(&mut self, search_key: Constant);
    fn next(&mut self) -> bool;
    fn get_data_rid(&self) -> RID;
    fn insert(&mut self, data_val: Constant, data_rid: RID);
    fn delete(&mut self, data_val: Constant, data_rid: RID);
    fn close(&mut self);
}
```

**Usage Pattern**:
```rust
// Point lookup
index.before_first(search_key);
while index.next() {
    let rid = index.get_data_rid();
    table_scan.move_to_rid(rid);
    // process record
}
index.close();
```

---

### 7.2 Hash Index
**Book**: Chapter 12

**Purpose**: Static hash index with fixed number of buckets.

**Core Types**:
```
HashIndex {
    tx,
    idxname: String,
    layout: Layout,
    search_key: Option<Constant>,
    ts: Option<TableScan>   -- scan of current bucket
}
```

**Structure**:
```
Index files: {idxname}0, {idxname}1, ..., {idxname}{NUM_BUCKETS-1}
Each bucket is a table file with schema: (dataval, block, id)
Hash function: hash(search_key) % NUM_BUCKETS
```

**Key Operations**:
```
HashIndex::new(tx, idxname, layout) -> HashIndex
HashIndex::search_cost(num_blocks, rpb) -> i32   -- blocks per bucket

-- Index trait implementation
before_first(key): compute bucket, open TableScan on bucket file
next(): find next matching record in bucket
get_data_rid(): return RID from current index record
insert(val, rid): compute bucket, insert index record
delete(val, rid): compute bucket, find and delete index record
```

**Design Decisions**:
- NUM_BUCKETS: typically 100 (static, never grows)
- Linear scan within bucket (could be slow for skewed data)
- No bucket overflow handling (just more records per bucket)

---

### 7.3 B-Tree Index
**Book**: Chapter 12

**Purpose**: Balanced tree for efficient range queries and point lookups.

**Core Types**:
```
BTreeIndex {
    tx,
    layout_dir: Layout,     -- directory entry layout
    layout_leaf: Layout,    -- leaf entry layout
    idxname: String,
    leaf: Option<BTreeLeaf>,
    root_blk: BlockId
}

BTreeLeaf {
    tx,
    layout,
    contents: BTPage,       -- current leaf page
    current_slot: i32,
    search_key: Constant,
    filename: String
}

BTreeDir {
    tx,
    layout,
    contents: BTPage,       -- current directory page
    filename: String
}

BTPage {
    tx,
    current_blk: Option<BlockId>,
    layout
}

DirEntry { dataval: Constant, block_num: i32 }  -- returned when split occurs
```

**Page Layouts**:
```
Leaf entry: (dataval, block, id)
Dir entry:  (dataval, child_block)

Both page types: [num_recs: i32] [flag: i32] [entry0] [entry1] ...
  flag: 0 = leaf, 1 = directory
```

**Key Operations**:
```
BTreeIndex::new(tx, idxname, layout_leaf) -> BTreeIndex
BTreeIndex::search_cost(num_leaves, rpb) -> i32   -- log(num_leaves) for dir traversal

-- Index trait implementation
before_first(key): traverse from root to leaf, position in leaf
next(): next matching entry in leaf (follow overflow chain if needed)
get_data_rid(): RID from current leaf entry
insert(val, rid): find leaf, insert, split if necessary, propagate up
delete(val, rid): find leaf, delete entry (no rebalancing in SimpleDB)

-- Internal operations
BTreeDir::search(key) -> i32    -- find child block for key
BTreeDir::insert(dir_entry) -> Option<DirEntry>  -- insert, return new entry if split
BTreeDir::make_new_root(entry)  -- create new root after root split

BTreeLeaf::insert(rid) -> Option<DirEntry>  -- insert, return dir entry if split
BTreeLeaf::delete(rid)
BTreeLeaf::next() -> bool
BTreeLeaf::get_data_rid() -> RID
```

**Split Algorithm**:
```
1. Leaf splits: new leaf gets upper half, return (first_key_of_new_leaf, new_block)
2. Dir splits: similar, but promote middle key to parent
3. If root splits: create new root with old root and new block as children
```

**Rust Concepts**:
- Recursive structures
- Tree traversal with mutable references
- Handling splits (returning new entries up the stack)

---

### 7.4 Index Scans & Plans
**Book**: Chapter 12

**Purpose**: Use indexes in query execution.

**Core Types**:
```
IndexSelectScan {
    ts: TableScan,
    idx: Box<dyn Index>,
    val: Constant
}

IndexJoinScan {
    lhs: Box<dyn Scan>,
    idx: Box<dyn Index>,
    joinfield: String,
    rhs: TableScan
}

IndexSelectPlan { plan, idx_info, val }
IndexJoinPlan { plan1, plan2, idx_info, joinfield }
```

**IndexSelectScan** (point lookup via index):
```
before_first(): idx.before_first(val)
next(): if idx.next() { ts.move_to_rid(idx.get_data_rid()); true } else false
```

**IndexJoinScan** (index nested loop join):
```
before_first(): lhs.before_first(); reset_index()
next():
    while true:
        if idx.next(): ts.move_to_rid(idx.get_data_rid()); return true
        if !lhs.next(): return false
        reset_index()
reset_index(): val = lhs.get_val(joinfield); idx.before_first(val)
```

**Cost Estimation**:
```
IndexSelectPlan:
    blocks_accessed = idx_info.blocks_accessed() + idx_info.records_output()
    records_output = idx_info.records_output()

IndexJoinPlan:
    blocks_accessed = plan1.blocks_accessed() + (plan1.records_output() * idx_info.blocks_accessed()) + records_output()
```

---

### 8.1 Embedded Server
**Book**: Chapter 2

**Purpose**: In-process database access. No network overhead.

**Core Types**:
```
SimpleDB {
    file_mgr: FileMgr,
    log_mgr: LogMgr,
    buffer_mgr: BufferMgr,
    lock_table: LockTable,
    metadata_mgr: Option<MetadataMgr>,
    planner: Option<Planner>
}
```

**Key Operations**:
```
SimpleDB::new(db_name, block_size, buffer_size) -> SimpleDB
SimpleDB::new_with_metadata(db_name) -> SimpleDB   -- default sizes, init metadata
SimpleDB::file_mgr(&self) -> &FileMgr
SimpleDB::log_mgr(&self) -> &LogMgr
SimpleDB::buffer_mgr(&self) -> &BufferMgr
SimpleDB::metadata_mgr(&self) -> &MetadataMgr
SimpleDB::planner(&self) -> &Planner
SimpleDB::new_tx(&self) -> Transaction
```

**Initialization**:
```
1. Create/open FileMgr for database directory
2. Create LogMgr (creates log file if new)
3. Create BufferMgr with fixed pool size
4. Create shared LockTable
5. If new DB: create MetadataMgr (initializes catalog tables)
6. Run recovery (Transaction::recover)
7. Create Planner
```

---

### 8.2 Network Server
**Book**: Chapters 2, 11

**Purpose**: TCP server for remote database access.

**Core Types**:
```
Server {
    db: SimpleDB,
    address: SocketAddr
}

Connection {
    db: &SimpleDB,
    current_tx: Option<Transaction>,
    stream: TcpStream
}

RemoteStatement {
    conn: &mut Connection,
    sql: String
}

RemoteResultSet {
    scan: Box<dyn Scan>,
    schema: Schema
}
```

**Wire Protocol** (simple text-based):
```
Client sends:    SQL statement terminated by newline
Server responds: For queries: schema line, then data lines, then "END"
                 For updates: affected row count
                 For errors: "ERROR: message"
```

**Key Operations**:
```
Server::new(db_name) -> Server
Server::start(&self, port)   -- listen and accept connections

Connection::new(db, stream) -> Connection
Connection::handle_request(&mut self)   -- read SQL, execute, send response

-- Statement execution
execute_query(sql) -> RemoteResultSet
execute_update(sql) -> i32
close()
commit()
rollback()
```

**Threading Model**:
- Main thread: accepts connections
- Spawn thread per connection
- Each connection has its own transaction context

**Rust Concepts**:
- `std::net::{TcpListener, TcpStream}`
- `std::thread::spawn`
- Shared state (`Arc<SimpleDB>`)
- Protocol design

---

## Project Structure

> **Note**: Files are created incrementally as each phase is implemented.
> Do NOT scaffold empty files upfront. Create only what's needed for the current phase.

```
tests/
├── file_test.rs
├── buffer_test.rs
├── tx_test.rs
├── record_test.rs
├── sql_test.rs
└── recovery_test.rs
src/
├── file/
│   ├── mod.rs
│   ├── block_id.rs
│   ├── page.rs
│   └── file_mgr.rs
├── log/
│   ├── mod.rs
│   ├── log_mgr.rs
│   └── log_iterator.rs
├── buffer/
│   ├── mod.rs
│   ├── buffer.rs
│   └── buffer_mgr.rs
├── tx/
│   ├── mod.rs
│   ├── transaction.rs
│   ├── buffer_list.rs
│   ├── concurrency/
│   │   ├── mod.rs
│   │   ├── lock_table.rs
│   │   └── concurrency_mgr.rs
│   └── recovery/
│       ├── mod.rs
│       ├── log_record.rs
│       └── recovery_mgr.rs
├── record/
│   ├── mod.rs
│   ├── schema.rs
│   ├── layout.rs
│   ├── record_page.rs
│   ├── rid.rs
│   └── table_scan.rs
├── metadata/
│   ├── mod.rs
│   ├── table_mgr.rs
│   ├── view_mgr.rs
│   ├── stat_mgr.rs
│   ├── index_mgr.rs
│   └── metadata_mgr.rs
├── query/
│   ├── mod.rs
│   ├── constant.rs
│   ├── expression.rs
│   ├── term.rs
│   ├── predicate.rs
│   ├── scan.rs
│   ├── update_scan.rs
│   ├── select_scan.rs
│   ├── project_scan.rs
│   └── product_scan.rs
├── parse/
│   ├── mod.rs
│   ├── lexer.rs
│   ├── parser.rs
│   └── data.rs          # QueryData, InsertData, etc.
├── plan/
│   ├── mod.rs
│   ├── plan.rs          # Plan trait
│   ├── table_plan.rs
│   ├── select_plan.rs
│   ├── project_plan.rs
│   ├── product_plan.rs
│   ├── query_planner.rs
│   ├── update_planner.rs
│   └── planner.rs
├── index/
│   ├── mod.rs
│   ├── index.rs         # Index trait
│   ├── hash/
│   │   ├── mod.rs
│   │   └── hash_index.rs
│   ├── btree/
│   │   ├── mod.rs
│   │   ├── btree_index.rs
│   │   ├── btree_leaf.rs
│   │   ├── btree_dir.rs
│   │   └── bt_page.rs
│   ├── index_select_scan.rs
│   ├── index_join_scan.rs
│   ├── index_select_plan.rs
│   └── index_join_plan.rs
├── server/
│   ├── mod.rs
│   ├── simple_db.rs     # 8.1 Embedded
│   └── network.rs       # 8.2 Network server
├── error.rs             # DbError, Result type alias
├── lib.rs
└── main.rs
Cargo.toml               # thiserror, tempfile (for tests)
```

## Session Notes

<!-- Concise notes from LLM sessions - key decisions, gotchas, Rust learnings -->

| Session | Module | Title | Date | Link |
|---------|--------|-------|------|------|
| 1 | 1.1 File Manager | Pre-Design Concepts | 2026-01-05 | [Details](session_notes/01_file_manager_pre_design.md) |
