---
tags:
  - chapter/array-linkedlist
status: taught
date: 2026-08-07
---

# Linked List

## Core idea
A **linked list** is the other physical storage primitive alongside the array (recall
[[Classification of Data Structure]] naming exactly these two). Instead of one
contiguous block, it's a chain of independent **node** objects, each holding a value
plus a **reference (pointer)** to the next node — so the nodes don't need to sit next to
each other in memory at all.

### Why linked lists exist — the motivating problem

In a real system, free memory is often scattered across many small chunks, not one giant
free block. If you need a very large array, the system might not have a single
contiguous region big enough — even if the *total* free memory across the whole system
would be plenty. A linked list sidesteps this entirely: since its nodes don't need to be
adjacent, it can grab whatever small pockets of free memory happen to be available,
wherever they physically are.

### The logical chain vs. the actual memory layout

Diagram: **`linkedlist_definition.png`** shows this split directly. On the left, the
*logical* view: `1 → 3 → 2 → 5 → 4 → None` — a clean, ordered chain, easy to reason
about. On the right, the *actual* memory layout: node `1` sits at one grid address, node
`4` sits somewhere else entirely, nodes `3`, `2`, `5` are scattered further still. The
only thing holding the chain together is that each node's stored pointer happens to
point at wherever the next node landed — the diagram literally draws an arrow from
node `3`'s memory cell diagonally across the grid to where node `2` lives. That arrow
**is** the `next` pointer; it's the only reason the list "knows" `2` comes after `3`.

Each node bundles exactly two things, per the diagram's own labels: **Value** (the data,
e.g. `3`) and **Reference (Pointer) to the next node**. The last node's pointer is
`None`/`nullptr` — nothing follows it. In C++:

```cpp
struct ListNode {
    int val;         // Node value
    ListNode *next;  // Pointer to the next node
    ListNode(int x) : val(x), next(nullptr) {}  // Constructor
};
```

This struct is bigger than a plain `int` — it carries a pointer alongside the value.
**That's the direct trade-off for scattered storage**: a linked list needs extra memory
per element (the pointer) that an array doesn't, because the array gets "next" implied
for free just from being contiguous.

### Building a linked list — worked example

Standard example used throughout: build the chain `1 → 3 → 2 → 5 → 4`. This is two
separate steps.

**Step 1 — create each node independently** (they don't know about each other yet):
```cpp
ListNode* n0 = new ListNode(1);
ListNode* n1 = new ListNode(3);
ListNode* n2 = new ListNode(2);
ListNode* n3 = new ListNode(5);
ListNode* n4 = new ListNode(4);
```
At this point each node's `next` is `nullptr` (set by the constructor) — five separate,
disconnected nodes.

**Step 2 — link them together** by setting each node's `next` pointer:
```cpp
n0->next = n1;   // 1 -> 3
n1->next = n2;   // 3 -> 2
n2->next = n3;   // 2 -> 5
n3->next = n4;   // 5 -> 4
```
Now, starting from `n0` and following `.next` repeatedly reaches every node in order.
This is why **the head node stands in for the whole list** — the entire chain is
referred to as "linked list `n0`," because `n0` is the only reference you need;
everything else is reachable by walking `next` pointers from there.

### Inserting a node — worked trace

Diagram: **`linkedlist_insert_node.png`** — example: insert new node `P` (value `0`)
between `n0` (value 1) and `n1` (value 3), turning `1 → 3` into `1 → 0 → 3`.

Two pointer updates, done in a **specific order**:

1. `P.next = n1` — first, point the *new* node forward at `n1`. At this moment the list
   is briefly unchanged (`n0 → n1`), with `P` dangling off to the side also pointing at
   `n1`.
2. `n0.next = P` — now redirect `n0` to point at `P` instead of `n1` directly.

Final state: `n0 → P → n1 → ...`, i.e. `1 → 0 → 3 → ...`. The order matters: you must
set `P.next` **before** overwriting `n0.next` — otherwise, once `n0.next` is reassigned,
if `P` didn't already know where `n1` was, that rest of the list would become
unreachable (nothing would point to it anymore).

C++ code, [linked_list.cpp:9-14](../codes/cpp/chapter_array_and_linkedlist/linked_list.cpp):
```cpp
void insert(ListNode *n0, ListNode *P) {
    ListNode *n1 = n0->next;  // save n1 before we lose the reference
    P->next = n1;             // step 1
    n0->next = P;              // step 2
}
```

Only **two pointer writes**, no shifting of any other node — that's why linked-list
insertion is **O(1)**, dramatically better than an array's O(n) shift-everything-over
cost. This is the single biggest advantage a linked list has over an array.

### Removing a node — worked trace

Diagram: **`linkedlist_remove_node.png`** — example: delete node `P` (value `0`)
currently sitting between `n0` (1) and `n1` (3), turning `1 → 0 → 3` back into `1 → 3`.

Just **one** pointer update: `n0.next = n1` — skip straight over `P`.

Subtle but important detail the diagram calls out explicitly: after this, `P` **still
physically points at `n1`** (its own `.next` field was never touched) — but that no
longer matters, because nothing in the list points *to* `P` anymore. Starting at `n0`
and walking forward, you'll never land on `P` again; it's been cut out of the reachable
chain even though its own internal pointer is stale-but-intact. Deletion here means
*unreachability*, not necessarily erasing the node's own data.

C++ code, [linked_list.cpp:16-26](../codes/cpp/chapter_array_and_linkedlist/linked_list.cpp):
```cpp
void remove(ListNode *n0) {
    if (n0->next == nullptr) return;
    ListNode *P = n0->next;
    ListNode *n1 = P->next;
    n0->next = n1;   // the one pointer update that matters
    delete P;         // C++-specific: free P's memory (no automatic GC like Java/Python)
}
```
Again — **no shifting**, so removal is **O(1)**, versus an array's O(n).

### Accessing a node — worked trace (why it's slow)

Here's where the trade-off bites. In an array, the address of index `i` was computed
directly with arithmetic (`base + i*size`) — O(1), per [[Array]]. A linked list has no
such shortcut: there's no formula turning "index 3" into a memory address, because
nodes are scattered wherever the allocator happened to put them. **The only way to reach
the i-th node is to start at the head and follow `next` pointers, one hop at a time.**

Worked trace: to reach index 3 (value `5`) in `1 → 3 → 2 → 5 → 4`, you must physically
visit node `1`, follow its pointer to `3`, follow `3`'s pointer to `2`, follow `2`'s
pointer to `5` — **3 hops** to reach index 3. In general, reaching index `i` costs `i`
hops, so accessing the i-th node is **O(n)**.

C++ code, [linked_list.cpp:29-36](../codes/cpp/chapter_array_and_linkedlist/linked_list.cpp):
```cpp
ListNode *access(ListNode *head, int index) {
    for (int i = 0; i < index; i++) {
        if (head == nullptr) return nullptr;
        head = head->next;   // one hop per iteration
    }
    return head;
}
```
The loop body executes exactly `index` times — literally the hop-count from the trace
above.

### Finding a node

[linked_list.cpp:39-48](../codes/cpp/chapter_array_and_linkedlist/linked_list.cpp)
`find()` walks from the head, comparing each node's value against `target`, counting
the index as it goes — same **linear search** concept as [[Array]]'s `find()`, same
**O(n)** cost, just via pointer-hopping instead of index arithmetic.

## Complexity

### Array vs. linked list — direct comparison

| | Array | Linked List |
|---|---|---|
| Storage | Contiguous memory | Scattered memory |
| Capacity | Fixed length | Grows flexibly (allocate + link a new node) |
| Memory per element | Less (no pointer overhead) — but pre-allocated space may go unused | More (value + pointer per node) |
| Access by index | O(1) — direct address formula | O(n) — must hop node-by-node from the head |
| Insert | O(n) — must shift elements | O(1) — just rewire two pointers |
| Remove | O(n) — must shift elements | O(1) — just rewire one pointer |

This table is the clean summary of everything traced above: array and linked list are
**mirror-image trade-offs** — whichever operation is cheap for one is expensive for the
other, because they're built on opposite storage strategies (contiguous vs. scattered).

### Three variants of linked lists

Diagram: **`linkedlist_common_types.png`** shows all three, using the same values
`1, 3, 2, 5, 4` for direct comparison:

1. **Singly linked list** — what's covered above: each node points only forward
   (`next`), and the list ends at `None`.
2. **Circular linked list** — take a singly linked list and make the tail's `next`
   point back to the head instead of `None`. The diagram shows the arrow from `4`
   looping all the way back to `1`. Since there's no fixed "start," any node could
   serve as the entry point.
3. **Doubly linked list** — each node carries **two** pointers: `next` (forward) *and*
   `prev` (backward). The diagram shows arrows going both directions between every
   adjacent pair. This costs more memory per node (an extra pointer) but lets you walk
   the list in either direction — e.g. from `4` you can immediately step back to `5`
   without needing to restart from the head.

C++ struct for a doubly linked node:
```cpp
struct ListNode {
    int val;
    ListNode *next;  // Pointer to the successor node
    ListNode *prev;  // Pointer to the predecessor node
    ListNode(int x) : val(x), next(nullptr), prev(nullptr) {}
};
```

## Code
[linked_list.cpp](../codes/cpp/chapter_array_and_linkedlist/linked_list.cpp) —
`insert()`, `remove()`, `access()`, `find()`, all exercised in `main()` building the
chain `1 → 3 → 2 → 5 → 4`.

Diagrams (from `en/docs/chapter_array_and_linkedlist/linked_list.assets/`):
- `linkedlist_definition.png` — logical chain vs. scattered memory layout
- `linkedlist_insert_node.png` — insert node `P` between `n0` and `n1`, two-pointer trace
- `linkedlist_remove_node.png` — remove node `P`, one-pointer trace, unreachability
- `linkedlist_common_types.png` — singly, circular, doubly linked list side-by-side

## Related
- [[Array]]
- [[Classification of Data Structure]]

### Typical applications
- **Singly linked lists** — build stacks and queues (insert/delete only at one or both
  ends), hash tables (separate chaining: colliding entries stored as a linked list per
  bucket — recall this exact point from [[Classification of Data Structure]]), and
  graphs (adjacency lists: each vertex owns a linked list of its neighbors).
- **Doubly linked lists** — needed whenever you must move in *both* directions
  efficiently: red-black trees / B-trees (accessing a parent node), browser
  forward/back history, and LRU cache eviction (needs fast add/remove from both ends).
- **Circular linked lists** — round-robin CPU scheduling (cycle through a fixed set of
  processes indefinitely), and circular data buffers (e.g. audio/video streaming
  buffers that need to loop back to the start).

## Self-check questions
1. Trace through inserting node `X` (value `9`) between `n1` and `n2` in the chain
   `1 → 3 → 2 → 5 → 4` — what two pointer assignments happen, and in what order?
2. Why must `P.next = n1` happen *before* `n0.next = P` during insertion?
3. After removing node `P` from a list, `P.next` still points to a valid node. Why does
   the list still correctly consider `P` "deleted"?
4. Why is accessing the i-th element O(n) for a linked list but O(1) for an array?
5. Name one use case where a doubly linked list's extra `prev` pointer earns back its
   memory cost.
