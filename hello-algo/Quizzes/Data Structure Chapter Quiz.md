---
tags:
  - quiz/data-structure
status: done
date: 2026-08-07
---

# Data Structure Chapter Quiz

Covers: [[Classification of Data Structure]], [[Basic Data Types]],
[[Number Encoding]], [[Character Encoding]] (sections 3.1-3.4).

Result: **8/8 correct** on first attempt. Chapter marked `done` on 2026-08-07.

## Q1 — A binary search tree where each node has one parent but up to two children is an example of which logical structure category?

- **Non-linear, tree structure** — One-to-many relationship (one parent, multiple
  children) — exactly the tree-structure definition. ✅ *(user's answer, correct)*
- Non-linear, network structure — Network structures are many-to-many (e.g. graphs) — a
  tree's one-parent-per-node rule doesn't fit this.
- Linear — Linear means each element has at most one predecessor AND one successor — a
  tree node with two children breaks this.
- None of these — physical structure only — Logical structure classification does apply
  here; this isn't a physical-storage question.

## Q2 — Why can you compute the address of arr[i] directly in an array, but not the address of the i-th node in a linked list?

- **Array is contiguous (base + i\*size); linked list is dispersed, must traverse via
  pointers** — Correct — contiguous memory allows direct address arithmetic; dispersed
  memory requires following next-pointers one at a time. ✅ *(user's answer, correct)*
- Linked lists don't store any addresses at all — Linked lists absolutely store
  addresses/pointers — that's how nodes connect.
- Arrays are always slower because they need pointers too — Arrays don't need pointers
  between elements — that's the linked list's mechanism, not the array's.
- Both are O(1) access — no real difference — Array access is O(1); linked list access
  to the i-th node is O(n).

## Q3 — Hand-trace: compute 1 + (-2) using SIGN-MAGNITUDE 8-bit representation by directly adding the bit patterns. What do you get?

- **1000 0011 → interpreted as -3 (wrong answer)** — `0000 0001 + 1000 0010 = 1000 0011`
  — sign-magnitude breaks under naive bit addition, giving -3 instead of the correct -1.
  ✅ *(user's answer, correct — this IS the correct educational answer, demonstrating
  sign-magnitude's flaw)*
- 1111 1111 → interpreted as -1 (correct answer) — This would be the 2's complement
  result, not sign-magnitude's naive bit-addition result.
- 0000 0001 → interpreted as 1 (wrong answer) — Doesn't match actual bit addition of the
  two given patterns.
- Sign-magnitude addition always works correctly — This is exactly the flaw
  sign-magnitude has — it does NOT work correctly under plain bit addition.

## Q4 — Why does 2's complement end up with only ONE representation of zero, while sign-magnitude and 1's complement each have two (+0 and -0)?

- **Computing 2's complement of -0 overflows the extra carry bit off the end, collapsing
  it to the same pattern as +0** — 1's complement of -0 (`1111 1111`) + 1 =
  `1 0000 0000` — the 9th bit overflows off an 8-bit byte, leaving `0000 0000`, identical
  to +0. ✅ *(user's answer, correct)*
- 2's complement doesn't allow negative numbers at all — 2's complement absolutely
  represents negative numbers — that's its whole purpose.
- Computers just define -0 as illegal and refuse to store it — Not the actual mechanism
  — the collapse happens naturally from overflow, no special-casing needed.
- Sign-magnitude and 1's complement don't actually have two zeros either — They do —
  `0000 0000` and `1000 0000` are distinct patterns in both schemes.

## Q5 — Why does byte's range end up as [-128, 127] instead of the symmetric [-127, 127]?

- **The leftover bit pattern 1000 0000 (which would be a 2nd zero) is assigned to mean
  -128 by convention** — Correct — verified by computing (-127) + (-1) in 2's complement,
  which lands exactly on `1000 0000`. ✅ *(user's answer, correct)*
- byte actually has 257 possible values, not 256 — byte has exactly 2^8 = 256 values —
  the range asymmetry comes from reassigning a leftover pattern, not from extra values.
- -128 is a rounding error in the standard — Not a rounding error — it's a deliberate,
  mathematically consistent assignment (verified via addition).
- int has this issue too, not just byte — The question is about byte specifically —
  though the same -128-style asymmetry does apply proportionally to int and other signed
  integer types.

## Q6 — Why does float have a vastly bigger range than int despite both being 4 bytes?

- **float uses an exponent field that scales the value multiplicatively (like scientific
  notation), not just a flat bit count** — `val = (-1)^S × 2^(E-127) × (1+N)` — the
  exponent grows range multiplicatively, unlike int's flat linear bit count. ✅ *(user's
  answer, correct)*
- float actually uses more physical bits than int, just hidden from the programmer — Both
  are exactly 32 bits — float's larger range comes from how those bits are interpreted,
  not from extra hidden bits.
- float has no precision trade-off compared to int — float DOES sacrifice precision —
  gaps between representable values grow as magnitude grows, unlike int's uniform
  spacing.
- int can represent decimals too, so the comparison doesn't apply — int cannot represent
  decimals — that's precisely float's job, and irrelevant to the range-size question.

## Q7 — In UTF-8, a character's first byte is 1110 0111. How many total bytes does this character occupy, and how do you know?

- **3 bytes — count the leading 1s before the first 0 (three 1s = 3-byte character)** —
  Correct — UTF-8's encoding rule: the first byte's leading run of 1s (before a 0)
  signals the total character length. ✅ *(user's answer, correct)*
- 4 bytes — count all the 1s anywhere in the byte — You only count the LEADING run of 1s
  before the first 0, not all 1 bits scattered in the byte.
- 1 byte — this looks like plain ASCII — ASCII (1-byte) characters start with a leading
  0, not `1110…` — this byte signals a multi-byte character.
- Cannot be determined from the first byte alone — UTF-8 is specifically designed so the
  first byte alone determines the character's total length.

## Q8 — Why does std::string::size() on a C++ string containing the character '算' (encoded in UTF-8) return 3, not 1?

- **std::string is a raw byte sequence with no encoding awareness; '算' takes 3 bytes in
  UTF-8, and size() counts bytes** — Correct — size() returns byte count, not character
  count, because std::string has no concept of "character" under UTF-8. ✅ *(user's
  answer, correct)*
- '算' is actually 3 separate characters combined — '算' is a single Unicode
  character/code point — it just happens to require 3 bytes to encode in UTF-8.
- size() is buggy and should be avoided entirely — size() works exactly as documented
  (byte count) — it's not a bug, just a common source of confusion for UTF-8 text.
- This only happens with std::wstring, not std::string — The byte-vs-character mismatch
  is specifically a std::string + UTF-8 issue — std::wstring uses a different (usually
  fixed-width) encoding.

## Related
- [[Classification of Data Structure]]
- [[Basic Data Types]]
- [[Number Encoding]]
- [[Character Encoding]]
