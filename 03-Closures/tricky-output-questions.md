# Closures — Tricky Output Questions

## Table of Contents

1. [Basic Closure Questions](#1-basic-closure-questions)
2. [Counter and State Questions](#2-counter-and-state-questions)
3. [Closure in Loop Questions](#3-closure-in-loop-questions)
4. [setTimeout with Closure Questions](#4-settimeout-with-closure-questions)
5. [Factory Function Questions](#5-factory-function-questions)
6. [Nested Closure Questions](#6-nested-closure-questions)
7. [Stale Closure Questions](#7-stale-closure-questions)
8. [Advanced Closure Questions](#8-advanced-closure-questions)

---

## 1. Basic Closure Questions

### Q1. What will be the output?

```js
function outer() {
  let x = 10;

  function inner() {
    console.log(x);
  }

  x = 20;
  return inner;
}

const fn = outer();
fn();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
20
```

### Explanation
The closure captures the **variable** `x` (the binding), not its value at the time `inner` is defined. When `x` is updated to `20` before `inner` is returned, the closure sees the latest value.

</details>

---

### Q2. What will be the output?

```js
function makeGreeter(greeting) {
  return function (name) {
    console.log(greeting + ", " + name + "!");
  };
}

const hello = makeGreeter("Hello");
const hi    = makeGreeter("Hi");

hello("Alice");
hi("Bob");
hello("Charlie");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Hello, Alice!
Hi, Bob!
Hello, Charlie!
```

### Explanation
Each call to `makeGreeter` creates a new closure with its own independent `greeting` variable. `hello` always has `"Hello"` and `hi` always has `"Hi"`.

</details>

---

### Q3. What will be the output?

```js
let count = 0;

function increment() {
  count++;
}

function getCount() {
  return count;
}

increment();
increment();
console.log(getCount());

count = 100;
console.log(getCount());
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
2
100
```

### Explanation
Both `increment` and `getCount` close over the module-level `count` variable. When `count` is directly reassigned to `100`, `getCount` immediately sees the updated value because closures capture the binding.

</details>

---

### Q4. What will be the output?

```js
function secret() {
  let msg = "hidden";

  return {
    get: function () { return msg; },
    set: function (val) { msg = val; },
  };
}

const s = secret();
console.log(s.get());
s.set("revealed");
console.log(s.get());
console.log(s.msg);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
hidden
revealed
undefined
```

### Explanation
`msg` is private — it lives inside `secret()`'s scope. Only `get` and `set` have closure access to it. `s.msg` is `undefined` because `msg` is not a property on the returned object.

</details>

---

### Q5. What will be the output?

```js
function a() {
  let n = 0;

  function b() {
    n++;
    return n;
  }

  return b;
}

const x = a();
const y = a();

console.log(x()); // ?
console.log(x()); // ?
console.log(y()); // ?
console.log(x()); // ?
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
1
2
1
3
```

### Explanation
`x` and `y` are independent closures from separate `a()` calls — each has its own `n`. `x()` increments its own `n` (0→1, 1→2, 2→3), while `y()` has a completely separate `n` that starts at 0 (→1).

</details>

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 2. Counter and State Questions

### Q6. What will be the output?

```js
function counter() {
  let count = 0;

  return function () {
    return count++;
  };
}

const c = counter();
console.log(c()); // ?
console.log(c()); // ?
console.log(c()); // ?
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
0
1
2
```

### Explanation
`count++` is post-increment — it returns the **current** value, then increments. First call returns `0` (then count becomes 1), second returns `1` (then count becomes 2), third returns `2`.

</details>

---

### Q7. What will be the output?

```js
function counter() {
  let count = 0;

  return function () {
    return ++count;
  };
}

const c = counter();
console.log(c()); // ?
console.log(c()); // ?
console.log(c()); // ?
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
1
2
3
```

### Explanation
`++count` is pre-increment — it increments **first**, then returns the new value. Compare with Q6 where post-increment returns the value before incrementing.

</details>

---

### Q8. What will be the output?

```js
const makeCounter = (function () {
  let count = 0;

  return function () {
    count++;
    return count;
  };
})();

console.log(makeCounter()); // ?
console.log(makeCounter()); // ?
console.log(makeCounter()); // ?
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
1
2
3
```

### Explanation
The IIFE executes immediately, creating a single `count` variable in memory. `makeCounter` is the returned function. Each call increments and returns `count`. There is only ever one `count` — no way to create a second independent counter from this.

</details>

---

### Q9. What will be the output?

```js
function makeCounter() {
  let count = 0;

  return {
    inc: () => ++count,
    dec: () => --count,
    val: () => count,
  };
}

const c1 = makeCounter();
const c2 = makeCounter();

c1.inc(); c1.inc(); c1.inc();
c2.inc();
c1.dec();

console.log(c1.val()); // ?
console.log(c2.val()); // ?
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
2
1
```

### Explanation
`c1` and `c2` are completely independent. `c1`: 0 → +1 → +1 → +1 → -1 = **2**. `c2`: 0 → +1 = **1**.

</details>

---

### Q10. What will be the output?

```js
function createStack() {
  const items = [];

  return {
    push: (item) => items.push(item),
    pop:  ()     => items.pop(),
    peek: ()     => items[items.length - 1],
    size: ()     => items.length,
  };
}

const stack = createStack();
stack.push(1);
stack.push(2);
stack.push(3);

console.log(stack.peek()); // ?
console.log(stack.pop());  // ?
console.log(stack.size()); // ?
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
3
3
2
```

### Explanation
`peek()` returns the top element (`3`) without removing it. `pop()` removes and returns the top element (`3`). After the pop, only `[1, 2]` remain, so `size()` is `2`.

</details>

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 3. Closure in Loop Questions

### Q11. What will be the output?

```js
var funcs = [];

for (var i = 0; i < 3; i++) {
  funcs.push(function () { return i; });
}

console.log(funcs[0]()); // ?
console.log(funcs[1]()); // ?
console.log(funcs[2]()); // ?
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
All three functions close over the **same** `i` (because `var` is function-scoped, not block-scoped). After the loop finishes, `i` is `3`. All three functions return `3`.

</details>

---

### Q12. What will be the output?

```js
var funcs = [];

for (let i = 0; i < 3; i++) {
  funcs.push(function () { return i; });
}

console.log(funcs[0]()); // ?
console.log(funcs[1]()); // ?
console.log(funcs[2]()); // ?
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
0
1
2
```

### Explanation
`let` creates a new binding for each iteration of a `for` loop. Each closure captures its own independent `i`.

</details>

---

### Q13. What will be the output?

```js
var funcs = [];

for (var i = 0; i < 3; i++) {
  funcs.push(
    (function (j) {
      return function () { return j; };
    })(i)
  );
}

console.log(funcs[0]()); // ?
console.log(funcs[1]()); // ?
console.log(funcs[2]()); // ?
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
0
1
2
```

### Explanation
The IIFE creates a new scope for each iteration. The current value of `i` is passed in as `j` and each returned function closes over its own `j`.

</details>

---

### Q14. What will be the output?

```js
const arr = [10, 20, 30];
const fns  = [];

arr.forEach(function (val) {
  fns.push(function () { return val; });
});

console.log(fns[0]()); // ?
console.log(fns[1]()); // ?
console.log(fns[2]()); // ?
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
10
20
30
```

### Explanation
`forEach` creates a new function scope for each callback invocation. Each pushed function closes over its own `val` parameter — no shared state issue.

</details>

---

### Q15. What will be the output?

```js
for (var i = 0; i < 3; i++) {
  var log = function () { console.log(i); };
}

log(); // ?
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
3
```

### Explanation
`var log` is hoisted to the enclosing function/global scope. It is reassigned on each iteration, so only the last assigned version survives the loop. That function closes over `var i`, which is `3` after the loop completes.

</details>

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 4. setTimeout with Closure Questions

### Q16. What will be the output?

```js
for (var i = 1; i <= 3; i++) {
  setTimeout(function () {
    console.log(i);
  }, i * 1000);
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
4
4
4
```

### Explanation
`var i` is shared across all iterations. By the time the callbacks fire (after 1s, 2s, 3s), the loop has completed and `i` is `4`. All three callbacks print `4`.

</details>

---

### Q17. What will be the output?

```js
for (let i = 1; i <= 3; i++) {
  setTimeout(function () {
    console.log(i);
  }, i * 1000);
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
1
2
3
```

### Explanation
`let` creates a new `i` binding per iteration. Each `setTimeout` callback closes over its own independent `i`.

</details>

---

### Q18. What will be the output?

```js
function doLater(val) {
  setTimeout(function () {
    console.log(val);
  }, 1000);
}

for (var i = 0; i < 3; i++) {
  doLater(i);
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
0
1
2
```

### Explanation
Passing `i` to `doLater` as a parameter creates a new scope for each call — `val` is a separate binding that captures the current value of `i` at call time. This is the function-parameter fix for the `var` loop problem.

</details>

---

### Q19. What will be the output?

```js
let x = 1;

setTimeout(function () {
  console.log(x); // ?
}, 0);

x = 2;
console.log(x); // ?
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
2
2
```

### Explanation
The synchronous `console.log(x)` runs first, printing `2`. The `setTimeout` callback is queued in the event loop and runs after all synchronous code completes. By then, `x` is already `2`, so it also prints `2`.

</details>

---

### Q20. What will be the output?

```js
function makeTimer(label) {
  let count = 0;

  setInterval(function () {
    count++;
    if (count <= 3) console.log(label + ": " + count);
  }, 100);
}

makeTimer("A");
makeTimer("B");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
A: 1
B: 1
A: 2
B: 2
A: 3
B: 3
```

### Explanation
Each `makeTimer` call creates an independent closure with its own `label` and `count`. The two intervals interleave, so "A" and "B" alternate. (Exact interleaving depends on the event loop but they fire at ~the same 100ms interval.)

</details>

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 5. Factory Function Questions

### Q21. What will be the output?

```js
function power(exp) {
  return function (base) {
    return Math.pow(base, exp);
  };
}

const square = power(2);
const cube   = power(3);

console.log(square(4));        // ?
console.log(cube(3));          // ?
console.log(square(cube(2)));  // ?
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
16
27
64
```

### Explanation
`square` closes over `exp = 2`. `cube` closes over `exp = 3`. `cube(2) = 8`, then `square(8) = 64`.

</details>

---

### Q22. What will be the output?

```js
function makeAdder(x) {
  return function (y) {
    return x + y;
  };
}

const add5  = makeAdder(5);
const add10 = makeAdder(10);

console.log(add5(3));         // ?
console.log(add10(3));        // ?
console.log(add5(add10(1)));  // ?
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
8
13
16
```

### Explanation
`add10(1) = 11`, then `add5(11) = 16`.

</details>

---

### Q23. What will be the output?

```js
function createLogger(prefix) {
  let logCount = 0;

  return function (msg) {
    logCount++;
    console.log(`[${prefix}][${logCount}] ${msg}`);
  };
}

const errorLog = createLogger("ERROR");
const infoLog  = createLogger("INFO");

errorLog("disk full");
infoLog("server started");
errorLog("timeout");
infoLog("connected");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
[ERROR][1] disk full
[INFO][1] server started
[ERROR][2] timeout
[INFO][2] connected
```

### Explanation
`errorLog` and `infoLog` are independent closures — each has its own `prefix` and its own `logCount` that starts at 0.

</details>

---

### Q24. What will be the output?

```js
function multiplier(factor) {
  return (number) => number * factor;
}

const ops = [1, 2, 3].map(multiplier);

console.log(ops[0](10)); // ?
console.log(ops[1](10)); // ?
console.log(ops[2](10)); // ?
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
10
20
30
```

### Explanation
`Array.prototype.map` passes each element as the first argument to `multiplier`. `ops[0]` closes over `factor = 1`, `ops[1]` over `2`, `ops[2]` over `3`.

</details>

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 6. Nested Closure Questions

### Q25. What will be the output?

```js
function a() {
  let x = 1;

  return function b() {
    let y = 2;

    return function c() {
      let z = 3;
      return x + y + z;
    };
  };
}

console.log(a()()()); // ?
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
6
```

### Explanation
`c` closes over `z = 3` (its own), `y = 2` from `b`, and `x = 1` from `a`. The sum is `1 + 2 + 3 = 6`.

</details>

---

### Q26. What will be the output?

```js
function outer() {
  let x = 10;

  function middle() {
    let y = 20;

    function inner() {
      console.log(x + y);
    }

    x = 30;
    return inner;
  }

  return middle;
}

outer()()();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
50
```

### Explanation
`outer()` returns `middle`. `outer()()` calls `middle`, which reassigns `x = 30` and returns `inner`. `outer()()()` calls `inner`, which reads `x = 30` and `y = 20`, printing `50`.

</details>

---

### Q27. What will be the output?

```js
function createCounter() {
  let n = 0;

  function increment() {
    n++;
    return n;
  }

  function reset() {
    const prev = n;
    n = 0;
    return prev;
  }

  return { increment, reset };
}

const { increment, reset } = createCounter();

console.log(increment()); // ?
console.log(increment()); // ?
console.log(reset());     // ?
console.log(increment()); // ?
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
1
2
2
1
```

### Explanation
`increment` and `reset` share the same `n`. After two increments `n = 2`. `reset()` saves `prev = 2`, sets `n = 0`, and returns `2`. The next `increment()` starts from `0`, returning `1`.

</details>

---

### Q28. What will be the output?

```js
function makeOps() {
  let n = 0;

  const add = (x) => { n += x; return api; };
  const sub = (x) => { n -= x; return api; };
  const val = ()  => n;

  const api = { add, sub, val };
  return api;
}

const r = makeOps().add(5).add(3).sub(2).val();
console.log(r); // ?
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
6
```

### Explanation
This is a fluent/chainable API. Each `add`/`sub` call mutates `n` and returns `api` for chaining. `0 + 5 + 3 - 2 = 6`.

</details>

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 7. Stale Closure Questions

### Q29. What will be the output?

```js
function makeIncrementer() {
  let count = 0;

  const increment = () => {
    count++;
    return count;
  };

  const double = () => count * 2;

  count = 10; // set before returning

  return { increment, double };
}

const { increment, double } = makeIncrementer();

console.log(double());    // ?
console.log(increment()); // ?
console.log(double());    // ?
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
20
11
22
```

### Explanation
`count` is set to `10` before the object is returned. `double()` returns `10 * 2 = 20`. `increment()` increments `count` to `11` and returns `11`. `double()` then returns `11 * 2 = 22`.

</details>

---

### Q30. What will be the output?

```js
function createFunctions() {
  const result = [];
  let i = 0;

  while (i < 3) {
    result.push(function () { return i; });
    i++;
  }

  return result;
}

const fns = createFunctions();
console.log(fns[0]()); // ?
console.log(fns[1]()); // ?
console.log(fns[2]()); // ?
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
`i` is declared with `let` **outside** the `while` loop — there is only one `i` in the function scope, not per-iteration bindings. All closures share the same `i`. After the loop, `i = 3`, so all return `3`. (A `for` loop with `let` would create per-iteration bindings; a `while` loop does not.)

</details>

---

### Q31. What will be the output?

```js
let value = "original";

const getVal = function () {
  return value;
};

value = "updated";
console.log(getVal()); // ?
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
updated
```

### Explanation
`getVal` closes over the `value` variable. By the time `getVal()` is called, `value` has been reassigned to `"updated"`. Closures capture variable bindings, not values at definition time.

</details>

---

### Q32. What will be the output?

```js
function setup() {
  let data = { count: 0 };

  const increase  = () => { data.count++; };
  const replace   = () => { data = { count: 100 }; };
  const getCount  = () => data.count;

  return { increase, replace, getCount };
}

const obj = setup();
obj.increase();
obj.increase();
console.log(obj.getCount()); // ?
obj.replace();
console.log(obj.getCount()); // ?
obj.increase();
console.log(obj.getCount()); // ?
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
2
100
101
```

### Explanation
All methods close over the `data` variable (not the object it points to). After two increases, `data.count = 2`. `replace()` reassigns `data` to a brand-new object `{ count: 100 }`. All methods now reference this new object. `increase()` increments the new object's count to `101`.

</details>

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 8. Advanced Closure Questions

### Q33. What will be the output?

```js
function once(fn) {
  let done = false;
  let result;

  return function (...args) {
    if (!done) {
      done = true;
      result = fn(...args);
    }
    return result;
  };
}

const initOnce = once(function (x) {
  console.log("init with", x);
  return x * 2;
});

console.log(initOnce(5));  // ?
console.log(initOnce(10)); // ?
console.log(initOnce(20)); // ?
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
init with 5
10
10
10
```

### Explanation
Only the first call executes `fn`, logs `"init with 5"`, and caches `result = 10`. Subsequent calls skip `fn` and return the cached `result`.

</details>

---

### Q34. What will be the output?

```js
function memoize(fn) {
  const cache = {};

  return function (n) {
    if (n in cache) return cache[n];
    cache[n] = fn(n);
    return cache[n];
  };
}

let callCount = 0;

const factorial = memoize(function f(n) {
  callCount++;
  return n <= 1 ? 1 : n * factorial(n - 1);
});

console.log(factorial(4)); // ?
console.log(factorial(4)); // ?
console.log(callCount);    // ?
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
24
24
4
```

### Explanation
The first `factorial(4)` recursively calls `factorial(3)`, `factorial(2)`, `factorial(1)` — 4 total calls, so `callCount = 4`, result cached as `24`. The second call hits the cache immediately. `callCount` stays at `4`.

</details>

---

### Q35. What will be the output?

```js
function compose(...fns) {
  return (x) => fns.reduceRight((v, f) => f(v), x);
}

const add1   = x => x + 1;
const times2 = x => x * 2;
const minus3 = x => x - 3;

const transform = compose(minus3, times2, add1);
console.log(transform(5)); // ?
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
9
```

### Explanation
`compose` applies functions right-to-left: `add1(5) = 6`, `times2(6) = 12`, `minus3(12) = 9`.

</details>

---

### Q36. What will be the output?

```js
function partial(fn, ...args) {
  return function (...moreArgs) {
    return fn(...args, ...moreArgs);
  };
}

function greet(greeting, name, punctuation) {
  return `${greeting}, ${name}${punctuation}`;
}

const sayHello        = partial(greet, "Hello");
const sayHelloToAlice = partial(sayHello, "Alice");

console.log(sayHello("Bob", "!"));   // ?
console.log(sayHelloToAlice("?"));   // ?
console.log(sayHelloToAlice("!!!")); // ?
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Hello, Bob!
Hello, Alice?
Hello, Alice!!!
```

### Explanation
`sayHello` has `greeting = "Hello"` preset. `sayHelloToAlice` further presets `name = "Alice"` (by partially applying `sayHello`). Only `punctuation` is provided at each call.

</details>

---

### Q37. What will be the output?

```js
function createAccumulator(initial) {
  let total = initial;

  return function (value) {
    total += value;
    return total;
  };
}

const acc = createAccumulator(10);
console.log(acc(5));   // ?
console.log(acc(3));   // ?
console.log(acc(-8));  // ?
console.log(acc(0));   // ?
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
15
18
10
10
```

### Explanation
`total` starts at `10`. 10+5=15, 15+3=18, 18-8=10, 10+0=10. Each call returns the running total.

</details>

---

### Q38. What will be the output?

```js
function tricky() {
  const fns = [];

  for (let i = 0; i < 3; i++) {
    const j = i;
    fns.push(() => {
      j; // referenced but not returned
      return i;
    });
  }

  return fns;
}

const fns = tricky();
console.log(fns[0]()); // ?
console.log(fns[1]()); // ?
console.log(fns[2]()); // ?
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
0
1
2
```

### Explanation
`let` in a `for` loop creates per-iteration bindings for `i`. Each closure closes over its own `i` (0, 1, 2). The `j` variable is defined per iteration but the returned value is `i`, not `j`. The output is still `0`, `1`, `2`.

</details>

---

### Q39. What will be the output?

```js
function outer() {
  var x = 1;

  return {
    get: function () { return x; },
    set: function (v) { x = v; },
    inc: function () { x++; },
  };
}

const obj = outer();
console.log(obj.get()); // ?
obj.inc();
obj.inc();
console.log(obj.get()); // ?
obj.set(10);
obj.inc();
console.log(obj.get()); // ?
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
1
3
11
```

### Explanation
All three methods share the same `x`. Starting at `1`, two `inc()` calls make `x = 3`. `set(10)` makes `x = 10`. One more `inc()` makes `x = 11`.

</details>

---

### Q40. What will be the output?

```js
function makeMultiplierChain() {
  let result = 1;

  const api = {
    by: function (factor) {
      result *= factor;
      return api;
    },
    value: function () {
      return result;
    },
  };

  return api;
}

const answer = makeMultiplierChain()
  .by(2)
  .by(3)
  .by(5)
  .value();

console.log(answer); // ?
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
30
```

### Explanation
A chainable closure API. `result` starts at `1`. `.by(2)` → 2, `.by(3)` → 6, `.by(5)` → 30. `.value()` returns `30`.

</details>

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## Final Tips

- Closures capture **variable bindings**, not values — the closure always sees the current value of the variable, not what it held when the function was defined.
- `var` in a loop shares one binding across all iterations; `let` in a `for` loop creates a new binding per iteration; `let` outside a `while` loop does not.
- Every `setTimeout` / `setInterval` callback, event listener, and React `useEffect` callback is a closure — always ask "what does this close over?"
- Stale closures happen when a function captures an old binding that is no longer updated — most common in React hooks with empty dependency arrays.
- The `once`, `memoize`, `partial`, `curry`, and `compose` patterns are all built on closures — expect them to appear in senior-level interviews.
- If a closure holds a large object, that object stays in memory. Null out references when done.
- Two closures from the **same factory call** share state; closures from **separate factory calls** have independent state.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
