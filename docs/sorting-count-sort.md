## Counting Sort: Explanation and When to Use It

Counting Sort is a specialized sorting algorithm ideal for sorting arrays of integers when you know the range of possible values ahead of time (i.e., you know both the `min` and `max` possible numbers in your data set). It is **not a comparison-based algorithm**. Its time complexity can approach `O(n + k)`, where `n` is the number of elements, and `k` is the range (`max - min + 1`).

### When should you use Counting Sort?

- The **input must be integers** (or can be mapped to integers).
- You **know the minimum and maximum values** in the array, and the **range (max - min)** is not much larger than the number of items (otherwise, it will use a lot of memory).
- Useful for sorting things like grades, ages, or items with limited/known categories.

### How does it work?

1. **Determine the Range**  
   Identify the `min` and `max` values in your array. The size of the counting array (`counts`) should be `max - min + 1` to account for every possible number in that range.

2. **Counting Phase**  
   Iterate over the array and count how many times each distinct value occurs. Each index in the `counts` array corresponds to a unique number in your input (offset by `min` if `min` is not 0).

3. **Reconstruction Phase**  
   Go through `counts` and rebuild the sorted array: for each index, append its value to the result as many times as it was counted.

### Example Implementation

```javascript
/**
 * Counting Sort for an array with values in [minVal, maxVal]
 * @param {number[]} nums - Array of integers
 * @param {number} minVal - Minimum possible value in nums
 * @param {number} maxVal - Maximum possible value in nums
 * @returns {number[]} Sorted array
 */
function countingSort(nums, minVal, maxVal) {
  // 1. Size of counts array is maxVal - minVal + 1
  const range = maxVal - minVal + 1;
  const counts = new Array(range).fill(0);

  // 2. Count occurrences, shifting index by minVal
  for (let num of nums) {
    counts[num - minVal]++;
  }

  // 3. Reconstruct sorted array
  const sorted = [];
  for (let i = 0; i < range; i++) {
    while (counts[i] > 0) {
      sorted.push(i + minVal);
      counts[i]--;
    }
  }
  return sorted;
}
```

#### Example: How to pick the size of array?

Suppose `nums = [4, 2, 6, 5]`, so `minVal = 2`, `maxVal = 6`.  
Then `range = 6 - 2 + 1 = 5`, so `counts = [0,0,0,0,0]`, which corresponds to values `[2, 3, 4, 5, 6]`.

#### Pros and Cons

- **Pros:** Very fast for small ranges, avoids comparisons, works in linear time (`O(n + k)`).
- **Cons:** Uses extra space for the counts array (can be wasteful if `max - min` is large), only works well when range is manageable.

---

:::tip
**Use Counting Sort when you have a known, small integer range!**
:::

:::info

You might notice that the principles of **Counting Sort** are closely related to those used in the ["Top K Frequent Elements"](https://neetcode.io/problems/top-k-elements-in-list) problem. In Counting Sort, we create an array indexed by value to efficiently count how often each number appears.

For the Top K Frequent Elements exercise, we want to quickly identify the numbers that appear most often. The trick is to organize or "bucket" numbers by their frequency of occurrence. For this, we use a variation called **Bucket Sort**:

- Instead of using the index to represent values (like in Counting Sort), we use the index to represent _number of occurrences_ (frequency). Each bucket at index `i` contains all numbers that appear `i` times in the original array.

**Counting Sort is a special case of Bucket Sort** because it uses the same idea of binning elements—in fact, when the value range is small, these approaches overlap perfectly.

:::

#### Example: Using Buckets to Solve "Top K Frequent Elements"

Suppose our input is:

```javascript
const nums = [1, 1, 2, 2, 2, 3];
const k = 2;
```

1. **Count the frequencies:**

| Number | Frequency |
| ------ | --------- |
| 1      | 2         |
| 2      | 3         |
| 3      | 1         |

2. **Bucket sort by frequency:**

Create an array of buckets, where the index is the frequency, and each bucket contains a list of numbers seen that many times:

```javascript
// Initialize buckets: index is frequency (0 to max frequency)
const buckets = Array(nums.length + 1)
  .fill()
  .map(() => []);
// Fill buckets:
for (const [num, freq] of Object.entries(frequencies)) {
  buckets[freq].push(Number(num));
}
```

After populating:

- `buckets[1] = [3]` // Numbers that appear once
- `buckets[2] = [1]` // Numbers that appear twice
- `buckets[3] = [2]` // Numbers that appear three times

3. **Collect top k frequent:**

Loop from the highest frequency toward 1, appending numbers until we've collected k:

```javascript
const result = [];
for (let freq = buckets.length - 1; freq > 0 && result.length < k; freq--) {
  for (const num of buckets[freq]) {
    result.push(num);
    if (result.length === k) break;
  }
}
// result = [2, 1] (the two most frequent numbers)
```

---

**Summary:**

- Counting Sort and Bucket Sort both group elements by some property using indexed arrays.
- For "Top K Frequent Elements," bucket sort groups by frequency, _not_ value.
- Counting Sort is a special case where grouping is by element value within a small known range.

## Is Counting Sort Stable?

**By default, basic Counting Sort is _not stable_.**  
A stable sort keeps equal elements in the same original order—important when you need to preserve "duplicates" order in the output.

Let’s see this in action with a tiny example:

**Suppose we want to stably sort:**  
`[2, 1, 2]`

To make things clear, imagine we label the two 2s:

- `2_A` (the first 2)
- `2_B` (the second 2)

Our array is: `[2_A, 1, 2_B]`  
A stable sort MUST return: `[1, 2_A, 2_B]`  
— **2_A should remain before 2_B.**

---

### Step-by-step: Stable Counting Sort

#### **Step 1: Count occurrences**

Build a counts array with one slot for each value (e.g., for values 0–2):

| Number | Count |
| ------ | ----- |
| 0      | 0     |
| 1      | 1     |
| 2      | 2     |

That is:  
`counts = [0, 1, 2]`

---

#### **Step 2: Compute cumulative counts ("ticket counters")**

Transform counts into a prefix sum, so each slot tells us **the ending position** for each value:

- For index 0: `0` (remains 0)
- For index 1: `counts[0] + counts[1] = 0 + 1 = 1`
- For index 2: `counts[1] + counts[2] = 1 + 2 = 3`

After processing:  
`counts = [0, 1, 3]`

- Now: "The last '1' ends at position 1, the last '2' at position 3"

---

#### **Step 3: Fill output array _backwards_ to preserve stability**

Create an empty output array: `[ _, _, _ ]`

**Process input right-to-left**  
This is the step that _guarantees_ stability!

Original: `[2_A, 1, 2_B]` (process from rightmost to leftmost element)

- Rightmost: `2_B`

  - `counts[2]` = 3 ⇒ place at index 2
  - Decrement `counts[2]` to 2
  - Output: `[ _, _, 2_B ]`

- Next: `1`

  - `counts[1]` = 1 ⇒ place at index 0
  - Decrement `counts[1]` to 0
  - Output: `[ 1, _, 2_B ]`

- Leftmost: `2_A`
  - `counts[2]` = 2 ⇒ place at index 1
  - Decrement `counts[2]` to 1
  - Output: `[ 1, 2_A, 2_B ]`

**Result:**  
`[1, 2_A, 2_B]`  
Now, `2_A` comes before `2_B` &mdash; the algorithm is stable!

---

### **Stable Counting Sort Implementation (JavaScript)**

```javascript
function stableCountingSort(arr) {
  if (arr.length === 0) return arr.slice();
  const max = Math.max(...arr);
  const min = Math.min(...arr);

  // 1. Count occurrences
  const count = Array(max - min + 1).fill(0);
  for (let num of arr) {
    count[num - min]++;
  }

  // 2. Compute prefix sums
  for (let i = 1; i < count.length; i++) {
    count[i] += count[i - 1];
  }

  // 3. Fill output backwards for stability
  const output = Array(arr.length);
  for (let i = arr.length - 1; i >= 0; i--) {
    const num = arr[i];
    const idx = count[num - min] - 1;
    output[idx] = num;
    count[num - min]--;
  }
  return output;
}

// Demo:
const input = [2, 1, 2];
console.log(stableCountingSort(input)); // [1, 2, 2]
```

---

### **TL;DR**

- Basic counting sort isn’t stable, but it's easy to make it stable:
  - **Prefix sums**: turn counts into "last position" tickets
  - **Iterate right-to-left**: when placing, so equal elements retain order

_In most interviews, it's enough to say:_

> "Naive counting sort isn't stable, but we can make it stable using a prefix sum and outputting from right to left."

That shows you know the difference!

## Practice Problems: Testing Counting Sort

The best way to internalize Counting Sort is to recognize the **patterns and constraints** in LeetCode problems that scream _"Use Counting Sort!"_ (usually a very small, fixed range of integers).

Here are the top three problems to practice:

### 1. [LeetCode 75: Sort Colors](https://leetcode.com/problems/sort-colors/) (Medium)

- **The Pattern:** You are sorting an array containing only `0`, `1`, and `2`. The range is exactly 3 (`k = 3`).
- **How to apply it:** Do a 2-pass Counting Sort. Pass 1: Count the occurrences of 0s, 1s, and 2s. Pass 2: Overwrite the original array with the exact number of 0s, then 1s, then 2s.

### Solution: LeetCode 75 (Sort Colors) using Counting Sort

```javascript
/**
 * @param {number[]} nums
 * @return {void} Do not return anything, modify nums in-place instead.
 */
var sortColors = function (nums) {
  // 1. The range is strictly 0, 1, 2. No offset needed.
  const counts = [0, 0, 0];

  // 2. Count frequencies
  for (const n of nums) {
    counts[n]++;
  }

  // 3. Reconstruct in-place
  let insertIndex = 0;

  // Loop through our 3 possible colors (0, 1, 2)
  for (let color = 0; color < 3; color++) {
    // While we still have counts of this color left
    while (counts[color] > 0) {
      nums[insertIndex] = color; // Overwrite the original array
      insertIndex++; // Move to the next spot
      counts[color]--; // Decrement the bucket
    }
  }
};
```

### 2. [LeetCode 1122: Relative Sort Array](https://leetcode.com/problems/relative-sort-array/) (Easy)

**The Pattern:** Sort `arr1` based on the exact order of elements in `arr2`. Any leftover elements not present in `arr2` should be appended at the end in ascending order.

- **The Pattern:** Sort `arr1` based on the order of elements in `arr2`.
- **The Giveaway:** The problem constraints state `0 <= arr1[i] <= 1000`. This tiny, fixed range is a massive hint to use a frequency array of size 1001.
- **How to apply it:** Build a `counts` array for `arr1`. Loop through `arr2` and append to your result array based on the frequencies in `counts`. Append whatever is left over at the end.

:::tip Interview Talking Point: Dynamic vs. Fixed Constraints
The constraints for this problem state `0 <= arr1[i] <= 1000`.
In an interview, you can mention: _"Because of these constraints, we could just hardcode an array of size 1001. However, I am calculating the `min` and `max` dynamically to make the function completely reusable and scalable for any dataset."_ Interviewers love this pragmatic approach.
:::

#### My Solution

```javascript
/**
 * @param {number[]} arr1
 * @param {number[]} arr2
 * @return {number[]}
 */
var relativeSortArray = function (arr1, arr2) {
  // 1. Create a counts array from arr1 (find range dynamically)
  const max = Math.max(...arr1);
  const min = Math.min(...arr1);
  const countsLength = max - min + 1;

  const counts = Array.from({ length: countsLength }).fill(0);

  // 2. Populate the counts array
  for (const el of arr1) {
    // We remove min in case it is bigger than 0 (array will be 0 indexed)
    counts[el - min]++;
  }

  // 3. Create the output array
  const outputArray = [];

  // 4. Respect the order of arr2: add as many items as the count array says
  for (const el of arr2) {
    while (counts[el - min] > 0) {
      outputArray.push(el);
      counts[el - min]--;
    }
  }

  // 5. Add the extra nums that were not in arr2
  // Iterating over the counts array automatically adds them in ascending order
  for (let i = 0; i < counts.length; i++) {
    while (counts[i] > 0) {
      outputArray.push(i + min); // Add the min back to get the original value
      counts[i]--;
    }
  }

  return outputArray;
};
```

### 3. [LeetCode 912: Sort an Array](https://leetcode.com/problems/sort-an-array/) (Medium)

- **The Pattern:** Sort an array without built-in functions. Constraints say numbers range from `-50,000` to `50,000`.
- **How to apply it:** This is the ultimate test for your core Counting Sort implementation. Because there are negative numbers, you _must_ use the `minVal` offset trick (`counts[num - minVal]++`) to avoid negative array indices.

**The Pattern:** This tests your ability to handle negative numbers using the `min` offset technique. Because values drop to `-50,000`, we must shift everything up so the lowest number maps to index `0`.

:::tip Interview Flex: Time vs. Space Trade-offs
In this solution, we hardcode the `min` and `max` based on the problem constraints to save the time of scanning the array. However, this always allocates an array of size 100,001. In a real-world scenario where the array might just be `[-5, 0, 5]`, it is better to dynamically find the `min` and `max` first to save memory.
:::

#### My Solution

```javascript
/**
 * @param {number[]} nums
 * @return {number[]}
 */
var sortArray = function (nums) {
  // 1. Hardcode bounds based on problem constraints
  const min = -50000;
  const max = 50000;

  const countsLength = max - min + 1;

  // 2. Initialize the frequency array
  const counts = Array.from({ length: countsLength }).fill(0);

  // 3. Count frequencies, offsetting negative numbers to index 0+
  for (const num of nums) {
    counts[num - min]++;
  }

  // 4. Reconstruct the sorted array
  const output = [];
  for (let i = 0; i < counts.length; i++) {
    while (counts[i] > 0) {
      output.push(i + min); // Add the min offset back to get the real number
      counts[i]--;
    }
  }

  return output;

  /* Optional In-Place Optimization:
    Instead of creating 'output', use a pointer and overwrite nums directly:
    let index = 0;
    for (let i = 0; i < counts.length; i++) {
        while (counts[i] > 0) {
            nums[index++] = i + min;
            counts[i]--;
        }
    }
    return nums;
    */
};
```
