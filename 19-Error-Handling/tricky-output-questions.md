# Error Handling — Tricky Output Questions

## Table of Contents

1. [try/catch/finally Questions](#1-trycatchfinally-questions)
2. [Error Type Questions](#2-error-type-questions)
3. [Custom Error Questions](#3-custom-error-questions)
4. [Promise Error Handling Questions](#4-promise-error-handling-questions)
5. [async/await Error Questions](#5-asyncawait-error-questions)
6. [finally Return Questions](#6-finally-return-questions)
7. [Advanced Error Questions](#7-advanced-error-questions)

---

## 1. try/catch/finally Questions

---

### Q1. What will be the output?

```js
function test() {
  try {
    console.log('try');
    throw new Error('oops');
    console.log('after throw');
  } catch (e) {
    console.log('catch:', e.message);
  } finally {
    console.log('finally');
  }
  console.log('after try/catch');
}

test();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
try
catch: oops
finally
after try/catch
```

### Explanation

`throw` immediately stops `try` execution — `"after throw"` is never reached. `catch` runs with the error. `finally` runs unconditionally. After the entire `try/catch/finally` completes normally (no rethrow), execution continues with `"after try/catch"`.

</details>

---

### Q2. Does `catch` run when there is no error?

```js
function test() {
  try {
    console.log('try: no error');
  } catch (e) {
    console.log('catch:', e.message);
  } finally {
    console.log('finally');
  }
}

test();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
try: no error
finally
```

### Explanation

`catch` only runs when `try` throws. If `try` completes without error, `catch` is skipped entirely. `finally` runs regardless of whether an error occurred.

</details>

---

### Q3. What will be the output?

```js
function inner() {
  try {
    throw new Error('from inner');
  } catch (e) {
    console.log('inner catch:', e.message);
    throw e; // rethrow the same error
  }
}

try {
  inner();
} catch (e) {
  console.log('outer catch:', e.message);
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
inner catch: from inner
outer catch: from inner
```

### Explanation

The inner `catch` handles the error and logs it, then rethrows the same error object with `throw e`. The rethrown error propagates up the call stack where the outer `try/catch` catches it. Both catch blocks run with the same `message`.

</details>

---

### Q4. What will be the output?

```js
function test() {
  try {
    throw new Error('first');
  } catch (e) {
    console.log('catch:', e.message);
    throw new Error('second'); // throw a new error from catch
  } finally {
    console.log('finally');
    // No return here — the 'second' error continues propagating
  }
}

try {
  test();
} catch (e) {
  console.log('outer:', e.message);
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
catch: first
finally
outer: second
```

### Explanation

`catch` logs `"first"` then throws a new `Error('second')`. `finally` still runs before the new error propagates. Since `finally` has no `return` statement, the `"second"` error is not suppressed and reaches the outer `catch`.

</details>

---

## 2. Error Type Questions

---

### Q1. What will be the output?

```js
try {
  const obj = null;
  console.log(obj.name);
} catch (e) {
  console.log(e instanceof TypeError);
  console.log(e instanceof ReferenceError);
  console.log(e.name);
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
true
false
"TypeError"
```

### Explanation

Accessing a property on `null` throws a `TypeError` because `null` is not an object. `TypeError` extends `Error`, so `instanceof TypeError` is `true` and `instanceof ReferenceError` is `false`. The `name` property on the error object reflects the constructor name.

</details>

---

### Q2. What will be the output?

```js
try {
  console.log(typeof undeclaredVariable);
} catch (e) {
  console.log('caught');
}

try {
  console.log(undeclaredVariable);
} catch (e) {
  console.log(e.name);
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
"undefined"
"ReferenceError"
```

### Explanation

`typeof` is the one operator that does **not** throw a `ReferenceError` when given an undeclared variable — it safely returns `"undefined"`. Directly accessing the undeclared variable (without `typeof`) throws a `ReferenceError`. This is why `typeof` is commonly used as a safe guard: `if (typeof myVar !== 'undefined')`.

</details>

---

### Q3. What will be the output?

```js
// What type of error does each line throw?
const errors = [];

try { new Array(-1); }               catch (e) { errors.push(e.name); }
try { decodeURIComponent('%'); }      catch (e) { errors.push(e.name); }
try { null(); }                        catch (e) { errors.push(e.name); }
try { let x = x; }                    catch (e) { errors.push(e.name); }

console.log(errors);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
["RangeError", "URIError", "TypeError", "ReferenceError"]
```

### Explanation

- `new Array(-1)` → `RangeError`: negative length is out of the valid range
- `decodeURIComponent('%')` → `URIError`: `%` is an incomplete percent-encoding sequence
- `null()` → `TypeError`: `null` is not a callable function
- `let x = x` → `ReferenceError`: `x` is in the temporal dead zone when its own initialiser runs

</details>

---

## 3. Custom Error Questions

---

### Q1. What will be the output?

```js
class AppError extends Error {
  constructor(message, code) {
    super(message);
    this.name = 'AppError';
    this.code = code;
  }
}

const err = new AppError('Something failed', 503);

console.log(err instanceof AppError);
console.log(err instanceof Error);
console.log(err.name);
console.log(err.message);
console.log(err.code);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
true
true
"AppError"
"Something failed"
503
```

### Explanation

`AppError` extends `Error`, so `instanceof` checks pass for both. `this.name = 'AppError'` overrides the inherited `"Error"` name. `super(message)` properly sets `this.message`. Custom properties like `code` are attached to the instance and are fully accessible.

</details>

---

### Q2. What will be the output when `this.name` is NOT set in a custom error class?

```js
class DatabaseError extends Error {
  constructor(message) {
    super(message);
    // Intentionally not setting this.name
  }
}

const err = new DatabaseError('Connection refused');
console.log(err.name);
console.log(err.message);
console.log(err instanceof DatabaseError);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
"Error"
"Connection refused"
true
```

### Explanation

When you extend `Error` without setting `this.name`, the instance inherits `name` from `Error.prototype.name`, which is `"Error"`. The `instanceof` check still works correctly because it checks the prototype chain, not the `name` string. Always set `this.name = 'DatabaseError'` explicitly to get the correct name in logs and stack traces.

</details>

---

### Q3. What will be the output?

```js
class HttpError extends Error {
  constructor(status, message) {
    super(message);
    this.name = 'HttpError';
    this.status = status;
  }
}

function handle(err) {
  if (err instanceof HttpError && err.status === 404) {
    console.log('Not found:', err.message);
  } else if (err instanceof HttpError) {
    console.log('HTTP error', err.status, ':', err.message);
  } else {
    console.log('Unknown error:', err.message);
  }
}

handle(new HttpError(404, 'User not found'));
handle(new HttpError(500, 'Internal server error'));
handle(new Error('Something else'));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
"Not found: User not found"
"HTTP error 500 : Internal server error"
"Unknown error: Something else"
```

### Explanation

Custom errors with `instanceof` enable type-safe error dispatching. This pattern is far more reliable than checking `err.message` strings, which can change. The `instanceof` check walks the prototype chain, so a `404 HttpError` passes both `instanceof HttpError` and `instanceof Error`.

</details>

---

## 4. Promise Error Handling Questions

---

### Q1. What will be the output?

```js
Promise.reject('something went wrong')
  .then(val => console.log('then:', val))
  .catch(err => console.log('catch:', err));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
catch: something went wrong
```

### Explanation

`Promise.reject()` creates an immediately rejected promise. The `.then()` handler is skipped entirely because the promise is rejected. Execution jumps directly to `.catch()`. Note that `'something went wrong'` is a plain string — not an Error object — which is valid but loses the stack trace benefit.

</details>

---

### Q2. What will be the output?

```js
Promise.resolve('start')
  .then(val => {
    console.log('then1:', val);
    throw new Error('error in then1');
  })
  .then(val => {
    console.log('then2:', val);  // What happens here?
  })
  .catch(err => {
    console.log('catch:', err.message);
    return 'recovered';
  })
  .then(val => {
    console.log('then3:', val);
  });
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
then1: start
catch: error in then1
then3: recovered
```

### Explanation

When `then1` throws, the chain enters a rejected state. `then2` is **skipped** because it has no rejection handler. `.catch()` receives the error and logs it. Crucially, `.catch()` returns `'recovered'` — a non-throwing return from `.catch()` resolves the chain back to a fulfilled state. `then3` then runs with the resolved value `"recovered"`.

</details>

---

### Q3. What will be the output — and what happens after?

```js
const p = Promise.reject(new Error('unhandled!'));
console.log('after reject creation');
// No .catch() attached to p
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
after reject creation
// Then asynchronously:
// UnhandledPromiseRejection: Error: unhandled!
// (Node.js may exit; browser fires 'unhandledrejection' event)
```

### Explanation

`Promise.reject()` creates the rejected promise synchronously, but the unhandled rejection warning fires **asynchronously** after the current task. The `console.log` runs first. In modern Node.js, an unhandled rejection terminates the process with exit code 1 unless a handler is registered for `process.on('unhandledRejection', ...)`. In browsers, the `unhandledrejection` event fires on `window`.

</details>

---

### Q4. What will be the output?

```js
Promise.reject('error')
  .then(null, err => {
    console.log('rejection handler:', err);
    return 'handled';
  })
  .then(val => console.log('next then:', val));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
rejection handler: error
next then: handled
```

### Explanation

`.then(onFulfilled, onRejected)` accepts a second argument as a rejection handler — it is equivalent to `.catch()` but only for that specific step. The rejection handler returns `'handled'`, which resolves the chain. The next `.then` runs with `'handled'`. This is less readable than `.catch()` and is rarely used in practice.

</details>

---

## 5. async/await Error Questions

---

### Q1. What will be the output?

```js
async function fail() {
  throw new Error('async error');
}

async function run() {
  try {
    await fail();
    console.log('after await'); // does this run?
  } catch (e) {
    console.log('caught:', e.message);
  }
  console.log('after try/catch');
}

run();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
caught: async error
after try/catch
```

### Explanation

`await fail()` throws because `fail()` returns a rejected promise. The `throw` inside `fail` causes the returned promise to reject, and `await` re-throws that rejection as a synchronous exception inside the `async` function, which `try/catch` then catches. `"after await"` is skipped. After `catch`, execution resumes normally with `"after try/catch"`.

</details>

---

### Q2. What will be the output? (The classic forgetting-await bug)

```js
async function getData() {
  try {
    const result = Promise.reject(new Error('oops')); // forgot await!
    console.log('reached this line');
  } catch (e) {
    console.log('caught:', e.message);
  }
}

getData();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
"reached this line"
// Then: UnhandledPromiseRejection: Error: oops
```

### Explanation

Without `await`, `Promise.reject(...)` creates a rejected promise but does **not** throw synchronously. The `try` block continues past the `Promise.reject` line and logs `"reached this line"`. The `catch` block is never entered. The rejected promise has no `.catch()` handler, so it becomes an unhandled rejection. The fix is simply `const result = await Promise.reject(...)`.

</details>

---

### Q3. What will be the output?

```js
async function test() {
  try {
    await Promise.resolve('first');
    console.log('A');
    await Promise.reject(new Error('second failed'));
    console.log('B');   // does this run?
    await Promise.resolve('third');
    console.log('C');   // does this run?
  } catch (e) {
    console.log('catch:', e.message);
  }
  console.log('D');
}

test();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
A
catch: second failed
D
```

### Explanation

The `async` function runs sequentially through each `await`. `'first'` resolves fine, `A` is logged. `'second failed'` rejects, which throws inside the `async` function. `B` and `C` are skipped. `catch` runs. After `catch`, execution continues past the `try/catch` and logs `D`.

</details>

---

## 6. finally Return Questions

---

### Q1. What will be the output? (The core gotcha)

```js
function test() {
  try {
    return 1;
  } finally {
    return 2;
  }
}

console.log(test());
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
2
```

### Explanation

This is the most commonly asked `finally` gotcha. The `return 1` inside `try` is "pending" — it sets the return value but execution does not leave the function until `finally` runs. When `finally` executes its own `return 2`, it **overrides** the pending return value. The function returns `2`, not `1`.

</details>

---

### Q2. What will be the output? (`finally` suppresses a thrown error)

```js
function test() {
  try {
    throw new Error('error from try');
  } finally {
    return 'from finally';
  }
}

console.log(test());
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
"from finally"
```

### Explanation

A `return` in `finally` **silently suppresses** any exception thrown in `try` (or `catch`). The `Error` is discarded completely. No exception propagates. The function returns `"from finally"` as if nothing went wrong. This is one of the most dangerous JavaScript anti-patterns — bugs hidden this way are extremely hard to track down.

</details>

---

### Q3. What will be the output when `finally` has no `return`?

```js
function test() {
  try {
    return 'try result';
  } finally {
    console.log('finally runs'); // side effect only, no return
  }
}

console.log(test());
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
finally runs
try result
```

### Explanation

When `finally` does **not** have a `return` statement, the pending return value from `try` is preserved. `finally` runs its side effects (the `console.log`) and then the function returns `"try result"`. This is the safe and expected usage of `finally` — for cleanup without affecting the return value.

</details>

---

### Q4. What will be the output?

```js
function test() {
  try {
    throw new Error('from try');
  } catch (e) {
    console.log('catch:', e.message);
    return 'from catch';
  } finally {
    console.log('finally');
    // No return in finally — catch's return is preserved
  }
}

console.log(test());
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
catch: from try
finally
from catch
```

### Explanation

`catch` logs the message and sets the pending return value to `'from catch'`. `finally` runs and logs — but has no `return`, so it does not override the pending value. The function returns `'from catch'`. The order is: catch body → finally body → actual return.

</details>

---

## 7. Advanced Error Questions

---

### Q1. Does `window.onerror` fire for locally caught errors?

```js
window.onerror = (message) => {
  console.log('global onerror:', message);
  return true;
};

// Case A: locally caught
try {
  throw new Error('local error');
} catch (e) {
  console.log('caught locally:', e.message);
}

// Case B: uncaught (inside setTimeout, outside try/catch)
setTimeout(() => {
  throw new Error('uncaught async error');
}, 10);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
caught locally: local error
// After 10ms:
global onerror: Uncaught Error: uncaught async error
```

### Explanation

`window.onerror` is a **last-resort** handler. It does NOT fire for errors that are caught by a `try/catch` — Case A handles the error locally and `onerror` is never involved. Case B throws inside a `setTimeout` callback where there is no enclosing `try/catch`, so it becomes an uncaught exception and `onerror` fires.

</details>

---

### Q2. What will be the output when throwing a non-Error value?

```js
try {
  throw 42;
} catch (e) {
  console.log(typeof e);
  console.log(e instanceof Error);
  console.log(e);
}

try {
  throw { code: 500, msg: 'server down' };
} catch (e) {
  console.log(typeof e);
  console.log(e.msg);
  console.log(typeof e.stack);
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
"number"
false
42
"object"
"server down"
"undefined"
```

### Explanation

JavaScript allows `throw` with any value. Throwing a number gives you a number in `catch`. Throwing a plain object gives you that object. Neither has a `.stack` property, which means you lose the call-stack trace — making debugging significantly harder. Always throw `Error` objects (or subclasses of `Error`) to preserve the stack trace.

</details>

---

### Q3. What will be the output? (unhandledrejection event timing)

```js
window.addEventListener('unhandledrejection', (event) => {
  console.log('unhandled:', event.reason.message);
  event.preventDefault();
});

console.log('A');

Promise.reject(new Error('forgotten'));

console.log('B');
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
A
B
unhandled: forgotten
```

### Explanation

`Promise.reject()` is synchronous — it creates the rejected promise immediately. But the `unhandledrejection` event fires **asynchronously** after the current synchronous task finishes. So `A` and `B` log first, then the event fires. This is why you sometimes see "this promise was rejected but no handler was attached" messages appearing after other log output.

</details>

---

### Q4. What will be the output with error chaining?

```js
function connectToDatabase() {
  throw new Error('Connection timeout');
}

function loadUser(id) {
  try {
    connectToDatabase();
  } catch (cause) {
    throw new Error(`Could not load user ${id}`, { cause });
  }
}

try {
  loadUser(99);
} catch (e) {
  console.log(e.message);
  console.log(e.cause.message);
  console.log(e.cause instanceof Error);
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
Could not load user 99
Connection timeout
true
```

### Explanation

The `cause` option (ES2022) chains errors while preserving the original. `loadUser` catches the low-level `"Connection timeout"` error and wraps it with a more descriptive message. At the top level, you can access both the high-level message (`e.message`) and the root cause (`e.cause`). This creates a full error chain that is invaluable for debugging layered systems.

</details>

---

## Final Tips

- `catch` only runs when `try` **throws** — it is skipped on clean exit.
- `finally` **always runs**: on clean exit, on throw, and on `return`.
- A `return` in `finally` **overrides** any `return` in `try`/`catch` and **silently suppresses** thrown errors.
- Throwing a non-`Error` value (string, number, plain object) loses the stack trace — always throw `Error` instances.
- `typeof undeclaredVar` is `"undefined"` — it does NOT throw a `ReferenceError`.
- `instanceof` checks in `catch` are the correct way to distinguish custom error types.
- Forgetting `await` means `Promise.reject(...)` is unhandled — `try/catch` cannot intercept it.
- `unhandledrejection` fires **asynchronously** after the current synchronous task.
- `window.onerror` does NOT fire for errors that are caught by a local `try/catch`.
- Use `new Error('msg', { cause: originalErr })` for error chaining — preserves the root cause.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
