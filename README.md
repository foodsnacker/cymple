# CYMPLE Language Package v1.3

**Procedural Programming with Unicode Symbols**

---

## What's in This Package?

This is the complete language specification package for **Cymple Version 1.3** (Released December 2, 2025).

### 📄 Files Included

1. **cymple_spec_1_3.md** - Complete language specification (40+ pages)
2. **cymple_ebnf_1_3.txt** - Formal EBNF grammar definition
3. **CYMPLE_1_3_CHANGELOG.md** - Detailed changelog with migration guide
4. **README.md** - This file

---

## What is Cymple?

Cymple is a modern procedural programming language that uses Unicode emoji symbols instead of traditional keywords. It's designed for:

- **Safety**: Move semantics, ownership, RAII - no garbage collection
- **Clarity**: Visual symbols make code structure immediately apparent
- **Concurrency**: Share-nothing tasks with channels
- **Performance**: Deterministic cleanup, no GC pauses
- **Simplicity**: Procedural design, no classes or OOP

---

## What's New in Version 1.3?

### 🆕 Major Features

1. **String Interpolation** - Embed variables directly: `"Value: 🔢x"`
2. **Else Symbol ⤵️** - Dedicated symbol for else blocks
3. **Property Access** - Clean syntax: `📋list.length` instead of `length(list)`
4. **Alternative Loop Syntax** - Use `i = 1..10` or `i in 1..10`
5. **Optional Return Types** - Void functions don't need return declaration

### ⚠️ Breaking Changes

- Use `==` for comparison (not `=`)
- Use `&&` and `||` for logical operations
- Bitwise operators are `&`, `|`, `^`

See **CYMPLE_1_3_CHANGELOG.md** for detailed migration guide.

---

## Quick Start Examples

### Hello World with String Interpolation
```cymple
🔤name ← "World"
💬 "Hello, 🔤name!"
```

### FizzBuzz (1.3 Style)
```cymple
🔁 i = 1..100
    ❓ i % 15 == 0
        💬 "FizzBuzz"
    ⤵️
        ❓ i % 3 == 0
            💬 "Fizz"
        ⤵️
            ❓ i % 5 == 0
                💬 "Buzz"
            ⤵️
                💬 "🔢i"
```

### Function with Property Access
```cymple
🧵 analyze_list(items: 📋)
    💬 "List has 🔢items.length elements"
    
    🔢sum ← 0
    🔁 item in items
        🔢sum ← 🔢sum + item
    
    🔢avg ← 🔢sum / 🔢items.length
    💬 "Average: 🔢avg"
```

---

## Symbol Quick Reference

### Core Language
- `←` Assignment
- `↩` Return
- `❓` If/Conditional
- `⤵️` Else (new in 1.3!)
- `🔁` Loop
- `📝` Comment

### Data Types
- `🔢` Number
- `🔤` String
- `📋` List
- `🗺️` Map
- `✅` True
- `✗` False

### Functions & Tasks
- `🧵` Function/Task
- `📡` Channel
- `🛰️` Create Channel
- `🚀` Send
- `🎯` Receive

### I/O
- `💬` Print/Output
- `🔤🔽` Input (prompt)

See **cymple_spec_1_3.md** for complete symbol reference.

---

## Documentation Structure

### cymple_spec_1_3.md

Complete language specification with:
- Overview and design principles
- Full syntax and semantics
- Type system details
- Control flow structures
- Concurrency model
- Error handling
- FFI (Foreign Function Interface)
- Complete examples
- Design rationale

**Start here** if you're new to Cymple or implementing the language.

### cymple_ebnf_1_3.txt

Formal grammar definition in Extended Backus-Naur Form (EBNF).

**Use this** for:
- Parser implementation
- Syntax validation
- Tool development
- IDE integration

### CYMPLE_1_3_CHANGELOG.md

Detailed changelog with:
- New features explained with examples
- Breaking changes highlighted
- Migration guide from v1.2
- Before/after code comparisons
- Backward compatibility notes

**Use this** if upgrading from v1.2 or tracking language evolution.

---

## Language Characteristics

### ✅ What Cymple IS

- Procedural programming language
- Unicode symbols for keywords
- Move semantics & ownership
- No garbage collection (RAII)
- Share-nothing concurrency
- Both interpreted and compiled
- Strongly typed with explicit prefixes

### ❌ What Cymple IS NOT

- Object-oriented (no classes)
- Functional (no first-class functions)
- Dynamically typed
- Garbage collected
- Shared-memory concurrent

---

## Implementation Status

**Version 1.3 is a SPECIFICATION only.**

This package contains:
- ✅ Complete language specification
- ✅ Formal grammar (EBNF)
- ✅ Syntax rules and semantics
- ✅ Example programs

This package does NOT contain:
- ❌ Compiler implementation
- ❌ Interpreter implementation
- ❌ Standard library
- ❌ IDE tools

### For Implementers

If you're interested in implementing Cymple:

1. Read **cymple_spec_1_3.md** thoroughly
2. Study the **cymple_ebnf_1_3.txt** grammar
3. Review example programs in the spec
4. Consider starting with an interpreter (simpler than compiler)
5. Focus on core features first (variables, functions, control flow)
6. Add concurrency features later

The specification is designed to be implementation-agnostic - you can target any platform.

---

## Design Philosophy

### Why Unicode Symbols?

- **Visual clarity**: `🔢` instantly indicates a number
- **International**: No English keywords to learn
- **Compact**: Less typing than `let`, `function`, etc.
- **Distinctive**: Code structure is immediately visible

### Why No Garbage Collection?

- **Deterministic**: Know exactly when cleanup happens
- **Predictable**: No GC pauses
- **Real-time ready**: Suitable for embedded systems
- **Lower latency**: Better performance characteristics

### Why Move Semantics?

- **Safety**: Prevents data races by design
- **Clear ownership**: Always know who owns what
- **No hidden copies**: Performance is explicit
- **Compiler enforced**: Catches errors at compile time

### Why Share-Nothing Concurrency?

- **No data races**: Impossible by design
- **No locks needed**: Tasks communicate via channels
- **Simpler reasoning**: Clear message-passing model
- **Scales well**: Each task is independent

---

## Comparison to Other Languages

| Feature | Cymple | Rust | Go | Python |
|---------|--------|------|----|----|
| GC | No (RAII) | No | Yes | Yes |
| Ownership | Yes | Yes | No | No |
| Concurrency | Channels | Async/Threads | Goroutines | Threads |
| Type System | Explicit Emoji | Inferred | Inferred | Dynamic |
| OOP | No | Traits | Interfaces | Classes |
| Binary Size | Tiny | Medium | Large | N/A |

### When to Use Cymple

✅ **Good for:**
- Real-time systems
- Embedded systems
- Systems with strict memory constraints
- Applications requiring predictable performance
- Learning programming (visual symbols)
- Internationalization (no English keywords)

❌ **Not ideal for:**
- Large OOP codebases
- Rapid prototyping (currently no implementation)
- Projects requiring mature ecosystem
- Teams unfamiliar with move semantics

---

## Version History

- **v1.3** (2025-12-02) - String interpolation, else symbol, property access
- **v1.2 FINAL** (2025-11-28) - Complete specification, production-ready
- **v1.1** - Earlier development version
- **v1.0** - Initial specification

---

## License

MIT License

Copyright (c) 2025 Jörg Burbach

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

---

## Contact & Community

- **Website**: https://joerg-burbach.de/projekte/cymple
- **GitHub**: https://github.com/foodsnacker/cymple
- **Author**: Jörg Burbach

---

## Acknowledgments

This language specification was developed with assistance from:
- **ChatGPT** (OpenAI) - Interactive language design and iteration
- **Claude** (Anthropic) - Documentation, debugging, and refinement

---

**Thank you for your interest in Cymple!**

We hope this specification inspires implementations and contributes to the evolution of programming language design.

---

*Cymple v1.3 - December 2, 2025*
