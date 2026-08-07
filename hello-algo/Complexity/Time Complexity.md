---
tags:
  - chapter/complexity
  - complexity/time
status: done
date: 2026-08-07
---

# Time Complexity

## Core idea
Time complexity counts the number of operations as a function of input size `n`, then
keeps only the highest-order term (dropping coefficients and lower-order terms) —
because that term dominates as `n → ∞`. This is the **asymptotic upper bound**
(Big-O): `T(n) = O(f(n))` means once `n` is large enough, the real operation count is
always bounded above by some constant multiple of `f(n)`. We always report the
tightest valid bound.

Common classes, from best to worst:
- **O(1)** constant — operation count independent of `n`.
- **O(log n)** logarithmic — input halves each round (e.g. binary search).
- **O(n)** linear — single loop over `n`.
- **O(n log n)** linearithmic — log-depth recursion tree, O(n) work per level (e.g.
  merge sort, quicksort, heap sort).
- **O(n²)** quadratic — nested loops (e.g. bubble sort).
- **O(2ⁿ)** exponential — recursion branching in 2 at each of n levels (e.g. naive
  Fibonacci, brute-force search).
- **O(n!)** factorial — permutation generation.

Also: **worst-case** (Big-O) is what's normally reported — a safety guarantee the
algorithm never exceeds. **Best-case** (Ω) rarely useful (low probability). **Average
case** (Θ) is the most "honest" number but often hard to compute, so people loosely
say "average time O(n)" meaning Θ(n).

## Complexity
See classes above — this section *is* the complexity taxonomy.

## Code
See: `codes/cpp/chapter_computational_complexity/time_complexity.cpp` — constant(),
linear(), arrayTraversal(), quadratic(), bubbleSort(), exponential(), expRecur(),
logarithmic(), logRecur(), linearLogRecur(), factorialRecur()
See: `codes/cpp/chapter_computational_complexity/worst_best_time_complexity.cpp` —
findOne() (worst/best case example)

Diagrams (from `en/docs/chapter_computational_complexity/time_complexity.assets/`):
- `time_complexity_common_types.png` — 5-curve growth comparison
- `asymptotic_upper_bound.png` — Big-O formal definition
- `time_complexity_simple_example.png` — constant vs linear example
- `time_complexity_exponential.png` — recursion tree for O(2ⁿ)
- `time_complexity_logarithmic.png` — halving diagram for O(log n)

## Related
- [[Iteration and Recursion]]
- [[Space Complexity]]
- [[Algorithm Efficiency Evaluation]]

## Self-check questions
1. Why do we drop lower-order terms and coefficients when stating Big-O?
2. What's the time complexity of a nested double loop over n?
3. What's the time complexity of a recursive function with 2 branches, n levels deep?
4. Why is worst-case complexity usually reported instead of best-case or average-case?
