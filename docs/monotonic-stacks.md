---
sidebar_position: 3
title: Monotonic Stack Pattern
description: Intuitive explanation and template for Next Greater/Smaller Element problems using the Monotonic Stack, with a LeetCode 496 (Next Greater Element I) walkthrough in JavaScript.
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Monotonic Stack Pattern

The **Monotonic Stack** is a powerful $O(n)$ technique for finding the *nearest* neighbor in an array that satisfies a comparison property (Greater/Smaller).

---

## 1. The Core Mental Model: _"The Waiting Room"_

Think of the stack as a _waiting room_ for elements that haven't yet found their "Next Greater" or "Next Smaller" match.

- Elements enter the waiting room (_stack_) because they're waiting for a match.
- The room is strictly ordered: each new arrival compares against the top of the stack (the "peek").
- If the new element _wins_ the comparison, the element at the top is popped and records the new arrival as its match.

---

## 2. Directions & Extremes (Bottom $\to$ Top)

The **direction** of the stack describes the sort order from _bottom_ to _top_.

| Stack Type  | Slope (Bottom → Top)    | Top (Peek) is... | Purpose                |
|-------------|------------------------|------------------|------------------------|
| Decreasing  | [100, 80, 50, 20]      | Minimum          | Find Next **Greater**  |
| Increasing  | [10, 20, 50, 80]       | Maximum          | Find Next **Smaller**  |

---

## 3. Implementation Template (JavaScript)

> **Tip:** Store the _index_ in the stack, not the value, so you can compute "how far away" the next match is.

```javascript
/**
 * Finds the Next Greater Element for every item in an array.
 * Time Complexity: O(N) - Each element is pushed/popped once.
 * Space Complexity: O(N) - To store the results/stack.
 */
function nextGreaterElement(arr) {
  const stack = []; // Monotonic Decreasing
  const results = new Array(arr.length).fill(-1);

  for (let i = 0; i < arr.length; i++) {
    // While current element is BIGGER than the MIN at the top...
    while (stack.length > 0 && arr[i] > arr[stack[stack.length - 1]]) {
      const topIndex = stack.pop();
      results[topIndex] = arr[i]; // Found the match!
    }
    stack.push(i); // Current enters the waiting room
  }
  return results;
}
```

---

## 4. The Cheat Sheet: All 4 Variations

| Goal           | Stack Direction | Iteration          | While Condition           |
|----------------|----------------|--------------------|---------------------------|
| Next Greater   | Decreasing     | Left $\to$ Right   | `current > stack.top()`   |
| Next Smaller   | Increasing     | Left $\to$ Right   | `current < stack.top()`   |
| Prev Greater   | Decreasing     | Right $\to$ Left   | `current > stack.top()`   |
| Prev Smaller   | Increasing     | Right $\to$ Left   | `current < stack.top()`   |

---

## 5. Why It Scales: $O(n)$ Time

Even with a while-loop inside a for-loop, the time complexity is linear:

- Every element is **pushed** onto the stack exactly once.
- Every element is **popped** at most once.
- Total operations = $2N$ → $O(N)$.

---

## Comparison: Stack vs. Heap

- **Monotonic Stack:** Best for finding the _nearest_ relative neighbor; $O(n)$.
- **Heap (Priority Queue):** Best for finding the _absolute_ global min/max in a dynamic set; $O(n \log n)$.

---

# Next Greater Element I (LeetCode 496)

:::note[Problem]

Given two integer arrays `nums1` and `nums2`, for each element in `nums1`, find the next greater element in `nums2` to its right. If there is none, return `-1` for that element.

:::

**Pattern:** Monotonic Stack (Decreasing)

**Related Links:**
- [LeetCode 496: Next Greater Element I](https://leetcode.com/problems/next-greater-element-i/)
- [NeetCode: Next Greater Element I](https://neetcode.io/problems/next-greater-element-i)

## Solution: Monotonic Decreasing Stack

This implementation uses a Monotonic Decreasing Stack to pre-process the "Next Greater" relationships in $O(n)$ time.

<Tabs>
<TabItem value="JavaScript" label="JavaScript">

```js
/**
 * @param {number[]} nums1
 * @param {number[]} nums2
 * @return {number[]}
 */
function nextGreaterElement(nums1, nums2) {
  const nextGreaters = new Map();
  const stack = [];

  // 1. Build the "Next Greater" Map using nums2
  for (let i = 0; i < nums2.length; i++) {
    // While current element is bigger than the one at the top of the stack...
    while (stack.length > 0 && nums2[i] > stack[stack.length - 1]) {
      const smallerElement = stack.pop();
      nextGreaters.set(smallerElement, nums2[i]); // Match found!
    }
    stack.push(nums2[i]);
  }

  // 2. Map the results back to nums1
  return nums1.map(num => nextGreaters.has(num) ? nextGreaters.get(num) : -1);
}
```

</TabItem>
</Tabs>

---

## Why this solution works

The Monotonic Stack is optimal for "nearest greater neighbor" problems due to its efficiency and locality of comparison. Here’s a breakdown of its advantages:

### 1. From Quadratic to Linear Time ($O(n^2) \rightarrow O(n)$)

- **Brute Force:** For every element in `nums1`, you find its position in `nums2` and scan to the right — potentially $O(n \times m)$.
- **Monotonic Stack:** Every element in `nums2` is added and removed from the stack at most once. Pre-processing is strictly $O(n)$.

### 2. Locality: Nearest Neighbor Guarantee

- Sorting or using a Heap won’t help — they lose element order. 
- The Monotonic Stack keeps elements in order and efficiently finds the *first* element breaking the current trend (e.g., the next greater value to the right).

### 3. Space-Time Tradeoff (Using a Map)

- We precompute a Map from elements to their next greater value in `nums2` ($O(n)$ space), then for each query in `nums1`, return the answer in $O(1)$ time.
- This "Pre-compute + Lookup" pattern is a hallmark of performance engineering.

### Tags (496)

- **Pattern:** Monotonic Stack (Decreasing)
- **Data structures:** Stack (Array), Hash Map
- **Complexity:** Time $O(n + m)$ · Space $O(n)$


---

# Daily Temperatures: Monotonic Stack "Distance" Trick

After mastering **Next Greater Element I**, the next step is tackling **Daily Temperatures**. This problem introduces an important optimization: *store indices, not values, in your monotonic stack*—unlocking the ability to answer "How many days until a warmer temperature?" in one pass.

---

## Problem Statement

Given a list of daily temperatures, return a list where, for each day in the input, you tell how many days you would have to wait until a warmer temperature. If there is no future day for which this is possible, put `0` instead.

---

## The Index vs. Value Strategy

While the logic remains a **monotonic decreasing stack**, the shift from values to indices in the stack is a game-changer:

- With indices, you don't just know whether a warmer temperature exists—you know *how far away* it is!
- For each temperature at index `i`, you can instantly compute the distance to the next warmer day:
  
  **Distance = i(current) − i(popped)**

---

## JavaScript Implementation

```javascript
/**
 * @param {number[]} temperatures
 * @return {number[]}
 * Pattern: Monotonic Decreasing Stack (Indices)
 */
function dailyTemperatures(temperatures) {
  // 1. Result array initialized with 0's (for "no warmer day" cases)
  const res = new Array(temperatures.length).fill(0);
  const stack = []; // Holds indices

  for (let i = 0; i < temperatures.length; i++) {
    // 2. Pop from stack while the current day is warmer than stack top
    while (stack.length > 0 && temperatures[i] > temperatures[stack[stack.length - 1]]) {
      const prevIndex = stack.pop();
      res[prevIndex] = i - prevIndex; // 3. Record distance (# of days)
    }
    stack.push(i); // Store current index for future comparison
  }

  return res;
}
```

---

## Why This "Levels Up" the Approach

- **Implicit Results:** No need for extra mappings. The result array is updated directly using indices.
- **Zero Initialization:** By pre-filling the array with zeros, you naturally handle days that never see a warmer temperature.
- **Generalizable:** This "store index in stack" trick works for nearly all monotonic stack problems asking for "distance" or "position"—not just "next greater value".

---

## Interview Tip

If an interviewer asks for a "next greater element" or "how far until X changes"—suggest using indices in your stack. This gives you both *the value and the positional relationship* in a single pass, and solves almost every variation efficiently.

---
 
Ready for an even greater challenge? This same stack pattern powers "Largest Rectangle in Histogram" and other hard problems!
