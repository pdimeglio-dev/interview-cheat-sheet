---
title: Linked Lists
description: Core mental model for linked lists, the pointer-rewiring pattern, and walkthroughs of Reverse Linked List and Merge Two Sorted Lists in JavaScript.
---

# Linked Lists

A **Linked List** is a linear data structure where each element (node) holds a value and a pointer to the next node. Unlike arrays, elements are not stored contiguously in memory — traversal is only possible by following pointers from one node to the next.

```
 head
  ↓
[ 1 | •→]  →  [ 2 | •→]  →  [ 3 | •→]  →  null
```

### When You'll See Linked Lists in Interviews

- Reversing sequences in-place
- Detecting cycles (Floyd's tortoise & hare)
- Merging sorted lists
- Finding the middle or k-th from end (fast/slow pointers)
- Re-ordering or partitioning nodes

### The Core Mental Model: _"Pointer Rewiring"_

Almost every linked list problem boils down to **changing where `.next` points**. You never move data between nodes — you just redirect arrows. The tricky part is that once you overwrite a `.next` pointer, **you lose access to whatever it used to point at**, so you always need a temporary variable to save the reference before rewiring.

> 💡 **Golden rule:** Before you overwrite any `.next`, stash the old value in a temp variable. If you lose the reference, you lose the rest of the list.

### The Standard ListNode Definition

```javascript
class ListNode {
    constructor(val = 0, next = null) {
        this.val = val;
        this.next = next;
    }
}
```

---

# Reverse Linked List (LeetCode 206)

:::note[Problem]
Given the `head` of a singly linked list, reverse the list and return the new head.

- **Input:** `head = [1, 2, 3, 4, 5]`
- **Output:** `[5, 4, 3, 2, 1]`
:::

**Related Links:**
- [LeetCode 206: Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/)
- [NeetCode: Reverse Linked List](https://neetcode.io/problems/reverse-a-linked-list)

---

## How to Think About It

The key insight is: **you're walking through the list one node at a time, and at each step you flip the arrow to point backwards instead of forwards.**

You need exactly three variables to pull this off:

| Variable | Role |
|----------|------|
| `prev` | The node behind you (starts as `null` because there's nothing before the head) |
| `aux` (current) | The node you're currently visiting |
| `next` | A temporary save of `aux.next` **before** you overwrite it |

### Step-by-Step Walkthrough

Starting state: `1 → 2 → 3 → null`

```
Step 0:   prev=null   aux=1   
          Save next=2, then flip: 1.next = null
          Move: prev=1, aux=2

          null ← 1    2 → 3 → null
                 ↑     ↑
               prev   aux

Step 1:   prev=1   aux=2
          Save next=3, then flip: 2.next = 1
          Move: prev=2, aux=3

          null ← 1 ← 2    3 → null
                      ↑    ↑
                    prev   aux

Step 2:   prev=2   aux=3
          Save next=null, then flip: 3.next = 2
          Move: prev=3, aux=null

          null ← 1 ← 2 ← 3
                           ↑
                         prev   aux=null → STOP
```

When `aux` becomes `null`, we've walked past the end. `prev` is now pointing at what used to be the last node — that's our new head.

---

## Implementation

```javascript
class Solution {
    /**
     * @param {ListNode} head
     * @return {ListNode}
     */
    reverseList(head) {
        let aux = head;
        let prev = null;

        while (aux != null) {
            let next = aux.next;   // 1. Save the next node
            aux.next = prev;       // 2. Flip the arrow
            prev = aux;            // 3. Advance prev

            aux = next;            // 4. Advance current
        }

        return prev; // prev is the new head
    }
}
```

---

:::tip Complexity
- **Time Complexity:** $O(n)$ — We visit each node exactly once.
- **Space Complexity:** $O(1)$ — Only three pointer variables, no extra data structures.
:::

---

## The Pattern to Memorize

Every iterative linked list reversal follows this exact 4-line loop body:

```javascript
let next = current.next;  // save
current.next = prev;      // flip
prev = current;           // advance prev
current = next;           // advance current
```

> 🧠 **Mnemonic:** **S**ave → **F**lip → **A**dvance prev → **A**dvance current. Think "**SFAA**" — or just "save it before you break it, then move on."

---

:::info Key Takeaway
Linked list problems feel tricky because you can't "see" the whole structure — you only have the node in front of you. The antidote is to always think: *"What do I need to save before I overwrite anything?"* If you answer that question at each step, the rest is mechanical.
:::

---

# Merge Two Sorted Lists (LeetCode 21)

:::note[Problem]
You are given the heads of two sorted linked lists `list1` and `list2`. Merge them into one sorted list by splicing together the nodes from both lists. Return the head of the merged linked list.

- **Input:** `list1 = [1, 2, 4]`, `list2 = [1, 3, 4]`
- **Output:** `[1, 1, 2, 3, 4, 4]`
:::

**Related Links:**
- [LeetCode 21: Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/)
- [NeetCode: Merge Two Sorted Lists](https://neetcode.io/problems/merge-two-sorted-lists)

---

## Solution 1 — Iterative (Dummy Node)

### How to Think About It

You have two sorted chains and a single "output" chain you're building. Walk both lists with pointers `p1` and `p2`, always appending the smaller node to the output.

The key insight is the **Dummy Node** pattern: create a throwaway node `new ListNode(0)` as a permanent anchor for the output list. A `current` pointer starts there and advances as you append nodes. When you're done, the real merged list starts at `dummy.next`.

### Why the Dummy Node?

Without it you'd need a separate `if / else` block just to decide the very first node of the merged list — and then **duplicate** the same comparison logic inside the loop. The dummy node makes the loop 100% consistent from the first node to the last.

:::danger[Gotcha — Redundant Head Initialization]
A first attempt often checks `p1.val < p2.val` once to set `newHead`, then checks again inside the loop. This is the same logic written twice — error-prone and unnecessary. The Dummy Node eliminates this entirely.
:::

### The "Tail" Cleanup

When one list runs out (`p1` or `p2` becomes `null`), there's no need to keep looping through the other. Because the remaining nodes are **already sorted and already linked**, you just point `current.next` at the surviving list — one $O(1)$ pointer assignment instead of an $O(n)$ traversal.

:::danger[Gotcha — Over-Looping]
Don't loop `while (p1 !== null || p2 !== null)` with null-checks inside the body. As soon as one side is exhausted, just attach the other side's remainder and you're done.
:::

### Step-by-Step Walkthrough

```
list1: 1 → 3 → 5 → null
list2: 2 → 4 → 6 → null

dummy → [0]      (anchor, will be discarded)
         ↑
       current

── Iteration 1: p1.val(1) < p2.val(2) → append p1(1), advance p1 ──

dummy → [0] → [1]
                ↑
              current      p1=3, p2=2

── Iteration 2: p2.val(2) < p1.val(3) → append p2(2), advance p2 ──

dummy → [0] → [1] → [2]
                      ↑
                    current      p1=3, p2=4

── Iteration 3: p1.val(3) < p2.val(4) → append p1(3), advance p1 ──

dummy → [0] → [1] → [2] → [3]
                             ↑
                           current      p1=5, p2=4

── Iteration 4: p2.val(4) < p1.val(5) → append p2(4), advance p2 ──

dummy → [0] → [1] → [2] → [3] → [4]
                                    ↑
                                  current      p1=5, p2=6

── Iteration 5: p1.val(5) < p2.val(6) → append p1(5), advance p1 ──

dummy → [0] → [1] → [2] → [3] → [4] → [5]
                                          ↑
                                        current      p1=null, p2=6

── p1 is null → attach remainder of p2 ──

dummy → [0] → [1] → [2] → [3] → [4] → [5] → [6] → null

Return dummy.next → [1] → [2] → [3] → [4] → [5] → [6] → null  ✅
```

### Implementation

```javascript
class Solution {
    /**
     * @param {ListNode} list1
     * @param {ListNode} list2
     * @return {ListNode}
     */
    mergeTwoLists(list1, list2) {
        const dummy = new ListNode(0);
        let current = dummy;
        let p1 = list1;
        let p2 = list2;

        while (p1 !== null && p2 !== null) {
            if (p1.val < p2.val) {
                current.next = p1;
                p1 = p1.next;
            } else {
                current.next = p2;
                p2 = p2.next;
            }
            current = current.next;
        }

        // Tail cleanup — attach whichever list still has nodes
        current.next = p1 !== null ? p1 : p2;

        return dummy.next;
    }
}
```

:::tip Complexity
- **Time Complexity:** $O(n + m)$ — Each node from both lists is visited at most once.
- **Space Complexity:** $O(1)$ — Only pointer variables; we re-use the existing nodes.
:::

---

## Solution 2 — Recursive

### How to Think About It

Instead of manually building the output chain, let the **call stack** do the work. At each level of recursion, compare the two heads, pick the smaller one, and recurse on that node's `.next` paired with the other list. The base case is when either list is `null` — just return the other.

:::danger[Gotcha — Scope and `this` in JavaScript Classes]
When calling the method recursively inside a class, you **must** use `this.mergeTwoLists(...)`. Without `this`, JavaScript looks for a global function named `mergeTwoLists` and throws a `ReferenceError`.
:::

:::danger[Gotcha — Losing the Linkage]
A common first attempt picks the winner node and recurses, but forgets to **assign** the result back. Without `p1.next = this.mergeTwoLists(...)`, the winner's `.next` still points at its original successor and the merged tail is lost.
:::

:::danger[Gotcha — The "Invisible" Head]
It feels like you're losing the start of the merged list because there's no explicit `newHead` variable. But the very first call sits at the **bottom** of the call stack and "waits" — it returns the overall winner only after every recursive call above it has resolved. The head isn't lost; it's the last thing to be returned.
:::

### Implementation

```javascript
class Solution {
    /**
     * @param {ListNode} list1
     * @param {ListNode} list2
     * @return {ListNode}
     */
    mergeTwoLists(list1, list2) {
        // Base cases
        if (list1 === null) return list2;
        if (list2 === null) return list1;

        if (list1.val < list2.val) {
            list1.next = this.mergeTwoLists(list1.next, list2);
            return list1;
        } else {
            list2.next = this.mergeTwoLists(list1, list2.next);
            return list2;
        }
    }
}
```

:::tip Complexity
- **Time Complexity:** $O(n + m)$ — One recursive call per node.
- **Space Complexity:** $O(n + m)$ — Each call adds a frame to the stack; in the worst case the stack is as deep as the total number of nodes.
:::

---

## Iterative vs. Recursive — Which to Use?

| | Iterative (Dummy Node) | Recursive |
|---|---|---|
| **Lines of code** | More | Fewer, more elegant |
| **Space** | $O(1)$ | $O(n + m)$ (call stack) |
| **Stack overflow risk** | None | Yes, on very long lists |
| **Readability** | Explicit control flow | Implicit via recursion |

> 🎤 **Interview answer:** *"I'd go with the iterative version. While recursion is more elegant and fewer lines of code, the iterative approach with a Dummy Node is safer for production because it uses $O(1)$ space. It won't cause a Stack Overflow if we're merging massive lists (like a high-frequency event stream), whereas the recursive version's memory usage grows linearly with the list length."*

---

## Patterns to Memorize

### The Dummy Node Pattern

Whenever you're **building a new list** node-by-node, start with a dummy:

```javascript
const dummy = new ListNode(0);
let current = dummy;

// ... loop that does current.next = <something>; current = current.next; ...

return dummy.next; // skip the dummy
```

> 🧠 **When to reach for it:** Any problem where you're constructing an output list and the first node isn't known upfront (merge, partition, add-two-numbers, etc.).

### The Tail-Attach Shortcut

When merging or zipping two lists, as soon as one is exhausted, attach the other:

```javascript
current.next = p1 !== null ? p1 : p2;
```

> 🧠 **Why it works:** The remaining nodes are already sorted and already linked — no need to iterate through them.
