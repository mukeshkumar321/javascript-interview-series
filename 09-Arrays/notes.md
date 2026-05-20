# Arrays in JavaScript

## Table of Contents

1. [Array Basics and Creation](#1-array-basics-and-creation)
2. [Mutating vs Non-Mutating Methods](#2-mutating-vs-non-mutating-methods)
3. [push, pop, shift, unshift](#3-push-pop-shift-unshift)
4. [splice vs slice](#4-splice-vs-slice)
5. [map](#5-map)
6. [filter](#6-filter)
7. [reduce](#7-reduce)
8. [forEach](#8-foreach)
9. [find and findIndex](#9-find-and-findindex)
10. [some and every](#10-some-and-every)
11. [flat and flatMap](#11-flat-and-flatmap)
12. [Array.from() and Array.of()](#12-arrayfrom-and-arrayof)
13. [indexOf vs includes](#13-indexof-vs-includes)
14. [sort() and Its Gotchas](#14-sort-and-its-gotchas)
15. [reverse()](#15-reverse)
16. [fill()](#16-fill)
17. [Spread with Arrays](#17-spread-with-arrays)
18. [Array Destructuring](#18-array-destructuring)
19. [Array-like Objects](#19-array-like-objects)
20. [Set vs Array](#20-set-vs-array)
21. [Chaining Array Methods](#21-chaining-array-methods)
22. [Performance: map vs forEach vs for loop](#22-performance-map-vs-foreach-vs-for-loop)
23. [Summary Table](#23-summary-table)

---

## 1. Array Basics and Creation

An array is an ordered, indexed collection of values. Arrays in JavaScript are dynamic, zero-indexed, and can hold values of mixed types.

```js
// Array literal (most common)
const nums = [1, 2, 3, 4, 5];

// Array constructor
const a1 = new Array(3);        // [empty × 3] — creates sparse array with length 3
const a2 = new Array(1, 2, 3);  // [1, 2, 3]

// Accessing elements
console.log(nums[0]);    // 1
console.log(nums.length); // 5
console.log(nums[nums.length - 1]); // 5 — last element

// Sparse arrays (holes)
const sparse = [1, , 3];
console.log(sparse.length); // 3
console.log(sparse[1]);     // undefined

// Arrays can hold mixed types
const mixed = [1, "two", true, null, { id: 1 }, [2, 3]];
console.log(mixed[4].id); // 1
console.log(mixed[5][0]); // 2
```

### Output

```js
1
5
5
3
undefined
1
2
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 2. Mutating vs Non-Mutating Methods

Understanding which methods mutate the original array and which return a new array is critical for predictable code.

| Method | Mutates? | Returns |
|---|---|---|
| `push` | Yes | New length |
| `pop` | Yes | Removed element |
| `shift` | Yes | Removed element |
| `unshift` | Yes | New length |
| `splice` | Yes | Array of removed elements |
| `sort` | Yes | Same array (sorted) |
| `reverse` | Yes | Same array (reversed) |
| `fill` | Yes | Same array |
| `copyWithin` | Yes | Same array |
| `map` | No | New array |
| `filter` | No | New array |
| `reduce` | No | Single accumulated value |
| `slice` | No | New shallow-copy array |
| `concat` | No | New array |
| `flat` | No | New array |
| `flatMap` | No | New array |
| `find` | No | Element or `undefined` |
| `findIndex` | No | Index or `-1` |
| `indexOf` | No | Index or `-1` |
| `includes` | No | Boolean |
| `some` | No | Boolean |
| `every` | No | Boolean |
| `forEach` | No | `undefined` |
| `join` | No | String |

```js
const arr = [3, 1, 2];

// Mutating
arr.sort();
console.log(arr); // [1, 2, 3] — original changed

// Non-mutating
const original = [3, 1, 2];
const sorted = [...original].sort();
console.log(original); // [3, 1, 2] — unchanged
console.log(sorted);   // [1, 2, 3]
```

### Output

```js
[1, 2, 3]
[3, 1, 2]
[1, 2, 3]
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 3. push, pop, shift, unshift

These four methods add or remove elements from the ends of an array, and they all mutate the original array.

```js
const arr = [2, 3, 4];

// push — adds to the end, returns new length
const newLen = arr.push(5, 6);
console.log(arr);    // [2, 3, 4, 5, 6]
console.log(newLen); // 5

// pop — removes from the end, returns removed element
const popped = arr.pop();
console.log(arr);    // [2, 3, 4, 5]
console.log(popped); // 6

// unshift — adds to the beginning, returns new length
const newLen2 = arr.unshift(0, 1);
console.log(arr);     // [0, 1, 2, 3, 4, 5]
console.log(newLen2); // 6

// shift — removes from the beginning, returns removed element
const shifted = arr.shift();
console.log(arr);     // [1, 2, 3, 4, 5]
console.log(shifted); // 0
```

### Output

```js
[2, 3, 4, 5, 6]
5
[2, 3, 4, 5]
6
[0, 1, 2, 3, 4, 5]
6
[1, 2, 3, 4, 5]
0
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 4. splice vs slice

`splice` mutates the array and can add/remove/replace elements. `slice` returns a new sub-array without modifying the original.

```js
// splice(start, deleteCount, ...itemsToAdd) — MUTATES
const fruits = ["apple", "banana", "cherry", "date"];

// Remove 2 elements starting at index 1
const removed = fruits.splice(1, 2);
console.log(fruits);  // ["apple", "date"]
console.log(removed); // ["banana", "cherry"]

// Insert without removing
fruits.splice(1, 0, "mango", "grape");
console.log(fruits); // ["apple", "mango", "grape", "date"]

// Replace elements
fruits.splice(2, 1, "kiwi");
console.log(fruits); // ["apple", "mango", "kiwi", "date"]

// slice(start, end) — does NOT mutate
const nums = [10, 20, 30, 40, 50];
const sub = nums.slice(1, 4); // index 1 up to (not including) 4
console.log(sub);  // [20, 30, 40]
console.log(nums); // [10, 20, 30, 40, 50] — unchanged

// Negative indices in slice
console.log(nums.slice(-2));    // [40, 50]
console.log(nums.slice(1, -1)); // [20, 30, 40]
```

### Output

```js
["apple", "date"]
["banana", "cherry"]
["apple", "mango", "grape", "date"]
["apple", "mango", "kiwi", "date"]
[20, 30, 40]
[10, 20, 30, 40, 50]
[40, 50]
[20, 30, 40]
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 5. map

`map` creates a **new array** by applying a callback to every element of the original array. It never mutates the original.

```js
const numbers = [1, 2, 3, 4, 5];

// Double each number
const doubled = numbers.map(n => n * 2);
console.log(doubled);  // [2, 4, 6, 8, 10]
console.log(numbers);  // [1, 2, 3, 4, 5] — unchanged

// Transform objects
const users = [
  { name: "Alice", age: 25 },
  { name: "Bob",   age: 30 }
];
const names = users.map(user => user.name);
console.log(names); // ["Alice", "Bob"]

// map callback receives (element, index, array)
const withIndex = ["a", "b", "c"].map((el, i) => `${i}:${el}`);
console.log(withIndex); // ["0:a", "1:b", "2:c"]

// map always returns an array of the same length as the input
const parsed = ["1", "2", "3"].map(Number);
console.log(parsed); // [1, 2, 3]
```

### Output

```js
[2, 4, 6, 8, 10]
[1, 2, 3, 4, 5]
["Alice", "Bob"]
["0:a", "1:b", "2:c"]
[1, 2, 3]
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 6. filter

`filter` creates a **new array** containing only elements for which the callback returns a truthy value.

```js
const numbers = [1, 2, 3, 4, 5, 6, 7, 8];

// Keep only even numbers
const evens = numbers.filter(n => n % 2 === 0);
console.log(evens); // [2, 4, 6, 8]

// Filter objects
const users = [
  { name: "Alice", active: true },
  { name: "Bob",   active: false },
  { name: "Carol", active: true }
];
const activeUsers = users.filter(user => user.active);
console.log(activeUsers.map(u => u.name)); // ["Alice", "Carol"]

// filter returns an empty array if nothing matches
const result = [1, 3, 5].filter(n => n % 2 === 0);
console.log(result); // []

// Using filter to remove falsy values
const withFalsy = [0, 1, "", "hello", null, undefined, false, true];
const truthy = withFalsy.filter(Boolean);
console.log(truthy); // [1, "hello", true]
```

### Output

```js
[2, 4, 6, 8]
["Alice", "Carol"]
[]
[1, "hello", true]
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 7. reduce

`reduce` iterates over an array and accumulates a single result using a callback `(accumulator, currentValue, index, array)` and an optional initial value.

```js
const numbers = [1, 2, 3, 4, 5];

// Sum
const sum = numbers.reduce((acc, n) => acc + n, 0);
console.log(sum); // 15

// Product
const product = numbers.reduce((acc, n) => acc * n, 1);
console.log(product); // 120

// Find max
const max = numbers.reduce((acc, n) => (n > acc ? n : acc), -Infinity);
console.log(max); // 5

// Flatten array with reduce
const nested = [[1, 2], [3, 4], [5]];
const flat = nested.reduce((acc, arr) => acc.concat(arr), []);
console.log(flat); // [1, 2, 3, 4, 5]

// Count occurrences
const fruits = ["apple", "banana", "apple", "cherry", "banana", "apple"];
const counts = fruits.reduce((acc, fruit) => {
  acc[fruit] = (acc[fruit] || 0) + 1;
  return acc;
}, {});
console.log(counts); // { apple: 3, banana: 2, cherry: 1 }

// Without initial value — first element becomes accumulator
const sum2 = [10, 20, 30].reduce((acc, n) => acc + n);
console.log(sum2); // 60
```

### Output

```js
15
120
5
[1, 2, 3, 4, 5]
{ apple: 3, banana: 2, cherry: 1 }
60
```

> If you call `reduce` on an empty array without an initial value, it throws a `TypeError`. Always provide an initial value when the array might be empty.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 8. forEach

`forEach` iterates over an array and calls a callback for each element. It **always returns `undefined`** and cannot be stopped with a `return` statement.

```js
const fruits = ["apple", "banana", "cherry"];

// Basic usage
fruits.forEach(fruit => console.log(fruit));
// apple, banana, cherry

// With index
fruits.forEach((fruit, index) => {
  console.log(`${index}: ${fruit}`);
});
// 0: apple  1: banana  2: cherry

// forEach returns undefined
const result = fruits.forEach(f => f.toUpperCase());
console.log(result); // undefined

// You cannot break out of forEach — use for...of or some() instead
const numbers = [1, 2, 3, 4, 5];
let found = -1;
numbers.forEach(n => {
  if (n === 3) found = n; // can't stop, all 5 iterations run
});
console.log(found); // 3
```

### Output

```js
apple
banana
cherry
0: apple
1: banana
2: cherry
undefined
3
```

> `forEach` is for side effects only. If you need a return value, use `map`. If you need to stop early, use `for...of` or `some()`.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 9. find and findIndex

`find` returns the **first element** that satisfies the callback. `findIndex` returns the **index** of the first matching element. Both return `undefined` / `-1` if nothing matches.

```js
const users = [
  { id: 1, name: "Alice" },
  { id: 2, name: "Bob" },
  { id: 3, name: "Carol" }
];

// find — returns the element itself
const user = users.find(u => u.id === 2);
console.log(user); // { id: 2, name: "Bob" }

// find returns undefined if not found
const missing = users.find(u => u.id === 99);
console.log(missing); // undefined

// findIndex — returns the index
const idx = users.findIndex(u => u.name === "Carol");
console.log(idx); // 2

// findIndex returns -1 if not found
const missingIdx = users.findIndex(u => u.name === "Dave");
console.log(missingIdx); // -1

// Stops at first match (unlike filter which checks all elements)
const nums = [5, 12, 8, 130, 44];
const firstAbove10 = nums.find(n => n > 10);
console.log(firstAbove10); // 12
```

### Output

```js
{ id: 2, name: "Bob" }
undefined
2
-1
12
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 10. some and every

`some` returns `true` if **at least one** element passes the test. `every` returns `true` if **all** elements pass the test. Both short-circuit.

```js
const numbers = [1, 2, 3, 4, 5];

// some — true if at least one matches
console.log(numbers.some(n => n > 4));  // true  (5 matches)
console.log(numbers.some(n => n > 10)); // false (none match)

// every — true only if all match
console.log(numbers.every(n => n > 0)); // true  (all > 0)
console.log(numbers.every(n => n > 2)); // false (1 and 2 fail)

// Short-circuit behavior
const results = [];
[1, 2, 3, 4, 5].some(n => {
  results.push(n);
  return n === 3; // stops when true
});
console.log(results); // [1, 2, 3]

// every short-circuits on first false
const results2 = [];
[1, 2, 3, 4, 5].every(n => {
  results2.push(n);
  return n < 3; // stops when false (at n=3)
});
console.log(results2); // [1, 2, 3]

// Edge cases: empty array
console.log([].some(n => n > 0));  // false
console.log([].every(n => n > 0)); // true (vacuously true)
```

### Output

```js
true
false
true
false
[1, 2, 3]
[1, 2, 3]
false
true
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 11. flat and flatMap

`flat` flattens a nested array to a specified depth. `flatMap` maps each element and then flattens one level deep — it is equivalent to `.map(...).flat(1)` but more efficient.

```js
// flat
const nested = [1, [2, 3], [4, [5, 6]]];

console.log(nested.flat());      // [1, 2, 3, 4, [5, 6]] — depth 1 (default)
console.log(nested.flat(2));     // [1, 2, 3, 4, 5, 6]   — depth 2
console.log(nested.flat(Infinity)); // [1, 2, 3, 4, 5, 6] — fully flat

const deepNested = [1, [2, [3, [4, [5]]]]];
console.log(deepNested.flat(Infinity)); // [1, 2, 3, 4, 5]

// flatMap — map + flat(1)
const sentences = ["Hello World", "Foo Bar"];
const words = sentences.flatMap(s => s.split(" "));
console.log(words); // ["Hello", "World", "Foo", "Bar"]

// flatMap can also filter by returning an empty array
const nums = [1, 2, 3, 4, 5];
const doubledEvens = nums.flatMap(n => (n % 2 === 0 ? [n * 2] : []));
console.log(doubledEvens); // [4, 8]
```

### Output

```js
[1, 2, 3, 4, [5, 6]]
[1, 2, 3, 4, 5, 6]
[1, 2, 3, 4, 5, 6]
[1, 2, 3, 4, 5]
["Hello", "World", "Foo", "Bar"]
[4, 8]
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 12. Array.from() and Array.of()

`Array.from` creates a new array from an array-like or iterable object. `Array.of` creates an array from a list of arguments — fixing the ambiguity of `new Array()`.

```js
// Array.from(iterable)
console.log(Array.from("hello"));    // ["h", "e", "l", "l", "o"]
console.log(Array.from(new Set([1, 2, 3]))); // [1, 2, 3]
console.log(Array.from(new Map([["a", 1], ["b", 2]]))); // [["a", 1], ["b", 2]]

// Array.from with a map function
const squares = Array.from({ length: 5 }, (_, i) => (i + 1) ** 2);
console.log(squares); // [1, 4, 9, 16, 25]

// Array.from array-like (arguments, NodeList)
function args() {
  return Array.from(arguments);
}
console.log(args(10, 20, 30)); // [10, 20, 30]

// Array.of — consistent behavior with any number of args
console.log(Array.of(3));       // [3] — NOT a sparse array of length 3
console.log(Array.of(1, 2, 3)); // [1, 2, 3]

// Contrast with new Array()
console.log(new Array(3));      // [empty × 3]
console.log(new Array(1, 2, 3)); // [1, 2, 3]
```

### Output

```js
["h", "e", "l", "l", "o"]
[1, 2, 3]
[["a", 1], ["b", 2]]
[1, 4, 9, 16, 25]
[10, 20, 30]
[3]
[1, 2, 3]
[empty × 3]
[1, 2, 3]
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 13. indexOf vs includes

`indexOf` returns the first index of a value (or `-1`). `includes` returns a boolean. The key difference is how they handle `NaN`.

```js
const arr = [1, 2, 3, NaN, "hello"];

// indexOf — uses strict equality (===)
console.log(arr.indexOf(2));      // 1
console.log(arr.indexOf(99));     // -1
console.log(arr.indexOf(NaN));    // -1  ← NaN !== NaN

// includes — uses SameValueZero (like === but treats NaN as NaN)
console.log(arr.includes(2));     // true
console.log(arr.includes(99));    // false
console.log(arr.includes(NaN));   // true  ← handles NaN correctly

// Second argument: starting index
const nums = [1, 2, 3, 2, 1];
console.log(nums.indexOf(2));      // 1
console.log(nums.indexOf(2, 2));   // 3 — starts searching from index 2

console.log(nums.includes(1, 3));  // true — starts from index 3, finds 1 at index 4
```

### Output

```js
1
-1
-1
true
false
true
1
3
true
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 14. sort() and Its Gotchas

`sort` mutates the array and sorts elements **in place**. By default, it converts elements to strings and sorts them **lexicographically (alphabetically)**, which produces unexpected results with numbers.

```js
// Default sort — lexicographic (string comparison)
const nums = [10, 9, 2, 1, 21];
nums.sort();
console.log(nums); // [1, 10, 2, 21, 9]  ← "10" < "2" lexicographically!

// Correct numeric sort — pass a comparator
const nums2 = [10, 9, 2, 1, 21];
nums2.sort((a, b) => a - b);
console.log(nums2); // [1, 2, 9, 10, 21]

// Descending
const nums3 = [10, 9, 2, 1, 21];
nums3.sort((a, b) => b - a);
console.log(nums3); // [21, 10, 9, 2, 1]

// Sorting strings correctly
const words = ["banana", "apple", "cherry", "date"];
words.sort();
console.log(words); // ["apple", "banana", "cherry", "date"]

// Case-insensitive sort
const mixed = ["Banana", "apple", "Cherry"];
mixed.sort((a, b) => a.toLowerCase().localeCompare(b.toLowerCase()));
console.log(mixed); // ["apple", "Banana", "Cherry"]

// Sort objects by property
const people = [
  { name: "Carol", age: 30 },
  { name: "Alice", age: 25 },
  { name: "Bob",   age: 28 }
];
people.sort((a, b) => a.age - b.age);
console.log(people.map(p => p.name)); // ["Alice", "Bob", "Carol"]
```

### Output

```js
[1, 10, 2, 21, 9]
[1, 2, 9, 10, 21]
[21, 10, 9, 2, 1]
["apple", "banana", "cherry", "date"]
["apple", "Banana", "Cherry"]
["Alice", "Bob", "Carol"]
```

**The classic interview gotcha:**

```js
console.log([10, 9, 2, 1].sort()); // [1, 10, 2, 9]
```

> Comparator rules: return negative to sort `a` before `b`, positive to sort `b` before `a`, zero to keep order.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 15. reverse()

`reverse` **mutates** the original array, reversing its elements in place, and returns a reference to the same array.

```js
const arr = [1, 2, 3, 4, 5];

const reversed = arr.reverse();
console.log(arr);             // [5, 4, 3, 2, 1] — original mutated
console.log(reversed === arr); // true — same reference

// To reverse without mutating, use spread + reverse
const original = [1, 2, 3, 4, 5];
const safeReversed = [...original].reverse();
console.log(original);     // [1, 2, 3, 4, 5] — unchanged
console.log(safeReversed); // [5, 4, 3, 2, 1]

// Reversing a string using array methods
const str = "hello";
const reversedStr = str.split("").reverse().join("");
console.log(reversedStr); // "olleh"
```

### Output

```js
[5, 4, 3, 2, 1]
true
[1, 2, 3, 4, 5]
[5, 4, 3, 2, 1]
olleh
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 16. fill()

`fill` mutates an array, replacing elements with a static value from a start index (inclusive) to an end index (exclusive).

```js
const arr = [1, 2, 3, 4, 5];

// fill(value)
arr.fill(0);
console.log(arr); // [0, 0, 0, 0, 0]

// fill(value, start)
const arr2 = [1, 2, 3, 4, 5];
arr2.fill(9, 2);
console.log(arr2); // [1, 2, 9, 9, 9]

// fill(value, start, end)
const arr3 = [1, 2, 3, 4, 5];
arr3.fill(7, 1, 4);
console.log(arr3); // [1, 7, 7, 7, 5]

// Create an array of zeros
const zeros = new Array(5).fill(0);
console.log(zeros); // [0, 0, 0, 0, 0]

// Trap: fill with a reference type shares the same reference
const matrix = new Array(3).fill([]);
matrix[0].push(1);
console.log(matrix); // [[1], [1], [1]] — all rows are the same reference!

// Fix: use Array.from
const safeMatrix = Array.from({ length: 3 }, () => []);
safeMatrix[0].push(1);
console.log(safeMatrix); // [[1], [], []]
```

### Output

```js
[0, 0, 0, 0, 0]
[1, 2, 9, 9, 9]
[1, 7, 7, 7, 5]
[0, 0, 0, 0, 0]
[[1], [1], [1]]
[[1], [], []]
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 17. Spread with Arrays

The spread operator (`...`) expands an iterable into individual elements. It is useful for copying, merging, and passing arrays as function arguments.

```js
// Shallow copy
const original = [1, 2, 3];
const copy = [...original];
copy.push(4);
console.log(original); // [1, 2, 3] — unchanged
console.log(copy);     // [1, 2, 3, 4]

// Merge arrays
const a = [1, 2];
const b = [3, 4];
const merged = [...a, ...b];
console.log(merged); // [1, 2, 3, 4]

// Spread into the middle
const withMiddle = [...a, 99, ...b];
console.log(withMiddle); // [1, 2, 99, 3, 4]

// Pass array as function arguments
function add(x, y, z) { return x + y + z; }
const nums = [1, 2, 3];
console.log(add(...nums)); // 6

// Convert Set or string to array
console.log([...new Set([1, 1, 2, 3, 3])]); // [1, 2, 3]
console.log([..."hello"]);                   // ["h", "e", "l", "l", "o"]

// Spread is shallow — nested objects are still references
const nested = [{ id: 1 }];
const shallowCopy = [...nested];
shallowCopy[0].id = 99;
console.log(nested[0].id); // 99 — shared reference
```

### Output

```js
[1, 2, 3]
[1, 2, 3, 4]
[1, 2, 3, 4]
[1, 2, 99, 3, 4]
6
[1, 2, 3]
["h", "e", "l", "l", "o"]
99
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 18. Array Destructuring

Array destructuring unpacks values from an array into distinct variables using positional matching.

```js
const rgb = [255, 128, 0];

// Basic destructuring
const [r, g, b] = rgb;
console.log(r, g, b); // 255 128 0

// Skip elements
const [, second, , fourth = 0] = [10, 20, 30];
console.log(second); // 20
console.log(fourth); // 0 — default value

// Rest element in destructuring
const [first, ...rest] = [1, 2, 3, 4, 5];
console.log(first); // 1
console.log(rest);  // [2, 3, 4, 5]

// Swap variables
let x = 1, y = 2;
[x, y] = [y, x];
console.log(x, y); // 2 1

// Destructure from function return
function getCoords() {
  return [40.7128, -74.0060];
}
const [lat, lon] = getCoords();
console.log(lat, lon); // 40.7128 -74.006

// Nested destructuring
const matrix = [[1, 2], [3, 4]];
const [[a, b], [c, d]] = matrix;
console.log(a, b, c, d); // 1 2 3 4
```

### Output

```js
255 128 0
20
0
1
[2, 3, 4, 5]
2 1
40.7128 -74.006
1 2 3 4
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 19. Array-like Objects

Array-like objects have a `length` property and numeric indices but do not inherit from `Array.prototype` — they lack array methods like `map`, `filter`, and `push`.

```js
// arguments is array-like
function demo() {
  console.log(arguments.length);     // 3
  console.log(Array.isArray(arguments)); // false
  // arguments.map(...) — TypeError

  // Convert to real array
  const arr = Array.from(arguments);
  console.log(Array.isArray(arr));   // true
  return arr.map(n => n * 2);
}
console.log(demo(1, 2, 3)); // [2, 4, 6]

// NodeList from DOM is array-like (in browser)
// const nodes = document.querySelectorAll("div");
// Array.from(nodes).forEach(n => console.log(n));

// Custom array-like object
const arrayLike = { 0: "a", 1: "b", 2: "c", length: 3 };
console.log(Array.from(arrayLike));               // ["a", "b", "c"]
console.log([].slice.call(arrayLike));             // ["a", "b", "c"]
console.log(Array.prototype.map.call(arrayLike, x => x.toUpperCase())); // ["A", "B", "C"]
```

### Output

```js
3
false
true
[2, 4, 6]
["a", "b", "c"]
["a", "b", "c"]
["A", "B", "C"]
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 20. Set vs Array

A `Set` stores **unique** values in insertion order. Use it when you need to eliminate duplicates or perform fast membership checks.

```js
// Set vs Array
const arr = [1, 2, 2, 3, 3, 3];
const set = new Set(arr);

console.log(arr); // [1, 2, 2, 3, 3, 3]
console.log(set); // Set { 1, 2, 3 }
console.log(set.size); // 3

// Convert Set back to Array
console.log([...set]);         // [1, 2, 3]
console.log(Array.from(set));  // [1, 2, 3]

// Deduplicate array
const unique = [...new Set([4, 5, 4, 6, 5, 7])];
console.log(unique); // [4, 5, 6, 7]

// Membership check: Set.has() is O(1) vs Array.includes() is O(n)
console.log(set.has(2));  // true
console.log(set.has(99)); // false

// Set operations
const setA = new Set([1, 2, 3, 4]);
const setB = new Set([3, 4, 5, 6]);

// Union
const union = new Set([...setA, ...setB]);
console.log([...union]); // [1, 2, 3, 4, 5, 6]

// Intersection
const intersection = new Set([...setA].filter(x => setB.has(x)));
console.log([...intersection]); // [3, 4]

// Difference
const difference = new Set([...setA].filter(x => !setB.has(x)));
console.log([...difference]); // [1, 2]
```

### Output

```js
[1, 2, 2, 3, 3, 3]
Set { 1, 2, 3 }
3
[1, 2, 3]
[1, 2, 3]
[4, 5, 6, 7]
true
false
[1, 2, 3, 4, 5, 6]
[3, 4]
[1, 2]
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 21. Chaining Array Methods

Array methods that return new arrays can be chained together to build readable data transformation pipelines.

```js
const employees = [
  { name: "Alice", dept: "Engineering", salary: 90000 },
  { name: "Bob",   dept: "Marketing",   salary: 60000 },
  { name: "Carol", dept: "Engineering", salary: 85000 },
  { name: "Dave",  dept: "Marketing",   salary: 70000 },
  { name: "Eve",   dept: "Engineering", salary: 95000 }
];

// Get names of Engineering employees with salary > 88000, sorted by name
const result = employees
  .filter(e => e.dept === "Engineering")
  .filter(e => e.salary > 88000)
  .map(e => e.name)
  .sort();

console.log(result); // ["Alice", "Eve"]

// Sum of salaries for Marketing department
const marketingPayroll = employees
  .filter(e => e.dept === "Marketing")
  .reduce((sum, e) => sum + e.salary, 0);

console.log(marketingPayroll); // 130000

// Note: mutating methods (sort, reverse) should come LAST in a chain
// and should be applied to a copy if the original matters:
const names = employees
  .map(e => e.name);
const sortedNames = [...names].sort(); // sort a copy
```

### Output

```js
["Alice", "Eve"]
130000
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 22. Performance: map vs forEach vs for loop

| Method | Returns | Can break early | Performance | Best for |
|---|---|---|---|---|
| `for` loop | — | Yes (`break`) | Fastest | Simple iteration, early exit |
| `for...of` | — | Yes (`break`) | Fast | Iterable traversal, early exit |
| `forEach` | `undefined` | No | Similar to `for` | Side effects only |
| `map` | New array | No | Slight overhead (new array) | Transforming values |
| `filter` | New array | No | Slight overhead | Selecting elements |
| `reduce` | Accumulated value | No | Similar to `map` | Aggregating values |

```js
const million = Array.from({ length: 1_000_000 }, (_, i) => i);

// for loop — fastest, can break early
let sumFor = 0;
for (let i = 0; i < million.length; i++) {
  sumFor += million[i];
}

// for...of — readable, can break early
let sumForOf = 0;
for (const n of million) {
  sumForOf += n;
}

// forEach — no early exit, slight overhead per call
let sumForEach = 0;
million.forEach(n => { sumForEach += n; });

// map — creates a second array in memory (avoid if you don't need the new array)
const doubled = million.map(n => n * 2); // allocates 1M element array

console.log(sumFor === sumForOf && sumForOf === sumForEach); // true
```

### Output

```js
true
```

> Use `map` when you need a transformed array. Use `forEach` for side effects. Use a `for` loop when you need to break early or maximize performance in tight loops.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 23. Summary Table

| Method | Mutates | Returns | Notes |
|---|---|---|---|
| `push(...items)` | Yes | New length | Adds to end |
| `pop()` | Yes | Removed element | Removes from end |
| `shift()` | Yes | Removed element | Removes from start |
| `unshift(...items)` | Yes | New length | Adds to start |
| `splice(i, n, ...items)` | Yes | Removed elements array | Add/remove/replace |
| `sort(compareFn)` | Yes | Same array | Lexicographic by default — always pass comparator for numbers |
| `reverse()` | Yes | Same array | In-place reversal |
| `fill(val, start, end)` | Yes | Same array | Fills with static value |
| `slice(start, end)` | No | New shallow copy | Negative indices work |
| `map(fn)` | No | New array | Same length as input |
| `filter(fn)` | No | New array | Only passing elements |
| `reduce(fn, init)` | No | Single value | Always provide initial value |
| `forEach(fn)` | No | `undefined` | Side effects; cannot break |
| `find(fn)` | No | First match or `undefined` | Stops at first match |
| `findIndex(fn)` | No | First index or `-1` | Stops at first match |
| `some(fn)` | No | Boolean | True if any match |
| `every(fn)` | No | Boolean | True if all match |
| `flat(depth)` | No | New array | Default depth = 1 |
| `flatMap(fn)` | No | New array | map + flat(1) |
| `indexOf(val)` | No | Index or `-1` | Fails on NaN |
| `includes(val)` | No | Boolean | Handles NaN correctly |
| `concat(...arrays)` | No | New array | Merges arrays |
| `join(sep)` | No | String | Joins with separator |
| `Array.from(iterable, mapFn)` | — | New array | Converts iterable/array-like |
| `Array.of(...values)` | — | New array | Consistent argument handling |
| `[...arr]` (spread) | No | New shallow copy | Works with any iterable |

---

## Final Notes

Arrays are the most used data structure in JavaScript. Mastering the distinction between mutating and non-mutating methods prevents subtle bugs — always reach for spread copy (`[...arr]`) or `slice()` before calling `sort()` or `reverse()` if you need to keep the original intact. The `sort()` default being lexicographic is one of the most common interview traps — always pass a comparator function when sorting numbers. Chain `map`, `filter`, and `reduce` to write expressive, readable data pipelines, but be mindful of performance in hot paths where multiple passes through large arrays matter.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
