---
tags:
  - chapter/data-structure
status: done
date: 2026-08-07
---

# Classification of Data Structure

## Core idea
Data structures can be classified along two independent axes: how the data relates
conceptually (logical structure), and how the data actually sits in memory
(physical structure). These two axes are independent — knowing one doesn't tell you
the other, and understanding both is what lets you reason about a new data structure
you haven't seen before.

### Axis 1 — Logical structure: how do the elements relate to each other?

**Linear** means every element has at most one predecessor and at most one
successor — you can walk through the whole structure in a single line, front to
back. Concretely: an array `[10, 20, 30, 40]` — element `20` has exactly one element
before it (`10`) and one after it (`30`). Arrays, linked lists, stacks, and queues
are all linear for this reason: no matter how you implement them, conceptually
they're a single sequence.

**Non-linear** means an element can relate to more than one other element at once.
This further splits into two cases:
- **Tree structures** — one-to-many. Picture a company org chart: one manager has
  many direct reports, but each report has exactly one manager. A binary tree is the
  same shape: one parent node points to up to two children, but each child has
  exactly one parent. Heaps and hash tables (internally, when you look at how a
  bucket holds multiple entries) are also tree-shaped in this sense.
- **Network structures** — many-to-many. Picture a social network: person A can be
  friends with B, C, and D, and B can also be friends with C and E — there's no
  single "parent." This is exactly what a **graph** models: any node can connect to
  any number of other nodes, with no hierarchy.

**Worked example**: Take these four data structures and classify them:
1. `int arr[5]` — linear (each slot has one neighbor before/after).
2. A singly linked list `1 -> 2 -> 3 -> nullptr` — linear (same reasoning, just
   different physical storage).
3. A binary search tree with root `50`, left child `30`, right child `70` — non-linear,
   tree structure (`50` relates to two children, but `30` and `70` each relate back
   to only one parent).
4. A friend-graph where Alice, Bob, and Carol are all mutually connected — non-linear,
   network structure (each node can have many connections with no single parent).

### Axis 2 — Physical structure: how is the data actually stored in memory?

**Contiguous** (array-based): all elements live in one unbroken block of memory,
back-to-back. This is what makes indexed access `arr[i]` fast — the address of
element `i` can be computed directly as `base_address + i * element_size`, no
searching required.

**Dispersed** (linked-list-based): elements are scattered anywhere in memory, and
each element stores a pointer/reference to the next one to stitch them together
logically. There's no way to jump directly to element `i` — you must start at the
head and follow `next` pointers one at a time.

**Worked example**: imagine 4 integers, value `7`.
- Array version: memory addresses `1000, 1004, 1008, 1012` hold `7, 9, 3, 5` — one
  contiguous block (each `int` is 4 bytes, so consecutive addresses are 4 apart).
  Want element at index 2? Compute `1000 + 2*4 = 1008` directly — O(1).
  - Array of value `7`
- Linked-list version: node holding `7` might live at address `2048` and point to a
  node holding `9` at address `5104`, which points to a node holding `3` at address
  `1024`, and so on — addresses are scattered, no arithmetic shortcut exists. To
  reach the 3rd element you must traverse: head → next → next — O(n).

This physical distinction is exactly why **arrays and linked lists are called the
two storage primitives**: literally every other data structure you'll learn is built
by combining or wrapping one or both of these. A **hash table** is typically an
array of buckets, where each bucket is a linked list (handles collisions by chaining
multiple entries). A **tree** is typically built from linked node objects (each node
holds pointers to its children) — though Chapter 7.3 will show trees can also be
represented as arrays. A **graph** is typically an array (or hash map) of adjacency
lists, where each adjacency list is... a linked list of neighbors.

### Static vs dynamic (a consequence of the physical-structure choice)

Because an array is one contiguous block, its size must be fixed at allocation time
— **static**. You cannot grow it in place if it fills up; you'd need to allocate a
brand-new, bigger block and copy everything over. A linked list, by contrast, grows
one node at a time by allocating a new node and linking it in — **dynamic**, no
upfront size commitment needed.

**Worked example**: `std::vector<int> v` in C++ *looks* dynamically resizable, but
under the hood it's still array-based — when it outgrows its current capacity, it
silently allocates a new, larger array and copies every element over (this is why
`push_back` is usually O(1) but occasionally O(n) on the resize). A
`std::list<int>` (doubly linked list), by contrast, truly allocates one new node per
`push_back`, no copying ever needed.

## Complexity
N/A — this section is pure classification, no algorithm/complexity to analyze yet.
(The O(1) vs O(n) access difference between array and linked list, mentioned above,
is previewed here but formally covered in Chapter 4.)

## Code
No C++ code for this chapter — Data Structure (Chapter 3) is conceptual only. See
later chapters (Array & LinkedList, Stack & Queue, Tree, Graph, Hashing) for the
concrete implementations of each structure named here.

Diagrams (from `en/docs/chapter_data_structure/classification_of_data_structure.assets/`):
- `classification_logic_structure.png` — linear vs non-linear (tree/network) breakdown
- `classification_phisical_structure.png` — contiguous (array) vs dispersed (linked list)

## Related
- [[Basic Data Types]]

## Self-check questions
1. Classify a stack: is it linear or non-linear, and why?
2. Why can you compute the memory address of `arr[i]` directly, but not the address
   of the 5th node in a linked list?
3. Why is `std::vector` still considered "array-based/static" under the hood even
   though it can grow at runtime?
4. Name one data structure that's built by combining array + linked list, and
   explain which part is which.
