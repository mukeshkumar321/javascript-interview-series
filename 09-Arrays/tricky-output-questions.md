# Arrays — Tricky Output Questions

## Table of Contents
1. [Mutation Questions](#1-mutation-questions)
2. [map/filter/reduce Questions](#2-mapfilterreduce-questions)
3. [sort() Questions](#3-sort-questions)
4. [splice vs slice Questions](#4-splice-vs-slice-questions)
5. [Spread and Destructuring Questions](#5-spread-and-destructuring-questions)
6. [Chaining Questions](#6-chaining-questions)
7. [Array-like Object Questions](#7-array-like-object-questions)
8. [Advanced Array Questions](#8-advanced-array-questions)

---

## 1. Mutation Questions

---

### Q1. What will be the output?

```js
const a = [1, 2, 3];
const b = a;
b.push(4);

console.log(a);
console.log(b);
console.log(a === b);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
[1, 2, 3, 4]
[1, 2, 3, 4]
true
```

### Explanation
`const b = a` does NOT create a copy — both `a` and `b` point to the same array in memory. Mutating `b` with `push` also mutates `a`. `a === b` is `true` because they are the same reference. To create an independent copy, use `[...a]`, `a.slice()`, or `Array.from(a)`.

</details>

---

### Q2. What will be the output?

```js
const arr = [3, 1, 4, 1, 5];
const sorted = arr.sort((a, b) => a - b);

console.log(arr);
console.log(sorted);
console.log(arr === sorted);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
[1, 1, 3, 4, 5]
[1, 1, 3, 4, 5]
true
```

### Explanation
`sort` mutates the original array and returns a reference to the **same** array (not a new one). Both `arr` and `sorted` point to the same sorted array. `arr === sorted` is `true`. To sort without mutating, use `[...arr].sort(...)`.

</details>

---

### Q3. What will be the output?

```js
const arr = [1, 2, 3, 4, 5];
arr.length = 3;
console.log(arr);

arr.length = 6;
console.log(arr);
console.log(arr[4]);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
[1, 2, 3]
[1, 2, 3, empty × 3]
undefined
```

### Explanation
Setting `arr.length = 3` truncates the array, removing elements at indices 3 and 4. Setting `arr.length = 6` extends it, but the new slots are empty (sparse). Accessing `arr[4]` returns `undefined` because no value was assigned to that index.

</details>

---

### Q4. What will be the output?

```js
const matrix = [[1, 2], [3, 4], [5, 6]];
const copy = matrix.slice();

copy[0].push(99);
copy.push([7, 8]);

console.log(matrix);
console.log(copy);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
[[1, 2, 99], [3, 4], [5, 6]]
[[1, 2, 99], [3, 4], [5, 6], [7, 8]]
```

### Explanation
`slice()` creates a **shallow copy** of `matrix`. The outer array is copied, but the inner arrays are still the same references. Pushing `99` to `copy[0]` also modifies `matrix[0]` because they share the same inner array reference. Pushing `[7, 8]` to `copy` only affects `copy` because that modifies the outer array (which was copied).

</details>

---

### Q5. What will be the output?

```js
const arr = new Array(3).fill({ value: 0 });
arr[0].value = 99;
console.log(arr);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
[{ value: 99 }, { value: 99 }, { value: 99 }]
```

### Explanation
`fill` with a reference type fills all slots with the **same object reference**. When `arr[0].value` is changed to `99`, all three elements reflect the change because they all point to the same object. Use `Array.from({ length: 3 }, () => ({ value: 0 }))` to create independent objects.

</details>

---

## 2. map/filter/reduce Questions

---

### Q1. What will be the output?

```js
const result = [1, 2, 3].map(parseInt);
console.log(result);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
[1, NaN, NaN]
```

### Explanation
`map` calls its callback with three arguments: `(element, index, array)`. `parseInt` accepts two arguments: `(string, radix)`. So the calls become:
- `parseInt(1, 0)` → radix 0 treated as 10 → `1`
- `parseInt(2, 1)` → radix 1 is invalid → `NaN`
- `parseInt(3, 2)` → `"3"` is not valid in base 2 → `NaN`

To fix: `[1, 2, 3].map(n => parseInt(n))` or `[1, 2, 3].map(Number)`.

</details>

---

### Q2. What will be the output?

```js
const nums = [1, 2, 3, 4, 5];
const result = nums.filter(n => n).map(n => n * 2);
console.log(result);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
[2, 4, 6, 8, 10]
```

### Explanation
`filter(n => n)` keeps all elements that are truthy. All numbers 1–5 are truthy, so nothing is filtered out. `map(n => n * 2)` then doubles each element. The result is `[2, 4, 6, 8, 10]`.

</details>

---

### Q3. What will be the output?

```js
const result = [1, 2, 3, 4, 5].reduce((acc, val) => {
  if (val % 2 === 0) acc.push(val * 10);
  return acc;
}, []);

console.log(result);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
[20, 40]
```

### Explanation
This uses `reduce` to both filter and transform in one pass. For each element, if it is even, it multiplies by 10 and pushes it into the accumulator array. Even numbers are `2` and `4`, giving `[20, 40]`. This is equivalent to `.filter(n => n % 2 === 0).map(n => n * 10)` but done in a single iteration.

</details>

---

### Q4. What will be the output?

```js
const result = [].reduce((acc, val) => acc + val);
console.log(result);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
TypeError: Reduce of empty array with no initial value
```

### Explanation
Calling `reduce` on an empty array without an initial value throws a `TypeError`. When no initial value is provided, `reduce` tries to use the first element as the accumulator — but with an empty array, there is no first element. Always provide an initial value when the array may be empty: `[].reduce((acc, val) => acc + val, 0)` returns `0`.

</details>

---

### Q5. What will be the output?

```js
const arr = [1, 2, 3];
const result = arr.map(function(x) {
  return this.multiplier * x;
}, { multiplier: 3 });

console.log(result);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
[3, 6, 9]
```

### Explanation
`map` accepts an optional second argument that becomes `this` inside the callback. When a regular function is used (not an arrow function), `this.multiplier` refers to the second argument object `{ multiplier: 3 }`. This feature does not work with arrow functions because they have lexical `this`.

</details>

---

## 3. sort() Questions

---

### Q1. What will be the output?

```js
console.log([10, 9, 2, 1].sort());
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
[1, 10, 2, 9]
```

### Explanation
Without a comparator, `sort` converts elements to strings and sorts them **lexicographically**. In string comparison: `"1" < "10" < "2" < "9"` because `"1"` comes before `"2"` as a character, and `"10"` starts with `"1"` so it sorts before `"2"`. This is the classic interview gotcha. Always use `.sort((a, b) => a - b)` for numeric sorting.

</details>

---

### Q2. What will be the output?

```js
const arr = ["banana", "Apple", "cherry", "apple"];
arr.sort();
console.log(arr);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
["Apple", "apple", "banana", "cherry"]
```

### Explanation
JavaScript's default string sort is case-sensitive and uses Unicode code points. Uppercase letters (A = 65) have lower code points than lowercase letters (a = 97), so all uppercase strings sort before all lowercase strings. `"Apple"` comes before `"apple"`, `"banana"`, and `"cherry"`. For case-insensitive sorting, use `.sort((a, b) => a.localeCompare(b))`.

</details>

---

### Q3. What will be the output?

```js
const items = [
  { name: "C", priority: 2 },
  { name: "A", priority: 1 },
  { name: "B", priority: 2 },
  { name: "D", priority: 1 }
];

items.sort((a, b) => a.priority - b.priority);
console.log(items.map(i => i.name));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
["A", "D", "C", "B"]
```

### Explanation
Items are sorted by `priority` in ascending order. Priority 1 items (`A`, `D`) come first, then priority 2 items (`C`, `B`). The relative order of equal-priority items depends on the sort algorithm's stability. Modern JavaScript engines (V8, SpiderMonkey) use **stable sort**, so the original order of equal elements is preserved: `A` before `D` (as they appeared), and `C` before `B`.

</details>

---

### Q4. What will be the output?

```js
const nums = [3, 1, 4, 1, 5];
const sorted1 = nums.sort((a, b) => a - b);
const sorted2 = nums.sort((a, b) => b - a);

console.log(sorted1);
console.log(sorted2);
console.log(sorted1 === sorted2);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
[5, 4, 3, 1, 1]
[5, 4, 3, 1, 1]
true
```

### Explanation
Both `sort` calls mutate the same array `nums` and return the same reference. `sorted1` and `sorted2` both point to `nums`. After the first sort, `nums` is `[1, 1, 3, 4, 5]`. After the second sort, `nums` is `[5, 4, 3, 1, 1]`. At the time of `console.log`, both variables hold the same (descending) sorted array, and they are the same reference (`=== true`).

</details>

---

## 4. splice vs slice Questions

---

### Q1. What will be the output?

```js
const arr = [1, 2, 3, 4, 5];
const removed = arr.splice(1, 2);

console.log(arr);
console.log(removed);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
[1, 4, 5]
[2, 3]
```

### Explanation
`splice(1, 2)` starts at index 1 and removes 2 elements (`2` and `3`). The original array is mutated to `[1, 4, 5]`. The removed elements are returned as a new array `[2, 3]`.

</details>

---

### Q2. What will be the output?

```js
const arr = [1, 2, 3, 4, 5];
const sliced = arr.slice(1, 3);

console.log(arr);
console.log(sliced);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
[1, 2, 3, 4, 5]
[2, 3]
```

### Explanation
`slice(1, 3)` returns a new array with elements from index 1 up to (but not including) index 3: `[2, 3]`. The original array is NOT modified. The end index is exclusive.

</details>

---

### Q3. What will be the output?

```js
const arr = ["a", "b", "c", "d", "e"];
arr.splice(2, 0, "X", "Y");
console.log(arr);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
["a", "b", "X", "Y", "c", "d", "e"]
```

### Explanation
`splice(2, 0, "X", "Y")` means: start at index 2, delete 0 elements, and insert `"X"` and `"Y"`. This effectively inserts elements without removing any. `"X"` and `"Y"` are inserted before `"c"`.

</details>

---

### Q4. What will be the output?

```js
const arr = [10, 20, 30, 40, 50];
console.log(arr.slice(-3));
console.log(arr.slice(-3, -1));
console.log(arr.slice(2, -1));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
[30, 40, 50]
[30, 40]
[30, 40]
```

### Explanation
Negative indices in `slice` count from the end of the array. The array has 5 elements (indices 0–4).
- `slice(-3)` → from index `5-3=2` to end → `[30, 40, 50]`
- `slice(-3, -1)` → from index 2 to index `5-1=4` (exclusive) → `[30, 40]`
- `slice(2, -1)` → from index 2 to index 4 (exclusive) → `[30, 40]`

</details>

---

## 5. Spread and Destructuring Questions

---

### Q1. What will be the output?

```js
const a = [1, 2, 3];
const b = [4, 5, 6];
const c = [...a, ...b];

a.push(99);

console.log(c);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
[1, 2, 3, 4, 5, 6]
```

### Explanation
`c` is created by spreading `a` and `b` — it copies the primitive values at the time of creation. Pushing `99` to `a` afterward does not affect `c` because primitives are copied by value, not by reference.

</details>

---

### Q2. What will be the output?

```js
const [x, y, ...rest] = [10, 20, 30, 40, 50];
console.log(x);
console.log(y);
console.log(rest);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
10
20
[30, 40, 50]
```

### Explanation
Array destructuring assigns values positionally. `x` gets `10`, `y` gets `20`, and `...rest` collects the remaining elements into a new array `[30, 40, 50]`. The rest element must be the last in a destructuring pattern.

</details>

---

### Q3. What will be the output?

```js
let a = 1;
let b = 2;
[a, b] = [b, a];

console.log(a);
console.log(b);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
2
1
```

### Explanation
This is the idiomatic way to swap two variables using array destructuring. The right-hand side `[b, a]` creates a temporary array `[2, 1]`. Destructuring then assigns `2` to `a` and `1` to `b` simultaneously, without needing a temporary variable.

</details>

---

### Q4. What will be the output?

```js
const [a = 10, b = 20, c = 30] = [1, undefined];
console.log(a, b, c);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
1 20 30
```

### Explanation
Destructuring default values activate when the value at that position is `undefined` or the position does not exist. `a = 1` (provided). `b = undefined` → default `20` activates. `c` has no corresponding value in the array → default `30` activates.

</details>

---

### Q5. What will be the output?

```js
const nested = [{ id: 1 }, { id: 2 }];
const copy = [...nested];

copy[0].id = 99;
copy.push({ id: 3 });

console.log(nested);
console.log(copy);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
[{ id: 99 }, { id: 2 }]
[{ id: 99 }, { id: 2 }, { id: 3 }]
```

### Explanation
Spread creates a **shallow copy** of the outer array. The objects inside are still the same references. Modifying `copy[0].id` also changes `nested[0].id`. However, `copy.push({ id: 3 })` only affects `copy`'s outer array (which was cloned), so `nested` still has 2 elements.

</details>

---

## 6. Chaining Questions

---

### Q1. What will be the output?

```js
const result = [1, 2, 3, 4, 5]
  .map(n => n * 2)
  .filter(n => n > 4)
  .reduce((sum, n) => sum + n, 0);

console.log(result);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
24
```

### Explanation
Step by step:
- `map(n => n * 2)` → `[2, 4, 6, 8, 10]`
- `filter(n => n > 4)` → `[6, 8, 10]`
- `reduce((sum, n) => sum + n, 0)` → `0 + 6 + 8 + 10 = 24`

</details>

---

### Q2. What will be the output?

```js
const words = ["hello world", "foo bar", "a b c"];
const result = words
  .flatMap(s => s.split(" "))
  .filter(w => w.length > 1)
  .map(w => w.toUpperCase());

console.log(result);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
["HELLO", "WORLD", "FOO", "BAR"]
```

### Explanation
- `flatMap(s => s.split(" "))` → `["hello", "world", "foo", "bar", "a", "b", "c"]`
- `filter(w => w.length > 1)` → `["hello", "world", "foo", "bar"]` (single-char words removed)
- `map(w => w.toUpperCase())` → `["HELLO", "WORLD", "FOO", "BAR"]`

</details>

---

### Q3. What will be the output?

```js
const arr = [1, 2, 3];

const result = arr
  .map(n => n * 2)
  .reverse()
  .join(" - ");

console.log(result);
console.log(arr);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
6 - 4 - 2
[1, 2, 3]
```

### Explanation
`map` returns a new array `[2, 4, 6]`. `reverse()` mutates **that new array** to `[6, 4, 2]` — the original `arr` is not affected because `map` returned a new array. `join(" - ")` returns the string `"6 - 4 - 2"`. `arr` remains `[1, 2, 3]`.

</details>

---

### Q4. What will be the output?

```js
const result = [1, [2, [3, [4]]]].flat(Infinity).filter(Boolean).reduce((a, b) => a + b);
console.log(result);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
10
```

### Explanation
- `flat(Infinity)` → `[1, 2, 3, 4]`
- `filter(Boolean)` → `[1, 2, 3, 4]` (all truthy, no change)
- `reduce((a, b) => a + b)` → `1 + 2 + 3 + 4 = 10`

</details>

---

## 7. Array-like Object Questions

---

### Q1. What will be the output?

```js
function test() {
  console.log(Array.isArray(arguments));
  const arr = Array.from(arguments);
  console.log(Array.isArray(arr));
  console.log(arr.map(n => n * 2));
}

test(1, 2, 3);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
false
true
[2, 4, 6]
```

### Explanation
`arguments` is array-like but not a real array — `Array.isArray(arguments)` returns `false`. `Array.from(arguments)` creates a real array, so `Array.isArray(arr)` is `true`. The real array has `map`, which doubles each element.

</details>

---

### Q2. What will be the output?

```js
const arrayLike = { 0: "a", 1: "b", 2: "c", length: 3 };

const arr1 = Array.from(arrayLike);
const arr2 = [...arrayLike];

console.log(arr1);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
["a", "b", "c"]
```

### Explanation
`Array.from` works with any array-like object (has numeric indices and `length`). It reads indices 0 through `length - 1` and creates a real array.

Note: `[...arrayLike]` would throw a `TypeError` because plain objects are not iterable by default (they don't have `[Symbol.iterator]`). Only `Array.from` handles plain array-like objects without an iterator.

</details>

---

### Q3. What will be the output?

```js
function sum() {
  return [].reduce.call(arguments, (acc, n) => acc + n, 0);
}

console.log(sum(1, 2, 3, 4));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
10
```

### Explanation
`[].reduce.call(arguments, ...)` borrows the `reduce` method from `Array.prototype` and applies it to the `arguments` object using `.call`. Since `arguments` has numeric indices and a `length`, `reduce` can iterate over it correctly. `1 + 2 + 3 + 4 = 10`.

</details>

---

## 8. Advanced Array Questions

---

### Q1. What will be the output?

```js
const arr = [1, 2, 3];
arr[10] = 99;

console.log(arr.length);
console.log(arr[5]);
console.log(arr.filter(Boolean));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
11
undefined
[1, 2, 3, 99]
```

### Explanation
Setting `arr[10] = 99` creates a **sparse array** with a `length` of `11` (index 10 + 1). Indices 3–9 are empty slots. `arr[5]` is `undefined`. `filter(Boolean)` skips sparse holes and falsy values, returning only `[1, 2, 3, 99]` — empty slots are treated as `undefined` (falsy) in iteration methods.

</details>

---

### Q2. What will be the output?

```js
const a = [1, 2, 3];
const b = [1, 2, 3];

console.log(a == b);
console.log(a === b);
console.log(a.toString() === b.toString());
console.log(JSON.stringify(a) === JSON.stringify(b));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
false
false
true
true
```

### Explanation
Arrays are objects. `a == b` and `a === b` compare references, not contents — `a` and `b` are different objects in memory, so both are `false`. `a.toString()` and `b.toString()` both produce `"1,2,3"` (same string), so string comparison is `true`. `JSON.stringify` also serializes the content to `"[1,2,3]"`, making that comparison `true`.

</details>

---

### Q3. What will be the output?

```js
const arr = [1, 2, 3, 4, 5];

arr.forEach((item, index) => {
  if (index === 2) arr.splice(index, 1);
});

console.log(arr);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
[1, 2, 4, 5]
```

### Explanation
`forEach` iterates from index 0 to the original length. When `index === 2`, `arr.splice(2, 1)` removes the element `3`. The array becomes `[1, 2, 4, 5]`. Subsequent iterations continue with the remaining indices, but since the splice shifted elements, `4` is now at index 2 and `5` at index 3. The forEach still ends after the original `length - 1 = 4` iterations, but the spliced gap means index 3 now holds `5` (formerly index 4). The end result is `[1, 2, 4, 5]`.

Note: Mutating an array while iterating with `forEach` can produce unexpected results in more complex cases.

</details>

---

### Q4. What will be the output?

```js
const set = new Set([1, 2, 3, 2, 1, 4]);
const arr = [...set];

console.log(arr);
console.log(arr.length);
console.log(set.size);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
[1, 2, 3, 4]
4
4
```

### Explanation
`Set` stores only unique values in insertion order. Duplicate values `2` and `1` are ignored. Spreading the Set into an array produces `[1, 2, 3, 4]` with 4 elements. `set.size` is also 4.

</details>

---

### Q5. What will be the output?

```js
const arr = [1, 2, 3];

const result = arr
  .map(n => {
    console.log("map:", n);
    return n * 2;
  })
  .filter(n => {
    console.log("filter:", n);
    return n > 2;
  });

console.log(result);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
map: 1
map: 2
map: 3
filter: 2
filter: 4
filter: 6
[4, 6]
```

### Explanation
`map` runs first on **all** elements, then `filter` runs on **all** elements of the mapped array. Methods in a chain are not interleaved — each method completes a full pass before the next begins. This is why `map` logs all three values before `filter` starts. The final result keeps only values greater than 2: `[4, 6]`.

</details>

---

### Q6. What will be the output?

```js
console.log(typeof []);
console.log([] instanceof Array);
console.log(Array.isArray([]));
console.log(Array.isArray({}));
console.log(Array.isArray(new Array(3)));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
object
true
true
false
true
```

### Explanation
`typeof []` returns `"object"` — not very useful for distinguishing arrays from objects. `instanceof Array` works but can fail across iframes (different `Array` constructors). `Array.isArray` is the most reliable check — it correctly identifies `[]` and `new Array(3)` as arrays, and `{}` as not an array.

</details>

---

## Final Tips

- **`sort()` gotcha**: Always pass a comparator `(a, b) => a - b` for numbers. The default lexicographic sort will sort `[10, 9, 2, 1]` as `[1, 10, 2, 9]`.
- **`map(parseInt)` trap**: `map` passes `(element, index, array)` to its callback. `parseInt` uses the second argument as a radix — use `map(Number)` or `map(n => parseInt(n))`.
- **Shallow copy danger**: `[...arr]`, `arr.slice()`, and `arr.concat()` create shallow copies. Nested objects still share references.
- **`reduce` on empty array**: Always provide an initial value to `reduce` when the array may be empty, or you'll get a `TypeError`.
- **`fill` reference trap**: `new Array(n).fill({})` fills with the same object reference. Use `Array.from({ length: n }, () => ({}))` instead.
- **`sort` mutates**: `sort` and `reverse` both mutate the original array AND return it. Spread a copy first if you need the original: `[...arr].sort(...)`.
- **`forEach` vs `map`**: `forEach` returns `undefined` and is for side effects. `map` returns a new array and is for transformations.
- **Array equality**: Arrays cannot be compared with `==` or `===` by content. Use `JSON.stringify` for simple cases or a deep-equal utility for complex ones.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
