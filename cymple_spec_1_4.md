# CYMPLE: Official Language Specification

**Version 1.4, 2025.12.04 – Procedural Programming with Unicode Symbols**

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

## What's New in Version 1.4

### 1. Enhanced Quantum Operations

Version 1.4 significantly improves parallel task execution with better control and visibility:

#### **Timeout with Units**
```cymple
🌀⚡ 🔤result ← [fetch_eu(), fetch_us(), fetch_asia()]
    ⏱️ 3s
        💬 "Timeout after 3 seconds"
        ↩ cached_value()
    ✅ 🔤winner
        ↩ 🔤winner
```

**Supported time units:**
- `ms` - milliseconds
- `s` - seconds  
- `m` - minutes
- `h` - hours
- No unit or variable → milliseconds (default)

#### **Progress Events with Frequency Control**
```cymple
🌀📦 📋results ← [100 tasks...]
    ⏩ 📋partial every 10
        💬 "Progress: 📋partial.length/100"
        🖼️update_progress(📋partial.length)
    ✅ 📋all
        ↩ 📋all
```

The `every N` modifier reduces progress event frequency for better performance.

#### **Early Cancellation with 🛑**
```cymple
🌀📦 📋results ← [search_db1(), search_db2(), search_db3()]
    ⏩ 📋partial every 5
        ❓ 📋partial.length >= 20
            💬 "Found enough results"
            🛑
            ↩ 📋partial
    ✅ 📋all
        ↩ 📋all
```

Use `🛑` to stop all running tasks and return partial results - consistent with channels and loops.

#### **Total Failure Event**
```cymple
🌀📦 📋results ← [download1(), download2(), download3()]
    ✅ 📋all
        📋ok ← filter_ok(📋all)
        ↩ 📋ok
    ❌ 🔤total_failure
        💬 "All tasks failed"
        🧘 🔤total_failure
            ↩ false
```

The `❌ total_failure` event fires only when every single task has failed.

### 2. Simplified Race Operations

Race no longer emits progress events (which made no sense since other tasks are cancelled):

```cymple
🌀⚡ 🔤winner ← [task1(), task2(), task3()]
    ⏱️ 5s
        💬 "Timeout"
        ↩ fallback()
    ✅ 🔤winner
        💬 "Winner: 🔤winner"
        ↩ 🔤winner
    ❌ 🔤error
        🧘 🔤error
            ↩ default_value()
```

**Race events:**
- `⏱️` - Timeout (optional)
- `✅` - Success (required)
- `❌` - Error (optional)

### 3. Consistent Stop Signal

The `🛑` symbol now works consistently across:
- **Quantum Operations** - stops all running tasks
- **Channels** - closes channel
- **Loops** - breaks out of loop

```cymple
📝 In Collect:
⏩ 📋partial
    ❓ condition
        🛑

📝 In Channel:
🔁 msg in 📨channel
    ❓ msg == "stop"
        🛑

📝 In Task:
🧵 long_task()
    🔁 i in 1..1000
        ❓ check_cancelled()
            🛑
```

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
- **String interpolation**: Variables can be embedded directly using their emoji prefix
- Example: `🔤greeting ← "Hello World"`
- Example with interpolation: `💬 "Hello, 🔤name!"`
- Property access: `🔤text.length` returns number of characters
- Single character: `🔤text[1]` (no separate char type)
- Can be `null`

### Collections

**List (📋[...])**
- Mutable, 1-based indexing
- Move-only (requires borrowing for access)
- Homogeneous (single type only)
- Example: `📋nums ← [1, 2, 3]`
- Property access: `📋list.length` returns number of items
- Can be `null`

**Map (🗺️{...})**
- String keys only
- Move-only
- Example: `🗺️data ← {"name": "Alice", "age": 30}`
- Property access: `🗺️map.size` returns number of key-value pairs
- Can be `null`

**Bytearray (🔣[...])**
- Flat binary data container
- Move-only, 1-based indexing
- Example: `🔣bytes ← [0x01, 0x02, 0xFF]`
- Property access: `🔣bytes.length` returns number of bytes
- Can be `null`

### Composite Types

**Struct (🧱)**
```cymple
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

```cymple
❓ x == null
    💬 "x is not defined"

❓ handle == null_handle
    💬 "Invalid handle"
```

---

## Variables & Assignment

**Declaration & Assignment:**
```cymple
🔢x ← 42
🔤name ← "Alice"
```

**Type prefix is mandatory** - this is a core feature that makes variables immediately recognizable and prevents ambiguity.

**Constants (📘):**
```cymple
📘 PI ← 3.1415        📝 Global if at top-level
📘 MAX_SIZE ← 1000

🧵 foo()
    📘 LOCAL ← 100    📝 Function-scoped
```

**Handles (move semantics):**
```cymple
💾f1 ← open("data.txt")
💾f2 ← f1  📝 f1 is now null_handle
🗑️💾f2     📝 explicit cleanup
```

---

## String Operations

**Concatenation (old style, still works):**
```cymple
🔤greeting ← "Hello, " + 🔤name + "!"
```

**Interpolation (v1.3+, recommended):**
```cymple
🔤greeting ← "Hello, 🔤name!"
💬 "Count: 🔢count, Name: 🔤name"
```

**String methods:**
```cymple
🔢len ← 🔤text.length
🔤upper ← 🔤text.upper()
🔤lower ← 🔤text.lower()
🔤sub ← 🔤text.substr(1, 5)
✅contains ← 🔤text.contains("xyz")
📋parts ← 🔤text.split(",")
```

---

## Collection Operations

**Length/Size access:**
```cymple
📋nums ← [1, 2, 3, 4, 5]
🔢count ← 📋nums.length

🗺️data ← {"a": 1, "b": 2, "c": 3}
🔢size ← 🗺️data.size

🔣bytes ← [0xFF, 0xAA, 0xBB]
🔢len ← 🔣bytes.length
```

---

## Operators

**Assignment:**
- `←` (assignment)

**Comparison:**
- `==` (equal)
- `!=` (not equal)
- `<` `>` `<=` `>=`

**Arithmetic:**
- `+` `-` `*` `/` `%`

**Logical:**
- `&&` (AND)
- `||` (OR)
- `!` (NOT)

**Bitwise:**
- `&` (AND)
- `|` (OR)
- `^` (XOR)
- `<<` `>>` (shift)

---

## Borrowing (🔗)

Collections are move-only. To access without transferring ownership:

**Read-only borrow:**
```cymple
📋list ← [1, 2, 3]
🔗 list -> L
    💬 L[1]  📝 Prints 1
```

**Mutable borrow:**
```cymple
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

```cymple
🔀 value
    ➜ 0
        💬 "Zero"
    ➜ 1..10
        💬 "Small"
    ➜ x ❓ x > 100
        💬 "Large: 🔢x"
    ➜ _
        💬 "Default"
```

**With structs:**
```cymple
🔀 person
    ➜ Person(name: "Alice", age: a)
        💬 "Alice is 🔢a years old"
    ➜ Person(name: n, age: a) ❓ a >= 18
        💬 "Adult: 🔤n"
    ➜ _
        💬 "Other person"
```

---

## Control Flow

### If/Else

```cymple
❓ x > 0
    💬 "positive"
⤵️
    💬 "not positive"

❓ age >= 18
    💬 "Adult"
⤵️
    ❓ age >= 13
        💬 "Teenager"
    ⤵️
        💬 "Child"
```

### Loops

**While-style:**
```cymple
🔁 condition
    📝 statements
```

**Range loop:**
```cymple
🔁 i in 1..10
    💬 "Number: 🔢i"

🔁 i = 1..10        📝 Alternative syntax
    💬 "Number: 🔢i"
```

**Collection loop:**
```cymple
🔁 item in list
    💬 "Item: 🔤item"
```

**With step:**
```cymple
🔁 i in 1..100 ⏩ 10
    💬 "Step: 🔢i"  📝 10, 20, 30, ...
```

---

## Functions

**Declaration:**
```cymple
🧵 add(a: 🔢, b: 🔢) -> 🔢
    ↩ a + b

🧵 greet(name: 🔤)     📝 No return type for void functions
    💬 "Hello, 🔤name!"
```

**Multiple return values:**
```cymple
🧵 divide(a: 🔢, b: 🔢) -> (🔢, ✅)
    ❓ b == 0
        ↩ 0, ✗
    ↩ a / b, ✅
```

**Calling:**
```cymple
🔢result ← add(5, 3)
greet("Alice")
```

---

## Quantum Operations (v1.4 Enhanced)

Quantum operations enable efficient parallel task execution with full control over timing, cancellation, and error handling.

### Race (`🌀⚡`) - First Result Wins

Starts multiple tasks in parallel and returns the result of the first one to complete. All other tasks are automatically cancelled.

**Minimal syntax:**
```cymple
🌀⚡ 🔤result ← [task1(), task2(), task3()]
    ✅ 🔤winner
        ↩ 🔤winner
```

**Full syntax with all events:**
```cymple
🌀⚡ 🔤result ← [fetch_eu(), fetch_us(), fetch_asia()]
    ⏱️ 5s
        💬 "All servers timeout"
        🧘 "Timeout"
            ↩ cached_value()
    
    ✅ 🔤winner
        💬 "Fastest server: 🔤winner"
        ↩ 🔤winner
    
    ❌ 🔤error
        💬 "All tasks failed: 🔤error"
        🧘 🔤error
            ↩ fallback()
```

**Race events:**
| Event | When | Required |
|-------|------|----------|
| `⏱️` | Timeout reached | Optional |
| `✅` | First task succeeds | **Required** |
| `❌` | All tasks fail | Optional |

**Semantics:**
- Starts all tasks simultaneously
- Returns first successful result
- Automatically cancels remaining tasks with `🛑`
- Deterministic: If multiple tasks finish simultaneously, lowest index wins
- Without timeout handler: Timeout throws `❌ "Timeout"` automatically

### Collect (`🌀📦`) - Gather All Results

Starts multiple tasks in parallel and collects results from all tasks.

**Minimal syntax:**
```cymple
🌀📦 📋results ← [task1(), task2(), task3()]
    ✅ 📋all
        ↩ 📋all
```

**Full syntax with progress and cancellation:**
```cymple
🌀📦 📋results ← [process_batch1(), process_batch2(), process_batch3()]
    ⏱️ 30s
        💬 "Timeout - 📋results.length tasks completed"
        ↩ 📋results
    
    ⏩ 📋partial every 5
        🔢percent ← (📋partial.length * 100) / 🔢total
        💬 "Progress: 🔢percent%"
        🖼️update_progress(🔢percent)
        
        📝 Early cancellation
        ❓ 📋partial.length >= 50
            💬 "Found enough results"
            🛑
            ↩ 📋partial
    
    ✅ 📋all
        📋ok ← filter_ok(📋all)
        📋err ← filter_error(📋all)
        💬 "Success: 📋ok.length, Errors: 📋err.length"
        ↩ 📋ok
    
    ❌ 🔤total_failure
        💬 "All tasks failed completely"
        🧘 🔤total_failure
            ↩ false
```

**Collect events:**
| Event | When | Required |
|-------|------|----------|
| `⏱️` | Timeout reached | Optional |
| `⏩ every N` | Every N tasks complete | Optional |
| `✅` | At least 1 task succeeds | **Required** |
| `❌ total_failure` | ALL tasks fail | Optional |

**Progress frequency control:**
```cymple
⏩ 📋partial every 10       📝 Every 10 tasks
⏩ 📋partial every 🔢interval  📝 Variable frequency
```

**Early cancellation:**
```cymple
⏩ 📋partial
    ❓ condition
        🛑  📝 Stops all running tasks
        ↩ 📋partial
```

**Semantics:**
- Starts all tasks simultaneously
- `✅` fires when at least one task succeeds
- Failed tasks are included in results (use `filter_ok()`/`filter_error()`)
- `🛑` stops all running tasks, keeps completed results
- `❌ total_failure` only fires if every single task failed
- Without timeout handler: Returns partial results at timeout

### Timeout Units (v1.4)

**With time units:**
```cymple
⏱️ 500ms   📝 500 milliseconds
⏱️ 5s      📝 5 seconds
⏱️ 2m      📝 2 minutes
⏱️ 1h      📝 1 hour
```

**With variables (always milliseconds):**
```cymple
🔢timeout ← 3000
⏱️ 🔢timeout     📝 3000 milliseconds = 3 seconds
```

**Default (no unit):**
```cymple
⏱️ 5000     📝 5000 milliseconds
```

### Error Handling in Quantum Operations

**Manual filtering:**
```cymple
✅ 📋all
    📋successes ← filter_ok(📋all)
    📋failures ← filter_error(📋all)
    ↩ 📋successes
```

**Result type properties:**
```cymple
✅ 📋all
    🔁 result in 📋all
        ❓ result.is_ok
            process(result.value)
        ⤵️
            💬 "Error: result.error"
```

**Guru meditation on total failure:**
```cymple
❌ 🔤total_failure
    🧘 🔤total_failure
        💬 "All operations failed"
        ↩ false
```

---

## Tasks & Channels

**Channel creation:**
```cymple
📡ch ← 🛰️🔢  📝 Channel for numbers
```

**Sending:**
```cymple
🚀 ch, 42
```

**Receiving:**
```cymple
🔢value ← 🎯 ch
```

**Task as function:**
```cymple
🧵 worker(📡input, 📡output)
    🔁 msg in input
        🔢result ← process(msg)
        🚀 output, 🔢result
```

**Starting task:**
```cymple
📡in ← 🛰️🔢
📡out ← 🛰️🔢

worker(📡in, 📡out)  📝 Runs concurrently
```

**Stopping tasks:**
```cymple
🛑  📝 Signals task to stop
```

---

## Events

Tasks can emit events with dedicated event blocks:

```cymple
🧵 download(url: 🔤)
    ✅ data
        💬 "Downloaded 🔢data.length bytes"
    
    ❌ error
        💬 "Error: 🔤error"
    
    ⏩ progress
        💬 "Progress: 🔢progress%"
```

**Event types:**
- `✅` Success
- `❌` Error
- `⏩` Progress
- `⏹️` Stopped

---

## Timers

**One-shot timer:**
```cymple
⏱️▶ 1000, my_callback  📝 Calls after 1000ms
```

**Periodic timer:**
```cymple
⏱️🔁 1000, my_callback  📝 Calls every 1000ms
```

---

## Error Handling (Guru Meditation 🧘)

Guru meditation provides interactive error handling with recovery options:

```cymple
🧵 risky_operation()
    🧘 guru(e)
        🔀 e.code
            ➜ 404
                💬 "⚠️ Not found"
                ↩
            ➜ 403
                ❌ Unauthorized() -> 403
            ➜ _
                💀 e  📝 Fatal error
    
    📝 risky code here
    💾f ← open("file.txt")
    🔗 f -> F
        📋data ← F.read()
```

**Error levels:**
- `⚠️` Warning (continues)
- `❌` Error (can be handled)
- `💀` Fatal (terminates)

---

## Modules

**Internal module:**
```cymple
🧩 "math"
🔢result ← sqrt(16)
```

**External plugin:**
```cymple
🛠️ "custom_plugin"
process_data()
```

---

## Foreign Function Interface (FFI)

Link to C libraries:

```cymple
🔗 "libc.so.6"
    🧵 strlen(s: *🔤) -> 🔢
    🧵 malloc(size: 🔢) -> *💾
    🧵 free(ptr: *💾)

🔢len ← strlen("Hello")
💬 "Length: 🔢len"
```

**Pointer types:**
- `*🔢` = number pointer
- `*🔤` = string pointer
- `*💾` = void pointer
- `*📋` = array pointer

---

## Colors & Text

**Command mode:**
```cymple
🎨:🔵
💬 "Blue text"
🎨:⚪  📝 Reset
```

**Inline mode:**
```cymple
💬 "🔴 Warning ⚪ Normal 🟩 OK"
```

---

## Comments

**Single-line:**
```cymple
📝 This is a comment
```

**Multi-line:**
```cymple
📝
    Multi-line comment
    Everything indented is a comment
    
code_continues()
```

---

## Memory Model

### No Garbage Collection

Cymple does **not** use garbage collection:

**Handles:**
- RAII: Freed at block end (OUTDENT)
- Deterministic: Exact timing known
- Thread-local: Each task owns its resources

**Collections:**
- Move-only: Explicit ownership transfer
- Freed when owner goes out of scope

### Isolation Guarantees

1. Each task has its own heap
2. Values transfer only via channels
3. Borrowing ensures exclusive access
4. Move-only prevents simultaneous modification
5. Tasks interact exclusively through channels and events

---

## Symbol Reference

### Basic Language
| Symbol | Meaning |
|--------|---------|
| `←` | Assignment |
| `↩` | Return |
| `❓` | If |
| `⤵️` | Else |
| `🔁` | Loop |
| `📝` | Comment |
| `📘` | Constant |

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
| `📡` | Channel |
| `🛰️` | Create channel |
| `🚀` | Send |
| `🎯` | Receive |
| `🛑` | Stop/Cancel |
| `🌀⚡` | Quantum race |
| `🌀📦` | Quantum collect |

### Events
| Symbol | Event |
|--------|-------|
| `✅` | Success |
| `❌` | Error |
| `⏩` | Progress |
| `⏹️` | Stopped |

### Memory
| Symbol | Meaning |
|--------|---------|
| `🔗` | Borrow/FFI |
| `🗑️` | Explicit free |

### Timers (v1.4)
| Symbol | Meaning |
|--------|---------|
| `⏱️` | Timeout |
| `⏱️▶` | One-shot timer |
| `⏱️🔁` | Periodic timer |

### Error Handling
| Symbol | Meaning |
|--------|---------|
| `🧘` | Guru meditation |
| `⚠️` | Warning |
| `❌` | Error |
| `💀` | Fatal |

---

## Complete Examples

### Example 1: Multi-Server Search with Quantum Race

```cymple
🧵 fetch_fastest(🔤query) -> 🔤
    🌀⚡ 🔤result ← [
        search_eu(🔤query),
        search_us(🔤query),
        search_asia(🔤query)
    ]
        ⏱️ 3s
            💬 "All servers too slow"
            ↩ cached_search(🔤query)
        
        ✅ 🔤winner
            💬 "Fastest server responded"
            ↩ 🔤winner
        
        ❌ 🔤error
            🧘 🔤error
                💬 "Search failed: 🔤error"
                ↩ ""
```

### Example 2: Batch Processing with Progress

```cymple
🧵 process_images(📋files) -> 📋
    🔢batch_size ← 10
    📋batches ← split_batches(📋files, 🔢batch_size)
    
    🌀📦 📋results ← create_batch_tasks(📋batches)
        ⏱️ 5m
            💬 "Timeout - 📋results.length batches done"
            ↩ 📋results
        
        ⏩ 📋partial every 5
            🔢done ← 📋partial.length
            🔢total ← 📋batches.length
            🔢percent ← (🔢done * 100) / 🔢total
            💬 "Progress: 🔢percent%"
            🖼️update_ui(🔢percent)
            
            📝 User cancellation
            ❓ 🔘user_cancelled
                💬 "Cancelled by user"
                🛑
                ↩ filter_ok(📋partial)
        
        ✅ 📋all
            📋ok ← filter_ok(📋all)
            📋err ← filter_error(📋all)
            💬 "Done: 📋ok.length OK, 📋err.length errors"
            ↩ 📋ok
        
        ❌ 🔤total_failure
            💬 "All batches failed"
            ↩ []
```

### Example 3: Parallel Fibonacci

```cymple
🧵 fib(n: 🔢) -> 🔢
    ❓ n <= 1
        ↩ n
    
    🌀📦 📋results ← [fib(n - 1), fib(n - 2)]
        ✅ 📋done
            ↩ 📋done[1] + 📋done[2]

🔢result ← fib(10)
💬 "Fibonacci(10) = 🔢result"
```

### Example 4: Timeout Comparison

```cymple
🧵 compare_timeouts()
    📝 Milliseconds (default)
    🌀⚡ result1 ← [slow_task()]
        ⏱️ 5000
            💬 "Timeout after 5000ms"
    
    📝 Seconds
    🌀⚡ result2 ← [slow_task()]
        ⏱️ 5s
            💬 "Timeout after 5 seconds"
    
    📝 Variable (msec)
    🔢timeout_ms ← 3000
    🌀⚡ result3 ← [slow_task()]
        ⏱️ 🔢timeout_ms
            💬 "Timeout after 3 seconds"
```

### Example 5: Early Cancellation Search

```cymple
🧵 search_until_enough(🔤query) -> 📋
    🌀📦 📋results ← [
        search_db1(🔤query),
        search_db2(🔤query),
        search_db3(🔤query),
        search_db4(🔤query),
        search_db5(🔤query)
    ]
        ⏩ 📋partial
            💬 "Found: 📋partial.length results"
            
            📝 Stop when we have enough
            ❓ 📋partial.length >= 20
                💬 "Enough results found"
                🛑
                ↩ 📋partial
        
        ✅ 📋all
            💬 "All databases searched"
            ↩ 📋all
```

---

## Design Rationale

### Why Explicit Type Prefixes?

The emoji type prefixes are mandatory because they:
- Make variables instantly recognizable
- Prevent ambiguity
- Enable better tooling
- Improve readability
- Are a core design principle

### Why No Garbage Collection?

RAII provides deterministic cleanup without GC pauses, making Cymple suitable for:
- Real-time systems
- Embedded systems
- Low-latency applications
- Predictable performance

### Why Share-Nothing Concurrency?

Share-nothing prevents data races by design:
- No locks needed
- No race detectors needed
- Compiler enforces safety
- Simpler mental model

### Why Quantum Operations?

Traditional parallel programming is verbose and error-prone. Quantum operations provide:
- Concise syntax (3-4x shorter than other languages)
- Built-in progress tracking
- Easy timeout handling
- Deterministic behavior
- Consistent error handling

---

## Version History

### Version 1.4 (2025-12-04)
- **NEW**: Timeout with time units (`5s`, `500ms`, `2m`, `1h`)
- **NEW**: Progress event frequency control (`every N`)
- **NEW**: Early cancellation with `🛑` in Collect
- **NEW**: Total failure event (`❌ total_failure`)
- **IMPROVED**: Simplified Race (no progress events)
- **IMPROVED**: Consistent `🛑` across quantum ops, channels, loops
- **IMPROVED**: Better error handling in Collect operations

### Version 1.3 (2025-12-02)
- String interpolation
- Else symbol `⤵️`
- Property-based length/size access
- Alternative range loop syntax
- Optional return type for void functions
- Clarified comparison (`==`) and logical operators

### Version 1.2 FINAL (2025-11-28)
- Completed specification
- Production-ready language definition

---

**End of Specification**

Version 1.4 represents a significant improvement in concurrent programming ergonomics while maintaining Cymple's core principles of safety, simplicity, and deterministic execution.

For benchmark comparisons and migration guides, see accompanying documentation.
