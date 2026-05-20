# Utility Functions in JavaScript

## Table of Contents

1. [Debounce](#1-debounce)
2. [Throttle](#2-throttle)
3. [Debounce vs Throttle](#3-debounce-vs-throttle)
4. [Memoize](#4-memoize)
5. [Deep Clone](#5-deep-clone)
6. [Curry](#6-curry)
7. [Compose](#7-compose)
8. [Pipe](#8-pipe)
9. [once](#9-once)
10. [retry](#10-retry)
11. [chunk](#11-chunk)
12. [flatten](#12-flatten)
13. [groupBy](#13-groupby)
14. [unique](#14-unique)
15. [sleep](#15-sleep)
16. [EventEmitter](#16-eventemitter)
17. [Deep Equal](#17-deep-equal)
18. [pick and omit](#18-pick-and-omit)
19. [Summary](#19-summary)

---

## 1. Debounce

**Debounce** delays function execution until a specified time has passed since the last invocation. If the function is called again within the delay window, the timer resets.

**Real-world use case:** Search-input autocomplete — fire the API call only after the user stops typing for 300ms.

```js
function debounce(fn, delay) {
  let timer = null;

  return function (...args) {
    // Clear any previously scheduled call
    clearTimeout(timer);

    // Schedule a new call
    timer = setTimeout(() => {
      fn.apply(this, args);
      timer = null;
    }, delay);
  };
}

// Usage
function search(query) {
  console.log('Searching for:', query);
}

const debouncedSearch = debounce(search, 300);

debouncedSearch('J');        // timer starts
debouncedSearch('Ja');       // resets timer
debouncedSearch('Jav');      // resets timer
debouncedSearch('Java');     // resets timer — this one fires after 300ms
```

### Output

```js
// After 300ms of no further calls:
Searching for: Java
```

**Edge cases:**
- Only the **last** call within the delay window executes
- `this` context is preserved via `.apply(this, args)`
- Useful to add a `cancel()` method: expose it on the returned function so pending calls can be cancelled programmatically

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 2. Throttle

**Throttle** limits how often a function can fire. After each invocation, subsequent calls within the `limit` window are ignored.

**Real-world use case:** Window scroll or resize events — run the handler at most once every 200ms.

```js
function throttle(fn, limit) {
  let lastCallTime = 0;

  return function (...args) {
    const now = Date.now();

    if (now - lastCallTime >= limit) {
      lastCallTime = now;
      return fn.apply(this, args);
    }
    // Calls within the limit window are silently dropped
  };
}

// Usage
function handleScroll(event) {
  console.log('Scroll handled at:', Date.now());
}

const throttledScroll = throttle(handleScroll, 200);

// Simulating rapid calls
throttledScroll(); // fires immediately
throttledScroll(); // ignored — within 200ms
throttledScroll(); // ignored — within 200ms
setTimeout(throttledScroll, 250); // fires — 250ms have passed
```

### Output

```js
Scroll handled at: <timestamp1>
Scroll handled at: <timestamp2>   // ~250ms later
```

**Trailing-edge variant (fire on the trailing edge too):**

```js
function throttleWithTrailing(fn, limit) {
  let lastCallTime = 0;
  let trailingTimer = null;

  return function (...args) {
    const now = Date.now();
    const remaining = limit - (now - lastCallTime);

    clearTimeout(trailingTimer);

    if (remaining <= 0) {
      lastCallTime = now;
      fn.apply(this, args);
    } else {
      // Schedule the trailing call
      trailingTimer = setTimeout(() => {
        lastCallTime = Date.now();
        fn.apply(this, args);
      }, remaining);
    }
  };
}
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 3. Debounce vs Throttle

| Feature | Debounce | Throttle |
|---|---|---|
| When does it fire? | After the last call + delay | At most once per interval |
| Repeated calls | Resets the timer | Ignored until interval passes |
| First call | Delayed | Fires immediately (leading edge) |
| Best for | Search input, form validation | Scroll, resize, mouse move |
| Use case | "I stopped typing" | "I am scrolling right now" |
| Library | `_.debounce` (lodash) | `_.throttle` (lodash) |

```js
// Debounce: fires once AFTER rapid calls stop
window.addEventListener('input', debounce(fetchSuggestions, 300));

// Throttle: fires at most once every 100ms WHILE scrolling
window.addEventListener('scroll', throttle(updateProgressBar, 100));
```

### Output

```js
// Debounce: single call after typing stops
// Throttle: periodic calls during continuous event
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 4. Memoize

**Memoize** caches the results of expensive function calls. When the same arguments are seen again, the cached result is returned immediately without re-executing the function.

```js
function memoize(fn) {
  const cache = new Map();

  return function (...args) {
    // JSON.stringify creates a consistent cache key from any arguments
    const key = JSON.stringify(args);

    if (cache.has(key)) {
      console.log('Cache hit for:', key);
      return cache.get(key);
    }

    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

// Usage
function expensiveCalculation(n) {
  console.log('Computing for:', n);
  let result = 0;
  for (let i = 1; i <= n; i++) result += i;
  return result;
}

const memoCalc = memoize(expensiveCalculation);

console.log(memoCalc(1000));
console.log(memoCalc(1000)); // cache hit
console.log(memoCalc(500));
console.log(memoCalc(1000)); // cache hit again
```

### Output

```js
Computing for: 1000
500500
Cache hit for: [1000]
500500
Computing for: 500
125250
Cache hit for: [1000]
500500
```

**Edge cases:**
- `JSON.stringify` fails for functions, `undefined`, circular references, and `Symbol` keys — consider a custom serializer for those cases
- For recursive memoization, pass the memoized function to itself:
  ```js
  const memoFib = memoize(function fib(n) {
    if (n <= 1) return n;
    return memoFib(n - 1) + memoFib(n - 2);
  });
  ```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 5. Deep Clone

`JSON.stringify/parse` is a quick but lossy deep clone — it drops functions, `Date` objects become strings, `undefined` values disappear. A robust deep clone handles all types.

```js
function deepClone(value) {
  // Primitives and null
  if (value === null || typeof value !== 'object') return value;

  // Date
  if (value instanceof Date) return new Date(value.getTime());

  // RegExp
  if (value instanceof RegExp) return new RegExp(value.source, value.flags);

  // Array
  if (Array.isArray(value)) return value.map(item => deepClone(item));

  // Map
  if (value instanceof Map) {
    const clonedMap = new Map();
    value.forEach((v, k) => clonedMap.set(deepClone(k), deepClone(v)));
    return clonedMap;
  }

  // Set
  if (value instanceof Set) {
    const clonedSet = new Set();
    value.forEach(v => clonedSet.add(deepClone(v)));
    return clonedSet;
  }

  // Plain objects — preserve prototype
  const cloned = Object.create(Object.getPrototypeOf(value));
  for (const key of Object.keys(value)) {
    cloned[key] = deepClone(value[key]);
  }
  return cloned;
}

// Usage
const original = {
  name: 'Alice',
  scores: [95, 87, 92],
  meta: { active: true, createdAt: new Date('2024-01-01') },
};

const clone = deepClone(original);
clone.scores.push(100);
clone.meta.active = false;

console.log(original.scores);       // unchanged
console.log(original.meta.active);  // unchanged
console.log(clone.meta.createdAt instanceof Date);
```

### Output

```js
[95, 87, 92]
true
true
```

**Edge cases:**
- Does NOT handle circular references (would cause infinite recursion) — use a `WeakMap` to track visited objects for that
- Functions are passed by reference (cannot truly clone a function)
- `JSON.stringify` limitations: drops functions, converts `Date` to string, drops `undefined`, drops `Symbol` keys

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 6. Curry

**Currying** transforms a function that takes multiple arguments `f(a, b, c)` into a chain of functions each accepting one argument `f(a)(b)(c)`.

```js
function curry(fn) {
  return function curried(...args) {
    // If enough arguments are provided, call the original function
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    // Otherwise return a new function waiting for more arguments
    return function (...moreArgs) {
      return curried.apply(this, [...args, ...moreArgs]);
    };
  };
}

// Usage
function add(a, b, c) {
  return a + b + c;
}

const curriedAdd = curry(add);

console.log(curriedAdd(1)(2)(3));      // fully curried
console.log(curriedAdd(1, 2)(3));      // partial application
console.log(curriedAdd(1)(2, 3));      // partial application
console.log(curriedAdd(1, 2, 3));      // all at once

// Practical example
const multiply = curry((a, b) => a * b);
const double = multiply(2);
const triple = multiply(3);

console.log([1, 2, 3, 4].map(double));
console.log([1, 2, 3, 4].map(triple));
```

### Output

```js
6
6
6
6
[2, 4, 6, 8]
[3, 6, 9, 12]
```

**Key insight:** `fn.length` returns the number of **declared** parameters. Rest parameters (`...args`) have a length of 0 — curry does not work automatically with variadic functions.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 7. Compose

**Compose** combines multiple functions right-to-left: `compose(f, g, h)(x)` is equivalent to `f(g(h(x)))`. Each function's output is the next function's input.

```js
function compose(...fns) {
  if (fns.length === 0) return x => x;
  if (fns.length === 1) return fns[0];

  return function (x) {
    return fns.reduceRight((acc, fn) => fn(acc), x);
  };
}

// Usage
const double = x => x * 2;
const addOne = x => x + 1;
const square = x => x * x;

const transform = compose(double, addOne, square);
// Execution order (right to left): square → addOne → double
// square(3) = 9, addOne(9) = 10, double(10) = 20
console.log(transform(3));

// Composing string operations
const trim = s => s.trim();
const toLower = s => s.toLowerCase();
const exclaim = s => s + '!';

const shout = compose(exclaim, toLower, trim);
console.log(shout('  HELLO WORLD  '));
```

### Output

```js
20
hello world!
```

**Remember:** Compose is right-to-left. The rightmost function is applied first. This mirrors mathematical function composition: `(f ∘ g)(x) = f(g(x))`.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 8. Pipe

**Pipe** is the left-to-right version of compose: `pipe(f, g, h)(x)` is equivalent to `h(g(f(x)))`. More intuitive for reading as a data pipeline.

```js
function pipe(...fns) {
  if (fns.length === 0) return x => x;
  if (fns.length === 1) return fns[0];

  return function (x) {
    return fns.reduce((acc, fn) => fn(acc), x);
  };
}

// Usage
const double = x => x * 2;
const addOne = x => x + 1;
const square = x => x * x;

const pipeline = pipe(square, addOne, double);
// Execution order (left to right): square → addOne → double
// square(3) = 9, addOne(9) = 10, double(10) = 20
console.log(pipeline(3));

// Data transformation pipeline
const processUser = pipe(
  user => ({ ...user, name: user.name.trim() }),
  user => ({ ...user, email: user.email.toLowerCase() }),
  user => ({ ...user, role: user.role || 'viewer' }),
);

const raw = { name: '  Alice  ', email: 'ALICE@EXAMPLE.COM', role: '' };
console.log(processUser(raw));
```

### Output

```js
20
{ name: 'Alice', email: 'alice@example.com', role: 'viewer' }
```

| | Compose | Pipe |
|---|---|---|
| Direction | Right to left | Left to right |
| Reads like | Math notation | Unix pipe |
| First applied | Last function | First function |

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 9. once

**once** returns a wrapper that executes the original function **only on the first call**. All subsequent calls return the cached result of the first call.

```js
function once(fn) {
  let called = false;
  let result;

  return function (...args) {
    if (!called) {
      called = true;
      result = fn.apply(this, args);
    }
    return result;
  };
}

// Usage
function initializeApp() {
  console.log('App initialized!');
  return { initialized: true, timestamp: Date.now() };
}

const init = once(initializeApp);

const r1 = init();
const r2 = init(); // silently ignored
const r3 = init(); // silently ignored

console.log(r1 === r2); // same reference — cached
console.log(r3 === r1);

// Practical: Prevent multiple event listener setups
const setupListeners = once(() => {
  console.log('Listeners attached');
  document.addEventListener('click', () => {});
});
```

### Output

```js
App initialized!
true
true
```

**Edge cases:**
- The `result` is cached even if the first call returns `undefined` or `null`
- `called` flag ensures the function cannot be re-executed even if it threw an error on the first call (consider whether error-case re-try is desired)

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 10. retry

**retry** wraps an async operation and automatically re-attempts it up to `n` times if it fails.

```js
async function retry(fn, retries, delayMs = 0) {
  try {
    return await fn();
  } catch (err) {
    if (retries <= 0) {
      throw err; // no more retries — propagate error
    }
    if (delayMs > 0) {
      await new Promise(resolve => setTimeout(resolve, delayMs));
    }
    return retry(fn, retries - 1, delayMs);
  }
}

// Usage
let attemptCount = 0;

async function unreliableApi() {
  attemptCount++;
  console.log(`Attempt #${attemptCount}`);
  if (attemptCount < 3) {
    throw new Error('Network error');
  }
  return 'Success!';
}

retry(unreliableApi, 3, 100)
  .then(result => console.log(result))
  .catch(err => console.error('All retries failed:', err.message));
```

### Output

```js
Attempt #1
Attempt #2
Attempt #3
Success!
```

**Edge cases:**
- `retries = 0` means no retries (one attempt total); `retries = 3` means one initial + three retries (four total attempts)
- Exponential backoff: replace `delayMs` with `delayMs * Math.pow(2, totalRetries - retriesLeft)` for production use
- Non-retryable errors: check error type and rethrow immediately if it is not a transient failure

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 11. chunk

**chunk** splits an array into groups of a given size. The last group may be smaller if the array does not divide evenly.

```js
function chunk(arr, size) {
  if (!Array.isArray(arr)) throw new TypeError('First argument must be an array');
  if (size <= 0) throw new RangeError('Chunk size must be greater than 0');

  const result = [];
  for (let i = 0; i < arr.length; i += size) {
    result.push(arr.slice(i, i + size));
  }
  return result;
}

// Usage
console.log(chunk([1, 2, 3, 4, 5], 2));
console.log(chunk([1, 2, 3, 4, 5], 3));
console.log(chunk([1, 2, 3], 5));    // smaller than chunk size
console.log(chunk([], 2));            // empty array
```

### Output

```js
[[1, 2], [3, 4], [5]]
[[1, 2, 3], [4, 5]]
[[1, 2, 3]]
[]
```

**Edge cases:**
- `arr.slice(i, i + size)` naturally handles the last partial chunk
- `size > arr.length` wraps entire array in one chunk
- Original array is not mutated

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 12. flatten

**flatten** deeply flattens a nested array without using `Array.prototype.flat()`. Supports an optional depth limit.

```js
function flatten(arr, depth = Infinity) {
  const result = [];

  function helper(array, currentDepth) {
    for (const item of array) {
      if (Array.isArray(item) && currentDepth > 0) {
        helper(item, currentDepth - 1);
      } else {
        result.push(item);
      }
    }
  }

  helper(arr, depth);
  return result;
}

// Alternative: one-liner using reduce
function flattenReduce(arr) {
  return arr.reduce(
    (acc, val) =>
      Array.isArray(val) ? acc.concat(flattenReduce(val)) : acc.concat(val),
    []
  );
}

// Usage
const nested = [1, [2, [3, [4, [5]]]]];

console.log(flatten(nested, 1));
console.log(flatten(nested, 2));
console.log(flatten(nested));
console.log(flattenReduce(nested));
```

### Output

```js
[1, 2, [3, [4, [5]]]]
[1, 2, 3, [4, [5]]]
[1, 2, 3, 4, 5]
[1, 2, 3, 4, 5]
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 13. groupBy

**groupBy** groups array elements by the result of a key function (or property name), returning an object with keys mapping to arrays of matching elements.

```js
function groupBy(arr, keyFn) {
  return arr.reduce((acc, item) => {
    const key = typeof keyFn === 'function' ? keyFn(item) : item[keyFn];
    if (!acc[key]) acc[key] = [];
    acc[key].push(item);
    return acc;
  }, {});
}

// Usage
const people = [
  { name: 'Alice', dept: 'Engineering' },
  { name: 'Bob',   dept: 'Marketing' },
  { name: 'Carol', dept: 'Engineering' },
  { name: 'Dave',  dept: 'Marketing' },
  { name: 'Eve',   dept: 'Engineering' },
];

// Group by property name (string)
console.log(groupBy(people, 'dept'));

// Group by function
const words = ['apple', 'banana', 'avocado', 'blueberry', 'cherry'];
console.log(groupBy(words, w => w[0]));
```

### Output

```js
{
  Engineering: [
    { name: 'Alice', dept: 'Engineering' },
    { name: 'Carol', dept: 'Engineering' },
    { name: 'Eve',   dept: 'Engineering' }
  ],
  Marketing: [
    { name: 'Bob',  dept: 'Marketing' },
    { name: 'Dave', dept: 'Marketing' }
  ]
}
{
  a: ['apple', 'avocado'],
  b: ['banana', 'blueberry'],
  c: ['cherry']
}
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 14. unique

Removing duplicates from an array. Three different approaches with trade-offs.

```js
// Approach 1: Set (best performance — O(n), preserves insertion order)
function unique(arr) {
  return [...new Set(arr)];
}

// Approach 2: filter + indexOf (works for primitives, readable)
function uniqueV2(arr) {
  return arr.filter((item, index) => arr.indexOf(item) === index);
}

// Approach 3: reduce (flexible — can be adapted to use a custom comparator)
function uniqueV3(arr) {
  return arr.reduce((acc, item) => {
    if (!acc.includes(item)) acc.push(item);
    return acc;
  }, []);
}

// Approach 4: unique by object property
function uniqueBy(arr, keyFn) {
  const seen = new Map();
  return arr.filter(item => {
    const key = keyFn(item);
    if (seen.has(key)) return false;
    seen.set(key, true);
    return true;
  });
}

// Usage
const nums = [1, 2, 3, 2, 1, 4, 3, 5];
console.log(unique(nums));
console.log(uniqueV2(nums));
console.log(uniqueV3(nums));

const users = [
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' },
  { id: 1, name: 'Alice (duplicate)' },
];
console.log(uniqueBy(users, u => u.id));
```

### Output

```js
[1, 2, 3, 4, 5]
[1, 2, 3, 4, 5]
[1, 2, 3, 4, 5]
[{ id: 1, name: 'Alice' }, { id: 2, name: 'Bob' }]
```

| Approach | Time Complexity | Notes |
|---|---|---|
| `Set` | O(n) | Best; works for primitives |
| `filter + indexOf` | O(n²) | Readable; slow for large arrays |
| `reduce + includes` | O(n²) | Flexible; slow for large arrays |
| `uniqueBy` with Map | O(n) | For objects with a key property |

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 15. sleep

**sleep** pauses async execution for a given number of milliseconds by returning a Promise that resolves after a `setTimeout`.

```js
function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

// Usage
async function example() {
  console.log('Start');
  await sleep(1000);
  console.log('After 1 second');
  await sleep(500);
  console.log('After another 500ms');
}

example();

// Rate-limited loop
async function processWithDelay(items) {
  for (const item of items) {
    await processItem(item);
    await sleep(200); // wait 200ms between each
  }
}
```

### Output

```js
Start
// 1000ms pause
After 1 second
// 500ms pause
After another 500ms
```

**Key point:** `sleep` only works inside `async` functions when used with `await`. Simply calling `sleep(1000)` without `await` does nothing — it creates an unattended Promise.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 16. EventEmitter

A custom **EventEmitter** that supports subscribing (`on`), unsubscribing (`off`), emitting (`emit`), and one-time listeners (`once`).

```js
class EventEmitter {
  constructor() {
    this._events = Object.create(null); // no prototype — safer than {}
  }

  on(event, listener) {
    if (typeof listener !== 'function') {
      throw new TypeError('Listener must be a function');
    }
    if (!this._events[event]) {
      this._events[event] = [];
    }
    this._events[event].push(listener);
    return this; // enables chaining
  }

  off(event, listener) {
    if (!this._events[event]) return this;
    this._events[event] = this._events[event].filter(l => l !== listener);
    return this;
  }

  emit(event, ...args) {
    if (!this._events[event]) return false;
    // Snapshot the listeners array to allow safe removal during emit
    [...this._events[event]].forEach(listener => listener(...args));
    return true;
  }

  once(event, listener) {
    const wrapper = (...args) => {
      listener(...args);
      this.off(event, wrapper); // auto-remove after first call
    };
    this.on(event, wrapper);
    return this;
  }

  removeAllListeners(event) {
    if (event) {
      delete this._events[event];
    } else {
      this._events = Object.create(null);
    }
    return this;
  }
}

// Usage
const emitter = new EventEmitter();

function onData(data) {
  console.log('Data received:', data);
}

emitter.on('data', onData);
emitter.once('connect', () => console.log('Connected!'));

emitter.emit('connect'); // logs: Connected!
emitter.emit('connect'); // nothing — once removed after first call
emitter.emit('data', { value: 42 });
emitter.emit('data', { value: 99 });

emitter.off('data', onData);
emitter.emit('data', { value: 100 }); // nothing — removed
```

### Output

```js
Connected!
Data received: { value: 42 }
Data received: { value: 99 }
```

**Key design decisions:**
- `Object.create(null)` prevents accidental clashes with `Object.prototype` property names (e.g., `constructor`, `toString`)
- Snapshot `[...this._events[event]]` before iterating so that `off()` calls during `emit` do not cause missed or double-called listeners
- Return `this` from `on`, `off`, `once` for method chaining

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 17. Deep Equal

**deepEqual** checks whether two values are structurally equal — same shape, same values at every level.

```js
function deepEqual(a, b) {
  // Same reference or same primitive value
  if (a === b) return true;

  // One is null/undefined, the other is not
  if (a == null || b == null) return false;

  // Different types
  if (typeof a !== typeof b) return false;

  // Non-object primitives that didn't pass the === check above
  if (typeof a !== 'object') return false;

  // Array vs non-array
  if (Array.isArray(a) !== Array.isArray(b)) return false;

  // Date comparison
  if (a instanceof Date && b instanceof Date) {
    return a.getTime() === b.getTime();
  }

  // Compare keys
  const keysA = Object.keys(a);
  const keysB = Object.keys(b);

  if (keysA.length !== keysB.length) return false;

  for (const key of keysA) {
    if (!Object.prototype.hasOwnProperty.call(b, key)) return false;
    if (!deepEqual(a[key], b[key])) return false;
  }

  return true;
}

// Usage
console.log(deepEqual({ a: 1, b: { c: 2 } }, { a: 1, b: { c: 2 } }));
console.log(deepEqual({ a: 1, b: { c: 2 } }, { a: 1, b: { c: 3 } }));
console.log(deepEqual([1, [2, 3]], [1, [2, 3]]));
console.log(deepEqual([1, [2, 3]], [1, [2, 4]]));
console.log(deepEqual(new Date('2024-01-01'), new Date('2024-01-01')));
console.log(deepEqual(null, null));
console.log(deepEqual(null, undefined));
```

### Output

```js
true
false
true
false
true
true
false
```

**Edge cases:**
- `NaN`: `NaN === NaN` is `false`, so `deepEqual(NaN, NaN)` returns `false` with this implementation. Add `Number.isNaN(a) && Number.isNaN(b)` check if needed.
- Circular references will cause infinite recursion — use a `WeakSet` of visited pairs to handle them.
- `{ a: 1 }` vs `{ a: 1, b: undefined }` — `Object.keys` picks up the extra key in the second, returning `false`.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 18. pick and omit

**pick** creates a new object with only the specified keys. **omit** creates a new object excluding the specified keys. Both are inspired by Lodash.

```js
// pick: include only specified keys
function pick(obj, keys) {
  return keys.reduce((acc, key) => {
    if (Object.prototype.hasOwnProperty.call(obj, key)) {
      acc[key] = obj[key];
    }
    return acc;
  }, {});
}

// omit: exclude specified keys
function omit(obj, keys) {
  const keySet = new Set(keys);
  return Object.keys(obj).reduce((acc, key) => {
    if (!keySet.has(key)) {
      acc[key] = obj[key];
    }
    return acc;
  }, {});
}

// Usage
const user = {
  id: 1,
  name: 'Alice',
  email: 'alice@example.com',
  password: 'secret123',
  role: 'admin',
};

const publicProfile = pick(user, ['id', 'name', 'role']);
console.log(publicProfile);

const safeUser = omit(user, ['password']);
console.log(safeUser);
```

### Output

```js
{ id: 1, name: 'Alice', role: 'admin' }
{ id: 1, name: 'Alice', email: 'alice@example.com', role: 'admin' }
```

**Edge cases:**
- `pick` with a key that does not exist: the key is simply omitted from the result (no error)
- Using `Set` in `omit` improves lookup from O(n) to O(1) for large `keys` arrays
- Neither mutates the original object
- These are shallow copies — nested objects are still shared references

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 19. Summary

| Utility | Core Mechanism | Key Interview Points |
|---|---|---|
| `debounce` | `clearTimeout` + `setTimeout` | Fires after last call; resets on each call |
| `throttle` | `Date.now()` timestamp gate | Fires at most once per interval |
| `memoize` | `Map` as cache keyed by args | `JSON.stringify` for key; cache-busting |
| `deepClone` | Recursive type dispatch | Handles `Date`, `Array`, `Map`, `Set`, RegExp |
| `curry` | Accumulate args until `fn.length` met | Works via `fn.length`; breaks with rest params |
| `compose` | `reduceRight` | Right-to-left; mirrors math notation |
| `pipe` | `reduce` | Left-to-right; readable pipeline |
| `once` | `called` flag + cached result | Returns cached result on repeat calls |
| `retry` | Recursive async with try/catch | Distinguish total attempts from retries |
| `chunk` | `slice` in loop with step `size` | Last chunk may be smaller |
| `flatten` | Recursive + depth tracking | `Infinity` flattens completely |
| `groupBy` | `reduce` into object | Accepts property name or key function |
| `unique` | `Set`, `filter+indexOf`, or `reduce` | `Set` is O(n); `indexOf` is O(n²) |
| `sleep` | `Promise` + `setTimeout` | Must be `await`ed inside `async` |
| `EventEmitter` | Listener arrays per event name | Snapshot before emit; `once` uses wrapper |
| `deepEqual` | Recursive key-by-key comparison | `===` for primitives; handle `Date`, arrays |
| `pick` | `reduce` with key allowlist | Only own properties; shallow copy |
| `omit` | `reduce` with key denylist | `Set` for O(1) lookup |

---

## Final Notes

Utility functions are the Swiss Army knife of JavaScript interviews. They reveal whether a candidate can translate abstract requirements into clean, working code under pressure.

When implementing any utility function:

1. **Handle edge cases first** — null inputs, empty arrays, wrong types
2. **Use the right data structure** — `Map` over plain objects for dynamic keys, `Set` for deduplication
3. **Preserve semantics** — `this` binding matters in `debounce`/`throttle` when used as object methods
4. **Know the trade-offs** — `JSON.stringify` for memoize keys is simple but breaks for non-serializable values; `deepClone` vs `structuredClone` (native, available in modern environments)

The most frequently asked pairs in senior interviews are **debounce vs throttle**, **compose vs pipe**, and **deep clone vs JSON clone**.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
