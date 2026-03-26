---
title: Sliding Window Pattern
description: Introduction to the sliding window technique, with an illustrative example (Best Time to Buy and Sell Stock).
---

# Sliding Window Pattern

The **sliding window pattern** is a powerful technique for solving problems that involve finding optimal subarrays or subranges within a larger array or string. Instead of using nested loops (which can be slow), a sliding window "slides" a subsection of the input over the data in a single pass, reducing both time and space complexity.

This approach works especially well for:
- Finding maximum/minimum values in a subarray of a certain size
- Calculating sums or averages over a fixed (or dynamic) window
- Identifying the longest/shortest substring or subarray that meets specific criteria

A sliding window is typically maintained with two pointers (left and right), which move across the array to maintain a current subarray or subsequence, adjusting as constraints change.

---

# Best Time to Buy and Sell Stock: Sliding Window Application

A classic sliding window problem is finding the **maximum difference between two elements in an array, where the smaller comes before the larger**. The "Buy and Sell Stock" problem is a prime example, and the *dynamic sliding window* finds the solution in $O(n)$ time.

## How It Works

Unlike patterns where the pointers start at opposite ends, a sliding window here begins with both pointers at the start, moving them right together:

- The **left pointer** (`buy`) marks the candidate to buy the stock (the lowest price found so far).
- The **right pointer** (`sell`) scans forward, looking for a selling opportunity.

**As we scan:**
1. **If it's profitable** (`prices[buy] < prices[sell]`): Compute the profit, update the maximum profit, and continue moving `sell` right to search for a better sell price.
2. **If we find a lower price** (`prices[buy] >= prices[sell]`): Move `buy` up to `sell`—start a new window with this new (potentially lower) buy price.

## JavaScript Implementation

```javascript
function maxProfit(prices) {
  if (prices.length < 2) return 0;

  let buy = 0;
  let sell = 1;
  let maxProfit = 0;

  while (sell < prices.length) {
    if (prices[buy] < prices[sell]) {
      maxProfit = Math.max(maxProfit, prices[sell] - prices[buy]);
    } else {
      buy = sell;
    }
    sell++;
  }

  return maxProfit;
}
```

## Complexity Analysis

- **Time Complexity:** $O(n)$ &mdash; The `sell` pointer traverses the array once.
- **Space Complexity:** $O(1)$ &mdash; Only a few variables are allocated (`buy`, `sell`, and `maxProfit`), so extra memory usage is constant.

---

# Longest Substring Without Repeating Characters (LeetCode 3): Sliding Window + Set

A foundational application of the **Sliding Window** pattern is finding the longest contiguous substring without any repeating characters. This problem demonstrates how to efficiently manage subarrays or substrings with variable constraints and achieve optimal time complexity by leveraging fast membership checks with a **Set**.

## Problem Statement

Given a string `s`, find the length of the longest substring without repeating characters.

## Sliding Window Approach

We use two pointers, commonly called `left` and `right`, to define the current window—`s[left...right]`. As we expand the window to the right, we keep track of which characters are present in the window using a **Set** for $O(1)$ lookup and deletion.

### Step-by-Step Logic

1. **Initialize**:
    - `left` and `right` both at the start of the string.
    - An empty set to store characters in the current window.
    - A variable to keep track of the maximum length found.

2. **Expand Right Pointer**:
    - Incrementally move `right` through the string.
    - If `s[right]` is not in the set, it's a unique character:
        - Add it to the set.
        - Update the maximum length.

3. **Handle Duplicates (Shrink Window from the Left)**:
    - If `s[right]` is already in the set, keep removing `s[left]` from the set and incrementing `left` until the duplicate character is removed from the window.
    - Then add `s[right]` and continue.

This ensures that at all times, the window only contains unique characters.

## JavaScript Implementation

```javascript
function lengthOfLongestSubstring(s) {
  let charSet = new Set();
  let maxLen = 0;
  let left = 0;

  for (let right = 0; right < s.length; right++) {
    while (charSet.has(s[right])) {
      charSet.delete(s[left]);
      left++;
    }
    charSet.add(s[right]);
    maxLen = Math.max(maxLen, right - left + 1);
  }
  return maxLen;
}
```

## Complexity Analysis

- **Time Complexity:** $O(n)$ — Each character is considered at most twice (once added by `right`, once removed by `left`).
- **Space Complexity:** $O(k)$ — At most $k$ unique characters are in the set at any one time, where $k$ is the size of the character set.

---

You now have a powerful template for substring and subarray problems that require tracking "unique" constraints with fast lookups. 

**Next pattern**: Try one more advanced variation—**Longest Repeating Character Replacement (LeetCode 424)**—where you’ll track *character frequencies* (counts) in the window instead of just uniqueness!
---

### 🚫 Why Not Use a String Instead of a Set?

You might wonder: *What if I tried building a string and removing its characters directly, rather than using a Set?*

#### Example:

Here's a (naive) alternative: use a substring to represent the current window, and check for character inclusion with `indexOf`, `slice`, and concatenation.

```javascript
function lengthOfLongestSubstring_naive(s) {
  let window = "";
  let maxLen = 0;
  for (let char of s) {
    const dupIndex = window.indexOf(char);
    if (dupIndex !== -1) {
      // Remove left side up to (and including) the first occurrence of char
      window = window.slice(dupIndex + 1);
    }
    window += char;
    maxLen = Math.max(maxLen, window.length);
  }
  return maxLen;
}
```

#### Why is Set Better?

- **Speed**: `Set.has()` and `Set.delete()` are average $O(1)$ due to hashing. But for a string, `indexOf` and `slice` are $O(k)$ ($k$ = current window size).
- **Efficiency**: Each duplication triggers a *scan* of the string window (linear), whereas Set does direct lookup.
- **Scalability**: As the input grows, the inefficiency of string search/removal becomes significant, making it much slower.

**Summary:**  
Manipulating a string window *works* for small inputs or interviews, but the Set approach is much faster ($O(n)$ overall vs up to $O(n^2)$ for the string version in the worst case). That's why you'll see Set as the standard choice in sliding window "unique substring" problems.


# Longest Repeating Character Replacement: Sliding Window + Frequency Map

> **Related NeetCode Problem:**  
> [Longest Repeating Character Replacement (LeetCode 424)](https://neetcode.io/problems/longest-repeating-character-replacement)

When asked to find the longest substring under the condition that you can change at most `k` characters, use a **Sliding Window** paired with a **frequency map**.

---

## Core Logic: The "Golden Equation"

To determine if the current window is valid, you don't need to manually check every single character.  
The key: **just track the *highest frequency* of any single character currently inside the window**.

Mathematically:

<div class="admonition admonition-tip">
  <p><strong>Characters to Replace</strong> = <code>window size</code> <strong>−</strong> <code>max frequency</code></p>
</div>

- If <code>Characters to Replace ≤ k</code>: Window is valid. Expand the window and update max length.
- If <code>Characters to Replace &gt; k</code>: Window is invalid. Shrink it from the left until it becomes valid again.

---

## JavaScript Implementation

```javascript
/**
 * @param {string} s
 * @param {number} k
 * @return {number}
 */
function characterReplacement(s, k) {
  const count = new Map();
  let maxFreq = 0;
  let maxLen = 0;
  let left = 0;

  for (let right = 0; right < s.length; right++) {
    count.set(s[right], (count.get(s[right]) ?? 0) + 1);
    maxFreq = Math.max(maxFreq, count.get(s[right]));

    // Window is invalid if replacements needed > k
    while (right - left + 1 - maxFreq > k) {
      count.set(s[left], count.get(s[left]) - 1);
      left++;
      // No need to recompute maxFreq here for correctness;
      // the window will self-correct as the loop progresses.
    }

    maxLen = Math.max(maxLen, right - left + 1);
  }

  return maxLen;
}
```

---

## Complexity Analysis

- **Time Complexity:** $O(n)$ – Each pointer scans the string at most once; updating and querying character counts is $O(1)$.
- **Space Complexity:** $O(1)$ – At most 26 entries for uppercase letters, so memory is constant.

---

**Summary:**  
By tracking the max frequency inside the window and comparing the window size to it, you efficiently determine if it's possible to replace enough characters to make the whole window identical—with sliding window, it's optimal.

For more details and explanations, see the [NeetCode discussion and video walkthrough.](https://neetcode.io/problems/longest-repeating-character-replacement)