---
tags:
  - chapter/array-linkedlist
status: taught
date: 2026-08-07
---

# List

## Core idea
A **list** is an *abstract* data structure concept — not a specific one of "array" or
"linked list," but a description of *behavior*: an ordered collection supporting
access, modification, insertion, deletion, and traversal, **without requiring the user
to think about capacity limits**. Both physical primitives from [[Array]] and
[[Linked List]] can serve as a list's underlying implementation:

- A linked list can naturally serve as a list — it already supports insert/delete/
  search/update, and grows flexibly as needed (no fixed size to worry about).
- A plain array can also serve as a list — but because its length is *fixed*, it's
  really only "a list with a hard capacity ceiling." You can't naturally grow past it.

### The motivating problem

When writing a program, you usually don't know in advance exactly how many elements
you'll need to store. If you commit to a fixed-size array and guess too small, it
fails once you exceed capacity. Guess too large, and you've wasted memory that could
have gone elsewhere. Neither guess is satisfying — this is the exact same
scattered-vs-contiguous tension that motivated [[Linked List]], but here the fix stays
inside the array world.

### The solution: dynamic array

The fix is a **dynamic array** — it inherits everything good about a plain array
(contiguous storage, O(1) index access) but adds the ability to **resize itself during
execution** when it runs out of room. This is, in fact, what "list" means in most
languages' standard libraries: `list` in Python, `ArrayList` in Java, `vector` in C++,
`List` in C#. From here on, "list" and "dynamic array" are treated as the same concept
— and in this project's C++ code, that's `std::vector`.

### Access — O(1), same as array

Since a list *is* an array underneath, access and update are still direct-address
arithmetic — **O(1)**, exactly as covered in [[Array]]:

```cpp
int num = nums[1];  // Access element at index 1
nums[1] = 0;         // Update element at index 1 to 0
```

Nothing new here — the underlying storage is still contiguous, so the same address
formula from [[Array]] still applies.

### Insert and delete — worked trace, O(1) at the end vs O(n) elsewhere

This is where a list genuinely improves on a plain array. **Adding at the end** is
O(1) (usually — more on that below), but inserting/deleting **in the middle** is still
O(n), because elements after the target index still have to shift, exactly like a
plain array.

From the driver code, [list.cpp:30-47](../codes/cpp/chapter_array_and_linkedlist/list.cpp):
```cpp
nums.clear();          // Clear the list

nums.push_back(1);     // Add elements at the end — O(1) each
nums.push_back(3);
nums.push_back(2);
nums.push_back(5);
nums.push_back(4);

nums.insert(nums.begin() + 3, 6);  // Insert 6 at index 3 — O(n), shifts elements
nums.erase(nums.begin() + 3);      // Delete element at index 3 — O(n), shifts elements
```

Worked trace on `push_back`: starting from an empty `nums`, each `push_back` call just
writes the new value into the next free slot and bumps a length counter — no shifting
needed, because you're writing at the *end*, not squeezing something into the middle.
That's the O(1) case. Compare that with `nums.insert(nums.begin() + 3, 6)`: to make
room at index 3, everything from index 3 onward (values `5`, `4` at that point) has to
shift one slot right first — same shift-from-the-back mechanic as [[Array]]'s
`insert()`, just wrapped behind a library call instead of hand-written.

### Traverse, concatenate, sort

All straightforward, from [list.cpp:49-69](../codes/cpp/chapter_array_and_linkedlist/list.cpp):
```cpp
// Traverse by index
int count = 0;
for (int i = 0; i < nums.size(); i++) {
    count += nums[i];
}
// Traverse elements directly
count = 0;
for (int num : nums) {
    count += num;
}

// Concatenate: append every element of nums1 onto the end of nums
vector<int> nums1 = {6, 8, 7, 10, 9};
nums.insert(nums.end(), nums1.begin(), nums1.end());

// Sort ascending
sort(nums.begin(), nums.end());
```
Traversal is O(n) — visiting every element once, same reasoning as [[Array]]. Sorting
a list is worth calling out specifically because, once sorted, it unlocks **binary
search** and **two-pointer** techniques — both are extremely common in array-based
algorithm problems, and both *require* sorted input to work correctly.

### How dynamic arrays actually expand — worked trace through `my_list.cpp`

This is the part that makes "dynamic" concrete instead of magic. The repo includes a
hand-rolled implementation, `MyList`, that mirrors what `std::vector` does internally.
Three design decisions drive it:

- **Initial capacity** — start with some fixed-size underlying array. This
  implementation picks `10`.
- **Size tracking** — a separate `arrSize` counter tracks how many slots are *actually
  used* (as opposed to `arrCapacity`, which is how many slots physically exist). This
  is what lets `add()` know when it's about to overflow.
- **Expansion mechanism** — when `arrSize` reaches `arrCapacity`, allocate a
  *brand-new*, bigger array, copy every old element into it, then discard the old
  array. This is exactly [[Array]]'s `extend()` operation, just triggered
  automatically instead of by an explicit user call.

Class fields, [my_list.cpp:10-15](../codes/cpp/chapter_array_and_linkedlist/my_list.cpp):
```cpp
class MyList {
  private:
    int *arr;             // underlying array (stores list elements)
    int arrCapacity = 10; // list capacity (physical slots)
    int arrSize = 0;      // list length (currently used slots)
    int extendRatio = 2;  // multiplier each time we expand
```

The `add()` method — this is the O(1)-usually operation:
```cpp
void add(int num) {
    if (size() == capacity())   // out of room?
        extendCapacity();       // grow first
    arr[size()] = num;          // write into the next free slot
    arrSize++;                  // one more element now
}
```

**Worked trace**: the driver code calls `nums->add(...)` five times (values
`1, 3, 2, 5, 4`), which just fills slots `0` through `4` of the 10-slot initial array —
no expansion triggered yet. `arrSize` goes `0 → 1 → 2 → 3 → 4 → 5`, `arrCapacity` stays
`10`.

Then the driver runs a loop calling `add()` ten more times, [my_list.cpp:157-161](../codes/cpp/chapter_array_and_linkedlist/my_list.cpp):
```cpp
for (int i = 0; i < 10; i++) {
    // when i = 5, list length will exceed capacity, expansion triggers here
    nums->add(i);
}
```
At the point this loop starts, `arrSize` is already `5` (from the earlier five
`add()` calls) and `arrCapacity` is `10`. Trace it slot-by-slot: `i=0` writes to slot 5
(`arrSize` becomes 6), `i=1` writes to slot 6 (`arrSize`=7), `i=2` → slot 7
(`arrSize`=8), `i=3` → slot 8 (`arrSize`=9), `i=4` → slot 9 (`arrSize`=10 — **now
full**, `arrCapacity == arrSize`). The next iteration, `i=5`, calls `add()` again: this
time `size() == capacity()` is true *before* the write happens, so `extendCapacity()`
fires first.

`extendCapacity()`, [my_list.cpp:95-107](../codes/cpp/chapter_array_and_linkedlist/my_list.cpp):
```cpp
void extendCapacity() {
    int newCapacity = capacity() * extendRatio;  // 10 * 2 = 20
    int *tmp = arr;
    arr = new int[newCapacity];                   // brand-new, bigger array
    for (int i = 0; i < size(); i++) {             // copy all 10 old elements over
        arr[i] = tmp[i];
    }
    delete[] tmp;                                   // free the old array
    arrCapacity = newCapacity;                       // now 20
}
```
So at `i=5`: `newCapacity = 10 * 2 = 20`, all 10 existing elements get copied into the
new 20-slot array, the old array is freed, `arrCapacity` becomes `20`. *Then* control
returns to `add()`, which writes `5` into slot 10 and bumps `arrSize` to `11`. The loop
finishes writing `i=6` through `i=9` into slots 11-14, ending with `arrSize = 15`,
`arrCapacity = 20`.

This is exactly why `push_back`/`add` is called "O(1) **usually**" rather than a flat
O(1): almost every call is a cheap single-slot write, but occasionally — whenever
capacity runs out — one call has to pay an O(n) copy-everything cost to expand.
Averaged over many insertions, this still works out to O(1) *amortized*, but any
individual call can be the expensive one.

## Complexity

| Operation | Complexity | Why |
|---|---|---|
| Access by index | O(1) | Still direct address arithmetic — a list is an array underneath |
| Add at the end | O(1) amortized | Usually a single write; occasionally triggers an O(n) expansion |
| Insert/remove in the middle | O(n) | Elements after the target index must shift, same as [[Array]] |
| Traverse | O(n) | Must visit every element once |
| Sort | O(n log n) (typical) | Enables binary search / two-pointer techniques afterward |
| Expand (triggered internally) | O(n) | Must copy every existing element into the new, larger array |

## Code
- [list.cpp](../codes/cpp/chapter_array_and_linkedlist/list.cpp) — `std::vector` usage:
  init, access, insert/erase, traverse, concatenate, sort.
- [my_list.cpp](../codes/cpp/chapter_array_and_linkedlist/my_list.cpp) — hand-rolled
  `MyList` class implementing a dynamic array from scratch: `add()`, `insert()`,
  `remove()`, `get()`, `set()`, `extendCapacity()`, all exercised in `main()`
  (including the expansion trigger at the 11th `add()` call).

No diagrams for this section (list.md in the source repo is code-only, no `.assets/`
folder).

## Related
- [[Array]]
- [[Linked List]]
- [[Classification of Data Structure]]

## Self-check questions
1. Why is `push_back` described as "O(1) amortized" rather than a flat O(1)?
2. Starting from an empty `MyList` with `arrCapacity = 10` and `extendRatio = 2`, after
   exactly how many `add()` calls does the first expansion trigger, and what does
   `arrCapacity` become afterward?
3. Why does `extendCapacity()` need to copy every old element into the new array
   instead of just extending the existing one in place?
4. Why must you sort a list before applying binary search or the two-pointer
   technique?
5. What's the concrete difference between "list" and "array" as concepts, given that a
   list is usually *implemented* as a dynamic array?
