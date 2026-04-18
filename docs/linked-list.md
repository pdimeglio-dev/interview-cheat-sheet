---
title: Linked Lists
description: Core mental model for linked lists, the pointer-rewiring pattern, and walkthroughs of Reverse Linked List, Merge Two Sorted Lists, Remove Linked List Elements, Remove Nth Node From End, and Middle of the Linked List in JavaScript.
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

---

# Remove Linked List Elements (LeetCode 203)

:::note[Problem]
Given the `head` of a linked list and an integer `val`, remove all nodes whose value equals `val` and return the new head.

- **Input:** `head = [1, 2, 6, 3, 4, 5, 6]`, `val = 6`
- **Output:** `[1, 2, 3, 4, 5]`
:::

**Related Links:**
- [LeetCode 203: Remove Linked List Elements](https://leetcode.com/problems/remove-linked-list-elements/)
- [NeetCode: Remove Linked List Elements](https://neetcode.io/problems/remove-linked-list-elements)

---

## How to Think About It

Two things make this problem tricky:

1. **The head itself might need to be removed** — and possibly several heads in a row (e.g. `[7, 7, 7, 1]`, `val = 7`).
2. **Consecutive matches** — after you splice out a node, the *new* `cur.next` might also match and needs to be re-inspected.

Both problems dissolve with the **Dummy Node** pattern: place a sentinel node before `head` so that removing the first real node works exactly like removing any other node.

### The Key Insight: Don't Advance After a Deletion

When you find a match, you rewire `cur.next = cur.next.next` to bypass it. **Do not move `cur` forward afterwards.** The new `cur.next` is a brand-new candidate that hasn't been checked yet — advancing would skip it.

This flips the loop shape compared to most traversals:

- Loop condition is `cur.next !== null` (we inspect the node *ahead* of us)
- On a match: rewire, **stay put**
- On a miss: advance `cur`

---

## Implementation

```javascript
class Solution {
    /**
     * @param {ListNode} head
     * @param {number} val
     * @return {ListNode}
     */
    removeElements(head, val) {
        const dummy = new ListNode(0);
        dummy.next = head;

        let cur = dummy;

        while (cur.next !== null) {
            if (cur.next.val === val) {
                // Bypass the match — don't advance, the new cur.next needs checking
                cur.next = cur.next.next;
            } else {
                cur = cur.next;
            }
        }

        return dummy.next;
    }
}
```

:::tip Complexity
- **Time Complexity:** $O(n)$ — Each node is inspected at most once.
- **Space Complexity:** $O(1)$ — Only the dummy and a single pointer.
:::

---

## Why the Edge Cases Just Work

| Case | Why the code handles it |
|------|-------------------------|
| **Consecutive matches** `[7, 7, 7]`, `val = 7` | `cur` stays at dummy; `dummy.next` is rewired three times, finally landing on `null`. |
| **Head must be removed** | `cur` starts *before* `head`, so removing the first real node is just another `cur.next = cur.next.next`. |
| **Empty list** (`head = null`) | `cur.next` is `null` immediately; the loop never runs and `dummy.next` (i.e. `null`) is returned. |

:::danger[Gotcha — Advancing Past a Fresh Candidate]
A common mistake is treating this like a normal traversal and always doing `cur = cur.next` at the bottom of the loop. That skips the node that just became `cur.next` after a deletion, and consecutive matches leak through.
:::

:::info Key Takeaway
**Look ahead, not at.** When you're filtering a list, stand on the node *before* the candidate. That way one pointer rewires (`cur.next = cur.next.next`) does the removal, and a dummy node makes the head just another regular position.
:::

---

# Remove Nth Node From End of List (LeetCode 19)

:::note[Problem]
Given the `head` of a linked list, remove the `n`-th node from the end of the list and return its head.

- **Input:** `head = [1, 2, 3, 4, 5]`, `n = 2`
- **Output:** `[1, 2, 3, 5]`
:::

**Related Links:**
- [LeetCode 19: Remove Nth Node From End of List](https://leetcode.com/problems/remove-nth-node-from-end-of-list/)
- [NeetCode: Remove Node From End of Linked List](https://neetcode.io/problems/remove-node-from-end-of-linked-list)

---

## How to Think About It

The naive approach needs two passes: count the nodes, then walk `length - n` steps to find the one to delete. You can do it in **one pass** with the **two-pointer gap** trick.

### The Two-Pointer Gap Trick

Keep two pointers exactly `n` nodes apart. When the front pointer walks off the end (hits `null`), the back pointer is sitting right before the node that's `n` from the end — which is exactly where you need to splice.

```
n = 2
list:        1 → 2 → 3 → 4 → 5 → null
dummy → [0] → 1 → 2 → 3 → 4 → 5 → null
         ↑
     preDelete, cur

── Step 1: advance cur by n (=2) ──

dummy → [0] → 1 → 2 → 3 → 4 → 5 → null
         ↑         ↑
     preDelete    cur         (gap of n nodes)

── Step 2: advance both until cur == null ──

dummy → [0] → 1 → 2 → 3 → 4 → 5 → null
                   ↑              ↑
               preDelete         cur=null

── Step 3: splice out preDelete.next ──

dummy → [0] → 1 → 2 → 3 → 5 → null
                   ↑
               preDelete

Return dummy.next → 1 → 2 → 3 → 5  ✅
```

### Why the Dummy Node Is Essential Here

If `n` equals the list length (i.e. remove the head), the front pointer becomes `null` *immediately* after the first phase. Without a dummy, the back pointer would be at `head` and there'd be no node "before" it to splice from. With a dummy, `preDelete` stays on the dummy, and `dummy.next = dummy.next.next` correctly removes the original head.

---

## Implementation

```javascript
class Solution {
    /**
     * @param {ListNode} head
     * @param {number} n
     * @return {ListNode}
     */
    removeNthFromEnd(head, n) {
        const dummy = new ListNode(0);
        dummy.next = head;

        let cur = head;
        let preDelete = dummy;

        // Phase 1: open a gap of n nodes between cur and preDelete
        let forward = 0;
        while (cur !== null && forward < n) {
            cur = cur.next;
            forward++;
        }

        // Phase 2: walk both until cur falls off the end
        while (cur !== null) {
            cur = cur.next;
            preDelete = preDelete.next;
        }

        // preDelete is now right before the node to remove
        preDelete.next = preDelete.next.next;

        return dummy.next;
    }
}
```

:::tip Complexity
- **Time Complexity:** $O(L)$ — One pass; each pointer traverses the list at most once.
- **Space Complexity:** $O(1)$ — Two pointers and a dummy.
:::

### Why `while` with a Counter Instead of a `for` Loop?

A bare `for (let i = 0; i < n; i++) cur = cur.next;` would work **given the LeetCode constraint** that `1 <= n <= length`. But it has no safety net: if `n > length`, you eventually do `null.next` and crash.

The `while (cur !== null && forward < n)` version is defensive — it stops the instant `cur` runs off the end, even if `n` is larger than the list. In an interview setting, this is a nice thing to point out: *"I'm guarding against `n > length` so the code doesn't null-deref on unexpected input."*

Both are $O(n)$; the only difference is robustness.

---

## Why the Edge Cases Just Work

| Case | Why the code handles it |
|------|-------------------------|
| **Remove the head** (`n == length`) | After phase 1, `cur` is `null`; phase 2 never runs; `preDelete` is still the dummy, so `dummy.next = dummy.next.next` removes the head. |
| **Single-node list** `[1]`, `n = 1` | Same as "remove head" — dummy stays in place, `dummy.next` becomes `null`. |
| **Remove the tail** | Phase 2 walks `preDelete` to the second-to-last node; `preDelete.next = null` drops the tail. |

:::danger[Gotcha — Forgetting the Dummy]
Without a dummy, `n == length` has no "node before the head" to splice from. You'd end up writing a special-case branch for "is the head the one we're removing?" — exactly the kind of duplication the dummy exists to eliminate.
:::

:::info Key Takeaway
You can't go backwards in a singly linked list, so "the node `n` from the end" feels like it needs two passes — one to measure the length, one to walk to it. The **fixed-gap two-pointer** trick gives you the answer in a single pass: hold two pointers exactly `n` apart, then move them together until the lead hits `null`. Because the gap never changes, when the lead runs out the trailing pointer has landed exactly `n` from the end.

Whenever a problem says *"the n-th from the end"* or *"the middle"* (a gap that's half the length), reach for this pattern.
:::

---

# Middle of the Linked List (LeetCode 876)

:::note[Problem]
Given the `head` of a singly linked list, return the middle node. If the list has an even number of nodes, return the **second** of the two middle nodes.

- **Input:** `head = [1, 2, 3, 4, 5]` → **Output:** `[3, 4, 5]` (middle node is `3`)
- **Input:** `head = [1, 2, 3, 4, 5, 6]` → **Output:** `[4, 5, 6]` (second middle is `4`)
:::

**Related Links:**
- [LeetCode 876: Middle of the Linked List](https://leetcode.com/problems/middle-of-the-linked-list/)
- [NeetCode: Find the Middle of a Linked List](https://neetcode.io/problems/find-the-middle-of-a-linked-list)

---

## How to Think About It

The previous problem used two pointers with a **fixed gap** (`n` apart, moving together). This one uses two pointers with a **growing gap** — they start at the same node but move at different speeds. That's the **fast/slow pointer** pattern, often called **tortoise and hare**.

### The Core Trick

- `slow` advances **one** node per step.
- `fast` advances **two** nodes per step.
- When `fast` reaches the end, `slow` has covered half the distance — it's sitting on the middle.

```
── Odd length: [1, 2, 3, 4, 5] ──

Step 0:  slow=1, fast=1
Step 1:  slow=2, fast=3
Step 2:  slow=3, fast=5      (fast.next === null → stop)
Return slow → [3, 4, 5]  ✅

── Even length: [1, 2, 3, 4, 5, 6] ──

Step 0:  slow=1, fast=1
Step 1:  slow=2, fast=3
Step 2:  slow=3, fast=5
Step 3:  slow=4, fast=null    (fast === null → stop)
Return slow → [4, 5, 6]  ✅  (second of the two middles, as required)
```

### Why the Loop Condition Has Two Checks

```javascript
while (fast !== null && fast.next !== null)
```

Both checks are needed because `fast` advances by **two** each iteration:

- `fast !== null` — so `fast.next` doesn't throw on an even-length list (where `fast` lands on `null`).
- `fast.next !== null` — so `fast.next.next` doesn't throw on an odd-length list (where `fast` lands on the last node).

Skipping either check causes a null-deref on the opposite parity of list length.

---

## Implementation

```javascript
class Solution {
    /**
     * @param {ListNode} head
     * @return {ListNode}
     */
    middleNode(head) {
        let slow = head;
        let fast = head;

        while (fast !== null && fast.next !== null) {
            fast = fast.next.next;
            slow = slow.next;
        }

        return slow;
    }
}
```

:::tip Complexity
- **Time Complexity:** $O(n)$ — `fast` traverses the list once (at double speed, so `n/2` iterations).
- **Space Complexity:** $O(1)$ — Two pointers.
:::

---

## The Early-Return Guard Isn't Needed

A first attempt often starts with:

```javascript
if (head.next === null) return head;   // single-node early return
```

This is **redundant**. For a single-node list, `head.next` is already `null`, so the loop condition `fast !== null && fast.next !== null` is false on the first check — the loop never runs and `slow` (which is `head`) is returned correctly.

> 🧠 **Pattern:** If you catch yourself writing a special case that the main loop would already handle correctly, delete it. Same reasoning as using a dummy node to avoid a special case for the head — the less branching, the fewer places for bugs to hide.

:::info Key Takeaway
**Fast/slow (tortoise and hare) is the `Math.floor(length / 2)` of linked lists.** When you can't index into the list, doubling one pointer's speed lets you reach "halfway" in a single pass. This is the same tooling used for **cycle detection** (Floyd's algorithm) — if `fast` ever catches `slow` again, there's a cycle; if `fast` reaches `null`, there isn't.
:::

### Compare: Fixed-Gap vs. Growing-Gap

| Pattern | Movement | Use when |
|---------|----------|----------|
| **Fixed gap** (LC 19) | Both pointers advance 1 step, but lead starts `n` ahead | You need a node at a specific offset from the end |
| **Growing gap** (LC 876, cycle detection) | `fast` goes 2, `slow` goes 1 | You need the midpoint, or to detect a cycle |
