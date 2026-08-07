---
tags:
  - chapter/complexity
status: done
date: 2026-08-07
---

# Iteration and Recursion

## Core idea
Two basic control structures for repeating work. **Iteration** (`for`/`while` loops)
repeats a block while a condition holds — bottom-up. **Recursion** (a function calling
itself) decomposes a problem into smaller subproblems, descending until a termination
condition, then ascending back up combining results — top-down. Recursion uses more
memory (call stack frames pile up) and is generally slower (call overhead) than the
equivalent loop.

## Complexity
- Single loop over `n`: linear growth.
- Nested loop (n inside n): quadratic growth.
- Recursion with 2 branches per call, n levels deep: exponential growth (see
  [[Time Complexity]]).

## Code
See: `codes/cpp/chapter_computational_complexity/iteration.cpp` (forLoop, whileLoop,
whileLoopII, nestedForLoop)
See: `codes/cpp/chapter_computational_complexity/recursion.cpp` (recur, tailRecur, fib,
forLoopRecur)

## Related
- [[Algorithm Efficiency Evaluation]]
- [[Time Complexity]]
- [[Space Complexity]]

## Self-check questions
1. What's the difference between "descend" and "ascend" phases in recursion?
2. Why does recursion typically use more memory than an equivalent loop?
3. What is tail recursion, and why might it be more efficient?
