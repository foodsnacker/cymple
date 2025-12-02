# CYMPLE CHANGELOG - Version 1.3

**Release Date:** December 2, 2025

---

## New Features

### 1. 🎨 String Interpolation

Variables can now be embedded directly in strings using their emoji type prefix.

**Before (v1.2):**
```cymple
🔢count ← 5
💬 "Count: " + 🔢count
```

**After (v1.3):**
```cymple
🔢count ← 5
💬 "Count: 🔢count"
```

**Benefits:**
- Cleaner, more readable code
- No need for string concatenation with `+`
- Works with all variable types
- Old syntax still supported for backward compatibility

**Examples:**
```cymple
🔢x ← 10
🔢y ← 20
💬 "Sum: 🔢x + 🔢y = 🔢result"

🔤name ← "Alice"
🔢age ← 30
💬 "Hello, 🔤name! You are 🔢age years old."
```

---

### 2. ⤵️ Dedicated Else Symbol

A new symbol `⤵️` for else blocks makes control flow more explicit and readable.

**Before (v1.2):**
```cymple
❓ x > 0
    💬 "positive"
❓ x < 0
    💬 "negative"
```

**After (v1.3):**
```cymple
❓ x > 0
    💬 "positive"
⤵️
    💬 "not positive"

❓ age >= 18
    💬 "Adult"
⤵️
    💬 "Minor"
```

**Benefits:**
- Clearer intent - explicitly marks alternative path
- Better visual distinction from sequential if statements
- More intuitive for beginners
- Reduces ambiguity in nested conditions

---

### 3. 📏 Property-Based Length/Size Access

Collections now support property-style access for lengths and sizes.

**Before (v1.2):**
```cymple
🔢len ← length([1, 2, 3])
🔢size ← length(🗺️map)
```

**After (v1.3):**
```cymple
📋nums ← [1, 2, 3]
🔢len ← 📋nums.length

🗺️data ← {"a": 1, "b": 2}
🔢size ← 🗺️data.size

🔤text ← "Hello"
🔢chars ← 🔤text.length
```

**Benefits:**
- Consistent with property access pattern
- More object-oriented feel
- Matches string `.length` syntax
- Old `length()` function still works

**Property Reference:**
| Type | Property | Returns |
|------|----------|---------|
| 🔤 String | `.length` | Number of characters |
| 📋 List | `.length` | Number of items |
| 🗺️ Map | `.size` | Number of key-value pairs |
| 🔣 Bytearray | `.length` | Number of bytes |

---

### 4. 🔁 Alternative Range Loop Syntax

Two ways to write range-based loops - choose what feels natural!

**Both syntaxes work:**
```cymple
🔁 i in 1..10
    💬 "Number: 🔢i"

🔁 i = 1..10
    💬 "Number: 🔢i"
```

**Benefits:**
- Flexibility - use `in` or `=` based on preference
- `=` feels more like traditional for loops
- `in` is more explicit about iteration
- Both compile to identical code

**Works with all range features:**
```cymple
🔁 i = 1..100 ⏩ 10
    💬 "Step: 🔢i"

🔁 item = list
    💬 "Item: 🔤item"
```

---

### 5. 🧵 Optional Return Type for Void Functions

Functions that don't return values no longer need return type declarations.

**Before (v1.2):**
```cymple
🧵 greet(name: 🔤) -> 🔤
    💬 "Hello, 🔤name!"
    ↩ ""  📝 awkward empty return
```

**After (v1.3):**
```cymple
🧵 greet(name: 🔤)
    💬 "Hello, 🔤name!"
    📝 no return needed!
```

**Benefits:**
- Less boilerplate for action functions
- Clearer intent - function performs actions, doesn't return data
- No need for dummy return values
- Makes code more concise

**Examples:**
```cymple
🧵 print_report(data: 📋)
    💬 "=== Report ==="
    🔁 item in data
        💬 "- 🔤item"

🧵 initialize()
    💬 "Starting..."
    setup_connections()
    load_config()
```

---

## Clarifications and Breaking Changes

### ⚖️ Comparison Operators

**BREAKING CHANGE:** Cymple now uses `==` for equality comparison (not `=`).

**Correct (v1.3):**
```cymple
❓ x == 5
    💬 "Equal"

❓ name == "Alice"
    💬 "Hello, Alice!"
```

**Incorrect:**
```cymple
❓ x = 5     📝 ERROR: Use == for comparison
```

**Why this change?**
- Aligns with most programming languages (Python, JavaScript, C, Java, etc.)
- Reduces confusion for developers coming from other languages
- `=` is not used in Cymple (assignment uses `←`)

---

### 🔣 Logical vs Bitwise Operators

**CLARIFIED:** Clear distinction between logical and bitwise operations.

**Logical Operators (boolean logic):**
```cymple
❓ x > 0 && y > 0
    💬 "Both positive"

❓ x < 0 || y < 0
    💬 "At least one negative"

❓ !ready
    💬 "Not ready"
```

**Bitwise Operators (bit manipulation):**
```cymple
🔢flags ← 0b1010
🔢mask ← 0b1100

🔢result ← flags & mask     📝 bitwise AND
🔢result ← flags | mask     📝 bitwise OR
🔢result ← flags ^ mask     📝 bitwise XOR
🔢result ← flags << 2       📝 left shift
🔢result ← flags >> 1       📝 right shift
```

**Operator Reference:**
| Purpose | Operator | Example |
|---------|----------|---------|
| Logical AND | `&&` | `x > 0 && y > 0` |
| Logical OR | `||` | `x < 0 || y < 0` |
| Logical NOT | `!` | `!ready` |
| Bitwise AND | `&` | `flags & mask` |
| Bitwise OR | `|` | `flags | mask` |
| Bitwise XOR | `^` | `flags ^ mask` |
| Left Shift | `<<` | `value << 2` |
| Right Shift | `>>` | `value >> 1` |

---

## Backward Compatibility

### What Still Works

✅ **Old string concatenation:**
```cymple
💬 "Hello, " + 🔤name + "!"
```

✅ **Function-style length:**
```cymple
🔢len ← length(📋list)
```

✅ **Single comparison operator:**
```cymple
❓ x == 5  📝 works in both versions
```

### What Changed

⚠️ **Comparison operator:**
- v1.2: Used `=` for comparison (unusual)
- v1.3: Uses `==` for comparison (standard)
- **Migration:** Replace `=` with `==` in all conditions

⚠️ **Logical operators:**
- v1.2: Used `&` and `|` for both logical AND bitwise (ambiguous)
- v1.3: Separate `&&`, `||` (logical) from `&`, `|` (bitwise)
- **Migration:** Use `&&` and `||` for boolean logic

---

## Migration Guide

### From v1.2 to v1.3

**Step 1: Update comparison operators**
```cymple
📝 Old (v1.2)
❓ x = 5

📝 New (v1.3)
❓ x == 5
```

**Step 2: Update logical operators**
```cymple
📝 Old (v1.2)
❓ x > 0 & y > 0

📝 New (v1.3)
❓ x > 0 && y > 0
```

**Step 3: Simplify strings (optional)**
```cymple
📝 Old style (still works)
💬 "Value: " + 🔢x

📝 New style (recommended)
💬 "Value: 🔢x"
```

**Step 4: Add else blocks (optional)**
```cymple
📝 Old style
❓ x > 0
    💬 "positive"
❓ x <= 0
    💬 "not positive"

📝 New style (clearer)
❓ x > 0
    💬 "positive"
⤵️
    💬 "not positive"
```

**Step 5: Use property access (optional)**
```cymple
📝 Old style (still works)
🔢len ← length(📋list)

📝 New style (recommended)
🔢len ← 📋list.length
```

---

## Summary of Changes

| Feature | v1.2 | v1.3 | Breaking? |
|---------|------|------|-----------|
| String interpolation | ❌ | ✅ | No (additive) |
| Else symbol `⤵️` | ❌ | ✅ | No (additive) |
| Property `.length`/`.size` | ❌ | ✅ | No (additive) |
| Range loop `i = 1..10` | ❌ | ✅ | No (additive) |
| Optional void return | ❌ | ✅ | No (additive) |
| Comparison `==` (not `=`) | ⚠️ | ✅ | **Yes** |
| Logical `&&`/`||` | ⚠️ | ✅ | **Yes** |

**Upgrade recommendation:** Update comparison and logical operators immediately. Other features can be adopted gradually.

---

## Examples: Before and After

### Example 1: Simple Loop

**v1.2:**
```cymple
🔢i ← 1
🔁 i <= 10
    💬 "Number: " + 🔢i
    🔢i ← 🔢i + 1
```

**v1.3:**
```cymple
🔁 i = 1..10
    💬 "Number: 🔢i"
```

### Example 2: Conditional Logic

**v1.2:**
```cymple
❓ age = 18
    💬 "Exactly 18"
❓ age > 18
    💬 "Over 18"
❓ age < 18
    💬 "Under 18"
```

**v1.3:**
```cymple
❓ age == 18
    💬 "Exactly 18"
⤵️
    ❓ age > 18
        💬 "Over 18"
    ⤵️
        💬 "Under 18"
```

### Example 3: List Processing

**v1.2:**
```cymple
📋numbers ← [1, 2, 3, 4, 5]
🔢count ← length(📋numbers)
💬 "List has " + 🔢count + " items"
```

**v1.3:**
```cymple
📋numbers ← [1, 2, 3, 4, 5]
💬 "List has 🔢numbers.length items"
```

---

## Looking Forward

Version 1.3 focuses on ergonomics and clarity while maintaining Cymple's core principles:
- ✅ Explicit type prefixes (not changing - it's a core feature)
- ✅ No garbage collection
- ✅ Move semantics and ownership
- ✅ Share-nothing concurrency
- ✅ Procedural simplicity

Future versions will continue to refine the developer experience without compromising these fundamentals.

---

**Thank you for using Cymple!**

For questions or feedback, please visit: https://github.com/foodsnacker/cymple

---

*Cymple 1.3 - December 2, 2025*
*© 2025 Jörg Burbach*
