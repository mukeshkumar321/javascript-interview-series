# Functions — Tricky Output Questions

## Table of Contents
1. [Function Declaration vs Expression Questions](#1-function-declaration-vs-expression-questions)
2. [IIFE Questions](#2-iife-questions)
3. [Arrow vs Regular Function Questions](#3-arrow-vs-regular-function-questions)
4. [Default and Rest Parameter Questions](#4-default-and-rest-parameter-questions)
5. [Arguments Object Questions](#5-arguments-object-questions)
6. [Currying Questions](#6-currying-questions)
7. [Generator Function Questions](#7-generator-function-questions)
8. [Advanced Function Questions](#8-advanced-function-questions)

---

## 1. Function Declaration vs Expression Questions

---

### Q1. What will be the output?

```js
console.log(foo());
console.log(bar());

function foo() {
  return "foo called";
}

var bar = function() {
  return "bar called";
};
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
foo called
TypeError: bar is not a function
```

### Explanation
`foo` is a function declaration and is **fully hoisted** — both its variable and its definition are available before the code runs. `bar` is declared with `var`, so the variable `bar` is hoisted and initialized to `undefined`. When `bar()` is called before the assignment, JavaScript tries to call `undefined` as a function, causing a `TypeError`.

</details>

---

### Q2. What will be the output?

```js
function test() {
  console.log(a);
  console.log(b());

  var a = 1;
  function b() {
    return "b";
  }
}

test();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
undefined
b
```

### Explanation
Inside `test`, the inner function declaration `b` is fully hoisted. The `var a` declaration is hoisted but not its assignment, so `a` is `undefined` when logged. `b()` is a function declaration, so it is hoisted fully and can be called before its position in source code.

</details>

---

### Q3. What will be the output?

```js
var x = 1;
function x() {}
console.log(typeof x);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
number
```

### Explanation
Function declarations are hoisted before `var` declarations. During hoisting, `x` becomes the function. Then during execution, `var x = 1` runs and overwrites `x` with the number `1`. By the time `typeof x` runs, `x` holds `1`, so the type is `"number"`.

</details>

---

### Q4. What will be the output?

```js
console.log(typeof foo);
console.log(typeof bar);

var foo = function bar() {};
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
undefined
undefined
```

### Explanation
`foo` is declared with `var`, so it is hoisted as `undefined`. `bar` is the internal name of a named function expression — it is **not** added to the outer scope, so `typeof bar` is `"undefined"` (not a `ReferenceError` because `typeof` is safe for undeclared variables). Neither has been assigned at the point of the `console.log` calls.

</details>

---

## 2. IIFE Questions

---

### Q1. What will be the output?

```js
(function() {
  var x = 10;
  console.log(x);
})();

console.log(typeof x);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
10
undefined
```

### Explanation
The IIFE creates its own scope. `var x` inside the IIFE is local to that scope, so it is not accessible outside. Outside the IIFE, `x` is not declared at all. `typeof x` returns `"undefined"` (instead of throwing a `ReferenceError`) because `typeof` on an undeclared variable is safe.

</details>

---

### Q2. What will be the output?

```js
var result = (function(n) {
  return n * n;
})(5);

console.log(result);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
25
```

### Explanation
The IIFE is called immediately with argument `5`. It returns `5 * 5 = 25`, which is assigned to `result`.

</details>

---

### Q3. What will be the output?

```js
(function() {
  console.log(1);
  setTimeout(function() { console.log(2); }, 0);
  console.log(3);
})();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
1
3
2
```

### Explanation
The IIFE runs synchronously. `1` and `3` are logged in order. The `setTimeout` callback is placed on the event queue even with a delay of `0`, so it runs after the current call stack is empty — after `3` is logged.

</details>

---

### Q4. What will be the output?

```js
var i = 10;
(function() {
  var i = 20;
  console.log(i);
})();
console.log(i);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
20
10
```

### Explanation
The `var i = 20` inside the IIFE creates a new `i` that shadows the outer `i = 10`. The IIFE logs its local `i` (20). After the IIFE runs, the outer `i` (10) is logged.

</details>

---

## 3. Arrow vs Regular Function Questions

---

### Q1. What will be the output?

```js
const obj = {
  name: "Alice",
  greet: function() {
    console.log(this.name);
  },
  greetArrow: () => {
    console.log(this.name);
  }
};

obj.greet();
obj.greetArrow();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Alice
undefined
```

### Explanation
`greet` is a regular function. When called as `obj.greet()`, `this` is `obj`, so `this.name` is `"Alice"`. `greetArrow` is an arrow function. Arrow functions do not have their own `this` — they inherit `this` from where they were **defined**, which is the enclosing scope (the module/global scope, not `obj`). In non-strict global scope, `this.name` is `undefined`.

</details>

---

### Q2. What will be the output?

```js
function Timer() {
  this.seconds = 0;
  setInterval(function() {
    this.seconds++;
    console.log(this.seconds);
  }, 1000);
}

// Consider just the concept — what would the first tick print?
const t = new Timer();
// After 1 second...
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
NaN
```

### Explanation
Inside `setInterval`, the callback is a regular function. When it fires, `this` is the global object (or `undefined` in strict mode), not the `Timer` instance. `this.seconds` on the global object is `undefined`, and `undefined++` is `NaN`. Using an arrow function (`() => { this.seconds++; }`) would fix this because it captures `this` from the `Timer` constructor.

</details>

---

### Q3. What will be the output?

```js
const arrowFn = () => arguments[0];

function wrapper() {
  const inner = () => arguments[0];
  return inner();
}

console.log(wrapper(42));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
42
```

### Explanation
Arrow functions do not have their own `arguments` object. Inside `wrapper`, `inner` is an arrow function, so its `arguments` refers to `wrapper`'s `arguments` object. `wrapper(42)` sets `arguments[0]` to `42`, which `inner` captures and returns.

Note: calling `arrowFn()` alone would throw a `ReferenceError` because there is no enclosing non-arrow function providing `arguments`.

</details>

---

### Q4. What will be the output?

```js
const fn = () => {
  return {
    value: 10
  };
};

const brokenFn = () => {
  value: 10
};

console.log(fn());
console.log(brokenFn());
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
{ value: 10 }
undefined
```

### Explanation
`fn` uses an explicit `return` inside braces and returns the object correctly. `brokenFn` uses `{}` as a block statement (not an object literal) — the `value: 10` is interpreted as a label `value` and the expression `10` (which is discarded). The function returns `undefined` implicitly. To return an object literal concisely from an arrow function, wrap the object in parentheses: `() => ({ value: 10 })`.

</details>

---

## 4. Default and Rest Parameter Questions

---

### Q1. What will be the output?

```js
function test(a = 1, b = a * 2) {
  console.log(a, b);
}

test();
test(5);
test(5, 3);
test(undefined, 3);
test(null, 3);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
1 2
5 10
5 3
1 3
null 3
```

### Explanation
- `test()`: `a` defaults to `1`, `b` defaults to `a * 2 = 2`.
- `test(5)`: `a = 5`, `b` defaults to `5 * 2 = 10`.
- `test(5, 3)`: both provided; `a = 5`, `b = 3`.
- `test(undefined, 3)`: `undefined` triggers the default for `a` → `a = 1`; `b = 3` is explicit.
- `test(null, 3)`: `null` does NOT trigger the default; `a = null`, `b = 3`.

</details>

---

### Q2. What will be the output?

```js
function sum(first, ...rest) {
  console.log(first);
  console.log(rest);
  console.log(rest.length);
}

sum(10, 20, 30, 40);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
10
[20, 30, 40]
3
```

### Explanation
`first` captures the first argument `10`. `...rest` collects all remaining arguments into a real array `[20, 30, 40]`. Unlike the `arguments` object, `rest` is a proper Array so it has `.length` and array methods.

</details>

---

### Q3. What will be the output?

```js
function makeCounter(start = 0, step = 1) {
  return () => (start += step);
}

const counter = makeCounter(10, 5);
console.log(counter());
console.log(counter());
console.log(counter());
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
15
20
25
```

### Explanation
`makeCounter` returns an arrow function (a closure) that increments `start` by `step` each time it is called. Initial `start = 10`, `step = 5`. Each call adds 5 to `start` and returns the new value: 15, 20, 25.

</details>

---

### Q4. What will be the output?

```js
function greet(name = "World") {
  console.log(arguments.length);
  console.log(name);
}

greet();
greet(undefined);
greet("Alice");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
0
World
1
World
1
Alice
```

### Explanation
`arguments.length` reflects the actual number of arguments passed to the function, not the number of parameters with defaults. `greet()` passes 0 arguments, `greet(undefined)` passes 1 argument (even though it triggers the default), and `greet("Alice")` passes 1 argument. The `name` defaults to `"World"` when `undefined` is passed.

</details>

---

## 5. Arguments Object Questions

---

### Q1. What will be the output?

```js
function test() {
  console.log(Array.isArray(arguments));
  console.log(arguments.length);
  console.log(arguments[1]);
}

test("a", "b", "c");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
false
3
b
```

### Explanation
`arguments` is an array-like object but NOT a real Array. `Array.isArray(arguments)` returns `false`. It has a `length` property (3 for 3 arguments) and numeric indices (`arguments[1]` is `"b"`), but it does not have array methods like `map`, `filter`, etc.

</details>

---

### Q2. What will be the output?

```js
function modify(a, b) {
  arguments[0] = 99;
  console.log(a);
  a = 55;
  console.log(arguments[0]);
}

modify(1, 2);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
99
55
```

### Explanation
In non-strict mode, named parameters and the `arguments` object are **linked** (they share the same memory slot). Changing `arguments[0]` to `99` also changes `a`. Changing `a` to `55` also changes `arguments[0]`. This two-way binding does not apply in strict mode or with default/rest parameters.

</details>

---

### Q3. What will be the output?

```js
"use strict";

function modify(a, b) {
  arguments[0] = 99;
  console.log(a);
  a = 55;
  console.log(arguments[0]);
}

modify(1, 2);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
1
99
```

### Explanation
In strict mode, the link between named parameters and the `arguments` object is broken. They become independent. Changing `arguments[0]` does NOT change `a`. Changing `a` does NOT change `arguments[0]`.

</details>

---

### Q4. What will be the output?

```js
function outer(x) {
  return () => arguments[0] * 2;
}

const fn = outer(7);
console.log(fn());
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
14
```

### Explanation
The arrow function inside `outer` does not have its own `arguments`. It inherits `arguments` from the enclosing `outer` function. `outer` was called with `7`, so `arguments[0]` is `7`. The arrow function returns `7 * 2 = 14`.

</details>

---

## 6. Currying Questions

---

### Q1. What will be the output?

```js
function add(a) {
  return function(b) {
    return function(c) {
      return a + b + c;
    };
  };
}

console.log(add(1)(2)(3));
console.log(add(1)(2));
console.log(typeof add(1)(2));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
6
[Function (anonymous)]
function
```

### Explanation
`add(1)` returns a function. `add(1)(2)` returns another function. Only when all three arguments are provided does the final computation happen: `1 + 2 + 3 = 6`. Calling only two levels deep returns the innermost function (not a number), which is why `typeof add(1)(2)` is `"function"`.

</details>

---

### Q2. What will be the output?

```js
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn(...args);
    }
    return (...more) => curried(...args, ...more);
  };
}

const add = curry((a, b, c) => a + b + c);

console.log(add(1)(2)(3));
console.log(add(1, 2)(3));
console.log(add(1)(2, 3));
console.log(add(1, 2, 3));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
6
6
6
6
```

### Explanation
The generic `curry` utility tracks accumulated arguments. It invokes the original function when `args.length >= fn.length` (3 for this `add`). All four call styles accumulate to 3 arguments and produce the same result: `1 + 2 + 3 = 6`.

</details>

---

### Q3. What will be the output?

```js
const multiply = a => b => a * b;

const double = multiply(2);
const triple = multiply(3);

console.log(double(5));
console.log(triple(5));
console.log(multiply(4)(5));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
10
15
20
```

### Explanation
`multiply` is a curried function written with arrow syntax. `double` is a closure over `a = 2`, and `triple` is a closure over `a = 3`. Each time the returned function is called, it multiplies `b` by its captured `a`. `multiply(4)(5)` is called without saving the intermediate function.

</details>

---

## 7. Generator Function Questions

---

### Q1. What will be the output?

```js
function* gen() {
  console.log("start");
  yield 1;
  console.log("middle");
  yield 2;
  console.log("end");
}

const g = gen();
console.log(g.next());
console.log(g.next());
console.log(g.next());
console.log(g.next());
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
start
{ value: 1, done: false }
middle
{ value: 2, done: false }
end
{ value: undefined, done: true }
{ value: undefined, done: true }
```

### Explanation
Calling `gen()` does not run the function body — it returns a generator object. Each call to `.next()` runs the function until the next `yield`. The first `.next()` runs until `yield 1` (printing "start") and returns `{ value: 1, done: false }`. The second `.next()` runs until `yield 2` (printing "middle") and returns `{ value: 2, done: false }`. The third `.next()` runs to the end (printing "end") and returns `{ value: undefined, done: true }`. Further calls return `{ value: undefined, done: true }`.

</details>

---

### Q2. What will be the output?

```js
function* counter() {
  let i = 0;
  while (true) {
    const reset = yield i;
    if (reset) {
      i = 0;
    } else {
      i++;
    }
  }
}

const c = counter();
console.log(c.next().value);    // 0
console.log(c.next().value);    // 1
console.log(c.next().value);    // 2
console.log(c.next(true).value); // reset!
console.log(c.next().value);    // 1
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
0
1
2
0
1
```

### Explanation
`yield i` pauses the generator and sends `i` out. The value passed to `.next(value)` becomes the result of the `yield` expression. `c.next()` with no argument means `reset` is `undefined` (falsy), so `i` increments. `c.next(true)` sends `true` as the value of `yield`, so `reset` is `true` and `i` resets to `0`. The next `.next()` increments `i` to `1`.

</details>

---

### Q3. What will be the output?

```js
function* take(n, iterable) {
  let count = 0;
  for (const item of iterable) {
    if (count >= n) return;
    yield item;
    count++;
  }
}

console.log([...take(3, [10, 20, 30, 40, 50])]);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
[10, 20, 30]
```

### Explanation
The `take` generator yields items from the iterable until `count >= n`. For `n = 3`, it yields `10`, `20`, and `30`, then returns (stopping iteration). The spread operator `[...take(...)]` collects all yielded values into an array.

</details>

---

### Q4. What will be the output?

```js
function* gen() {
  return yield yield 1;
}

const g = gen();
console.log(g.next());
console.log(g.next("a"));
console.log(g.next("b"));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
{ value: 1, done: false }
{ value: 'a', done: false }
{ value: 'b', done: true }
```

### Explanation
The generator has two `yield` expressions nested. First `.next()`: runs to the inner `yield 1`, sends `1` out → `{ value: 1, done: false }`. Second `.next("a")`: `"a"` becomes the result of `yield 1`, so the inner yield evaluates to `"a"`. Now the outer `yield "a"` pauses and sends `"a"` out → `{ value: 'a', done: false }`. Third `.next("b")`: `"b"` becomes the result of the outer yield. The `return` returns `"b"` → `{ value: 'b', done: true }`.

</details>

---

## 8. Advanced Function Questions

---

### Q1. What will be the output?

```js
function makeAdder(x) {
  return function(y) {
    return x + y;
  };
}

const add5 = makeAdder(5);
const add10 = makeAdder(10);

console.log(add5(3));
console.log(add10(3));
console.log(add5 === add10);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
8
13
false
```

### Explanation
Each call to `makeAdder` creates a new closure with its own `x`. `add5` closes over `x = 5` and `add10` closes over `x = 10`. They are different function objects in memory, so `add5 === add10` is `false`.

</details>

---

### Q2. What will be the output?

```js
function test(fn) {
  fn();
}

const obj = {
  name: "Alice",
  sayName: function() {
    console.log(this.name);
  }
};

test(obj.sayName);
test(obj.sayName.bind(obj));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
undefined
Alice
```

### Explanation
When `obj.sayName` is passed as a plain function reference to `test`, the method loses its `this` binding. Inside `test`, `fn()` is a plain function call, so `this` is the global object (or `undefined` in strict mode). `this.name` is `undefined` in a browser's global scope. `bind(obj)` returns a new function with `this` permanently set to `obj`, so the second call correctly logs `"Alice"`.

</details>

---

### Q3. What will be the output?

```js
const compose = (...fns) => x => fns.reduceRight((v, fn) => fn(v), x);

const add1  = x => x + 1;
const mul2  = x => x * 2;
const sub3  = x => x - 3;

const transform = compose(sub3, mul2, add1);
console.log(transform(5));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
9
```

### Explanation
`compose` applies functions right-to-left: `add1` first, then `mul2`, then `sub3`.
- `add1(5)` → `6`
- `mul2(6)` → `12`
- `sub3(12)` → `9`

</details>

---

### Q4. What will be the output?

```js
function memoize(fn) {
  const cache = {};
  return function(n) {
    if (n in cache) {
      return cache[n];
    }
    cache[n] = fn(n);
    return cache[n];
  };
}

let callCount = 0;

const square = memoize(function(n) {
  callCount++;
  return n * n;
});

console.log(square(4));
console.log(square(4));
console.log(square(5));
console.log(callCount);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
16
16
25
2
```

### Explanation
`square(4)` computes `4 * 4 = 16` and caches it (`callCount = 1`). `square(4)` again returns the cached `16` without calling the original function (`callCount` stays at 1). `square(5)` is a new input, so it computes `5 * 5 = 25` (`callCount = 2`). The original function was called only twice.

</details>

---

### Q5. What will be the output?

```js
function outer() {
  var x = 10;

  function inner() {
    console.log(x);
    var x = 20;
    console.log(x);
  }

  inner();
}

outer();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
undefined
20
```

### Explanation
Inside `inner`, `var x = 20` is hoisted to the top of the `inner` function scope. At the first `console.log(x)`, `x` exists in `inner`'s scope but has not been assigned yet, so it is `undefined` (not the outer `x = 10`, because `var x` in `inner` shadows the outer `x`). After the assignment, the second `console.log(x)` logs `20`.

</details>

---

### Q6. What will be the output?

```js
const fns = [];

for (var i = 0; i < 3; i++) {
  fns.push((function(j) {
    return function() { return j; };
  })(i));
}

console.log(fns[0]());
console.log(fns[1]());
console.log(fns[2]());
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
The IIFE `(function(j) { ... })(i)` immediately captures the current value of `i` into the parameter `j`. Each iteration creates a new closure over a distinct `j`. Without the IIFE (just `function() { return i; }` inside the loop with `var`), all three functions would return `3` because `var i` is function-scoped and shared. Using an IIFE (or `let` instead of `var`) captures the current value correctly.

</details>

---

## Final Tips

- **Declaration vs Expression**: When in doubt, remember declarations are fully hoisted; expressions with `let`/`const` are not.
- **Arrow function `this`**: Arrow functions are unsuitable as object methods because they do not have their own `this`.
- **`arguments` vs rest**: Prefer rest parameters in modern code. Remember that arrow functions inherit `arguments` from enclosing non-arrow functions.
- **Default parameter trap**: `undefined` triggers a default; `null` does not.
- **Generator execution**: The function body does not run until the first `.next()` call.
- **Currying vs Partial Application**: Currying always produces single-argument functions; partial application can fix multiple arguments at once.
- **Closure + loop**: When combining closures with `var` loops, use IIFE or `let` to capture the current iteration value.
- **Memoization**: Only cache results of pure functions — caching impure functions leads to stale data.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
