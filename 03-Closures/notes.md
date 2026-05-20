# Closures in JavaScript

## Table of Contents

1. [What is a Closure?](#1-what-is-a-closure)
2. [How Closures Work](#2-how-closures-work)
3. [Basic Closure Example](#3-basic-closure-example)
4. [Closures and Private Variables](#4-closures-and-private-variables)
5. [Counter Pattern](#5-counter-pattern)
6. [Factory Functions](#6-factory-functions)
7. [Module Pattern (IIFE-based)](#7-module-pattern-iife-based)
8. [Closure in Loops (var Problem)](#8-closure-in-loops-var-problem)
9. [Closure in setTimeout](#9-closure-in-settimeout)
10. [Memoization with Closures](#10-memoization-with-closures)
11. [Partial Application using Closures](#11-partial-application-using-closures)
12. [Currying with Closures](#12-currying-with-closures)
13. [Event Listeners and Closures](#13-event-listeners-and-closures)
14. [Closures in React (Stale Closure)](#14-closures-in-react-stale-closure)
15. [Closure vs Class](#15-closure-vs-class)
16. [Common Closure Mistakes](#16-common-closure-mistakes)
17. [Memory Implications](#17-memory-implications)
18. [Once Function](#18-once-function)
19. [Compose / Pipe using Closures](#19-compose--pipe-using-closures)
20. [Interview Q: What Gets Captured?](#20-interview-q-what-gets-captured)
21. [Summary](#21-summary)

---

## 1. What is a Closure?

A **closure** is the combination of a function and the **lexical environment** (scope) in which that function was declared. Even after the outer function has finished executing, the inner function retains access to the variables of the outer function's scope.

> Every function in JavaScript is a closure — it always carries a reference to its surrounding scope.

```js
function outer() {
  const message = "Hello from outer";

  function inner() {
    console.log(message); // accesses outer's variable
  }

  return inner;
}

const fn = outer();
fn();
```

### Output

```js
Hello from outer
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 2. How Closures Work

When a function is created, JavaScript attaches a hidden `[[Environment]]` property to it. This property holds a **reference** to the variable environment (scope chain) that was active at the time the function was defined — not when it is called.

Key points:
- The outer function's variables are **not garbage-collected** as long as the inner function is reachable.
- The inner function holds a **live reference** (not a copy) to those variables — changes to the variable are visible inside the closure.

```js
function makeAdder(x) {
  return function (y) {
    return x + y; // x is captured by reference in the closure
  };
}

const add5 = makeAdder(5);
const add10 = makeAdder(10);

console.log(add5(3));   // 8
console.log(add10(3));  // 13
console.log(add5(7));   // 12
```

### Output

```js
8
13
12
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 3. Basic Closure Example

A function defined inside another function, returned and called later, still remembers the variables of its parent scope.

```js
function greet(name) {
  return function () {
    console.log("Hi, " + name + "!");
  };
}

const greetAlice = greet("Alice");
const greetBob   = greet("Bob");

greetAlice(); // Hi, Alice!
greetBob();   // Hi, Bob!
```

### Output

```js
Hi, Alice!
Hi, Bob!
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 4. Closures and Private Variables

Closures are the primary mechanism for **data encapsulation** in JavaScript. Variables declared inside a function are not accessible from outside — only the inner functions that close over them can read or modify them.

```js
function createPerson(name) {
  let age = 0; // private

  return {
    birthday() {
      age++;
    },
    getAge() {
      return age;
    },
    getName() {
      return name;
    },
  };
}

const alice = createPerson("Alice");
alice.birthday();
alice.birthday();
console.log(alice.getName()); // Alice
console.log(alice.getAge());  // 2
console.log(alice.age);       // undefined — age is private
```

### Output

```js
Alice
2
undefined
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 5. Counter Pattern

The counter is the most commonly asked closure example in interviews. Each call to `makeCounter` creates an **independent** closure with its own `count` variable.

```js
function makeCounter() {
  let count = 0;

  return {
    increment() { count++; },
    decrement() { count--; },
    getCount()  { return count; },
  };
}

const counter1 = makeCounter();
const counter2 = makeCounter();

counter1.increment();
counter1.increment();
counter1.increment();
counter2.increment();

console.log(counter1.getCount()); // 3
console.log(counter2.getCount()); // 1
```

### Output

```js
3
1
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 6. Factory Functions

A **factory function** uses closures to create and return objects or functions with pre-configured state. Each invocation produces an independent closure with its own closed-over variables.

```js
function createMultiplier(multiplier) {
  return function (number) {
    return number * multiplier;
  };
}

const double = createMultiplier(2);
const triple = createMultiplier(3);

console.log(double(5)); // 10
console.log(triple(5)); // 15
console.log(double(9)); // 18
```

### Output

```js
10
15
18
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 7. Module Pattern (IIFE-based)

The **Module Pattern** combines an IIFE with closures to create a module-like structure with private state and a public API — a pattern that predates ES6 modules.

```js
const bankAccount = (function () {
  let balance = 1000; // private

  function log(action, amount) {
    console.log(`${action}: $${amount} | Balance: $${balance}`);
  }

  return {
    deposit(amount) {
      balance += amount;
      log("Deposit", amount);
    },
    withdraw(amount) {
      if (amount > balance) {
        console.log("Insufficient funds");
        return;
      }
      balance -= amount;
      log("Withdraw", amount);
    },
    getBalance() {
      return balance;
    },
  };
})();

bankAccount.deposit(500);
bankAccount.withdraw(200);
console.log(bankAccount.getBalance());
```

### Output

```js
Deposit: $500 | Balance: $1500
Withdraw: $200 | Balance: $1300
1300
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 8. Closure in Loops (var Problem)

> **Pre-requisite:** `var` hoisting and function scoping are covered in `02-Scope-Hoisting`. This section focuses only on the closure-specific behaviour.

When `var` is used in a loop, all iterations **share the same variable** because `var` is function-scoped. Closures inside the loop all close over the **same reference**, so they all see the final value after the loop finishes.

**The Problem:**

```js
for (var i = 0; i < 3; i++) {
  setTimeout(function () {
    console.log(i); // all callbacks close over the same `i`
  }, 100);
}
```

### Output

```js
3
3
3
```

**Fix 1 — IIFE (creates a new scope per iteration):**

```js
for (var i = 0; i < 3; i++) {
  (function (j) {
    setTimeout(function () {
      console.log(j);
    }, 100);
  })(i);
}
```

### Output

```js
0
1
2
```

**Fix 2 — `let` (block-scoped, new binding per iteration):**

```js
for (let i = 0; i < 3; i++) {
  setTimeout(function () {
    console.log(i);
  }, 100);
}
```

### Output

```js
0
1
2
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 9. Closure in setTimeout

`setTimeout` callbacks are closures — they retain access to the variables in scope when they were defined. This is useful for deferred execution that still needs access to the original context.

```js
function delayedMessage(msg, delay) {
  setTimeout(function () {
    console.log(msg); // `msg` is captured in the closure
  }, delay);
}

delayedMessage("Hello after 1s", 1000);
delayedMessage("Hello after 2s", 2000);
```

### Output

```js
Hello after 1s    // printed after 1 second
Hello after 2s    // printed after 2 seconds
```

A common gotcha — the closure captures the variable, not the value at the time of the call:

```js
let x = "before";

setTimeout(function () {
  console.log(x); // captures the variable `x`
}, 0);

x = "after"; // modifies x before the callback runs
console.log(x); // "after" — synchronous, runs first
```

### Output

```js
after
after
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 10. Memoization with Closures

**Memoization** is an optimisation technique that caches the results of expensive function calls. Closures make it clean — the cache object is private and persists between calls.

```js
function memoize(fn) {
  const cache = {}; // private cache, persists via closure

  return function (...args) {
    const key = JSON.stringify(args);
    if (key in cache) {
      console.log("From cache:", key);
      return cache[key];
    }
    cache[key] = fn(...args);
    return cache[key];
  };
}

function slowSquare(n) {
  return n * n;
}

const fastSquare = memoize(slowSquare);

console.log(fastSquare(4)); // 16
console.log(fastSquare(4)); // From cache: [4] → 16
console.log(fastSquare(5)); // 25
```

### Output

```js
16
From cache: [4]
16
25
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 11. Partial Application using Closures

**Partial application** fixes some arguments of a function and returns a new function that takes the remaining arguments. The fixed arguments are captured in the closure.

```js
function multiply(a, b) {
  return a * b;
}

function partial(fn, ...presetArgs) {
  return function (...laterArgs) {
    return fn(...presetArgs, ...laterArgs); // presetArgs captured in closure
  };
}

const double = partial(multiply, 2);
const triple = partial(multiply, 3);

console.log(double(5)); // 10
console.log(triple(5)); // 15
```

### Output

```js
10
15
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 12. Currying with Closures

**Currying** transforms a function that takes multiple arguments into a sequence of functions each taking a single argument. Each intermediate function closes over the arguments received so far.

```js
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn(...args);
    }
    return function (...moreArgs) {
      return curried(...args, ...moreArgs); // args captured in closure
    };
  };
}

function add(a, b, c) {
  return a + b + c;
}

const curriedAdd = curry(add);

console.log(curriedAdd(1)(2)(3));  // 6
console.log(curriedAdd(1, 2)(3));  // 6
console.log(curriedAdd(1)(2, 3));  // 6
console.log(curriedAdd(1, 2, 3));  // 6
```

### Output

```js
6
6
6
6
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 13. Event Listeners and Closures

Event listener callbacks are closures — they close over variables in their defining scope. This is powerful but can cause **memory leaks** if listeners are never removed and they hold large objects in their closure.

```js
function setupButton(buttonId, message) {
  const button = document.getElementById(buttonId);

  button.addEventListener("click", function () {
    console.log(message); // `message` captured — stays in memory as long as the listener exists
  });
}
```

**Memory Leak Risk:**

```js
function attachListener() {
  const largeData = new Array(1_000_000).fill("data"); // held in closure

  document.getElementById("btn").addEventListener("click", function () {
    console.log(largeData[0]); // largeData cannot be GC'd while listener is alive
  });
  // If the listener is never removed, largeData stays in memory forever
}
```

**Fix — Remove the listener when no longer needed:**

```js
function attachListener() {
  const largeData = new Array(1_000_000).fill("data");

  function handler() {
    console.log(largeData[0]);
    document.getElementById("btn").removeEventListener("click", handler);
  }

  document.getElementById("btn").addEventListener("click", handler);
}
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 14. Closures in React (Stale Closure)

In React, closures are everywhere — `useEffect`, `useState` callbacks, and event handlers all close over component state and props. A **stale closure** happens when a function captures an outdated value of a state variable that has since changed.

**Stale closure in useEffect:**

```js
function Counter() {
  const [count, setCount] = React.useState(0);

  React.useEffect(() => {
    const id = setInterval(() => {
      // This closure captures `count` from the first render: 0
      // It never sees updated values on re-renders
      setCount(count + 1); // stale! always sets to 0 + 1 = 1
    }, 1000);

    return () => clearInterval(id);
  }, []); // empty deps — effect runs once, closure captures count = 0
}
```

**Fix 1 — Use a functional updater (no need to close over `count`):**

```js
React.useEffect(() => {
  const id = setInterval(() => {
    setCount(prev => prev + 1); // no closure over `count` — always fresh
  }, 1000);

  return () => clearInterval(id);
}, []);
```

**Fix 2 — Add `count` to the dependency array:**

```js
React.useEffect(() => {
  const id = setInterval(() => {
    setCount(count + 1); // `count` is fresh on each re-run
  }, 1000);

  return () => clearInterval(id);
}, [count]); // re-runs whenever count changes
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 15. Closure vs Class

Both closures and classes can implement private state with public methods. Here is the same counter implemented both ways:

**Closure approach:**

```js
function makeCounter(initial = 0) {
  let count = initial; // truly private — no external access

  return {
    increment: () => ++count,
    decrement: () => --count,
    reset:     () => { count = initial; },
    getCount:  () => count,
  };
}

const c = makeCounter(10);
console.log(c.increment()); // 11
console.log(c.decrement()); // 10
console.log(c.getCount());  // 10
```

### Output

```js
11
10
10
```

**Class approach:**

```js
class Counter {
  #count; // private field (ES2022)

  constructor(initial = 0) {
    this.#count = initial;
  }

  increment() { return ++this.#count; }
  decrement() { return --this.#count; }
  getCount()  { return this.#count; }
}

const c2 = new Counter(10);
console.log(c2.increment()); // 11
console.log(c2.decrement()); // 10
console.log(c2.getCount());  // 10
```

### Output

```js
11
10
10
```

**Key Differences:**

| Aspect | Closure | Class |
|---|---|---|
| Privacy | Truly private (no external access) | Private with `#` fields (ES2022) |
| Memory | Each instance has its own function copies | Methods live on the prototype (shared) |
| Inheritance | Achieved via composition | Built-in `extends` |
| `this` binding | Not needed (uses closed-over variables) | Required — can be lost |
| Readability | Functional style | OOP style |

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 16. Common Closure Mistakes

**Mistake 1 — Accidentally sharing mutable state across closures:**

```js
function makeAdders() {
  const adders = [];

  for (var i = 0; i < 3; i++) {
    adders.push(function (x) { return x + i; }); // all close over the same `i`
  }

  return adders;
}

const adders = makeAdders();
console.log(adders[0](10)); // 13, not 10 — i is 3 after the loop
console.log(adders[1](10)); // 13
console.log(adders[2](10)); // 13
```

### Output

```js
13
13
13
```

**Mistake 2 — Assuming closures copy values (they capture by reference):**

```js
let x = 10;
const getX = () => x;

x = 20; // modifies the variable the closure refers to
console.log(getX()); // 20, not 10
```

### Output

```js
20
```

**Mistake 3 — Unintentionally holding large data in a closure:**

```js
function setup() {
  const bigArray = new Array(1_000_000).fill(0);

  return function () {
    return bigArray.length; // only needs .length, but entire array is retained
  };
}
```

Fix: extract only the needed value before creating the closure.

```js
function setup() {
  const bigArray = new Array(1_000_000).fill(0);
  const length = bigArray.length; // extract what is needed

  return function () {
    return length; // bigArray can now be GC'd
  };
}
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 17. Memory Implications

Closures keep the variables they close over **alive in memory** as long as the closure itself is reachable. This is the expected behaviour — but it becomes a problem when large objects are retained unintentionally.

```js
function outer() {
  const bigArray = new Array(1_000_000).fill(0); // ~8 MB

  return function inner() {
    return bigArray[0]; // only uses one element, but keeps the entire array alive
  };
}

const fn = outer(); // bigArray cannot be GC'd while fn is alive
fn();

// Fix: null out the reference when done
// fn = null; // now bigArray can be garbage collected
```

**Best Practices:**

- Null out closure references when they are no longer needed.
- Avoid closing over large objects when only a small part is needed — extract the needed value into a smaller variable first.
- Remove event listeners when components unmount or are destroyed.
- Be careful with global variables that hold closures — they live for the entire lifetime of the page.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 18. Once Function

A **once function** ensures a wrapped function executes only once — subsequent calls are no-ops that return the cached result. The `called` flag and `result` variable live in the closure and survive across calls.

```js
function once(fn) {
  let called = false;
  let result;

  return function (...args) {
    if (!called) {
      called = true;
      result = fn(...args);
    }
    return result;
  };
}

const init = once(function () {
  console.log("Initialised!");
  return 42;
});

console.log(init()); // Initialised! → 42
console.log(init()); // (no log) → 42
console.log(init()); // (no log) → 42
```

### Output

```js
Initialised!
42
42
42
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 19. Compose / Pipe using Closures

**Compose** and **Pipe** are higher-order functions that chain other functions together. The `fns` array is captured in the closure returned by each.

**Compose (right-to-left execution):**

```js
function compose(...fns) {
  return function (value) {
    return fns.reduceRight((acc, fn) => fn(acc), value);
  };
}

const double  = x => x * 2;
const addTen  = x => x + 10;
const square  = x => x * x;

const transform = compose(square, addTen, double);
// execution order: double → addTen → square

console.log(transform(3)); // double(3)=6, addTen(6)=16, square(16)=256
```

### Output

```js
256
```

**Pipe (left-to-right execution):**

```js
function pipe(...fns) {
  return function (value) {
    return fns.reduce((acc, fn) => fn(acc), value);
  };
}

const process = pipe(double, addTen, square);
// execution order: double → addTen → square

console.log(process(3)); // double(3)=6, addTen(6)=16, square(16)=256
```

### Output

```js
256
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 20. Interview Q: What Gets Captured?

One of the most nuanced closure questions: **does a closure capture the value or the reference?**

**Answer: Closures capture the variable binding (reference), not the value.**

**Primitives — closure sees the current value of the variable:**

```js
function test() {
  let num = 1;
  const inner = () => num;
  num = 2; // update after inner is defined
  return inner;
}

console.log(test()()); // 2 — sees the updated value, not 1
```

### Output

```js
2
```

**Objects — closure sees mutations to the same object:**

```js
function test() {
  const obj = { a: 1 };
  const inner = () => obj.a;
  obj.a = 99; // mutates the same object
  return inner;
}

console.log(test()()); // 99
```

### Output

```js
99
```

**Reassignment — closure sees the new reference:**

```js
function test() {
  let obj = { a: 1 };
  const inner = () => obj.a;
  obj = { a: 99 }; // reassigns the variable to a new object
  return inner;
}

console.log(test()()); // 99 — closure tracks the variable, which now points to { a: 99 }
```

### Output

```js
99
```

**Key Rule:** Closures close over **variables** (bindings), not values. Whatever the variable holds at the time the inner function **executes** is what you get — not what it held when the inner function was **defined**.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 21. Summary

| Topic | Key Point |
|---|---|
| Definition | A closure = function + its lexical environment |
| Memory model | Inner function holds a live reference to outer scope; outer variables are not GC'd while closure is reachable |
| Private variables | Variables in outer scope are inaccessible from outside — only the closure can read or modify them |
| Counter pattern | Each `makeCounter()` call creates an independent closure with its own state |
| Factory functions | Closures parameterise behaviour at creation time — each call produces independent state |
| Module pattern | IIFE + closure = private state + public API |
| Loop + `var` | All iterations share the same `var` binding — use `let` or an IIFE to create per-iteration scope |
| `setTimeout` | Callbacks are closures — they see the variable value at execution time, not at definition time |
| Memoization | Cache object persists across calls via closure |
| Partial application | Pre-set arguments are captured in the closure and prepended on each call |
| Currying | Each nested function closes over previously received args |
| Event listeners | Closures can cause memory leaks if listeners are never removed |
| Stale closure (React) | Old render's state captured — fix with functional updaters or correct dependency arrays |
| Closure vs Class | Closures: truly private, functional style; Classes: prototype methods shared, OOP style |
| Memory implications | Closed-over variables stay in memory as long as the closure is reachable — null out when done |
| Once function | `called` flag and `result` live in the closure — survive across calls |
| Compose / Pipe | Higher-order functions that chain closures together |
| Value vs reference | Closures capture the variable binding — both primitives and objects captured by reference to the binding |

---

## Final Notes

Closures are one of the most powerful and frequently misunderstood features in JavaScript. They are the foundation of the Module Pattern, memoization, currying, partial application, higher-order functions, and much of React's hook model. The single most important concept to internalise is that a closure captures the **variable binding**, not a snapshot of the value — so the closure always sees the most current value of any closed-over variable. Understanding this is essential for writing correct, predictable JavaScript and for acing every closure question in an interview.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
