---
tags:
  - chapter/array-linkedlist
status: taught
date: 2026-08-07
---

# Array

## Core idea
An **array** is a linear data structure that stores elements of the same type in one
contiguous block of memory. "Contiguous" is the whole story here — every other property
of arrays (fast access, slow insert/delete, fixed length) falls directly out of that one
fact.

Diagram: **`array_definition.png`** shows the array `[1, 3, 2, 5, 4]` sitting in 5
back-to-back slots carved out of a larger pool of available memory. Element `3` (index 1)
is physically right next to element `1` (index 0) — not just conceptually adjacent, but
actually adjacent in RAM.

### Why does indexing start at 0, not 1?

The diagram's own numbers answer this directly. Memory locations for indices
`0, 1, 2, 3, 4` are `00, 04, 08, 12, 16`. Index `0` sits at address `00` — that's an
offset of `0` bytes from the array's start. Index `1` sits at address `04` — an offset of
`4` bytes. **The index literally IS the offset**, measured in "how many element-slots
away from the start." The very first element hasn't moved away from the start at all, so
its offset — its index — is naturally `0`. If indexing started at 1, you'd have to
subtract 1 from every index before computing an address, for zero benefit.

### The address formula — worked example

Diagram: **`array_memory_location_calculation.png`** gives the formula:

```
Element's memory location = Array's memory location + Element length × Element index
```

Worked example straight from the diagram: array `[1, 3, 2, 5, 4]` starts at address
`000`, each `int` element is `4` bytes, and we want the address of the element at
**index 3** (value `5`):

```
address = 000 + 4 × 3
        = 000 + 12
        = 012
```

Checking against the diagram's table: index 3's listed memory location is `12`. Matches.
Notice this computation is **pure arithmetic** — one multiply, one add — no searching, no
walking through the array element by element. That's exactly why array access is **O(1)**:
the cost is the same no matter how big the array is or which index you ask for.

C++ code: [array.cpp:10-16](../codes/cpp/chapter_array_and_linkedlist/array.cpp) —
`randomAccess()`. Writing `nums[randomIndex]` in C++ doesn't spell out the formula
explicitly, but that formula is exactly what the compiler generates under the hood.

### Inserting an element — worked trace

Diagram: **`array_insert_element.png`** — example: insert value `3` at index `1` of
`[1, 2, 5, 4, 0]` (the trailing `0` is a placeholder; only 4 of the 5 slots hold real
data).

Because there's no gap between elements in memory, you can't just drop `3` into slot 1 —
`2` is already sitting there. The fix: shift everything from the **end** backward by one
slot, freeing up slot 1, then write `3` into it. Full sequence from the diagram:

1. Start: `[1, 2, 5, 4, 0]`
2. `nums[4] = nums[3]` → `[1, 2, 5, 4, 4]` (copy the `4` from slot 3 into slot 4)
3. `nums[3] = nums[2]` → `[1, 2, 5, 5, 4]` (copy the `5` from slot 2 into slot 3)
4. `nums[2] = nums[1]` → `[1, 2, 2, 5, 4]` (copy the `2` from slot 1 into slot 2)
5. Slot 1 is now free: `nums[1] = 3` → `[1, 3, 2, 5, 4]`

The shift direction matters: starting from the **back** and moving toward the target
index avoids overwriting data before it's been copied forward. If you shifted
front-to-back instead, you'd clobber values before saving them.

C++ code, [array.cpp:33-40](../codes/cpp/chapter_array_and_linkedlist/array.cpp):
```cpp
void insert(int *nums, int size, int num, int index) {
    // shift index and everything after it one position to the right
    for (int i = size - 1; i > index; i--) {
        nums[i] = nums[i - 1];
    }
    nums[index] = num;
}
```
The loop runs from `size-1` down to `index+1` — exactly the back-to-front shifting shown
above. Worst case (inserting at index 0) shifts every element — **O(n)**.

**The catch**: since array length is fixed, whatever sat in the *last* slot before the
insert (the `0` placeholder here) simply falls off the end and is discarded. In this
example that's harmless because it was already a placeholder — but if every slot held
real data, inserting one more element would silently lose the last real value.

### Removing an element — worked trace

Diagram: **`array_remove_element.png`** — example: delete the element at index `1`
(value `3`) from `[1, 3, 2, 5, 4]`.

Mirror image of insertion — shift everything **after** the target index forward by one,
closing the gap:

1. Start: `[1, 3, 2, 5, 4]`, deleting index 1 (`3`)
2. `nums[1] = nums[2]` → `[1, 2, 2, 5, 4]`
3. `nums[2] = nums[3]` → `[1, 2, 5, 5, 4]`
4. `nums[3] = nums[4]` → `[1, 2, 5, 4, 4]`

Final: `[1, 2, 5, 4, 4]` — logically 4 elements now (`1, 2, 5, 4`); the trailing `4` is a
leftover duplicate, simply ignored — no need to explicitly clear it.

C++ code, [array.cpp:43-48](../codes/cpp/chapter_array_and_linkedlist/array.cpp):
```cpp
void remove(int *nums, int size, int index) {
    for (int i = index; i < size - 1; i++) {
        nums[i] = nums[i + 1];
    }
}
```
Same **O(n)** worst case (deleting index 0 shifts everything).

### Traversing and finding

`traverse()` ([array.cpp:51-57](../codes/cpp/chapter_array_and_linkedlist/array.cpp))
walks index `0` to `size-1` summing values — plain **O(n)**, nothing subtle.

`find()` ([array.cpp:60-66](../codes/cpp/chapter_array_and_linkedlist/array.cpp)) does
the same walk but compares each element to a target, returning the first matching index
(or `-1`). This is called **linear search** — O(n) because in the worst case (target
absent, or it's the very last element) every slot must be checked.

### Expanding an array — worked trace

Because array length is fixed once allocated (the memory right after it may already
belong to something else — unsafe to grow into it), "expanding" in C++ actually means:
allocate a **brand-new, bigger** array, copy every old element over, then discard the
old one.

C++ code, [array.cpp:19-30](../codes/cpp/chapter_array_and_linkedlist/array.cpp):
```cpp
int *extend(int *nums, int size, int enlarge) {
    int *res = new int[size + enlarge];   // new bigger array
    for (int i = 0; i < size; i++) {
        res[i] = nums[i];                  // copy every old element
    }
    delete[] nums;                         // free the old array
    return res;
}
```

Worked trace from the driver code: starting `nums = [1, 3, 2, 5, 4]` (size 5), calling
`extend(nums, 5, 3)` allocates a new 8-slot array, copies all 5 old values into it, frees
the old 5-slot array, and returns the new one — size is now `5 + 3 = 8`, with 3 trailing
uninitialized slots. This copy is **O(n)** — expensive for large arrays, and exactly why
"auto-growing" containers like `std::vector` occasionally pay this same cost internally
when they outgrow their current capacity (see [[Classification of Data Structure]] for
the `std::vector` resize discussion).

## Complexity
| Operation | Complexity | Why |
|---|---|---|
| Access by index | O(1) | Direct address arithmetic — no traversal |
| Insert | O(n) worst case | Must shift all elements after the insertion point |
| Remove | O(n) worst case | Must shift all elements after the removal point |
| Traverse | O(n) | Must visit every element once |
| Find (linear search) | O(n) worst case | May need to check every element |
| Expand | O(n) | Must copy every old element into the new array |

### Advantages and limitations

**Why access is fast**: contiguous storage means the system doesn't just load the one
element you asked for — modern CPUs also cache **nearby** memory (**cache locality**),
so subsequent accesses to nearby indices tend to be even faster, hitting a fast cache
instead of a slow trip to main memory.

**The trade-off**: the same contiguity that makes access fast is exactly what makes
insert/delete expensive (everything must physically shift) and forces the fixed-length
restriction (unsafe to claim more space next door without knowing it's free).

### Typical applications
- **Random sampling**: store items in an array, generate random indices to sample from.
- **Sorting and searching**: quicksort, merge sort, binary search all operate on arrays.
- **Lookup tables**: e.g. mapping a character to its ASCII code by using the ASCII value
  directly as an index.
- **Machine learning**: tensors/matrices/vectors are built as arrays.
- **Building block for other structures**: stacks, queues, hash tables, heaps, graphs
  (e.g. a graph's adjacency matrix is literally a 2D array) — recall from
  [[Classification of Data Structure]] that array + linked list are the two physical
  primitives everything else is built from.

## Code
[array.cpp](../codes/cpp/chapter_array_and_linkedlist/array.cpp) — `randomAccess()`,
`insert()`, `remove()`, `traverse()`, `find()`, `extend()`, all exercised in `main()`.

Diagrams (from `en/docs/chapter_array_and_linkedlist/array.assets/`):
- `array_definition.png` — array layout in contiguous memory, index/address table
- `array_memory_location_calculation.png` — the direct-address formula, worked for index 3
- `array_insert_element.png` — insert `3` at index 1, back-to-front shift trace
- `array_remove_element.png` — delete index 1, front-to-back shift trace

## Related
- [[Classification of Data Structure]]
- [[Basic Data Types]]

## Self-check questions
1. Why is array access O(1) but insertion O(n) in the worst case?
2. Given an array starting at address `100`, with 8-byte elements, what's the address of
   the element at index 5? Show the formula.
3. Trace through deleting the element at index 0 of `[10, 20, 30, 40]` by hand — what's
   the resulting array state after the shift?
4. Why can't an array simply "grow" in place when it runs out of space, the way a linked
   list can add a new node?
5. Name two other data structures that are typically built on top of arrays.
