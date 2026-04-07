---
title: Binary Search
description: Introduction to the binary search technique — when to use it, how to spot it, and a walkthrough of the classic Binary Search problem.
---

# Binary Search

**Binary Search** is one of the most fundamental and efficient search algorithms in computer science. Instead of scanning every element one by one ($O(n)$), binary search repeatedly divides the search space in half, achieving $O(\log n)$ time complexity.

---

## When to Use Binary Search

Binary search should be your go-to whenever you see these signals:

- ✅ **The input is sorted** (or has a monotonic property) — this is the #1 giveaway.
- ✅ You're asked to **find a target** or **find a position** (e.g., insertion point, first/last occurrence).
- ✅ The problem asks for $O(\log n)$ time complexity explicitly.
- ✅ You need to **minimize or maximize** a value over a sorted/monotonic search space (binary search on the answer).

:::tip How to Spot It
If the problem hands you a **sorted array** and asks you to find something — think binary search immediately. Even if the word "sorted" isn't used, look for **monotonic** behavior: any property where "if it's true at index `i`, it's true for all indices to the right (or left)" can be binary-searched.
:::

### Common Problem Types

| Pattern | Example |
|---|---|
| Find exact target in sorted array | Classic Binary Search |
| Find insertion position | Search Insert Position (LC 35) |
| Find first/last occurrence | First & Last Position of Element (LC 34) |
| Search in rotated sorted array | Search in Rotated Sorted Array (LC 33) |
| Minimize/maximize over answer space | Koko Eating Bananas (LC 875), Split Array Largest Sum (LC 410) |

---

## How It Works

The idea is simple: maintain two pointers (`start` and `end`) defining the current search range, and repeatedly check the middle element:

1. **Calculate the midpoint**: `mid = Math.floor(start + (end - start) / 2)`
2. **Compare `nums[mid]` to the target**:
   - If `nums[mid] === target` → found it, return `mid`.
   - If `nums[mid] > target` → the target must be in the **left half**, so move `end = mid - 1`.
   - If `nums[mid] < target` → the target must be in the **right half**, so move `start = mid + 1`.
3. **Repeat** until `start` meets or passes `end`.

:::info Why `start + (end - start) / 2` instead of `(start + end) / 2`?
Both compute the same midpoint, but `start + (end - start) / 2` avoids potential **integer overflow** when `start + end` exceeds the max integer size. In JavaScript this isn't strictly necessary (numbers are floats), but it's a best practice that interviewers love to see — and it's critical in languages like Java or C++.
:::

### Visual Walkthrough

Searching for `target = 5` in `[1, 2, 3, 4, 5, 6, 7]`:

```
Step 1:  [1, 2, 3, 4, 5, 6, 7]
          ^        ^        ^
        start     mid      end
         mid = 4 → nums[3] = 4 < 5 → move start to mid + 1

Step 2:  [1, 2, 3, 4, 5, 6, 7]
                     ^  ^     ^
                   start mid  end
         mid = 5 → nums[5] = 6 > 5 → move end to mid - 1

Step 3:  [1, 2, 3, 4, 5, 6, 7]
                     ^
                 start/end/mid
         mid = 4 → nums[4] = 5 === 5 ✅ Found!
```

Each step **eliminates half** of the remaining elements. That's why it's $O(\log n)$.

---

# Binary Search (NeetCode / LeetCode 704)

> **Problem:**  
> [Binary Search — NeetCode](https://neetcode.io/problems/binary-search) / [LeetCode 704](https://leetcode.com/problems/binary-search/)

:::note Problem
Given a sorted array of integers `nums` and an integer `target`, return the index of `target` in the array. If `target` does not exist in the array, return `-1`.

You must write an algorithm with $O(\log n)$ runtime complexity.

- **Input:** `nums = [-1, 0, 3, 5, 9, 12]`, `target = 9`
- **Output:** `4`
:::

---

## Algorithm Step-by-Step

1. **Handle the edge case**: If the array has only one element, directly check if it matches the target.
2. **Initialize pointers**: `start = 0`, `end = nums.length - 1`.
3. **Loop while `start < end`**:
   - Compute `mid = Math.floor(start + (end - start) / 2)`.
   - If `nums[mid] === target`, return `mid` immediately.
   - If `nums[mid] > target`, the target is to the left → set `end = mid - 1`.
   - If `nums[mid] < target`, the target is to the right → set `start = mid + 1`.
4. **Post-loop check**: When the loop exits, `start === end`. Check if `nums[start] === target` — if so, return `start`; otherwise, return `-1`.

:::tip Why the post-loop check?
The `while (start < end)` condition exits when `start === end`, meaning there's one element left that was never checked as a `mid`. We need to verify it before returning `-1`.
:::

---

## JavaScript Implementation

```javascript
class Solution {
    /**
     * @param {number[]} nums
     * @param {number} target
     * @return {number}
     */
    search(nums, target) {
        if (nums.length === 1) {
            return nums[0] === target ? 0 : -1;
        }

        let start = 0;
        let end = nums.length - 1;

        while (start < end) {
            let mid = Math.floor(start + (end - start) / 2);

            if (nums[mid] === target) {
                return mid;
            }

            if (nums[mid] > target) {
                end = mid - 1;
            } else {
                start = mid + 1;
            }
        }

        return nums[start] === target ? start : -1;
    }
}
```

---

## Complexity Analysis

:::tip Complexity
- **Time Complexity:** $O(\log n)$ — Each iteration halves the search space, so it takes at most $\log_2(n)$ steps.
- **Space Complexity:** $O(1)$ — Only a few variables (`start`, `end`, `mid`) are used regardless of input size.
:::

---

## Common Pitfalls

| Mistake | Why It's Wrong | Fix |
|---|---|---|
| Using `(start + end) / 2` | Integer overflow in other languages | Use `start + (end - start) / 2` |
| `while (start <= end)` without early return | Can lead to infinite loops if not careful | Make sure to return inside the loop or use `start < end` with a post-loop check |
| Forgetting the post-loop check with `start < end` | Misses the last remaining element | Always check `nums[start]` after the loop |
| Off-by-one on `end = mid` vs `end = mid - 1` | May re-check the same `mid` forever | Since we already checked `nums[mid]`, safely exclude it with `mid - 1` |

---

# Search a 2D Matrix (NeetCode / LeetCode 74)

> **Problem:**  
> [Search a 2D Matrix — NeetCode](https://neetcode.io/problems/search-2d-matrix) / [LeetCode 74](https://leetcode.com/problems/search-a-2d-matrix/)

:::note Problem
You are given an `m × n` matrix where:
- Each row is sorted in ascending order.
- The first element of each row is greater than the last element of the previous row.

Given a `target`, return `true` if it exists in the matrix, or `false` otherwise. You must write a solution in $O(\log(m \times n))$ time.

- **Input:** `matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]]`, `target = 3`
- **Output:** `true`
:::

---

## The Key Insight: Flatten the Matrix Mentally

Because each row is sorted **and** the first element of row `i+1` is greater than the last element of row `i`, the entire matrix is effectively **one long sorted array** if you read it row by row:

```
Matrix:           Flattened (virtual):
[1,  3,  5,  7]    → [1, 3, 5, 7, 10, 11, 16, 20, 23, 30, 34, 60]
[10, 11, 16, 20]      index: 0  1  2  3   4   5   6   7   8   9  10  11
[23, 30, 34, 60]
```

You don't need to actually flatten it — just run a standard binary search on indices `0` to `rows * columns - 1`, and **convert the flat index to a row/column** on the fly:

:::tip The Index Conversion Trick
For a matrix with `columns` columns, a flat index `mid` maps to:
- **Row:** `Math.floor(mid / columns)`
- **Column:** `mid % columns`
:::

### Understanding the Row: Division (`/`)

This one is intuitive — if each row holds `columns` elements, dividing the flat index by `columns` tells you **how many full rows fit before this index**:

```
columns = 4

index 0–3   → 0/4=0, 1/4=0, 2/4=0, 3/4=0   → Row 0
index 4–7   → 4/4=1, 5/4=1, 6/4=1, 7/4=1   → Row 1
index 8–11  → 8/4=2, 9/4=2, 10/4=2, 11/4=2  → Row 2
```

Every 4 elements, we "move to the next row". Integer division naturally counts how many groups of 4 we've completed.

### Understanding the Column: Modulo (`%`)

The column is the **remainder** — after you've figured out which row you're in, `%` tells you **how far into that row** you are. Think of it as: *"I've used up full rows, what's left over?"*

```
columns = 4

index 5 → 5 % 4 = 1  →  Column 1
          (5 items in. 4 fill row 0, 1 left over → position 1 in row 1)

index 7 → 7 % 4 = 3  →  Column 3
          (7 items in. 4 fill row 0, 3 left over → position 3 in row 1)

index 8 → 8 % 4 = 0  →  Column 0
          (8 items in. 8 fills rows 0+1 exactly, 0 left over → first position in row 2)
```

:::info Another way to think about it
Imagine you're filling seats in a theater with 4 seats per row. Person #5 (index 5):
- **Row:** `5 ÷ 4 = 1` (they're in the 2nd row)
- **Seat:** `5 % 4 = 1` (they're in the 2nd seat of that row)

The modulo is literally "which seat in the current row?" — once you subtract all the full rows, what's the position within the current one.
:::

### Quick Reference Table (4 columns)

| Flat Index | `÷ 4` (Row) | `% 4` (Column) | Matrix Position |
|---|---|---|---|
| 0 | 0 | 0 | `matrix[0][0]` |
| 3 | 0 | 3 | `matrix[0][3]` |
| 4 | 1 | 0 | `matrix[1][0]` |
| 6 | 1 | 2 | `matrix[1][2]` |
| 9 | 2 | 1 | `matrix[2][1]` |
| 11 | 2 | 3 | `matrix[2][3]` |

### Visual Example

Searching for `target = 16` in a 3×4 matrix:

```
Flat indices:  0  1  2  3  4  5  6  7  8  9  10  11
Values:        1  3  5  7  10 11 16 20 23 30  34  60

Step 1: start=0, end=11 → mid=5 → row=5/4=1, col=5%4=1 → matrix[1][1]=11 < 16 → start=6
Step 2: start=6, end=11 → mid=8 → row=8/4=2, col=8%4=0 → matrix[2][0]=23 > 16 → end=7
Step 3: start=6, end=7  → mid=6 → row=6/4=1, col=6%4=2 → matrix[1][2]=16 ✅ Found!
```

---

## JavaScript Implementation

```javascript
class Solution {
    /**
     * @param {number[][]} matrix
     * @param {number} target
     * @return {boolean}
     */
    searchMatrix(matrix, target) {
        let rows = matrix.length;
        let columns = matrix[0].length;

        if (rows === 1 && columns === 1) {
            return matrix[0][0] === target;
        }

        // Treat the matrix as one long sorted array
        let start = 0;
        let end = rows * columns - 1;

        while (start <= end) {
            let mid = Math.floor(start + (end - start) / 2);

            // Convert flat index → matrix coordinates
            let midRow = Math.floor(mid / columns);
            let midColumn = mid % columns;

            const midValue = matrix[midRow][midColumn];

            if (midValue === target) {
                return true;
            }

            if (midValue < target) {
                start = mid + 1;
            } else {
                end = mid - 1;
            }
        }

        return false;
    }
}
```

---

## Algorithm Step-by-Step

1. **Treat it as a 1D array**: Compute the total number of elements as `rows * columns`. Set `start = 0` and `end = rows * columns - 1`.
2. **Standard binary search loop** (`while start <= end`):
   - Compute `mid` using the overflow-safe formula.
   - **Convert `mid` to matrix coordinates**: `midRow = mid / columns` (integer division), `midColumn = mid % columns`.
   - Compare `matrix[midRow][midColumn]` to `target` and adjust `start` / `end` as in classic binary search.
3. **Return `false`** if the loop exits without finding the target.

:::info Why `start <= end` here (not `start < end`)?
This version returns `true` inside the loop when the match is found, so we use `<=` to ensure every element is checked — including when `start === end`. There's no post-loop check needed because the only exit path without a match means the target doesn't exist.
:::

---

## Complexity Analysis

:::tip Complexity
- **Time Complexity:** $O(\log(m \times n))$ — Binary search over all `m × n` elements.
- **Space Complexity:** $O(1)$ — No extra data structures; just a few index variables.
:::
