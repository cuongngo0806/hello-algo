---
tags:
  - chapter/stack-and-queue
status: done
date: 2026-08-10
---

# Stack

## Core idea
A **stack** is a linear data structure that follows a **last-in, first-out (LIFO)**
rule. Think of it as a stack of plates: you can only add or remove one plate at a
time, and always from the top. To reach the bottom plate you must first remove every
plate above it, in reverse order of how they were placed.

Replace "plate" with any element type (integers, characters, objects) and you get the
stack data structure. The end where elements are added/removed is the **top**; the
other end is the **bottom**. The operation that adds to the top is called **push**;
the operation that removes from the top is called **pop**; reading the top element
without removing it is **peek** / `top()`.

Diagram: **`stack_operations.png`** — starts with stack `[1, 3, 2]` (bottom to top).
`push(5)` adds 5 on top. `push(4)` adds 4 on top (top-to-bottom reading is now
`4, 5, 2, 3, 1`). Then two `pop()` calls remove 4, then 5 — exactly reversing the order
they were added. That reversal is the LIFO rule made visual: whatever went in last
comes out first.

## Worked trace of the built-in stack

From [stack.cpp](../codes/cpp/chapter_stack_and_queue/stack.cpp), using C++'s built-in
`std::stack`:

```cpp
stack<int> stack;
stack.push(1);   // stack (bottom→top): [1]
stack.push(3);   // [1, 3]
stack.push(2);   // [1, 3, 2]
stack.push(5);   // [1, 3, 2, 5]
stack.push(4);   // [1, 3, 2, 5, 4]   <- top is 4

int top = stack.top();   // top = 4  (reads only, does not remove)

stack.pop();      // removes 4. stack is now [1, 3, 2, 5]. pop() itself returns nothing.

int size = stack.size();    // size = 4
bool empty = stack.empty(); // empty = false
```

**Important C++-specific detail**: `stack.pop()` has **no return value** — it's void.
To get the popped value you must call `top()` first to read it, *then* call `pop()` to
remove it — exactly what the driver code does. This differs from languages like
Python, where `stack.pop()` both removes and returns the value in a single call.

| Method | Description | Time complexity |
|---|---|---|
| `push()` | insert element at top | O(1) |
| `pop()` | remove top element | O(1) |
| `top()` / `peek()` | read top element without removing | O(1) |

All three are O(1) regardless of stack size, because they only ever touch the top.
That constant-time guarantee is the entire reason the stack abstraction is useful —
even though the same effect could be achieved with a raw array or linked list, the
stack interface simply forbids touching anything except the top, which is exactly
what LIFO logic needs.

## Building a stack from scratch

C++ provides `std::stack` for free, but building one manually shows *why* push/pop/top
are all O(1). Since a stack only allows adding/removing at one end, it can be thought
of as a **restricted array or restricted linked list** — take a normal array or linked
list, which allows insert/delete anywhere, and simply hide every operation except the
ones at one end.

### Implementation 1 — backed by a linked list

Idea: use the **head node of the linked list as the stack's top**. Pushing inserts a
new node at the head ("head insertion"); popping removes the head node.

Diagram: **`linkedlist_stack_step2_push.png`** — shows `push(4)`. On the right, the
actual linked-list structure: before the push, head was `5`, chain `5 → 2 → 3 → 1`.
After `push(4)`, a new node `4` becomes the head with `next` pointing at the old head
`5`, giving chain `4 → 5 → 2 → 3 → 1`. On the left, the same operation drawn as a
stack — 4 lands on top. Both views describe the identical underlying memory
operation; the stack view is just the restricted way of looking at the linked list.

Why the **head** and not the tail? Inserting at the head of a singly linked list is
O(1) — create the node, point its `next` at the current head, update the head
pointer. Inserting at the tail would need walking the whole list to find the last
node (O(n)) unless a separate tail pointer is tracked — and even then, *removing*
from the tail of a singly linked list stays O(n), since finding the new
second-to-last node requires a full walk. Using the head sidesteps all of that.

Code, [linkedlist_stack.cpp](../codes/cpp/chapter_stack_and_queue/linkedlist_stack.cpp):

```cpp
class LinkedListStack {
  private:
    ListNode *stackTop;  // head node doubles as the stack top
    int stkSize;

  public:
    /* push */
    void push(int num) {
        ListNode *node = new ListNode(num);
        node->next = stackTop;   // new node points at old head
        stackTop = node;         // new node becomes the head
        stkSize++;
    }

    /* pop */
    int pop() {
        int num = top();
        ListNode *tmp = stackTop;
        stackTop = stackTop->next;  // head moves to the second node
        delete tmp;                  // free the old head's memory
        stkSize--;
        return num;
    }

    /* peek */
    int top() {
        if (isEmpty()) throw out_of_range("stack is empty");
        return stackTop->val;
    }
};
```

Worked trace with `1, 3, 2, 5, 4`:

1. `push(1)`: new node `1`, `1->next = nullptr`, head = `1`. Chain: `1`.
2. `push(3)`: new node `3`, `3->next = 1`, head = `3`. Chain: `3 → 1`.
3. `push(2)`: new node `2`, `2->next = 3`, head = `2`. Chain: `2 → 3 → 1`.
4. `push(5)`: chain becomes `5 → 2 → 3 → 1`.
5. `push(4)`: chain becomes `4 → 5 → 2 → 3 → 1`.
6. `pop()`: reads `top() = 4`, sets head `= stackTop->next = 5`, deletes the old node
   holding `4`. Chain is now `5 → 2 → 3 → 1`. Returns `4`.

Every step touches only the head — no walking through the rest of the list — so push
and pop are both O(1), matching the table above.

### Implementation 2 — backed by an array

Idea: use the **tail end of the array as the stack's top**. Pushing appends an
element at the end of the array; popping removes the last element.

Diagram: **`array_stack_step2_push.png`** — shows `push(4)` on an array-backed stack.
Before the push, the array holds `1, 3, 2, 5` (index 0–3, with index 3 — value `5` —
as the "tail element"). After `push(4)`, the value `4` is written into the next free
slot (index 4), which becomes the new tail. On the left, the same operation drawn as
a stack: 4 lands on top. Again, both views are the same underlying operation —
appending to the end of the array — seen through two different lenses.

Code, [array_stack.cpp](../codes/cpp/chapter_stack_and_queue/array_stack.cpp):

```cpp
class ArrayStack {
  private:
    vector<int> stack;   // dynamic array

  public:
    /* push */
    void push(int num) {
        stack.push_back(num);   // append to the end
    }

    /* pop */
    int pop() {
        int num = top();
        stack.pop_back();       // remove the last element
        return num;
    }

    /* peek */
    int top() {
        if (isEmpty()) throw out_of_range("stack is empty");
        return stack.back();
    }
};
```

Notice the deliberate use of `vector<int>` — a **dynamic** array — rather than a
fixed-size C-style array. If push kept happening indefinitely and the backing array
had a fixed size, eventually there would be no room left. A dynamic array
automatically reallocates a bigger block and copies the old elements over when it
runs out of room, so the caller of `push()` never has to think about capacity.

Worked trace, same numbers:

1. `push(1)`: array = `[1]`
2. `push(3)`: array = `[1, 3]`
3. `push(2)`: array = `[1, 3, 2]`
4. `push(5)`: array = `[1, 3, 2, 5]`
5. `push(4)`: array = `[1, 3, 2, 5, 4]`
6. `pop()`: reads `top() = stack.back() = 4`, then `stack.pop_back()` removes it.
   Array = `[1, 3, 2, 5]`. Returns `4`.

Since both push and pop only touch the last element, both are O(1) — same conclusion
as the linked-list version, reached through a completely different mechanism
(contiguous memory + tail index, instead of pointers + head node).

## Comparing the two implementations

**Supported operations** — both support the full stack interface. The array version
additionally allows random access (reading any index directly), but that's outside
the definition of a stack, so it's rarely used in practice.

**Time efficiency** — the array version usually wins on raw speed, because its
elements sit in one contiguous block of memory, which is friendly to CPU caching
(cache locality). Catch: if a `push()` happens to be the one that exceeds the array's
current capacity, it triggers a resize — allocate a bigger block, copy every existing
element over — making *that particular* push O(n) instead of O(1). This is
low-frequency (capacity typically doubles each resize), so the *average* cost across
many pushes still comes out to O(1) — this is called **amortized O(1)**.

The linked-list version never has this resize spike — its "growth" is just
allocating one more node, always O(1), so it's more *consistently* O(1) push after
push. Trade-off: each push must allocate a new node object and rewire a pointer,
which per-operation tends to be slower than an array append for primitive types
like `int`.

**Space efficiency** — a dynamic array typically over-allocates (reserves more
capacity than currently needed, multiplying capacity by some factor like 2x on
resize), which can waste memory. A linked-list node must additionally store a
pointer alongside each value — extra memory per element. Neither is a clear winner;
which one saves more memory depends on the specific situation (element count,
element size, language overhead).

## Complexity

| Operation | Array-backed | Linked-list-backed |
|---|---|---|
| push | O(1) amortized (O(n) on resize) | O(1) consistently |
| pop | O(1) | O(1) |
| top/peek | O(1) | O(1) |

## Code
- [stack.cpp](../codes/cpp/chapter_stack_and_queue/stack.cpp) — `std::stack` usage,
  push/top/pop/size/empty, all exercised in `main()`.
- [linkedlist_stack.cpp](../codes/cpp/chapter_stack_and_queue/linkedlist_stack.cpp) —
  `LinkedListStack` class, head-insertion push/pop.
- [array_stack.cpp](../codes/cpp/chapter_stack_and_queue/array_stack.cpp) —
  `ArrayStack` class, tail-append push/pop backed by `vector<int>`.

Diagrams (from `docs/chapter_stack_and_queue/stack.assets/`):
- `stack_operations.png` — push/pop sequence on a stack, LIFO visualized
- `linkedlist_stack_step1.png`, `linkedlist_stack_step2_push.png`,
  `linkedlist_stack_step3_pop.png` — linked-list-backed stack, push and pop traced
  against the actual pointer structure
- `array_stack_step1.png`, `array_stack_step2_push.png`,
  `array_stack_step3_pop.png` — array-backed stack, push and pop traced against the
  actual array layout

## Related
- [[Linked List]] — the head-insertion technique used by the linked-list-backed stack
- [[Array]] — the tail-append technique used by the array-backed stack

### Typical applications
- **Browser back/forward, undo/redo in software.** Every new page visited gets pushed
  onto a stack; "back" pops it off. Supporting both back *and* forward needs two
  stacks working together.
- **Program memory management (call stack).** Every function call pushes a new
  **stack frame** (local variables, return address) onto the program's call stack. In
  recursion, "going deeper" keeps pushing frames; "returning back up" keeps popping
  them — this is why recursion's memory cost is proportional to recursion depth, and
  why very deep recursion can overflow the stack.

## Self-check questions
1. Trace `push(1)`, `push(3)`, `push(2)` on an array-backed stack, then `pop()` twice —
   what is the array's contents after each step, and what does each `pop()` return?
2. Why does the linked-list-backed stack use the **head** as the top instead of the
   tail?
3. In C++, why must you call `top()` *before* `pop()` if you want the popped value?
4. What does "amortized O(1)" mean for the array-backed stack's `push()`, and when
   does an individual push actually cost O(n)?
5. Name one real-world system that uses a stack, and explain which operation (push or
   pop) corresponds to which user action.

## Chapter 5 Quiz Results

**Date**: 2026-08-10 | **Score**: 4/7 | **Status**: passed

**Questions answered correctly:**
- Q1 ✅ Stack hand-trace: `push(10,20,30), pop(), push(40), top()` returns 40
- Q4 ✅ C++ `std::stack::pop()` returns void — correct pattern is `top()` then `pop()`
- Q6 ✅ Array-backed stack = amortized O(1) (vector resize), linked-list queue = consistent O(1), circular-array deque = consistent O(1)

**Questions missed (stack-related):**
- None specific to stack — all stack questions answered correctly

**Key takeaways from missed questions (deque-related, see [[Deque]]):**
- `popLast` on singly linked list is O(n) — this forces doubly linked list for deque
- Circular array `index()` helper wraps negative values via `(i + capacity) % capacity`
- Undo mapping: `pushLast` = record, `popLast` = undo (LIFO same end), `popFirst` = discard oldest
