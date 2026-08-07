---
tags:
  - quiz/complexity
status: done
date: 2026-08-07
---

# Complexity Chapter Quiz

Covers: [[Algorithm Efficiency Evaluation]], [[Iteration and Recursion]],
[[Time Complexity]], [[Space Complexity]] (sections 2.1-2.4).

Result: **8/8 correct** (Q7 needed one retry — see below). Chapter marked `done`
on 2026-08-07.

## Q1 — Why is "actual testing" unreliable for comparing algorithms?

- A) Too much skill required — It requires too much programming skill to set up correctly
- **B) Hardware/input-size dependent** — Results depend on hardware/environment and on
  the specific input size tested — they don't generalize ✅ *(user's answer, correct)*
- C) Can't measure memory — It can only measure time, never memory
- D) Compiler randomness — Compilers optimize differently every time you run the same code

## Q2 — What does time complexity (Big-O) actually measure?

- A) Exact seconds — The exact number of seconds the algorithm takes to run
- **B) Growth trend** — The growth trend of operation count as input size n increases,
  ignoring constants ✅ *(user's answer, correct)*
- C) Lines of code — The total number of lines of code in the algorithm
- D) CPU clock speed — The average CPU clock speed needed to run the algorithm

## Q3 — Nested double for-loop (both 0..n) incrementing a counter — time complexity?

- A) O(n) — Linear
- B) O(n log n) — Linearithmic
- **C) O(n²)** — Quadratic ✅ *(user's answer, correct)*
- D) O(2ⁿ) — Exponential

## Q4 — loop() calls func() n times in a for-loop; recur() recursively calls itself n times (recur(n-1) before returning). Complexity comparison?

- **A) loop O(n) time / O(1) space, recur O(n) time / O(n) space** — loop's stack frames
  don't accumulate; recur's do ✅ *(user's answer, correct)*
- B) Identical complexity — Both have identical time and space complexity
- C) loop O(n²), recur O(n) — loop is O(n²) time; recur is O(n) time
- D) recur is faster — recur is faster because recursion avoids loop overhead

## Q5 — A recursive function makes TWO recursive calls at each level, recursing n levels deep (like naive Fibonacci). Time complexity?

- A) O(n) — Linear
- B) O(n²) — Quadratic
- C) O(log n) — Logarithmic
- **D) O(2ⁿ)** — Exponential — branching recursion tree ✅ *(user's answer, correct)*

## Q6 — Why do we normally report WORST-CASE time complexity instead of best/average-case?

- A) Easiest to calculate — Worst-case is always the easiest to calculate mathematically
- **B) Safety guarantee** — Worst-case gives a reliable safety guarantee — the algorithm
  will never perform worse than this ✅ *(user's answer, correct)*
- C) No standard notation — Best-case and average-case don't have standard mathematical notations
- D) Language spec requirement — Worst-case is required by most programming language specifications

## Q7 — Why is space complexity normally reported as WORST-CASE only (unlike time complexity)?

- A) Never varies — Because space usage never varies between different inputs
- **B) Hard requirement** — Because memory is a hard requirement — you must always have
  enough reserved, no matter what input arrives ✅ *(correct answer)*
- C) Impossible to define — Because best-case and average-case space are impossible to define
- D) Doesn't depend on input — Because space complexity doesn't depend on input size

**First attempt**: user answered A) "Never varies" — ❌ incorrect. This isn't true — space
usage absolutely varies with input (e.g. an array of size n uses more memory as n grows).
The real reason worst-case matters specifically for *space* (and not just time) is that
memory is a hard ceiling: if you run out partway through, the program crashes — there's no
"it just runs a bit slower" fallback the way there is with time. So you must provision for
the worst case up front.

**Retry**: user re-answered B) "Hard requirement" — ✅ correct.

## Q8 — `int recur(int n) { if (n==1) return 1; int res = recur(n-1); return n+res; }` called as `recur(n)`. SPACE complexity?

- A) O(1) — Each call reuses the same stack space
- **B) O(n)** — Up to n stack frames exist simultaneously before any of them return ✅
  *(user's answer, correct)*
- C) O(n²) — Because of the addition operation
- D) O(log n) — Because the input decreases each call

**Worked trace**: `recur(4)` calls `recur(3)` calls `recur(2)` calls `recur(1)` — each call
sits on the stack *waiting* for its recursive call to return before it can compute
`n + res`, so by the time `recur(1)` finally returns `1`, there are 4 stack frames
alive simultaneously (for `recur(4)`, `recur(3)`, `recur(2)`, `recur(1)`). That's why the
space is O(n), not O(1) — it's not about how much work each call does, it's about how many
calls are stacked up unfinished at the deepest point.

## Related
- [[Algorithm Efficiency Evaluation]]
- [[Iteration and Recursion]]
- [[Time Complexity]]
- [[Space Complexity]]
