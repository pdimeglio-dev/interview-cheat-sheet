# Longest Consecutive Sequence

**The Problem:**  
Given an unsorted array of integers, return the length of the longest consecutive elements sequence. You must write an algorithm that runs in `O(n)` time.

---

## ⚠️ JavaScript Quirk: `Math.max` and `Math.min`

Before diving into the solutions, here is a crucial JavaScript reminder when working with algorithms:

:::danger Never pass an array directly to `Math.max()` or `Math.min()`
`Math.max([1, 2, 3])` will return `NaN`.  
**Why?** The `Math.max()` and `Math.min()` functions expect individual arguments (e.g., `Math.max(1, 2, 3)`), not an array. To pass an array, you **must use the spread operator** (`...`) to unpack the array into individual arguments:  
```
Math.max(...nums)
```
:::

---

## Approach 1: Counting Sort (Failed ❌)

Since we just learned Counting Sort, it's tempting to apply it here to sort the array in $O(N)$ time. However, this is a perfect example of why **checking the constraints** is critical.

**Why it fails:**  
The constraints for this problem usually allow numbers between `-10^9` and `10^9`.  
If your array is `[-1000000000, 1000000000]`, the `arraySize` (`max - min + 1`) will be `2,000,000,001`.  
JavaScript will instantly crash with a **Memory Limit Exceeded (MLE)** error because you are trying to allocate a massive array for just two numbers.

```javascript
/**
 * @param {number[]} nums
 * @return {number}
 */
var longestConsecutive = function(nums) {
    if (nums.length === 0) return 0;

    // 1. Find bounds (must use spread operator!)
    const min = Math.min(...nums);
    const max = Math.max(...nums);

    // FATAL FLAW: This range is way too big if min is -10^9 and max is 10^9.
    const arraySize = max - min + 1;
    const counts = Array.from({ length: arraySize }).fill(0);

    // 2. Count frequencies with min offset
    for (const num of nums) {
        counts[num - min]++;
    }

    // 3. Reconstruct unique sorted array
    const sorted = [];
    for (let i = 0; i < counts.length; i++) {
        if (counts[i] > 0) {
            sorted.push(i + min);
        }
    }

    // 4. Find the max consecutive sequence
    let maxSeq = 1;
    let currSeq = 1;
    for (let i = 1; i < sorted.length; i++) {
        if (sorted[i] === sorted[i - 1] + 1) {
            currSeq++;
            maxSeq = Math.max(maxSeq, currSeq);
        } else {
            currSeq = 1;
        }
    }
    return maxSeq;
};
```

---

## Approach 2: Standard Sorting $O(N \log N)$ (Sub-optimal ⚠️)

Since Counting Sort takes too much memory, the immediate fallback is to just use JavaScript's built-in sort. This works and avoids the memory issue, but since `nums.sort()` runs in $O(N \log N)$ time, it technically violates the strict $O(N)$ time requirement of the problem.

```javascript
/**
 * @param {number[]} nums
 * @return {number}
 */
var longestConsecutive = function(nums) {
    if (nums.length === 0) return 0;

    // Sort ascending. Time Complexity: O(N log N)
    nums.sort((a, b) => a - b);

    let maxSeq = 1;
    let currSeq = 1;

    for (let i = 1; i < nums.length; i++) {
        // Skip duplicates
        if (nums[i] === nums[i - 1]) continue;

        // Count consecutive numbers
        if (nums[i] === nums[i - 1] + 1) {
            currSeq++;
            maxSeq = Math.max(maxSeq, currSeq);
        } else {
            currSeq = 1;
        }
    }
    return maxSeq;
};
```

---

## Approach 3: The Optimal Set Solution (Passed ✅)

To achieve $O(N)$ time, we need $O(1)$ lookups. We can use a Set.

**The trick:**  
We only start counting a sequence if the current number is the absolute beginning of a sequence.
We know a number is the start of a sequence if the number right below it (`num - 1`) does **not** exist in the Set.

:::tip Why is this O(N)?
Even though there is a while loop inside a for loop, the inner while loop only ever runs when we are at the start of a sequence. Every number is visited at most twice (once in the outer loop, once in the inner loop), making it strict $O(N)$.
:::

```javascript
/**
 * @param {number[]} nums
 * @return {number}
 */
var longestConsecutive = function(nums) {
    if (nums.length === 0) {
        return 0;
    }

    // 1. Remove duplicates and enable O(1) lookups
    const numSet = new Set(nums);
    let maxCount = 0;

    for (const num of numSet) {
        // 2. Check if this element is the START of a sequence
        // (If num - 1 exists, we are in the middle of a sequence, so skip it)
        if (!numSet.has(num - 1)) {
            let count = 1;
            let current = num;

            // 3. Keep counting upwards as long as the next number exists
            while (numSet.has(current + 1)) {
                current += 1;
                count += 1;
            }

            maxCount = Math.max(maxCount, count);
        }
    }

    return maxCount;
};
```