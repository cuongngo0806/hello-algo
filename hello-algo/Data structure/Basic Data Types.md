---
tags:
  - chapter/data-structure
status: done
date: 2026-08-07
---

# Basic Data Types

## Core idea
A **basic data type** is a type the CPU's hardware can operate on directly, without
any library or compiler trick translating it into something simpler first. There are
four families you'll use constantly:

- **Integer types**: `byte`, `short`, `int`, `long` — whole numbers, no fraction.
- **Floating-point types**: `float`, `double` — numbers with a fractional part.
- **Character type**: `char` — a single letter, digit, punctuation mark, or symbol.
- **Boolean type**: `bool` — just `true` or `false`.

### Why size determines range — worked example

Every basic type is stored as pure binary. A **bit** is one binary digit (0 or 1); 8
bits make **1 byte** on essentially every modern system. The number of distinct
values a type can represent is `2^(number of bits)` — because each bit doubles how
many unique patterns you can form.

Let's actually count this for `byte` (1 byte = 8 bits):
- With 1 bit, you can form 2 patterns: `0`, `1`.
- With 2 bits: `00, 01, 10, 11` → 4 patterns.
- Each extra bit doubles the count. So with 8 bits: `2^8 = 256` distinct patterns.

Those 256 patterns get assigned to represent numbers. If `byte` were unsigned,
they'd map straightforwardly to `0` through `255`. Because `byte` is signed in most
languages, roughly half the patterns represent negatives and half represent
non-negatives, giving the range **−128 to 127** (256 total values — the exact
mechanics of *why* it's −128 to 127 and not −127 to 128 is covered in
[[Number Encoding]]).

Now compare `int` (4 bytes = 32 bits): `2^32` ≈ 4.29 billion distinct values — because
you have 32 bits instead of 8, and each additional bit *doubles* the count, going
from 8 to 32 bits multiplies the count by `2^24` (over 16 million times more values),
not just 4x. This is the core intuition: **doubling the byte count doesn't double
the range — it squares it (roughly)**, because range grows exponentially with bit
count, not linearly.

### Reference table (Java's fixed sizes)

| Type | Symbol | Space | Minimum | Maximum | Default |
|---|---|---|---|---|---|
| Integer | `byte` | 1 byte | −2⁷ (−128) | 2⁷−1 (127) | 0 |
| | `short` | 2 bytes | −2¹⁵ | 2¹⁵−1 | 0 |
| | `int` | 4 bytes | −2³¹ | 2³¹−1 | 0 |
| | `long` | 8 bytes | −2⁶³ | 2⁶³−1 | 0 |
| Float | `float` | 4 bytes | ~1.175×10⁻³⁸ | ~3.403×10³⁸ | 0.0f |
| | `double` | 8 bytes | ~2.225×10⁻³⁰⁸ | ~1.798×10³⁰⁸ | 0.0 |
| Character | `char` | 2 bytes | 0 | 2¹⁶−1 | 0 |
| Boolean | `bool` | 1 byte | false | true | false |

### The C++ portability gotcha — worked example

This table is **Java-specific**, because Java standardizes exact sizes across every
platform. **C and C++ deliberately do not** — sizes are implementation-defined,
which means the *same source code* can behave differently on different machines.

Concrete example: on 64-bit Linux/macOS (the LP64 data model), `long` is 8 bytes. On
64-bit Windows, `long` is only 4 bytes (same as `int`!). So this code:

```cpp
long x = 5000000000L;  // needs more than 32 bits
std::cout << x << std::endl;
```

prints the correct large value on Linux/macOS, but **silently overflows/truncates**
on Windows, because Windows's `long` can't hold a number that big. This is a real,
well-known cross-platform bug source — the fix in portable code is to use
`int64_t`/`uint64_t` from `<cstdint>` instead of `long`, since those fixed-width
types guarantee the same size everywhere.

Other platform notes worth remembering:
- `char` is 1 byte in C/C++ (not 2 like Java) — this matches ASCII/UTF-8's 1-byte
  minimum unit (see [[Character Encoding]]).
- `bool` only logically needs 1 bit, but is stored as a full byte, because a CPU's
  smallest *addressable* unit of memory is a byte — you cannot point to "just bit 3"
  of a byte in memory, only to the whole byte.

### Basic data types vs. data structures — the relationship

> **Basic data types provide the "content" — data structures provide the
> "organization."**

These two concepts are completely independent (orthogonal) — a data structure's job
is to arrange elements (in sequence, in a tree, in a graph...), and it doesn't care
what those elements actually *are*.

**Worked example**: consider an array of 5 slots. The exact same "5 contiguous
slots" organization can hold any basic type:

```cpp
// Initialize arrays using various basic data types
int numbers[5];      // 5 slots, each 4 bytes, holding integers
float decimals[5];   // 5 slots, each 4 bytes, holding decimals
char characters[5];  // 5 slots, each 1 byte, holding characters
bool bools[5];        // 5 slots, each 1 byte, holding booleans
```

In every case, the *array itself* — its layout, its indexing rule
`base + i * element_size`, its behavior — is identical. Only what's stored inside
each slot differs. This same split applies to every data structure you'll learn
later: a linked list node, a stack, a tree node, a hash map bucket — all of them
define *how* elements connect, while the basic type defines *what* an individual
element actually holds.

## Complexity
N/A — this section is conceptual, no algorithm/complexity to analyze.

## Code
No C++ sample file for this chapter (`codes/cpp/chapter_data_structure/` doesn't
exist — Data Structure is conceptual only).

## Related
- [[Classification of Data Structure]]
- [[Number Encoding]]
- [[Character Encoding]]

## Self-check questions
1. Why does going from `byte` (1 byte) to `int` (4 bytes) multiply the range by far
   more than 4x?
2. Give a concrete example of a bug that could occur from assuming `long` is always
   8 bytes in portable C++ code.
3. Why is `bool` stored as a full byte even though 1 bit would be logically enough?
4. In your own words: what does it mean for basic data types and data structures to
   be "orthogonal"?
