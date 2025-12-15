# CYMPLE Programming Language

**Version 1.4** - Procedural Programming with Unicode Symbols

---

## What is Cymple?

Cymple is a procedural programming language that uses Unicode symbols (emojis) instead of keywords. It combines modern safety features with classical simplicity, making concurrent programming accessible without sacrificing performance or correctness.

```cymple
🧵 fibonacci(n: 🔢) -> 🔢
    ❓ n <= 1
        ↩ n
    
    🌀📦 📋results ← [fibonacci(n - 1), fibonacci(n - 2)]
        ✅ 📋done
            ↩ 📋done[1] + 📋done[2]

💬 "Fibonacci(10) = fibonacci(10)"
```

---

## Key Features

### 🎨 **Visual & Accessible**
- Unicode symbols replace keywords
- Clear indentation-based structure
- No English keywords = internationally neutral
- Type prefixes make variables instantly recognizable

### 🔒 **Memory Safe**
- Exclusive ownership and move semantics
- RAII for deterministic resource management
- No garbage collection pauses
- No data races by design

### ⚡ **Powerful Concurrency**
- Share-nothing task model
- Channels for safe communication
- **Quantum Operations** for easy parallel programming
- Preemptive multitasking

### 🚀 **Simple Yet Powerful**
- Procedural, no OOP complexity
- No anonymous functions or closures
- Pattern matching with guards
- FFI for C library integration

---

## What's New in Version 1.4

### ⏱️ **Timeouts with Units**
```cymple
🌀⚡ 🔤result ← [fetch_eu(), fetch_us()]
    ⏱️ 3s  📝 Clear and readable!
        💬 "Timeout"
```

### ⏩ **Progress Frequency Control**
```cymple
🌀📦 📋results ← [100 tasks...]
    ⏩ 📋partial every 10  📝 50-90% less overhead
        💬 "Progress: 📋partial.length%"
```

### 🛑 **Early Cancellation**
```cymple
⏩ 📋partial
    ❓ 📋partial.length >= 20
        🛑  📝 Stop when you have enough!
        ↩ 📋partial
```

### ❌ **Total Failure Detection**
```cymple
❌ 🔤total_failure  📝 Only fires if ALL tasks fail
    💬 "Everything failed"
    ↩ fallback()
```

[See full changelog](cymple_1_4_changelog.md)

---

## Quick Start

### Hello World
```cymple
🔤name ← "World"
💬 "Hello, 🔤name!"
```

### Variables & Types
```cymple
🔢count ← 42          📝 Number
🔤text ← "Hello"      📝 String
✅ is_valid ← ✅       📝 Bool (true)
📋list ← [1, 2, 3]    📝 List
🗺️map ← {"a": 1}      📝 Map
```

### Control Flow
```cymple
❓ age >= 18
    💬 "Adult"
⤵️
    💬 "Minor"

🔁 i in 1..10
    💬 "Number: 🔢i"
```

### Functions
```cymple
🧵 add(a: 🔢, b: 🔢) -> 🔢
    ↩ a + b

🧵 greet(name: 🔤)  📝 No return type needed
    💬 "Hello, 🔤name!"
```

### Parallel Computing
```cymple
📝 Race - First result wins
🌀⚡ 🔤fastest ← [fetch_eu(), fetch_us(), fetch_asia()]
    ⏱️ 3s
        ↩ cached_value()
    ✅ 🔤winner
        ↩ 🔤winner

📝 Collect - Gather all results
🌀📦 📋results ← [process1(), process2(), process3()]
    ⏩ 📋partial every 5
        💬 "Progress: 📋partial.length"
    ✅ 📋all
        ↩ filter_ok(📋all)
```

---

## Why Cymple?

### Comparison with Other Languages

**Cymple vs JavaScript:**
```javascript
// JavaScript (40+ lines)
async function searchAll(query) {
    const searches = [
        searchGoogle(query),
        searchBing(query),
        searchDuck(query)
    ];
    
    const timeout = new Promise((_, reject) =>
        setTimeout(() => reject('Timeout'), 5000)
    );
    
    try {
        const results = await Promise.race([
            Promise.allSettled(searches),
            timeout
        ]);
        
        return results
            .filter(r => r.status === 'fulfilled')
            .map(r => r.value);
    } catch (error) {
        return [];
    }
}
```

**Cymple (15 lines):**
```cymple
🧵 search_all(🔤query) -> 📋
    🌀📦 📋results ← [
        search_google(🔤query),
        search_bing(🔤query),
        search_duck(🔤query)
    ]
        ⏱️ 5s
            ↩ 📋results
        ⏩ 📋partial
            ❓ 📋partial.length >= 20
                🛑
                ↩ 📋partial
        ✅ 📋all
            ↩ filter_ok(📋all)
```

**3-4x shorter, clearer intent, better control!**

---

## Core Concepts

### Memory Safety Without GC

```cymple
📝 Handles use RAII - cleanup at block end
💾file ← open("data.txt")
🔗 file -> F
    📋data ← F.read()
    💬 "Read: 📋data.length bytes"
📝 file automatically closed here
```

### Move Semantics

```cymple
📋list1 ← [1, 2, 3]
📋list2 ← list1  📝 list1 is now null (moved)
```

### Borrowing

```cymple
📋list ← [1, 2, 3]
🔗 list -> L  📝 Read-only borrow
    💬 L[1]

🔗 list -> mut L  📝 Mutable borrow
    L[1] ← 99
    L.push(4)
```

### Pattern Matching

```cymple
🔀 value
    ➜ 0
        💬 "Zero"
    ➜ 1..10
        💬 "Small"
    ➜ x ❓ x > 100
        💬 "Large: 🔢x"
    ➜ _
        💬 "Other"
```

### Channels

```cymple
📡ch ← 🛰️🔢  📝 Create number channel

🧵 sender(📡out)
    🔁 i in 1..10
        🚀 out, i

🧵 receiver(📡in)
    🔁 msg in in
        💬 "Received: 🔢msg"
```

---

## Quantum Operations (v1.4)

Cymple's killer feature for parallel programming.

### Race - First Wins

```cymple
🌀⚡ 🔤result ← [task1(), task2(), task3()]
    ⏱️ 5s
        💬 "Timeout"
        ↩ fallback()
    ✅ 🔤winner
        ↩ 🔤winner
    ❌ 🔤error
        🧘 🔤error
            ↩ default()
```

**Use cases:**
- Multi-region API calls
- Fallback strategies
- Latency optimization

### Collect - Gather All

```cymple
🌀📦 📋results ← [task1(), task2(), task3()]
    ⏱️ 30s
        ↩ 📋results  📝 Partial results
    
    ⏩ 📋partial every 10
        🔢percent ← (📋partial.length * 100) / 🔢total
        💬 "Progress: 🔢percent%"
        
        📝 Early cancellation
        ❓ 📋partial.length >= 50
            🛑
            ↩ 📋partial
    
    ✅ 📋all
        📋ok ← filter_ok(📋all)
        ↩ 📋ok
    
    ❌ 🔤total_failure
        💬 "All failed"
        ↩ []
```

**Use cases:**
- Batch processing
- MapReduce operations
- Parallel downloads
- Data aggregation

---

## Documentation

- [**Specification**](cymple_spec_1_4.md) - Complete language reference
- [**EBNF Grammar**](cymple_ebnf_1_4.txt) - Formal syntax definition
- [**Changelog**](cymple_1_4_changelog.md) - What's new in 1.4
- [**Version Comparison**](cymple_version_comparison.md) - 1.2 vs 1.3 vs 1.4

---

## Design Philosophy

### Simplicity Over Abstraction

- No generics
- No anonymous functions
- No classes or OOP
- Top-level functions only

**Why?** Predictable behavior, easy to learn, fast compilation.

### Safety Without Runtime Cost

- Compile-time ownership checking
- No garbage collection
- RAII for deterministic cleanup
- Share-nothing concurrency

**Why?** Real-time performance, embedded systems, low latency.

### Readability Through Symbols

- Visual distinction (🔢 vs `number`)
- Less verbose
- Language-neutral (no English)
- Accessible internationally

**Why?** Universal understanding, reduced cognitive load.

---

## Symbol Reference

### Basic
- `←` Assignment
- `↩` Return
- `❓` If
- `⤵️` Else
- `🔁` Loop
- `📝` Comment

### Types
- `🔢` Number
- `🔤` String
- `✅` / `✗` Bool
- `📋` List
- `🗺️` Map
- `🔣` Bytearray

### Concurrency
- `🧵` Function/Task
- `📡` Channel
- `🛰️` Create channel
- `🚀` Send
- `🎯` Receive
- `🛑` Stop
- `🌀⚡` Race
- `🌀📦` Collect

### Events
- `✅` Success
- `❌` Error
- `⏩` Progress
- `⏱️` Timeout

[See full symbol reference in specification](cymple_spec_1_4.md#symbol-reference)

---

## Examples

### Multi-Server Search
```cymple
🧵 fetch_best(🔤query) -> 🔤
    🌀⚡ 🔤result ← [
        search_server_eu(🔤query),
        search_server_us(🔤query),
        search_server_asia(🔤query)
    ]
        ⏱️ 3s
            ↩ cached_search(🔤query)
        ✅ 🔤fastest
            ↩ 🔤fastest
```

### Parallel Image Processing
```cymple
🧵 process_images(📋files) -> 📋
    🌀📦 📋results ← split_and_process(📋files)
        ⏱️ 5m
            💬 "Timeout - 📋results.length done"
            ↩ 📋results
        
        ⏩ 📋partial every 10
            🔢percent ← (📋partial.length * 100) / 📋files.length
            🖼️update_progress(🔢percent)
        
        ✅ 📋all
            ↩ filter_ok(📋all)
```

### Error Handling with Guru
```cymple
🧵 process_file(🔤path)
    🧘 guru(e)
        🔀 e.code
            ➜ 404
                💬 "⚠️ File not found: 🔤path"
                ↩
            ➜ 403
                ❌ Unauthorized() -> 403
            ➜ _
                💀 e
    
    💾f ← open(🔤path)
    🔗 f -> F
        📋data ← F.read()
        💬 "Processed: 🔢data.length bytes"
```

---

## Performance

### Cymple vs Other Languages

| Metric | Cymple | JavaScript | Rust | Go | Python |
|--------|--------|------------|------|-----|---------|
| **Lines of code** | **15** | 40+ | 35+ | 60+ | 30+ |
| **Progress events** | ✅ Built-in | ❌ Manual | ❌ Manual | ⚠️ Complex | ❌ Manual |
| **Early cancel** | ✅ Trivial | ❌ Complex | ⚠️ Complex | ⚠️ Very complex | ⚠️ Complex |
| **Timeout** | ✅ `⏱️ 5s` | ⚠️ Wrapper | ✅ Built-in | ✅ Built-in | ✅ Built-in |
| **Memory safety** | ✅ Compile-time | ❌ Runtime | ✅ Compile-time | ⚠️ Runtime | ❌ Runtime |
| **GC pauses** | ✅ None | ❌ Yes | ✅ None | ❌ Yes | ❌ Yes |

[See detailed benchmarks in changelog](cymple_1_4_changelog.md#performance-impact)

---

## Platform Support

### Target Platforms
- Linux (primary)
- macOS
- Windows
- Embedded systems
- WebAssembly (planned)

### Compilation
- Interpreted mode for development
- Compiled mode for production
- Identical semantics in both modes
- Small binaries (30-100 KB)

---

## Getting Started

### Installation

*Coming soon - compiler under development*

### Learning Resources

1. Read the [Quick Start](#quick-start) section above
2. Study the [Complete Specification](cymple_spec_1_4.md)
3. Review [Example Code](#examples)
4. Check the [Changelog](cymple_1_4_changelog.md) for latest features

---

## Project Status

**Version:** 1.4 (December 4, 2025)
**Status:** Language specification complete, compiler in development

### Completed
✅ Language specification v1.4
✅ EBNF grammar
✅ Core features defined
✅ Quantum operations enhanced
✅ Documentation complete

### In Progress
🔄 Reference compiler implementation
🔄 Standard library
🔄 Tooling (formatter, linter)

### Planned
📋 IDE support (VS Code, etc.)
📋 Package manager
📋 Testing framework
📋 WebAssembly target

---

## Contributing

Cymple is currently in specification phase. Feedback welcome!

- **Language Design:** Suggest improvements
- **Examples:** Share use cases
- **Documentation:** Report unclear sections
- **Implementation:** Join compiler development (coming soon)

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

## Contact

- **Website**: https://cymple.dev
- **GitHub**: https://github.com/foodsnacker/cymple
- **Author**: Jörg Burbach

---

## Acknowledgments

Cymple draws inspiration from:
- **Rust** - Ownership and borrowing
- **Go** - Simplicity and channels
- **Erlang** - Share-nothing concurrency
- **QOI** - Simplicity-first design philosophy
- and some more

---

**Cymple 1.4** - Making parallel programming simple again.

*© 2025 Jörg Burbach*
