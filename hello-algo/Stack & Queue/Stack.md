---
tags:
  - chapter/stack-queue
status: taught
date: 2026-08-07
---

# Stack

## Core idea
A **stack** is a linear data structure that follows the **Last In, First Out (LIFO)**
principle. The book's own analogy is a pile of plates on a table: if you're only
allowed to move one plate at a time, reaching the bottom plate means removing every
plate above it first, one at a time, in the reverse order they were stacked. Whatever
went on *last* comes off *first* — that reversal is the entire meaning of LIFO.

The end where elements go in and out is called the **top**; the other end is the
**bottom**. Adding an element is **push**; removing the top element is **pop**.

### Worked trace of push/pop

Diagram: **`stack_operations.png`** traces a stack starting as `[1, 3, 2]`
(bottom to top) through four operations:

1. `push(5)` → `[1, 3, 2, 5]` — `5` lands on top.
2. `push(4)` → `[1, 3, 2, 5, 4]` — `4` lands on top of that.
3. `pop()` → returns `4`, stack becomes `[1, 3, 2, 5]` — the *most recently pushed*
   element comes off first, not the oldest one.
4. `pop()` → returns `5`, stack becomes `[1, 3, 2]` — back to the starting state.

The order things came *out* was `4`, then `5` — the exact reverse of the order they
went *in* (`5`, then `4`). That reversal is what LIFO means in practice, not just in
definition.

### Why does a stack need its own type at all?

This is a framing point worth sitting with: **a stack can be viewed as a restricted
array or a restricted linked list.** Both of those structures already support
inserting/removing at any position — a stack doesn't add a new *capability* on top of
them. What it does is take an existing structure and hide every operation except
push/pop/peek at one end. The value isn't a new algorithm; it's a narrower *contract*.
By refusing to let any caller touch the middle, a stack guarantees LIFO ordering holds
everywhere it's used — which makes code that depends on that ordering much easier to
reason about, because you never have to check "did something sneak an element in out
of order."

### Common operations — all O(1)

| Method | Description | Time Complexity |
|---|---|---|
| `push()` | Push an element onto the stack (add to top) | O(1) |
| `pop()` | Pop the top element off the stack | O(1) |
| `top()` (`peek()` in some languages) | Read the top element without removing it | O(1) |

All three are O(1) for the same reason: they only ever touch the top slot. There's
never a reason to walk through the middle of a stack, unlike, say, [[Linked List]]'s
`access()`.

C++ usage, [stack.cpp:11-38](../en/codes/cpp/chapter_stack_and_queue/stack.cpp):
```cpp
stack<int> stack;

stack.push(1);   // Push elements
stack.push(3);
stack.push(2);
stack.push(5);
stack.push(4);
// stack is now (bottom to top): [1, 3, 2, 5, 4]

int top = stack.top();      // Access top element -> 4 (does NOT remove it)

stack.pop();                 // Pop top element -> stack becomes [1, 3, 2, 5]

int size = stack.size();     // Get stack length -> 4
bool empty = stack.empty();  // Check if empty -> false
```

**A real C++ gotcha worth calling out explicitly**: `std::stack::pop()` does **not**
return the popped value — unlike, say, a `vector`'s data being readable right up to
the moment you call `pop_back()`. To actually use the value you're removing, you must
call `.top()` first to read it, then call `.pop()` separately to remove it. This
two-step pattern shows up directly in both implementations below.

## Linked list implementation — worked trace

When a linked list backs a stack, the **head node becomes the stack's top**, and the
tail becomes the bottom. This is the exact same head-insertion mechanic from
[[Linked List]], just always applied at one fixed end.

Diagram: **`linkedlist_stack_step1.png`** — starting state: a stack drawn as a
vertical box `[5, 2, 3, 1]` (top to bottom) sitting next to its actual linked-list
representation: `Head node 5 → 2 → 3 → 1 Tail node`. Both drawings are the same data —
just two different ways of visualizing it.

Diagram: **`linkedlist_stack_step2_push.png`** — pushing `4`. Caption: "Push node `4`
to the head of the linked list." After the push: `Head node 4 → 5 → 2 → 3 → 1 Tail
node`. This is the **head insertion method**: every new element becomes the new head,
pointing forward at whatever used to be the head.

Diagram: **`linkedlist_stack_step3_pop.png`** — popping. Caption: "Remove the head
node." Starting from `5 → 2 → 3 → 1`, popping removes node `5` entirely, leaving
`2 → 3 → 1` as the new head-to-tail chain — `2` is now the top.

C++ code, [linkedlist_stack.cpp:36-53](../en/codes/cpp/chapter_stack_and_queue/linkedlist_stack.cpp):
```cpp
void push(int num) {
    ListNode *node = new ListNode(num);
    node->next = stackTop;    // new node points forward at the current top
    stackTop = node;           // new node becomes the top
    stkSize++;
}

int pop() {
    int num = top();           // read the value first (per the C++ gotcha above)
    ListNode *tmp = stackTop;
    stackTop = stackTop->next;  // advance top pointer to the next node
    delete tmp;                 // free the old top node's memory
    stkSize--;
    return num;
}
```
Trace `push(4)` onto `5 → 2 → 3 → 1`: a new node holding `4` is created, its `next` is
set to point at the current `stackTop` (node `5`), and *then* `stackTop` itself is
reassigned to point at the new node `4`. Compare this directly with [[Linked List]]'s
insert-at-head pattern — it's the identical two-step "point the new node forward
first, then redirect the reference" ordering, just with `stackTop` playing the role of
whatever pointer used to reference the old head. `pop()` is the mirror image: read
`top()`'s value, save a temporary reference to the current top node, move `stackTop`
forward to `next`, then free the now-unreachable old top node.

## Array implementation — worked trace

Diagrams: **`array_stack_step1.png`**, **`array_stack_step2_push.png`**,
**`array_stack_step3_pop.png`** show the same push(5) → push(4) → pop() → pop()
sequence as `stack_operations.png` above, but framed explicitly around an array where
**the end of the array is the top of the stack**.

C++ code, [array_stack.cpp:25-42](../en/codes/cpp/chapter_stack_and_queue/array_stack.cpp):
```cpp
void push(int num) {
    stack.push_back(num);    // add at the end — the list's O(1)-usually append
}

int pop() {
    int num = top();          // read the value first
    stack.pop_back();          // remove from the end — O(1), nothing to shift
    return num;
}

int top() {
    if (isEmpty())
        throw out_of_range("Stack is empty");
    return stack.back();
}
```
This is exactly [[List]]'s `push_back`/O(1)-amortized story, applied only at the end —
and that restriction is precisely *why* array-backed stacks work well: a stack never
needs to insert or delete in the *middle* the way a general list might, so it
completely sidesteps [[List]]'s O(n) middle-insert case. It only ever pays the cheap
append-at-end / remove-from-end cost, which is O(1) almost every time and occasionally
O(n) during an expansion (same mechanism as [[List]]'s `extendCapacity()`).

## Comparing the two implementations

**Supported operations**: both implementations support everything a stack contract
needs. The array version additionally *allows* random access (`stack[i]`) — but using
that steps outside the stack's contract and defeats the point of restricting access in
the first place.

**Time efficiency**: the array version generally has better cache locality
(contiguous memory — same reasoning as [[Array]]), so pushes/pops tend to be faster on
average — *except* when a push happens to trigger an expansion, which costs O(n) for
that one call (same spike [[List]] already covered). The linked-list version's
`push()` has to allocate a brand-new node object and rewire a pointer on every single
call, which is a bit more constant overhead per operation — but its performance is
more *consistent*, since it never has that occasional expensive expansion spike.

**Space efficiency**: the array version can waste space through over-provisioned
initial capacity and unused expansion headroom (the same [[List]] trade-off). The
linked-list version wastes space per-node on the `next` pointer (the same
[[Linked List]] trade-off). Neither implementation wins outright — the better choice
depends on the access pattern of the situation it's used in.

## Typical applications

- **Browser back/forward, undo/redo** — every time a new page is opened, the browser
  pushes the previous page onto a stack; clicking "back" is literally a `pop()`.
  Supporting *both* back and forward directions requires two stacks working together
  (one for "back" history, one for "forward" history).
- **Program memory management (the function call stack)** — every function call
  pushes a new **stack frame** holding that call's local variables and return
  address. Recall the space-complexity discussion in [[Iteration and Recursion]]: this
  is exactly why recursion depth costs O(n) space. Descending into recursive calls is
  a sequence of pushes; returning back up out of them (backtracking) is a sequence of
  pops — the call stack *is* a stack in the literal data-structure sense, not just a
  metaphor.

## Complexity

| Operation | Complexity | Why |
|---|---|---|
| `push()` | O(1) | Only touches the top — array end-append or linked-list head-insert |
| `pop()` | O(1) | Only touches the top — array end-remove or linked-list head-remove |
| `top()` / `peek()` | O(1) | Direct read of the top slot, no traversal |

## Code
- [stack.cpp](../en/codes/cpp/chapter_stack_and_queue/stack.cpp) — `std::stack<int>`
  usage: push, top, pop, size, empty.
- [linkedlist_stack.cpp](../en/codes/cpp/chapter_stack_and_queue/linkedlist_stack.cpp) —
  hand-rolled `LinkedListStack` class, head-insertion push/pop.
- [array_stack.cpp](../en/codes/cpp/chapter_stack_and_queue/array_stack.cpp) —
  hand-rolled `ArrayStack` class wrapping a `vector<int>`, end-of-array push/pop.

Diagrams (from `en/docs/chapter_stack_and_queue/stack.assets/`):
- `stack_operations.png` — push(5), push(4), pop(), pop() traced on `[1,3,2]`
- `linkedlist_stack_step1.png` — array-view vs. linked-list-view of the same stack,
  head = top, tail = bottom
- `linkedlist_stack_step2_push.png` — push(4) becomes the new head node
- `linkedlist_stack_step3_pop.png` — pop() removes the head node
- `array_stack_step1.png` / `array_stack_step2_push.png` / `array_stack_step3_pop.png`
  — same push/pop sequence, framed around the end of an array as the stack top

## Related
- [[Array]]
- [[Linked List]]
- [[List]]
- [[Iteration and Recursion]]

## Self-check questions
1. Trace `push(5)` then `push(4)` then `pop()` then `pop()` starting from
   `[1, 3, 2]` (bottom to top) — what's the stack's state after each step, and what
   value does each `pop()` return?
2. Why does `std::stack::pop()` in C++ not return the popped value, and what two-step
   pattern do you need to actually retrieve it?
3. In the linked-list implementation, why does `push()` set `node->next = stackTop`
   *before* reassigning `stackTop = node`, rather than the other way around?
4. Why can a stack's array implementation get away with never needing an O(n)
   middle-insert, even though the general [[List]] dynamic array can?
5. Give one concrete real-world example where a stack (not a queue) is the correct
   structure to model the problem, and explain why LIFO order is what's needed there.
