# Execution Context — Tricky Output Questions

## Table of Contents
1. [Call Stack Questions](#1-call-stack-questions)
2. [Memory Phase Questions](#2-memory-phase-questions)
3. [Execution Order Questions](#3-execution-order-questions)
4. [Stack Overflow Questions](#4-stack-overflow-questions)
5. [Nested Execution Questions](#5-nested-execution-questions)
6. [Advanced Execution Context Questions](#6-advanced-execution-context-questions)

---

## 1. Call Stack Questions

---

### Q1. What will be the output?

```js
function a() {
  console.log("a");
  b();
}

function b() {
  console.log("b");
  c();
}

function c() {
  console.log("c");
}

a();
console.log("done");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
a
b
c
done
```

### Explanation
`a()` is pushed to the call stack and logs `"a"`, then calls `b()`. `b` is pushed, logs `"b"`, then calls `c()`. `c` is pushed, logs `"c"`, then returns (popped). `b` resumes and returns (popped). `a` resumes and returns (popped). Finally, `console.log("done")` runs in the Global EC.

</details>

---

### Q2. What will be the output?

```js
function first() {
  console.log("first - start");
  second();
  console.log("first - end");
}

function second() {
  console.log("second - start");
  console.log("second - end");
}

console.log("global - start");
first();
console.log("global - end");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
global - start
first - start
second - start
second - end
first - end
global - end
```

### Explanation
Code runs synchronously and the call stack is LIFO. `global - start` runs in the Global EC, then `first` is pushed, logs `"first - start"`, calls `second`. `second` logs both its lines and returns (popped). `first` resumes, logs `"first - end"`, and returns. Back in Global EC, `global - end` runs.

</details>

---

### Q3. What will be the output?

```js
function foo() {
  return bar();
}

function bar() {
  return baz();
}

function baz() {
  return 42;
}

console.log(foo());
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
42
```

### Explanation
`foo` calls `bar`, which calls `baz`. `baz` returns `42` → `bar` returns `42` → `foo` returns `42`. Each EC is pushed and popped in sequence. `console.log` receives the final return value `42`.

</details>

---

### Q4. What will be the output?

```js
function x() {
  console.log("x");
  y();
  console.log("x again");
}

function y() {
  console.log("y");
}

x();
y();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
x
y
x again
y
```

### Explanation
`x()` is called first: logs `"x"`, then calls `y()` (logs `"y"`), then logs `"x again"`. After `x` returns, `y()` is called directly from the global EC and logs `"y"` again.

</details>

---

### Q5. What will be the output?

```js
function greet() {
  var name = "World";
  console.log("Hello, " + name);
}

greet();
console.log(typeof name);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Hello, World
undefined
```

### Explanation
`name` is declared inside the `greet` function EC. Once `greet` returns and its EC is destroyed, `name` is no longer accessible. `typeof name` in the Global EC returns `"undefined"` because `name` does not exist in the global scope (no `var name` at the top level).

</details>

---

## 2. Memory Phase Questions

---

### Q6. What will be the output?

```js
console.log(a);
var a = 5;
console.log(a);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
undefined
5
```

### Explanation
During the memory creation phase, `var a` is hoisted and initialized to `undefined`. The first `console.log(a)` runs before the assignment, so it sees `undefined`. After the code execution phase assigns `a = 5`, the second log prints `5`.

</details>

---

### Q7. What will be the output?

```js
console.log(typeof foo);
console.log(typeof bar);

function foo() {
  return 1;
}

var bar = function () {
  return 2;
};
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
function
undefined
```

### Explanation
During the memory phase: `foo` (function declaration) is fully hoisted as a function. `bar` (function expression assigned to `var`) is hoisted as `undefined` — only the `var bar` declaration is hoisted, not the assignment. `typeof foo` is `"function"`, `typeof bar` is `"undefined"`.

</details>

---

### Q8. What will be the output?

```js
let x = 1;
console.log(x);

{
  console.log(x);
  let x = 2;
  console.log(x);
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
1
// ReferenceError: Cannot access 'x' before initialization
```

### Explanation
Inside the block, `let x = 2` is hoisted to the top of the block but placed in the Temporal Dead Zone (TDZ) until the declaration line. The second `console.log(x)` — before `let x = 2` — accesses `x` while it is in the TDZ, throwing a `ReferenceError`. The block's `x` shadows the outer `x`.

</details>

---

### Q9. What will be the output?

```js
var x = 10;

function outer() {
  console.log(x);
  var x = 20;
  console.log(x);
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
Inside `outer`'s execution context, `var x = 20` is hoisted to the top of the function (not the global `x`). During the memory creation phase of `outer`'s EC, the local `x` is `undefined`. The first `console.log(x)` reads the local (hoisted but not yet assigned) `x` — `undefined`. Then `x = 20` is assigned, so the second log prints `20`.

</details>

---

### Q10. What will be the output?

```js
console.log(sum(2, 3));
console.log(multiply(2, 3));

function sum(a, b) {
  return a + b;
}

var multiply = function (a, b) {
  return a * b;
};
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
5
// TypeError: multiply is not a function
```

### Explanation
`sum` is a function declaration — fully hoisted with its body, so `sum(2, 3)` works and returns `5`. `multiply` is a `var` holding a function expression — only the `var multiply` is hoisted (as `undefined`). Calling `multiply(2, 3)` before the assignment is like calling `undefined(2, 3)`, throwing a `TypeError`.

</details>

---

### Q11. What will be the output?

```js
function test() {
  console.log(a, b, c);
  var a = 1;
  let b = 2;
  const c = 3;
}

test();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
// ReferenceError: Cannot access 'b' before initialization
```

### Explanation
`var a` is hoisted to `undefined`, so `a` would print `undefined`. However, `let b` and `const c` are in the TDZ when `console.log` executes. The engine throws a `ReferenceError` for `b` before even evaluating `c`.

</details>

---

## 3. Execution Order Questions

---

### Q12. What will be the output?

```js
var n = 1;

function fn() {
  var n = 2;
  console.log(n);
}

fn();
console.log(n);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
2
1
```

### Explanation
Each execution context has its own variable environment. Inside `fn`, `var n = 2` creates a local `n` in `fn`'s EC. The global `n = 1` is unaffected. `fn()` logs its local `2`. After `fn` returns, global `n` remains `1`.

</details>

---

### Q13. What will be the output?

```js
function outer() {
  var x = 10;

  function inner() {
    console.log(x);
  }

  x = 20;
  inner();
}

outer();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
20
```

### Explanation
`inner` closes over `outer`'s variable environment. It does not capture the **value** of `x` at the time `inner` is defined — it captures a **live reference** to the variable `x`. By the time `inner()` is called, `x` has been updated to `20`, so that is what is logged.

</details>

---

### Q14. What will be the output?

```js
for (var i = 0; i < 3; i++) {
  setTimeout(function () {
    console.log(i);
  }, 0);
}
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
`var i` is function-scoped (or global here), so all three `setTimeout` callbacks share a reference to the **same** `i` in the Global EC. The loop completes (i = 3) before any callback runs. When they execute, they all read `i = 3`.

</details>

---

### Q15. What will be the output?

```js
for (let i = 0; i < 3; i++) {
  setTimeout(function () {
    console.log(i);
  }, 0);
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
`let` is block-scoped. Each iteration of the `for` loop creates a **new lexical environment** with its own `i`. Each `setTimeout` callback closes over a different `i` binding. When they run, they each read their own saved value.

</details>

---

### Q16. What will be the output?

```js
function makeAdder(x) {
  return function (y) {
    return x + y;
  };
}

const add5 = makeAdder(5);
const add10 = makeAdder(10);

console.log(add5(3));
console.log(add10(3));
console.log(add5(add10(1)));
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
Each call to `makeAdder` creates a new EC with its own `x`. `add5` closes over `x = 5`, `add10` over `x = 10`. `add5(3)` → `5+3=8`. `add10(3)` → `10+3=13`. `add10(1)` → `11`, then `add5(11)` → `16`.

</details>

---

## 4. Stack Overflow Questions

---

### Q17. What will be the output?

```js
function loop() {
  loop();
}

try {
  loop();
} catch (e) {
  console.log(e.name);
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
RangeError
```

### Explanation
`loop` calls itself indefinitely, continuously pushing new ECs onto the call stack until the stack exceeds its maximum size. JavaScript throws a `RangeError` with the message `"Maximum call stack size exceeded"`. The `try/catch` catches it and logs the error name.

</details>

---

### Q18. What will be the output?

```js
function count(n) {
  if (n === 0) return "done";
  return count(n - 1);
}

console.log(count(3));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
done
```

### Explanation
`count(3)` calls `count(2)` → `count(1)` → `count(0)`. At `n === 0`, it returns `"done"`. Each return value propagates back up the chain. No overflow because the recursion has a clear base case and depth is only 4.

</details>

---

### Q19. What will be the output?

```js
function isMutuallyRecursive(n) {
  if (n <= 0) return "base";
  return isEven(n);
}

function isEven(n) {
  return isMutuallyRecursive(n - 1);
}

try {
  console.log(isMutuallyRecursive(50000));
} catch (e) {
  console.log(e.name + ": stack exhausted");
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
RangeError: stack exhausted
```

### Explanation
Even with a base case, calling 50000 levels of mutual recursion pushes ~50000 EC pairs onto the stack, exhausting it. JavaScript engines typically allow ~10,000–15,000 stack frames. The `RangeError` is caught and logged.

</details>

---

## 5. Nested Execution Questions

---

### Q20. What will be the output?

```js
var result = [];

function buildFunctions() {
  for (var i = 0; i < 3; i++) {
    result.push(function () {
      console.log(i);
    });
  }
}

buildFunctions();
result[0]();
result[1]();
result[2]();
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
`var i` is scoped to `buildFunctions`'s EC (not the loop block). All three pushed functions close over the same `i` variable. After the loop, `i = 3`. All three functions read `3` when invoked.

</details>

---

### Q21. What will be the output?

```js
var result = [];

function buildFunctions() {
  for (var i = 0; i < 3; i++) {
    result.push(
      (function (j) {
        return function () {
          console.log(j);
        };
      })(i)
    );
  }
}

buildFunctions();
result[0]();
result[1]();
result[2]();
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
An IIFE (Immediately Invoked Function Expression) is used to create a **new execution context** for each iteration, capturing the current value of `i` as `j`. Each inner function closes over a different `j`, giving `0`, `1`, `2`.

</details>

---

### Q22. What will be the output?

```js
function outer() {
  var count = 0;

  function inner() {
    count++;
    console.log(count);
  }

  return inner;
}

const fn1 = outer();
const fn2 = outer();

fn1(); fn1(); fn1();
fn2(); fn2();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
1
2
3
1
2
```

### Explanation
Each call to `outer()` creates a **new execution context** with a new, independent `count = 0`. `fn1` and `fn2` close over separate `count` variables. Calling `fn1` three times increments its own `count` to `3`. Calling `fn2` twice increments its own `count` to `2`.

</details>

---

### Q23. What will be the output?

```js
function parent() {
  let val = "parent";

  function child() {
    let val = "child";

    function grandchild() {
      console.log(val);
    }

    grandchild();
  }

  child();
  console.log(val);
}

parent();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
child
parent
```

### Explanation
`grandchild` looks up `val` through its scope chain: finds `val = "child"` in `child`'s lexical environment and logs it. After `child()` returns, `parent` logs its own `val = "parent"`. Each EC has its own `val` — they shadow each other, not overwrite.

</details>

---

### Q24. What will be the output?

```js
function createMultiplier(factor) {
  return {
    multiply(n) {
      return factor * n;
    },
    multiplyTwice(n) {
      return this.multiply(this.multiply(n));
    },
  };
}

const triple = createMultiplier(3);
console.log(triple.multiply(4));
console.log(triple.multiplyTwice(2));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
12
18
```

### Explanation
`triple.multiply(4)` → `3 * 4 = 12`. `triple.multiplyTwice(2)` calls `this.multiply(2)` → `6`, then `this.multiply(6)` → `18`. `factor` is captured from `createMultiplier`'s EC via closure.

</details>

---

### Q25. What will be the output?

```js
let x = "global";

function foo() {
  let x = "foo";
  bar();
}

function bar() {
  console.log(x);
}

foo();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
global
```

### Explanation
This demonstrates **lexical (static) scoping**. `bar`'s outer environment reference points to the Global EC (where `bar` was **defined**), not to `foo`'s EC (where `bar` was **called**). `x` in `bar`'s scope chain resolves to `"global"`.

</details>

---

## 6. Advanced Execution Context Questions

---

### Q26. What will be the output?

```js
function test() {
  console.log(arguments[0]);
  console.log(arguments[1]);
}

test(10, 20, 30);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
10
20
```

### Explanation
The `arguments` object is part of the Function EC's variable environment. It holds all passed arguments regardless of formal parameter count. `arguments[0]` is `10`, `arguments[1]` is `20`. `arguments[2]` (30) exists but is not logged.

</details>

---

### Q27. What will be the output?

```js
function outer() {
  var x = 10;

  function inner() {
    var x = 20;
    return x;
  }

  return inner() + x;
}

console.log(outer());
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
30
```

### Explanation
`inner()` creates its own EC with `x = 20` and returns `20`. Back in `outer`'s EC, `x` is still `10`. `inner() + x` → `20 + 10 = 30`. The two `x` variables are completely independent.

</details>

---

### Q28. What will be the output?

```js
function counter() {
  var count = 0;
  return {
    increment() { count++; },
    decrement() { count--; },
    value()     { return count; },
  };
}

const c = counter();
c.increment();
c.increment();
c.increment();
c.decrement();
console.log(c.value());
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
2
```

### Explanation
`counter()` returns an object whose three methods all close over the same `count` variable in `counter`'s (now-destroyed) EC. Three increments and one decrement: `0 + 3 - 1 = 2`.

</details>

---

### Q29. What will be the output?

```js
var x = 1;

function a() {
  var x = 2;
  b();
}

function b() {
  var x = 3;
  c();
}

function c() {
  console.log(x);
}

a();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
1
```

### Explanation
`c` is defined at the global level. Its outer environment reference is the Global EC, where `x = 1`. Despite being **called** from within `b`, which is called from `a`, the scope chain of `c` only goes through its **lexical** ancestors — which is just the Global EC. So `x` resolves to the global `1`.

</details>

---

### Q30. What will be the output?

```js
function init() {
  var name = "init";

  function display() {
    console.log(name);
  }

  return display;
}

const fn = init();
fn();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
init
```

### Explanation
Classic closure demonstration. `init()` returns `display`. Even after `init`'s EC is popped off the call stack, the `name` binding lives on in `display`'s closed-over lexical environment. Calling `fn()` logs `"init"`.

</details>

---

### Q31. What will be the output?

```js
var funcs = [];

for (var i = 0; i < 5; i++) {
  funcs[i] = function () { return i * i; };
}

console.log(funcs[0]());
console.log(funcs[2]());
console.log(funcs[4]());
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
25
25
25
```

### Explanation
`var i` is in the Global EC (or the enclosing function if there were one). All five functions share the same `i`. After the loop completes, `i = 5`. When called, all functions compute `5 * 5 = 25`.

</details>

---

### Q32. What will be the output?

```js
function memoize(fn) {
  const cache = {};
  return function (n) {
    if (n in cache) {
      console.log("cached:", n);
      return cache[n];
    }
    console.log("computing:", n);
    cache[n] = fn(n);
    return cache[n];
  };
}

const square = memoize(function (x) { return x * x; });

console.log(square(4));
console.log(square(4));
console.log(square(5));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
computing: 4
16
cached: 4
16
computing: 5
25
```

### Explanation
`memoize` returns a closure that captures `cache` from `memoize`'s EC. First call with `4`: not in cache, computes `16` and stores it. Second call with `4`: found in cache, returns `16` without recomputing. Call with `5`: not in cache, computes and stores `25`.

</details>

---

### Q33. What will be the output?

```js
function outer() {
  let shared = 0;

  const inc = () => { shared++; return shared; };
  const dec = () => { shared--; return shared; };
  const get = () => shared;

  return [inc, dec, get];
}

const [inc, dec, get] = outer();

console.log(inc());
console.log(inc());
console.log(dec());
console.log(get());
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
1
2
1
1
```

### Explanation
All three arrow functions close over the same `shared` variable in `outer`'s lexical environment. `inc()` twice → `shared` becomes `2`. `dec()` → `shared` becomes `1`. `get()` returns the current value `1`.

</details>

---

## Final Tips

- The call stack is LIFO — the last function pushed is the first to complete and be popped.
- The **memory creation phase** happens before any line of code executes — this is why `var` declarations and function declarations are available before their line in the source.
- `let` and `const` are hoisted but live in the **TDZ** — accessing them before their declaration is a `ReferenceError`, not `undefined`.
- Functions declared inside other functions get their **outer environment reference** set to the enclosing function's lexical environment at **definition** time, not call time — this is lexical (static) scoping.
- Closures keep the outer EC's variable environment alive even after the function returns — only a garbage collector can clean them up when no references remain.
- Recursive functions with no base case will always cause a `RangeError: Maximum call stack size exceeded`.
- Each call to a factory/outer function creates a **new, independent** EC and closure — two calls to `counter()` produce two separate `count` variables.
- The classic `var` in a `for` loop closure bug (`3 3 3` vs `0 1 2`) is one of the most common execution-context interview traps — remember `let` creates a new binding per iteration.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
