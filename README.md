# 🎨 CYMPLE

**A Procedural Programming Language with Unicode Symbols**

[![Version](https://img.shields.io/badge/version-1.2%20FINAL-blue)](https://github.com/yourusername/cymple)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)
[![Status](https://img.shields.io/badge/status-specification-orange)](https://cymple.dev)

---

## Overview

Cymple is a research programming language that explores novel approaches to procedural programming through Unicode symbols, share-nothing concurrency, and deterministic memory management without garbage collection.

**Key Features:**
- 🔣 Unicode symbols instead of keywords
- 🚀 Zero GC pauses (pure RAII)
- 🔐 Share-nothing concurrency by design
- ⚡ Built-in parallel evaluation patterns
- 🎯 Pattern matching with guards
- 🔗 C library FFI support
- 📦 Dual-mode: interpreted OR compiled

---

## Quick Example

```cymple
🧵 fib(n: 🔢) -> 🔢
    ❓ n <= 1
        ↩ n
    ↩ fib(n - 1) + fib(n - 2)

📝 Parallel execution
🌀📦 results ← [
    fib(10),
    fib(15),
    fib(20)
]
    ✅ done
        💬 "Results: " + done
```

---

## Documentation

### Core Documents

- **[Language Specification v1.2 FINAL](cymple_spec_1_2_FINAL.md)** - Complete language reference
- **[EBNF Grammar](cymple_ebnf_1_2_FINAL.txt)** - Formal syntax specification
- **[Changelog](CYMPLE_1_2_CHANGELOG.md)** - Version history and design decisions

### Analysis & Comparison

- **[Language Comparison](cymple_comparison.md)** - Cymple vs Rust/Go/Python/JavaScript
- **[Benchmark Scenarios](cymple_benchmarks.md)** - Theoretical performance analysis

---

## Language Features

### Pattern Matching

```cymple
🔀 value
    ➜ 0
        💬 "Zero"
    ➜ 1..10
        💬 "Small"
    ➜ x ❓ x > 100
        💬 "Large"
    ➜ _
        💬 "Default"
```

### Quantum Operations

**Race Mode** - First result wins:
```cymple
🌀⚡ result ← [
    fetch_server_eu(),
    fetch_server_us(),
    fetch_server_asia()
]
    ✅ winner
        💬 winner
```

**Collect Mode** - Wait for all:
```cymple
🌀📦 results ← [
    process_chunk_1(),
    process_chunk_2(),
    process_chunk_3()
]
    ✅ done
        💬 "All done: " + done
```

### Borrowing

```cymple
📋list ← [1, 2, 3]

🔗 list -> L
    💬 L[1]  📝 Read-only borrow

🔗 list -> mut L
    L[1] ← 99  📝 Mutable borrow
```

### FFI (Foreign Function Interface)

```cymple
🔗 "libc.so.6"
    🧵 strlen(s: *🔤) -> 🔢

len ← strlen("hello")
💬 len  📝 5
```

---

## Design Philosophy

### What Cymple IS

✅ **Safety without complexity** - Simpler than Rust, safer than Go  
✅ **Deterministic** - No GC pauses, predictable performance  
✅ **Concurrent by design** - Share-nothing message passing  
✅ **Visual** - Symbols make code structure obvious  
✅ **Educational** - Exploring language design concepts

### What Cymple is NOT

❌ **Production-ready** - This is a specification, not an implementation  
❌ **Object-oriented** - Pure procedural, no classes  
❌ **Generic** - No generics (by design)  
❌ **For everyone** - Symbols may alienate traditional programmers

---

## Memory Model

Cymple uses **RAII** (Resource Acquisition Is Initialization) for deterministic cleanup:

- ✅ **No garbage collection** - Zero GC pauses
- ✅ **Predictable** - Resources freed at block end
- ✅ **Real-time suitable** - Deterministic latency
- ✅ **Move semantics** - Explicit ownership transfer

Each task has its own isolated heap. Communication happens only through channels.

---

## Performance Profile (Theoretical)

| Metric | Cymple | vs Rust | vs Go | vs Python |
|--------|--------|---------|-------|-----------|
| Single-threaded | 0.6-0.8× C | 0.7-0.9× | 0.8-1.0× | 8-15× faster |
| Parallel (4 cores) | 0.7-0.85× C | 0.8-0.95× | 0.85-1.0× | 3-8× faster |
| GC pauses | **ZERO** | **ZERO** | 1-10ms | 10-100ms |
| Startup (interp) | 5-10ms | N/A | N/A | 2-3× faster |
| Startup (comp) | <1ms | ~1× | ~1× | 40-60× faster |

See [benchmarks.md](cymple_benchmarks.md) for detailed analysis.

---

## Symbol Reference

### Quick Lookup

```
📝 Comment          🔢 Number          ← Assignment       = Equality
❓ If               ↩ Return           🔁 Loop            🧵 Function
🧱 Struct           📋 List            🗺️ Map             🔣 Bytearray
📡 Channel          🛰️ Create          🚀 Send            🎯 Receive
🌀⚡ Race            🌀📦 Collect        🔗 Borrow          🗑️ Free
🔀 Match            ➜ Arm              _ Wildcard         ⏩ Step
🧩 Module           🛠️ Plugin          🧘 Guru            ❌ Error
```

See [specification](cymple_spec_1_2_FINAL.md#symbol-reference) for complete list.

---

## Use Cases

### ✅ Ideal For

- Educational systems programming
- Console applications with rich output
- Concurrent data processing pipelines
- Real-time systems (no GC pauses)
- Microservices
- Language design research

### ❌ Not Ideal For (Yet)

- Production systems (no implementation exists)
- Large teams (no mature tooling)
- Rapid prototyping (no ecosystem)
- Projects needing existing libraries NOW

---

## Current Status

**Phase:** Complete Language Specification  
**Version:** 1.2 FINAL (2025.11.28)  
**Next:** Seeking implementers for interpreter/compiler

### Implementation Roadmap

1. **Phase 1:** Core Interpreter
   - Lexer/Parser (Unicode-aware)
   - Basic type system
   - RAII memory management

2. **Phase 2:** Compiler
   - LLVM backend
   - Optimization passes
   - Static analysis

3. **Phase 3:** Standard Library
   - File I/O
   - String operations
   - Network primitives

4. **Phase 4:** Tooling
   - IDE plugins
   - Debugger
   - Package manager

---

## Contributing

Cymple is a **specification project**. Contributions welcome:

- 🐛 **Specification issues** - Found ambiguities or errors?
- 💡 **Design suggestions** - Ideas for language improvements?
- 🛠️ **Implementation** - Want to build an interpreter/compiler?
- 📚 **Examples** - Write Cymple programs to test the spec?

Open an issue or submit a PR!

---

## Repository Structure

```
cymple/
├── cymple_spec_1_2_FINAL.md       # Main specification
├── cymple_ebnf_1_2_FINAL.txt      # Formal grammar
├── CYMPLE_1_2_CHANGELOG.md        # Version history
├── cymple_comparison.md           # Language comparisons
├── cymple_benchmarks.md           # Performance analysis
├── index.html                     # Landing page
├── examples/                      # Code examples
│   ├── fibonacci.cymple
│   ├── web_server.cymple
│   └── parallel_processing.cymple
└── README.md                      # This file
```

---

## License

MIT License - See [LICENSE](LICENSE) for details.

The language specification is freely available for anyone to implement.

---

## Author

**Jörg Burbach**  
🌐 [joerg-burbach.de](https://joerg-burbach.de)  
📧 [Contact](mailto:your@email.com)

---

## Acknowledgments

- Inspired by QOA Format, Rust, Go, and Erlang
- Thanks to the programming language design community
- Built with assistance from Claude (Anthropic)

---

## Links

- 🌐 [Website](https://cymple.dev)
- 📖 [Full Specification](cymple_spec_1_2_FINAL.md)
- 💬 [Discussions](https://github.com/yourusername/cymple/discussions)
- 🐛 [Issues](https://github.com/yourusername/cymple/issues)

---

**Cymple** - Exploring procedural programming through symbols 🎨
