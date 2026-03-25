# The Two Pointers Pattern

The **Two Pointers** pattern is a common algorithmic technique used to solve problems involving linear data structures like arrays and strings. The idea is to use two distinct pointers (indices or variables) that traverse the structure at different speeds or from opposite ends, often to accomplish tasks more efficiently than simple iteration.

### When to Use Two Pointers

- Searching for pairs in a sorted array (e.g., sum problems)
- Reversing or comparing elements in place
- Removing or partitioning elements (like moving zeroes)
- Scanning from both ends to check conditions (palindrome check, trapping water, etc.)
- Sliding window problems

### How It Works

The two pointers can move towards each other (one from the start, one from the end), or move independently based on logic. The main advantage is that it often reduces time complexity by processing the data in a single pass or by avoiding unnecessary work.

### Example Template

```javascript
let left = 0;
let right = arr.length - 1;
while (left < right) {
    // Do something with arr[left] and arr[right]
    left++;
    right--;
}
```

### Visual Intuition

Suppose you have an array:

```
[ 1, 2, 3, 4, 5 ]
  ^           ^
 left       right
```
You can compare, swap, or sum `arr[left]` and `arr[right]`, then move the pointers inward.

---

The Two Pointers technique is widely applicable for optimizing many classic problems in coding interviews!



# Valid Palindrome: Two Pointers Pattern

When checking if a string reads the same forward and backward, the **Two Pointers** pattern is one of the most efficient approaches. Instead of creating a reversed copy of the string to compare against, we can place one pointer at the beginning and one at the end, moving them toward the center.

## Handling Alphanumeric Characters in JavaScript

The classic "Valid Palindrome" problem requires us to ignore spaces, punctuation, and case differences. We only care about letters (`a-z`) and numbers (`0-9`). 

There are two main ways to handle this using Regular Expressions (Regex):

### 1. The `.replace()` Method (For cleaning the whole string)

If we want to strip out everything that isn't a letter or a number from a string, we can use `.replace()`.

- `/[^a-z0-9]/g` means: "Find absolutely everything that is **NOT** (`^`) a lowercase letter or a number, **globally** (`g`), and replace it with an empty string."

```javascript
let cleanedString = s.toLowerCase().replace(/[^a-z0-9]/g, '');
```

---

### 2. The `.test()` Method (For checking a single character)

If we want to check characters on the fly without creating a new string, we use `.test()`. It returns true or false.

- `/[a-z0-9]/i` means: "Check if this character is a letter or number, and make it case-insensitive (`i`)."

```javascript
let isAlphaNumeric = /[a-z0-9]/i;
console.log(isAlphaNumeric.test('A')); // true
console.log(isAlphaNumeric.test(',')); // false
```

---

## Approach 1: String Cleaning (O(n) Space)

This approach cleans the entire string first, then uses two pointers to verify the palindrome.

```javascript
class Solution {
    isPalindrome(s) {
        // 1. Clean the string and make it lowercase
        let cleanedString = s.toLowerCase().replace(/[^a-z0-9]/g, '');

        // 2. Set up two pointers
        let s1 = 0;
        let s2 = cleanedString.length - 1;

        // 3. Move towards the center
        while (s1 < s2) {
            if (cleanedString.charAt(s1) !== cleanedString.charAt(s2)) {
                return false; // Mismatch found
            }
            s1++;
            s2--;
        }

        return true; 
    }
}
```

**Time Complexity:** O(n) - We iterate through the string linearly.  
**Space Complexity:** O(n) - We allocate new memory to store `cleanedString`.

---

## Approach 2: In-Place Two Pointers (O(1) Space Optimization)

To optimize our space complexity to O(1), we avoid creating a `cleanedString`. Instead, we traverse the original string and use inner while loops as "fast-forward" and "rewind" buttons to skip over any non-alphanumeric characters before we make our comparison.

#### The "Skip Before You Compare" Strategy
- **Left Pointer (`s1`)**: If it's looking at a symbol/space, keep moving it right (`s1++`) until it hits a valid character.
- **Right Pointer (`s2`)**: If it's looking at a symbol/space, keep moving it left (`s2--`) until it hits a valid character.

```javascript
class Solution {
    isPalindrome(s) {
        let s1 = 0;
        let s2 = s.length - 1;

        // Define regex outside the loop for better performance
        let isAlphaNumeric = /[a-z0-9]/i;

        while (s1 < s2) {
            // Skip non-alphanumeric characters from the left
            while (!isAlphaNumeric.test(s.charAt(s1)) && s1 < s2) {
                s1++;
            }

            // Skip non-alphanumeric characters from the right
            while (!isAlphaNumeric.test(s.charAt(s2)) && s1 < s2) {
                s2--;
            }

            // Compare the valid characters (converted to lowercase on the fly)
            if (s.charAt(s1).toLowerCase() !== s.charAt(s2).toLowerCase()) {
                return false;
            }

            s1++;
            s2--;
        }

        return true;
    }
}
```

**Time Complexity:** O(n) - Each character in the string is visited at most once.  
**Space Complexity:** O(1) - We only use a few pointers and a regex definition; no new strings are created in memory.



# Container With Most Water (Two Pointers Pattern)

[LeetCode #11 - Container With Most Water](https://leetcode.com/problems/container-with-most-water/)  
[NeetCode Solution (Video)](https://neetcode.io/problems/container-with-most-water)  

---

:::note Problem
Given `n` non-negative integers `heights` where each element represents a vertical line at position `i`, each with height `heights[i]`, find two lines that together with the x-axis form a container, such that the container contains the most water.  
Return the maximum amount of water a container can store.

- **Input:** `heights = [1,8,6,2,5,4,8,3,7]`
- **Output:** `49`
:::

---

## Why Two Pointers? (Greedy Insight)

- The area between two lines is **determined by the shorter line**:  
  `Area = min(height[left], height[right]) * (right - left)`
- The widest container is only possible with the two farthest lines.
- As we move pointers inward, *width* decreases, so to potentially increase area, the *height* must increase.
- **Greedy choice:** Always move the pointer at the shorter line, hoping for a taller one next.

---

## Implementation

```javascript
class Solution {
    maxArea(heights) {
        let left = 0;
        let right = heights.length - 1;
        let maxArea = 0;

        while (left < right) {
            // Calculate current area
            const width = right - left;
            const ht = Math.min(heights[left], heights[right]);
            const area = width * ht;

            maxArea = Math.max(maxArea, area);

            // Move in the pointer which is at the shorter line
            if (heights[left] < heights[right]) {
                left++;
            } else {
                right--;
            }
        }

        return maxArea;
    }
}
```

---

:::tip Complexity
- **Time Complexity:** $O(n)$ &mdash; Both pointers traverse the entire array at most once.
- **Space Complexity:** $O(1)$ &mdash; Only a few variables for tracking pointers and result.
:::

---

:::info Key Idea
> Move the pointer with the shorter line.  
> Never shrink the width unless you're hoping to find a taller line to increase the min-height!
:::

# Two Sum II – Input Array Is Sorted: Two Pointers Pattern

When given a **sorted** array and asked to find pairs that meet a certain condition (like a target sum) with an $O(1)$ space constraint, the **Two Pointers** pattern—moving inward from opposite ends—is the standard approach.

---

## 🧠 The Core Logic: Exploiting the Sort

Because the array is sorted in ascending order, we have a predictable way to increase or decrease our sum:

1. Moving the left pointer to the right **increases** the sum.
2. Moving the right pointer to the left **decreases** the sum.

If we start at the absolute extremes (`left = 0`, `right = numbers.length - 1`):

- If `sum > target`: The sum is too big; decrease it by moving the right pointer down (`p2--`).
- If `sum < target`: The sum is too small; increase it by moving the left pointer up (`p1++`).
- If `sum === target`: We found our pair!

---

## 💻 Implementation

:::info
This specific LeetCode problem asks for a 1-indexed array in the answer, so we add `1` to our indices before returning them.
:::

```javascript
class Solution {
    twoSum(numbers, target) {
        // A Hash Map/Set would use O(n) space.
        // Since the array is sorted and we need O(1) space, we use Two Pointers.
        let p1 = 0;
        let p2 = numbers.length - 1;

        while (p1 < p2) {
            let sum = numbers[p1] + numbers[p2];

            if (sum === target) {
                return [p1 + 1, p2 + 1]; // 1-based indexing
            }

            if (sum > target) {
                p2--; // Sum is too big, shrink it
            } else {
                p1++; // Sum is too small, grow it
            }
        }

        return [];
    }
}
```

---

## 🧮 Complexity Analysis

- **Time Complexity:** $O(n)$ &mdash; where $n$ is the length of the array. In the worst-case scenario, we evaluate each number exactly once.
- **Space Complexity:** $O(1)$ &mdash; since we use only two variables for our pointers, satisfying the strict memory constraint.

---
