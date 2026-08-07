---
tags:
  - chapter/data-structure
status: done
date: 2026-08-07
---

# Character Encoding

## Core idea
A character, like any data, ultimately must be stored as binary. A **character
set** is the table that defines the mapping: which binary number represents which
character. This section traces the actual historical evolution of character sets —
each new one exists specifically to fix a limitation of the one before it, which is
the easiest way to understand *why* things are the way they are.

### Step 1 — ASCII: the original, English-only set

Diagram: **`ascii_table.png`** shows the complete 128-entry table.

ASCII uses only **7 bits**, giving `2^7 = 128` possible characters — enough for
uppercase/lowercase English letters, digits `0`-`9`, common punctuation, and a
handful of control characters (like newline `LF` = decimal 10, tab `HT` = decimal 9).

Worked example straight from the table: the letter `'A'` is entry decimal 65, binary
`0100 0001`. The letter `'a'` (lowercase) is decimal 97, binary `0110 0001` — notice
it's exactly `32` more than `'A'`, i.e. bit 5 (`0010 0000`) flips on. This
"uppercase/lowercase differ by one bit" pattern is why `toupper()`/`tolower()` can
be implemented as a simple bit flip in ASCII-based systems.

Problem: 128 slots is nowhere near enough to also cover accented European letters,
Cyrillic, Chinese, Arabic, etc. **ASCII can only represent English.**

### Step 2 — EASCII: use the 8th bit

EASCII extends to the full **8 bits** = `2^8 = 256` characters. The first 128 stay
identical to ASCII (so anything valid in ASCII is still valid in EASCII — this
backward-compatibility idea will come back with UTF-8). The extra 128 slots (128-255)
were defined *differently* by different regional standards — e.g. one EASCII variant
for Western Europe, another for Cyrillic, etc.

Problem: this only buys you 128 *extra* characters, which is still nowhere near
enough for languages like Chinese (tens of thousands of characters), and worse — if
two systems use *different* EASCII variants, byte 200 might mean one letter on
system A and a completely different letter on system B. Exchange a file between
them and you get garbled text.

### Step 3 — GB2312 / GBK: solve it for one language

China's GB2312 (1980) used **two bytes** for Chinese characters, covering 6,763
characters — enough for everyday use. GBK later extended this to 21,886 characters
(including rare/traditional characters). The scheme: ASCII characters still use 1
byte (compatible with plain English text), Chinese characters use 2 bytes.

Problem: this is still a **single-language solution**. Every other language needing
non-English characters needed its own separate standard, and mixing two languages'
custom byte-mappings in the same document is exactly the mess that led to the next
step.

### Step 4 — Unicode: one universal numbering scheme, but no storage format

The insight: instead of many incompatible per-language character sets, define **one**
character set covering every language and symbol on Earth, so cross-language text
never garbles. That's Unicode — as of 2022 it assigns **~149,186 characters**
(covering essentially every living language, plus symbols, plus emoji), each given a
unique **code point** in the range `U+0000` to `U+10FFFF`.

Worked example: `'H'` is Unicode code point `U+0048` (decimal 72 — notice this
matches ASCII exactly, since Unicode's first 128 code points *are* ASCII by design).
`'算'` (a Chinese character) is code point `U+7B97`.

But here's the catch Unicode deliberately leaves open: **it only assigns numbers, it
does not specify how to store those numbers as actual bytes in memory or on disk.**
If code points can be anywhere from 1 to 6 hex digits, how does a program parsing a
byte stream know where one character ends and the next begins?

### Step 5 — the naive fix: fixed-length padding

Diagram: **`unicode_hello_algo.png`** shows this approach applied to "Hello 算法".

The simplest fix: pick a fixed width (say, 2 bytes) and pad every character's code
point up to that width with leading zeros. `'H'` (code point 72, binary
`0100 1000`) becomes `0000 0000 0100 1000` — 2 full bytes, even though only 7 bits
of actual information were needed. `'算'` (code point needing more bits) also
becomes exactly 2 bytes. Now every character is uniform width, so a parser can just
read 2 bytes at a time, no ambiguity.

Problem: this is wasteful for ASCII-heavy text. Plain English text that used to take
1 byte per character under ASCII now takes 2 bytes per character — **doubling the
size of ordinary English documents** for no real benefit, since English never
needed those extra bits in the first place.

### Step 6 — UTF-8: variable length, the actual winner

Diagram: **`utf-8_hello_algo.png`** shows this applied to the same "Hello 算法".

UTF-8 uses **1 to 4 bytes per character**, chosen based on how large the code point
is: ASCII characters (code points 0-127) get exactly **1 byte** — meaning UTF-8 is
**fully backward-compatible with plain ASCII text**, byte-for-byte. Latin/Greek
extended characters typically need 2 bytes. Common Chinese/Japanese/Korean
characters typically need 3 bytes. Rare characters and most emoji need 4 bytes.

**The encoding trick — how does a parser know how many bytes a character uses?**
Look at the worked example from the diagram: the character `'算'` becomes the
3-byte sequence `1110 0111  1010 1110  1001 0111`.
- The **first byte** starts with three `1`s followed by a `0`: `1110...`. Counting
  those leading 1s (three of them) tells the parser "this character is 3 bytes long."
- Every **continuation byte** (the 2nd, 3rd, ... bytes of a multi-byte character)
  starts with `10`: notice both `1010 1110` and `1001 0111` above start with `10`.

**Why `10` specifically, as a "checksum"?** By UTF-8's own rules, it's
*mathematically impossible* for a valid character's first byte to start with `10` —
a 1-byte (ASCII) character's first byte must start with `0`, and any multi-byte
character's first byte must start with two or more `1`s. So if a parser starts
reading in the middle of a multi-byte sequence by mistake, the very next byte it
sees will start with `10` where it *expected* a proper leading byte — a signal
something's wrong, letting the parser detect and recover from corruption or
misalignment.

### The alternatives, and the real trade-off

- **UTF-16**: 2 or 4 bytes per character (2 bytes covers the vast majority of
  characters in use, including common CJK characters — more compact than UTF-8's 3
  bytes for those). Used internally by Java, JavaScript, and C#.
- **UTF-32**: always exactly 4 bytes per character, no exceptions. Simplest to
  reason about, but the most wasteful — even plain `'A'` costs 4 bytes.

**Worked comparison — indexing the i-th character**:
- Under UTF-32 (fixed 4 bytes always): the address of character `i` is simply
  `base + i * 4` — direct arithmetic, **O(1)**.
- Under UTF-8 (variable 1-4 bytes): there is no formula, because you don't know how
  wide characters 0 through i-1 were without reading them. You must start at byte 0
  and walk forward, decoding each character's length as you go, until you've passed
  `i` characters — **O(n)** in the worst case.

Same logic applies to computing a string's total character count: trivial arithmetic
under a fixed-width scheme, but requires a full traversal under UTF-8.

**C++ practical consequence**: `std::string` is just a raw sequence of bytes with no
enforced encoding baked into the type. If a `std::string` holds UTF-8 text
containing, say, the character `'算'` (3 bytes), then `.size()` returns the **byte
count**, not the character count — calling `.size()` on a string containing one
Chinese character and getting back `3` (not `1`) is a very common source of
off-by-N bugs when working with internationalized text in C++.

## Complexity
- Random access to the i-th character, and total character count: **O(1)** under
  fixed-width encodings (UTF-16 for BMP characters, UTF-32 always).
- Same operations: **O(n)** under UTF-8's variable-length encoding — must traverse
  from the start, decoding lengths along the way.

## Code
No C++ sample for this chapter (`codes/cpp/chapter_data_structure/` doesn't exist).

Diagrams (from `en/docs/chapter_data_structure/character_encoding.assets/`):
- `ascii_table.png` — full 128-character ASCII table (Oct/Binary/Char/Name columns)
- `unicode_hello_algo.png` — naive fixed-length (2-byte) Unicode padding, "Hello 算法"
- `utf-8_hello_algo.png` — UTF-8 variable-length encoding of the same string, showing
  the leading-1s length signal and `10` continuation-byte marker

## Related
- [[Number Encoding]]
- [[Basic Data Types]]

## Self-check questions
1. Why is UTF-8 backward-compatible with ASCII, byte-for-byte?
2. A UTF-8 byte starts with `1110`. How many total bytes does this character occupy,
   and how do you know?
3. If a UTF-8 parser starts reading from the wrong byte offset (misaligned mid a
   multi-byte character), how does it detect the error?
4. Why does `std::string::size()` NOT equal the number of visible characters when
   the string contains non-ASCII text?
5. Give one concrete reason UTF-16 might be preferred over UTF-8 for a
   Chinese-heavy document, and one reason UTF-8 might be preferred for an
   English-heavy document.
