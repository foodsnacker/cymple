# CYMPLE 1.5 Changelog

**Version:** 1.5  
**Release Date:** December 15, 2025  
**Focus:** Memory safety, concurrency clarity, and footgun prevention

---


## Breaking Changes Summary

**🔴 CRITICAL BREAKING CHANGES:**

| Change | Impact | Migration Effort |
|--------|--------|------------------|
| Pointers removed | HIGH - All pointer code invalid | Rewrite with handles |
| EBNF updated to 1.5 | MEDIUM - Syntax unchanged, semantics formal | Update parsers |
| Handle validation mandatory | LOW - Was implicit, now explicit | Add error handlers |
| Task cap enforced | MEDIUM - May block on spawn storms | Add backpressure |
| Channel close mandatory | LOW - Was implicit RAII, still works | Make explicit where needed |

**⚠️ SEMANTIC CHANGES:**

| What Changed | v1.4 Behavior | v1.5 Behavior |
|--------------|---------------|---------------|
| Scope exit | Unspecified task cleanup | MUST join/cancel (formal) |
| Race simultaneous | Undefined | Lowest index wins (formal) |
| Stop signal | Implied behavior | Formally specified |
| Channel close | Implicit | Explicit or RAII (both valid) |
| Panic unwinding | Unspecified order | RAII reverse order (formal) |

---
## Overview

Version 1.5 is a **major safety and clarity update** to Cymple's memory and concurrency model. This release removes unsafe features (pointers), adds precise validation semantics (generational handles), improves concurrency safety (structured concurrency), and provides clear guidance on avoiding common pitfalls.

---


## Formal Specification (NEW in 1.5)

**1.5 is the first version with complete formal specification.**

### What's Formal?

**EBNF Grammar:**
- All syntax formally defined
- 80 productions
- 17 semantic notes
- 25 MUST requirements in grammar comments

**Formal Semantics Section:**
- Task lifecycle state machine
- Scope exit algorithm
- Stop signal formal definition
- Race determinism rules
- Channel state machine
- Panic/unwinding algorithm
- Task cap enforcement
- Error object structure

### Normative vs Informative

**Normative (MUST follow):**
- Handles section
- Memory Blocks section
- Concurrency section
- Channels section
- Error Handling section
- Formal Semantics section
- EBNF Grammar

**Informative (Examples only):**
- Code examples
- Design rationale
- Footguns & guidance

**In case of conflict:** Normative sections are authoritative.

### Why This Matters

**Before 1.5:**
- Spec was mostly examples
- Implementation details unclear
- No way to verify conformance

**After 1.5:**
- Unambiguous requirements
- Testable semantics
- Implementation guide built-in
- Multiple compatible implementations possible

### Documentation Improvements (December 15, 2025)

**Added to Specification:**

1. **RFC 2119 Conformance Keywords**
   - Formal definition of MUST/SHOULD/MAY
   - Clear requirement levels for implementers
   - Standards-compliant specification language

2. **Design Principles Section**
   - Five explicit core principles
   - Safety First, Determinism, Explicitness, Fail-Fast, Minimalism
   - Explains design decisions (e.g., no break/continue)

3. **Statement Reference Index**
   - Maps all EBNF statements to semantic sections
   - Quick navigation for implementers
   - Completeness verification

4. **Function Return Semantics**
   - Clarified typed vs. untyped vs. void functions
   - Explicit rules for missing returns
   - Removed ambiguity in return behavior

5. **Structured Error Code Ranges**
   - Organized ranges by category (1000-1999: Memory, 2000-2999: Channels, etc.)
   - Reserved ranges for future expansion
   - Clear guidelines for custom error codes (9000-9999)

6. **Minimalism Principle**
   - Explicitly documents absence of break/continue as design choice
   - Functions and return are primary control flow abstractions
   - Aligns with "keep it simple" philosophy

**Impact:** Better specification clarity, easier implementation, clearer design rationale.

---
## Breaking Changes

### 🔥 Pointers Completely Removed

**What changed:**
- All pointer types removed from the language
- No pointer arithmetic  
- No dereference operators

**Migration:**
```cymple
📝 v1.4 - Pointers (REMOVED)
*🔢ptr ← &value
🔢val ← *ptr

📝 v1.5 - Handles only
💾block ← allocate(100)
📋val ← block[0]  📝 Array-like access, NOT pointer arithmetic
```

**Rationale:**
- Pointers were a safety hazard
- All memory access now through validated handles
- Indexing syntax (`block[i]`) is clearer and safer

**Impact:** HIGH - Any code using pointers must be rewritten

---

### 🔄 Handle Semantics Completely Redesigned

**What changed:**
- Handles are now **opaque and read-only**
- Generational validation added
- Fail-fast on stale handles

**New handle structure (internal):**
```
handle:
    slot/index      📝 Into runtime handle table
    generation      📝 Detect use-after-release
    kind/type       📝 Resource type
```

**Validation requirements (MUST):**
1. Slot in range?
2. Slot occupied?
3. Generation matches?
4. Kind matches?

**Example:**
```cymple
💾block ← allocate(100)
release(block)
📋val ← block[0]  📝 ERROR: Stale handle detected!
```

**Rationale:**
- Prevents use-after-free bugs
- Catches errors immediately
- Type-safe resource access

**Impact:** MEDIUM - Mostly transparent, but invalid handle use now fails fast

---

## Major Features

### ✨ Generational Handle Validation

Handles now carry **generation counters** to detect stale handle use:

```cymple
🧵 demonstrate_validation()
    💾block ← allocate(100)
    🔗 block -> B
        B[0] ← 42        📝 OK - valid handle, generation matches
    
    release(block)       📝 Generation increments, resource freed
    
    📋val ← block[0]     📝 ERROR: Generation mismatch!
    📝 Runtime catches this immediately
```

**Benefits:**
- Use-after-free detection
- No undefined behavior
- Clear error messages

---

### 🛠️ Memory Blocks with Bounds Checking

Memory blocks are now **explicit heap-managed objects**:

```cymple
💾block ← allocate(1024)  📝 Allocate 1024 bytes

📝 Safe indexing with validation
📋val ← block[0]         📝 OK - within bounds
block[42] ← 99           📝 OK - within bounds
block[1500] ← 5          📝 ERROR: Bounds check fails!
```

**Validation on every access:**
1. Handle valid?
2. Index in range?
3. Perform operation

**Properties:**
```cymple
🔢size ← block.size      📝 Get size
✅valid ← block.is_valid  📝 Check validity
```

---

### 🧵 Structured Concurrency

Tasks are now **automatically managed** within their scope:

```cymple
🧵 parent_function()
    🌀📦 📋results ← [task1(), task2(), task3()]
        ⏱️ 30s
            💬 "Timeout"
        ✅ 📋all
            ↩ 📋all
    
    📝 Tasks GUARANTEED to be done or cancelled here
    💬 "All tasks finished"
```

**Key features:**
- Tasks joined/cancelled on scope exit
- No orphaned tasks
- Clear lifecycle

**Explicit detach (with warning):**
```cymple
🧵 fire_and_forget()
    🧵detach worker_task()  📝 WARNING: May retain resources!
    ↩  📝 Parent returns immediately
```

---

### 🚦 Runtime Task Cap

The runtime enforces a **maximum concurrent tasks** (default: 1000):

```cymple
📝 Configurable at startup
runtime.max_concurrent_tasks ← 1000
```

**Purpose:**
- Prevents "spawn storms"
- Provides backpressure
- Protects system resources

**Behavior when limit reached:**
- Task creation blocks or queues
- No silent failures
- Clear feedback

---

### 📡 Channel Close Semantics

Channels now have **deterministic close behavior**:

**Explicit close:**
```cymple
📡ch ← 🛰️🔢
🚀 ch, 42
close(ch)  📝 Explicitly close
```

**RAII close (automatic):**
```cymple
🧵 sender()
    📡ch ← 🛰️🔢
    🚀 ch, 42
    📝 Channel auto-closes here
```

**Closed channel behavior:**
- **Send to closed channel:** Error/panic
- **Receive from closed channel:** Returns null/special value
- **Loop over closed channel:** Terminates cleanly

**Example:**
```cymple
🧵 producer(📡out)
    🔁 i = 1..10
        🚀 out, i
    close(out)  📝 Signal end

🧵 consumer(📡in)
    🔁 msg in in
        💬 "Got: 🔢msg"
    📝 Loop exits when channel closed
    💬 "Done"
```

---


### 📜 Normative Requirements Added

**NEW in v1.5:** Comprehensive normative requirements using RFC 2119 keywords.

**Requirements Count:**
- 57 MUST statements (critical requirements)
- 8 SHOULD statements (strong recommendations)
- 9 MAY statements (optional features)

**Key Requirements:**

**Memory Safety (MUST):**
- MUST validate handles on every use
- MUST check bounds before every memory access
- MUST detect stale handles via generation counters
- MUST panic on validation failures
- MUST release resources on scope exit

**Concurrency (MUST):**
- MUST enforce runtime task cap (prevent spawn storms)
- MUST join/cancel tasks on scope exit
- MUST use fail-fast panic behavior
- MUST NOT allow shared memory between tasks

**Error Handling (MUST):**
- MUST provide deterministic panic behavior
- MUST unwind stack with RAII cleanup
- MUST support Guru meditation for catching panics
- MUST provide standard error types

**Best Practices (SHOULD):**
- SHOULD provide explicit timeouts for blocking operations
- SHOULD close channels explicitly or via RAII
- SHOULD zero allocated memory by default
- SHOULD warn about detached tasks

**Rationale:** Clear requirements enable consistent implementations and prevent ambiguity.

---

## Code Examples

### Example 1: Safe Memory Block Indexing

```cymple
🧵 safe_buffer_demo()
    💾buffer ← allocate(100)
    
    📝 Safe indexing with validation
    🔗 buffer -> mut B
        🔁 i = 0..100
            B[i] ← i * 2
    
    📝 Validate before access
    🧘 guru(e)
        🔀 e.type
            ➜ "BoundsCheck"
                💬 "Index out of bounds"
                ↩
    
    💬 "Value at 50: buffer[50]"
    💬 "Value at 150: buffer[150]"  📝 ERROR caught!
```

### Example 2: Stale Handle Detection

```cymple
🧵 stale_handle_demo()
    💾block ← allocate(100)
    
    🔗 block -> B
        B[0] ← 42  📝 OK
    
    release(block)  📝 Handle now stale
    
    🧘 guru(e)
        🔀 e.type
            ➜ "GenerationMismatch"
                💬 "Stale handle detected!"
                ↩
    
    📋val ← block[0]  📝 ERROR: Stale handle
```

### Example 3: Structured Concurrency with Timeout

```cymple
🧵 parallel_fetch(📋urls) -> 📋
    🌀📦 📋results ← create_fetch_tasks(📋urls)
        ⏱️ 30s  📝 Overall timeout
            💬 "Fetch timeout - returning partial results"
            ↩ 📋results
        
        ⏩ 📋partial every 10  📝 Progress updates
            🔢percent ← (📋partial.length * 100) / 📋urls.length
            💬 "Progress: 🔢percent%"
            
            📝 Early cancellation if we have enough
            ❓ 📋partial.length >= 50
                🛑  📝 Stop remaining tasks
                ↩ 📋partial
        
        ✅ 📋all
            ↩ filter_ok(📋all)
    
    📝 All tasks cleaned up here
```

---

## Migration Guide

### From v1.4 to v1.5

#### 1. Remove All Pointer Code

**Before (v1.4):**
```cymple
*🔢ptr ← &value
🔢result ← *ptr + 10
```

**After (v1.5):**
```cymple
💾block ← allocate(100)
block[0] ← value
🔢result ← block[0] + 10
```

#### 2. Add Timeouts to Quantum Operations

**Before (v1.4):**
```cymple
🌀⚡ 🔤result ← [fetch1(), fetch2()]
    ✅ 🔤winner
        ↩ 🔤winner
```

**After (v1.5):**
```cymple
🌀⚡ 🔤result ← [fetch1(), fetch2()]
    ⏱️ 5s  📝 Add timeout!
        ↩ fallback()
    ✅ 🔤winner
        ↩ 🔤winner
```

#### 3. Handle Validation Errors

**Before (v1.4):**
```cymple
💾block ← allocate(100)
📋val ← block[0]  📝 May panic
```

**After (v1.5):**
```cymple
💾block ← allocate(100)

🧘 guru(e)
    🔀 e.type
        ➜ "HandleValidation"
            ↩ null

📋val ← block[0]  📝 Validated
```

---

## Summary

**Version 1.5 is all about safety and clarity:**

✅ Pointers removed - handles only
✅ Generational validation - detect stale handles
✅ Memory blocks - explicit heap management
✅ Structured concurrency - no orphaned tasks
✅ Runtime caps - prevent spawn storms
✅ Channel close semantics - deterministic cleanup
✅ Footguns & guidance - learn from mistakes

**Migration effort:** Low to Medium (2-4 hours for typical project)
**Safety improvement:** HIGH
**Recommended:** Upgrade immediately for better safety

---

**Cymple v1.5** - Memory safety through validated handles.

*Release Date: December 15, 2025*
*© 2025 Jörg Burbach*
