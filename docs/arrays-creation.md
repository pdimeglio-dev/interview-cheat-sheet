To create an array of `n` elements in JavaScript, you can use `Array.from({ length: n })`. 

## Why does `Array.from({ length: 5 }, () => [])` create distinct arrays?

`Array.from({ length: 5 }, () => [])` is a common and effective way to create an array of a specific length, where each element is a unique, empty array. Here's why it works:

### Example

```javascript
const arr = Array.from({ length: 5 }, () => []);
arr[0].push(1);
console.log(arr); // [[1], [], [], [], []]
```

### Explanation

- **`{ length: 5 }` is "array-like":**  
  The first argument to `Array.from` is an object with a `length` property. JavaScript treats this as an array-like object and creates an array with 5 slots.

- **The mapping function (`() => []`):**  
  The second argument is a function that is called *once for each index*. Each call returns a new, empty array `[]`.

  This means:
  - `arr[0]`, `arr[1]`, ..., `arr[4]` are all distinct arrays.
  - Modifying one (like `arr[0].push(1)`) does *not* affect the others.

  **Contrast:**  
  If you used `.fill([])`, like:

  ```javascript
  const arr = Array(5).fill([]);
  arr[0].push(1);
  console.log(arr); // [[1], [1], [1], [1], [1]]
  ```

  All elements refer to the *same* array—changing one changes all!

- **Summary Table:**

  | Pattern                        | Description                                | Distinct Arrays? |
  |------------------------------- |--------------------------------------------|------------------|
  | `Array(5).fill([])`            | All share the same array reference         | ❌               |
  | `Array.from({ length: 5 }, () => [])` | Each slot gets a new array               | ✅               |

### Key Takeaway

Use `Array.from({ length: n }, () => [])` when you need an array of *unique* nested arrays.

For more, see [MDN Array.from documentation](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Array/from).


If you want each position to be initialized with a new array (not the same array object), **do not use** `fill([])`. Using `fill([])` fills all positions with a reference to the same array. So, modifying one element would modify them all, because they share the same object reference.

Instead, use `Array.from()` with a mapping function. For example:

```javascript
// Create an array of 5 arrays, each a unique empty array
const arr = Array.from({ length: 5 }, () => []);
arr[0].push(1);
console.log(arr); // [[1], [], [], [], []]
```

This way, each array inside the parent array is a distinct object.

You can also use `.fill(null).map(() => [])` to create an array of new, unique arrays. For example:

```javascript
// Create an array of 5 separate empty arrays
const arr = Array(5).fill(null).map(() => []);
arr[0].push(1);
console.log(arr); // [[1], [], [], [], []]
```

**Why does this work?**

When you use `.fill(null)` on an array, like `Array(5).fill(null)`, all positions are explicitly set to `null`. This matters because JavaScript arrays that are initialized with just `Array(n)` contain "holes" (uninitialized slots), and the `.map()` method skips these holes entirely—it only visits slots that have a defined value.

Setting each slot to `null` with `.fill(null)` removes the holes. Then, when you call `.map(() => [])`, your mapping function will be called on each element, letting you return a new, unique array at every index.

If you skip the `.fill(null)` step:

```javascript
Array(5).map(() => []) // [ <5 empty items> ]
```

`.map()` doesn't run for any index, so the resulting array is still filled with holes.

Using `.fill(null).map(() => [])` ensures that all slots are visited and replaced with new arrays.