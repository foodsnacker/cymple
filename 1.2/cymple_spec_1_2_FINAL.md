# CYMPLE: Official Language Specification

**Version 1.2 FINAL, 2025.11.28 – Procedural Programming with Unicode Symbols**

---

## Overview

Cymple is a procedural programming language using Unicode symbols instead of keywords. It supports both interpretation and compilation with identical semantics. The language enforces memory safety through ownership and move semantics, provides deterministic resource management via RAII, and enables preemptive multitasking with share-nothing tasks communicating over channels.

Key characteristics:
- **Readable & accessible**: Clear indentation, simple rules, consistent symbols
- **Portable**: UTF-8 (NFC-normalized), tabs for block structure
- **Dual-mode**: Interpreted and compiled with identical semantics
- **Safe**: Exclusive ownership, move-only handles, deterministic cleanup
- **Concurrent**: Preemptive tasks with channels, no shared memory
- **No GC**: RAII ensures immediate resource cleanup without garbage collection pauses

---

## Fundamental Principles

1. Unicode symbols replace keywords
2. One statement per line
3. Structure through tabs (indent/outdent)
4. Memory-safe: handles with ownership/move semantics
5. Procedural, no classes or OOP
6. Internal libraries, external plugins
7. Preemptive multitasking with channels
8. Events with dedicated event blocks
9. All functions must be top-level (no nesting)
10. Deterministic resource management (RAII, no garbage collection)

---

## Data Types

### Primitives

**Number (🔢)**
- Unified type for integers and floats
- 64-bit IEEE-754 binary64
- Integers exact up to ±9,007,199,254,740,992
- Range: ≈ ±1.797×10³⁰⁸ ... ±4.94×10⁻³²⁴
- Special values: NaN, +∞, −∞
- Can be `null`

**Bool (✅/✗)**
- ✅ = true
- ✗ = false
- Can be `null`

**String (🔤)**
- Immutable UTF-8 strings
- Escapes: `\"`, `\\`, `\n`, `\t`, `\u{HEX}`
- Example: `🔤greeting ← "Hello World"`
- Single character: `🔤[1]` (no separate char type)
- Can be `null`

### Collections

**List (📋[...])**
- Mutable, 1-based indexing
- Move-only (requires borrowing for access)
- Homogeneous (single type only)
- Example: `📋nums ← [1, 2, 3]`
- Can be `null`

**Map (🗺️{...})**
- String keys only
- Move-only
- Example: `🗺️data ← {"name": "Alice", "age": 30}`
- Can be `null`

**Bytearray (🔣[...])**
- Flat binary data container
- Move-only, 1-based indexing
- Example: `🔣bytes ← [0x01, 0x02, 0xFF]`
- Can be `null`

### Composite Types

**Struct (🧱)**
```
🧱 Person(name: 🔤, age: 🔢)
p ← Person(name: "Alice", age: 30)
💬 p.name
p.age ← 31
```
Can be `null`

**Handle (💾-based)**
- Resource handles with unique ownership
- Move-only semantics
- RAII: deterministic cleanup at block end (no GC)
- Prefix indicates resource type: `🎵snd`, `🖼️img`, `💾file`
- Can be `null_handle`

---

## Null Handling

All types can be `null`. Use the `null` keyword for checks:

```
❓ x = null
    💬 "x is not defined"

❓ handle = null_handle
    💬 "Invalid handle"
```

---

## Variables & Assignment

**Declaration & Assignment:**
```
🔢x ← 42
🔤name ← "Alice"
```

**Constants (📘):**
```
📘 PI ← 3.1415        📝 Global if at top-level
📘 MAX_SIZE ← 1000

🧵 foo()
    📘 LOCAL ← 100    📝 Function-scoped
```

**Handles (move semantics):**
```
💾f1 ← open("data.txt")
💾f2 ← f1  📝 f1 is now null_handle
🗑️💾f2     📝 explicit cleanup
```

---

## Operators

**Assignment:**
- `←` (assignment)

**Comparison:**
- `=` (equal)
- `!=` (not equal)
- `<` `>` `<=` `>=`

**Arithmetic:**
- `+` `-` `*` `/` `%`

**Logical:**
- `&&` (and)
- `||` (or)
- `!` (not)

**Note:** Use `=` for comparison, not `==`. Assignment uses `←` only.

---

## Borrowing (🔗)

Collections are move-only. To access without transferring ownership:

**Read-only borrow:**
```
📋list ← [1, 2, 3]
🔗 list -> L
    💬 L[1]  📝 Prints 1
```

**Mutable borrow:**
```
🔗 list -> mut L
    L[1] ← 99
    L.push(4)
```

**Rules:**
- Borrow is block-scoped
- Cannot store, send via channel, or return borrowed values
- Original value cannot be moved while borrowed
- Ensures exclusive access during borrow

---

## Pattern Matching (🔀)

Match expressions with destructuring and guards:

```
🔀 value
    ➜ 0
        💬 "Zero"
    ➜ 1..10
        💬 "Small"
    ➜ x ❓ x > 100
        💬 "Large: " + x
    ➜ _
        💬 "Default"
```

**With structs:**
```
🔀 person
    ➜ Person(name, age) ❓ age >= 18
        💬 name + " is adult"
    ➜ Person(name, _)
        💬 name + " is minor"
```

**Symbols:**
- `🔀` = match
- `➜` = match arm
- `_` = wildcard
- `❓` = guard condition

---

## Ranges (..)

**Syntax:**
- `a..b` = inclusive range [a, b]
- `a..<b` = exclusive range [a, b)

**In loops:**
```
🔁 i in 1..10
    💬 i  📝 Prints 1 through 10

🔁 i in 10..1
    💬 i  📝 Prints 10 down to 1 (reverse)
```

**With step (⏩):**
```
🔁 x in 1.5..3.5 ⏩ 0.5
    💬 x  📝 1.5, 2.0, 2.5, 3.0, 3.5

🔁 i in 0..100 ⏩ 10
    💬 i  📝 0, 10, 20, ..., 100
```

**In patterns:**
```
🔀 x
    ➜ 0..9
        💬 "Single digit"
    ➜ 10..99
        💬 "Double digit"
```

**List creation:**
```
📋numbers ← [1..100]  📝 List of 1 to 100
```

**Character ranges:**
```
🔁 c in "a".."z"
    💬 c  📝 Each c is 🔤 (single-char string)
```

---

## Control Flow

**Conditional:**
```
❓ x > 0
    💬 "positive"
❓ x < 0
    💬 "negative"
```

**Loop:**
```
🔁 i < 10
    i ← i + 1
    💬 i
```

**Loop with range:**
```
🔁 item in list
    💬 item
```

**Return:**
```
↩ expression
```

**Break/Continue:**
```
🔁 i in 1..100
    ❓ i = 50
        break
    ❓ i % 2 = 0
        continue
    💬 i
```

---

## Functions

**All functions must be top-level (no nesting).**

```
🧵 add(x: 🔢, y: 🔢) -> 🔢
    ↩ x + y

🧵 greet(name: 🔤) -> 🔤
    ↩ "Hello, " + name

result ← add(5, 3)
```

**Functions are also tasks** - can be called synchronously or spawned asynchronously.

**No anonymous functions.** All functions must be named and top-level.

**Function references:**
```
🧵 double(x: 🔢) -> 🔢
    ↩ x * 2

🧵 apply(list: 📋[🔢], fn_name: 🔤) -> 📋[🔢]
    📝 User must pass function name as string
    📝 or implement custom dispatch
```

**Note:** No first-class functions or closures. Keep it simple and procedural.

---

## Built-in Operations

Standard operations are built-in:

```
len ← length("hello")      📝 5
len ← length([1, 2, 3])    📝 3

upper ← to_upper("hello")  📝 "HELLO"
lower ← to_lower("WORLD")  📝 "world"

📋sorted ← sort([3, 1, 2]) 📝 [1, 2, 3]
📋rev ← reverse([1, 2, 3]) 📝 [3, 2, 1]
```

Additional operations can be implemented by users or loaded via plugins.

---

## Tasks & Channels

**Task Definition:**
```
🧵 worker(n: 🔢) -> 🔢
    ↩ n * 2
```

**Synchronous call:**
```
result ← worker(21)  📝 result = 42
```

**Asynchronous spawn:**
```
📡 ch ← 🛰️(capacity: 0)
🚀 ch, worker(21)
🎯 ch → result
💬 result
```

**Channel operations:**
- `🛰️(capacity: n)` = create channel
  - `capacity: 0` = unbuffered (always blocks)
  - `capacity: n` = buffered (n messages before blocking)
- `🚀 ch, value` = send
- `🎯 ch → var` = receive

**Channel semantics:**
- Unbuffered (`capacity: 0`): Sender blocks until receiver ready, receiver blocks until sender ready
- Buffered (`capacity: n`): Sender blocks when buffer full, receiver blocks when buffer empty

**Task events:**
```
spawn worker(10)
    ⏩ progress
        💬 "Progress: " + progress
    ❌ err
        💬 "Error: " + err.message
    ✅ result
        💬 "Success: " + result
```

**Cancel task:**
```
🛑 task_handle
```

**Channel select:**
```
select
    🎯 ch1 → a
        💬 "Received from ch1"
    🚀 ch2, value
        💬 "Sent to ch2"
```

---

## Quantum Operations

Parallel evaluation primitives for deterministic concurrency.

### Race Mode (🌀⚡)

Returns first completed result, cancels others:

```
🌀⚡ result ← [
    fetch_server_eu(),
    fetch_server_us(),
    fetch_server_asia()
]
    ⏩ partial
        💬 "One finished: " + partial
    ✅ winner
        💬 "Winner: " + winner
```

**Semantics:**
- All expressions start as parallel tasks
- First to complete returns its value
- Remaining tasks receive cancel signal (🛑)
- Deterministic: If multiple complete simultaneously, lowest index wins
- Emits ⏩ event for each completion, ✅ for final winner

### Collect Mode (🌀📦)

Waits for all results, returns array:

```
🌀📦 results ← [
    process_chunk_1(),
    process_chunk_2(),
    process_chunk_3()
]
    ✅ all_done
        💬 "All finished: " + all_done.length
```

**Semantics:**
- All expressions start as parallel tasks
- Blocks until all tasks complete
- Returns `📋[result1, result2, ..., resultn]` in index order
- If any task throws ❌, error propagates (other tasks continue unless manually cancelled)
- Emits ✅ when all complete

**Manual cancellation in collect:**
```
🌀📦 results ← [calc_a(), calc_b(), calc_c()]
    ❌ err
        💬 "Error occurred: " + err.message
        ❌ cancel  📝 Explicitly cancel remaining tasks
```

---

## Timer Events

**One-shot timer:**
```
⏱️▶ 100ms
    💬 "Timeout"
```

**Periodic timer:**
```
⏱️🔁 1s
    💬 "Tick"
```

**With cancel:**
```
timer_handle ← ⏱️🔁 500ms
    💬 "Periodic"
    
🛑 timer_handle  📝 Stop timer
```

---

## Error Handling

**Error levels:**
- `⚠️` = Warning (continues execution)
- `❌` = Error (propagates up automatically)
- `💀` = Fatal (terminates)

**Guru meditation (exception handler):**
```
🧘 guru(e)
    🔀 e.code
        ➜ 404
            💬 "Not found: " + e.message
        ➜ 500..599
            💬 "Server error"
        ➜ _
            💀 e  📝 Fatal propagation
```

**Error object structure:**
```
e.code = 404
e.message = "file.txt"
e.type = "NotFound"
```

**HTTP-based error codes:**
```
❌ NotFound(resource: 🔤) -> 404
❌ BadRequest(reason: 🔤) -> 400
❌ Unauthorized(user: 🔤) -> 401
❌ ServerError(details: 🔤) -> 500
```

**Custom errors:**
```
❌ InvalidInput(field: 🔤) -> 422
❌ Timeout(duration: 🔢) -> 408
```

**Error propagation:**
Errors propagate automatically up the call stack until caught by a `🧘` block.

---

## Modules & Plugins

### Internal Modules (🧩)

**Import syntax:**
```
🧩🎵 as audio
```

**Usage (return type ALWAYS required):**
```
🎵snd ← 🧩🎵▶🎵("applause.wav")
🧩🎵▶("kick.wav")  📝 ❌ ERROR: Missing return type
```

**Module call format:**
```
🧩<module>▶<return_type>(args)
```

The return type is mandatory to make handles and resources immediately recognizable.

### External Plugins (🛠️)

```
🛠️graphics
🖼️img ← 🛠️graphics▶🖼️("logo.png")
🛠️graphics▶("render", 🖼️img)
```

---

## Foreign Function Interface (FFI)

Cymple supports calling C libraries through FFI declarations using the `🔗` symbol.

### Syntax

```
🔗 "library_name"
    🧵 function_name(param: type, ...) -> return_type
```

### Pointer Types

FFI functions may use pointer types denoted by `*` prefix:

- `*🔤` = `const char*` (C string)
- `*🔢` = `double*` (number pointer)
- `*💾` = `void*` (opaque pointer)
- `*📋` = array pointer

### Example

```
🔗 "libc.so.6"
    🧵 strlen(s: *🔤) -> 🔢
    🧵 malloc(size: 🔢) -> *💾
    🧵 free(ptr: *💾)

len ← strlen("Hello")
💬 len  📝 5

ptr ← malloc(1024)
free(ptr)
```

### Loading Mechanism

1. Library is loaded via platform-specific dynamic loader (`dlopen` on POSIX, `LoadLibrary` on Windows)
2. Symbols are resolved at first call
3. Functions use C calling convention
4. Type conversions are automatic where possible (e.g., `🔤` → `*🔤`)

### Safety Considerations

**User Responsibility:**
FFI calls bypass Cymple's safety guarantees. Users are responsible for:
- Providing valid pointers
- Matching argument types correctly
- Managing memory allocated by C functions
- Handling potential segmentation faults

**Compiler Behavior:**
- ❌ ERROR: Library not found at compile/link time
- ❌ ERROR: Symbol not found in library
- ⚠️ WARNING: Type mismatch between Cymple and declared C types
- 💀 FATAL: Segfaults in C code are not catchable

### Best Practices

**Wrap FFI calls:**
```
🔗 "libc.so.6"
    🧵 _c_strlen(s: *🔤) -> 🔢

🧵 safe_strlen(s: 🔤) -> 🔢
    ❓ s = ""
        ↩ 0
    ↩ _c_strlen(s)
```

**Check for NULL returns:**
```
ptr ← c_function()
❓ ptr = null
    ❌ FFIError("C function returned NULL") -> 500
```

**Clean up resources:**
```
🧵 process_with_c_lib()
    ptr ← c_allocate()
    📝 ... use ptr ...
    c_free(ptr)  📝 Always free!
```

### Platform Notes

- **Linux/Unix:** Use `.so` libraries (e.g., `"libc.so.6"`)
- **macOS:** Use `.dylib` libraries (e.g., `"libc.dylib"`)
- **Windows:** Use `.dll` libraries (e.g., `"msvcrt.dll"`)

Library paths can be absolute or relative. System library paths are searched automatically.

---

## Colors & Text

**Command mode (global):**
```
🎨:🔵
💬 "Blue text"
🎨:⚪  📝 Reset to default
```

**Inline mode (in strings):**
```
💬 "🔴 Warning ⚪ Normal 🟩 OK"
```

**Custom colors:**
```
📋palette ← [🟥, 🟩, 🟦]
🎨:📋palette[2]
💬 "Palette color 2"

🎨:rgba(255, 0, 0, 0.5)
💬 "Semi-transparent red"
```

---

## Comments

**Single-line:**
```
📝 This is a comment
```

**Multi-line (indent-based):**
```
📝
    This is a multi-line comment
    Everything indented under 📝 is a comment
    Until OUTDENT
    
code_continues_here()
```

---

## Memory Model

### No Garbage Collection

Cymple does **not** use garbage collection. All memory management is deterministic:

**Handles (💾, 🎵, 🖼️, etc.):**
- RAII: Freed at block end (OUTDENT)
- Deterministic: Exact timing known
- Thread-local: Each task owns its resources
- No GC pauses

**Primitives (🔢, 🔤, ✅/✗):**
- Stack-allocated where possible
- Small copies on assignment
- No heap fragmentation

**Collections (📋, 🗺️, 🔣):**
- Move-only: Explicit ownership transfer
- Freed when owner goes out of scope
- No reference counting overhead

### Isolation Guarantees

1. Each task has its own heap
2. Values transfer only via channels
3. Borrowing (🔗) is block-local and ensures exclusive access
4. Move-only values cannot be simultaneously modified by multiple tasks
5. Tasks interact exclusively through channels and events

### Lifetime Rules

**Example:**
```
🧵 process_file(path: 🔤)
    💾f ← open(path)
    📝 file is automatically closed at function end (RAII)
    
    🔗 data -> D
        💬 D.length
    📝 borrow ends here
    
    ↩ "done"
```

**Result:**
- ✅ Zero GC pauses
- ✅ Predictable memory usage
- ✅ Suitable for real-time systems
- ✅ Lower latency than Go/Python/JavaScript

---

## Lexical Rules

**Encoding:** UTF-8, NFC-normalized before tokenization

**Line endings:** `\n` (all others normalized)

**Indentation:** Only TAB (U+0009) for block structure; spaces not allowed

**Comments:** `📝` single-line or indent-based multi-line

**Tokenization:** Grapheme-sensitive (UAX-29), identifiers per UAX-31 (emoji allowed)

**Scope:**
- Global: Top-level declarations
- Function: Inside function body
- Block: Inside control flow blocks

---

## Symbol Reference

### Basic Language Elements
| Symbol | Meaning |
|--------|---------|
| `←` | Assignment |
| `↩` | Return |
| `❓` | If/Conditional |
| `🔁` | Loop |
| `📝` | Comment |
| `📘` | Constant |
| `⏩` | Step (in ranges) |

### Data Types
| Symbol | Type |
|--------|------|
| `🔢` | Number |
| `🔤` | String |
| `✅` | True |
| `✗` | False |
| `📋` | List |
| `🗺️` | Map |
| `🔣` | Bytearray |
| `🧱` | Struct |

### Pattern Matching
| Symbol | Meaning |
|--------|---------|
| `🔀` | Match |
| `➜` | Match arm |
| `_` | Wildcard |

### Tasks & Concurrency
| Symbol | Meaning |
|--------|---------|
| `🧵` | Function/Task |
| `📡` | Channel variable |
| `🛰️` | Create channel |
| `🚀` | Send to channel |
| `🎯` | Receive from channel |
| `🛑` | Cancel task |
| `🌀⚡` | Quantum race |
| `🌀📦` | Quantum collect |

### Events
| Symbol | Event |
|--------|-------|
| `✅` | Success |
| `❌` | Error |
| `⏩` | Progress |
| `⏹️` | Stopped |
| `🔁` | Repeat |

### Memory & Ownership
| Symbol | Meaning |
|--------|---------|
| `🔗` | Borrow (also FFI) |
| `🗑️` | Explicit free |

### Modules
| Symbol | Meaning |
|--------|---------|
| `🧩` | Internal module |
| `🛠️` | External plugin |
| `▶` | Call operator (with return type) |

### I/O
| Symbol | Meaning |
|--------|---------|
| `💬` | Print |
| `🎨` | Color command |

### Timers
| Symbol | Meaning |
|--------|---------|
| `⏱️▶` | One-shot timer |
| `⏱️🔁` | Periodic timer |

### Error Handling
| Symbol | Meaning |
|--------|---------|
| `🧘` | Guru meditation (exception handler) |
| `⚠️` | Warning |
| `❌` | Error |
| `💀` | Fatal |

---

## Complete Examples

### Example 1: Fibonacci with Parallel Tasks

```
🧵 fib(n: 🔢) -> 🔢
    ❓ n <= 1
        ↩ n
    ↩ fib(n - 1) + fib(n - 2)

🧵 fib_parallel(n: 🔢) -> 🔢
    ❓ n <= 1
        ↩ n
    
    🌀📦 results ← [
        fib_parallel(n - 1),
        fib_parallel(n - 2)
    ]
        ✅ done
            ↩ done[1] + done[2]

💬 "Fibonacci(10) = " + fib_parallel(10)
```

### Example 2: File Processing with Error Handling

```
🧵 process_files(paths: 📋[🔤]) -> 🔢
    processed ← 0
    
    🔁 path in paths
        🧘 guru(e)
            🔀 e.code
                ➜ 404
                    💬 "⚠️ Skipping missing file: " + path
                    continue
                ➜ 403
                    ❌ Unauthorized(path) -> 403
                ➜ _
                    💀 e
        
        💾f ← open(path)
        🔗 f -> F
            data ← F.read()
            💬 "Processing: " + path + " (" + length(data) + " bytes)"
        
        processed ← processed + 1
    
    ↩ processed

📋files ← ["data1.txt", "data2.txt", "data3.txt"]
count ← process_files(files)
💬 "Processed " + count + " files"
```

### Example 3: Web Server with Pattern Matching

```
🧱 Request(method: 🔤, path: 🔤, body: 🔤)
🧱 Response(status: 🔢, body: 🔤)

🧵 handle_request(req: Request) -> Response
    🔀 (req.method, req.path)
        ➜ ("GET", "/")
            ↩ Response(status: 200, body: "Welcome!")
        ➜ ("GET", path) ❓ starts_with(path, "/api/")
            ↩ api_handler(path)
        ➜ ("POST", "/submit")
            ↩ Response(status: 201, body: "Created")
        ➜ _
            ↩ Response(status: 404, body: "Not Found")

🧵 api_handler(path: 🔤) -> Response
    🔀 path
        ➜ "/api/status"
            ↩ Response(status: 200, body: "OK")
        ➜ _
            ↩ Response(status: 404, body: "API endpoint not found")

📝 Simulated request
req ← Request(method: "GET", path: "/api/status", body: "")
resp ← handle_request(req)
💬 "Status: " + resp.status + ", Body: " + resp.body
```

### Example 4: Concurrent Image Processing

```
🧵 process_image(path: 🔤) -> 🖼️
    🖼️img ← load_image(path)
    🖼️processed ← apply_filter(img, "grayscale")
    ↩ processed

🧵 batch_process(paths: 📋[🔤])
    🌀📦 images ← [
        process_image(paths[1]),
        process_image(paths[2]),
        process_image(paths[3]),
        process_image(paths[4])
    ]
        ✅ done
            💬 "Processed " + length(done) + " images"
            🔁 img in done
                save_image(img)

📋files ← ["img1.png", "img2.png", "img3.png", "img4.png"]
batch_process(files)
```

### Example 5: FFI with C Library

```
🔗 "libm.so.6"
    🧵 sqrt(x: 🔢) -> 🔢
    🧵 pow(x: 🔢, y: 🔢) -> 🔢
    🧵 sin(x: 🔢) -> 🔢

🧵 calculate_distance(x1: 🔢, y1: 🔢, x2: 🔢, y2: 🔢) -> 🔢
    dx ← x2 - x1
    dy ← y2 - y1
    dist_squared ← pow(dx, 2) + pow(dy, 2)
    ↩ sqrt(dist_squared)

distance ← calculate_distance(0, 0, 3, 4)
💬 "Distance: " + distance  📝 5.0
```

---

## Design Rationale

### Why No Generics?

Cymple prioritizes simplicity over abstraction. Users can implement type-specific operations or use dynamic dispatch patterns. The absence of generics keeps the type system simple and compilation fast.

### Why No Anonymous Functions?

Anonymous functions and closures introduce complexity in scoping, capture semantics, and memory management. Cymple's top-level-only functions ensure:
- Clear ownership of captured variables (none)
- No hidden allocations
- Predictable performance
- Simpler implementation

### Why No Garbage Collection?

RAII provides deterministic cleanup without GC pauses. This makes Cymple suitable for:
- Real-time systems
- Embedded systems
- Low-latency applications
- Predictable performance

### Why Unicode Symbols?

Symbols provide:
- Visual distinction (🔢 vs `number`)
- Reduced verbosity
- International neutrality (no English keywords)
- Accessibility for non-English speakers

### Why Share-Nothing Concurrency?

Share-nothing prevents data races by design:
- No locks needed
- No race detectors needed
- Compiler enforces safety
- Simpler mental model than shared memory

---

**End of Specification**

Version 1.2 FINAL represents a complete, consistent language design focused on safety, simplicity, and deterministic execution.

For implementation notes, benchmark scenarios, and language comparisons, see accompanying documentation.
