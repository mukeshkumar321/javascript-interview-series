# Error Handling in JavaScript

## Table of Contents

1. [What are Errors?](#1-what-are-errors)
2. [Error Types](#2-error-types)
3. [try/catch/finally](#3-trycatchfinally)
4. [finally Always Runs](#4-finally-always-runs)
5. [Error Object](#5-error-object)
6. [throw Statement](#6-throw-statement)
7. [Custom Error Classes](#7-custom-error-classes)
8. [Error Chaining](#8-error-chaining)
9. [Synchronous Error Handling](#9-synchronous-error-handling)
10. [Promise Error Handling](#10-promise-error-handling)
11. [Unhandled Promise Rejection](#11-unhandled-promise-rejection)
12. [async/await Error Handling](#12-asyncawait-error-handling)
13. [Async Error — Common Mistake](#13-async-error--common-mistake)
14. [Global Error Handlers](#14-global-error-handlers)
15. [Error Boundaries in React](#15-error-boundaries-in-react)
16. [Rethrowing Errors](#16-rethrowing-errors)
17. [Nested try/catch](#17-nested-trycatch)
18. [TypeError vs ReferenceError](#18-typeerror-vs-referenceerror)
19. [finally with return Value Gotcha](#19-finally-with-return-value-gotcha)
20. [Summary](#20-summary)

---

## 1. What are Errors?

An **error** in JavaScript is an object that represents something unexpected that occurred during execution. Errors can be caused by the JavaScript engine (e.g., syntax mistakes, invalid operations) or deliberately thrown by application code.

JavaScript has two broad categories:
- **Synchronous errors** — thrown immediately during code execution (e.g., `TypeError`, `ReferenceError`)
- **Asynchronous errors** — occur inside Promises or async functions and must be handled via `.catch` or `try/catch` with `await`

```js
// Synchronous error — caught by try/catch
try {
  null.name;
} catch (e) {
  console.log('Caught sync error:', e.name); // "TypeError"
}

// Asynchronous error — must use .catch or await + try/catch
Promise.reject(new Error('async fail'))
  .catch(e => console.log('Caught async error:', e.message)); // "async fail"
```

### Output

```js
"Caught sync error: TypeError"
"Caught async error: async fail"
```

---

## 2. Error Types

JavaScript defines six built-in error types, all extending `Error`.

| Type | When it occurs |
|---|---|
| `SyntaxError` | Code is malformed and cannot be parsed (e.g., missing bracket) |
| `ReferenceError` | Accessing a variable that has not been declared |
| `TypeError` | Value is not of the expected type (e.g., calling `null` as a function) |
| `RangeError` | Numeric value is outside the allowed range |
| `URIError` | Invalid argument passed to `encodeURI`/`decodeURI` |
| `EvalError` | Legacy; related to misuse of `eval()` (rarely seen today) |

```js
// TypeError — wrong type
try { null.property; }
catch (e) { console.log(e.name); } // "TypeError"

// ReferenceError — undeclared variable
try { undeclaredVar; }
catch (e) { console.log(e.name); } // "ReferenceError"

// RangeError — value out of range
try { new Array(-1); }
catch (e) { console.log(e.name); } // "RangeError"

// SyntaxError — only caught via eval (static syntax errors halt parsing)
try { eval('if ('); }
catch (e) { console.log(e.name); } // "SyntaxError"
```

### Output

```js
"TypeError"
"ReferenceError"
"RangeError"
"SyntaxError"
```

---

## 3. try/catch/finally

`try/catch/finally` is the primary mechanism for handling synchronous errors (and awaited async errors).

- **try** — code that might throw
- **catch** — runs only if `try` throws; receives the error object
- **finally** — always runs, regardless of whether an error was thrown or not

```js
function divide(a, b) {
  try {
    if (b === 0) throw new Error('Division by zero');
    const result = a / b;
    console.log('Result:', result);
    return result;
  } catch (e) {
    console.log('Error:', e.message);
    return null;
  } finally {
    console.log('finally: cleanup');
  }
}

divide(10, 2);
divide(10, 0);
```

### Output

```js
"Result: 5"
"finally: cleanup"
"Error: Division by zero"
"finally: cleanup"
```

---

## 4. finally Always Runs

`finally` executes in **all** exit paths from `try/catch`: normal completion, error thrown, or even a `return` statement inside `try` or `catch`.

```js
function test() {
  try {
    console.log('try');
    return 'try result';       // finally still runs before this return
  } catch (e) {
    console.log('catch');
  } finally {
    console.log('finally');    // always runs
  }
}

console.log(test());
```

### Output

```js
"try"
"finally"
"try result"
```

---

## 5. Error Object

Every error object has at minimum three standard properties.

| Property | Description |
|---|---|
| `message` | Human-readable description of the error |
| `name` | The error type name (e.g., `"TypeError"`) |
| `stack` | A string containing the stack trace (non-standard but universally supported) |

```js
function inner() {
  throw new TypeError('invalid input');
}

function outer() {
  inner();
}

try {
  outer();
} catch (e) {
  console.log(e.name);           // "TypeError"
  console.log(e.message);        // "invalid input"
  console.log(typeof e.stack);   // "string"
  console.log(e instanceof TypeError); // true
  console.log(e instanceof Error);     // true
}
```

### Output

```js
"TypeError"
"invalid input"
"string"
true
true
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 6. throw Statement

`throw` deliberately creates an error condition. You can throw any value, but best practice is to always throw an `Error` object so you get a `stack` trace.

```js
// You can throw any value — but avoid it
function riskyA() {
  throw 'a string error';  // bad practice
}

// Throw an Error object — correct
function riskyB() {
  throw new Error('something failed');
}

// Throw a specific built-in error type
function riskyC(val) {
  if (typeof val !== 'number') {
    throw new TypeError(`Expected number, got ${typeof val}`);
  }
}

try { riskyA(); } catch (e) { console.log(typeof e, e); }
try { riskyB(); } catch (e) { console.log(e.name, e.message); }
try { riskyC('hello'); } catch (e) { console.log(e.name, e.message); }
```

### Output

```js
"string" "a string error"
"Error" "something failed"
"TypeError" "Expected number, got string"
```

---

## 7. Custom Error Classes

Custom errors extend `Error` to give errors a meaningful `name` and optional extra properties. This lets `catch` blocks distinguish between different error types using `instanceof`.

```js
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = 'ValidationError'; // override default "Error"
    this.field = field;
  }
}

class NetworkError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.name = 'NetworkError';
    this.statusCode = statusCode;
  }
}

try {
  throw new ValidationError('Email is required', 'email');
} catch (e) {
  console.log(e instanceof ValidationError); // true
  console.log(e instanceof Error);           // true
  console.log(e.name);                       // "ValidationError"
  console.log(e.field);                      // "email"
  console.log(e.message);                    // "Email is required"
}
```

### Output

```js
true
true
"ValidationError"
"email"
"Email is required"
```

---

## 8. Error Chaining

ES2022 introduced the `cause` option on the `Error` constructor. It lets you wrap a low-level error inside a higher-level one while preserving the original, creating an **error chain**.

```js
async function fetchUser(id) {
  try {
    const res = await fetch(`/api/users/${id}`);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return await res.json();
  } catch (cause) {
    // Wrap the low-level error with context
    throw new Error(`Failed to fetch user ${id}`, { cause });
  }
}

fetchUser(42).catch((e) => {
  console.log(e.message);        // "Failed to fetch user 42"
  console.log(e.cause.message);  // "HTTP 404"
});
```

### Output

```js
"Failed to fetch user 42"
"HTTP 404"
```

---

## 9. Synchronous Error Handling

Synchronous errors thrown during normal code execution can be caught with `try/catch`. Errors not caught bubble up the call stack until they reach the global scope and become uncaught exceptions.

```js
function step1() { throw new Error('step1 failed'); }
function step2() { step1(); }
function step3() { step2(); }

// Without try/catch: uncaught error crashes the program
// step3(); // Uncaught Error: step1 failed

// With try/catch at the right level:
try {
  step3();
} catch (e) {
  console.log('Caught at top:', e.message);
  // e.stack will show: step1 → step2 → step3 → anonymous
}
```

### Output

```js
"Caught at top: step1 failed"
```

---

## 10. Promise Error Handling

Errors inside a Promise are handled with `.catch()` or a second argument to `.then()`. An error (or rejection) skips all `.then()` handlers until it reaches the next `.catch()`.

```js
Promise.resolve('start')
  .then(val => {
    console.log('then1:', val);
    throw new Error('thrown in then1');
  })
  .then(val => {
    console.log('then2:', val); // SKIPPED — error above
  })
  .catch(err => {
    console.log('catch:', err.message);
    return 'recovered'; // returning here resumes the chain
  })
  .then(val => {
    console.log('then3:', val); // resumes after .catch returns
  });
```

### Output

```js
"then1: start"
"catch: thrown in then1"
"then3: recovered"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 11. Unhandled Promise Rejection

A Promise that rejects without a `.catch()` handler is an **unhandled promise rejection**. In Node.js this terminates the process (unless handled). In browsers, it fires the `unhandledrejection` event on `window`.

```js
// In Node.js (modern versions):
// UnhandledPromiseRejectionWarning — process may exit with code 1

// In the browser:
window.addEventListener('unhandledrejection', (event) => {
  console.log('Unhandled rejection:', event.reason);
  event.preventDefault(); // suppress the default console error
});

Promise.reject(new Error('forgotten rejection'));
console.log('after reject');
```

### Output

```js
"after reject"
// Asynchronously (after current task):
"Unhandled rejection: Error: forgotten rejection"
```

---

## 12. async/await Error Handling

Inside an `async` function, `throw` and `await` errors can both be caught with a regular `try/catch`, making async error handling look identical to synchronous code.

```js
async function fetchData() {
  try {
    const res = await fetch('/api/data');

    if (!res.ok) {
      throw new Error(`HTTP ${res.status}`);
    }

    const data = await res.json();
    console.log('data:', data);
    return data;
  } catch (e) {
    // Catches: network errors, HTTP errors we threw, and JSON parse errors
    console.log('caught:', e.message);
    return null;
  }
}

fetchData();
```

### Output

```js
// On success:
"data:" { /* parsed JSON */ }

// On 404:
"caught: HTTP 404"

// On network failure:
"caught: Failed to fetch"
```

---

## 13. Async Error — Common Mistake

Forgetting `await` before a Promise that rejects means the rejected Promise is never awaited, so the `try/catch` cannot intercept it.

```js
async function buggy() {
  try {
    const p = Promise.reject(new Error('oops')); // forgot await
    console.log('after rejected promise creation'); // still runs!
  } catch (e) {
    console.log('caught:', e.message); // NEVER reached
  }
}
buggy();
// Result: unhandled promise rejection for "oops"

// Fixed version:
async function fixed() {
  try {
    const p = await Promise.reject(new Error('oops')); // with await
  } catch (e) {
    console.log('caught:', e.message); // "oops"
  }
}
fixed();
```

### Output

```js
// buggy():
"after rejected promise creation"
// UnhandledPromiseRejection: Error: oops

// fixed():
"caught: oops"
```

---

## 14. Global Error Handlers

Global handlers catch errors that were not caught by any local `try/catch`. They are a last resort and are typically used for logging — they do not prevent the error from propagating.

```js
// In the browser:

// Catches synchronous uncaught errors and script load errors
window.onerror = (message, source, line, col, error) => {
  console.log('Global onerror:', message);
  return true; // returning true prevents the default browser error logging
};

// Catches unhandled promise rejections
window.addEventListener('unhandledrejection', (event) => {
  console.log('Unhandled rejection:', event.reason.message);
  event.preventDefault();
});

// These will be caught by the global handlers:
// setTimeout(() => { throw new Error('async throw'); }, 0);
// Promise.reject(new Error('unhandled'));

// Errors caught locally are NOT forwarded to global handlers:
try {
  throw new Error('local error');
} catch (e) {
  console.log('locally caught:', e.message); // window.onerror is NOT called
}
```

### Output

```js
"locally caught: local error"
// window.onerror is NOT called for locally caught errors
```

---

## 15. Error Boundaries in React

In React, a runtime error during rendering, in lifecycle methods, or in constructors of child components will crash the entire component tree. **Error Boundaries** are class components that implement `componentDidCatch` and `getDerivedStateFromError` to catch those errors and display a fallback UI.

```js
// React — Error Boundary class component
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // Update state so the next render shows the fallback UI
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    // Log to an error reporting service
    console.log('Caught by boundary:', error.message);
    console.log('Component stack:', info.componentStack);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}

// Usage:
// <ErrorBoundary>
//   <MyComponent />   ← errors here are caught
// </ErrorBoundary>
```

### Output

```js
// When MyComponent throws during render:
"Caught by boundary: [error message]"
"Component stack: ..."
// Fallback UI is shown; rest of the app keeps working
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 16. Rethrowing Errors

A `catch` block can inspect an error and rethrow it if it does not know how to handle it. This ensures only expected errors are handled at a given level and unexpected ones bubble up.

```js
class ValidationError extends Error {}
class NetworkError extends Error {}

function process(input) {
  try {
    if (!input) throw new ValidationError('Input required');
    if (input === 'network-fail') throw new NetworkError('Connection failed');
    return `Processed: ${input}`;
  } catch (e) {
    if (e instanceof ValidationError) {
      console.log('Handling validation error:', e.message);
      return null; // handled — do not rethrow
    }
    throw e; // rethrow anything we don't handle
  }
}

try {
  process(null);         // ValidationError — handled
  process('network-fail'); // NetworkError — rethrown
} catch (e) {
  console.log('Outer caught:', e.name, e.message);
}
```

### Output

```js
"Handling validation error: Input required"
"Outer caught: NetworkError Connection failed"
```

---

## 17. Nested try/catch

`try/catch` blocks can be nested. An inner `catch` handles errors first; if it rethrows, the outer `catch` handles the rethrown error.

```js
function inner() {
  try {
    throw new Error('original error');
  } catch (e) {
    console.log('inner catch:', e.message);
    throw new Error('wrapped error'); // throw a NEW error
  }
}

try {
  inner();
} catch (e) {
  console.log('outer catch:', e.message);
} finally {
  console.log('outer finally');
}
```

### Output

```js
"inner catch: original error"
"outer catch: wrapped error"
"outer finally"
```

---

## 18. TypeError vs ReferenceError

These are the two most commonly confused error types.

| Scenario | Error Type |
|---|---|
| Calling a non-function | TypeError |
| Accessing property of `null` or `undefined` | TypeError |
| Calling a method that does not exist on a value | TypeError |
| Accessing a variable that was never declared | ReferenceError |
| Using a variable before its `let`/`const` declaration (TDZ) | ReferenceError |

```js
// TypeError examples
try { null.prop; }          catch (e) { console.log('1:', e.name); }
try { undefined(); }        catch (e) { console.log('2:', e.name); }
try { (1).toUpperCase(); }  catch (e) { console.log('3:', e.name); }

// ReferenceError examples
try { neverDeclared; }      catch (e) { console.log('4:', e.name); }
try { (() => { console.log(x); let x = 1; })(); }
                            catch (e) { console.log('5:', e.name); }
```

### Output

```js
"1: TypeError"
"2: TypeError"
"3: TypeError"
"4: ReferenceError"
"5: ReferenceError"
```

---

## 19. finally with return Value Gotcha

If `finally` contains a `return` statement, it **overrides** any `return` in `try` or `catch` — and also **suppresses** any thrown exception from `try` or `catch`.

```js
// Case 1: finally return overrides try return
function f1() {
  try {
    return 'try';
  } finally {
    return 'finally'; // overrides 'try'
  }
}
console.log(f1()); // "finally"

// Case 2: finally return suppresses a thrown error
function f2() {
  try {
    throw new Error('from try');
  } finally {
    return 'from finally'; // silently swallows the error!
  }
}
console.log(f2()); // "from finally" — no error thrown

// Case 3: finally without return — try's return is used
function f3() {
  try {
    return 'try';
  } finally {
    console.log('finally runs'); // no return
  }
}
console.log(f3()); // logs "finally runs", then "try"
```

### Output

```js
"finally"       // f1
"from finally"  // f2  — the Error was silently discarded
"finally runs"  // f3 finally block
"try"           // f3 return value
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 20. Summary

| Topic | Key Point |
|---|---|
| Error types | SyntaxError, ReferenceError, TypeError, RangeError all extend Error |
| try/catch/finally | catch runs on throw; finally ALWAYS runs regardless |
| finally | Runs on normal exit, throw, and return |
| Error object | Properties: name, message, stack |
| throw | Throw Error objects, not primitives; gives you a stack trace |
| Custom errors | Extend Error, set `this.name`, add custom properties |
| Error chaining | `new Error('msg', { cause: originalErr })` — preserves root cause |
| Synchronous errors | Caught by try/catch; uncaught ones crash the program |
| Promise .catch | Handles rejections; resumes the chain after returning a value |
| Unhandled rejection | Fires `unhandledrejection` event; can terminate Node.js process |
| async/await errors | Use try/catch — catches awaited rejections and thrown errors |
| Forgetting await | Rejected promise is unhandled; try/catch does not intercept it |
| Global handlers | `window.onerror`, `window.onunhandledrejection` — last resort logging |
| Error Boundaries | React-specific; class components that catch render-phase errors |
| Rethrowing | Check error type in catch; rethrow if not yours to handle |
| TypeError | Wrong type: null access, calling non-function |
| ReferenceError | Undeclared variable or temporal dead zone access |
| finally + return | finally return overrides try/catch return and suppresses thrown errors |

---

## Final Notes

Robust error handling separates production-quality code from fragile scripts. Use specific error types and custom error classes to make catch blocks intentional. Always handle promise rejections — unhandled ones silently fail or crash in Node.js. The single most dangerous anti-pattern is a `return` statement inside `finally`: it silently suppresses exceptions and overrides return values, causing subtle and hard-to-debug issues.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
