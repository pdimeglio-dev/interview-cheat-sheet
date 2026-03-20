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
- Instead of using the index to represent values (like in Counting Sort), we use the index to represent *number of occurrences* (frequency). Each bucket at index `i` contains all numbers that appear `i` times in the original array.

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
|--------|-----------|
|   1    |     2     |
|   2    |     3     |
|   3    |     1     |

2. **Bucket sort by frequency:**

Create an array of buckets, where the index is the frequency, and each bucket contains a list of numbers seen that many times:

```javascript
// Initialize buckets: index is frequency (0 to max frequency)
const buckets = Array(nums.length + 1).fill().map(() => []);
// Fill buckets:
for (const [num, freq] of Object.entries(frequencies)) {
    buckets[freq].push(Number(num));
}
```

After populating:
- `buckets[1] = [3]`  // Numbers that appear once
- `buckets[2] = [1]`  // Numbers that appear twice
- `buckets[3] = [2]`  // Numbers that appear three times

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
- For "Top K Frequent Elements," bucket sort groups by frequency, *not* value.
- Counting Sort is a special case where grouping is by element value within a small known range.


## Is Counting Sort Stable?

**By default, basic Counting Sort is *not stable*.**  
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
|--------|-------|
|   0    |   0   |
|   1    |   1   |
|   2    |   2   |

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

#### **Step 3: Fill output array *backwards* to preserve stability**

Create an empty output array: `[ _, _, _ ]`

**Process input right-to-left**  
This is the step that *guarantees* stability!

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

*In most interviews, it's enough to say:*  
> "Naive counting sort isn't stable, but we can make it stable using a prefix sum and outputting from right to left."

That shows you know the difference!