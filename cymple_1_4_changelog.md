# CYMPLE CHANGELOG - Version 1.4

**Release Date:** December 4, 2025

---

## Overview

Version 1.4 focuses on making parallel programming more practical and powerful. The enhancements to Quantum Operations provide better control, visibility, and error handling while keeping the syntax concise and readable.

**Key Improvements:**
- ⏱️ Timeout with human-readable time units
- ⏩ Progress events with frequency control
- 🛑 Early cancellation in Collect operations
- ❌ Total failure detection
- Simplified Race (no progress events)
- Consistent stop signal across all features

---

## New Features

### 1. ⏱️ Timeout with Time Units

Timeouts now support human-readable time units instead of just milliseconds.

**Before (v1.3):**
```cymple
🌀⚡ 🔤result ← [fetch_eu(), fetch_us()]
    ⏱️ 3000  📝 Is this 3000 milliseconds? Unclear!
        💬 "Timeout"
```

**After (v1.4):**
```cymple
🌀⚡ 🔤result ← [fetch_eu(), fetch_us()]
    ⏱️ 3s  📝 Clear: 3 seconds!
        💬 "Timeout"
```

**Supported units:**
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

**Benefits:**
- More readable code
- No mental math required
- Less confusion about units
- Still supports variables for dynamic timeouts

---

### 2. ⏩ Progress Event Frequency Control

Control how often progress events fire with the `every N` modifier.

**Before (v1.3):**
```cymple
🌀📦 📋results ← [100 tasks...]
    ⏩ 📋partial
        💬 "Progress: 📋partial.length"
        📝 Fires after EVERY SINGLE task = 100 events!
```

**After (v1.4):**
```cymple
🌀📦 📋results ← [100 tasks...]
    ⏩ 📋partial every 10
        💬 "Progress: 📋partial.length"
        📝 Fires only 10 times = much better performance!
```

**With variables:**
```cymple
🔢update_interval ← 5

🌀📦 📋results ← [50 tasks...]
    ⏩ 📋partial every 🔢update_interval
        💬 "Progress: 📋partial.length"
```

**Benefits:**
- Reduces event overhead for large task lists
- Better UI performance (fewer updates)
- Flexible with literal numbers or variables
- Still works without `every` (fires after each task)

**Recommended values:**
- 1-10 tasks: No modifier (default)
- 10-100 tasks: `every 5` or `every 10`
- 100+ tasks: `every 10` or higher

---

### 3. 🛑 Early Cancellation in Collect

Stop all running tasks when you have enough results.

**New feature (v1.4):**
```cymple
🌀📦 📋results ← [search_db1(), search_db2(), search_db3()]
    ⏩ 📋partial
        💬 "Found: 📋partial.length results"
        
        📝 Stop when we have enough
        ❓ 📋partial.length >= 20
            💬 "Enough results found"
            🛑  📝 Stops all running tasks!
            ↩ 📋partial
    
    ✅ 📋all
        ↩ 📋all
```

**Use cases:**
- **Search aggregation**: Stop when you have enough results
- **Resource allocation**: Stop when target is reached
- **User cancellation**: Let user stop long-running operations
- **Error threshold**: Stop if too many errors occur

**Example: User cancellation**
```cymple
🌀📦 📋results ← [100 tasks...]
    ⏩ 📋partial every 5
        🖼️update_progress(📋partial.length)
        
        📝 Check if user clicked "Cancel"
        ❓ 🔘user_cancelled
            💬 "Cancelled by user"
            🛑
            ↩ 📋partial
    
    ✅ 📋all
        ↩ 📋all
```

**Example: Error threshold**
```cymple
🌀📦 📋results ← [download1(), download2(), download3()]
    ⏩ 📋partial
        📋errors ← filter_error(📋partial)
        
        📝 Stop if too many failures
        ❓ 📋errors.length > 10
            💬 "Too many errors - stopping"
            🛑
            ↩ filter_ok(📋partial)
```

**Benefits:**
- Saves resources (stops unnecessary work)
- Better user experience (responsive cancellation)
- Flexible condition checking
- Keeps already-completed results

---

### 4. ❌ Total Failure Event

Detect when every single task has failed.

**New event (v1.4):**
```cymple
🌀📦 📋results ← [download1(), download2(), download3()]
    ✅ 📋all
        📋ok ← filter_ok(📋all)
        📋err ← filter_error(📋all)
        ↩ 📋ok
    
    ❌ 🔤total_failure  📝 NEW: Only fires if ALL failed
        💬 "All downloads failed"
        🧘 🔤total_failure
            ↩ false
```

**When it fires:**
- ONLY when every single task has failed
- NOT when some tasks succeed and some fail
- Provides clean fallback logic

**Use cases:**
```cymple
📝 Multi-server fallback
🌀📦 📋data ← [fetch_primary(), fetch_backup1(), fetch_backup2()]
    ✅ 📋all
        📋ok ← filter_ok(📋all)
        ↩ 📋ok[1]  📝 Return first success
    
    ❌ 🔤total_failure
        💬 "All servers unreachable"
        ↩ cached_data()

📝 Required tasks
🌀📦 📋required ← [init_db(), load_config(), setup_auth()]
    ✅ 📋all
        📋ok ← filter_ok(📋all)
        ❓ 📋ok.length == 3
            💬 "All initialized"
            ↩ ✅
        💬 "Partial initialization"
        ↩ ✗
    
    ❌ 🔤total_failure
        💀 "Critical: Cannot start application"
```

**Benefits:**
- Clear semantics (all vs some failures)
- Better error handling
- Cleaner fallback logic
- Works with Guru meditation

---

### 5. Simplified Race Operations

Race no longer emits progress events (which made no sense).

**Before (v1.3):**
```cymple
🌀⚡ 🔤result ← [task1(), task2(), task3()]
    ⏩ 🔤partial  📝 Confusing: other tasks are cancelled!
        💬 "Task done: 🔤partial"
    ✅ 🔤winner
        ↩ 🔤winner
```

**After (v1.4):**
```cymple
🌀⚡ 🔤result ← [task1(), task2(), task3()]
    📝 No ⏩ event! Cleaner and makes sense.
    ✅ 🔤winner
        ↩ 🔤winner
```

**Why?**
- Race returns the FIRST result
- All other tasks are immediately cancelled with 🛑
- Progress events for cancelled tasks don't make sense
- Simpler mental model

**Race events (v1.4):**
| Event | When | Required |
|-------|------|----------|
| `⏱️` | Timeout | Optional |
| `✅` | First success | **Required** |
| `❌` | All fail | Optional |

---

### 6. Consistent 🛑 Stop Signal

The `🛑` symbol now works consistently across all features.

**Quantum Operations:**
```cymple
⏩ 📋partial
    ❓ condition
        🛑  📝 Stops all running tasks
```

**Channels:**
```cymple
🔁 msg in 📨channel
    ❓ msg == "stop"
        🛑  📝 Closes channel
```

**Loops:**
```cymple
🔁 i in 1..1000
    ❓ should_stop()
        🛑  📝 Breaks loop (same as "break")
```

**Tasks:**
```cymple
🧵 long_task()
    🔁 i in 1..1000
        ❓ 🛑  📝 Check if task was cancelled
            💬 "Task cancelled"
            ↩
        process(i)
```

**Benefits:**
- One symbol, consistent meaning
- Easier to remember
- Works everywhere
- Clear intent

---

## Behavior Changes

### Timeout Default Behavior

**Without handler (v1.3):**
```cymple
🌀⚡ 🔤result ← [slow_task()]
    ⏱️ 5000
    ✅ 🔤winner
        ↩ 🔤winner
📝 What happens on timeout? Unclear!
```

**Without handler (v1.4):**
```cymple
🌀⚡ 🔤result ← [slow_task()]
    ⏱️ 5s
    ✅ 🔤winner
        ↩ 🔤winner
📝 Timeout automatically throws ❌ "Timeout"
```

**For Collect:**
```cymple
🌀📦 📋results ← [tasks...]
    ⏱️ 10s
    ✅ 📋all
        ↩ 📋all
📝 Timeout returns partial results (📋results contains completed tasks)
```

---

## Migration Guide

### From v1.3 to v1.4

**Step 1: Update timeouts to use units (optional but recommended)**
```cymple
📝 Old (still works)
⏱️ 5000

📝 New (clearer)
⏱️ 5s
```

**Step 2: Add frequency control to progress events (for large task lists)**
```cymple
📝 Old (v1.3)
🌀📦 📋results ← [100 tasks...]
    ⏩ 📋partial
        update(📋partial.length)

📝 New (v1.4, better performance)
🌀📦 📋results ← [100 tasks...]
    ⏩ 📋partial every 10
        update(📋partial.length)
```

**Step 3: Remove progress events from Race (if you had any)**
```cymple
📝 Old (v1.3)
🌀⚡ 🔤result ← [tasks...]
    ⏩ 🔤partial  📝 Remove this!
        💬 "Progress"
    ✅ 🔤winner
        ↩ 🔤winner

📝 New (v1.4)
🌀⚡ 🔤result ← [tasks...]
    ✅ 🔤winner
        ↩ 🔤winner
```

**Step 4: Consider early cancellation (optional)**
```cymple
📝 New feature in v1.4
🌀📦 📋results ← [search_tasks...]
    ⏩ 📋partial
        ❓ 📋partial.length >= 20
            🛑
            ↩ 📋partial
```

**Step 5: Add total failure handling (optional)**
```cymple
📝 New feature in v1.4
🌀📦 📋results ← [tasks...]
    ✅ 📋all
        ↩ 📋all
    ❌ 🔤total_failure
        💬 "Everything failed"
        ↩ []
```

---

## Backward Compatibility

### What Still Works

✅ **Old timeout syntax (no units):**
```cymple
⏱️ 5000  📝 Still interpreted as milliseconds
```

✅ **Progress without frequency:**
```cymple
⏩ 📋partial  📝 Fires after each task (v1.3 behavior)
```

✅ **Collect without total_failure event:**
```cymple
🌀📦 📋results ← [tasks...]
    ✅ 📋all
        ↩ 📋all
📝 Works fine, just no special handling for total failure
```

### What Changed

⚠️ **Race no longer has ⏩ events:**
- v1.3: Had `⏩` event (confusing)
- v1.4: No `⏩` event (clearer)
- **Migration:** Remove `⏩` blocks from Race operations

⚠️ **🛑 is now consistent everywhere:**
- v1.3: Used in some places
- v1.4: Works in Quantum ops, channels, loops, tasks
- **Migration:** Can now use `🛑` in more contexts

---

## Before and After Examples

### Example 1: Multi-Server Search

**v1.3:**
```cymple
🌀⚡ 🔤result ← [fetch_eu(), fetch_us(), fetch_asia()]
    ⏱️ 3000
        💬 "Timeout after 3000 what?"
        ↩ cached()
    ⏩ 🔤partial
        💬 "Server done"  📝 Confusing!
    ✅ 🔤winner
        ↩ 🔤winner
```

**v1.4:**
```cymple
🌀⚡ 🔤result ← [fetch_eu(), fetch_us(), fetch_asia()]
    ⏱️ 3s  📝 Clear!
        💬 "Timeout after 3 seconds"
        ↩ cached()
    📝 No ⏩ event - cleaner!
    ✅ 🔤winner
        ↩ 🔤winner
```

### Example 2: Batch Processing

**v1.3:**
```cymple
🌀📦 📋results ← [100 tasks...]
    ⏱️ 60000
        💬 "Timeout"
        ↩ 📋results
    
    ⏩ 📋partial  📝 Fires 100 times!
        💬 "Progress: 📋partial.length"
    
    ✅ 📋all
        ↩ 📋all
```

**v1.4:**
```cymple
🌀📦 📋results ← [100 tasks...]
    ⏱️ 1m  📝 Readable!
        💬 "Timeout"
        ↩ 📋results
    
    ⏩ 📋partial every 10  📝 Fires only 10 times!
        💬 "Progress: 📋partial.length"
        
        📝 NEW: Can stop early
        ❓ user_cancelled()
            🛑
            ↩ 📋partial
    
    ✅ 📋all
        ↩ 📋all
    
    📝 NEW: Handle total failure
    ❌ 🔤total_failure
        💬 "Everything failed"
        ↩ []
```

### Example 3: Search Aggregation

**v1.3:**
```cymple
🌀📦 📋results ← [search_db1(), search_db2(), search_db3()]
    ⏩ 📋partial
        💬 "Found: 📋partial.length"
        📝 No way to stop early!
    
    ✅ 📋all
        📝 Always waits for all, even if we have enough
        ↩ 📋all
```

**v1.4:**
```cymple
🌀📦 📋results ← [search_db1(), search_db2(), search_db3()]
    ⏩ 📋partial
        💬 "Found: 📋partial.length"
        
        📝 NEW: Stop when we have enough!
        ❓ 📋partial.length >= 20
            💬 "Enough results"
            🛑
            ↩ 📋partial
    
    ✅ 📋all
        ↩ 📋all
```

---

## Performance Impact

### Progress Event Frequency

**v1.3 (100 tasks):**
- 100 progress events fired
- 100 UI updates
- High overhead

**v1.4 (100 tasks with `every 10`):**
- 10 progress events fired
- 10 UI updates
- **90% reduction in overhead**

### Early Cancellation

**Scenario:** Searching 10 databases for 20 results

**v1.3:**
- Searches all 10 databases
- Takes ~10 seconds
- Wastes resources on databases 4-10

**v1.4:**
- Finds 20 results in first 3 databases
- Stops remaining 7 searches with 🛑
- Takes ~3 seconds
- **70% faster, 70% less resource usage**

---

## Summary of Changes

| Feature | v1.3 | v1.4 | Impact |
|---------|------|------|--------|
| **Timeout units** | Only numbers | `ms`, `s`, `m`, `h` | Better readability |
| **Progress frequency** | Every task | `every N` | 50-90% less overhead |
| **Early cancellation** | ❌ Not possible | ✅ `🛑` in `⏩` | Faster, more efficient |
| **Total failure** | ❌ No event | ✅ `❌ total_failure` | Better error handling |
| **Race progress** | ⚠️ Had `⏩` | ✅ No `⏩` | Clearer semantics |
| **Stop signal** | ⚠️ Inconsistent | ✅ Universal `🛑` | Easier to use |

**Upgrade recommendation:** 
- Timeouts: Update to time units (optional but recommended)
- Progress: Add `every N` for large task lists
- Race: Remove `⏩` events if present
- Collect: Consider early cancellation and total failure handling

---

## Looking Forward

Version 1.4 significantly improves the practicality of parallel programming in Cymple. Future versions will continue to refine concurrent operations while maintaining:
- ✅ Simplicity and readability
- ✅ Deterministic behavior
- ✅ No garbage collection
- ✅ Share-nothing concurrency
- ✅ Type safety

---

**Thank you for using Cymple!**

For questions or feedback, please visit: https://github.com/foodsnacker/cymple

---

*Cymple 1.4 - December 4, 2025*
*© 2025 Jörg Burbach*
