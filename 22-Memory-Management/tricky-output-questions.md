# Memory Management — Tricky Output Questions

## Table of Contents
1. [Stack vs Heap Questions](#1-stack-vs-heap-questions)
2. [Garbage Collection Questions](#2-garbage-collection-questions)
3. [Memory Leak Questions](#3-memory-leak-questions)
4. [WeakMap / WeakRef Questions](#4-weakmap--weakref-questions)
5. [Closure Memory Questions](#5-closure-memory-questions)

---

## 1. Stack vs Heap Questions

---

### Q1. What will be the output?

```js
let a = { score: 10 };
let b = a;
b.score = 99;
console.log(a.score);
console.log(a === b);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
99
true
```

### Explanation
`a` and `b` are both reference types. When you write `let b = a`, you copy the **reference** (pointer), not the object. Both variables point to the same heap object. Mutating `b.score` changes the shared object, so `a.score` also reflects the change. Since they point to the same memory address, the strict equality check returns `true`.

</details>

---

### Q2. What will be the output?

```js
let x = 10;
let y = x;
y = 50;
console.log(x);
console.log(x === y);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
10
false
```

### Explanation
`x` is a primitive (`number`). When you write `let y = x`, the **value** `10` is copied directly on the stack. `y` is an independent variable. Reassigning `y = 50` only changes `y`'s own stack slot; `x` remains `10`. They now hold different values, so strict equality is `false`.

</details>

---

### Q3. What will be the output?

```js
function mutate(obj) {
  obj.value = 42;
}

function replace(obj) {
  obj = { value: 999 };
}

let original = { value: 1 };
mutate(original);
console.log(original.value);

replace(original);
console.log(original.value);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
42
42
```

### Explanation
`mutate` receives a copy of the reference. Since both the caller and the function point to the same heap object, `obj.value = 42` mutates the shared object, and `original.value` becomes `42`.

`replace` also receives a copy of the reference. Assigning `obj = { value: 999 }` rebinds the **local** parameter to a new object on the heap. The original reference `original` in the caller is unaffected. `original.value` remains `42`.

</details>

---

### Q4. What will be the output?

```js
const arr1 = [1, 2, 3];
const arr2 = arr1;
const arr3 = [...arr1];

arr2.push(4);
arr3.push(99);

console.log(arr1);
console.log(arr2);
console.log(arr3);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
[1, 2, 3, 4]
[1, 2, 3, 4]
[1, 2, 3, 99]
```

### Explanation
`arr2 = arr1` copies the reference — both point to the same array. `arr2.push(4)` mutates the shared array, so `arr1` also shows `4`.

`arr3 = [...arr1]` creates a **shallow copy** via the spread operator — a brand new array on the heap. `arr3.push(99)` only affects `arr3`. `arr1` is unaffected.

</details>

---

## 2. Garbage Collection Questions

---

### Q5. After this code runs, which objects are eligible for garbage collection?

```js
let a = { name: "A" };
let b = { name: "B" };

a.ref = b;
b.ref = a;

a = null;
b = null;

console.log("done");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
done
```

### Explanation
After `a = null` and `b = null`, there are no more root-level references to either object. Even though each object still references the other (a circular reference), **mark-and-sweep** correctly identifies that neither can be reached starting from any root. Both `{ name: "A" }` and `{ name: "B" }` are eligible for collection.

A pure **reference-counting** engine would fail here (each still has a count of 1 from the other), which is why modern engines use mark-and-sweep instead.

</details>

---

### Q6. What will be the output, and which value becomes eligible for GC?

```js
function getGreeting() {
  const message = "Hello, world!";
  const bigArray = new Array(100000).fill(0);
  return message;
}

const greeting = getGreeting();
console.log(greeting);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Hello, world!
```

### Explanation
`getGreeting` returns only the `message` string. After the function returns, `message` is returned by value (strings are primitives — its value is copied into `greeting`). The local `message` variable and the `bigArray` are no longer reachable from any root after the function completes. Both are eligible for GC. The `100000`-element array, which held significant memory, is freed on the next GC cycle.

</details>

---

### Q7. What will be the output? Will the inner function prevent GC of `data`?

```js
function outer() {
  const data = { payload: new Array(10000).fill("x") };

  return function inner() {
    return "I don't use data at all";
  };
}

const fn = outer();
console.log(fn());
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
I don't use data at all
```

### Explanation
Even though `inner` is a closure defined inside `outer`, **modern V8** is smart enough to recognize that `inner` does not actually reference `data`. V8 will not include `data` in the closure's scope. The `data` object is eligible for collection once `outer` returns — `fn` holding a reference to `inner` does not keep `data` alive.

Note: this is an implementation-level optimization. In some older engines or with more complex code, closures could retain the entire scope even if a variable was not used.

</details>

---

## 3. Memory Leak Questions

---

### Q8. Is there a memory leak? What is it?

```js
const handlers = [];

function setup(element) {
  const heavyData = new Array(50000).fill("record");

  function onClick() {
    console.log("Clicked, data length:", heavyData.length);
  }

  element.addEventListener("click", onClick);
  handlers.push(onClick); // stored globally
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
// No direct output — this is a leak analysis question
```

### Explanation
Yes, there are two interrelated leaks:

1. **`handlers` array never shrinks** — every call to `setup` pushes a new `onClick` into the global `handlers` array. Since `handlers` is never cleared, it grows indefinitely.
2. **`heavyData` is captured by `onClick`** — each `onClick` closure holds a reference to its own `50000`-element array via the closure scope. Because `handlers` keeps every `onClick` alive, every `heavyData` array is also kept alive.

Fix: call `element.removeEventListener("click", onClick)` and remove the entry from `handlers` when the element is destroyed.

</details>

---

### Q9. What will be logged, and is there a memory leak?

```js
let counter = 0;
const intervalId = setInterval(() => {
  counter++;
  if (counter === 3) {
    console.log("Done!");
    // forgot clearInterval
  }
}, 100);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Done!
```

### Explanation
`"Done!"` is logged when `counter` reaches `3`. However, the interval is **never cleared** — it continues firing indefinitely even after the desired work is complete. The callback (and everything it closes over, including `counter`) is kept alive by the timer subsystem.

Fix: add `clearInterval(intervalId)` inside the `if (counter === 3)` block.

</details>

---

### Q10. Will the removed element's memory be freed?

```js
let cachedEl = document.getElementById("header");
document.body.removeChild(document.getElementById("header"));

console.log(cachedEl.tagName); // still accessible?
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
HEADER
// (or whatever tagName the element has)
```

### Explanation
Removing an element from the DOM with `removeChild` only detaches it from the **document tree**. The JavaScript variable `cachedEl` still holds a strong reference to the element object on the heap. The element is now a **detached DOM node** — it cannot be GC'd as long as `cachedEl` is alive.

This is a classic memory leak in SPAs where component code caches DOM references but is never cleaned up. Fix: `cachedEl = null` after removal.

</details>

---

## 4. WeakMap / WeakRef Questions

---

### Q11. What will be the output?

```js
const wm = new WeakMap();

let key = {};
wm.set(key, "secret data");

console.log(wm.has(key)); // A
console.log(wm.get(key)); // B

key = null; // allow GC

// After GC runs, wm.has(originalKey) would return false
// but we no longer have a reference to the original key to check
console.log("key nulled"); // C
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
true
secret data
key nulled
```

### Explanation
Before `key = null`: `wm.has(key)` returns `true` (A) and `wm.get(key)` returns `"secret data"` (B) because the key object is still alive and in the WeakMap.

After `key = null`: the original object is eligible for GC. Once collected, the WeakMap entry is automatically removed. You cannot observe this directly because you no longer have the key to look up — that's the point of WeakMap. The string `"key nulled"` is printed (C) immediately; the GC happens asynchronously.

</details>

---

### Q12. What is the difference in behavior between this Map and WeakMap usage?

```js
const regularMap = new Map();
const weakMap = new WeakMap();

let obj = { id: 1 };
regularMap.set(obj, "regular");
weakMap.set(obj, "weak");

obj = null;

console.log(regularMap.size);   // A
// weakMap.size is always undefined — WeakMaps are not enumerable
console.log(weakMap.size);      // B
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
1
undefined
```

### Explanation
(A) `regularMap.size` is `1` even after `obj = null`. A regular `Map` holds a **strong reference** to its keys. The original `{ id: 1 }` object is not eligible for GC because `regularMap` still holds it. `size` remains `1`.

(B) `weakMap.size` is `undefined`. `WeakMap` intentionally does not expose a `size` property because its entries can disappear at any time when the GC collects their keys. Exposing `size` would give a non-deterministic value. After `obj = null`, the object is eligible for GC, and the WeakMap entry will be automatically removed.

</details>

---

### Q13. What will be the output?

```js
let obj = { name: "temp" };
const ref = new WeakRef(obj);

console.log(ref.deref()?.name); // A

obj = null;

// Immediately after null assignment — GC may NOT have run yet
console.log(ref.deref()?.name); // B — likely still "temp"

// After GC eventually runs:
// ref.deref() returns undefined
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
temp
temp
// (at some later, non-deterministic point: undefined from deref)
```

### Explanation
(A) `ref.deref()?.name` is `"temp"` because the object is still alive.

(B) Even after `obj = null`, the GC has not necessarily run yet. Within the same synchronous block, the object is **not guaranteed** to be collected immediately. So `ref.deref()` will still return the object in most cases. Only after a future GC cycle will `deref()` return `undefined`. This non-determinism is by design — you should always check `if (ref.deref())` before using it.

</details>

---

## 5. Closure Memory Questions

---

### Q14. What will be the output? Which value is retained in memory?

```js
function makeAdder(base) {
  const unused = new Array(100000).fill("waste");
  return function add(n) {
    return base + n;
  };
}

const add10 = makeAdder(10);
console.log(add10(5));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
15
```

### Explanation
The closure `add` captures the entire scope of `makeAdder`, which includes both `base` and `unused`. `base` is needed (`add10` uses it). `unused` is **not** referenced by `add` at all. Modern V8 optimizes this — it only keeps variables that are actually referenced in the closure, so `unused` is eligible for GC when `makeAdder` returns. The closure only retains `base = 10` in memory.

</details>

---

### Q15. How many objects are kept in memory after this code runs?

```js
function createCounter() {
  let count = 0;
  return {
    increment() { count++; },
    decrement() { count--; },
    value()     { return count; }
  };
}

const counter = createCounter();
counter.increment();
counter.increment();
counter.decrement();
console.log(counter.value());
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
1
```

### Explanation
All three methods (`increment`, `decrement`, `value`) are defined in the same scope and **share the same closure environment**. They all reference the same `count` variable. `count` starts at `0`, is incremented twice (`2`), decremented once (`1`), and `value()` returns `1`.

Memory: the returned object (with three method properties) is kept alive by `counter`. All three functions share a single closure environment object that contains `count`. Only **one** environment record is kept in memory, not three separate ones — closures created in the same scope share the same scope chain.

</details>

---

### Q16. What is the output? Does this code leak?

```js
const results = [];

for (let i = 0; i < 5; i++) {
  results.push(function() {
    return i;
  });
}

console.log(results[0]());
console.log(results[4]());
console.log(results.length);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
0
4
5
```

### Explanation
With `let`, each iteration of the loop creates a **new block scope** (and a new binding of `i`). Each pushed function closes over its own distinct `i`, capturing the value at that iteration. `results[0]()` returns `0`, `results[4]()` returns `4`.

Memory note: `results` is an array of 5 functions, each holding a closure over a small `i` value. As long as `results` is in scope, all 5 functions and their closure environments are kept alive. This is not a leak — it is expected behavior — but if `results` were unintentionally global and accumulated across many calls, it would become one.

</details>

---

## Final Tips

- **Primitives** are copied by value; **objects** are copied by reference. Mutating a parameter mutates the original object but reassigning the parameter does not.
- **Mark-and-sweep** handles circular references correctly. Reference counting does not.
- The three most common sources of real-world leaks: **forgotten event listeners**, **uncleared intervals**, and **growing caches** (plain `Map` or arrays never pruned).
- Use **`WeakMap`** when you want to attach data to an object's lifetime. Use **`WeakRef`** when you want an optional reference that does not prevent collection.
- Closures only retain variables they **actually reference** in modern V8 — but always verify with a heap snapshot when in doubt.
- Use the Chrome DevTools **Memory tab** → Heap Snapshot comparison to confirm that heap usage stabilizes over time.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
