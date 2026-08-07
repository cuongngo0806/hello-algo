---
tags:
  - quiz/array-linkedlist
status: done
date: 2026-08-07
---

# Array & LinkedList Chapter Quiz

Covers: [[Array]], [[Linked List]], [[List]] (sections 4.1-4.3).

Result: **8/8 correct** on first attempt. Chapter marked `done` on 2026-08-07.

## Q1 — Given an array starting at address 200, with 8-byte elements, what's the address of the element at index 4?

- **232 — `200 + 8×4 = 232`** ✅ *(user's answer, correct)*
- 216 — `200 + 8×2` — wrong index used in the formula.
- 208 — `200 + 8×1` — wrong index used in the formula.
- 240 — `200 + 8×5` — wrong index used in the formula.

## Q2 — Why is accessing the i-th node of a linked list O(n), while inserting a node (given a pointer to the node before it) is O(1)?

- **No address formula for access; insert only rewires pointers** — Access must hop
  node-by-node from the head (no arithmetic shortcut exists because nodes are
  scattered); insert just changes 1-2 pointers, no traversal needed. ✅ *(user's
  answer, correct)*
- Both are actually O(1) — Access is NOT O(1) for a linked list — there's no direct
  address formula, unlike an array.
- Access is O(1) too, just like arrays — Linked lists lack the contiguous-memory
  property that makes array access O(1).
- Insert is O(n) because you must shift nodes — That's the array insert cost, not the
  linked list's — linked list insert doesn't shift anything.

## Q3 — Trace: insert node X (value 9) between n1 and n2 in the chain 1→3→2→5→4. What two pointer assignments happen, and in what order?

- **X.next = n2, then n1.next = X** — Point the new node forward first (preserves
  reachability of n2 throughout), then redirect the predecessor. ✅ *(user's answer,
  correct)*
- n1.next = X, then X.next = n2 — Wrong order — overwriting n1.next first would lose
  the only reference to n2 before X had a chance to point at it.
- X.next = n1, then n2.next = X — Points the new node at the wrong neighbor entirely.
- Only one assignment needed: n1.next = X — Skips linking X forward — X would dangle
  with no next pointer, breaking the chain at that point.

## Q4 — Why must P.next be set BEFORE n0.next is overwritten during linked list insertion?

- **Otherwise the rest of the list becomes unreachable** — Once n0.next is reassigned,
  if P doesn't already know where n1 is, nothing points to n1 anymore — that portion
  of the list is lost. ✅ *(user's answer, correct)*
- C++ requires initialization order — This is a language syntax rule, not a logic
  requirement — the actual reason is about reachability, not syntax.
- It doesn't matter, either order works — The order is exactly what prevents losing
  the rest of the chain — reversing it breaks the list.
- To avoid a memory leak — The concern is unreachability of the rest of the list, not
  a leak (no memory is actually leaked either way in this trace).

## Q5 — After removing node P from a linked list (n0.next = n1), P.next still points to a valid node (n1). Why does the list still correctly consider P "deleted"?

- **Nothing in the reachable chain points to P anymore** — Starting at the head and
  walking forward, you'll never land on P — deletion means unreachability, not erasing
  P's own data. ✅ *(user's answer, correct)*
- P's value is automatically zeroed out — Removal doesn't touch P's value field at
  all.
- P.next is actually set to nullptr during removal — remove() never touches P's own
  next pointer — it stays stale-but-intact.
- The list keeps a separate "deleted" flag on each node — No such flag exists in this
  implementation; deletion is purely about reachability.

## Q6 — Trace: deleting the element at index 0 of the array [10, 20, 30, 40] by hand — what's the resulting state and complexity?

- **[20, 30, 40, 40], O(n)** — Shift every remaining element left one slot
  (`nums[0]=nums[1]`, `nums[1]=nums[2]`, `nums[2]=nums[3]`); worst case (index 0)
  shifts all of them, leaving a leftover duplicate `40` at the end. ✅ *(user's answer,
  correct)*
- [20, 30, 40], O(n) — Array length is fixed — it doesn't physically shrink; the
  trailing duplicate remains, just logically ignored.
- [10, 20, 30, 40] unchanged, O(1) — Deletion in an array-based structure requires
  shifting to close the gap — it's not a flag-based removal.
- [20, 30, 40, 40], O(1) — Correct resulting state, but the shift itself costs O(n),
  not O(1).

## Q7 — Why is push_back/add() on a dynamic array usually O(1) but occasionally O(n)? What triggers the expensive case?

- **Usually a single write; O(n) only when capacity is full and it must expand** —
  Most calls just write to the next free slot; when `arrSize == arrCapacity`, it must
  allocate a bigger array and copy every old element over first. ✅ *(user's answer,
  correct)*
- It's always O(n) because C++ vectors always copy on every push_back — Most calls are
  cheap single-slot writes; the copy only happens on the rare expansion call.
- It's always a flat O(1), no exceptions — Expansion is a real, occasional O(n) cost —
  that's exactly why the complexity is described as "amortized," not flat O(1).
- O(n) happens because of sorting after every insert — push_back doesn't sort
  anything; the O(n) cost comes purely from the copy-on-expansion mechanism.

## Q8 — Starting from an empty MyList with arrCapacity=10, extendRatio=2: after exactly how many add() calls does the first expansion trigger, and what's arrCapacity afterward?

- **11th call, arrCapacity becomes 20** — Calls 1-10 fill the initial 10 slots
  (`arrSize` reaches 10 = `arrCapacity`); the 11th call sees `size()==capacity()`
  first, so it expands to `10×2=20` BEFORE writing the 11th element. ✅ *(user's
  answer, correct)*
- 10th call, arrCapacity becomes 20 — The 10th call still has room (it fills the last
  of the original 10 slots); the capacity check that triggers expansion only fires
  on the 11th call, which finds the array already full.
- 11th call, arrCapacity becomes 11 — `extendRatio` multiplies capacity (`×2`), it
  doesn't just add 1.
- 5th call, arrCapacity becomes 20 — Expansion only triggers once capacity is
  actually exhausted (all 10 slots full), not halfway through.

## Related
- [[Array]]
- [[Linked List]]
- [[List]]
