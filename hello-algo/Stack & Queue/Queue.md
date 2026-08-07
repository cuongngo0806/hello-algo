---
tags:
  - chapter/stack-queue
status: taught
date: 2026-08-07
---

# Queue

## Core idea
A **queue** is a linear data structure that follows the **First In, First Out
(FIFO)** rule — the mirror image of [[Stack]]'s LIFO. The book's own analogy is a
real-world line of people waiting: newcomers keep joining at the back, while the
person who's been waiting longest (at the front) leaves first. Whatever went in
*first* comes out *first* — unlike a stack, there's no reversal here; the original
order is preserved all the way through.

The end where elements leave is the **front**; the end where new elements join is the
**rear**. Adding an element at the rear is **push** (also called enqueue); removing
the front element is **pop** (dequeue).

### Worked trace of push/pop

Diagram: **`queue_operations.png`** traces a queue starting as `[1, 3, 2]`
(front to rear, front listed first) through four operations:

1. `push(5)` → `[1, 3, 2, 5]` — `5` joins at the **rear**.
2. `push(4)` → `[1, 3, 2, 5, 4]` — `4` joins at the rear, behind `5`.
3. `pop()` → returns `1`, queue becomes `[3, 2, 5, 4]` — the element that has been
   waiting *longest* (`1`, sitting at the front) is the one that leaves.
4. `pop()` → returns `3`, queue becomes `[2, 5, 4]`.

It's worth directly contrasting this with [[Stack]]'s equivalent trace: there,
`pop()` returned `4` then `5` — most-recently-pushed first, a *reversed* order.
Here, `pop()` returns `1` then `3` — the *same* order they were originally pushed
in, fully preserved. Same push sequence in both cases; opposite retrieval order.
That contrast is the entire practical difference between a stack and a queue.

### Common operations — all O(1)

| Method | Description | Time Complexity |
|---|---|---|
| `push()` | Enqueue — add an element at the rear | O(1) |
| `pop()` | Dequeue — remove the front element | O(1) |
| `peek()` / `front()` | Read the front element without removing it | O(1) |

C++ usage, [queue.cpp:9-38](../en/codes/cpp/chapter_stack_and_queue/queue.cpp):
```cpp
queue<int> queue;

queue.push(1);   // Enqueue elements
queue.push(3);
queue.push(2);
queue.push(5);
queue.push(4);
// queue is now (front to rear): [1, 3, 2, 5, 4]

int front = queue.front();   // Access front element -> 1 (does NOT remove it)

queue.pop();                  // Dequeue front element -> queue becomes [3, 2, 5, 4]

int size = queue.size();      // Get queue length -> 4
bool empty = queue.empty();   // Check if empty -> false
```
Notice the insertion method is named `push()` on **both** `std::stack` and
`std::queue` — same method name, different meaning underneath (stack: add at top;
queue: add at rear). That's fine because each structure only ever has exactly one
insertion point, so the name doesn't need to specify *where*.

## Linked list implementation — worked trace

Just like [[Stack]], a queue can be built on top of a linked list — but this time
**two** pointers are needed instead of one, because push and pop happen at
*opposite* ends of the list: the **head node is the front**, the **tail node is the
rear**.

Diagram: **`linkedlist_queue_step1.png`** — starting state: a queue drawn as a
vertical box `[1, 3, 2, 5]` (front to rear) next to its actual linked-list form:
`Head node 1 → 3 → 2 → 5 Tail node`. Head = front, tail = rear.

Diagram: **`linkedlist_queue_step2_push.png`** — pushing `4`. Caption: "Push node
`4` to the tail of the linked list." After the push: `Head node 1 → 3 → 2 → 5 → 4
Tail node` — `4` becomes the new tail.

Diagram: **`linkedlist_queue_step3_pop.png`** — popping. Caption: "Remove the head
node." Starting from `1 → 3 → 2 → 5 → 4`, popping removes node `1`, leaving
`3 → 2 → 5 → 4` as the remaining chain — `3` is now the front.

This is a genuine asymmetry worth calling out compared to [[Stack]]'s linked-list
implementation: a stack does **head**-insertion *and* **head**-removal — both
operations touch the same end. A queue instead does **tail**-insertion,
**head**-removal — it needs to reach *both* ends of the list.

C++ code, [linkedlist_queue.cpp:10-71](../en/codes/cpp/chapter_stack_and_queue/linkedlist_queue.cpp):
```cpp
class LinkedListQueue {
  private:
    ListNode *front, *rear;  // head node = front, tail node = rear
    int queSize;

  public:
    void push(int num) {
        ListNode *node = new ListNode(num);
        if (front == nullptr) {   // empty queue: no existing rear to attach to
            front = node;          // the new node must become BOTH front and rear
            rear = node;
        } else {                   // non-empty: attach after the current rear
            rear->next = node;
            rear = node;
        }
        queSize++;
    }

    int pop() {
        int num = peek();
        ListNode *tmp = front;
        front = front->next;   // advance front pointer to the next node
        delete tmp;
        queSize--;
        return num;
    }

    int peek() {
        if (size() == 0) throw out_of_range("Queue is empty");
        return front->val;
    }
};
```
Trace why `push()` needs that empty-check branch, something [[Stack]]'s `push()`
never needed: when the queue is empty, `rear` is `nullptr` — there is no existing
node to set `rear->next` on. So the very first pushed node has to become **both**
`front` and `rear` at once. From the second push onward, the simpler branch takes
over: link the new node after the current `rear`, then move `rear` to point at it.
`pop()`, by contrast, only ever touches `front` — the same head-removal pattern
already seen in [[Stack]].

## Array implementation — the clever bit

A naive array implementation would dequeue by deleting index 0 — but per [[Array]],
removing index 0 requires shifting every remaining element one slot left, an O(n)
cost paid on **every single dequeue**. That would defeat the entire point of a queue
being cheap to operate on.

The fix used here: **don't shift anything at all — just move a pointer.** Keep a
variable `front` that tracks the *index* of the current front element (instead of
always assuming it's sitting at index 0), and a variable `size` tracking how many
elements are currently stored. From these two, `rear = front + size` gives the index
one slot past the last real element. The valid data always lives in the index range
`[front, rear - 1]`.

Diagram: **`array_queue_step1.png`** — setup: the array holds `[1, 3, 2, 5]` in
consecutive slots, `front` points at the index of value `1`, and the diagram states
the defining relationship directly: **`rear = front + size`**. With `size = 4` here,
`rear` points exactly one slot past `5`.

Diagram: **`array_queue_step2_push.png`** — `push(4)`. The caption spells out the
3-step recipe precisely:
1. Update the variable `rear`.
2. Write the new element at the `rear` index.
3. Increase `size` by 1.

After this: `4` gets written at the old `rear` slot, `size` becomes `5`, and `rear`
advances one further slot.

Diagram: **`array_queue_step3_pop.png`** — `pop()`. Caption: **1. Increase `front`
by 1. 2. Decrease `size` by 1.** That is the *entire* operation. The old front
element (value `1`) is not erased and nothing is shifted — it's simply left sitting
in its memory slot, now permanently outside the `[front, rear-1]` valid range, so
it's logically gone even though its bytes are untouched. Compare this with [[List]]'s
array deletion, which required an actual O(n) shift — this queue design sidesteps
that entirely by redefining what counts as "valid" instead of physically moving data.

### The wraparound problem — worked trace

As pushes and pops keep happening, both `front` and `rear` keep creeping rightward
through the array. Eventually `rear` would run off the *physical* end of the array —
even though there might be plenty of *logically* free space sitting at the
beginning, freed up by earlier pops. The fix: treat the array as **circular**. When
`front` or `rear` would step past the last valid index, wrap back around to index
`0` instead. This wraparound is implemented with the **modulo operator**.

C++ code, [array_queue.cpp:44-65](../en/codes/cpp/chapter_stack_and_queue/array_queue.cpp):
```cpp
void push(int num) {
    if (queSize == queCapacity) { /* full, refuse */ return; }
    // rear = (front + size) % capacity -- the modulo makes rear wrap around
    int rear = (front + queSize) % queCapacity;
    nums[rear] = num;
    queSize++;
}

int pop() {
    int num = peek();
    // front advances by 1, wrapping back to 0 if it runs past the last index
    front = (front + 1) % queCapacity;
    queSize--;
    return num;
}
```

**Worked trace of the wraparound itself**, using `queCapacity = 10`: suppose after a
series of prior pushes/pops the queue currently has `front = 8` and `queSize = 3`
(meaning it occupies logical slots `8, 9, 0` — it has already wrapped once before).
Now call `push(99)`:

```
rear = (front + queSize) % queCapacity
     = (8 + 3) % 10
     = 11 % 10
     = 1
```

The new element gets written at **physical index `1`** — not index `11`, which
doesn't even exist in a 10-slot array. Without the modulo operation, this would be
an out-of-bounds write past the end of the array. With it, the array behaves exactly
as if index `9` is immediately followed by index `0` again — the same wraparound
logic as a clock face going from `12` straight back to `1`. This exact scenario is
what the `array_queue.cpp` driver code's "circular array test" loop exercises: it
runs 10 rounds of push-then-pop specifically to force `front` and `rear` all the way
around the array at least once, confirming the wraparound logic holds.

**A remaining limitation, noted but not built in this repo**: this circular-array
queue still has a *fixed* total capacity — `push()` simply refuses (and prints an
error) once the array is full. The source doc points out this could be fixed the
same way [[List]] fixed a plain array's fixed-length problem: swap the fixed array
for a dynamic array with an expansion mechanism. That combination isn't actually
implemented anywhere in this repo's code, so there's no worked trace for it here —
just the explicit conceptual link back to what [[List]] already covered.

## Comparing the two implementations

The source doc states this comparison reaches **the same conclusion as [[Stack]]'s**
comparison, so it isn't repeated from scratch — but stated concretely for queues:
array-based queues get better cache locality (contiguous memory, same reasoning as
[[Array]]) at the cost of a capacity that's either fixed outright, or occasionally
pays an O(n) expansion cost if backed by a dynamic array (per [[List]]).
Linked-list-based queues have more *consistent* per-operation cost and grow with no
capacity ceiling at all, at the cost of per-node pointer memory overhead (per
[[Linked List]]) and an allocation on every single push.

## Typical applications

- **Order processing systems** — the source doc's own example is e-commerce orders:
  as shoppers place orders, each one is added to a queue, and the system processes
  orders strictly in that same arrival order. This is exactly "first come, first
  served," and is precisely why systems facing a burst of simultaneous orders (a big
  sale event, for instance) rely on a queue rather than a stack — processing orders
  out of arrival sequence would be unfair or simply wrong in that context.
- **To-do task queues** — a printer's print queue, or a restaurant's order queue: any
  situation that needs "first come, first served" behavior maps directly onto a
  queue's FIFO contract.

## Complexity

| Operation | Complexity | Why |
|---|---|---|
| `push()` | O(1) | Array: write at `rear` index + advance pointer. Linked list: attach after tail + move tail pointer |
| `pop()` | O(1) | Array: advance `front` index (no shift). Linked list: advance head pointer, free old head |
| `peek()` / `front()` | O(1) | Direct read of the front slot/node, no traversal |

## Code
- [queue.cpp](../en/codes/cpp/chapter_stack_and_queue/queue.cpp) — `std::queue<int>`
  usage: push, front, pop, size, empty.
- [linkedlist_queue.cpp](../en/codes/cpp/chapter_stack_and_queue/linkedlist_queue.cpp) —
  hand-rolled `LinkedListQueue` class, tail-insertion push / head-removal pop, using
  separate `front` and `rear` pointers.
- [array_queue.cpp](../en/codes/cpp/chapter_stack_and_queue/array_queue.cpp) —
  hand-rolled `ArrayQueue` class implementing a **circular array** via the modulo
  operator, including a driver-code test loop that exercises the wraparound.

Diagrams (from `en/docs/chapter_stack_and_queue/queue.assets/`):
- `queue_operations.png` — push(5), push(4), pop(), pop() traced on `[1,3,2]`,
  showing FIFO order preserved (contrast with [[Stack]]'s reversed order)
- `linkedlist_queue_step1.png` — array-view vs. linked-list-view of the same queue,
  head = front, tail = rear
- `linkedlist_queue_step2_push.png` — push(4) becomes the new tail node
- `linkedlist_queue_step3_pop.png` — pop() removes the head node
- `array_queue_step1.png` — the `rear = front + size` relationship
- `array_queue_step2_push.png` — the 3-step push recipe (update rear, write, size++)
- `array_queue_step3_pop.png` — the 2-step pop recipe (front++, size--)

## Related
- [[Array]]
- [[Linked List]]
- [[List]]
- [[Stack]]

## Self-check questions
1. Given the same push sequence `push(5)` then `push(4)` on `[1, 3, 2]`, why does
   `pop()` return `1` then `3` for a queue, but `4` then `5` for a stack?
2. Why does `LinkedListQueue::push()` need a special empty-queue branch that
   `LinkedListStack::push()` never needed?
3. Why is `rear` tracked as a separate variable (`front + size`) rather than the
   queue always assuming its front sits at index 0?
4. Trace the wraparound: with `queCapacity = 10`, `front = 8`, `queSize = 3`, what
   physical array index does the next `push()` write to, and why?
5. What real limitation does this circular-array `ArrayQueue` still have, and which
   earlier concept from [[List]] would fix it?
