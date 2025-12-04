# CYMPLE Version Comparison

Comparing Cymple versions 1.2, 1.3, and 1.4

---

## Quick Reference

| Version | Release Date | Focus | Status |
|---------|-------------|-------|---------|
| **1.2** | 2025-11-28 | Production-ready baseline | Final |
| **1.3** | 2025-12-02 | Ergonomics & clarity | Superseded |
| **1.4** | 2025-12-04 | Parallel programming power | **Current** |

---

## Feature Matrix

### Core Language Features

| Feature | 1.2 | 1.3 | 1.4 |
|---------|-----|-----|-----|
| **String interpolation** | ❌ | ✅ | ✅ |
| **Else symbol (`⤵️`)** | ❌ | ✅ | ✅ |
| **Property access (`.length`)** | ❌ | ✅ | ✅ |
| **Range loop (`i = 1..10`)** | ❌ | ✅ | ✅ |
| **Optional void return** | ❌ | ✅ | ✅ |
| **Comparison (`==`)** | ⚠️ Used `=` | ✅ | ✅ |
| **Logical ops (`&&`, `||`)** | ⚠️ Used `&`, `|` | ✅ | ✅ |

### Quantum Operations

| Feature | 1.2 | 1.3 | 1.4 |
|---------|-----|-----|-----|
| **Race (`🌀⚡`)** | ✅ Basic | ✅ | ✅ Enhanced |
| **Collect (`🌀📦`)** | ✅ Basic | ✅ | ✅ Enhanced |
| **Timeout** | ✅ Numbers only | ✅ | ✅ **Time units** |
| **Race progress events** | ✅ | ✅ | ❌ **Removed** |
| **Collect progress** | ✅ Every task | ✅ | ✅ **`every N`** |
| **Early cancellation** | ❌ | ❌ | ✅ **`🛑`** |
| **Total failure event** | ❌ | ❌ | ✅ **New** |
| **Universal stop (`🛑`)** | ⚠️ Partial | ⚠️ Partial | ✅ **Universal** |

### Memory & Safety

| Feature | 1.2 | 1.3 | 1.4 |
|---------|-----|-----|-----|
| **Move semantics** | ✅ | ✅ | ✅ |
| **Borrowing** | ✅ | ✅ | ✅ |
| **RAII** | ✅ | ✅ | ✅ |
| **No GC** | ✅ | ✅ | ✅ |
| **Share-nothing** | ✅ | ✅ | ✅ |

---

## Side-by-Side Code Examples

### Example 1: String Interpolation

**Version 1.2:**
```cymple
🔢count ← 5
🔤name ← "Alice"
💬 "Count: " + 🔢count + ", Name: " + 🔤name
```

**Version 1.3+:**
```cymple
🔢count ← 5
🔤name ← "Alice"
💬 "Count: 🔢count, Name: 🔤name"
```

**Improvement:** Cleaner, more readable, less `+` signs.

---

### Example 2: Else Blocks

**Version 1.2:**
```cymple
❓ age >= 18
    💬 "Adult"
❓ age < 18  📝 Separate condition - unclear
    💬 "Minor"
```

**Version 1.3+:**
```cymple
❓ age >= 18
    💬 "Adult"
⤵️  📝 Clear alternative
    💬 "Minor"
```

**Improvement:** Explicit else path, clearer intent.

---

### Example 3: Collection Length

**Version 1.2:**
```cymple
📋nums ← [1, 2, 3]
🔢count ← length(📋nums)
💬 "Count: " + 🔢count
```

**Version 1.3+:**
```cymple
📋nums ← [1, 2, 3]
💬 "Count: 📋nums.length"
```

**Improvement:** Property-style access, works with interpolation.

---

### Example 4: Quantum Race with Timeout

**Version 1.2:**
```cymple
🌀⚡ 🔤result ← [fetch_eu(), fetch_us()]
    ⏱️ 3000  📝 Milliseconds? Unclear!
        💬 "Timeout"
    ⏩ 🔤partial  📝 Confusing - tasks cancelled!
        💬 "Server responded"
    ✅ 🔤winner
        ↩ 🔤winner
```

**Version 1.3:**
```cymple
🌀⚡ 🔤result ← [fetch_eu(), fetch_us()]
    ⏱️ 3000  📝 Still unclear
        💬 "Timeout"
    ⏩ 🔤partial  📝 Still confusing
        💬 "Server responded"
    ✅ 🔤winner
        ↩ 🔤winner
```

**Version 1.4:**
```cymple
🌀⚡ 🔤result ← [fetch_eu(), fetch_us()]
    ⏱️ 3s  📝 Clear!
        💬 "Timeout"
    📝 No ⏩ event - cleaner!
    ✅ 🔤winner
        ↩ 🔤winner
```

**Improvements:**
- Time units (3s vs 3000)
- Removed confusing progress events
- Clearer semantics

---

### Example 5: Batch Processing

**Version 1.2:**
```cymple
🌀📦 📋results ← [100 tasks...]
    ⏱️ 60000  📝 Minutes? Seconds?
        💬 "Timeout"
    
    ⏩ 📋partial  📝 Fires 100 times!
        💬 "Progress"
    
    ✅ 📋all
        📝 No way to stop early
        ↩ 📋all
```

**Version 1.3:**
```cymple
🌀📦 📋results ← [100 tasks...]
    ⏱️ 60000  📝 Still unclear
        💬 "Timeout"
    
    ⏩ 📋partial  📝 Still fires 100 times
        💬 "Progress"
    
    ✅ 📋all
        📝 Still no early stop
        ↩ 📋all
```

**Version 1.4:**
```cymple
🌀📦 📋results ← [100 tasks...]
    ⏱️ 1m  📝 Clear: 1 minute!
        💬 "Timeout"
    
    ⏩ 📋partial every 10  📝 Only 10 events!
        💬 "Progress: 📋partial.length"
        
        📝 NEW: Early cancellation
        ❓ user_cancelled()
            🛑
            ↩ 📋partial
    
    ✅ 📋all
        ↩ 📋all
    
    📝 NEW: Total failure handling
    ❌ 🔤total_failure
        💬 "All failed"
        ↩ []
```

**Improvements:**
- Time units (1m vs 60000)
- Progress frequency control (`every 10`)
- Early cancellation with `🛑`
- Total failure detection

---

## Breaking Changes

### Version 1.2 → 1.3

| Change | v1.2 | v1.3 | Migration |
|--------|------|------|-----------|
| **Comparison** | Used `=` | Uses `==` | Replace `=` with `==` in conditions |
| **Logical AND** | Used `&` | Uses `&&` | Replace `&` with `&&` in logic |
| **Logical OR** | Used `|` | Uses `||` | Replace `|` with `||` in logic |

**Impact:** High - Requires code changes
**Justification:** Aligns with standard programming conventions

### Version 1.3 → 1.4

| Change | v1.3 | v1.4 | Migration |
|--------|------|------|-----------|
| **Race progress** | Had `⏩` | No `⏩` | Remove `⏩` blocks from Race |

**Impact:** Low - Rarely used feature, confusing anyway
**Justification:** Clearer semantics (cancelled tasks don't need progress)

---

## Non-Breaking Additions

### Version 1.3 Additions

All backward compatible:
- ✅ String interpolation (old `+` still works)
- ✅ `⤵️` else symbol (old style still works)
- ✅ Property `.length` (function `length()` still works)
- ✅ Range loop `i = 1..10` (old `in` still works)
- ✅ Optional void return (old style still works)

### Version 1.4 Additions

All backward compatible:
- ✅ Timeout time units (numbers still work)
- ✅ Progress `every N` (without `every` still works)
- ✅ Early cancellation with `🛑` (optional feature)
- ✅ Total failure event (optional)
- ✅ Universal `🛑` (works in more places)

---

## Performance Comparison

### Progress Events (100 tasks)

| Version | Events Fired | Overhead | Notes |
|---------|-------------|----------|-------|
| **1.2** | 100 | High | No control |
| **1.3** | 100 | High | No control |
| **1.4** | 10 | **Low** | With `every 10` |

**Improvement:** 90% reduction in event overhead

### Early Cancellation (Search 10 DBs for 20 results)

| Version | DBs Searched | Time | Resources |
|---------|--------------|------|-----------|
| **1.2** | 10 | 10s | 100% |
| **1.3** | 10 | 10s | 100% |
| **1.4** | 3 | **3s** | **30%** |

**Improvement:** 70% faster, 70% less resource usage

---

## Code Complexity Comparison

### Multi-Server Search

| Version | Lines | Features |
|---------|-------|----------|
| **1.2** | 8 | Basic race, numeric timeout |
| **1.3** | 8 | Same as 1.2 |
| **1.4** | 8 | + Time units, cleaner semantics |

**Complexity:** Same lines, better readability

### Batch Processing with Progress

| Version | Lines | Features |
|---------|-------|----------|
| **1.2** | 6 | Basic collect, 100 events |
| **1.3** | 6 | Same as 1.2 |
| **1.4** | 12 | + Freq control, early cancel, total failure |

**Complexity:** +6 lines, 3x more features

---

## Migration Paths

### 1.2 → 1.3

**Required changes:**
1. Replace `=` with `==` in conditions
2. Replace `&` with `&&` for logical AND
3. Replace `|` with `||` for logical OR

**Optional improvements:**
4. Use string interpolation
5. Use `⤵️` for else blocks
6. Use property `.length`/`.size`
7. Use `i = 1..10` for ranges

**Effort:** Low to Medium (30 minutes to 2 hours)
**Risk:** Low (breaking changes are mechanical)

### 1.3 → 1.4

**Required changes:**
1. Remove `⏩` events from Race operations (if any)

**Optional improvements:**
2. Add time units to timeouts (`5s` instead of `5000`)
3. Add `every N` to progress events (large task lists)
4. Consider early cancellation with `🛑`
5. Add total failure handling where appropriate

**Effort:** Very Low (15-30 minutes)
**Risk:** Very Low (mostly additive features)

### 1.2 → 1.4 (Direct)

**Required changes:**
1. All changes from 1.2 → 1.3
2. All changes from 1.3 → 1.4

**Effort:** Medium (1-3 hours)
**Risk:** Low (well-defined changes)

---

## Recommended Version

### For New Projects

**Use 1.4** - Latest features, best performance, clearest semantics

### For Existing Projects

**From 1.2:** Migrate to 1.4 (1-3 hours, worth it)
**From 1.3:** Upgrade to 1.4 (15-30 minutes, easy)

---

## Feature Timeline

```
1.2 (Nov 2025)
├─ Production-ready baseline
├─ Basic quantum operations
└─ Core language features

1.3 (Dec 2025)
├─ String interpolation
├─ Else symbol (⤵️)
├─ Property access
├─ Range loop alternatives
└─ Operator clarifications

1.4 (Dec 2025)
├─ Timeout time units
├─ Progress frequency control
├─ Early cancellation
├─ Total failure detection
├─ Simplified Race
└─ Universal stop signal
```

---

## Summary

| Aspect | 1.2 | 1.3 | 1.4 |
|--------|-----|-----|-----|
| **Maturity** | Baseline | Refined | Enhanced |
| **Ergonomics** | Good | Better | Best |
| **Parallel Perf** | Good | Good | Excellent |
| **Code Length** | Baseline | -10% | -15% |
| **Readability** | Good | Better | Best |
| **Breaking Changes** | N/A | 3 | 1 |
| **New Features** | N/A | 6 | 6 |
| **Recommended** | ⚠️ Outdated | ⚠️ Superseded | ✅ **Current** |

---

## Changelog Links

- [Cymple 1.3 Changelog](cymple_1_3_changelog.md)
- [Cymple 1.4 Changelog](cymple_1_4_changelog.md)

---

## Specification Links

- [Cymple 1.3 Specification](cymple_spec_1_3.md)
- [Cymple 1.4 Specification](cymple_spec_1_4.md)

---

*Last updated: December 4, 2025*
*© 2025 Jörg Burbach*
