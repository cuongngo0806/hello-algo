---
tags:
  - chapter/stack-and-queue
status: done
date: 2026-08-10
---

# Queue

## Core idea
A **queue** is a linear data structure that follows the **first-in, first-out
(FIFO)** rule — the opposite of the [[Stack]]'s LIFO rule. It models people lining
up: newcomers join the **rear**, and the person at the **front** leaves first.
Whoever got in line first, leaves first.

The operation that adds an element at the rear is called **push** (enqueue); the
operation that removes the element at the front is called **pop** (dequeue); reading
the front element without removing it is **peek**.

Diagram: **`queue_operations.png`** — starts with queue `[1, 3, 2]` (front to rear —
`1` is at front, `2` is at rear). `push(5)` adds 5 at the rear, `push(4)` adds 4 at
the rear (front-to-rear now reads `1, 3, 2, 5, 4`). Then two `pop()` calls remove
`1`, then `3` — always from the **front**, in the order they were originally added.

Compare against the [[Stack]] diagram: same push sequence of numbers, but the pop
order is opposite. Stack popped `4` then `5` (LIFO — last added, first out). Queue
pops `1` then `3` (FIFO — first added, first out). That contrast is the entire
distinction between the two structures.

## Worked trace of the built-in queue

From [queue.cpp](../codes/cpp/chapter_stack_and_queue/queue.cpp), using C++'s
built-in `std::queue`:

```cpp
queue<int> queue;
queue.push(1);   // queue (front→rear): [1]
queue.push(3);   // [1, 3]
queue.push(2);   // [1, 3, 2]
queue.push(5);   // [1, 3, 2, 5]
queue.push(4);   // [1, 3, 2, 5, 4]

int front = queue.front();   // front = 1 (reads only, does not remove)

queue.pop();      // removes 1 (the FRONT, not the rear). queue is now [3, 2, 5, 4].

int size = queue.size();     // size = 4
bool empty = queue.empty();  // empty = false
```

Same asymmetry as `std::stack`: `queue.pop()` in C++ has **no return value**. To get
the dequeued value, read it with `front()` first, then call `pop()` to remove it —
mirrors the `stack.top()` then `stack.pop()` pattern from [[Stack]].

| Method | Description | Time complexity |
|---|---|---|
| `push()` | enqueue — add element at rear | O(1) |
| `pop()` | dequeue — remove front element | O(1) |
| `front()` / `peek()` | read front element without removing | O(1) |

## Building a queue from scratch

Same restriction idea as the stack: implement a queue by limiting an array or a
linked list to only add at one end and remove at the other — except unlike the
stack, insertion and removal happen at **opposite** ends, not the same end.

### Implementation 1 — backed by a linked list

Idea: use the **head node as the front**, and the **tail node as the rear**.
Enqueue inserts a node at the tail; dequeue removes the head node.

Diagram: **`linkedlist_queue_step2_push.png`** — shows `push(4)`. On the right, the
linked-list structure before the push: head is `1`, chain `1 → 3 → 2 → 5`, tail is
`5`. After `push(4)`, a new node `4` is appended after the old tail `5`
(`5->next = 4`), and `4` becomes the new tail. This differs from the stack's
linked-list push, which inserts at the **head** — the queue must insert at the
**tail**, since enqueue adds to the rear, not the front.

Code, [linkedlist_queue.cpp](../codes/cpp/chapter_stack_and_queue/linkedlist_queue.cpp):

```cpp
class LinkedListQueue {
  private:
    ListNode *front, *rear;   // head node = front, tail node = rear
    int queSize;

  public:
    /* enqueue */
    void push(int num) {
        ListNode *node = new ListNode(num);
        if (front == nullptr) {          // empty queue: both point to the new node
            front = node;
            rear = node;
        } else {                          // non-empty: append after the current tail
            rear->next = node;
            rear = node;
        }
        queSize++;
    }

    /* dequeue */
    int pop() {
        int num = peek();
        ListNode *tmp = front;
        front = front->next;   // front moves to the second node
        delete tmp;
        queSize--;
        return num;
    }

    /* peek */
    int peek() {
        if (size() == 0) throw out_of_range("queue is empty");
        return front->val;
    }
};
```

Worked trace with `1, 3, 2, 5, 4`:

1. `push(1)`: queue empty, so `front = rear = node(1)`. Chain: `1`.
2. `push(3)`: not empty, `rear->next = node(3)`, `rear = node(3)`. Chain: `1 → 3`.
3. `push(2)`: chain becomes `1 → 3 → 2`.
4. `push(5)`: chain becomes `1 → 3 → 2 → 5`.
5. `push(4)`: chain becomes `1 → 3 → 2 → 5 → 4`.
6. `pop()`: reads `peek() = front->val = 1`, sets `front = front->next = node(3)`,
   deletes the old node holding `1`. Chain is now `3 → 2 → 5 → 4`. Returns `1`.

Both enqueue (touches only the tail) and dequeue (touches only the head) are O(1) —
no traversal either way, because the class tracks *both* a `front` and a `rear`
pointer. This is the key structural difference from the stack's linked-list
implementation, which only needed one tracked end (head doubled as both insertion
and removal point). A queue needs two tracked ends since insertion and removal
happen at opposite sides.

### Implementation 2 — backed by a circular array

More subtle than the stack's array implementation.

**Why the naive approach fails.** A plain array with `push_back` for enqueue would
need dequeue to remove the *first* element — but removing index 0 requires shifting
every remaining element left by one slot, O(n). That defeats O(1) operations.

**The clever fix.** Track two variables instead of physically shifting elements:
- `front` — index of the current front element
- `size` — element count (so `rear = front + size` is the index just past the last
  element)

The elements currently in the queue occupy index range `[front, front + size - 1]`.
Enqueue writes the new element at index `front + size` and increments `size`.
Dequeue doesn't move any data — it just increments `front` and decrements `size`,
leaving the old front slot untouched (it's simply no longer considered part of the
queue).

**The problem this creates.** As enqueue/dequeue keep happening, `front` keeps
creeping rightward and eventually runs off the end of the fixed-size array — even
though there may be unused space at the *beginning* of the array (from earlier
dequeues). Diagram: **`array_queue_step2_push.png`** — `push(4)`: `front` sits at
index 1 (value `1`), `rear` computed as `front + size = 1 + 5 = 6`, so `4` is written
at index 6, `size` becomes 6. Diagram: **`array_queue_step3_pop.png`** — the
following `pop()`: `front` increments from 1 to 2, `size` decrements from 5 to 4 —
value `1` is still physically in the array at index 1, but the queue no longer
considers it part of the valid range.

**The circular fix.** Treat the array as **circular** — conceptually joining head
and tail into a loop — so when an index would run past the last slot, it wraps back
to index 0. Implemented with the modulo operator (`%`).

Code, [array_queue.cpp](../codes/cpp/chapter_stack_and_queue/array_queue.cpp):

```cpp
class ArrayQueue {
  private:
    int *nums;        // fixed-size backing array
    int front;          // index of the front element
    int queSize;        // current number of elements
    int queCapacity;    // total array capacity

  public:
    /* enqueue */
    void push(int num) {
        if (queSize == queCapacity) {
            cout << "Queue is full" << endl;
            return;
        }
        int rear = (front + queSize) % queCapacity;   // wrap around via modulo
        nums[rear] = num;
        queSize++;
    }

    /* dequeue */
    int pop() {
        int num = peek();
        front = (front + 1) % queCapacity;   // front wraps around too
        queSize--;
        return num;
    }

    /* peek */
    int peek() {
        if (isEmpty()) throw out_of_range("queue is empty");
        return nums[front];
    }
};
```

Worked trace of the wraparound — matches the driver code's "test circular array"
loop in [array_queue.cpp](../codes/cpp/chapter_stack_and_queue/array_queue.cpp)
(lines 118–123). Say `queCapacity = 10` and after the initial pushes/pop above,
`front = 1`, `queSize = 4` (contents `3, 2, 5, 4` at indices 1–4). Now
`push(i); pop();` repeatedly for `i = 0..9`:

- First round (`i=0`): `rear = (1 + 4) % 10 = 5`, so `0` is written at index 5,
  `size` becomes 5. Then `pop()`: `front = (1+1) % 10 = 2`, `size` becomes 4.
- Repeat: each round, `front` and `rear` creep further right through indices 5, 6,
  7, 8, 9...
- Eventually `rear` computes to something like `(8 + 4) % 10 = 2` — **wrapped past
  index 9 straight back to index 2**. Without the modulo, `rear` would compute to
  `12`, out of bounds for a 10-slot array. The modulo is exactly what turns the flat
  array into a logically circular one, letting `front` and `rear` cycle through the
  same fixed block of memory indefinitely instead of running off the end.

This circular-array queue still has one limitation vs. the array-backed stack: its
capacity is fixed at construction time (`ArrayQueue(int capacity)` in the
constructor) — it doesn't automatically grow like `std::vector`. The fix would be to
replace the fixed array with a dynamic array plus a resize/re-index step when full;
the example code leaves that as an exercise since the mechanics mirror the stack's
dynamic-array approach.

## Comparison of the two implementations

The same conclusions from comparing the stack's two implementations apply here:
array-backed tends to be faster on average (cache locality) but the circular-array
queue additionally has a fixed capacity unless resizing is added; linked-list-backed
has more consistent O(1) performance per-operation but higher per-node overhead. The
trade-off logic is identical to [[Stack]], just applied to a structure where
insertion happens at one end and removal at the other, instead of both at the same
end.

## Complexity

| Operation | Array-backed (circular) | Linked-list-backed |
|---|---|---|
| push (enqueue) | O(1), capacity-limited unless resized | O(1) |
| pop (dequeue) | O(1) | O(1) |
| peek | O(1) | O(1) |

## Code
- [queue.cpp](../codes/cpp/chapter_stack_and_queue/queue.cpp) — `std::queue` usage,
  push/front/pop/size/empty, all exercised in `main()`.
- [linkedlist_queue.cpp](../codes/cpp/chapter_stack_and_queue/linkedlist_queue.cpp) —
  `LinkedListQueue` class, tail-insertion enqueue / head-removal dequeue.
- [array_queue.cpp](../codes/cpp/chapter_stack_and_queue/array_queue.cpp) —
  `ArrayQueue` class, circular-array enqueue/dequeue via modulo, including a
  wraparound demonstration loop in `main()`.

Diagrams (from `en/docs/chapter_stack_and_queue/queue.assets/`):
- `queue_operations.png` — push/pop sequence on a queue, FIFO visualized
- `linkedlist_queue_step1.png`, `linkedlist_queue_step2_push.png`,
  `linkedlist_queue_step3_pop.png` — linked-list-backed queue, push and pop traced
  against the actual pointer structure (front = head, rear = tail)
- `array_queue_step1.png`, `array_queue_step2_push.png`,
  `array_queue_step3_pop.png` — array-backed queue, push and pop traced against
  `front`/`rear`/`size` index bookkeeping

## Related
- [[Stack]] — same restricted-array/restricted-linked-list construction idea,
  opposite LIFO vs. FIFO ordering, same amortized-O(1)-vs-consistent-O(1) trade-off
  discussion
- [[Linked List]] — tail-insertion technique used by the linked-list-backed queue

### Typical applications
- **Order processing systems** (e.g. e-commerce checkouts). Orders are added to a
  queue as customers place them; the system processes them in arrival order — first
  come, first served. A burst of orders in a short window (e.g. a big sales event)
  is exactly the high-concurrency scenario that makes queue-based processing
  performance a real engineering concern.
- **Task queues / to-do processing** — any "first come, first served" scenario, such
  as a printer's print queue or a restaurant's order queue, is naturally modeled by
  a queue.

## Self-check questions
1. Given the push sequence `push(1), push(3), push(2)`, what does a stack's `pop()`
   return first, and what does a queue's `pop()` return first? Why do they differ?
2. Why does the linked-list-backed queue need to track **two** pointers (`front` and
   `rear`) while the linked-list-backed stack only needs one?
3. In the circular-array queue, why is `rear` computed as `(front + size) % capacity`
   instead of just tracking a separate `rear` variable directly?
4. Trace: `queCapacity = 5`, `front = 3`, `queSize = 2` (so occupied indices are 3
   and 4). What index does the next `push()` write to? What happens to `front` after
   the following `pop()`?
5. Why would a plain (non-circular) array-backed queue eventually stop working even
   if the total number of elements never exceeds its capacity?

## Chapter 5 Quiz Results

**Date**: 2026-08-10 | **Score**: 4/7 | **Status**: passed

**Questions answered correctly (queue-related):**
- Q2 ✅ Circular-array queue with capacity=6, front=4, queSize=3 — occupied indices are 4, 5, 0 (wraps around via modulo)

**Questions missed (queue-related):**
- None specific to queue — all queue questions answered correctly

See [[Stack]] for full quiz breakdown and [[Deque]] for missed-question details.
