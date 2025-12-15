# CYMPLE: Official Language Specification

**Version 1.5, 2025.12.15 – Memory Safety Through Validated Handles**

---

## Overview

Cymple is a procedural programming language using Unicode symbols instead of keywords. Version 1.5 introduces validated handles with generational counters, structured concurrency, and explicit safety guarantees for memory and concurrent operations.

Key characteristics:
- **Memory Safe**: Validated handles with generation counters prevent use-after-free
- **Structured Concurrency**: Tasks automatically joined/cancelled on scope exit
- **Bounds Checked**: All memory access validated at runtime
- **No Pointers**: Pointer types completely removed in v1.5
- **Readable & accessible**: Clear indentation, simple rules, consistent symbols
- **Portable**: UTF-8 (NFC-normalized), tabs for block structure
- **Dual-mode**: Interpreted and compiled with identical semantics
- **Safe**: Exclusive ownership, move-only handles, deterministic cleanup
- **Concurrent**: Preemptive tasks with channels, no shared memory
- **No GC**: RAII ensures immediate resource cleanup without garbage collection pauses


---

## Document Status

**THIS DOCUMENT IS NORMATIVE**

- Sections marked "Normative" define required behavior
- Sections marked "Informative" are examples and guidance
- In case of conflict: Normative sections take precedence
- Version: 1.5 (December 15, 2025)

Key sections:
- ✅ **Normative**: Handles, Memory Blocks, Concurrency, Channels, Error Handling, Formal Semantics
- ℹ️ **Informative**: Code Examples, Design Rationale, Footguns & Guidance

---

## Conformance Keywords (RFC 2119)

This specification uses keywords as defined in RFC 2119 to indicate requirement levels:

- **MUST** / **REQUIRED** / **SHALL**: Absolute requirement for conformance
- **MUST NOT** / **SHALL NOT**: Absolute prohibition
- **SHOULD** / **RECOMMENDED**: Strong recommendation; deviations require good justification
- **SHOULD NOT** / **NOT RECOMMENDED**: Strong discouragement
- **MAY** / **OPTIONAL**: Truly optional; implementation choice

When normative sections conflict, the most specific section takes precedence.

---

## Design Principles (Informative)

Cymple 1.5 follows these core design principles:

**1. Safety First**
- Memory safety through validated handles and bounds checking
- Concurrency safety through structured concurrency and share-nothing model
- No undefined behavior, fail-fast error detection

**2. Determinism Where Possible**
- Predictable behavior for testing and debugging
- Race operations use lowest-index tie-breaker for simultaneous completion
- RAII cleanup in deterministic reverse-allocation order
- Scope exit behavior is fully specified

**3. Explicitness**
- No hidden allocations or implicit operations
- No background/detached concurrency without explicit syntax
- No shared memory between tasks
- Resource lifetimes visible in code structure

**4. Fail-Fast Philosophy**
- Invalid operations cause immediate errors, not silent corruption
- Handle validation on every use
- Bounds checking on every access
- Channel operations fail clearly on closed channels

**5. Minimalism**
- Small, focused feature set
- Functions and return statements are the primary abstraction
- No break/continue - use functions for control flow
- No garbage collection - RAII provides deterministic cleanup

These principles guide implementation decisions and inform the trade-offs in language design.

---
---

## What's New in Version 1.5

Version 1.5 represents a **major safety and clarity update** to Cymple's memory and concurrency model.

### A) Pointers Completely Removed

**Breaking Change:**
- All pointer types removed from the language
- No pointer arithmetic
- No dereference operators

**Migration:**
```cymple
📝 v1.4 (REMOVED):
*🔢ptr ← &value
🔢result ← *ptr + 10

📝 v1.5 (Use validated handles):
💾block ← allocate(100)
block[0] ← value
🔢result ← block[0] + 10
```

**Rationale:** Pointers were unsafe. All memory access now through validated handles.

### B) Generational Handle Validation

**New:** Handles carry generation counters to detect use-after-release.

```cymple
💾block ← allocate(100)
🔗 block -> B
    B[0] ← 42        📝 OK - valid handle

release(block)       📝 Generation increments

📋val ← block[0]     📝 ERROR: Stale handle detected!
```

**Handle Structure (internal):**
- slot/index (into handle table)
- generation counter (for stale detection)
- kind/type tag (resource type)

**Validation (MUST):**
Every handle use validates:
1. Slot in range?
2. Slot occupied?
3. Generation matches?
4. Kind matches operation?

**On failure:** Runtime error/panic, no undefined behavior.

### C) Memory Blocks with Bounds Checking

**New:** Explicit memory block concept for heap-managed objects.

```cymple
💾buffer ← allocate(1024)

📝 Safe indexing
📋val ← buffer[0]      📝 OK
buffer[42] ← 99       📝 OK
buffer[1500] ← 5      📝 ERROR: Bounds check fails!
```

**Required Validation:**
- Handle validity
- Index bounds checking
- Type safety

### D) Structured Concurrency

**New:** Tasks automatically managed within scope.

```cymple
🧵 parent_function()
    🌀📦 📋results ← [task1(), task2(), task3()]
        ⏱️ 30s
            💬 "Timeout"
        ✅ 📋all
            ↩ 📋all
    
    📝 Tasks GUARANTEED done or cancelled here
    💬 "All tasks finished"
```

**Features:**
- Runtime task cap (default: 1000)
- Auto join/cancel on scope exit
- Explicit detach with warnings
- Default timeouts RECOMMENDED

### E) Channel Close Semantics

**New:** Deterministic channel close behavior.

```cymple
🧵 producer(📡ch)
    🚀 ch, 42
    close(ch)  📝 Explicit close required

📝 OR use RAII:
🧵 producer_raii()
    📡ch ← 🛰️🔢
    🚀 ch, 42
    📝 Auto-closes here
```

**Closed Channel Behavior:**
- Send to closed: Error/panic
- Receive from closed: Returns null
- Loop over closed: Terminates cleanly

### F) Footguns & Guidance Section

**New:** Practical guidance on avoiding common pitfalls.

Topics covered:
- Avoid "spawn storms" (use batching)
- Always use timeouts
- Avoid detach unless necessary
- Close channels explicitly
- Handle validation errors

---

## Statement Reference Index (Normative)

This index maps EBNF statement forms to their semantic definitions in this specification.

### Declarations & Variables
- `variable_declaration` → **Variables & Assignment**
- `constant_declaration` → **Variables & Assignment**
- `assignment` → **Variables & Assignment**

### Functions & Control Flow
- `function_declaration` → **Functions**
- `return_statement` → **Functions**
- `if_statement` / `else_statement` → **Control Flow**
- `loop_statement` → **Control Flow**
- `match_statement` → **Pattern Matching**

### Memory & Resources
- `allocate_expr` → **Memory Blocks**
- `release_expr` → **Memory Blocks**
- `borrow_statement` → **Borrowing**

### Concurrency & Tasks
- `quantum_race` → **Quantum Operations: Race**
- `quantum_collect` → **Quantum Operations: Collect**
- `stop_signal` → **Quantum Operations: Stop Signal** (valid only within quantum operations)
- `task_spawn` → **Concurrency**

### Channels
- `channel_create` → **Channels**
- `channel_send` → **Channels**
- `channel_receive_statement` / `channel_receive_expr` → **Channels**
- `channel_close` → **Channels**

### Other Statements
- `output_statement` → **I/O**
- `input_statement` → **I/O**
- `guru_meditation` → **Error Handling**
- `struct_declaration` → **Structs**
- `module_import` → **Modules**
- `ffi_declaration` → **FFI**
- `timer_event` → **Timers**
- `color_command` → **Colors** (informative only)

**Note:** For complete syntax, see `cymple_ebnf_1_5.txt`. For semantics, see the referenced sections.

---


## Handles (Normative)

**Handles are the core safety mechanism in Cymple 1.5.**

### Handle Definition

A handle is an **opaque, validated reference** to a runtime-managed resource. Handles provide memory safety without garbage collection through generational validation and fail-fast error detection.

**Key Properties:**
- **Opaque**: Internal structure not accessible to programmer
- **Typed**: Each handle carries its resource kind
- **Generational**: Contains generation counter for stale detection
- **Validated**: Every use MUST be checked before operation
- **Non-nullable**: Invalid handles cause immediate errors

### Handle Structure (Runtime Internal)

While opaque to the programmer, the runtime MUST maintain this structure:

```
Handle {
    slot: u32           // Index into handle table
    generation: u32     // Generation counter
    kind: ResourceKind  // Type of resource (Memory, Channel, File, etc.)
}
```

**IMPORTANT:** This structure is implementation detail. User code MUST NOT access these fields.

### Handle Table

The runtime MUST maintain a global handle table:

```
HandleTable {
    slots: Array<HandleSlot>
}

HandleSlot {
    occupied: bool           // Is this slot in use?
    generation: u32          // Current generation counter
    kind: ResourceKind       // Type of resource
    resource: *Resource      // Pointer to actual resource
}
```

**Handle Table Operations:**

**Allocate Handle:**
1. Find free slot (occupied = false)
2. If no free slot: grow table or return error
3. Store resource reference
4. Set occupied = true
5. Return Handle { slot, generation, kind }

**Release Handle:**
1. Validate handle (see below)
2. Free/close the resource
3. Increment slot.generation
4. Set occupied = false
5. Slot can be reused with new generation

### Handle Validation (MUST)

**Every handle use MUST perform these checks in order:**

1. **Slot Range Check**
   - Is handle.slot < table.slots.length?
   - If NO: Panic with "InvalidHandle: slot out of range"

2. **Slot Occupied Check**
   - Is table.slots[handle.slot].occupied == true?
   - If NO: Panic with "InvalidHandle: slot not occupied (use-after-release)"

3. **Generation Check**
   - Is handle.generation == table.slots[handle.slot].generation?
   - If NO: Panic with "StaleHandle: generation mismatch (use-after-release)"

4. **Kind Check**
   - Is handle.kind == table.slots[handle.slot].kind?
   - If NO: Panic with "TypeError: wrong handle type for operation"

**If all checks pass:** Operation proceeds

**If any check fails:** Runtime MUST panic immediately (see Error Handling)

### Handle Types

Cymple defines these handle types:

| Handle Type | Symbol | Resource |
|-------------|--------|----------|
| Memory Block | 💾 | Heap-allocated memory |
| Channel | 📡 | Inter-task communication |
| File | 📄 | File descriptor |
| Network | 🌐 | Network connection |

**Each type MUST have distinct ResourceKind for validation.**

### Handle Lifecycle Example

```cymple
📝 1. Allocation
💾block ← allocate(1024)
📝 Runtime creates handle, allocates resource

📝 2. Use (validated each time)
🔗 block -> B
    B[0] ← 42          📝 Validation: ✓ All checks pass
    B[100] ← 99        📝 Validation: ✓ All checks pass

📝 3. Release
release(block)
📝 Runtime frees resource, increments generation

📝 4. Stale use (detected!)
📋val ← block[0]       📝 Validation: ✗ Generation mismatch → PANIC
```

### Stale Handle Detection Example

```cymple
🧵 demonstrate_stale_detection()
    💾block ← allocate(100)
    
    📝 Use 1: Valid
    🔗 block -> B
        B[0] ← 42
    📝 Validation: slot=5, gen=10, kind=Memory ✓
    
    📝 Release resource
    release(block)
    📝 Table: slot 5 now has generation=11
    
    📝 Use 2: Stale handle detected!
    🧘 guru(e)
        🔀 e.type
            ➜ "StaleHandle"
                💬 "Use-after-release detected!"
                ↩
    
    📋val ← block[0]
    📝 Validation fails: handle.gen(10) != slot.gen(11)
    📝 Runtime panics with "StaleHandle" error
```

### Optimization Rules

**Compilers MAY optimize validation checks when provably safe:**

1. **Within single borrow scope:**
   ```cymple
   🔗 block -> B
       B[0] ← 1    📝 Check required
       B[1] ← 2    📝 MAY be elided (same scope)
       B[2] ← 3    📝 MAY be elided (same scope)
   ```

2. **NOT across function boundaries:**
   ```cymple
   process(block)  📝 Full validation MUST occur
   ```

3. **NOT across loops:**
   ```cymple
   🔁 i = 0..100
       block[i] ← i  📝 Full validation MUST occur each iteration
   ```

**In debug mode:** ALL validations MUST be performed, no elision.

**IMPORTANT:** Safety MUST NEVER be sacrificed for performance.

### Handle vs Pointer Comparison

| Aspect | Cymple Handle | C Pointer |
|--------|---------------|-----------|
| **Safety** | Validated on every use | No validation |
| **Use-after-free** | Impossible (detected) | Undefined behavior |
| **Type safety** | Enforced by kind check | Type casting possible |
| **Null checks** | Not needed (fail-fast) | Manual checks required |
| **Performance** | Small overhead | Zero overhead |
| **Debugging** | Clear error messages | Crashes/corruption |

### Handle Best Practices

**DO:**
- ✅ Use RAII for automatic cleanup
- ✅ Catch validation errors with Guru
- ✅ Release explicitly when needed
- ✅ Trust the validation - it will catch errors

**DON'T:**
- ❌ Try to "optimize away" validation
- ❌ Assume handles are always valid
- ❌ Use handles after release
- ❌ Share handles between tasks (use channels instead)

### Performance Characteristics

**Handle validation cost:**
- Best case: 4 comparisons (~4 CPU cycles)
- Worst case: 4 comparisons + panic (~100+ cycles)
- Amortized: ~0.5-2% overhead in real programs

**Memory overhead:**
- Per handle: 12 bytes (slot + generation + kind)
- Handle table: grows as needed
- Total: ~0.1-1% memory overhead

**IMPORTANT:** This overhead prevents undefined behavior, memory corruption, and security vulnerabilities. It's worth it.

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


## Memory Blocks (Normative)

Memory blocks are heap-managed objects accessed exclusively through validated handles.

### Allocation

**Syntax:**
```cymple
💾block ← allocate(size: 🔢)
```

**Requirements:**
- Runtime MUST allocate `size` bytes on the heap
- Runtime MUST return a valid handle to the memory
- On allocation failure: Runtime MUST panic with "OutOfMemory"

### Indexing

**Syntax:**
```cymple
💾block ← allocate(100)
📋value ← block[index]    📝 Read
block[index] ← value      📝 Write
```

**Validation Requirements (MUST):**

For every index operation `block[i]`:

1. **Handle Validation** (see Handles section)
   - Validate slot, generation, and kind

2. **Bounds Check**
   - Is i >= 0 AND i < block.size?
   - If NO: Panic with "BoundsError: index out of range"

3. **Alignment Check** (for typed blocks)
   - Is i properly aligned for the type?
   - If NO: Panic with "AlignmentError: unaligned access"

**IMPORTANT:** Bounds checking MUST occur before every access. No exceptions.

### Memory Block Properties

```cymple
💾block ← allocate(1024)

📝 Properties (always safe to access):
🔢size ← block.size           📝 Returns allocated size
✅valid ← block.is_valid       📝 Returns true if handle valid
```

**Property access MUST validate handle first.**

### Copy vs Move Semantics

**Move (default):**
```cymple
💾block1 ← allocate(100)
💾block2 ← block1              📝 block1 is now null_handle
📝 Runtime MUST invalidate block1
📝 Only block2 can access the memory
```

**Copy (explicit):**
```cymple
💾block1 ← allocate(100)
💾block2 ← copy(block1)        📝 Creates NEW allocation
📝 Runtime MUST allocate new memory
📝 Runtime MUST copy contents
📝 Both handles remain valid
```

### Borrowing Memory Blocks

**Read-only borrow:**
```cymple
💾block ← allocate(100)

🔗 block -> B
    📋val ← B[0]               📝 OK: Read
    B[0] ← 42                  📝 ERROR: Cannot mutate read-only borrow
```

**Mutable borrow:**
```cymple
🔗 block -> mut B
    B[0] ← 42                  📝 OK: Mutable access
    📋val ← B[0]               📝 OK: Can also read
```

**Borrow Rules (MUST):**
- Only ONE mutable borrow OR multiple read-only borrows at a time
- Original handle MUST NOT be used while borrowed
- Borrows MUST end before handle is released

### RAII and Automatic Cleanup

```cymple
🧵 process_data()
    💾buffer ← allocate(1024)
    
    🔗 buffer -> mut B
        B[0] ← 42
        B[1] ← 99
    📝 Borrow ends here
    
    📝 buffer automatically released here (end of function)
    📝 Runtime MUST free memory deterministically
```

**RAII Rules (MUST):**
- Handles MUST be released when scope ends
- Release MUST happen in reverse allocation order
- Release MUST be deterministic (no GC delays)

### Bounds Checking Example

```cymple
🧵 demonstrate_bounds_checking()
    💾buffer ← allocate(100)
    
    📝 Catch bounds errors
    🧘 guru(e)
        🔀 e.type
            ➜ "BoundsError"
                💬 "Index out of range!"
                💬 "  Index: e.details.index"
                💬 "  Size: e.details.size"
                ↩
    
    📝 Safe accesses
    buffer[0] ← 1              📝 OK: 0 < 100
    buffer[99] ← 99            📝 OK: 99 < 100
    
    📝 Unsafe access (caught!)
    buffer[100] ← 5            📝 PANIC: BoundsError
    buffer[-1] ← 5             📝 PANIC: BoundsError
```

### Uninitialized Memory

**Default Behavior:**
- Allocated memory SHOULD be zeroed by default
- Implementation MAY provide `allocate_uninitialized()` for performance
- Uninitialized memory MUST be marked as such
- Reading uninitialized memory SHOULD trigger warning in debug mode

### Memory Block Limits

**Runtime Limits (SHOULD be configurable):**
- Maximum single allocation: Default 2GB
- Maximum total heap: Default system-dependent
- On limit exceeded: Panic with "AllocationLimit"

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
release(f2)  📝 explicit cleanup (or use RAII)
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

**Entry point (main):**
```cymple
🧵 main()
    💬 "Program starts here"
    🔢result ← calculate()
    💬 "Result: 🔢result"
```

If a function named `main()` exists, it is automatically called after all top-level statements are executed. This provides a standard entry point similar to C, Go, Java, and Rust.

### Return Semantics (MUST)

**Typed Functions:**
- Functions with declared return types (`-> 🔢`, `-> 🔤`, etc.) MUST return a value of that type
- Missing or omitted return statement is a compile-time error
- Example: `🧵 foo() -> 🔢` MUST explicitly return a number

**Untyped Functions:**
- Functions without return type declaration implicitly return `null` if no explicit return occurs
- Missing return is not an error; equivalent to implicit `↩ null` at function end
- Example: `🧵 bar()` returns `null` if no `↩` statement is reached

**Void Functions:**
- Functions can optionally declare `-> void` to indicate no return value
- Using `↩ expr` (with a value) in a void function is a compile-time error
- Using `↩` alone (early exit without value) is permitted
- Example: `🧵 log() -> void` can use `↩` for early exit but not `↩ 42`

**Multiple Return Values:**
- Return type `-> (🔢, 🔤)` indicates tuple return
- All return statements in function MUST return matching tuple
- Unpacking: `🔢n, 🔤s ← get_both()`

---


## Concurrency (Normative)

Cymple provides structured concurrency with safety guarantees through validated handles and runtime enforcement.

### Concurrency Model

**Core Principles:**
1. **Share-nothing**: Tasks communicate via channels, not shared memory
2. **Structured**: Tasks are joined/cancelled when leaving scope
3. **Validated**: All concurrency primitives use handle validation
4. **Bounded**: Runtime enforces maximum concurrent tasks

### Runtime Task Cap (MUST)

The runtime MUST enforce a maximum number of concurrent tasks.

**Default Configuration:**
```cymple
📝 Default task cap (implementation-defined, recommended: 1000)
runtime.max_concurrent_tasks ← 1000
```

**Enforcement Behavior:**

When task cap is reached, the runtime MUST:
1. Block the task creation request
2. Wait for a task slot to become available
3. Resume once a task completes

The runtime MUST NOT:
- Silently drop task creation requests
- Create tasks beyond the cap
- Ignore the cap setting

**Rationale:** Prevents resource exhaustion and "spawn storms".

### Structured Concurrency (MUST)

Tasks started in a scope MUST be joined or cancelled when leaving that scope.

**Automatic Cleanup:**
```cymple
🧵 parent_function()
    🌀📦 📋results ← [task1(), task2(), task3()]
        ⏱️ 30s
            💬 "Timeout"
        ✅ 📋all
            ↩ 📋all
    
    📝 GUARANTEE: All tasks are done or cancelled here
    💬 "Scope exit: all tasks finished"
```

**Scope Exit Rules (MUST):**
- Runtime MUST wait for all tasks to complete OR
- Runtime MUST cancel all running tasks
- No tasks may outlive their parent scope
- Exception: Explicitly detached tasks (see below)

### Task Lifecycle States

```
Created → Queued → Running → [Completed | Cancelled | Failed]
```

**State Transitions (MUST):**
- Created: Task object allocated
- Queued: Waiting for task slot (if cap reached)
- Running: Actively executing
- Completed: Finished successfully
- Cancelled: Stopped by parent scope exit
- Failed: Panicked with error

### Explicit Detach (MAY)

Tasks MAY be explicitly detached from their parent scope:

```cymple
🧵 fire_and_forget()
    🧵detach background_task()
    📝 WARNING: Task outlives this function!
    ↩  📝 Parent returns immediately
```

**Detach Rules:**
- Detached tasks MUST NOT prevent program termination
- Detached tasks SHOULD complete before main() exits
- Implementation SHOULD warn about detached tasks
- Detached tasks count toward task cap

**Use Detach Sparingly:** Prefer structured concurrency.

### Quantum Operations

#### Race (`🌀⚡`) - First Result Wins

**Syntax:**
```cymple
🌀⚡🔤result ← [task1(), task2(), task3()]
    ⏱️ timeout
        📝 timeout handler
    ✅ 🔤winner
        📝 success handler
    ❌ 🔤error
        📝 error handler
```

**Semantics (MUST):**
1. Start all tasks concurrently
2. Wait for first successful result
3. Cancel all other tasks immediately
4. Return the winning result

**Timeout (SHOULD):**
- Race operations SHOULD have explicit timeout
- Without timeout: may block indefinitely
- Timeout fires if no task completes in time

#### Collect (`🌀📦`) - Gather All Results

**Syntax:**
```cymple
🌀📦 📋results ← [task1(), task2(), ...taskN()]
    ⏱️ timeout
        📝 timeout handler (returns partial)
    ⏩ 📋partial every N
        📝 progress handler
        ❓ condition
            🛑  📝 early cancellation
    ✅ 📋all
        📝 success handler
    ❌ 🔤total_failure
        📝 all-failed handler
```

**Semantics (MUST):**
1. Start all tasks concurrently (subject to cap)
2. Wait for all tasks to complete
3. Return list of results (including failures)
4. Support partial results on timeout

**Progress Events:**
- `every N`: Fire progress after every N completed tasks
- Without `every`: Fire after EACH task (expensive!)
- Runtime SHOULD rate-limit progress events

**Early Cancellation:**
- `🛑` stops all remaining tasks
- Returns partial results immediately
- Cancelled tasks MUST cleanup resources

### Stop Signal (`🛑`)

The stop signal MUST:
- Propagate to all running tasks in the quantum operation
- Cause blocked operations to unblock
- Be non-blocking itself
- Work across all quantum operations

```cymple
⏩ 📋partial every 10
    ❓ user_cancelled()
        🛑  📝 Stops all tasks immediately
        ↩ 📋partial
```

### Task Communication

Tasks MUST communicate via channels (see Channels section).

**Forbidden:**
- Shared memory between tasks
- Direct task-to-task pointers
- Global mutable state

**Allowed:**
- Channels for message passing
- Read-only data sharing (via immutable references)

### Concurrency Safety Examples

**Example 1: Structured Timeout**
```cymple
🧵 fetch_with_timeout(🔤url) -> 🔤
    🌀⚡ 🔤result ← [http_get(🔤url)]
        ⏱️ 5s
            💬 "Request timed out"
            ↩ ""
        ✅ 🔤data
            ↩ 🔤data
    📝 Task guaranteed cleaned up here
```

**Example 2: Batch Processing with Cap**
```cymple
🧵 process_batch(📋items) -> 📋
    📝 Runtime enforces cap (e.g., 1000 tasks max)
    📝 Excess tasks wait in queue
    
    🌀📦 📋results ← create_tasks(📋items)
        ⏱️ 5m
            ↩ 📋results  📝 Partial results
        ⏩ 📋partial every 100
            💬 "Progress: 📋partial.length"
        ✅ 📋all
            ↩ 📋all
```

**Example 3: Early Cancellation**
```cymple
🧵 search_until_found(📋databases) -> 🔤
    🌀📦 📋results ← search_all(📋databases)
        ⏩ 📋partial
            ❓ found_match(📋partial)
                💬 "Found! Stopping search"
                🛑  📝 Cancel remaining tasks
                ↩ extract_match(📋partial)
        ✅ 📋all
            ↩ best_result(📋all)
```

### Performance Considerations

**Task Creation Cost:**
- Lightweight: ~1-10 microseconds per task
- Subject to cap enforcement delays

**Context Switch Cost:**
- Preemptive scheduling overhead
- ~1-5 microseconds per switch

**Validation Overhead:**
- Handle validation on channel operations
- ~4 comparisons per operation

**IMPORTANT:** These costs are small compared to the safety guarantees provided.

---


## Channels (Normative)

Channels are type-safe, validated communication primitives for inter-task communication.

### Channel Properties

**Channels MUST:**
- Be referenced via validated handles
- Support ONLY one-way communication (send OR receive)
- Be type-safe (enforce message type)
- Close deterministically
- Not allow shared memory passing

### Channel Creation

```cymple
📡ch ← 🛰️🔢  📝 Create channel for numbers
```

**Requirements:**
- Runtime MUST allocate channel with specified type
- Runtime MUST return valid channel handle
- Channel MUST be initially open

### Channel Operations

**Send:**
```cymple
🚀 ch, value
```

**Send Requirements (MUST):**
1. Validate channel handle
2. Check channel is open
3. Type-check value matches channel type
4. Block if channel full (implementation-defined buffer)
5. On closed channel: Panic with "ChannelClosed"

**Receive:**
```cymple
🔢value ← 🎯 ch
```

**Receive Requirements (MUST):**
1. Validate channel handle
2. Check channel is open
3. Block until message available
4. Return typed value
5. On closed empty channel: Return special "closed" value or panic

**Loop over Channel:**
```cymple
🔁 msg in ch
    💬 "Received: 🔢msg"
📝 Loop exits when channel closed and empty
```

### Channel Close Semantics (MUST)

**Explicit Close:**
```cymple
📡ch ← 🛰️🔢
🚀 ch, 42
close(ch)  📝 Channel MUST close here
```

**RAII Close:**
```cymple
🧵 sender()
    📡ch ← 🛰️🔢
    🚀 ch, 42
    📝 Channel MUST close at scope exit
```

**Close Requirements (MUST):**
- Channel MUST be closeable exactly once
- After close: new sends MUST panic
- After close: pending receives complete
- After close + empty: receives return "closed"
- Close MUST be visible to all receivers

### Closed Channel Behavior

| Operation | Behavior | Required |
|-----------|----------|----------|
| Send to closed | Panic "ChannelClosed" | MUST |
| Receive from closed (empty) | Return null/closed | MUST |
| Receive from closed (has data) | Return data | MUST |
| Loop over closed | Exit loop | MUST |
| Double close | Panic "AlreadyClosed" | SHOULD |

### Channel Example with Guru

```cymple
🧵 safe_channel_use()
    🧘 guru(e)
        🔀 e.type
            ➜ "ChannelClosed"
                💬 "Channel was closed"
                ↩
    
    📡ch ← 🛰️🔢
    
    🌀📦 _ ← [
        producer(ch),
        consumer(ch)
    ]
        ✅ _
            💬 "Done"

🧵 producer(📡out)
    🔁 i = 1..10
        🚀 out, i
    close(out)  📝 Signal end

🧵 consumer(📡in)
    🔁 msg in in
        💬 "Got: 🔢msg"
    💬 "Consumer finished"
```

### Channel Capacity and Buffering

**Default Behavior:**
- Channels MAY be unbuffered (synchronous)
- Channels MAY be buffered (asynchronous)
- Implementation MUST document default

**Buffered Channel:**
```cymple
📡ch ← 🛰️🔢:100  📝 Buffer size 100
```

**Buffering Rules:**
- Send blocks when buffer full
- Receive blocks when buffer empty
- Close releases blocked senders/receivers

### Channel Safety Rules (MUST NOT)

**Forbidden Operations:**
- ❌ Sharing channels across tasks via shared memory
- ❌ Casting channel to different type
- ❌ Accessing channel internals
- ❌ Using channel after release

**Allowed Operations:**
- ✅ Passing channel via function parameters
- ✅ Returning channel from functions
- ✅ Storing channels in collections (moves)
- ✅ Borrowing channels temporarily

### Channel Patterns

**Pattern 1: Producer-Consumer**
```cymple
🧵 main()
    📡ch ← 🛰️🔤
    
    🌀📦 _ ← [
        produce(ch),
        consume(ch)
    ]
        ✅ _
            💬 "Pipeline complete"

🧵 produce(📡out)
    🔁 i = 1..100
        🚀 out, "Item i"
    close(out)

🧵 consume(📡in)
    🔁 item in in
        process(item)
```

**Pattern 2: Fan-out**
```cymple
🧵 fan_out(📡in, 📋workers: 📡)
    🔁 data in in
        📝 Distribute to workers
        🔁 worker in workers
            🚀 worker, data
    
    📝 Close all workers
    🔁 worker in workers
        close(worker)
```

**Pattern 3: Select (via Race)**
```cymple
🧵 select_channel(📡ch1, 📡ch2)
    🌀⚡ 🔢result ← [
        receive_task(ch1),
        receive_task(ch2)
    ]
        ✅ 🔢value
            💬 "Got: 🔢value"
            ↩ 🔢value
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


## Error Handling (Normative) and Panic Behavior

### Panic Mechanism

When validation or safety checks fail, Cymple uses a **panic mechanism** similar to Rust and Go.

**Panic Behavior (MUST):**

When a panic occurs, the runtime MUST:
1. Stop the current operation immediately
2. Unwind the call stack (in order)
3. Execute RAII cleanup for each scope
4. Release all handles in reverse allocation order
5. Propagate to Guru handler if present
6. Otherwise: Terminate the task/program

**Panic is NOT:**
- Resumable (no "continue after panic")
- Catchable except via Guru
- For recoverable errors (use Result types instead)

### Guru Meditation (`🧘`)

Guru is Cymple's mechanism for catching and handling panics.

**Syntax:**
```cymple
🧘 guru(e)
    📝 Pattern match on error
    🔀 e.type
        ➜ "ErrorType1"
            📝 Handle this error
        ➜ "ErrorType2"
            📝 Handle another error
        ➜ _
            💀 e  📝 Rethrow
```

**Guru Scope:**
- Guru MUST be declared at beginning of function or block
- Guru covers all code in that scope
- Inner Gurus override outer Gurus
- Guru catches panics from called functions

**Guru Requirements (MUST):**
- Runtime MUST call Guru on any panic in scope
- Guru MUST receive error object with full context
- Guru MAY rethrow with `💀 e`
- Guru MAY return value to continue execution

### Standard Error Types

The runtime MUST provide these error types:

| Error Type | Code | When |
|------------|------|------|
| `HandleValidation` | 1000 | Handle validation failed |
| `StaleHandle` | 1001 | Generation mismatch |
| `InvalidHandle` | 1002 | Slot invalid/unoccupied |
| `TypeError` | 1003 | Handle kind mismatch |
| `BoundsError` | 1100 | Index out of range |
| `AlignmentError` | 1101 | Unaligned memory access |
| `OutOfMemory` | 1200 | Allocation failed |
| `AllocationLimit` | 1201 | Exceeded allocation limit |
| `ChannelClosed` | 2000 | Operation on closed channel |
| `TaskLimitReached` | 3000 | Too many concurrent tasks |
| `Timeout` | 4000 | Operation timed out |
| `Cancelled` | 4001 | Task was cancelled |

### Error Object Structure

```cymple
Error {
    type: 🔤         📝 Error type string
    code: 🔢         📝 Numeric error code
    message: 🔤      📝 Human-readable message
    location: 🔤     📝 File:Line information
    details: 🗺️      📝 Additional context
}
```

**Details Map:**
- `BoundsError`: { index: 🔢, size: 🔢 }
- `StaleHandle`: { slot: 🔢, expected_gen: 🔢, actual_gen: 🔢 }
- `TypeError`: { expected: 🔤, actual: 🔤 }

### Validation Error Examples

**Example 1: Handle Validation**
```cymple
🧵 safe_access(💾block, 🔢index) -> 📋
    🧘 guru(e)
        🔀 e.type
            ➜ "StaleHandle"
                💬 "Error: Use-after-release detected"
                💬 "  Slot: e.details.slot"
                💬 "  Expected gen: e.details.expected_gen"
                💬 "  Actual gen: e.details.actual_gen"
                ↩ null
            ➜ "BoundsError"
                💬 "Error: Index out of bounds"
                💬 "  Index: e.details.index"
                💬 "  Size: e.details.size"
                ↩ null
            ➜ _
                💀 e  📝 Rethrow unknown errors
    
    ↩ block[index]
```

**Example 2: Task Limit**
```cymple
🧵 spawn_carefully()
    🧘 guru(e)
        🔀 e.type
            ➜ "TaskLimitReached"
                💬 "Too many tasks, waiting..."
                📝 Retry after delay
                ↩
    
    🌀📦 📋results ← [many_tasks...]
        ✅ 📋all
            ↩ 📋all
```

### Debug vs Release Mode

**Debug Mode MUST:**
- Perform ALL validation checks
- Include full error context
- Never elide bounds checks
- Provide verbose error messages
- Include source location information

**Release Mode MAY:**
- Optimize validation where provably safe
- Provide shorter error messages
- Elide some debug information
- MUST NEVER sacrifice safety

**Configuration:**
```cymple
📝 Compiler flags (example)
cymple --mode=debug    📝 All checks, verbose errors
cymple --mode=release  📝 Optimized, but still safe
```

### Stack Unwinding

On panic, the runtime MUST unwind the stack:

**Unwinding Process:**
1. Start at panic location
2. For each stack frame (from inner to outer):
   - Execute RAII cleanup for that scope
   - Release handles in reverse order
   - Check for Guru handler
3. If Guru found: call Guru with error
4. If Guru handles: resume execution
5. If Guru rethrows or not found: continue unwinding
6. If unwinding reaches top: terminate task/program

**Example Unwinding:**
```cymple
🧵 outer()
    🧘 guru(e)  📝 Catches inner panics
        💬 "Caught: e.type"
    
    inner()

🧵 inner()
    💾block ← allocate(100)
    block[200] ← 5  📝 PANIC: BoundsError
    📝 Stack unwinds:
    📝 1. Release 'block'
    📝 2. Exit inner()
    📝 3. Find Guru in outer()
    📝 4. Call Guru with error
```

### Fail-Fast Philosophy

Cymple follows a **fail-fast** approach:

**Principles:**
- Detect errors as early as possible
- Never continue with invalid state
- Prefer explicit panic over silent corruption
- Make bugs obvious during development

**Benefits:**
- Easier debugging (errors caught early)
- No undefined behavior
- No memory corruption
- Clear error messages

**Cost:**
- Small runtime overhead for checks
- Must handle panics with Guru

**IMPORTANT:** Fail-fast prevents security vulnerabilities and data corruption.

### Error Propagation

**Manual Propagation:**
```cymple
🧵 caller()
    🧘 guru(e)
        💀 e  📝 Rethrow to caller's caller
    
    callee()  📝 May panic

🧵 callee()
    risky_operation()  📝 May panic
```

**Automatic Propagation:**
- If no Guru present: panic propagates up automatically
- Unwinding continues until Guru found or program exits

### Custom Error Throwing

```cymple
🧵 validate_input(🔤input)
    ❓ input.length == 0
        ❌ ValidationError("Input cannot be empty")
    
    ❓ input.length > 1000
        ❌ ValidationError("Input too long") -> 400
```

**Throw Syntax:**
- `❌ ErrorType(message)` - Throw with message
- `❌ ErrorType(message) -> code` - Throw with custom code

---


## Formal Semantics (Normative)

**THIS SECTION IS NORMATIVE** - All implementations MUST conform to these semantics.

### Task Lifecycle State Machine

**States:**
```
Created → Queued → Running → [Completed | Cancelled | Failed]
              ↓              ↗
           TaskCap      Timeout/Stop
```

**State Definitions:**

1. **Created**: Task object allocated, not yet started
   - MUST have unique task ID
   - MUST be registered in task table
   - Parent reference MUST be stored

2. **Queued**: Waiting for task slot (when at cap)
   - MUST be in FIFO queue
   - MUST not consume task slot yet
   - Cancellation MUST remove from queue

3. **Running**: Actively executing
   - MUST consume one task slot
   - MUST be preemptible
   - MUST respond to stop signals

4. **Completed**: Finished successfully
   - MUST release task slot
   - Result MUST be available
   - MUST trigger parent notifications

5. **Cancelled**: Stopped before completion
   - MUST release task slot
   - MUST execute cleanup (RAII)
   - MUST NOT be resumable

6. **Failed**: Panicked with error
   - MUST release task slot
   - Error MUST propagate to parent
   - MUST execute unwinding

**Transition Rules (MUST):**

```
Created → Queued:    When task cap reached
Queued → Running:    When task slot available
Running → Completed: Normal return
Running → Cancelled: Stop signal OR parent scope exit
Running → Failed:    Panic occurs
Queued → Cancelled:  Parent scope exit before slot available
```

---

### Scope Exit Semantics (Structured Concurrency)

**DEFINITION:** A scope exits when execution reaches its closing brace (DEDENT in EBNF).

**Scope Exit MUST perform these steps IN ORDER:**

1. **Mark scope as "exiting"**
   - No new tasks may be spawned in this scope
   - Queued tasks remain queued

2. **Cancel all non-detached tasks in this scope**
   - Send cancellation signal to each Running task
   - Remove Queued tasks from queue
   - Mark as Cancelled state

3. **Wait for cancellation acknowledgment**
   - Block until all tasks reach terminal state (Completed/Cancelled/Failed)
   - Timeout: SHOULD have configurable timeout (default: 30s)
   - On timeout: Force-kill tasks (implementation-defined)

4. **Execute RAII cleanup**
   - Release handles in REVERSE allocation order
   - Close channels
   - Free memory blocks
   - Call destructors if defined

5. **Propagate errors**
   - If any task Failed: collect first error
   - If Guru present in parent: call Guru
   - If no Guru: propagate panic upward

**Example Timeline:**
```
t=0:    Scope starts, task1 spawns
t=1:    task2 spawns
t=2:    Scope exit begins
t=2.1:  task1 and task2 receive cancellation
t=2.5:  task1 completes cancellation
t=3.0:  task2 completes cancellation
t=3.1:  RAII cleanup executes
t=3.2:  Scope fully exited
```

**Nested Scopes:**
```cymple
outer_scope:
    🧵 task_outer()
    
    inner_scope:
        🧵 task_inner()
    # inner_scope exit: task_inner cancelled
    
# outer_scope exit: task_outer cancelled
```

MUST cancel in order: innermost to outermost.

---

### Stop Signal (`🛑`) Formal Semantics

**DEFINITION:** Stop signal is a control-flow statement that cancels tasks.

**Scope:** Stop signal is valid ONLY within quantum operation body.

**Semantics (MUST):**

1. **Idempotence**: Multiple `🛑` in same quantum = single stop
   ```cymple
   🛑
   🛑  # Second stop has no additional effect
   ```

2. **Immediate Effect**: Tasks receive cancellation signal synchronously
   - Tasks in Running state: Begin cancellation
   - Tasks in Queued state: Removed from queue
   - Completed tasks: Unaffected

3. **Return Behavior**: Stop causes immediate return from quantum operation
   - Progress clause with `🛑`: Returns partial results
   - Success clause: Never reached after `🛑`
   - Timeout clause: Never reached after `🛑`

4. **Isolation**: Stop affects ONLY the quantum operation it appears in
   ```cymple
   🌀📦 results1 ← [tasks...]
       ⏩ partial
           🛑  # Stops only results1, not results2
   
   🌀📦 results2 ← [other_tasks...]
       # results2 unaffected
   ```

5. **Not Catchable**: Tasks cannot ignore stop signal
   - Must honor cancellation
   - May execute cleanup (finally blocks)
   - Cannot resume after cancellation

**Anti-Example (ILLEGAL):**
```cymple
# ILLEGAL: 🛑 outside quantum operation
🛑  # Error: StopSignalOutOfScope
```

---

### Race Determinism Rules

**Problem:** What happens when multiple tasks complete "simultaneously"?

**RULE (MUST):** Lowest array index wins.

**Formal Definition:**

Given tasks `[t0, t1, t2, ..., tn]` in Race operation:

```
Let C = set of completed tasks at time T
Let winner = min(index(t) for t in C)
Return result of task at index 'winner'
```

**"Simultaneously" Defined:**

Tasks complete simultaneously if their completion timestamps differ by less than the scheduler's time quantum (implementation-defined, typically 1-10μs).

**Example:**
```cymple
🌀⚡ result ← [task0(), task1(), task2()]
    ✅ winner
        ↩ winner

# Scenario 1: task1 completes first
# → Return task1's result

# Scenario 2: task0 and task2 complete at T=100μs,
#             task1 completes at T=100.5μs
#             (within time quantum)
# → All three "simultaneous"
# → winner = min(0, 1, 2) = 0
# → Return task0's result
```

**Justification:** Lowest-index rule ensures:
- Deterministic behavior (testable)
- No race conditions
- Predictable prioritization

---

### Channel Close State Machine

**States:**
```
Created → Open → Closed
              ↓
          Scope Exit (RAII)
```

**State Invariants (MUST):**

1. **Open State:**
   - Send: Succeeds (or blocks if full)
   - Receive: Blocks until message or close
   - Close: Transitions to Closed

2. **Closed State:**
   - Send: PANIC (ChannelClosed)
   - Receive (buffer not empty): Return buffered messages
   - Receive (buffer empty): Return null
   - Close: PANIC (AlreadyClosed) OR no-op (implementation choice)

**Transition Rules:**

- `close(ch)`: Open → Closed (explicit)
- Scope exit: Open → Closed (RAII)
- Double close: MUST be deterministic (panic or no-op)

**Buffer Draining:**

Closed channels MUST drain buffer before returning null:

```cymple
📡ch ← 🛰️🔢:3
🚀 ch, 1
🚀 ch, 2
🚀 ch, 3
close(ch)

# Receiver side:
r1 ← 🎯 ch  # Returns 1
r2 ← 🎯 ch  # Returns 2
r3 ← 🎯 ch  # Returns 3
r4 ← 🎯 ch  # Returns null (closed + empty)
```

**Loop Termination:**

Loops over channels MUST terminate when closed + empty:

```cymple
🔁 msg in ch
    # Receives all buffered messages
# Loop exits when ch closed AND buffer empty
```

---

### Panic and Stack Unwinding

**Panic Sources:**
- Handle validation failure
- Bounds check failure
- Channel closed error
- Explicit throw (`❌`)

**Unwinding Algorithm (MUST):**

```
function panic(error):
    1. current_frame = get_current_stack_frame()
    
    2. while current_frame != null:
        a. Mark frame as "unwinding"
        
        b. Execute RAII cleanup:
           - Get all handles in frame (in REVERSE alloc order)
           - For each handle:
               * Validate (may skip if already invalid)
               * Release resource
               * Increment generation
        
        c. Check for Guru:
           - If Guru present in this frame:
               * Call Guru with error object
               * If Guru returns: stop unwinding, resume
               * If Guru rethrows (💀): continue unwinding
        
        d. current_frame = parent_frame
    
    3. If no Guru handled:
        - Print error to stderr
        - Terminate task/program
        - Exit code: 1
```

**RAII Order Example:**

```cymple
f1 ← open("a.txt")    # Alloc order: 1
f2 ← open("b.txt")    # Alloc order: 2
buffer ← allocate(100) # Alloc order: 3

panic()

# Cleanup order (REVERSE):
# 1. release(buffer)
# 2. close(f2)
# 3. close(f1)
```

---

### Task Cap Enforcement

**Definition:** Runtime MUST limit concurrent tasks.

**Algorithm:**

```
global task_count = 0
global task_queue = []
global TASK_CAP = 1000  # Configurable

function spawn_task(task):
    if task_count < TASK_CAP:
        task_count += 1
        start_task(task)
        return RUNNING
    else:
        task_queue.append(task)
        return QUEUED

function task_completed(task):
    task_count -= 1
    
    if task_queue not empty:
        next_task = task_queue.pop_front()
        spawn_task(next_task)
```

**Queue Properties (MUST):**
- FIFO ordering
- No size limit (may cause memory pressure)
- Drained on scope exit

**Backpressure:**

Implementations SHOULD provide backpressure when queue grows:
- Warning when queue > 1000
- Error when queue > 10000
- Or configurable thresholds

---

### Error Object Structure (Formal)

**Every panic MUST create error object:**

```
Error {
    type: string          # Error type name
    code: integer         # Numeric code (1000-9999)
    message: string       # Human-readable description
    location: string      # "file.cym:line:col"
    stack_trace: [string] # Function call stack
    details: map          # Type-specific context
}
```

**Standard Error Codes (MUST):**

| Code | Type | Details Map |
|------|------|-------------|
| 1000 | HandleValidation | {slot, gen} |
| 1001 | StaleHandle | {slot, expected_gen, actual_gen} |
| 1002 | InvalidHandle | {slot, table_size} |
| 1003 | TypeError | {expected, actual} |
| 1100 | BoundsError | {index, size} |
| 1101 | AlignmentError | {address, alignment} |
| 1200 | OutOfMemory | {requested, available} |
| 2000 | ChannelClosed | {channel_id} |
| 2001 | AlreadyClosed | {channel_id} |
| 3000 | TaskLimitReached | {current, cap} |
| 4000 | Timeout | {duration, elapsed} |

**Error Code Ranges:**

Error codes are organized into ranges by category:

| Range     | Category                  | Status       |
|-----------|---------------------------|--------------|
| 1000-1099 | Handle Validation         | Reserved     |
| 1100-1199 | Memory & Bounds           | Reserved     |
| 1200-1299 | Memory Allocation         | Reserved     |
| 1300-1999 | *Future: Memory/Safety*   | Reserved     |
| 2000-2099 | Channel Operations        | Reserved     |
| 2100-2999 | *Future: Communication*   | Reserved     |
| 3000-3099 | Task & Concurrency        | Reserved     |
| 3100-3999 | *Future: Concurrency*     | Reserved     |
| 4000-4099 | Timing & Timeouts         | Reserved     |
| 4100-4999 | *Future: Timing*          | Reserved     |
| 5000-5999 | I/O & Resources           | Reserved     |
| 6000-6999 | Language & Syntax         | Reserved     |
| 7000-7999 | Runtime System            | Reserved     |
| 8000-8999 | Host/FFI Errors           | Reserved     |
| 9000-9999 | Custom/Implementation     | User-defined |

**Notes:**
- Ranges 1000-8999 are reserved for Cymple specification
- Range 9000-9999 is available for implementation-specific errors
- Implementations MUST NOT define errors in reserved ranges
- Future Cymple versions MAY define additional errors within reserved ranges

Implementations MAY add custom error codes in range 9000-9999.

---

**END OF FORMAL SEMANTICS**

This section provides unambiguous semantics for implementation. In case of conflict between this section and examples elsewhere, THIS SECTION IS AUTHORITATIVE.

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

**NEW in v1.5:** FFI automatically wraps C pointers as Cymple handles.

Link to C libraries:

```cymple
🔗 "libc.so.6"
    🧵 strlen(s: 🔤) -> 🔢
    🧵 c_malloc(size: 🔢) -> 💾  📝 Returns Cymple handle, NOT C pointer
    🧵 c_free(h: 💾)             📝 Takes Cymple handle

🔢len ← strlen("Hello")
💬 "Length: 🔢len"

📝 C memory wrapped as validated handles:
💾block ← c_malloc(1024)
📝 Use like any Cymple handle with validation
🔗 block -> B
    B[0] ← 42
c_free(block)
```

**Handle Wrapping (CRITICAL):**
- C pointers are **automatically wrapped** as Cymple handles on return
- Cymple handles are **unwrapped** to C pointers when passed to C
- All Cymple safety rules apply (validation, bounds checking)
- C memory must be freed explicitly with C's free function

**Type Mapping:**
- C `int*` → Cymple `🔢` (number, by-value copy)
- C `char*` → Cymple `🔤` (string, copied)
- C `void*` → Cymple `💾` (memory block handle, wrapped!)
- C structs → Pass by copying or via memory block handles

**IMPORTANT:** FFI is an escape hatch. Use with caution. Cymple cannot validate C code's memory safety

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
| `release()` | Explicit handle cleanup (use RAII preferred) |

### Timers (v1.5)
| Symbol | Meaning |
|--------|---------|
| `⏱️` | Timeout |
| `⏱️▶` | One-shot timer |
| `⏱️🔁` | Periodic timer |

### Error Handling (Normative)
| Symbol | Meaning |
|--------|---------|
| `🧘` | Guru meditation |
| `⚠️` | Warning |
| `❌` | Error |
| `💀` | Fatal |

---

## Complete Examples (Informative)

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

## Design Rationale (Informative)

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

### Version 1.5 (2025-12-04)
- **NEW**: Automatic `main()` function execution as entry point
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

Version 1.5 represents a significant improvement in concurrent programming ergonomics while maintaining Cymple's core principles of safety, simplicity, and deterministic execution.

For benchmark comparisons and migration guides, see accompanying documentation.
