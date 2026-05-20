# Polyfills in JavaScript

## Table of Contents

1. [What is a Polyfill?](#1-what-is-a-polyfill)
2. [Polyfill vs Transpilation vs Shim](#2-polyfill-vs-transpilation-vs-shim)
3. [Array.prototype.map Polyfill](#3-arrayprototypemap-polyfill)
4. [Array.prototype.filter Polyfill](#4-arrayprototypefilter-polyfill)
5. [Array.prototype.reduce Polyfill](#5-arrayprototypereduce-polyfill)
6. [Array.prototype.forEach Polyfill](#6-arrayprototypeforeach-polyfill)
7. [Array.prototype.find Polyfill](#7-arrayprototypefind-polyfill)
8. [Array.prototype.flat Polyfill](#8-arrayprototypeflat-polyfill)
9. [Function.prototype.bind Polyfill](#9-functionprototypebind-polyfill)
10. [Function.prototype.call Polyfill](#10-functionprototypecall-polyfill)
11. [Function.prototype.apply Polyfill](#11-functionprototypeapply-polyfill)
12. [Object.create Polyfill](#12-objectcreate-polyfill)
13. [Object.assign Polyfill](#13-objectassign-polyfill)
14. [Promise Polyfill](#14-promise-polyfill)
15. [Promise.all Polyfill](#15-promiseall-polyfill)
16. [Promise.race Polyfill](#16-promiserace-polyfill)
17. [Array.from Polyfill](#17-arrayfrom-polyfill)
18. [String.prototype.includes Polyfill](#18-stringprototypeincludes-polyfill)
19. [String.prototype.startsWith Polyfill](#19-stringprototypestartswith-polyfill)
20. [Number.isNaN Polyfill](#20-numberisnan-polyfill)
21. [typeof Check Pattern in Polyfills](#21-typeof-check-pattern-in-polyfills)
22. [Summary](#22-summary)

---

## 1. What is a Polyfill?

A **polyfill** is a piece of code (usually JavaScript) that implements a feature natively missing in a browser or environment. The term comes from a wall-filler product — it fills in the gaps.

When a modern JS API does not exist in an older browser, a polyfill provides that API so older environments behave like modern ones. Unlike a workaround, a polyfill is transparent: your application code keeps using the standard API name.

```js
// Without polyfill — crashes in old browsers
[1, 2, 3].flat();

// With polyfill in place, same code works everywhere
if (!Array.prototype.flat) {
  Array.prototype.flat = function(depth = 1) { /* custom impl */ };
}
[1, 2, 3].flat(); // now works
```

### Output

```js
[1, 2, 3]
```

**Key properties of a good polyfill:**
- Detects whether the native feature already exists before patching
- Matches the native API signature exactly (name, arguments, return value)
- Handles all documented edge cases

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 2. Polyfill vs Transpilation vs Shim

| Concept | What it does | Tool / Example |
|---|---|---|
| **Polyfill** | Adds a missing runtime API | `core-js`, hand-written |
| **Transpilation** | Converts new syntax to old syntax | Babel, TypeScript |
| **Shim** | Broader term — any compatibility layer, may change existing behavior | `es5-shim` |

```js
// Transpilation (Babel converts arrow function → regular function)
// Source: const add = (a, b) => a + b;
// Output: var add = function(a, b) { return a + b; };

// Polyfill (adds Promise to old browsers at runtime)
if (typeof Promise === 'undefined') {
  window.Promise = MyPromiseImplementation;
}
```

> **Interview tip:** Babel transpiles *syntax* (arrow functions, classes, destructuring). It does NOT polyfill runtime APIs like `Promise`, `fetch`, or `Array.prototype.flat`. You need a separate polyfill library (e.g., `core-js`) for those.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 3. Array.prototype.map Polyfill

`Array.prototype.map` creates a new array by calling a callback on every element.

```js
if (!Array.prototype.myMap) {
  Array.prototype.myMap = function (callback, thisArg) {
    if (typeof callback !== 'function') {
      throw new TypeError(callback + ' is not a function');
    }

    const result = [];
    for (let i = 0; i < this.length; i++) {
      // 'i in this' skips holes in sparse arrays
      if (i in this) {
        result[i] = callback.call(thisArg, this[i], i, this);
      }
    }
    return result;
  };
}

// Usage
const nums = [1, 2, 3];
console.log(nums.myMap(x => x * 2));
console.log(nums.myMap((x, i) => `${i}:${x}`));
```

### Output

```js
[2, 4, 6]
['0:1', '1:2', '2:3']
```

**Edge cases:**
- Sparse array: `[1,,3].myMap(x => x * 2)` → `[2, empty, 6]` (hole preserved)
- `thisArg` binds `this` inside the callback
- Returns a **new** array — original is unchanged

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 4. Array.prototype.filter Polyfill

`Array.prototype.filter` returns a new array containing only elements for which the callback returns truthy.

```js
if (!Array.prototype.myFilter) {
  Array.prototype.myFilter = function (callback, thisArg) {
    if (typeof callback !== 'function') {
      throw new TypeError(callback + ' is not a function');
    }

    const result = [];
    for (let i = 0; i < this.length; i++) {
      if (i in this && callback.call(thisArg, this[i], i, this)) {
        result.push(this[i]);
      }
    }
    return result;
  };
}

// Usage
const nums = [1, 2, 3, 4, 5, 6];
console.log(nums.myFilter(x => x % 2 === 0));
console.log(nums.myFilter(x => x > 3));
```

### Output

```js
[2, 4, 6]
[4, 5, 6]
```

**Edge cases:**
- Does NOT preserve sparse array holes — result is always a dense array
- Callback receives `(element, index, array)`

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 5. Array.prototype.reduce Polyfill

`Array.prototype.reduce` accumulates a single value by applying a callback to each element.

```js
if (!Array.prototype.myReduce) {
  Array.prototype.myReduce = function (callback, initialValue) {
    if (typeof callback !== 'function') {
      throw new TypeError(callback + ' is not a function');
    }

    let acc;
    let startIndex;

    if (arguments.length >= 2) {
      acc = initialValue;
      startIndex = 0;
    } else {
      if (this.length === 0) {
        throw new TypeError('Reduce of empty array with no initial value');
      }
      acc = this[0];
      startIndex = 1;
    }

    for (let i = startIndex; i < this.length; i++) {
      if (i in this) {
        acc = callback(acc, this[i], i, this);
      }
    }
    return acc;
  };
}

// Usage
const nums = [1, 2, 3, 4];
console.log(nums.myReduce((acc, cur) => acc + cur, 0));
console.log(nums.myReduce((acc, cur) => acc + cur));  // no initial value
console.log(nums.myReduce((acc, cur) => ({ ...acc, [cur]: cur * 2 }), {}));
```

### Output

```js
10
10
{ '1': 2, '2': 4, '3': 6, '4': 8 }
```

**Edge cases:**
- Calling on an empty array WITHOUT an `initialValue` throws `TypeError`
- With an `initialValue`, empty array returns `initialValue`
- `arguments.length >= 2` check ensures `initialValue = 0` or `false` is handled correctly

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 6. Array.prototype.forEach Polyfill

`Array.prototype.forEach` calls a callback for each element. It always returns `undefined`.

```js
if (!Array.prototype.myForEach) {
  Array.prototype.myForEach = function (callback, thisArg) {
    if (typeof callback !== 'function') {
      throw new TypeError(callback + ' is not a function');
    }

    for (let i = 0; i < this.length; i++) {
      if (i in this) {
        callback.call(thisArg, this[i], i, this);
      }
    }
    // Explicitly returns undefined
  };
}

// Usage
const result = [10, 20, 30].myForEach((val, i) => {
  console.log(`Index ${i}: ${val}`);
});
console.log(result);
```

### Output

```js
Index 0: 10
Index 1: 20
Index 2: 30
undefined
```

**Edge cases:**
- Always returns `undefined` — do not try to chain it
- Does not mutate the original array (unless the callback does so explicitly)

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 7. Array.prototype.find Polyfill

`Array.prototype.find` returns the **first** element for which the callback returns truthy, or `undefined` if none found.

```js
if (!Array.prototype.myFind) {
  Array.prototype.myFind = function (callback, thisArg) {
    if (typeof callback !== 'function') {
      throw new TypeError(callback + ' is not a function');
    }

    for (let i = 0; i < this.length; i++) {
      // Note: find checks ALL indices including holes (unlike filter)
      if (callback.call(thisArg, this[i], i, this)) {
        return this[i];
      }
    }
    return undefined;
  };
}

// Usage
const users = [
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' },
  { id: 3, name: 'Charlie' },
];
console.log([10, 20, 30, 40].myFind(x => x > 15));
console.log(users.myFind(u => u.id === 2));
console.log([1, 2, 3].myFind(x => x > 10));
```

### Output

```js
20
{ id: 2, name: 'Bob' }
undefined
```

**Edge cases:**
- Returns the **element** itself, not a boolean
- Returns `undefined` (not `-1`) when nothing is found
- Pair with `findIndex` which returns the index instead

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 8. Array.prototype.flat Polyfill

`Array.prototype.flat` flattens nested arrays up to a given depth (default 1).

```js
if (!Array.prototype.myFlat) {
  Array.prototype.myFlat = function (depth = 1) {
    const result = [];

    function flatten(arr, currentDepth) {
      for (let i = 0; i < arr.length; i++) {
        if (i in arr) {
          if (Array.isArray(arr[i]) && currentDepth > 0) {
            flatten(arr[i], currentDepth - 1);
          } else {
            result.push(arr[i]);
          }
        }
      }
    }

    flatten(this, depth);
    return result;
  };
}

// Usage
console.log([1, [2, [3, [4]]]].myFlat());
console.log([1, [2, [3, [4]]]].myFlat(2));
console.log([1, [2, [3, [4]]]].myFlat(Infinity));
```

### Output

```js
[1, 2, [3, [4]]]
[1, 2, 3, [4]]
[1, 2, 3, 4]
```

**Edge cases:**
- `depth = 0` returns a copy of the original array
- `Infinity` flattens completely regardless of nesting level
- Holes in sparse arrays are removed (native behavior)

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 9. Function.prototype.bind Polyfill

`Function.prototype.bind` returns a new function permanently bound to a given `this` and optionally pre-fills arguments (partial application).

```js
if (!Function.prototype.myBind) {
  Function.prototype.myBind = function (thisArg, ...outerArgs) {
    if (typeof this !== 'function') {
      throw new TypeError('myBind must be called on a function');
    }

    const originalFn = this;

    function BoundFunction(...innerArgs) {
      // When used as a constructor (new BoundFunction()),
      // 'this' inside the new call should be the new instance,
      // not the bound thisArg.
      const context = new.target ? this : thisArg;
      return originalFn.apply(context, [...outerArgs, ...innerArgs]);
    }

    // Preserve prototype so instanceof works correctly
    if (originalFn.prototype) {
      BoundFunction.prototype = Object.create(originalFn.prototype);
    }

    return BoundFunction;
  };
}

// Usage
function greet(greeting, punctuation) {
  return `${greeting}, ${this.name}${punctuation}`;
}
const obj = { name: 'Alice' };
const sayHi = greet.myBind(obj, 'Hello');
console.log(sayHi('!'));
console.log(sayHi('?'));
```

### Output

```js
Hello, Alice!
Hello, Alice?
```

**Edge cases:**
- Pre-filled arguments (partial application) are prepended to any later arguments
- When the bound function is used with `new`, `thisArg` is ignored — a new object is created instead
- Prototype chain must be preserved for correct `instanceof` checks

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 10. Function.prototype.call Polyfill

`Function.prototype.call` invokes a function with a specific `this` and individual arguments.

```js
if (!Function.prototype.myCall) {
  Function.prototype.myCall = function (thisArg, ...args) {
    if (typeof this !== 'function') {
      throw new TypeError('myCall must be called on a function');
    }

    // Coerce null/undefined to global object
    const context =
      thisArg !== null && thisArg !== undefined ? Object(thisArg) : globalThis;

    // Use a unique Symbol to avoid overwriting existing properties
    const sym = Symbol('__fn__');
    context[sym] = this;

    const result = context[sym](...args);
    delete context[sym];
    return result;
  };
}

// Usage
function introduce(language) {
  return `${this.name} writes ${language}`;
}
const dev = { name: 'Bob' };
console.log(introduce.myCall(dev, 'JavaScript'));
console.log(Math.max.myCall(null, 3, 1, 4, 1, 5));
```

### Output

```js
Bob writes JavaScript
5
```

**Edge cases:**
- `null` or `undefined` as `thisArg` defaults to the global object (in non-strict mode)
- Symbol key prevents accidental property name collisions on `thisArg`
- Must `delete` the temporary property after invocation

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 11. Function.prototype.apply Polyfill

`Function.prototype.apply` is like `call` but accepts an **array** of arguments instead of individual ones.

```js
if (!Function.prototype.myApply) {
  Function.prototype.myApply = function (thisArg, argsArray) {
    if (typeof this !== 'function') {
      throw new TypeError('myApply must be called on a function');
    }

    const context =
      thisArg !== null && thisArg !== undefined ? Object(thisArg) : globalThis;

    const args = Array.isArray(argsArray) ? argsArray : [];

    const sym = Symbol('__fn__');
    context[sym] = this;

    const result = context[sym](...args);
    delete context[sym];
    return result;
  };
}

// Usage
function sum(a, b, c) {
  return a + b + c;
}
console.log(sum.myApply(null, [1, 2, 3]));

const numbers = [5, 6, 2, 3, 7];
console.log(Math.max.myApply(null, numbers));
```

### Output

```js
6
7
```

**Edge cases:**
- `argsArray` may be `null` or `undefined` — treat as empty array
- The classic use case: `Math.max.apply(null, largeArray)` to find the max of an array

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 12. Object.create Polyfill

`Object.create(proto)` creates a new object whose prototype is set to `proto`.

```js
if (!Object.myCreate) {
  Object.myCreate = function (proto, propertiesObject) {
    if (typeof proto !== 'object' && typeof proto !== 'function') {
      throw new TypeError(
        'Object prototype may only be an Object or null: ' + proto
      );
    }

    // Use an empty constructor and set its prototype
    function F() {}
    F.prototype = proto;
    const obj = new F();

    // If property descriptors are provided, define them
    if (propertiesObject !== undefined) {
      Object.defineProperties(obj, propertiesObject);
    }

    return obj;
  };
}

// Usage
const animal = {
  speak() {
    return `${this.name} makes a sound.`;
  },
};

const dog = Object.myCreate(animal);
dog.name = 'Rex';
console.log(dog.speak());
console.log(Object.getPrototypeOf(dog) === animal);
```

### Output

```js
Rex makes a sound.
true
```

**Edge cases:**
- `Object.create(null)` creates an object with NO prototype — useful for pure maps
- The polyfill cannot fully replicate `Object.create(null)` using a constructor trick because `new F()` always links to `Object.prototype`. Use `Object.setPrototypeOf` in environments that support it for the null case.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 13. Object.assign Polyfill

`Object.assign(target, ...sources)` copies all **own enumerable** properties from source objects into the target.

```js
if (!Object.myAssign) {
  Object.myAssign = function (target, ...sources) {
    if (target === null || target === undefined) {
      throw new TypeError('Cannot convert undefined or null to object');
    }

    const to = Object(target);

    for (const source of sources) {
      if (source !== null && source !== undefined) {
        for (const key in source) {
          // Only own enumerable properties (in...for also iterates prototype chain)
          if (Object.prototype.hasOwnProperty.call(source, key)) {
            to[key] = source[key];
          }
        }
      }
    }
    return to;
  };
}

// Usage
const target = { a: 1, b: 2 };
const source1 = { b: 4, c: 5 };
const source2 = { c: 6, d: 7 };

const result = Object.myAssign(target, source1, source2);
console.log(result);
console.log(target === result); // modifies target in-place
```

### Output

```js
{ a: 1, b: 4, c: 6, d: 7 }
true
```

**Edge cases:**
- Does NOT deep clone — nested objects are copied by reference
- Later sources overwrite earlier sources for the same key
- Does NOT copy non-enumerable or inherited properties
- Symbol keys are copied by native `Object.assign` but not by this `for...in` based polyfill (acceptable simplification for interviews)

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 14. Promise Polyfill

A basic `Promise` implementation covering the three states: `pending`, `fulfilled`, and `rejected`, plus `.then()` and `.catch()`.

```js
class MyPromise {
  constructor(executor) {
    this.state = 'pending';
    this.value = undefined;
    this.reason = undefined;
    this._onFulfilledCallbacks = [];
    this._onRejectedCallbacks = [];

    const resolve = (value) => {
      if (this.state === 'pending') {
        this.state = 'fulfilled';
        this.value = value;
        this._onFulfilledCallbacks.forEach(fn => fn(value));
      }
    };

    const reject = (reason) => {
      if (this.state === 'pending') {
        this.state = 'rejected';
        this.reason = reason;
        this._onRejectedCallbacks.forEach(fn => fn(reason));
      }
    };

    try {
      executor(resolve, reject);
    } catch (err) {
      reject(err);
    }
  }

  then(onFulfilled, onRejected) {
    onFulfilled =
      typeof onFulfilled === 'function' ? onFulfilled : value => value;
    onRejected =
      typeof onRejected === 'function'
        ? onRejected
        : reason => { throw reason; };

    return new MyPromise((resolve, reject) => {
      const handleFulfilled = (value) => {
        setTimeout(() => {
          try { resolve(onFulfilled(value)); } catch (e) { reject(e); }
        }, 0);
      };
      const handleRejected = (reason) => {
        setTimeout(() => {
          try { resolve(onRejected(reason)); } catch (e) { reject(e); }
        }, 0);
      };

      if (this.state === 'fulfilled') {
        handleFulfilled(this.value);
      } else if (this.state === 'rejected') {
        handleRejected(this.reason);
      } else {
        this._onFulfilledCallbacks.push(handleFulfilled);
        this._onRejectedCallbacks.push(handleRejected);
      }
    });
  }

  catch(onRejected) {
    return this.then(null, onRejected);
  }

  static resolve(value) {
    if (value instanceof MyPromise) return value;
    return new MyPromise(resolve => resolve(value));
  }

  static reject(reason) {
    return new MyPromise((_, reject) => reject(reason));
  }
}

// Usage
new MyPromise((resolve, reject) => {
  setTimeout(() => resolve(42), 100);
})
  .then(val => {
    console.log('Resolved:', val);
    return val * 2;
  })
  .then(val => console.log('Chained:', val))
  .catch(err => console.error('Error:', err));
```

### Output

```js
Resolved: 42
Chained: 84
```

**Key points:**
- State transitions are one-way: `pending → fulfilled` or `pending → rejected`
- Callbacks are stored when still pending and flushed on resolution
- `setTimeout(..., 0)` makes `.then` callbacks asynchronous (mimics microtask queue)
- `.then` returns a **new** `MyPromise` enabling chaining

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 15. Promise.all Polyfill

`Promise.all` takes an array of promises and resolves when **all** resolve, or rejects as soon as **any** one rejects.

```js
MyPromise.all = function (promises) {
  return new MyPromise((resolve, reject) => {
    if (!Array.isArray(promises)) {
      return reject(new TypeError('Argument must be an array'));
    }

    const results = [];
    let completedCount = 0;
    const total = promises.length;

    if (total === 0) return resolve(results);

    promises.forEach((p, index) => {
      MyPromise.resolve(p).then(value => {
        results[index] = value;   // preserve order
        completedCount++;
        if (completedCount === total) {
          resolve(results);
        }
      }).catch(reject);           // first rejection short-circuits
    });
  });
};

// Usage
const p1 = MyPromise.resolve(1);
const p2 = new MyPromise(res => setTimeout(() => res(2), 50));
const p3 = MyPromise.resolve(3);

MyPromise.all([p1, p2, p3]).then(values => console.log(values));

// Rejection example
MyPromise.all([p1, MyPromise.reject('oops'), p3])
  .catch(err => console.log('Caught:', err));
```

### Output

```js
[1, 2, 3]
Caught: oops
```

**Edge cases:**
- Results are stored by **index** to preserve order regardless of resolution order
- An empty array resolves immediately with `[]`
- `MyPromise.resolve(p)` handles non-Promise values in the array (numbers, strings, etc.)

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 16. Promise.race Polyfill

`Promise.race` resolves or rejects as soon as the **first** promise in the array settles.

```js
MyPromise.race = function (promises) {
  return new MyPromise((resolve, reject) => {
    if (!Array.isArray(promises)) {
      return reject(new TypeError('Argument must be an array'));
    }

    // Each promise calls resolve/reject, but only the first one has effect
    // because the internal state of the returned promise is already set.
    promises.forEach(p => {
      MyPromise.resolve(p).then(resolve).catch(reject);
    });
  });
};

// Usage
const slow = new MyPromise(res => setTimeout(() => res('slow'), 200));
const fast = new MyPromise(res => setTimeout(() => res('fast'), 50));

MyPromise.race([slow, fast]).then(val => console.log('Winner:', val));
```

### Output

```js
Winner: fast
```

**Edge cases:**
- If the array is empty, the returned promise stays **pending forever** (same as native)
- A rejected promise can also win the race

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 17. Array.from Polyfill

`Array.from` converts an array-like or iterable object into a real array, with an optional mapping function.

```js
if (!Array.myFrom) {
  Array.myFrom = function (arrayLike, mapFn, thisArg) {
    if (arrayLike === null || arrayLike === undefined) {
      throw new TypeError(
        'Array.from requires an array-like or iterable object'
      );
    }

    const items = Object(arrayLike);
    const len = Math.abs(parseInt(items.length)) || 0;
    const result = new Array(len);

    for (let i = 0; i < len; i++) {
      const val = items[i];
      result[i] =
        typeof mapFn === 'function' ? mapFn.call(thisArg, val, i) : val;
    }
    return result;
  };
}

// Usage
console.log(Array.myFrom('hello'));
console.log(Array.myFrom({ length: 3, 0: 'a', 1: 'b', 2: 'c' }));
console.log(Array.myFrom([1, 2, 3], x => x * 10));
```

### Output

```js
['h', 'e', 'l', 'l', 'o']
['a', 'b', 'c']
[10, 20, 30]
```

**Edge cases:**
- Works with array-like objects (have `.length` and indexed properties)
- Does NOT handle true iterables (Sets, Maps, generators) — those require checking for `Symbol.iterator`
- The `mapFn` is applied during construction, not after

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 18. String.prototype.includes Polyfill

`String.prototype.includes` checks whether a string contains a given substring, starting from an optional position.

```js
if (!String.prototype.myIncludes) {
  String.prototype.myIncludes = function (searchString, position) {
    if (searchString instanceof RegExp) {
      throw new TypeError('searchString must not be a regular expression');
    }
    // Default position is 0; negative values are treated as 0
    const start = Math.max(0, position | 0);
    return this.indexOf(String(searchString), start) !== -1;
  };
}

// Usage
const sentence = 'The quick brown fox';
console.log(sentence.myIncludes('quick'));
console.log(sentence.myIncludes('slow'));
console.log(sentence.myIncludes('fox', 10));
console.log(sentence.myIncludes('fox', 17));
```

### Output

```js
true
false
true
false
```

**Edge cases:**
- Throws `TypeError` if a RegExp is passed (matches native behavior)
- Case-sensitive by default
- `position` defaults to 0; negative values are treated as 0

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 19. String.prototype.startsWith Polyfill

`String.prototype.startsWith` checks whether a string begins with a given substring, starting from an optional position.

```js
if (!String.prototype.myStartsWith) {
  String.prototype.myStartsWith = function (searchString, position) {
    if (searchString instanceof RegExp) {
      throw new TypeError('searchString must not be a regular expression');
    }
    const startPos = Math.max(0, position | 0);
    return this.indexOf(String(searchString), startPos) === startPos;
  };
}

// Usage
const url = 'https://example.com';
console.log(url.myStartsWith('https'));
console.log(url.myStartsWith('http://'));
console.log(url.myStartsWith('example', 8));
console.log('hello world'.myStartsWith('world', 6));
```

### Output

```js
true
false
true
true
```

**Edge cases:**
- The key insight: `indexOf(search, pos) === pos` means the match starts exactly at `pos`
- An empty string `''` always returns `true` (matches native)
- Throws `TypeError` for RegExp input

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 20. Number.isNaN Polyfill

`Number.isNaN` is the **strict** version of the global `isNaN`. It returns `true` only for the actual `NaN` value, without type coercion.

```js
if (!Number.myIsNaN) {
  Number.myIsNaN = function (value) {
    // NaN is the only value in JS that is not equal to itself
    return typeof value === 'number' && value !== value;
  };
}

// Usage
console.log(Number.myIsNaN(NaN));         // true
console.log(Number.myIsNaN(0 / 0));       // true
console.log(Number.myIsNaN('NaN'));        // false — string, not NaN
console.log(Number.myIsNaN(undefined));   // false
console.log(Number.myIsNaN(null));         // false

// Contrast with global isNaN (performs coercion)
console.log(isNaN('NaN'));                 // true — coerces 'NaN' to NaN first
console.log(Number.myIsNaN('NaN'));        // false — no coercion
```

### Output

```js
true
true
false
false
false
true
false
```

**Key insight:** The `value !== value` trick works because `NaN` is the only JavaScript value that is not equal to itself (IEEE 754 standard). The `typeof` check ensures we only return `true` for the number type.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 21. typeof Check Pattern in Polyfills

There are three common guard patterns used when writing polyfills. Each has trade-offs.

```js
// Pattern 1: Truthiness check (simplest, most common)
if (!Array.prototype.flat) {
  Array.prototype.flat = function() { /* ... */ };
}

// Pattern 2: typeof check (safer — avoids issues with Object.prototype.flat = 0)
if (typeof Array.prototype.flat === 'undefined') {
  Array.prototype.flat = function() { /* ... */ };
}

// Pattern 3: Feature detection via try/catch (for complex features)
// Used when the method may exist but be buggy in certain environments
(function() {
  try {
    const test = [1, [2]].flat();
    if (!Array.isArray(test)) throw new Error();
  } catch (e) {
    Array.prototype.flat = function() { /* ... */ };
  }
})();

// Pattern 4: Assign only if the property is not own (for prototype augmentation)
if (!Object.prototype.hasOwnProperty.call(Array.prototype, 'flat')) {
  Array.prototype.flat = function() { /* ... */ };
}
```

### Output

```js
// No output — these are guard checks only
// Pattern 1 is sufficient for 99% of interview scenarios
```

**Which to use?**

| Pattern | Use when |
|---|---|
| `!method` | Quick polyfill, method should be truthy when it exists |
| `typeof method === 'undefined'` | Safer; avoids falsy-value pitfalls |
| `try/catch` feature detection | Method exists but has known bugs in some environments |
| `hasOwnProperty` | Ensuring the property is truly own, not inherited |

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 22. Summary

| Polyfill | Key Insight |
|---|---|
| `Array.prototype.map` | Use `i in this` to skip sparse holes; returns new array |
| `Array.prototype.filter` | Same sparse check; result is always dense |
| `Array.prototype.reduce` | Handle no-initial-value case; throw on empty array |
| `Array.prototype.forEach` | Always returns `undefined` |
| `Array.prototype.find` | Returns element or `undefined`; does NOT skip holes |
| `Array.prototype.flat` | Recursive helper; `Infinity` depth flattens completely |
| `Function.prototype.bind` | Handle `new.target` for constructor calls; preserve prototype |
| `Function.prototype.call` | Use `Symbol` key + `delete` pattern to set `this` |
| `Function.prototype.apply` | Same as `call` but accepts args as array |
| `Object.create` | Empty constructor trick to set `[[Prototype]]` |
| `Object.assign` | Shallow copy; `hasOwnProperty` guard; modifies target |
| `Promise` | Track state, queue callbacks, return new promise from `.then` |
| `Promise.all` | Store by index for order; first rejection wins |
| `Promise.race` | First settler wins; works via single-transition promise state |
| `Array.from` | Use `.length` property to create array from array-like |
| `String.prototype.includes` | Delegate to `indexOf !== -1`; reject RegExp input |
| `String.prototype.startsWith` | `indexOf(str, pos) === pos` trick |
| `Number.isNaN` | `typeof === 'number' && value !== value`; no coercion |
| Guard pattern | `!method` vs `typeof === 'undefined'` vs try/catch |

---

## Final Notes

Polyfills are one of the most commonly tested topics in senior JavaScript interviews because they reveal whether a candidate truly understands how native APIs work under the hood. When writing a polyfill:

1. Always guard against overwriting a natively available method
2. Match the exact API: same parameter names, same return value, same error cases
3. Handle edge cases: `null`/`undefined` arguments, sparse arrays, empty collections
4. Understand the difference between what Babel transpiles (syntax) and what polyfills handle (runtime APIs)

The `bind`, `call`, `apply` trio and the `Promise` family are the most frequently asked polyfills in senior-level interviews at product companies.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
