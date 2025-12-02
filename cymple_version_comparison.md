# Cymple Version Comparison: 1.2 vs 1.3

## Feature Comparison

| Feature | v1.2 FINAL | v1.3 | Status |
|---------|------------|------|--------|
| **String Interpolation** | ❌ Must use `+` concatenation | ✅ Direct: `"Value: 🔢x"` | NEW |
| **Else Symbol** | ❌ No dedicated symbol | ✅ `⤵️` for else blocks | NEW |
| **Property Length Access** | ❌ Must use `length()` function | ✅ `.length` / `.size` properties | NEW |
| **Range Loop Syntax** | ✅ `i in 1..10` only | ✅ `i in 1..10` OR `i = 1..10` | ENHANCED |
| **Void Function Returns** | ❌ Must declare `-> type` | ✅ Optional return type | NEW |
| **Comparison Operator** | ⚠️ Uses `=` (unusual) | ✅ Uses `==` (standard) | **BREAKING** |
| **Logical Operators** | ⚠️ Ambiguous `&` and `|` | ✅ Clear `&&` and `||` | **BREAKING** |
| **Bitwise Operators** | ✅ `&` `|` `^` | ✅ `&` `|` `^` (unchanged) | SAME |
| **Move Semantics** | ✅ Yes | ✅ Yes | SAME |
| **No GC (RAII)** | ✅ Yes | ✅ Yes | SAME |
| **Share-Nothing Concurrency** | ✅ Yes | ✅ Yes | SAME |
| **Quantum Operations** | ✅ Yes | ✅ Yes | SAME |
| **Pattern Matching** | ✅ Yes | ✅ Yes | SAME |
| **FFI Support** | ✅ Yes | ✅ Yes | SAME |
| **Type Prefixes Required** | ✅ Yes (🔢, 🔤, etc.) | ✅ Yes (🔢, 🔤, etc.) | SAME |

## Code Examples

### String Output

**v1.2:**
```cymple
🔢count ← 5
💬 "Count: " + 🔢count + " items"
```

**v1.3:**
```cymple
🔢count ← 5
💬 "Count: 🔢count items"
```

### Conditional Logic

**v1.2:**
```cymple
❓ x = 5        📝 = for comparison
    💬 "Five"
❓ x > 5
    💬 "More"
❓ x < 5
    💬 "Less"
```

**v1.3:**
```cymple
❓ x == 5       📝 == for comparison
    💬 "Five"
⤵️               📝 dedicated else
    ❓ x > 5
        💬 "More"
    ⤵️
        💬 "Less"
```

### Loop Syntax

**v1.2:**
```cymple
🔁 i in 1..10
    💬 "🔢i"
```

**v1.3 (both work):**
```cymple
🔁 i in 1..10
    💬 "Number: 🔢i"

🔁 i = 1..10
    💬 "Number: 🔢i"
```

### Collection Length

**v1.2:**
```cymple
📋nums ← [1, 2, 3]
🔢len ← length(📋nums)
💬 "Length: " + 🔢len
```

**v1.3:**
```cymple
📋nums ← [1, 2, 3]
💬 "Length: 🔢nums.length"
```

### Function Declaration

**v1.2:**
```cymple
🧵 greet(name: 🔤) -> 🔤
    💬 "Hello, " + name
    ↩ ""
```

**v1.3:**
```cymple
🧵 greet(name: 🔤)
    💬 "Hello, 🔤name!"
```

### Logical Operations

**v1.2:**
```cymple
❓ x > 0 & y > 0    📝 ambiguous: logical or bitwise?
    💬 "Both positive"
```

**v1.3:**
```cymple
❓ x > 0 && y > 0   📝 clear: logical AND
    💬 "Both positive"

🔢flags ← 0b1010 & 0b1100  📝 bitwise AND
```

## Migration Checklist

### Critical Changes (Required)

- [ ] Replace `=` with `==` in all comparisons
- [ ] Replace `&` with `&&` for logical AND
- [ ] Replace `|` with `||` for logical OR

### Optional Improvements (Recommended)

- [ ] Use string interpolation instead of concatenation
- [ ] Add `⤵️` else blocks for clarity
- [ ] Use property access for lengths (`.length`, `.size`)
- [ ] Remove return types from void functions
- [ ] Consider alternative range syntax where clearer

## Backward Compatibility

### Still Works in v1.3

✅ String concatenation with `+`
✅ `length()` function
✅ Explicit return types
✅ `in` keyword for ranges
✅ All core language features

### Breaking Changes

⚠️ Comparison operator change (`=` → `==`)
⚠️ Logical operator clarification (`&|` → `&&||`)

**Estimated migration time:** 30-60 minutes for typical project

## File Sizes

| Version | Spec File | EBNF | Total Package |
|---------|-----------|------|---------------|
| v1.2 | 47 KB | 8 KB | ~60 KB |
| v1.3 | 52 KB | 10 KB | ~70 KB |

**Documentation expansion:** +15% (more examples, clearer explanations)

## Recommended Version

**For new projects:** Use v1.3
- Modern syntax
- Clearer operators
- Better ergonomics

**For existing projects:** Migrate to v1.3
- Breaking changes are minimal
- Migration is straightforward
- Future improvements will build on v1.3

---

*Last updated: December 2, 2025*
