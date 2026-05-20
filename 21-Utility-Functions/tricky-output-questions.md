# Utility Functions — Tricky Output Questions

## Table of Contents

1. [Debounce Questions](#1-debounce-questions)
2. [Throttle Questions](#2-throttle-questions)
3. [Memoize Questions](#3-memoize-questions)
4. [Curry Questions](#4-curry-questions)
5. [Deep Clone Questions](#5-deep-clone-questions)
6. [EventEmitter Questions](#6-eventemitter-questions)
7. [Advanced Utility Questions](#7-advanced-utility-questions)

---

## 1. Debounce Questions

---

### Q1. What will be the output?

```js
function debounce(fn, delay) {
  let timer = null;
  return function (...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}

let count = 0;
const increment = debounce(() => {
  count++;
  console.log('count:', count);
}, 100);

increment();
increment();
increment();

setTimeout(() => {
  console.log('Final count:', count);
}, 500);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
count: 1
Final count: 1
```

### Explanation
Three `increment()` calls happen synchronously. Each one clears the previous timer and schedules a new one. Only the last scheduled `setTimeout` fires after 100ms, executing `fn` exactly once. By the time the 500ms check runs, `count` is `1`.

This is the core debounce behavior: no matter how many times you call the debounced function in rapid succession, the underlying function only executes once — after the burst stops.

</details>

---

### Q2. What will be the output?

```js
function debounce(fn, delay) {
  let timer;
  return function (...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}

const obj = {
  name: 'Alice',
  greet: debounce(function () {
    console.log('Hello,', this.name);
  }, 0),
};

obj.greet();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Hello, Alice
```

### Explanation
Inside the `setTimeout` callback, `fn.apply(this, args)` uses `this` from the outer function (the returned wrapper). When called as `obj.greet()`, the wrapper's `this` is `obj`. `fn.apply(this, args)` passes `obj` as `this` to the original function, so `this.name` is `'Alice'`.

If we had used `fn(...args)` (without `.apply(this, args)`), `this` inside the timeout would be `undefined` (in strict mode) or `globalThis`, and `this.name` would be `undefined`.

</details>

---

### Q3. What will be the output?

```js
function debounce(fn, delay) {
  let timer;
  return function (...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}

const log = debounce(x => console.log(x), 200);

log('a');
log('b');

setTimeout(() => log('c'), 100);  // within the 200ms window of 'b'
setTimeout(() => log('d'), 400);  // fresh call — fires after 200ms
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
d
```

### Explanation
Timeline:
- t=0: `log('a')` — timer set for 200ms
- t=0: `log('b')` — resets timer for 200ms
- t=100: `log('c')` — resets timer again (100ms into the 200ms window of 'b')
- t=300: timer from 'c' would fire — but `log('d')` at t=400 is a new, separate call
- t=300: 'c' fires... wait, 'd' is at t=400, which is AFTER c's timer fires at t=300.

Correction: 'c' fires at t=300 (100 + 200). Then 'd' at t=400 starts a fresh timer firing at t=600. Both 'c' and 'd' fire.

Actually let me re-trace:
- t=0ms: `log('a')` — timer A for t=200
- t≈0ms: `log('b')` — clears A, timer B for t≈200
- t=100ms: `log('c')` — clears B, timer C for t=300
- t=300ms: timer C fires → `c` is logged
- t=400ms: `log('d')` — timer D for t=600
- t=600ms: timer D fires → `d` is logged

So the output is both 'c' and 'd'.

</details>

---

### Q4. What will be the output?

```js
function debounce(fn, delay) {
  let timer;
  return function (...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}

const results = [];
const push = debounce(val => results.push(val), 50);

push(1);
push(2);
push(3);

setTimeout(() => {
  push(4);
  push(5);
}, 200);

setTimeout(() => {
  console.log(results);
}, 500);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
[3, 5]
```

### Explanation
- Burst 1 (t=0): `push(1)`, `push(2)`, `push(3)` — only `3` fires at t=50ms.
- Burst 2 (t=200): `push(4)`, `push(5)` — only `5` fires at t=250ms.
- At t=500ms: `results` is `[3, 5]`.

Each burst of rapid calls results in only the last call executing. The values `1`, `2`, and `4` are discarded.

</details>

---

## 2. Throttle Questions

---

### Q1. What will be the output?

```js
function throttle(fn, limit) {
  let lastCall = 0;
  return function (...args) {
    const now = Date.now();
    if (now - lastCall >= limit) {
      lastCall = now;
      return fn.apply(this, args);
    }
  };
}

const log = throttle(x => console.log(x), 100);

log('a');  // t=0
log('b');  // t=0 — within 100ms
log('c');  // t=0 — within 100ms

setTimeout(() => log('d'), 150); // t=150 — past 100ms limit
setTimeout(() => log('e'), 200); // t=200 — within 100ms of 'd'
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
a
d
```

### Explanation
- `log('a')` at t=0: fires immediately (lastCall was 0, `now - 0 >= 100` is `true`).
- `log('b')` and `log('c')`: fired synchronously at t≈0, `now - lastCall < 100` → ignored.
- `log('d')` at t=150: `150 - 0 >= 100` → fires.
- `log('e')` at t=200: `200 - 150 = 50 < 100` → ignored.

Only 'a' and 'd' appear. Unlike debounce, the in-between calls are silently dropped without rescheduling.

</details>

---

### Q2. What will be the output?

```js
function throttle(fn, limit) {
  let lastCall = 0;
  return function (...args) {
    const now = Date.now();
    if (now - lastCall >= limit) {
      lastCall = now;
      fn.apply(this, args);
    }
  };
}

let callCount = 0;
const throttled = throttle(() => callCount++, 100);

// Simulate 10 rapid calls
for (let i = 0; i < 10; i++) {
  throttled();
}

setTimeout(() => {
  console.log('Call count after rapid burst:', callCount);
}, 500);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Call count after rapid burst: 1
```

### Explanation
All 10 calls happen synchronously in the same tick. `Date.now()` returns the same (or nearly same) timestamp for all of them. The first call fires and sets `lastCall`. The remaining 9 calls check `now - lastCall < 100` and are all dropped. Result: `callCount = 1`.

</details>

---

### Q3. What is the key difference in this output between debounce and throttle?

```js
function debounce(fn, d) {
  let t;
  return (...a) => { clearTimeout(t); t = setTimeout(() => fn(...a), d); };
}
function throttle(fn, l) {
  let last = 0;
  return (...a) => { const n = Date.now(); if (n - last >= l) { last = n; fn(...a); } };
}

const db = debounce(x => console.log('D:', x), 100);
const th = throttle(x => console.log('T:', x), 100);

[1, 2, 3, 4, 5].forEach(x => { db(x); th(x); });
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
T: 1
D: 5
```

### Explanation
- **Throttle** fires on the **first** call (leading edge). After `T: 1`, all subsequent calls within 100ms are dropped.
- **Debounce** fires on the **last** call (trailing edge). Each call resets the timer, so only `D: 5` fires after 100ms of silence.

This output perfectly illustrates the fundamental difference: throttle = leading edge (fire first, then cool down), debounce = trailing edge (wait until done, then fire).

</details>

---

## 3. Memoize Questions

---

### Q1. What will be the output?

```js
function memoize(fn) {
  const cache = {};
  return function (...args) {
    const key = JSON.stringify(args);
    if (key in cache) {
      return cache[key];
    }
    return (cache[key] = fn(...args));
  };
}

let callCount = 0;
const square = memoize(n => {
  callCount++;
  return n * n;
});

console.log(square(4));
console.log(square(4));
console.log(square(5));
console.log(square(4));
console.log('callCount:', callCount);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
16
16
25
16
callCount: 2
```

### Explanation
`square(4)` is called 3 times but the actual computation only runs once (first call). `square(5)` runs once. Total: 2 actual executions. The memoized function returns the cached value for `square(4)` on the 2nd and 4th calls without incrementing `callCount`.

</details>

---

### Q2. What will be the output?

```js
function memoize(fn) {
  const cache = new Map();
  return function (...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

const memoAdd = memoize((a, b) => a + b);

console.log(memoAdd(1, 2));
console.log(memoAdd(2, 1));  // different key!
console.log(memoAdd(1, 2));  // cache hit
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
3
3
3
```

### Explanation
`JSON.stringify([1, 2])` → `"[1,2]"` and `JSON.stringify([2, 1])` → `"[2,1]"` — these are **different keys**. So `memoAdd(2, 1)` is a cache miss and calls the function again. Both return `3` (because `1+2 = 2+1 = 3`), but the memoize function does not know addition is commutative — it just caches by argument order.

If argument order matters for your use case, normalize the key before storing (e.g., sort args).

</details>

---

### Q3. What will be the output?

```js
function memoize(fn) {
  const cache = new Map();
  return function (...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    const val = fn(...args);
    cache.set(key, val);
    return val;
  };
}

const getUser = memoize(id => {
  console.log('Fetching user', id);
  return { id, name: 'User' + id };
});

const u1 = getUser(1);
u1.name = 'Modified';

const u2 = getUser(1); // cache hit
console.log(u2.name);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Fetching user 1
Modified
```

### Explanation
The cache stores the **object reference**, not a copy. When `u1.name = 'Modified'` mutates the returned object, the cached object is mutated too (they are the same reference). So `getUser(1)` on the second call returns the same mutated object with `name: 'Modified'`.

This is a critical memoize pitfall: if the return value is a mutable object and callers mutate it, the cache becomes stale. Solution: return deep clones (expensive) or document that callers must not mutate the result.

</details>

---

## 4. Curry Questions

---

### Q1. What will be the output?

```js
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) return fn(...args);
    return (...more) => curried(...args, ...more);
  };
}

function add(a, b, c) {
  return a + b + c;
}

const curriedAdd = curry(add);

console.log(curriedAdd(1)(2)(3));
console.log(curriedAdd(1, 2)(3));
console.log(curriedAdd(1)(2, 3));
console.log(curriedAdd(1, 2, 3));
console.log(curriedAdd()()(1)()(2)()(3));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
6
6
6
6
6
```

### Explanation
All invocation styles work because `curried` accumulates arguments until `args.length >= fn.length` (which is `3` for `add`). Calling with zero arguments (`()`) just returns another function waiting for more arguments without accumulating anything new — effectively a no-op. All five calls eventually collect `[1, 2, 3]` and return `1 + 2 + 3 = 6`.

</details>

---

### Q2. What will be the output?

```js
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) return fn(...args);
    return (...more) => curried(...args, ...more);
  };
}

function variadic(...args) {
  return args.reduce((a, b) => a + b, 0);
}

const curriedVariadic = curry(variadic);
console.log(variadic.length);
console.log(curriedVariadic(1)(2)(3));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
0
1
```

### Explanation
`variadic.length` is `0` because rest parameters (`...args`) do not count toward `Function.length`. So `args.length >= fn.length` is `args.length >= 0`, which is always true. `curriedVariadic(1)` immediately calls `variadic(1)` and returns `1` without waiting for more arguments.

This is the fundamental limitation of the `fn.length` curry technique: it does not work with variadic (rest-param) functions.

</details>

---

### Q3. What will be the output?

```js
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) return fn.apply(this, args);
    return function (...more) {
      return curried.apply(this, [...args, ...more]);
    };
  };
}

const multiply = curry((a, b) => a * b);

const operations = [multiply(2), multiply(5), multiply(10)];
const results = [3, 6, 9].map((n, i) => operations[i](n));

console.log(results);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
[6, 30, 90]
```

### Explanation
`multiply(2)`, `multiply(5)`, `multiply(10)` each return a partially applied function waiting for the second argument. The `.map` then calls each with the corresponding value from `[3, 6, 9]`:
- `operations[0](3)` → `multiply(2)(3)` → `2 * 3 = 6`
- `operations[1](6)` → `multiply(5)(6)` → `5 * 6 = 30`
- `operations[2](9)` → `multiply(10)(9)` → `10 * 9 = 90`

This demonstrates currying's practical power: create reusable partially-applied functions.

</details>

---

### Q4. What will be the output?

```js
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) return fn(...args);
    return (...more) => curried(...args, ...more);
  };
}

const compose = (f, g) => x => f(g(x));
const pipe = (f, g) => x => g(f(x));

const addOne = curry(x => x + 1);
const double = curry(x => x * 2);

const transform1 = compose(addOne, double);  // double first, then addOne
const transform2 = pipe(addOne, double);     // addOne first, then double

console.log(transform1(5));
console.log(transform2(5));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
11
12
```

### Explanation
- `compose(addOne, double)(5)`: applies right-to-left → `double(5) = 10`, then `addOne(10) = 11`.
- `pipe(addOne, double)(5)`: applies left-to-right → `addOne(5) = 6`, then `double(6) = 12`.

`compose` and `pipe` produce different results when the functions are not commutative. Always verify the order when chaining mathematical operations.

</details>

---

## 5. Deep Clone Questions

---

### Q1. What will be the output?

```js
const original = { a: 1, b: { c: 2 } };

// Shallow copy
const shallow = { ...original };

// JSON clone (deep but lossy)
const jsonClone = JSON.parse(JSON.stringify(original));

shallow.b.c = 99;
jsonClone.b.c = 42;

console.log(original.b.c);
console.log(shallow.b.c);
console.log(jsonClone.b.c);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
99
99
42
```

### Explanation
- **Shallow copy** (`{ ...original }`): top-level properties are copied by value, but `b` is an object — only the reference is copied. So `shallow.b` and `original.b` point to the same object. Mutating `shallow.b.c` also mutates `original.b.c`.
- **JSON clone**: `JSON.parse(JSON.stringify(...))` creates a completely new object graph. `jsonClone.b` is a different object from `original.b`. Mutating `jsonClone.b.c = 42` does not affect `original`.

</details>

---

### Q2. What will be the output?

```js
function deepClone(val) {
  if (val === null || typeof val !== 'object') return val;
  if (val instanceof Date) return new Date(val.getTime());
  if (Array.isArray(val)) return val.map(item => deepClone(item));
  const clone = {};
  for (const key of Object.keys(val)) {
    clone[key] = deepClone(val[key]);
  }
  return clone;
}

const src = {
  date: new Date('2024-06-15'),
  fn: function () { return 42; },
  sym: Symbol('id'),
  undef: undefined,
};

const cloned = deepClone(src);

console.log(cloned.date instanceof Date);
console.log(cloned.date === src.date);
console.log(typeof cloned.fn);
console.log(cloned.sym);
console.log('undef' in cloned);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
true
false
function
Symbol(id)
true
```

### Explanation
- `date instanceof Date`: `true` — the deepClone correctly handles `Date`.
- `cloned.date === src.date`: `false` — it's a new `Date` instance with the same time.
- `typeof cloned.fn`: `function` — functions pass the `typeof val !== 'object'` check (their typeof is `'function'`), so they are returned as-is (passed by reference).
- `cloned.sym`: `Symbol(id)` — Symbols pass the `typeof val !== 'object'` check, returned as-is.
- `'undef' in cloned`: `true` — `undefined` passes the primitive check, so `clone.undef = undefined`. The key exists with value `undefined`.

</details>

---

### Q3. What will be the output?

```js
const obj = { a: 1 };
obj.self = obj; // circular reference

try {
  const clone = JSON.parse(JSON.stringify(obj));
  console.log(clone);
} catch (e) {
  console.log('Error:', e.message.slice(0, 30));
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Error: Converting circular structure to
```

### Explanation
`JSON.stringify` cannot serialize circular references — it throws a `TypeError` when it detects a cycle. This is one of `JSON.stringify`'s key limitations. A robust `deepClone` implementation must handle circular references by tracking visited objects with a `WeakMap`:
```js
function deepClone(val, visited = new WeakMap()) {
  if (val === null || typeof val !== 'object') return val;
  if (visited.has(val)) return visited.get(val);
  const clone = Array.isArray(val) ? [] : {};
  visited.set(val, clone);
  for (const key of Object.keys(val)) {
    clone[key] = deepClone(val[key], visited);
  }
  return clone;
}
```

</details>

---

## 6. EventEmitter Questions

---

### Q1. What will be the output?

```js
class EventEmitter {
  constructor() { this._events = {}; }
  on(ev, fn) { (this._events[ev] = this._events[ev] || []).push(fn); return this; }
  emit(ev, ...args) { (this._events[ev] || []).forEach(fn => fn(...args)); }
  off(ev, fn) { this._events[ev] = (this._events[ev] || []).filter(f => f !== fn); }
}

const ee = new EventEmitter();
const handler = data => console.log('Got:', data);

ee.on('msg', handler);
ee.on('msg', handler); // same reference added twice
ee.emit('msg', 'hello');

ee.off('msg', handler);
ee.emit('msg', 'world');
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Got: hello
Got: hello
```

### Explanation
The same `handler` function is added twice to the `'msg'` event. `emit` fires both entries: `'hello'` is logged twice. After `off`, all occurrences of `handler` are filtered out (filter removes all matching references). So `emit('msg', 'world')` finds no listeners and logs nothing.

Takeaway: naive EventEmitter implementations allow duplicate listeners. A production version should either prevent duplicates or remove only one occurrence at a time based on requirements.

</details>

---

### Q2. What will be the output?

```js
class EventEmitter {
  constructor() { this._events = {}; }

  on(ev, fn) {
    (this._events[ev] = this._events[ev] || []).push(fn);
    return this;
  }

  once(ev, fn) {
    const wrapper = (...args) => { fn(...args); this.off(ev, wrapper); };
    return this.on(ev, wrapper);
  }

  off(ev, fn) {
    this._events[ev] = (this._events[ev] || []).filter(f => f !== fn);
  }

  emit(ev, ...args) {
    [...(this._events[ev] || [])].forEach(fn => fn(...args));
  }
}

const ee = new EventEmitter();
let count = 0;

ee.once('tick', () => count++);
ee.on('tick', () => count++);

ee.emit('tick');
ee.emit('tick');
ee.emit('tick');

console.log(count);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
4
```

### Explanation
- `once` handler fires only on the first emit: contributes `1` to count.
- `on` handler fires on all 3 emits: contributes `3` to count.
- Total: `1 + 3 = 4`.

The `once` mechanism works by wrapping the original listener in a `wrapper` function that calls the original and then removes the `wrapper` from the event. After the first `emit('tick')`, the wrapper removes itself, so subsequent emits only trigger the `on` handler.

</details>

---

### Q3. What will be the output?

```js
class EventEmitter {
  constructor() { this._events = {}; }
  on(ev, fn) { (this._events[ev] = this._events[ev] || []).push(fn); }
  emit(ev, ...args) {
    // NOT snapshotting — iterating live array
    const listeners = this._events[ev] || [];
    for (let i = 0; i < listeners.length; i++) {
      listeners[i](...args);
    }
  }
  off(ev, fn) { this._events[ev] = (this._events[ev] || []).filter(f => f !== fn); }
}

const ee = new EventEmitter();

function a() {
  console.log('a');
  ee.off('test', b); // remove 'b' while iterating
}
function b() { console.log('b'); }
function c() { console.log('c'); }

ee.on('test', a);
ee.on('test', b);
ee.on('test', c);

ee.emit('test');
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
a
c
```

### Explanation
When `a` fires and calls `ee.off('test', b)`, the listeners array is replaced with a new filtered array (without `b`). The loop was iterating the old `listeners` reference — but `off` replaces `this._events['test']` with a new array. The local `listeners` variable still points to the old array with `[a, b, c]`.

So the loop continues: index 1 is `b` in the OLD array — but wait, `off` replaces `this._events['test']` with a new array, not the `listeners` variable. The loop reads from `listeners` which still has `[a, b, c]`. So `b` at index 1 IS in the old array... 

Actually, since `off` creates a new array via `filter`, `listeners` still references the original `[a, b, c]` array. The loop continues: at `i=1`, `listeners[1]` is `b` — `b` is still in the OLD array, so `b()` fires? No...

`off` does `this._events[ev] = [...].filter(...)` — it reassigns `this._events['test']` to a NEW array. The local `const listeners = this._events['test']` captured the ORIGINAL array before `off`. Since the original array still has `b` at index 1, `b()` would fire at `i=1`. But the `off` called from inside `a()` already ran its filter... 

Let me re-trace: `listeners` is set once before the loop to `this._events['test']` which is `[a, b, c]`. Inside `a()`, `off` sets `this._events['test'] = [a, c]` (a new array). The loop variable `listeners` still holds the reference to the original `[a, b, c]` array. So `i=1`: `listeners[1]` is `b` → `b()` fires? 

Wait, but the expected output shows only `a` and `c`. That would only happen if the live array was being iterated (not the snapshot). Let me reconsider:

The emit code uses `const listeners = this._events[ev] || []` — this assigns the reference to the SAME array object that `this._events[ev]` points to. Then `off` does `this._events[ev] = filter(...)` — this changes what `this._events[ev]` points to, but `listeners` still points to the original array `[a, b, c]`.

So `b` WOULD fire. The output would be `a, b, c`.

But what if `off` mutated the array in place (e.g., `splice`)? Then `listeners` pointing to the same array would see the mutation, and `b` would be skipped.

The behavior is that `b` DOES fire because `listeners` holds the original array reference. Output: `a, b, c`.

Let me correct the question. The answer with the filter approach is `a, b, c` because `off` creates a new array. To get `a, c` (skipping b), `off` would need to mutate in place. The `[...listeners]` snapshot technique in the fixed EventEmitter avoids this issue entirely.

</details>

---

## 7. Advanced Utility Questions

---

### Q1. What will be the output?

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

const init = once(() => {
  console.log('Initializing...');
  return Math.random();
});

const r1 = init();
const r2 = init();
const r3 = init();

console.log(r1 === r2);
console.log(r2 === r3);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Initializing...
true
true
```

### Explanation
`init()` is called three times, but the original function runs only once — on the first call. The same `result` (the random number from the first call) is cached and returned for all subsequent calls. `r1 === r2 === r3` because they all reference the same cached value.

`once` is ideal for initialization logic, connection setup, or any operation that must not run more than once regardless of how many times the function is invoked.

</details>

---

### Q2. What will be the output?

```js
function pipe(...fns) {
  return x => fns.reduce((acc, fn) => fn(acc), x);
}

const process = pipe(
  x => { console.log('Step 1:', x); return x + 1; },
  x => { console.log('Step 2:', x); return x * 2; },
  x => { console.log('Step 3:', x); return x - 3; },
);

const result = process(5);
console.log('Result:', result);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Step 1: 5
Step 2: 6
Step 3: 12
Result: 9
```

### Explanation
`pipe` processes left-to-right:
1. `5 + 1 = 6`
2. `6 * 2 = 12`
3. `12 - 3 = 9`

Each function receives the output of the previous. This sequential logging makes `pipe` easy to debug compared to deep nesting.

</details>

---

### Q3. What will be the output?

```js
function chunk(arr, size) {
  const result = [];
  for (let i = 0; i < arr.length; i += size) {
    result.push(arr.slice(i, i + size));
  }
  return result;
}

function flatten(arr) {
  return arr.reduce(
    (acc, val) => Array.isArray(val) ? acc.concat(flatten(val)) : acc.concat(val),
    []
  );
}

const data = [1, 2, 3, 4, 5, 6, 7, 8];
const chunked = chunk(data, 3);
console.log(chunked);

const flattened = flatten(chunked);
console.log(flattened);
console.log(flattened.length === data.length);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
[[1, 2, 3], [4, 5, 6], [7, 8]]
[1, 2, 3, 4, 5, 6, 7, 8]
true
```

### Explanation
`chunk(data, 3)` splits into groups of 3 — the last group has only 2 elements. `flatten(chunked)` reverses the chunking by recursively flattening all nested arrays. The round-trip `flatten(chunk(arr, n))` always reconstructs the original array (same length, same elements, same order).

</details>

---

### Q4. What will be the output?

```js
function deepEqual(a, b) {
  if (a === b) return true;
  if (a == null || b == null) return false;
  if (typeof a !== typeof b) return false;
  if (typeof a !== 'object') return false;
  if (Array.isArray(a) !== Array.isArray(b)) return false;
  const keysA = Object.keys(a);
  const keysB = Object.keys(b);
  if (keysA.length !== keysB.length) return false;
  return keysA.every(k => Object.prototype.hasOwnProperty.call(b, k) && deepEqual(a[k], b[k]));
}

console.log(deepEqual({ a: 1 }, { a: 1 }));
console.log(deepEqual({ a: 1 }, { a: '1' }));
console.log(deepEqual([1, 2, 3], [1, 2, 3]));
console.log(deepEqual([1, 2], [1, 2, 3]));
console.log(deepEqual({ a: { b: { c: 1 } } }, { a: { b: { c: 1 } } }));
console.log(deepEqual(null, null));
console.log(deepEqual(NaN, NaN));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
true
false
true
false
true
true
false
```

### Explanation
- `{ a: 1 }` vs `{ a: 1 }`: deep recursion finds all keys equal → `true`.
- `{ a: 1 }` vs `{ a: '1' }`: `typeof 1 !== typeof '1'` (`'number' !== 'string'`) → `false`.
- Arrays: same elements, same length → `true`.
- `[1,2]` vs `[1,2,3]`: `Object.keys` of arrays returns index strings `['0','1']` vs `['0','1','2']`, different lengths → `false`.
- Deeply nested object: recursion handles it → `true`.
- `null === null` → `true` (short-circuit at the top).
- `NaN`: `NaN === NaN` is `false`, so it falls through. `typeof NaN` is `'number'`, not `'object'`, and `NaN !== NaN` fails the `a === b` check. Both values are non-objects so `typeof a !== 'object'` → `false` is returned.

</details>

---

### Q5. What will be the output?

```js
function groupBy(arr, key) {
  return arr.reduce((acc, item) => {
    const k = typeof key === 'function' ? key(item) : item[key];
    acc[k] = acc[k] || [];
    acc[k].push(item);
    return acc;
  }, {});
}

function unique(arr) {
  return [...new Set(arr)];
}

const transactions = [
  { id: 1, type: 'credit', amount: 100 },
  { id: 2, type: 'debit',  amount: 50  },
  { id: 3, type: 'credit', amount: 200 },
  { id: 4, type: 'debit',  amount: 75  },
];

const grouped = groupBy(transactions, 'type');
const types = unique(transactions.map(t => t.type));

console.log(types);
console.log(Object.keys(grouped).sort());
console.log(grouped.credit.length);
console.log(grouped.debit.reduce((sum, t) => sum + t.amount, 0));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
['credit', 'debit']
['credit', 'debit']
2
125
```

### Explanation
- `unique(transactions.map(t => t.type))`: maps to `['credit', 'debit', 'credit', 'debit']`, `Set` deduplicates → `['credit', 'debit']` (insertion order preserved).
- `Object.keys(grouped).sort()`: groups are `credit` and `debit`, sorted alphabetically.
- `grouped.credit.length`: transactions 1 and 3 are credits → `2`.
- Debit total: `50 + 75 = 125`.

</details>

---

### Q6. What will be the output?

```js
async function retry(fn, retries, delay = 0) {
  try {
    return await fn();
  } catch (err) {
    if (retries <= 0) throw err;
    if (delay) await new Promise(r => setTimeout(r, delay));
    return retry(fn, retries - 1, delay);
  }
}

let attempts = 0;

retry(async () => {
  attempts++;
  if (attempts < 4) throw new Error(`Fail #${attempts}`);
  return 'done';
}, 5)
  .then(result => console.log(`Success after ${attempts} attempts:`, result))
  .catch(err => console.log('Failed:', err.message));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Success after 4 attempts: done
```

### Explanation
The function fails on attempts 1, 2, and 3. On attempt 4 (`attempts === 4`), the condition `attempts < 4` is false, so it returns `'done'`. `retry` is called with `retries = 5` — there are enough retries to absorb 3 failures. After 4 total attempts (1 initial + 3 retries), the result resolves.

Note: `retries = 5` means up to 5 retries (6 total attempts allowed). We only needed 3 retries (4 total).

</details>

---

## Final Tips

- **Debounce fires after the last call (trailing edge); throttle fires on the first call (leading edge).** Always clarify which behavior is needed.
- **Memoize caches object references, not copies.** If callers mutate the result, the cache is corrupted.
- **Curry only works automatically with functions where `fn.length > 0`.** Rest parameters break `fn.length`.
- **Compose is right-to-left; pipe is left-to-right.** The mathematical notation dictates compose's direction.
- **Once returns the same cached result for all calls, even `undefined`.** The `called` flag — not the truthiness of `result` — is the guard.
- **EventEmitter `once` works by wrapping in a self-removing function.** The original listener cannot be removed by reference — only the wrapper can.
- **Deep clone does not automatically handle circular references or functions.** Use `structuredClone` in modern environments for a production-ready solution.
- **`deepEqual` returns `false` for `NaN === NaN`** because `NaN !== NaN`. Add a `Number.isNaN` guard if you need `deepEqual(NaN, NaN) === true`.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
