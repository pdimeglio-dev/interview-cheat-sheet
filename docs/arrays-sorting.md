## Numeric Sorting

By default, `.sort()` in JavaScript converts numbers to strings and sorts them lexicographically (e.g., `"10"` comes before `"2"`). To sort numerically, provide a comparison callback:

```javascript
const nums = [10, 2, 5, 1];
nums.sort((a, b) => a - b);
console.log(nums); // [1, 2, 5, 10]
```

The callback `(a, b) => a - b` ensures numeric sorting.

:::note Explanation
The comparison function `(a, b) => a - b` tells `.sort()` how to order the elements:

- If the result is **negative** (`a - b < 0`), JavaScript places `a` **before** `b`.
- If the result is **zero** (`a - b === 0`), their order stays the same.
- If the result is **positive** (`a - b > 0`), JavaScript places `b` **before** `a`.

So if `a = 1` and `b = 5`, then `1 - 5 = -4`, which is negative—`1` goes before `5`.

#### Example:
```javascript
const nums = [10, 2, 5, 1];
nums.sort((a, b) => a - b); // sorts numbers in ascending order
console.log(nums); // [1, 2, 5, 10]
```

In other words, this callback ensures the array is sorted from smallest to largest number.
:::