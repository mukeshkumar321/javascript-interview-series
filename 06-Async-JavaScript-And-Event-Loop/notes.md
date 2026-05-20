# Async JavaScript and Event Loop in JavaScript

## Table of Contents

1. [Synchronous vs Asynchronous JavaScript](#1-synchronous-vs-asynchronous-javascript)
2. [Why JavaScript is Single-Threaded](#2-why-javascript-is-single-threaded)
3. [The Call Stack](#3-the-call-stack)
4. [Web APIs](#4-web-apis)
5. [Callback Queue (Macrotask Queue)](#5-callback-queue-macrotask-queue)
6. [Microtask Queue](#6-microtask-queue)
7. [The Event Loop](#7-the-event-loop)
8. [Macrotask vs Microtask](#8-macrotask-vs-microtask)
9. [setTimeout(fn, 0) — Not Immediate](#9-settimeoutfn-0--not-immediate)
10. [Callbacks and Callback Hell](#10-callbacks-and-callback-hell)
11. [Promises](#11-promises)
12. [Promise Chaining](#12-promise-chaining)
13. [Promise Error Handling (.catch)](#13-promise-error-handling-catch)
14. [Promise.all](#14-promiseall)
15. [Promise.allSettled](#15-promiseallsettled)
16. [Promise.race](#16-promiserace)
17. [Promise.any](#17-promiseany)
18. [async/await Syntax](#18-asyncawait-syntax)
19. [Error Handling in async/await (try/catch)](#19-error-handling-in-asyncawait-trycatch)
20. [Async Functions Always Return a Promise](#20-async-functions-always-return-a-promise)
21. [Top-Level await](#21-top-level-await)
22. [Event Loop with async/await](#22-event-loop-with-asyncawait)
23. [setInterval and clearInterval](#23-setinterval-and-clearinterval)
24. [queueMicrotask](#24-queuemicrotask)
25. [Summary Table](#25-summary-table)

---

## 1. Synchronous vs Asynchronous JavaScript

**Synchronous** code runs line by line — each line must complete before the next one starts. **Asynchronous** code allows certain operations (like timers or network requests) to run in the background, so JavaScript can continue executing other code without waiting.

```js
// Synchronous — blocks until each line is done
console.log("One");
console.log("Two");
console.log("Three");
```

### Output

```js
One
Two
Three
```

---

```js
// Asynchronous — setTimeout callback runs later
console.log("Start");

setTimeout(() => {
  console.log("Timeout callback");
}, 1000);

console.log("End");
```

### Output

```js
Start
End
Timeout callback
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 2. Why JavaScript is Single-Threaded

JavaScript has a single call stack, meaning only one piece of code executes at a time. This design deliberately avoids complex concurrency bugs (like race conditions or deadlocks). The browser or Node.js runtime handles async work via Web APIs and queues, feeding results back into JavaScript one callback at a time.

```js
// Even with multiple async operations scheduled together,
// JS processes their callbacks one at a time in order
setTimeout(() => console.log("A"), 0);
setTimeout(() => console.log("B"), 0);
// A always logs before B — queued and processed in order
```

### Output

```js
A
B
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 3. The Call Stack

The call stack is a LIFO (Last In, First Out) data structure that tracks function execution. When a function is called it is pushed onto the stack; when it returns it is popped off. The event loop can only push new callbacks onto the stack when it is completely empty.

```js
function greet(name) {
  return `Hello, ${name}`;
}

function main() {
  const message = greet("Alice");
  console.log(message);
}

main();
// Call stack sequence:
// push main() → push greet() → pop greet() → pop main()
```

### Output

```js
Hello, Alice
```

> For a full deep-dive into the call stack, execution contexts, and the scope chain see `05-Execution-Context/notes.md`.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 4. Web APIs

Web APIs are provided by the **browser** (or Node.js runtime) and live **outside** the JavaScript engine. They handle time-consuming operations so the call stack is never blocked. When an operation completes, the callback is placed in the appropriate queue.

Common Web APIs:
- `setTimeout` / `setInterval`
- `fetch` (network requests)
- DOM event listeners (`click`, `keydown`, etc.)
- `XMLHttpRequest`
- `requestAnimationFrame`

```js
console.log("Before fetch");

fetch("https://jsonplaceholder.typicode.com/todos/1")
  .then(res => res.json())
  .then(data => console.log("Fetched:", data.title));

console.log("After fetch");
```

### Output

```js
Before fetch
After fetch
Fetched: delectus aut autem   // arrives asynchronously after network response
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 5. Callback Queue (Macrotask Queue)

When a Web API operation finishes (e.g., a `setTimeout` timer fires), its callback is placed in the **Callback Queue** — also called the **Macrotask Queue** or **Task Queue**. The event loop picks callbacks from here only when the call stack is empty **and** the microtask queue is also empty.

Sources of macrotasks:
- `setTimeout`
- `setInterval`
- `setImmediate` (Node.js)
- I/O callbacks
- UI rendering tasks

```js
setTimeout(() => console.log("Macrotask 1"), 0);
setTimeout(() => console.log("Macrotask 2"), 0);

console.log("Synchronous");
```

### Output

```js
Synchronous
Macrotask 1
Macrotask 2
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 6. Microtask Queue

The **Microtask Queue** (also called the Job Queue) holds callbacks that should run immediately after the current task completes — and crucially, before any macrotask. Microtasks are processed to completion before the event loop picks the next macrotask.

Sources of microtasks:
- `Promise.then` / `.catch` / `.finally`
- `queueMicrotask()`
- `MutationObserver`
- `async/await` continuations (code after `await`)

```js
console.log("Start");

Promise.resolve().then(() => console.log("Microtask 1"));
Promise.resolve().then(() => console.log("Microtask 2"));

console.log("End");
```

### Output

```js
Start
End
Microtask 1
Microtask 2
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 7. The Event Loop

The event loop is the mechanism that coordinates the call stack, microtask queue, and callback queue. Its algorithm (simplified):

1. Execute all synchronous code (run the current task to completion).
2. **Drain the microtask queue** — process every microtask until it is empty, including any new microtasks added during this step.
3. Pick **one** macrotask from the callback queue and execute it.
4. Go back to step 2.

```js
console.log("1 - sync");

setTimeout(() => console.log("2 - macrotask"), 0);

Promise.resolve()
  .then(() => console.log("3 - microtask"))
  .then(() => console.log("4 - chained microtask"));

console.log("5 - sync");
```

### Output

```js
1 - sync
5 - sync
3 - microtask
4 - chained microtask
2 - macrotask
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 8. Macrotask vs Microtask

**Microtasks always run before the next macrotask.** After every macrotask, the entire microtask queue is completely drained before another macrotask begins.

| Feature | Macrotask | Microtask |
|---|---|---|
| Sources | setTimeout, setInterval, I/O | Promise.then, queueMicrotask, MutationObserver |
| Priority | Lower | Higher |
| Queue drained | One per event loop turn | All at once before next macrotask |

```js
setTimeout(() => console.log("setTimeout"), 0);       // macrotask
Promise.resolve().then(() => console.log("promise")); // microtask
queueMicrotask(() => console.log("queueMicrotask"));  // microtask

console.log("sync");
```

### Output

```js
sync
promise
queueMicrotask
setTimeout
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 9. setTimeout(fn, 0) — Not Immediate

`setTimeout(fn, 0)` does **not** run the callback immediately. It schedules it as a macrotask after all current synchronous code and all queued microtasks have finished. The `0` is a minimum delay, not a guarantee of immediacy.

```js
console.log("before");

setTimeout(() => {
  console.log("inside setTimeout(0)");
}, 0);

Promise.resolve().then(() => console.log("inside .then"));

console.log("after");
```

### Output

```js
before
after
inside .then
inside setTimeout(0)
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 10. Callbacks and Callback Hell

A **callback** is a function passed as an argument to another function, to be called once an async operation completes. **Callback hell** (the "pyramid of doom") happens when multiple nested callbacks make code hard to read, debug, and maintain.

```js
// Callback hell example
getUser(userId, (user) => {
  getOrders(user.id, (orders) => {
    getOrderDetails(orders[0].id, (details) => {
      getShipping(details.shippingId, (shipping) => {
        console.log("Shipping info:", shipping);
        // deeply nested — error handling becomes a nightmare
      });
    });
  });
});
```

### Output

```js
// No direct output — illustrates the nesting structure
// Shipping info: { ... }  (when all callbacks resolve)
```

Callbacks also introduce **inversion of control** — you hand control to a third-party function and trust it will call your callback correctly, at the right time, and with the right arguments. Promises solve both problems.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 11. Promises

A **Promise** is an object representing the eventual completion or failure of an asynchronous operation. It solves callback hell and the inversion-of-control problem.

**Three states:**
- `pending` — initial state, neither fulfilled nor rejected
- `fulfilled` — operation completed successfully (has a resolved value)
- `rejected` — operation failed (has a reason or error)

A Promise transitions from `pending` to either `fulfilled` or `rejected` exactly once and never changes state again.

```js
const p = new Promise((resolve, reject) => {
  const success = true;
  if (success) {
    resolve("Data loaded");
  } else {
    reject(new Error("Something went wrong"));
  }
});

p.then(value => console.log(value))
 .catch(err => console.log(err.message));
```

### Output

```js
Data loaded
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 12. Promise Chaining

`.then()` always returns a **new Promise**. The return value of each `.then` callback becomes the resolved value passed to the next `.then` in the chain, enabling readable sequential async operations.

```js
Promise.resolve(1)
  .then(val => {
    console.log(val);   // 1
    return val + 1;
  })
  .then(val => {
    console.log(val);   // 2
    return val * 3;
  })
  .then(val => {
    console.log(val);   // 6
  });
```

### Output

```js
1
2
6
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 13. Promise Error Handling (.catch)

`.catch(fn)` is shorthand for `.then(undefined, fn)`. It catches any rejection anywhere in the preceding chain. `.finally(fn)` runs regardless of outcome but does not receive the resolved value or rejection reason.

```js
Promise.resolve("start")
  .then(() => {
    throw new Error("Something broke");
  })
  .then(() => {
    console.log("This will be skipped");
  })
  .catch(err => {
    console.log("Caught:", err.message);
    return "recovered";
  })
  .then(val => {
    console.log("After catch:", val);
  })
  .finally(() => {
    console.log("Finally runs always");
  });
```

### Output

```js
Caught: Something broke
After catch: recovered
Finally runs always
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 14. Promise.all

`Promise.all(iterable)` waits for **all** promises to fulfill and returns an array of results in the same order. If **any** promise rejects, the entire `Promise.all` rejects immediately (**fail-fast**) — but other promises still run to completion.

```js
const p1 = Promise.resolve(10);
const p2 = Promise.resolve(20);
const p3 = Promise.resolve(30);

Promise.all([p1, p2, p3]).then(values => {
  console.log(values);
});

// Fail-fast example
const pFail = Promise.reject(new Error("oops"));

Promise.all([p1, pFail, p3]).catch(err => {
  console.log("Error:", err.message);
});
```

### Output

```js
[10, 20, 30]
Error: oops
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 15. Promise.allSettled

`Promise.allSettled(iterable)` waits for **all** promises to settle — whether they fulfill or reject — and returns an array of result objects. Each object has a `status` property (`"fulfilled"` or `"rejected"`), plus `value` or `reason`. It never rejects.

```js
const promises = [
  Promise.resolve("success"),
  Promise.reject(new Error("failed")),
  Promise.resolve("another success"),
];

Promise.allSettled(promises).then(results => {
  results.forEach(r => {
    console.log(r.status, r.value ?? r.reason?.message);
  });
});
```

### Output

```js
fulfilled success
rejected failed
fulfilled another success
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 16. Promise.race

`Promise.race(iterable)` settles as soon as the **first** promise settles — whether it fulfills or rejects. Commonly used to implement timeouts.

```js
const slow = new Promise(resolve => setTimeout(() => resolve("slow"), 500));
const fast = new Promise(resolve => setTimeout(() => resolve("fast"), 100));

Promise.race([slow, fast]).then(winner => {
  console.log("Winner:", winner);
});
```

### Output

```js
Winner: fast
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 17. Promise.any

`Promise.any(iterable)` resolves as soon as the **first** promise **fulfills**. It only rejects if **all** promises reject, throwing an `AggregateError`. Introduced in ES2021.

```js
const p1 = Promise.reject(new Error("err 1"));
const p2 = Promise.reject(new Error("err 2"));
const p3 = Promise.resolve("first success");

Promise.any([p1, p2, p3]).then(value => {
  console.log("First fulfilled:", value);
});

// All reject case
Promise.any([
  Promise.reject(new Error("a")),
  Promise.reject(new Error("b")),
]).catch(err => {
  console.log(err instanceof AggregateError);
  console.log("All rejected");
});
```

### Output

```js
First fulfilled: first success
true
All rejected
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 18. async/await Syntax

`async` functions always return a Promise. `await` pauses execution **inside** the async function until the awaited Promise settles, without blocking the main thread. Code after `await` becomes a microtask continuation.

```js
function delay(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

async function fetchData() {
  console.log("Fetching...");
  await delay(1000);
  console.log("Data received");
  return "result";
}

fetchData().then(val => console.log("Returned:", val));
console.log("This runs while waiting");
```

### Output

```js
Fetching...
This runs while waiting
Data received
Returned: result
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 19. Error Handling in async/await (try/catch)

Use `try/catch` inside async functions to handle rejections. This reads like synchronous error handling but is equivalent to `.catch()` on a Promise chain.

```js
async function riskyOperation() {
  throw new Error("Network error");
}

async function main() {
  try {
    const result = await riskyOperation();
    console.log(result);
  } catch (err) {
    console.log("Caught:", err.message);
  } finally {
    console.log("Cleanup done");
  }
}

main();
```

### Output

```js
Caught: Network error
Cleanup done
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 20. Async Functions Always Return a Promise

An `async` function **always** returns a Promise, even when you return a plain value. The plain value is automatically wrapped with `Promise.resolve()`. A thrown error is wrapped with `Promise.reject()`.

```js
async function getValue() {
  return 42; // automatically wrapped: Promise.resolve(42)
}

async function getError() {
  throw new Error("fail"); // automatically wrapped: Promise.reject(...)
}

console.log(getValue() instanceof Promise); // runs synchronously

getValue().then(v => console.log(v));
getError().catch(e => console.log(e.message));
```

### Output

```js
true
42
fail
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 21. Top-Level await

In ES modules (`.mjs` files or `"type": "module"` in `package.json`), `await` can be used at the top level without wrapping it in an `async` function. It pauses the module's own execution until the Promise resolves.

```js
// top-level-example.mjs
const response = await fetch("https://jsonplaceholder.typicode.com/todos/1");
const data = await response.json();
console.log(data.title);
```

### Output

```js
delectus aut autem   // the actual todo title returned by the API
```

> Top-level `await` is only valid inside ES modules. Using it in a CommonJS (`require`-based) file will throw a `SyntaxError`.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 22. Event Loop with async/await

`await` yields control back to the event loop. Everything after an `await` line is scheduled as a microtask. This means synchronous code that runs after calling an async function executes before the awaited continuation resumes.

```js
async function asyncTask() {
  console.log("async: before await");
  await Promise.resolve();
  console.log("async: after await"); // queued as a microtask
}

console.log("1: sync start");
asyncTask();
console.log("2: sync end");
```

### Output

```js
1: sync start
async: before await
2: sync end
async: after await
```

The key insight: `asyncTask()` runs synchronously up to the first `await`, then suspends. Control returns to the caller, which logs `"2: sync end"`. Only then does the microtask queue drain and resume `asyncTask`.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 23. setInterval and clearInterval

`setInterval(fn, ms)` repeatedly calls `fn` approximately every `ms` milliseconds as a macrotask. `clearInterval(id)` stops the repetition. Like `setTimeout`, the delay is a minimum, not a precise guarantee.

```js
let count = 0;
const id = setInterval(() => {
  count++;
  console.log("Tick:", count);
  if (count === 3) {
    clearInterval(id);
    console.log("Stopped");
  }
}, 100);
```

### Output

```js
Tick: 1
Tick: 2
Tick: 3
Stopped
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 24. queueMicrotask

`queueMicrotask(fn)` schedules `fn` directly in the microtask queue. It is semantically equivalent to `Promise.resolve().then(fn)` but more explicit and slightly more efficient since it avoids creating a Promise object.

```js
console.log("1");

queueMicrotask(() => {
  console.log("2 - microtask via queueMicrotask");
});

Promise.resolve().then(() => {
  console.log("3 - microtask via Promise.then");
});

console.log("4");
```

### Output

```js
1
4
2 - microtask via queueMicrotask
3 - microtask via Promise.then
```

Both microtasks are enqueued in the same microtask queue and run in the order they were added.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 25. Summary Table

| Concept | Key Point |
|---|---|
| Synchronous code | Runs line by line; blocks execution until done |
| Asynchronous code | Runs in the background via Web APIs |
| Single-threaded | One call stack; only one task executes at a time |
| Call stack | LIFO structure tracking the current function calls |
| Web APIs | Browser/Node features outside the JS engine (timers, fetch, etc.) |
| Macrotask queue | Holds callbacks from setTimeout, setInterval, I/O |
| Microtask queue | Holds Promise.then, queueMicrotask — higher priority than macrotasks |
| Event loop | Drain all microtasks, then process one macrotask, repeat |
| Microtask priority | ALL microtasks run before the next macrotask begins |
| setTimeout(fn, 0) | Schedules a macrotask — never truly immediate |
| Callback hell | Deeply nested callbacks — avoided with Promises or async/await |
| Promise states | pending → fulfilled or rejected (immutable once settled) |
| Promise.all | All fulfill or fail-fast on first rejection |
| Promise.allSettled | Returns all results regardless of rejection |
| Promise.race | Settles with the first promise to settle (either way) |
| Promise.any | Resolves with the first fulfilled promise |
| async/await | Syntactic sugar over Promises; continuations run as microtasks |
| await | Pauses the async function; code after await is a microtask |
| Async return value | Always a Promise, even for plain return values |
| Top-level await | Available in ES modules only |
| setInterval | Repeating macrotask; stopped with clearInterval |
| queueMicrotask | Schedules a microtask directly without creating a Promise |

---

## Final Notes

Async JavaScript is one of the most heavily tested areas in frontend and backend interviews. The single most important mental model is the **execution order**: synchronous code runs first, then all microtasks (Promises, queueMicrotask, async/await continuations) are drained completely, and only then does the event loop process the next macrotask. `async/await` is pure syntactic sugar — under the hood it compiles to Promises, so the same microtask rules apply. Once you internalize this order you can confidently predict the output of any async interview question.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
