---
tags:
  - chapter/data-structure
status: done
date: 2026-08-07
---

# Number Encoding

## Core idea
This section answers two genuinely counter-intuitive questions: why does `byte`
range from **−128 to 127** (one extra negative number, not symmetric), and why does
`float` have a vastly bigger range than `int` despite occupying the same 4 bytes?
Both answers come down to *how* numbers are actually represented in binary — not
just how many bits they use.

### Three ways to represent a negative number

Computers went through three attempts at representing negative numbers, each fixing
a flaw in the previous one. Diagram: **`1s_2s_complement.png`** shows the conversion
process for all three, worked through for `+0`, `+5`, `-0`, and `-28`.

**Attempt 1 — Sign-magnitude**: reserve the top (leftmost) bit as a sign flag (`0` =
positive, `1` = negative); the remaining bits store the magnitude, literally.

Worked example: represent `+5` and `-5` in an 8-bit `byte`.
- `+5` = `0000 0101` (sign bit `0`, magnitude `0000101` = 5)
- `-5` = `1000 0101` (sign bit `1`, same magnitude bits)

This is intuitive to read, but it **breaks ordinary binary addition**. Let's actually
try to compute `1 + (-2)` by just adding the bit patterns:
```
   0000 0001   (+1 in sign-magnitude)
 + 1000 0010   (-2 in sign-magnitude)
 -----------
   1000 0011   → interpreted as -3
```
The correct answer is `-1`, but naive bit-addition gives `-3`. Sign-magnitude values
can't be fed straight into an adder circuit — you'd need special-case logic to
detect signs and subtract instead of add, which is exactly the complexity computers
want to avoid.

There's a second flaw too: **both `+0` and `-0` exist** as distinct bit patterns —
`0000 0000` and `1000 0000` — which wastes an encoding and complicates
zero-comparisons (is `1000 0000 == 0`? You'd need extra logic to say yes).

**Attempt 2 — 1's complement**: keep positive numbers identical to sign-magnitude;
for negatives, invert every bit *except* the sign bit.

Worked example: convert `-5` (sign-magnitude `1000 0101`) to 1's complement: keep
the sign bit `1`, invert the rest (`000 0101` → `111 1010`), giving `1111 1010`.

Now retry `1 + (-2)` in 1's complement:
```
   0000 0001   (+1, same as sign-magnitude)
 + 1111 1101   (-2 in 1's complement: sign-magnitude 1000 0010 → invert → 1111 1101)
 -----------
   1111 1110   (1's complement result)
```
Converting `1111 1110` back to sign-magnitude (invert the non-sign bits again):
sign bit `1`, invert `111 1110` → `000 0001`, giving `1000 0001` = **−1**. Correct
this time! 1's complement fixes the addition problem.

But it **still has the two-zeros problem**: `+0` = `0000 0000`, and `-0` in 1's
complement = invert all non-sign bits of `1000 0000` → `1111 1111`. Still two
distinct patterns for zero.

**Attempt 3 — 2's complement** (what every computer actually uses): keep positives
identical; for negatives, take the 1's complement and **add 1**.

Worked example — watch what happens to negative zero specifically:
```
  -0 in sign-magnitude:    1000 0000
  -0 in 1's complement:    1111 1111   (invert non-sign bits)
  -0 in 2's complement:    1111 1111 + 1 = 1 0000 0000
```
That result needs **9 bits**, but a `byte` only has 8. The 9th bit (the carry)
simply **overflows off the end and is discarded**, leaving `0000 0000` — which is
exactly the same pattern as `+0`. **There is now only one zero.** The ambiguity is
gone, and no extra circuitry was needed to force it — it fell out naturally from
finite-width overflow.

### Where does the extra negative value (−128) come from?

Every value in `[-127, 127]` has a clean, unique sign-magnitude / 1's complement /
2's complement triplet, convertible in both directions. But one 2's complement
pattern is left over with no positive counterpart: `1000 0000`.

If you try to convert `1000 0000` back to sign-magnitude using the normal rule
(subtract 1, then invert), you get `0000 0000` — which is a contradiction, since
that's the pattern for zero, and zero's own 2's complement should just be itself,
not `1000 0000`.

Computers resolve this by **special-casing** `1000 0000` to mean **−128**. You can
verify this is consistent by computing `(-127) + (-1)` directly in 2's complement
and checking it lands on `1000 0000`:
```
  -127 in 2's complement:  1000 0001
    -1 in 2's complement:  1111 1111
  --------------------------------
  Sum (mod 256):           1000 0000   → -128 ✓
```
So `byte`'s range is `[-128, 127]`: 256 total patterns, but shifted by one because
`-128` "steals" what would otherwise be the second representation of zero.

### Why 2's complement, specifically?

Because it lets the *same addition circuit* correctly handle both positive and
negative numbers — no special-case subtraction hardware needed, since
`a - b` is just computed as `a + (-b)`. Addition is the operation hardware is built
around, because it's the simplest and fastest to implement in silicon and the
easiest to parallelize (across bit positions, for carry propagation). 2's complement
is the encoding choice that lets one circuit do everything.

### Floating-point: why is `float`'s range so much bigger than `int`'s?

Diagram: **`ieee_754_float.png`** shows the IEEE 754 layout for `float`'s 32 bits,
worked through with a concrete example.

A `float` doesn't store a plain binary number the way `int` does. It splits its 32
bits into three fields, working like binary scientific notation:
- **Sign (S)**: 1 bit — `0` positive, `1` negative.
- **Exponent (E)**: 8 bits — a scaling power of 2 (biased by 127, so it can also
  represent negative exponents).
- **Fraction (N)**: 23 bits — the mantissa's fractional part.

Formula: `val = (-1)^S × 2^(E-127) × (1 + N)`

Worked example straight from the diagram: given `S = 0`, `E = 124` (binary
`0111 1100`), `N = 2⁻² + 2⁻³ = 0.25 + 0.125 = 0.375`:
```
val = (-1)^0 × 2^(124-127) × (1 + 0.375)
    = 1 × 2^-3 × 1.375
    = 1 × 0.125 × 1.375
    = 0.171875
```
That's how a single 32-bit pattern encodes `0.171875`.

**Why the range explodes**: `int`'s 32 bits are one flat binary number — its
biggest possible value is bounded by `2^31 - 1` ≈ 2.1 billion. `float`'s exponent
field lets it *scale* its mantissa by powers of 2 up to `2^127` — so the maximum
representable value is roughly `2^254-127 × (2 - 2⁻²³) ≈ 3.4 × 10^38`, vastly larger,
because the exponent grows the value **multiplicatively**, not by adding more bits
to a flat count.

**The trade-off — precision**: `int`'s 32 bits are evenly spaced; every representable
integer is exactly 1 apart from its neighbor. `float`'s mantissa only has 23 bits
*regardless of the exponent*, so as the exponent grows, the gap between two adjacent
representable floats grows too. Near `1.0`, adjacent floats might differ by about
`2⁻²³` ≈ 0.0000001; near `1,000,000`, adjacent floats can differ by whole integers —
you literally cannot represent every integer near a million exactly as a `float`.
This is why comparing floats with `==` is a classic bug: two computations that
"should" produce the same mathematical result can land on different representable
values due to this uneven spacing plus rounding during each intermediate step.

Special reserved exponent values: `E = 0` (with `N` all zero) represents `±0`;
`E = 0` with nonzero `N` represents tiny "subnormal" numbers (extra precision very
close to zero); `E = 255` represents `±∞` (if `N = 0`) or `NaN` (if `N ≠ 0`).

## Complexity
N/A — conceptual/representation topic, no algorithm complexity here.

## Code
No C++ sample for this chapter (`codes/cpp/chapter_data_structure/` doesn't exist).

Diagrams (from `en/docs/chapter_data_structure/number_encoding.assets/`):
- `1s_2s_complement.png` — sign-magnitude / 1's complement / 2's complement
  conversions worked out for +0, +5, -0, -28
- `ieee_754_float.png` — IEEE 754 float bit layout and the 0.171875 worked example

## Related
- [[Basic Data Types]]
- [[Character Encoding]]

## Self-check questions
1. Trace through computing `2 + (-3)` in 2's complement (8-bit) by hand and confirm
   you get `-1`.
2. Why does 2's complement end up with only one zero, while sign-magnitude and 1's
   complement each have two?
3. Given `S=0, E=128, N=0`, what decimal value does this `float` represent? (Hint:
   use the formula above.)
4. Why can two mathematically-equal floating-point computations sometimes fail an
   `==` comparison in code?
