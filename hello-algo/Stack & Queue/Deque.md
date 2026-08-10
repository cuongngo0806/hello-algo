---
tags:
  - chapter/stack-and-queue
status: done
date: 2026-08-10
---

# Deque (Double-ended Queue)

## Core idea
A **deque** (double-ended queue, pronounced "deck") is a linear data structure
that generalizes both the [[Stack]] and the [[Queue]]. A regular queue only
allows adding at the rear and removing from the front. A regular stack only
allows both operations at the same end (the top). A deque allows **adding and
removing at both ends** — front *and* rear.

Diagram: **`deque_operations.png`** — starts with deque `[3, 2, 5]`
(front-to-rear). Four operations are shown: `push_last(4)` adds `4` to the
rear, `push_first(1)` adds `1` to the front, `pop_last()` removes from the
rear, and `pop_first()` removes from the front. Every end is open for both
insertion and removal.

This means a deque can serve as a stack (use only one end), a queue (add at one
end, remove from the other), or something more flexible than either.

| Method | Description | Time complexity |
|---|---|---|
| `push_first()` | Add element to front | O(1) |
| `push_last()` | Add element to rear | O(1) |
| `pop_first()` | Remove front element | O(1) |
| `pop_last()` | Remove rear element | O(1) |
| `peek_first()` | Access front element | O(1) |
| `peek_last()` | Access rear element | O(1) |

Compare to [[Queue]] (3 operations) and [[Stack]] (3 operations) — the deque
has 6 operations, essentially the union of both.

## Worked trace of the built-in deque

From [deque.cpp](../codes/cpp/chapter_stack_and_queue/deque.cpp), using C++'s
`std::deque`:

```cpp
deque<int> deque;

deque.push_back(2);   // deque (front→rear): [2]
deque.push_back(5);   // [2, 5]
deque.push_back(4);   // [2, 5, 4]
deque.push_front(3);  // [3, 2, 5, 4]   ← 3 inserted at front
deque.push_front(1);  // [1, 3, 2, 5, 4] ← 1 inserted at front

int front = deque.front();  // front = 1
int back  = deque.back();   // back  = 4

deque.pop_front();  // removes 1. deque is now [3, 2, 5, 4]
deque.pop_back();   // removes 4. deque is now [3, 2, 5]
```

Key difference from `std::queue`: `std::deque` supports both
`push_front`/`pop_front` AND `push_back`/`pop_back`. The standard queue only
allows push at the back and pop from the front.

Same C++ quirk as `std::stack` and `std::queue`: `pop_front()` and `pop_back()`
are **void** — they do not return the removed value. You must call `front()` or
`back()` first to read it, then pop.

## Building a deque from scratch

### Implementation 1 — backed by a doubly linked list

Recall from [[Queue]] that the linked-list-backed queue used a **singly** linked
list with `front` and `rear` pointers. That was sufficient because insertion
only happened at the tail and removal only at the head.

A deque needs to insert *and* remove at **both** ends. Removing from the tail of
a singly linked list is O(n) — you need to find the second-to-last node to
update, which requires walking the entire chain from the head. The fix: use a
**doubly linked list**, where each node has both `next` and `prev` pointers.
This lets you step backward from the tail in O(1).

From [linkedlist_deque.cpp](../codes/cpp/chapter_stack_and_queue/linkedlist_deque.cpp):

```cpp
struct DoublyListNode {
    int val;
    DoublyListNode *next;  // successor
    DoublyListNode *prev;  // predecessor
    DoublyListNode(int val) : val(val), prev(nullptr), next(nullptr) {}
};
```

The class tracks `front` (head) and `rear` (tail), same as the queue. But now
`push` and `pop` each handle both directions via a single method with a boolean
flag:

```cpp
class LinkedListDeque {
  private:
    DoublyListNode *front, *rear;
    int queSize = 0;

  public:
    /* push to front or rear */
    void push(int num, bool isFront) {
        DoublyListNode *node = new DoublyListNode(num);
        if (isEmpty())
            front = rear = node;
        else if (isFront) {
            front->prev = node;   // old head's prev points to new node
            node->next = front;   // new node's next points to old head
            front = node;         // update head
        } else {
            rear->next = node;    // old tail's next points to new node
            node->prev = rear;    // new node's prev points to old tail
            rear = node;          // update tail
        }
        queSize++;
    }
    void pushFirst(int num) { push(num, true); }
    void pushLast(int num)  { push(num, false); }

    /* pop from front or rear */
    int pop(bool isFront) {
        if (isEmpty()) throw out_of_range("Queue is empty");
        int val;
        if (isFront) {
            val = front->val;
            DoublyListNode *fNext = front->next;
            if (fNext != nullptr) {
                fNext->prev = nullptr;
                front->next = nullptr;
            }
            delete front;
            front = fNext;
        } else {
            val = rear->val;
            DoublyListNode *rPrev = rear->prev;
            if (rPrev != nullptr) {
                rPrev->next = nullptr;
                rear->prev = nullptr;
            }
            delete rear;
            rear = rPrev;
        }
        queSize--;
        return val;
    }
    int popFirst() { return pop(true); }
    int popLast()  { return pop(false); }
};
```

**Worked trace** — following the driver code in `main()`:

1. `pushLast(3)`: empty, so `front = rear = node(3)`. Chain: `3`.
2. `pushLast(2)`: `rear->next = node(2)`, `node(2)->prev = rear(3)`,
   `rear = node(2)`. Chain: `3 ↔ 2`.
3. `pushLast(5)`: chain becomes `3 ↔ 2 ↔ 5`.
4. `pushLast(4)`: chain becomes `3 ↔ 2 ↔ 5 ↔ 4`.
5. `pushFirst(1)`: `front->prev = node(1)`, `node(1)->next = front(3)`,
   `front = node(1)`. Chain: `1 ↔ 3 ↔ 2 ↔ 5 ↔ 4`.
6. `popLast()`: reads `rear->val = 4`, sets `rPrev = node(5)`, clears links,
   deletes old rear, `rear = node(5)`. Chain: `1 ↔ 3 ↔ 2 ↔ 5`. Returns `4`.
7. `popFirst()`: reads `front->val = 1`, sets `fNext = node(3)`, clears links,
   deletes old front, `front = node(3)`. Chain: `3 ↔ 2 ↔ 5`. Returns `1`.

The `prev` pointer is exactly what makes `popLast()` possible in O(1). Compare
to the singly-linked queue: there, removing the tail would require walking the
entire chain from head to find the second-to-last node (O(n)), because there is
no `prev` pointer. The doubly-linked list pays for this with extra memory per
node (one additional pointer).

Diagram: **`linkedlist_deque_step2_push_last.png`** — shows `pushLast(4)` on
chain `3 ↔ 2 ↔ 5`. New node `4` is appended after the old tail `5`:
`5->next = 4`, `4->prev = 5`, and `rear` updates to `4`. Same as the queue's
linked-list push — appending at the tail.

Diagram: **`linkedlist_deque_step3_push_first.png`** — shows `pushFirst(1)` on
chain `3 ↔ 2 ↔ 5 ↔ 4`. New node `1` is prepended before the old head `3`:
`3->prev = 1`, `1->next = 3`, and `front` updates to `1`. Same as the stack's
linked-list push — prepending at the head.

### Implementation 2 — backed by a circular array

The deque's circular array implementation extends the queue's circular array.
Recall from [[Queue]] that the array-backed queue tracked `front` (index of the
front element) and `queSize` (element count), computing
`rear = (front + queSize) % capacity` for enqueue. The deque adds the reverse
operation: **enqueue at the front**.

From [array_deque.cpp](../codes/cpp/chapter_stack_and_queue/array_deque.cpp):

```cpp
class ArrayDeque {
  private:
    vector<int> nums;
    int front;
    int queSize;

  public:
    ArrayDeque(int capacity) {
        nums.resize(capacity);
        front = queSize = 0;
    }

    /* helper: circular index */
    int index(int i) {
        return (i + capacity()) % capacity();
    }

    /* enqueue at front */
    void pushFirst(int num) {
        if (queSize == capacity()) { cout << "Full" << endl; return; }
        front = index(front - 1);   // move front BACKWARD (wraps via modulo)
        nums[front] = num;
        queSize++;
    }

    /* enqueue at rear — same as the queue's push */
    void pushLast(int num) {
        if (queSize == capacity()) { cout << "Full" << endl; return; }
        int rear = index(front + queSize);
        nums[rear] = num;
        queSize++;
    }

    /* dequeue from front — same as the queue's pop */
    int popFirst() {
        int num = peekFirst();
        front = index(front + 1);
        queSize--;
        return num;
    }

    /* dequeue from rear — the new operation */
    int popLast() {
        int num = peekLast();
        queSize--;   // just shrink size; rear recomputes automatically
        return num;
    }

    int peekLast() {
        if (isEmpty()) throw out_of_range("Deque is empty");
        int last = index(front + queSize - 1);
        return nums[last];
    }
};
```

The key new trick is `pushFirst`: it moves `front` **backward** by one slot
using `index(front - 1)`. The `index()` helper adds `capacity()` before doing
the modulo, so if `front` is currently 0, `index(0 - 1) = index(-1) =
(-1 + capacity) % capacity = capacity - 1` — it wraps to the **last** slot of
the array, moving backward in the circle. This is the reverse of how
`popFirst` moves `front` forward.

Diagram: **`array_deque_step2_push_last.png`** — shows `pushLast(4)` on
`[3, 2, 5]` with `front = 0`, `queSize = 3`. `rear = (0 + 3) % 10 = 3`, so
`4` is written at index 3, `queSize` becomes 4. Identical to the regular
queue's push.

Diagram: **`array_deque_step3_push_first.png`** — shows `pushFirst(1)` on
`[3, 2, 5, 4]` with `front = 0`, `queSize = 4`.
`front = index(0 - 1) = (0 - 1 + 10) % 10 = 9`. So `1` is written at index 9
(the last slot), and `front` is now 9. The deque's elements now span indices
9, 0, 1, 2, 3 (wrapping around), reading front-to-rear as `1, 3, 2, 5, 4`.
This demonstrates the circular nature — the front can move backward past
index 0 and wrap to the end of the array, exactly mirroring how the rear can
move forward past the last index and wrap to 0.

**Worked trace of `popLast`**: After `pushFirst(1)`, `front = 9`,
`queSize = 5`, elements are `[1, 3, 2, 5, 4]` at indices 9, 0, 1, 2, 3. Now
`popLast()`: `peekLast()` computes `last = index(9 + 5 - 1) = index(13) =
13 % 10 = 3`, reads `nums[3] = 4`. Then `queSize` decrements to 4. The rear is
now implicitly at `index(9 + 4) = 3` — but since we only decremented size
without touching index 3, the value `4` is still physically there. It is just
no longer considered part of the deque.

## Why a deque instead of a stack or queue?

The documentation gives a concrete example: the **"undo" feature** in software.
You would normally model undo with a stack — push each action, pop to undo. But
most software limits the undo history (e.g., only 50 steps). When the stack is
full, you need to delete the **oldest** entry — the one at the **bottom** of the
stack. A regular stack cannot do this (it only operates on the top). A deque
can: it keeps the LIFO undo behavior at one end, while also allowing removal
from the other end when the history limit is reached.

More generally: any situation where you need "mostly stack" or "mostly queue"
behavior but occasionally need the opposite end too is a natural fit for a
deque.

## Comparison of the two implementations

Same trade-offs as the [[Stack]] and [[Queue]]: array-backed has better cache
locality but fixed capacity; doubly-linked-list-backed is consistently O(1) but
uses more memory per node (two pointers instead of one for the queue's singly
linked list, or none for the array).

## Complexity

| Operation | Doubly-linked-list-backed | Circular-array-backed |
|---|---|---|
| pushFirst | O(1) | O(1), capacity-limited |
| pushLast | O(1) | O(1), capacity-limited |
| popFirst | O(1) | O(1) |
| popLast | O(1) | O(1) |
| peekFirst | O(1) | O(1) |
| peekLast | O(1) | O(1) |

## Code
- [deque.cpp](../codes/cpp/chapter_stack_and_queue/deque.cpp) — `std::deque`
  usage, push_front/push_back/front/back/pop_front/pop_back, all exercised in
  `main()`.
- [linkedlist_deque.cpp](../codes/cpp/chapter_stack_and_queue/linkedlist_deque.cpp) —
  `LinkedListDeque` class, doubly-linked-list-backed push/pop at both ends.
- [array_deque.cpp](../codes/cpp/chapter_stack_and_queue/array_deque.cpp) —
  `ArrayDeque` class, circular-array-backed push/pop at both ends via modulo.

Diagrams (from `en/docs/chapter_stack_and_queue/deque.assets/`):
- `deque_operations.png` — push/pop at both ends visualized
- `linkedlist_deque_step1.png` through `linkedlist_deque_step5_pop_first.png` —
  doubly-linked-list deque, push and pop traced at both ends
- `array_deque_step1.png` through `array_deque_step5_pop_first.png` —
  circular-array deque, push and pop traced against `front`/`queSize` index
  bookkeeping

## Related
- [[Stack]] — LIFO subset of deque behavior (one-end operations only)
- [[Queue]] — FIFO subset of deque behavior (opposite-end operations only)
- [[Linked List]] — doubly linked list variant used by the linked-list-backed
  deque

### Typical applications
- **Undo with limited history** — stack-like LIFO undo at the top, with the
  ability to discard the oldest entry from the bottom when the history limit is
  reached. A regular stack cannot delete from the bottom; a deque can.
- **Any scenario combining stack and queue needs** — a deque can implement all
  application scenarios of both stacks and queues, while providing greater
  flexibility for cases where both ends need occasional access.

## Self-check questions
1. Why does the deque's linked-list implementation require a **doubly** linked
   list, while the queue's linked-list implementation works with a **singly**
   linked list?
2. In the circular-array deque, `pushFirst` computes
   `front = index(front - 1)`. If `front = 0` and `capacity = 10`, what value
   does `front` become? Why does this work?
3. How does `popLast()` in the array-backed deque work without storing a
   separate `rear` variable?
4. Explain why a deque can replace a stack for implementing "undo with limited
   steps" but a plain stack cannot.
5. Name all six O(1) deque operations and state which ones overlap with the
   stack's interface and which overlap with the queue's interface.

## Chapter 5 Quiz Results

**Date**: 2026-08-10 | **Score**: 4/7 | **Status**: passed

**Questions missed (deque-related):**
- Q3 ❌ **Why doubly linked list?** Answered "more operations" — too vague. Correct:
  `popLast` specifically needs `prev` pointer. Removing tail of singly linked list is
  O(n) (must walk from head to find second-to-last node). Doubly linked gives
  `rear->prev` in O(1). Queue never removes from tail, so singly linked suffices.
- Q5 ❌ **Circular array index wrapping.** Answered "front = -2" — impossible.
  `index()` helper computes `(i + capacity) % capacity`, so `index(0-1)` with
  capacity 8 = `(-1+8)%8 = 7`. Front wraps to valid index, never goes negative.
  Correct trace: after `pushFirst(5)` front=7, after `pushFirst(1)` front=6.
  Occupied: indices 6, 7, 0, 1, 2.
- Q7 ❌ **Undo deque mapping.** Correct mapping: `pushLast` = record new action
  (rear end), `popLast` = undo most recent (same rear end, LIFO), `popFirst` =
  discard oldest (front end, when 100-step limit hit). Key insight: record and undo
  use the SAME end (rear) — that's the stack behavior. The deque's extra power is
  `popFirst` at the opposite end.

**Questions answered correctly (deque-related):**
- None of the deque-specific questions were answered correctly

See [[Stack]] for full quiz breakdown.
