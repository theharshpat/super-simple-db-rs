# Session 1: File Manager Concepts (Pre-Design)

**Date**: 2026-01-05
**Module**: 1.1 File Manager

## Concepts Quizzed & Assessed

| Topic | Status | Notes |
|-------|--------|-------|
| Block vs Page | Solid | Block = on-disk, Page = in-memory. Correctly mentioned disabling kernel cache for direct I/O |
| Fixed-size blocks | Good | Mentioned ease of management + atomic write guarantees |
| String serialization | Gap filled | Length-prefix (4 bytes) + UTF-8 bytes. Enables knowing size before reading |
| File handle caching | Gap filled | Minimize syscalls (open/close expensive), one handle per file, kernel FD table management |
| Byte order | Solid | Chose big-endian for consistency + matching Java/Sciore book |

## Rust Concepts Covered

### 1. RefCell - Interior Mutability (Single-Threaded)

**Problem**: Need to mutate through `&self` (e.g., FileMgr::read needs to open files)

**Solution**: `RefCell<HashMap<String, File>>` - runtime borrow checking

```rust
use std::cell::RefCell;

struct FileMgr {
    open_files: RefCell<HashMap<String, File>>,
}

impl FileMgr {
    fn read(&self, filename: &str) {
        let mut files = self.open_files.borrow_mut();  // Runtime check
        // Now we can mutate!
    }
}
```

- `borrow()` returns `Ref<T>` (like `&T`), `borrow_mut()` returns `RefMut<T>` (like `&mut T`)
- Panics at runtime if borrow rules violated
- Use when compiler can't prove safety but we know it's safe

### 2. Send vs Sync Traits

| Trait | Meaning | Example |
|-------|---------|---------|
| `Send` | Type can be moved to another thread (ownership transfer) | `File`: Send |
| `Sync` | `&T` can be shared across threads safely | `File`: NOT Sync (cursor state) |

**Common types**:
- `File`: Send, NOT Sync (cursor state not thread-safe)
- `RefCell`: Send, NOT Sync (runtime checks not atomic)
- `Rc`: NOT Send, NOT Sync (non-atomic refcount)
- `Arc + Mutex`: Send + Sync (for multi-threaded)

### 3. Closures and `move`

**Syntax**: `|params| body` or `|params| { body }`

**Capture modes**:
- Borrow (`&T`) - default when possible
- Mutable borrow (`&mut T`)
- Move (take ownership) - use `move` keyword

**Why `move` with threads**:
```rust
let data = vec![1, 2, 3];
thread::spawn(move || {
    println!("{:?}", data);  // Thread OWNS data now
});
// data no longer accessible - moved into closure
```

Required when closure outlives current scope (thread may run after main ends).

### 4. Clone Trait

- Deep copy via `.clone()`
- `File` is NOT Clone - can't duplicate OS file descriptor in userspace

## Quiz Results (RefCell/Send/Sync)

| Q | Topic | Result |
|---|-------|--------|
| 1 | Multiple `borrow()` on RefCell | Correct - allowed, prints twice |
| 2 | `borrow()` + `borrow_mut()` together | Correct - runtime panic |
| 3 | Why `move` with thread::spawn | Partial - added: thread may outlive stack frame, needs ownership |
| 4 | RefCell is Send but not Sync | Correct - can move to thread, can't share &RefCell across threads |
| 5 | Why `RefCell<HashMap>` not `HashMap<RefCell>` | Correct - mutation at HashMap level (insert new files) |

## Key Decisions Made

- **Byte order**: Big-endian (matches Java/Sciore, more readable in hex dumps)
- **Interior mutability**: `RefCell` for single-threaded FileMgr
- **File handles**: One per file, cached in HashMap

## Next Steps

- DESIGN phase: Sketch out `BlockId`, `Page`, `FileMgr` structs
