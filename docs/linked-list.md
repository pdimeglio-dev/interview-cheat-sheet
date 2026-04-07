---
title: Linked Lists
description: Core mental model for linked lists, the pointer-rewiring pattern, and a walkthrough of the classic Reverse Linked List problem in JavaScript.
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
