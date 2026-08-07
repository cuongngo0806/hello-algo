---
tags:
  - chapter/complexity
  - complexity/space
status: done
date: 2026-08-07
---

# Space Complexity

## Core idea
Same idea as time complexity, but tracking memory instead of operation count. Memory
during execution breaks into: input space, temporary space (temporary data + stack
frame space + instruction space), and output space. In practice, "space complexity" =
temporary data + stack frame space + output data.

Unlike time complexity, **only worst-case space is reported** — memory is a hard
requirement (not a "graceful degradation" like slow time); you must reserve enough for
the worst-case input or the program crashes.

**Key insight — the recursion trap**: two functions can have identical time
complexity but different space complexity, because of the call stack. A loop calling a
function n times releases each call's stack frame before the next call — O(1) space.
A recursive function calling itself n times keeps all n frames alive simultaneously
until the base case returns — O(n) space. This is *why* recursion uses more memory
than iteration (see [[Iteration and Recursion]]).

Common classes mirror time complexity: O(1) fixed allocations, O(n) array/list/hashmap
sized n or recursion depth n, O(n²) an n×n matrix or n stacked frames each allocating
O(n), O(2ⁿ) a full binary tree of height n, O(log n) recursion depth shrinking by half
each level.

Closing idea: time and space are often a **trade-off** — caching/precomputing spends
more memory to save time (the basis of hashing and dynamic programming later in the
course).

## Complexity
See classes above.

## Code
See: `codes/cpp/chapter_computational_complexity/space_complexity.cpp` — constant(),
linear(), linearRecur(), quadratic(), quadraticRecur(), buildTree()

Diagram: `en/docs/chapter_computational_complexity/space_complexity.assets/space_types.png`
— input/temporary/output space breakdown

## Related
- [[Time Complexity]]
- [[Iteration and Recursion]]
- [[Algorithm Efficiency Evaluation]]

## Self-check questions
1. Why is only worst-case space complexity reported, unlike time complexity?
2. Why does `recur(n)` use O(n) space while a functionally similar loop uses O(1)?
3. What is the "trading time for space" idea, and where does it show up later in the
   course?
