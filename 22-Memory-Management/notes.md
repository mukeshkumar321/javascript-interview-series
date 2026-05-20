# Memory Management in JavaScript

## Table of Contents

1. [How JavaScript Manages Memory](#1-how-javascript-manages-memory)
2. [Memory Life Cycle](#2-memory-life-cycle)
3. [Stack vs Heap Memory](#3-stack-vs-heap-memory)
4. [Primitive Values on Stack](#4-primitive-values-on-stack)
5. [Reference Values on Heap](#5-reference-values-on-heap)
6. [Garbage Collection](#6-garbage-collection)
7. [Mark-and-Sweep Algorithm](#7-mark-and-sweep-algorithm)
8. [Reference Counting](#8-reference-counting)
9. [Memory Leaks — What They Are](#9-memory-leaks--what-they-are)
10. [Common Cause 1: Global Variables](#10-common-cause-1-global-variables)
11. [Common Cause 2: Closures Holding Large Data](#11-common-cause-2-closures-holding-large-data)
12. [Common Cause 3: Event Listeners Not Removed](#12-common-cause-3-event-listeners-not-removed)
13. [Common Cause 4: Timers Not Cleared](#13-common-cause-4-timers-not-cleared)
14. [Common Cause 5: Detached DOM Nodes](#14-common-cause-5-detached-dom-nodes)
15. [Common Cause 6: Caches That Grow Indefinitely](#15-common-cause-6-caches-that-grow-indefinitely)
16. [WeakMap and WeakSet](#16-weakmap-and-weakset)
17. [WeakRef and FinalizationRegistry](#17-weakref-and-finalizationregistry)
18. [Memory Profiling with Chrome DevTools](#18-memory-profiling-with-chrome-devtools)
19. [Summary](#19-summary)

---

## 1. How JavaScript Manages Memory

JavaScript uses **automatic memory management**. You do not need to manually allocate or free memory the way you do in C or C++. The JavaScript engine handles allocation when you create values and reclaims memory when values are no longer needed through a process called **garbage collection**.

This convenience removes an entire class of bugs (use-after-free, double-free) but introduces new ones: **memory leaks** caused by accidentally keeping references alive longer than needed.

```js
// No manual malloc/free — JS does it for you
let user = { name: "Alice" }; // memory allocated on the heap
user = null;                   // engine may now collect the old object
```

### Output

```js
// No visible output — this is about internal engine behavior
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 2. Memory Life Cycle

Every value in JavaScript goes through three phases:

1. **Allocate** — memory is reserved when a variable, object, or function is created.
2. **Use** — the allocated memory is read from or written to.
3. **Release** — the memory is returned to the pool when it is no longer reachable.

```js
// 1. Allocate
let arr = new Array(1000).fill(0); // ~8 KB allocated

// 2. Use
arr[0] = 42;
console.log(arr[0]); // 42

// 3. Release
arr = null; // the engine can now reclaim the 8 KB
```

### Output

```js
42
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 3. Stack vs Heap Memory

| Feature | Stack | Heap |
|---|---|---|
| What lives there | Primitives, execution frames | Objects, arrays, functions |
| Size | Fixed, small | Dynamic, large |
| Access speed | Very fast (LIFO pointer) | Slower (pointer lookup) |
| Management | Automatic (frame pop) | Garbage collector |

```js
function example() {
  let x = 10;          // x lives on the stack (primitive)
  let obj = { a: 1 };  // obj reference on stack, data on heap
}
// When example() returns, x and obj's reference are popped from the stack.
// The heap object { a: 1 } is eligible for GC if nothing else holds a reference.
```

### Output

```js
// No direct output — conceptual illustration
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 4. Primitive Values on Stack

Primitives (`number`, `string`, `boolean`, `null`, `undefined`, `symbol`, `bigint`) are stored **by value** directly on the call stack. Copying a primitive creates a completely independent copy.

```js
let a = 5;
let b = a; // b gets a copy of the value 5

b = 10;    // changing b does NOT affect a

console.log(a); // 5
console.log(b); // 10
```

### Output

```js
5
10
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 5. Reference Values on Heap

Objects, arrays, and functions are stored **by reference**. The variable holds a pointer (memory address) to the location on the heap. Copying a reference copies the pointer, not the data.

```js
let obj1 = { value: 1 };
let obj2 = obj1; // obj2 points to the same heap object

obj2.value = 99;

console.log(obj1.value); // 99 — both see the same object
console.log(obj1 === obj2); // true — same reference
```

### Output

```js
99
true
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 6. Garbage Collection

Garbage collection (GC) is the process by which the JavaScript engine automatically identifies and frees memory that is no longer **reachable** from the program's root references (global object, current call stack, active closures).

A value is considered **reachable** if it can be accessed by following references starting from a root. Anything unreachable is eligible for collection.

```js
function createUser() {
  let user = { name: "Bob", data: new Array(100000) };
  return user.name; // only the name string is returned
}
// After createUser() returns, the large `user` object is unreachable
// and will be collected by the GC.
let name = createUser();
console.log(name);
```

### Output

```js
Bob
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 7. Mark-and-Sweep Algorithm

The modern standard GC algorithm used in V8 and most JS engines. It works in two phases:

1. **Mark** — starting from roots, the engine traverses every reachable object and marks it as "alive."
2. **Sweep** — any object that was **not** marked is treated as garbage and its memory is reclaimed.

```js
let root = {
  child: {
    grandchild: { data: "important" }
  }
};

// root → child → grandchild  (all marked as reachable)

root.child = null;
// grandchild is now unreachable — no path from root reaches it
// Next GC cycle: grandchild and old child object are swept away
```

### Output

```js
// No direct output — illustrates GC traversal
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 8. Reference Counting

An older algorithm that tracks how many references point to each object. When the count drops to zero, the object is freed. Modern engines do **not** rely on this as the primary GC strategy because of a critical flaw: **circular references**.

```js
// Circular reference problem with reference counting
function createCycle() {
  let a = {};
  let b = {};
  a.ref = b; // a references b (b's count = 1)
  b.ref = a; // b references a (a's count = 1)
  // Both counts are 1, so neither is freed even after the function returns
}
createCycle();

// Mark-and-sweep handles this correctly:
// After createCycle() returns, neither a nor b is reachable from roots
// so both are correctly identified as garbage.
console.log("Cycle created and handled by mark-and-sweep");
```

### Output

```js
Cycle created and handled by mark-and-sweep
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 9. Memory Leaks — What They Are

A **memory leak** occurs when memory that is no longer needed is never released because the program still holds a reference to it — unintentionally keeping objects alive. Leaks cause the application to consume more and more memory over time, eventually leading to slowdowns or crashes.

```js
// Conceptual example of what a leak looks like in a heap snapshot
// Over time, "used heap" keeps growing instead of stabilizing

const leakyStore = [];

function addData() {
  leakyStore.push(new Array(10000).fill("leak")); // never removed
}

// If addData() is called repeatedly, leakyStore grows forever
// The arrays can never be GC'd because leakyStore holds references
```

### Output

```js
// No direct output — conceptual illustration
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 10. Common Cause 1: Global Variables

Variables accidentally declared without `let`/`const`/`var` (in non-strict mode) become properties of the global object and live for the entire page lifetime.

```js
function processData() {
  // Forgetting `let` makes this global — it never gets cleaned up
  result = { payload: new Array(50000).fill("data") };
}

processData();
// `result` is now window.result — it stays in memory forever

// Fix: always use strict mode and declare variables properly
"use strict";
function processDataSafe() {
  let result = { payload: new Array(50000).fill("data") };
  // result is local — GC can collect it after this function returns
}
```

### Output

```js
// No direct output — illustrates accidental global creation
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 11. Common Cause 2: Closures Holding Large Data

A closure captures variables from its outer scope. If a long-lived closure holds a reference to a large object, that object cannot be GC'd even if the closure itself only uses a small part of it.

```js
function setup() {
  const hugeData = new Array(1000000).fill("x"); // 1M strings

  // This closure only needs hugeData.length but captures the entire array
  return function getSize() {
    return hugeData.length; // hugeData is kept alive by this closure
  };
}

const getSize = setup();
console.log(getSize()); // 1000000 — hugeData stays in memory

// Fix: extract only what you need
function setupFixed() {
  const hugeData = new Array(1000000).fill("x");
  const size = hugeData.length; // capture only the number, not the array
  return function getSize() {
    return size;
  };
}
const getSizeFixed = setupFixed();
console.log(getSizeFixed()); // 1000000 — hugeData is now eligible for GC
```

### Output

```js
1000000
1000000
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 12. Common Cause 3: Event Listeners Not Removed

Every `addEventListener` call keeps a reference to the callback, and the callback's closure keeps references to any variables it captured. If the DOM element or the callback is never released, the memory it holds is leaked.

```js
function attachListener() {
  const heavyObject = { data: new Array(100000).fill(1) };

  document.getElementById("btn").addEventListener("click", function handler() {
    console.log(heavyObject.data.length); // heavyObject is captured
  });
  // If removeEventListener is never called, heavyObject lives as long as
  // the button element (potentially forever)
}

// Fix: remove the listener when it is no longer needed
function attachListenerFixed() {
  const heavyObject = { data: new Array(100000).fill(1) };

  function handler() {
    console.log(heavyObject.data.length);
    // Remove after first use if one-time only
    document.getElementById("btn").removeEventListener("click", handler);
  }

  document.getElementById("btn").addEventListener("click", handler);
}
```

### Output

```js
// Runs in browser — no Node.js output
// 100000  (printed on click)
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 13. Common Cause 4: Timers Not Cleared

`setInterval` keeps running and holds a reference to its callback (and everything the callback closes over) until `clearInterval` is called. Forgetting to clear an interval is a classic source of leaks in single-page applications.

```js
function startPolling() {
  const cache = { results: new Array(50000).fill("record") };

  const intervalId = setInterval(function() {
    // cache is referenced here — it stays alive as long as the interval runs
    console.log("Polling... cache size:", cache.results.length);
  }, 1000);

  // If this component unmounts but clearInterval is never called, `cache`
  // and the callback are never freed.
  return intervalId; // caller must call clearInterval(intervalId) to clean up
}

const id = startPolling();
// Later, when done:
clearInterval(id); // releases the callback and cache
```

### Output

```js
Polling... cache size: 50000
// (printed every second until clearInterval is called)
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 14. Common Cause 5: Detached DOM Nodes

A detached DOM node is one that has been removed from the document but still has a JavaScript reference pointing to it. The node (and its entire subtree) cannot be GC'd.

```js
// Leak: keeping a reference to a removed DOM node
let detachedDiv;

function createAndDetach() {
  const div = document.createElement("div");
  div.innerHTML = "<span>Lots of content</span>".repeat(1000);
  document.body.appendChild(div);

  detachedDiv = div; // global reference saved

  document.body.removeChild(div);
  // div is removed from the DOM, but detachedDiv still points to it
  // The entire subtree stays in memory
}

createAndDetach();

// Fix: nullify the reference when done
detachedDiv = null; // now the div and its subtree can be GC'd
```

### Output

```js
// Runs in browser — no direct console output
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 15. Common Cause 6: Caches That Grow Indefinitely

Using a plain object or `Map` as a cache without any eviction policy means entries accumulate forever.

```js
// Leak: unbounded cache
const cache = new Map();

function fetchUser(id) {
  if (cache.has(id)) return cache.get(id);
  const user = { id, data: new Array(10000).fill("info") };
  cache.set(id, user); // cached forever — cache grows without bound
  return user;
}

// Fix option 1: limit cache size (LRU pattern)
const MAX_SIZE = 100;
const lruCache = new Map();

function fetchUserLRU(id) {
  if (lruCache.has(id)) {
    const value = lruCache.get(id);
    lruCache.delete(id);
    lruCache.set(id, value); // move to end (most recently used)
    return value;
  }
  const user = { id, data: new Array(10000).fill("info") };
  if (lruCache.size >= MAX_SIZE) {
    // delete oldest entry (first key in insertion order)
    lruCache.delete(lruCache.keys().next().value);
  }
  lruCache.set(id, user);
  return user;
}

console.log(fetchUserLRU(1).id); // 1
```

### Output

```js
1
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 16. WeakMap and WeakSet

`WeakMap` and `WeakSet` hold their keys **weakly**, meaning the garbage collector can still collect the key object even if the WeakMap or WeakSet still references it. This makes them perfect for attaching metadata to objects without preventing their collection.

```js
const metadata = new WeakMap();

let user = { name: "Carol" };
metadata.set(user, { lastSeen: Date.now() });

console.log(metadata.has(user)); // true

user = null;
// The { name: "Carol" } object is now eligible for GC.
// When it is collected, the WeakMap entry is automatically removed.
// A regular Map would prevent this collection.

// WeakSet example: tracking visited objects
const visited = new WeakSet();

function process(obj) {
  if (visited.has(obj)) {
    console.log("Already processed");
    return;
  }
  visited.add(obj);
  console.log("Processing", obj.name);
}

let item = { name: "Task1" };
process(item); // Processing Task1
process(item); // Already processed
item = null;   // entry is automatically removed from WeakSet
```

### Output

```js
true
Processing Task1
Already processed
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 17. WeakRef and FinalizationRegistry

`WeakRef` (ES2021) lets you hold a **weak reference** to an object — the GC can still collect it. You call `.deref()` to get the object; it returns `undefined` if already collected.

`FinalizationRegistry` lets you register a **cleanup callback** that fires after an object is collected.

```js
let target = { name: "Ephemeral" };

// Create a weak reference — does NOT prevent GC
const weakRef = new WeakRef(target);

// Create a registry — callback fires when target is collected
const registry = new FinalizationRegistry((heldValue) => {
  console.log(`${heldValue} was garbage collected`);
});
registry.register(target, "Ephemeral object");

// Access the object while it still exists
const obj = weakRef.deref();
if (obj) {
  console.log(obj.name); // Ephemeral
}

// Allow GC to collect target
target = null;

// At some future GC cycle:
// weakRef.deref() returns undefined
// registry callback fires: "Ephemeral object was garbage collected"
```

### Output

```js
Ephemeral
// (later, after GC): Ephemeral object was garbage collected
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 18. Memory Profiling with Chrome DevTools

Chrome DevTools provides three main tools for memory investigation:

1. **Heap Snapshot** — captures all live objects and their sizes. Compare two snapshots to find what grew.
2. **Allocation instrumentation on timeline** — records allocations over time to find where objects are created.
3. **Allocation sampling** — lower overhead profiling showing which functions allocate the most memory.

```js
// Simulate a leak to practice with DevTools
const leaks = [];

document.getElementById("leak-btn")?.addEventListener("click", () => {
  // Each click allocates a large array and pushes it into a persistent array
  leaks.push(new Float64Array(100000));
  console.log("Heap growing:", leaks.length, "arrays held");
});

// Steps to profile:
// 1. Open DevTools → Memory tab
// 2. Take a heap snapshot (baseline)
// 3. Click the button several times
// 4. Take another heap snapshot
// 5. Use the "Comparison" view to see what objects grew
// 6. Look at retainers to find what is keeping them alive
```

### Output

```js
// In browser console after clicks:
Heap growing: 1 arrays held
Heap growing: 2 arrays held
Heap growing: 3 arrays held
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 19. Summary

| Topic | Key Point |
|---|---|
| Memory life cycle | Allocate → Use → Release |
| Stack | Primitives and call frames; fast, automatic cleanup |
| Heap | Objects and arrays; managed by GC |
| Primitives | Stored by value; copying creates independent copy |
| Reference types | Stored by reference; copying shares same heap object |
| Garbage collection | Automatic; based on reachability |
| Mark-and-sweep | Traces from roots, sweeps unmarked objects; handles cycles |
| Reference counting | Old algorithm; fails on circular references |
| Memory leak | Unintended reference keeping memory alive |
| Global variables | Accidental globals live for page lifetime |
| Closure leaks | Closure captures entire scope; extract only needed data |
| Event listeners | Always removeEventListener when element/component is gone |
| Timers | Always clearInterval / clearTimeout when done |
| Detached DOM nodes | Nullify JS references to removed DOM elements |
| Unbounded cache | Use LRU eviction or WeakMap to avoid infinite growth |
| WeakMap / WeakSet | Keys are weakly held; GC can collect them |
| WeakRef | Weak reference that returns undefined after GC |
| FinalizationRegistry | Callback fires after an object is collected |
| DevTools profiling | Heap Snapshot and Allocation Timeline to detect leaks |

---

## Final Notes

Memory management is an invisible but critical part of building production-quality JavaScript applications. Automatic garbage collection shields you from most low-level concerns, but it also creates a false sense of security. The most common production issues — sluggish SPAs, browser tab crashes, Node.js process OOM — often trace back to a single forgotten event listener, an uncleared interval, or an unbounded cache. Understanding how the mark-and-sweep algorithm works and what makes an object "reachable" gives you the mental model needed to write leak-free code. Reach for `WeakMap`, `WeakSet`, and `WeakRef` whenever you need to associate data with objects without taking ownership of their lifetime, and use Chrome DevTools' Memory panel regularly to validate that your application's heap stays stable over time.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
