# Polyfills — Tricky Output Questions

## Table of Contents

1. [Custom map / filter / reduce Questions](#1-custom-map--filter--reduce-questions)
2. [Custom bind / call / apply Questions](#2-custom-bind--call--apply-questions)
3. [Custom Promise Questions](#3-custom-promise-questions)
4. [Polyfill Edge Case Questions](#4-polyfill-edge-case-questions)
5. [Implementation Questions](#5-implementation-questions)

---

## 1. Custom map / filter / reduce Questions

---

### Q1. What will be the output?

```js
Array.prototype.myMap = function (cb) {
  const result = [];
  for (let i = 0; i < this.length; i++) {
    result.push(cb(this[i], i, this));
  }
  return result;
};

const doubled = [1, 2, 3].myMap((x, i) => x + i);
console.log(doubled);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
[1, 3, 5]
```

### Explanation
The callback receives `(element, index)`. So:
- `1 + 0 = 1`
- `2 + 1 = 3`
- `3 + 2 = 5`

The `map` callback signature is `(currentValue, index, array)`, not just the value. Many candidates forget the index argument and produce wrong answers.

</details>

---

### Q2. What will be the output?

```js
Array.prototype.myReduce = function (cb, init) {
  let acc = arguments.length >= 2 ? init : this[0];
  let start = arguments.length >= 2 ? 0 : 1;
  for (let i = start; i < this.length; i++) {
    acc = cb(acc, this[i], i, this);
  }
  return acc;
};

const result = [1, 2, 3, 4].myReduce((acc, val) => acc * val);
console.log(result);

const result2 = [1, 2, 3, 4].myReduce((acc, val) => acc * val, 0);
console.log(result2);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
24
0
```

### Explanation
- **First call (no initial value):** `acc` starts as `this[0] = 1`. Iteration starts from index 1. `1*2=2`, `2*3=6`, `6*4=24`.
- **Second call (initialValue = 0):** `acc` starts as `0`. `0*1=0`, `0*2=0`, etc. Any product with 0 stays 0.

This is a common gotcha: an initial value of `0` completely changes the result for multiplication.

</details>

---

### Q3. What will be the output?

```js
Array.prototype.myFilter = function (cb) {
  const result = [];
  for (let i = 0; i < this.length; i++) {
    if (i in this && cb(this[i], i, this)) {
      result.push(this[i]);
    }
  }
  return result;
};

const sparse = [1, , 3, , 5];
const filtered = sparse.myFilter(x => x > 0);
console.log(filtered);
console.log(filtered.length);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
[1, 3, 5]
3
```

### Explanation
`[1, , 3, , 5]` is a sparse array. The holes at indices 1 and 3 fail the `i in this` check, so the callback is never called for them. The holes are NOT included in the result. This is why `filtered.length` is `3`, not `5`.

</details>

---

### Q4. What will be the output?

```js
const users = [
  { name: 'Alice', age: 25 },
  { name: 'Bob',   age: 17 },
  { name: 'Carol', age: 30 },
];

const result = users
  .filter(u => u.age >= 18)
  .map(u => u.name)
  .reduce((acc, name) => acc + ', ' + name);

console.log(result);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Alice, Carol
```

### Explanation
1. `filter` keeps `Alice` (25) and `Carol` (30), removes `Bob` (17).
2. `map` extracts names: `['Alice', 'Carol']`.
3. `reduce` without an initial value: `acc = 'Alice'`, then `'Alice' + ', ' + 'Carol'`.

Note: if only one user passed the filter, `reduce` with no initial value returns that single element directly without calling the callback.

</details>

---

### Q5. What will be the output?

```js
const result = [].reduce((acc, val) => acc + val);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
TypeError: Reduce of empty array with no initial value
```

### Explanation
Calling `reduce` on an empty array without providing an `initialValue` throws a `TypeError`. This is by spec. To avoid this, always provide an initial value when the array might be empty: `[].reduce((acc, val) => acc + val, 0)` returns `0`.

</details>

---

## 2. Custom bind / call / apply Questions

---

### Q1. What will be the output?

```js
function greet(greeting, name) {
  return `${greeting}, ${name}! I am ${this.role}.`;
}

const admin = { role: 'Admin' };

const boundGreet = greet.bind(admin, 'Hello');
console.log(boundGreet('Alice'));
console.log(boundGreet('Bob'));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Hello, Alice! I am Admin.
Hello, Bob! I am Admin.
```

### Explanation
`bind(admin, 'Hello')` permanently sets `this` to `admin` and pre-fills the first argument `greeting = 'Hello'`. Each call to `boundGreet` only needs the second argument `name`. This demonstrates partial application.

</details>

---

### Q2. What will be the output?

```js
function Person(name) {
  this.name = name;
}

const BoundPerson = Person.bind({ name: 'Ignored' });

const p = new BoundPerson('Alice');
console.log(p.name);
console.log(p instanceof Person);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Alice
true
```

### Explanation
When a bound function is called with `new`, the `thisArg` passed to `bind` is **ignored**. The `new` operator creates a fresh object and passes it as `this`. So `this.name = 'Alice'` is set on the new instance, not on `{ name: 'Ignored' }`. `instanceof Person` returns `true` because `BoundPerson.prototype` is linked to `Person.prototype`.

</details>

---

### Q3. What will be the output?

```js
const obj = {
  value: 42,
  getValue: function () {
    return this.value;
  },
};

const fn = obj.getValue;
console.log(fn());                   // A
console.log(fn.call(obj));           // B
console.log(fn.call({ value: 99 })); // C
console.log(fn.apply(null));         // D (in non-strict mode)
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
undefined
42
99
undefined
```

### Explanation
- **A:** `fn()` is a plain function call. `this` is the global object (`window`/`globalThis`) where `value` is not defined → `undefined`.
- **B:** `fn.call(obj)` explicitly sets `this = obj` → `obj.value = 42`.
- **C:** `fn.call({ value: 99 })` sets `this` to a new object → `99`.
- **D:** `fn.apply(null)` in non-strict mode defaults `this` to the global object, where `value` is not defined → `undefined`.

</details>

---

### Q4. What will be the output?

```js
Function.prototype.myBind = function (context, ...outerArgs) {
  const fn = this;
  return function (...innerArgs) {
    return fn.apply(context, [...outerArgs, ...innerArgs]);
  };
};

function multiply(a, b, c) {
  return a * b * c;
}

const double = multiply.myBind(null, 2);
const sixTimes = double.myBind(null, 3);

console.log(double(3, 4));
console.log(sixTimes(5));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
24
30
```

### Explanation
- `double` = `multiply` with `a = 2` pre-filled. `double(3, 4)` calls `multiply(2, 3, 4) = 24`.
- `sixTimes` = `double` with the first remaining argument `3` pre-filled. But `double` is already a bound function, not `multiply`. `sixTimes(5)` calls `double(3, 5)` which calls `multiply(2, 3, 5) = 30`.

Note: `myBind` applied to another `myBind` result does **not** ignore the second binding's `context`. This differs slightly from native `bind` behavior where re-binding a bound function has no effect on `this`.

</details>

---

### Q5. What will be the output?

```js
const obj1 = { x: 10 };
const obj2 = { x: 20 };

function showX() {
  console.log(this.x);
}

const bound = showX.bind(obj1);
bound();
bound.call(obj2);       // Can you override bind with call?
bound.apply(obj2);      // Can you override bind with apply?
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
10
10
10
```

### Explanation
Once a function is bound with `.bind()`, its `this` is **permanently fixed**. Neither `.call()` nor `.apply()` can override the bound `this`. This is a key distinction between `bind` (creates a new permanently-bound function) and `call`/`apply` (temporarily sets `this` for one invocation).

</details>

---

## 3. Custom Promise Questions

---

### Q1. What will be the output?

```js
const p = new Promise((resolve, reject) => {
  resolve(1);
  reject(2);    // called after resolve
  resolve(3);   // called again
});

p.then(val => console.log('Resolved:', val))
 .catch(err => console.log('Rejected:', err));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Resolved: 1
```

### Explanation
A Promise's state transitions are **one-way and one-time**. Once `resolve(1)` is called, the state changes to `fulfilled`. The subsequent `reject(2)` and `resolve(3)` calls are completely ignored because `state !== 'pending'`. This must be guarded in any polyfill with a `if (this.state === 'pending')` check.

</details>

---

### Q2. What will be the output?

```js
Promise.resolve(1)
  .then(x => {
    console.log(x);
    return x + 1;
  })
  .then(x => {
    console.log(x);
    throw new Error('oops');
  })
  .then(x => {
    console.log('Never:', x);
  })
  .catch(e => {
    console.log('Caught:', e.message);
    return 99;
  })
  .then(x => console.log('After catch:', x));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
1
2
Caught: oops
After catch: 99
```

### Explanation
The `.then` chain passes values forward. When the second `.then` throws, execution skips the third `.then` and jumps to the nearest `.catch`. The `.catch` handler returns `99`, which resolves the promise it returns. The final `.then` receives `99`. This demonstrates that `.catch` does not terminate the chain — it can recover from errors.

</details>

---

### Q3. What will be the output?

```js
const p1 = Promise.resolve('a');
const p2 = new Promise(res => setTimeout(() => res('b'), 100));
const p3 = Promise.reject('error-c');

Promise.all([p1, p2, p3])
  .then(vals => console.log('All:', vals))
  .catch(err => console.log('Caught:', err));

Promise.race([p1, p2, p3])
  .then(val => console.log('Race resolved:', val))
  .catch(err => console.log('Race rejected:', err));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Race resolved: a
Caught: error-c
```

### Explanation
- **Promise.race:** `p1` is already resolved synchronously (via `Promise.resolve`). It settles first → `Race resolved: a`. Even though `p3` is also immediately rejected, `p1` wins.
- **Promise.all:** `p3` is rejected. `Promise.all` short-circuits on the first rejection → `Caught: error-c`.

Note: The race output appears before the all output because both `p1` and `p3` are micro-task-queue resolutions, but the order depends on how implementations process them. In practice both log nearly simultaneously; the exact order can vary slightly by environment but `all`'s catch fires before the 100ms `p2` resolves.

</details>

---

### Q4. What will be the output?

```js
async function test() {
  const result = await new Promise((resolve) => {
    resolve(Promise.resolve(42));
  });
  console.log(result);
}

test();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
42
```

### Explanation
When `resolve` is called with a Promise (a "thenable"), the outer Promise adopts the state of the inner Promise. It does NOT resolve with the inner Promise object itself — instead it waits for the inner Promise to settle. So `result` is `42`, not `Promise { 42 }`. This is part of the Promise resolution procedure.

</details>

---

### Q5. What will be the output?

```js
console.log('1');

setTimeout(() => console.log('2'), 0);

Promise.resolve().then(() => console.log('3'));

console.log('4');
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
1
4
3
2
```

### Explanation
Execution order:
1. Synchronous code runs first: `1` then `4`.
2. Microtask queue (Promise `.then` callbacks) runs before the task queue: `3`.
3. Macrotask queue (setTimeout callback) runs last: `2`.

This is the foundation of understanding Promise timing. Promise callbacks are microtasks — they always run before setTimeout callbacks, even if the timeout is 0ms.

</details>

---

## 4. Polyfill Edge Case Questions

---

### Q1. What will be the output?

```js
// A naive Number.isNaN polyfill
function myIsNaN(val) {
  return val !== val;
}

// vs the correct polyfill
function correctIsNaN(val) {
  return typeof val === 'number' && val !== val;
}

console.log(myIsNaN(NaN));
console.log(myIsNaN({ valueOf() { return NaN; } }));

console.log(correctIsNaN(NaN));
console.log(correctIsNaN({ valueOf() { return NaN; } }));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
true
false
true
false
```

### Explanation
`NaN !== NaN` is `true` only for the actual `NaN` primitive. An object is always equal to itself (`obj !== obj` is always `false`), so the naive polyfill correctly returns `false` here. Both implementations agree in this case. The difference matters for strings: `'abc' !== 'abc'` is `false`, so the naive version correctly returns `false` for strings too. The `typeof` guard in `correctIsNaN` is important for explicitness and matches the spec exactly.

</details>

---

### Q2. What will be the output?

```js
const obj = Object.create(null);
console.log(obj.toString);
console.log(Object.getPrototypeOf(obj));

const obj2 = Object.create(Object.prototype);
console.log(typeof obj2.toString);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
undefined
null
function
```

### Explanation
`Object.create(null)` creates a "pure" dictionary with no prototype chain. It inherits nothing — not even `toString` or `hasOwnProperty`. This is why `obj.toString` is `undefined`. `Object.getPrototypeOf(obj)` confirms the prototype is `null`. `obj2` is created with `Object.prototype` as its prototype, so it inherits all standard methods.

</details>

---

### Q3. What will be the output?

```js
const target = { a: 1 };
const source = Object.create({ inherited: true });
source.own = 2;

const result = Object.assign(target, source);
console.log(result);
console.log('inherited' in result);
console.log(result.hasOwnProperty('inherited'));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
{ a: 1, own: 2 }
false
false
```

### Explanation
`Object.assign` copies only **own enumerable** properties. `inherited` lives on `source`'s prototype chain — it is not an own property of `source` — so it is NOT copied. The `for...in` loop in a polyfill would iterate inherited properties too, so `hasOwnProperty.call(source, key)` guard is critical to match native behavior.

</details>

---

### Q4. What will be the output?

```js
Array.prototype.myFlat = function (depth = 1) {
  const result = [];
  (function flatten(arr, d) {
    for (const item of arr) {
      if (Array.isArray(item) && d > 0) {
        flatten(item, d - 1);
      } else {
        result.push(item);
      }
    }
  })(this, depth);
  return result;
};

console.log([1, [2, [3]]].myFlat(0));
console.log([1, [2, [3]]].myFlat(1));
console.log([1, [2, [3]]].myFlat(Infinity));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
[1, [2, [3]]]
[1, 2, [3]]
[1, 2, 3]
```

### Explanation
- `depth = 0`: No flattening at all — array is returned as-is (as a new array).
- `depth = 1`: Only the outermost level is flattened. `[2, [3]]` becomes `2, [3]`.
- `depth = Infinity`: Recursion continues as long as an item is an array → fully flattened.

</details>

---

### Q5. What will be the output?

```js
String.prototype.myStartsWith = function (searchStr, pos) {
  const start = Math.max(0, pos | 0);
  return this.indexOf(searchStr, start) === start;
};

console.log('hello world'.myStartsWith(''));
console.log('hello world'.myStartsWith('', 5));
console.log('hello world'.myStartsWith('world', 6));
console.log('hello world'.myStartsWith('world', 5));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
true
true
true
false
```

### Explanation
- Empty string `''`: `indexOf('', 0)` returns `0`, which equals `start = 0` → `true`. An empty search string always matches at any position.
- `('', 5)`: `indexOf('', 5)` returns `5`, which equals `start = 5` → `true`.
- `('world', 6)`: `indexOf('world', 6)` finds `'world'` at index 6, equals `start = 6` → `true`.
- `('world', 5)`: `indexOf('world', 5)` finds `'world'` at index 6, but `start = 5`, `6 !== 5` → `false`.

</details>

---

## 5. Implementation Questions

---

### Q1. What is wrong with this `bind` polyfill and what will be the output?

```js
Function.prototype.badBind = function (context) {
  const fn = this;
  return function () {
    return fn.call(context);
  };
};

function add(a, b) {
  return a + b;
}

const add5 = add.badBind(null, 5);
console.log(add5(3));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
NaN
```

### Explanation
The `badBind` polyfill does not capture the extra arguments passed to `bind` (partial application), nor does it forward arguments passed when calling the returned function. `a` and `b` are both `undefined` inside the call, so `undefined + undefined = NaN`.

The correct implementation needs `...outerArgs` in the bind and `...innerArgs` in the returned function:
```js
Function.prototype.myBind = function (context, ...outerArgs) {
  const fn = this;
  return function (...innerArgs) {
    return fn.apply(context, [...outerArgs, ...innerArgs]);
  };
};
```

</details>

---

### Q2. What will be the output?

```js
const original = Array.prototype.map;

Array.prototype.map = function (cb) {
  console.log('Custom map called');
  return original.call(this, cb);
};

const result = [1, 2, 3].map(x => x * 2);
console.log(result);

Array.prototype.map = original; // restore
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Custom map called
[2, 4, 6]
```

### Explanation
This pattern — wrapping and then delegating to the original — is common for debugging and instrumentation. The key technique is saving the original before overwriting and using `original.call(this, cb)` to invoke it with the correct `this` (the array instance) and forward all arguments.

</details>

---

### Q3. What will be the output?

```js
function myMapPolyfill(arr, cb) {
  return arr.reduce((acc, val, i) => {
    acc.push(cb(val, i, arr));
    return acc;
  }, []);
}

const result = myMapPolyfill([1, 2, 3], (x, i) => `${i}=${x}`);
console.log(result);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
['0=1', '1=2', '2=3']
```

### Explanation
`map` can be implemented using `reduce`. This is a popular interview question: implement one array method using another. `reduce` accumulates into an initially-empty array by pushing transformed values. The same technique can implement `filter`: check the condition before pushing.

</details>

---

### Q4. What will be the output?

```js
if (typeof Array.prototype.customMethod === 'undefined') {
  Array.prototype.customMethod = function () {
    return 'polyfilled';
  };
}

Array.prototype.customMethod = 0; // Someone accidentally sets it to 0

if (!Array.prototype.customMethod) {
  Array.prototype.customMethod = function () {
    return 'second polyfill';
  };
}

console.log([].customMethod());
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
second polyfill
```

### Explanation
After `Array.prototype.customMethod = 0`, the property is falsy. The second guard `if (!Array.prototype.customMethod)` is truthy, so the second polyfill overwrites the `0`. This illustrates why `typeof === 'undefined'` is a stricter guard: it only runs if the property is truly absent, not if it is falsy (0, null, false, ''). The first guard would NOT have been triggered by `= 0`, but the second `!` guard was.

</details>

---

### Q5. What will be the output?

```js
// Polyfill for Array.prototype.findIndex
Array.prototype.myFindIndex = function (cb, thisArg) {
  for (let i = 0; i < this.length; i++) {
    if (cb.call(thisArg, this[i], i, this)) return i;
  }
  return -1;
};

const arr = [5, 12, 8, 130, 44];
console.log(arr.myFindIndex(x => x > 10));
console.log(arr.myFindIndex(x => x > 200));
console.log(arr.myFindIndex((x, i) => i === 2));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
1
-1
2
```

### Explanation
- `x > 10`: First match is `12` at index `1`.
- `x > 200`: No element exceeds 200, so `-1` is returned.
- `i === 2`: Callback tests the index, not the value. Index `2` is `8` — the callback returns `true` when `i === 2`, returning index `2`.

`findIndex` returns `-1` on no match (like `indexOf`), whereas `find` returns `undefined`.

</details>

---

## Final Tips

- **Always guard polyfills:** Check for native existence before patching prototypes.
- **`bind` vs `call` vs `apply`:** `bind` returns a new function; `call`/`apply` invoke immediately. `apply` takes an array of args.
- **`reduce` with no initial value on an empty array:** Always throws `TypeError`. When in doubt, provide an initial value.
- **Promise state is immutable after first transition:** `resolve/reject` after the first call is a no-op.
- **Microtasks before macrotasks:** `.then` callbacks run before `setTimeout` callbacks, even at delay 0.
- **`Number.isNaN` vs `isNaN`:** `Number.isNaN` does no coercion — only `NaN` returns `true`. `isNaN('abc')` returns `true`; `Number.isNaN('abc')` returns `false`.
- **`Object.assign` is shallow:** Nested objects are copied by reference, not cloned.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
