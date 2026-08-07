---
tags:
  - chapter/stack-queue
status: taught
date: 2026-08-07
---

# Deque

## Core idea
A **deque** (pronounced "deck," short for double-ended queue) removes the one-directional
restriction both [[Stack]] and [[Queue]] impose. A stack only ever touches one end (the
top). A queue can push at one end (the rear) and pop only at the other (the front), but
never the reverse. A deque lifts both restrictions at once: elements can be pushed *and*
popped at **either** end.

That gives six operations instead of a stack/queue's three:

| Method | Description | Time Complexity |
|---|---|---|
| `push_first()` | Add an element at the front | O(1) |
| `push_last()` | Add an element at the rear | O(1) |
| `pop_first()` | Remove the front element | O(1) |
| `pop_last()` | Remove the rear element | O(1) |
| `peek_first()` | Read the front element without removing it | O(1) |
| `peek_last()` | Read the rear element without removing it | O(1) |

Diagram: **`deque_operations.png`** — a deque `[3, 2, 5, 4]` (front to rear) with four
arrows drawn at the two ends: `Pop first` / `Push first` at the front end, `Push last` /
`Pop last` at the rear end. The point is purely structural: operations exist at *both*
ends simultaneously — the one-sentence summary of what makes a deque different from a
stack (one end only) or a queue (push one end, pop the other only).

C++'s `std::deque<int>` already implements exactly this:
```cpp
deque<int> deque;

deque.push_back(2);    // push_last  -- add at rear
deque.push_back(5);
deque.push_front(3);   // push_first -- add at front
deque.push_front(1);
// deque is now (front to rear): [1, 3, 2, 5]

int front = deque.front();   // peek_first -> 1
int back = deque.back();     // peek_last  -> 5

deque.pop_front();   // pop_first -> removes 1
deque.pop_back();    // pop_last  -> removes 5
```
Note the standard library uses `push_back`/`push_front`/`pop_back`/`pop_front` instead
of the book's `push_last`/`push_first`/`pop_last`/`pop_first` naming — same six
operations, different verb pairs. The hand-rolled classes below use the book's naming.

## Linked list implementation — worked trace

Because both ends must be reachable, a deque cannot reuse [[Queue]]'s singly linked list
(only a `next` pointer, so it can only walk forward). It needs a **doubly linked list**:
every node carries both `next` (forward) and `prev` (backward), so the tail is reachable
from the head *and* the head from the tail without walking the whole chain.

Diagram: **`linkedlist_deque_step1.png`** — setup: the same operations diagram on the
left, paired with the actual doubly-linked-list structure on the right: `Head node
3 ⇄ 2 ⇄ 5 Tail node`, where `⇄` is a pair of pointers (a `next` arrow forward, a `prev`
arrow backward) between adjacent nodes. Head = front, tail = rear — same convention as
[[Queue]].

Diagram: **`linkedlist_deque_step2_push_last.png`** — `push_last(4)`. Caption: "Push
node 4 to the tail of the linked list." Result: `Head 3 ⇄ 2 ⇄ 5 ⇄ 4 Tail`. Identical in
mechanism to [[Queue]]'s tail-insertion — attach the new node after the current tail,
move the tail pointer to it. The only extra step versus a singly linked queue: the new
node's `prev` also gets wired back to the old tail.

Diagram: **`linkedlist_deque_step3_push_first.png`** — `push_first(1)`. Caption: "Push
node 1 to the head of the linked list." Result: `Head 1 ⇄ 3 ⇄ 2 ⇄ 5 ⇄ 4 Tail`. This is
the operation [[Queue]] could **never** do — a singly linked queue has no way to insert
before its own head, because nothing points backward from the old head. Here it's
possible precisely because of the `prev` pointer: new node `1` becomes head, its `next`
points at old-head `3`, and `3`'s `prev` gets wired back to `1`.

Diagram: **`linkedlist_deque_step4_pop_last.png`** — `pop_last()`. Caption: "Remove the
tail node." Starting from `1 ⇄ 3 ⇄ 2 ⇄ 5 ⇄ 4`, removes node `4`, leaving `1 ⇄ 3 ⇄ 2 ⇄ 5`
with `5` as new tail. Again an operation [[Queue]] could never do (no backward pointer
to reach the second-to-last node from the tail) — here `5`'s `next` is set to `nullptr`
and the tail pointer moves back to `5` using the `prev` pointer it already had.

Diagram: **`linkedlist_deque_step5_pop_first.png`** — `pop_first()`. Caption: "Remove
the head node." Starting from `3 ⇄ 2 ⇄ 5`, removes node `3`, leaving `2 ⇄ 5` with `2` as
new head. This part *is* something [[Queue]] could already do — head-removal only ever
needs the `next` pointer, not `prev`.

C++ code, [linkedlist_deque.cpp](../en/codes/cpp/chapter_stack_and_queue/linkedlist_deque.cpp):
```cpp
void push(int num, bool isFront) {
    DoublyListNode *node = new DoublyListNode(num);
    if (isEmpty())               // first element: it's both front and rear at once
        front = rear = node;
    else if (isFront) {          // push_first branch
        front->prev = node;
        node->next = front;
        front = node;
    } else {                     // push_last branch
        rear->next = node;
        node->prev = rear;
        rear = node;
    }
    queSize++;
}
```
Trace `push(1, true)` — `push_first(1)` — onto `3 ⇄ 2 ⇄ 5 ⇄ 4` (front=`3`, rear=`4`):
not empty and `isFront` is true, so the middle branch runs. `front->prev = node` sets
`3`'s `prev` to the new node `1`. `node->next = front` sets `1`'s `next` to `3`.
Finally `front = node` moves the front pointer to `1`. Order matters for the same
reason as [[Linked List]]'s insertion — the new node must be wired into the chain
(both directions) *before* `front` is reassigned, or the only reference to the old head
would be lost mid-operation.

```cpp
int pop(bool isFront) {
    if (isEmpty()) throw out_of_range("Deque is empty");
    int val;
    if (isFront) {                      // pop_first branch
        val = front->val;
        DoublyListNode *fNext = front->next;
        if (fNext != nullptr) {
            fNext->prev = nullptr;
            front->next = nullptr;
        }
        delete front;
        front = fNext;
    } else {                            // pop_last branch
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
```
Trace `pop(false)` — `pop_last()` — on `1 ⇄ 3 ⇄ 2 ⇄ 5 ⇄ 4` (front=`1`, rear=`4`):
`isFront` false, so the `pop_last` branch runs. `val = rear->val` reads `4` first.
`rPrev = rear->prev` is node `5`. Since `rPrev` isn't null, both directions are severed:
`rPrev->next = nullptr` (so `5` no longer points at the soon-to-be-freed `4`), and
`rear->prev = nullptr` (cleanup on the node about to be freed). `delete rear` frees
node `4`, then `rear = rPrev` moves rear back to `5`. Result: `1 ⇄ 3 ⇄ 2 ⇄ 5`, rear=`5`
— matching step 4's diagram exactly.

## Array implementation — worked trace

Just like [[Queue]]'s array implementation avoided O(n) shifts with a circular array
(`front` index + `size` counter), the deque reuses that exact trick — but `front` now
needs to move in **both** directions (left for `push_first`/`pop_first`, right for
`push_last`/`pop_last`), not just rightward like a plain queue.

Diagram: **`array_deque_step1.png`** — setup, directly parallel to [[Queue]]'s
`array_queue_step1.png`. Array holds `[3, 2, 5]`, `front` points at index of value `3`,
`rear` points one slot past `5`, same relationship: `rear = front + size`.

The new piece is the `index()` helper, [array_deque.cpp](../en/codes/cpp/chapter_stack_and_queue/array_deque.cpp):
```cpp
int index(int i) {
    // Perform modulo to achieve a circular array
    return (i + capacity()) % capacity();
}
```

Diagram: **`array_deque_step2_push_last.png`** — `push_last(4)`. Caption: "1. Update the
variable rear. 2. Set the element to rear. 3. Increase size by 1." Identical to
[[Queue]]'s `push()` — nothing new on this end, since rear only ever moves rightward.

Diagram: **`array_deque_step3_push_first.png`** — `push_first(1)`. Caption: "1. Decrease
front by 1. 2. Set the element to front. 3. Increase size by 1." This is the genuinely
new direction — `front` moves **left**, something [[Queue]]'s array implementation
never needed.

```cpp
void pushFirst(int num) {
    if (queSize == capacity()) { cout << "Deque is full" << endl; return; }
    front = index(front - 1);   // move front left, wrapping if needed
    nums[front] = num;
    queSize++;
}
```

**Worked trace of the left-wraparound**, using `capacity = 10`, suppose `front = 0`
(front element sits at physical index 0, nothing free to its immediate left inside
bounds). Call `push_first(1)`:
```
front = index(front - 1)
      = index(0 - 1)
      = index(-1)
      = (-1 + 10) % 10
      = 9 % 10
      = 9
```
The new element is written at physical index `9` — the *last* slot — even though this
is conceptually a front insertion. This is exactly why `index()` adds `capacity()`
before the modulo: in C++, `-1 % 10` evaluates to `-1`, not `9` (C++'s `%` does **not**
wrap negative numbers into a positive range the way Python's does). Adding `capacity()`
first — `(-1 + 10) = 9` — guarantees a non-negative value going into `% capacity()`, so
the wraparound lands correctly at index `9` instead of producing an invalid negative
array index. This is the one genuinely new trick in the deque's array implementation
that neither [[Queue]] nor [[Stack]] needed, since they only ever moved their index
pointers in one direction.

Diagram: **`array_deque_step4_pop_last.png`** — `pop_last()`. Caption: "1. Decrease
size by 1." That's the entire operation — no pointer move at all, because `rear` isn't
stored as its own variable; it's always *computed* as `front + size - 1` (via
`peekLast()`'s `index(front + queSize - 1)`). "Removing" the last element just shrinks
`size` by one — the value stays physically in memory, now outside the valid
`[front, front+size-1]` range, the same "logically gone, physically untouched" pattern
[[Queue]] used for its `pop()`.

Diagram: **`array_deque_step5_pop_first.png`** — `pop_first()`. Caption: "1. Increase
front by 1. 2. Decrease size by 1." Identical to [[Queue]]'s `pop()` — this direction
was already solved there, since `front` moving rightward is the queue's normal case.

```cpp
int popFirst() {
    int num = peekFirst();
    front = index(front + 1);
    queSize--;
    return num;
}

int popLast() {
    int num = peekLast();
    queSize--;   // no pointer to move -- rear is always derived from front + size
    return num;
}
```

## Typical applications

A deque essentially combines what a stack and a queue can each do, adding flexibility
neither has alone, so it naturally covers use cases needing characteristics of both.

Concrete example: **software undo functionality**. A plain LIFO stack (per [[Stack]])
is the obvious fit for undo, since "undo the most recent action" is exactly a `pop()`.
But in practice undo history is almost always **capped** — an editor might keep only
the last 50 actions, not an unbounded history. Once that cap is hit, adding a new
action means the *oldest* saved action (sitting at the bottom of the stack) must be
evicted — "remove from the bottom" is a `pop_first()`-style operation a plain stack
cannot do (a stack only ever touches its top). A deque fits perfectly: undo/redo
pushing and popping still happens at one end following LIFO order, while capped-history
eviction happens at the *other* end. Stack-like behavior at one end, queue-like
eviction at the other, in the same structure.

## Complexity

| Operation | Complexity | Why |
|---|---|---|
| `push_first()` / `push_last()` | O(1) | Array: write at computed index + move `front` or `rear`. Linked list: attach at head/tail with two pointer updates |
| `pop_first()` / `pop_last()` | O(1) | Array: move `front` or shrink `size` (no shift). Linked list: detach head/tail node, no traversal |
| `peek_first()` / `peek_last()` | O(1) | Direct read of the front/rear slot or node |

## Code
- [deque.cpp](../en/codes/cpp/chapter_stack_and_queue/deque.cpp) — `std::deque<int>`
  usage: push_back, push_front, front, back, pop_front, pop_back.
- [linkedlist_deque.cpp](../en/codes/cpp/chapter_stack_and_queue/linkedlist_deque.cpp) —
  hand-rolled `LinkedListDeque` class on a doubly linked list (`DoublyListNode` with
  `next` and `prev`), generalized `push(num, isFront)` / `pop(isFront)`.
- [array_deque.cpp](../en/codes/cpp/chapter_stack_and_queue/array_deque.cpp) — hand-rolled
  `ArrayDeque` class on a circular array, with an `index()` helper that handles
  negative-index wraparound via `(i + capacity()) % capacity()`.

Diagrams (from `en/docs/chapter_stack_and_queue/deque.assets/`):
- `deque_operations.png` — push/pop at both ends traced conceptually on `[3,2,5,4]`
- `linkedlist_deque_step1.png` — array-view vs. doubly-linked-list-view of the same
  deque, head = front, tail = rear
- `linkedlist_deque_step2_push_last.png` — push_last(4) becomes the new tail node
- `linkedlist_deque_step3_push_first.png` — push_first(1) becomes the new head node
- `linkedlist_deque_step4_pop_last.png` — pop_last() removes the tail node
- `linkedlist_deque_step5_pop_first.png` — pop_first() removes the head node
- `array_deque_step1.png` — the `rear = front + size` relationship
- `array_deque_step2_push_last.png` — the 3-step push_last recipe (update rear, write, size++)
- `array_deque_step3_push_first.png` — the 3-step push_first recipe (front--, write, size++)
- `array_deque_step4_pop_last.png` — the 1-step pop_last recipe (size--, no pointer move)
- `array_deque_step5_pop_first.png` — the 2-step pop_first recipe (front++, size--)

## Related
- [[Array]]
- [[Linked List]]
- [[List]]
- [[Stack]]
- [[Queue]]

## Self-check questions
1. Why does a deque's linked-list implementation require `prev` pointers, when
   [[Queue]]'s linked-list implementation only ever needed `next`?
2. Trace `push_first(1)` onto the doubly linked list `3 ⇄ 2 ⇄ 5 ⇄ 4` (front=3, rear=4)
   — what pointer assignments happen, and in what order, and why does the order matter?
3. With `capacity = 10` and `front = 0`, what physical array index does `push_first()`
   write to, and why does the `index()` helper add `capacity()` before applying `%`?
4. Why does `popLast()` in the array implementation not need to move any pointer at
   all, unlike `popFirst()`?
5. Give the concrete example from the source doc of why a plain [[Stack]] is
   insufficient for "undo" functionality with a capped history, and explain how a
   deque solves it.
