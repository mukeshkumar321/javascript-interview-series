# JavaScript Fundamentals — Interview Questions & Answers 🚀

> Comprehensive JavaScript fundamental interview questions with in-depth explanations, edge cases, pitfalls, and real-world understanding.

---

# Table of Contents

1. [What is JavaScript?](#1-what-is-javascript)
2. [Difference Between JavaScript and ECMAScript](#2-difference-between-javascript-and-ecmascript)
3. [How JavaScript Works Internally](#3-how-javascript-works-internally)
4. [Interpreted vs Compiled Language](#4-interpreted-vs-compiled-language)
5. [Single Threaded Nature of JavaScript](#5-single-threaded-nature-of-javascript)
6. [Synchronous vs Asynchronous JavaScript](#6-synchronous-vs-asynchronous-javascript)
7. [What is the JavaScript Engine?](#7-what-is-the-javascript-engine)
8. [Execution Context](#8-execution-context)
9. [Call Stack](#9-call-stack)
10. [Memory Heap](#10-memory-heap)
11. [Hoisting](#11-hoisting)
12. [Temporal Dead Zone (TDZ)](#12-temporal-dead-zone-tdz)
13. [var vs let vs const](#13-var-vs-let-vs-const)
14. [Scope in JavaScript](#14-scope-in-javascript)
15. [Lexical Scope](#15-lexical-scope)
16. [Closures](#16-closures)
17. [Primitive vs Non-Primitive Data Types](#17-primitive-vs-non-primitive-data-types)
18. [Dynamic Typing](#18-dynamic-typing)
19. [Type Coercion](#19-type-coercion)
20. [Truthy and Falsy Values](#20-truthy-and-falsy-values)
21. [== vs ===](#21--vs-)
22. [null vs undefined](#22-null-vs-undefined)
23. [NaN in JavaScript](#23-nan-in-javascript)
24. [Object Basics](#24-object-basics)
25. [Arrays in JavaScript](#25-arrays-in-javascript)
26. [Functions in JavaScript](#26-functions-in-javascript)
27. [First-Class Functions](#27-first-class-functions)
28. [Higher-Order Functions](#28-higher-order-functions)
29. [Callback Functions](#29-callback-functions)
30. [IIFE](#30-iife)
31. [Arrow Functions](#31-arrow-functions)
32. [this Keyword](#32-this-keyword)
33. [Strict Mode](#33-strict-mode)
34. [Template Literals](#34-template-literals)
35. [Destructuring](#35-destructuring)
36. [Spread vs Rest Operator](#36-spread-vs-rest-operator)
37. [Optional Chaining](#37-optional-chaining)
38. [Nullish Coalescing Operator](#38-nullish-coalescing-operator)
39. [Short Circuit Evaluation](#39-short-circuit-evaluation)
40. [Pass by Value vs Pass by Reference](#40-pass-by-value-vs-pass-by-reference)
41. [Shallow Copy vs Deep Copy](#41-shallow-copy-vs-deep-copy)
42. [setTimeout Internals](#42-settimeout-internals)
43. [Event Loop](#43-event-loop)
44. [Microtask Queue vs Callback Queue](#44-microtask-queue-vs-callback-queue)
45. [Browser APIs](#45-browser-apis)
46. [Debouncing](#46-debouncing)
47. [Throttling](#47-throttling)
48. [Currying](#48-currying)
49. [Memoization](#49-memoization)
50. [Common JavaScript Interview Traps](#50-common-javascript-interview-traps)

---

# 1. What is JavaScript?

## Answer

JavaScript is a:

- High-level
- Interpreted/JIT compiled
- Dynamically typed
- Prototype-based
- Single-threaded
- Multi-paradigm programming language

It is mainly used for:

- Web development
- Server-side development
- Mobile apps
- Desktop apps
- Game development

---

## Key Features

| Feature | Description |
|---|---|
| Dynamic Typing | Variable types determined at runtime |
| Prototype-based | Inheritance via prototypes |
| Event-driven | Supports asynchronous programming |
| Single-threaded | Executes one task at a time |
| First-class functions | Functions treated like variables |

---

## Example

```js
let name = "Dilkhush";

function greet(user) {
  return `Hello ${user}`;
}

console.log(greet(name));
```

---

## Important Interview Points

### JavaScript is NOT Java

Common beginner confusion.

| JavaScript | Java |
|---|---|
| Scripting Language | Programming Language |
| Prototype-based | Class-based |
| Dynamic Typing | Static Typing |
| Browser Focused | JVM Based |

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 2. Difference Between JavaScript and ECMAScript

## Answer

### ECMAScript

ECMAScript is the standard/specification.

### JavaScript

JavaScript is the implementation of ECMAScript.

---

## Real World Analogy

| Term | Example |
|---|---|
| ECMAScript | Rules Book |
| JavaScript | Actual Player Following Rules |

---

## Example

ES6 introduced:

- let
- const
- arrow functions
- classes
- promises

JavaScript engines implemented them later.

---

## Important Interview Points

### ECMAScript Versions

| Version | Features |
|---|---|
| ES5 | Strict mode |
| ES6 | let/const/classes |
| ES7 | async/await |
| ES2020 | Optional chaining |

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 3. How JavaScript Works Internally

## Answer

JavaScript execution mainly involves:

1. Parsing
2. Compilation
3. Execution
4. Garbage Collection

---

## Internal Components

| Component | Responsibility |
|---|---|
| JS Engine | Executes code |
| Memory Heap | Stores data |
| Call Stack | Tracks function execution |
| Event Loop | Handles async tasks |

---

## Execution Flow

```txt
Code → Parse → AST → Compilation → Execution
```

---

## Important Interview Points

### JavaScript Uses JIT Compilation

Modern engines like V8 use:

- Interpreter
- Compiler
- Optimizer

Together for performance.

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 4. Interpreted vs Compiled Language

## Answer

### Interpreted

Code executed line by line.

### Compiled

Entire code converted into machine code before execution.

---

## JavaScript Reality

JavaScript is technically:

```txt
Interpreted + JIT Compiled
```

---

## Important Interview Points

### Why JIT?

Improves performance by compiling frequently executed code.

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 5. Single Threaded Nature of JavaScript

## Answer

JavaScript executes one task at a time using a single call stack.

---

## Example

```js
console.log(1);

while (true) {}

console.log(2);
```

`2` never executes because stack is blocked.

---

## Important Interview Points

### Then How Async Works?

Using:

- Browser APIs
- Callback Queue
- Event Loop

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 6. Synchronous vs Asynchronous JavaScript

## Synchronous

Tasks execute one after another.

```js
console.log(1);
console.log(2);
```

---

## Asynchronous

Does not block execution.

```js
setTimeout(() => {
  console.log("Async");
}, 1000);

console.log("Sync");
```

Output:

```txt
Sync
Async
```

---

## Important Interview Points

### Async ≠ Parallel

JavaScript itself is single-threaded.

Browser APIs handle async behavior.

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 7. What is the JavaScript Engine?

## Answer

A JavaScript engine executes JS code.

---

## Popular Engines

| Engine | Browser |
|---|---|
| V8 | Chrome |
| SpiderMonkey | Firefox |
| JavaScriptCore | Safari |

---

## V8 Internals

- Parser
- Ignition Interpreter
- TurboFan Optimizer
- Garbage Collector

---

## Important Interview Points

### Node.js Uses V8

Node.js runs JavaScript outside browsers using V8.

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 8. Execution Context

## Answer

Execution Context is the environment where code executes.

---

## Types

| Type | Description |
|---|---|
| Global | Default execution |
| Function | Created for each function |
| Eval | Rarely used |

---

## Phases

### 1. Creation Phase

- Memory allocation
- Hoisting occurs

### 2. Execution Phase

- Code executes line by line

---

## Example

```js
var a = 10;

function test() {
  var b = 20;
}

test();
```

Separate execution contexts created.

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 9. Call Stack

## Answer

Call Stack manages function execution order.

---

## Example

```js
function one() {
  two();
}

function two() {
  three();
}

function three() {
  console.log("Hello");
}

one();
```

Stack Flow:

```txt
one()
two()
three()
```

---

## Important Interview Points

### Stack Overflow

Occurs with excessive recursion.

```js
function test() {
  test();
}
```

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 10. Memory Heap

## Answer

Heap stores:

- Objects
- Arrays
- Functions

---

## Example

```js
const obj = {
  name: "JS"
};
```

Object stored in heap memory.

---

## Important Interview Points

### Garbage Collection

Unused memory automatically cleaned.

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 11. Hoisting

## Answer

Hoisting is JavaScript's behavior of moving declarations to the top during compilation.

---

## Example

```js
console.log(a);

var a = 10;
```

Output:

```txt
undefined
```

Equivalent:

```js
var a;

console.log(a);

a = 10;
```

---

## Important Interview Points

### Function Hoisting

```js
test();

function test() {
  console.log("Hello");
}
```

Works because functions are fully hoisted.

---

### Function Expression

```js
test();

var test = function () {};
```

Throws:

```txt
TypeError
```

---

### let and const Hoisting

They are hoisted but remain in TDZ.

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 12. Temporal Dead Zone (TDZ)

## Answer

TDZ is the time between variable hoisting and initialization.

---

## Example

```js
console.log(a);

let a = 10;
```

Output:

```txt
ReferenceError
```

---

## Important Interview Points

### Why TDZ Exists?

Prevents accidental access before initialization.

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 13. var vs let vs const

| Feature | var | let | const |
|---|---|---|---|
| Scope | Function | Block | Block |
| Reassign | Yes | Yes | No |
| Redeclare | Yes | No | No |
| Hoisted | Yes | Yes | Yes |
| TDZ | No | Yes | Yes |

---

## Example

```js
{
  var a = 1;
  let b = 2;
}

console.log(a);
console.log(b);
```

`b` throws ReferenceError.

---

## Important Interview Points

### const Object Mutation

```js
const obj = {
  name: "JS"
};

obj.name = "React";
```

Allowed because reference unchanged.

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 14. Scope in JavaScript

## Types of Scope

- Global Scope
- Function Scope
- Block Scope
- Module Scope

---

## Example

```js
{
  let a = 10;
}

console.log(a);
```

Throws error.

---

## Important Interview Points

### var Ignores Block Scope

```js
{
  var x = 10;
}

console.log(x);
```

Works.

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 15. Lexical Scope

## Answer

Inner functions can access outer function variables.

---

## Example

```js
function outer() {
  let name = "JS";

  function inner() {
    console.log(name);
  }

  inner();
}

outer();
```

---

## Important Interview Points

### Scope Chain

JavaScript searches variables outward until found.

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 16. Closures

## Answer

Closure is when a function remembers variables from its lexical scope even after outer function execution finishes.

---

## Example

```js
function counter() {
  let count = 0;

  return function () {
    count++;
    return count;
  };
}

const increment = counter();

console.log(increment());
console.log(increment());
```

---

## Real World Uses

- Data hiding
- Memoization
- Currying
- React hooks

---

## Important Interview Points

### Common Interview Trap

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => {
    console.log(i);
  }, 1000);
}
```

Output:

```txt
3
3
3
```

Because same `i` shared.

---

### Fix

```js
for (let i = 0; i < 3; i++) {
  setTimeout(() => {
    console.log(i);
  }, 1000);
}
```

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 17. Primitive vs Non-Primitive Data Types

## Primitive Types

- string
- number
- boolean
- undefined
- null
- bigint
- symbol

---

## Non-Primitive

- Object
- Array
- Function

---

## Important Interview Points

### typeof null

```js
typeof null
```

Returns:

```txt
object
```

Historical bug.

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 18. Dynamic Typing

## Answer

Variable types can change during runtime.

---

## Example

```js
let data = 10;

data = "Hello";
```

Allowed.

---

## Important Interview Points

### Flexibility vs Bugs

Dynamic typing increases flexibility but may cause runtime issues.

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 19. Type Coercion

## Answer

Automatic type conversion by JavaScript.

---

## Example

```js
"5" + 1
```

Output:

```txt
51
```

---

```js
"5" - 1
```

Output:

```txt
4
```

---

## Important Interview Points

### Weird Cases

```js
true + true
```

Output:

```txt
2
```

---

```js
[] + []
```

Output:

```txt
""
```

---

```js
[] + {}
```

Output:

```txt
"[object Object]"
```

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 20. Truthy and Falsy Values

## Falsy Values

```txt
false
0
-0
0n
""
null
undefined
NaN
```

Everything else is truthy.

---

## Example

```js
if ("hello") {
  console.log("Truthy");
}
```

---

## Important Interview Points

### Empty Array is Truthy

```js
if ([]) {
  console.log("Yes");
}
```

Runs successfully.

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 21. == vs ===

## ==

Loose equality with coercion.

## ===

Strict equality without coercion.

---

## Example

```js
5 == "5"
```

true

---

```js
5 === "5"
```

false

---

## Important Interview Points

### Recommended

Always prefer `===`.

---

### Weird Comparisons

```js
[] == false
```

true

---

```js
null == undefined
```

true

---

```js
NaN == NaN
```

false

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 22. null vs undefined

| null | undefined |
|---|---|
| Intentional absence | Variable not assigned |
| Object type bug | Undefined type |

---

## Example

```js
let a;
console.log(a);
```

undefined

---

```js
let b = null;
```

Intentional empty value.

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 23. NaN in JavaScript

## Answer

NaN means:

```txt
Not a Number
```

But type is number.

---

## Example

```js
typeof NaN
```

Output:

```txt
number
```

---

## Important Interview Points

### Correct NaN Check

```js
Number.isNaN(value)
```

Preferred over global `isNaN()`.

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 24. Object Basics

## Answer

Objects store key-value pairs.

---

## Example

```js
const user = {
  name: "JS",
  age: 10
};
```

---

## Important Interview Points

### Object Keys are Strings

```js
const obj = {};

obj[1] = "one";

console.log(obj);
```

Key becomes `"1"`.

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 25. Arrays in JavaScript

## Important Interview Points

### Arrays are Objects

```js
typeof []
```

Returns:

```txt
object
```

---

### Sparse Arrays

```js
const arr = [1, , 3];
```

Creates empty slot.

---

### Array Length Behavior

```js
const arr = [1, 2];

arr.length = 0;

console.log(arr);
```

Clears array.

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 26. Functions in JavaScript

## Types

- Function Declaration
- Function Expression
- Arrow Function
- Anonymous Function

---

## Example

```js
function greet() {}

const hello = function () {};
```

---

## Important Interview Points

Functions are:

- First-class citizens
- Objects
- Callable

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 27. First-Class Functions

## Answer

Functions can be:

- Passed as arguments
- Returned from functions
- Assigned to variables

---

## Example

```js
function greet() {
  return function () {
    console.log("Hello");
  };
}
```

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 28. Higher-Order Functions

## Answer

Functions that:

- Take functions as arguments
- Return functions

---

## Examples

- map
- filter
- reduce

---

```js
const nums = [1, 2, 3];

nums.map(num => num * 2);
```

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 29. Callback Functions

## Answer

Function passed into another function.

---

## Example

```js
function fetchData(callback) {
  callback();
}
```

---

## Important Interview Points

### Callback Hell

Nested callbacks reduce readability.

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 30. IIFE

## Answer

Immediately Invoked Function Expression.

---

## Example

```js
(function () {
  console.log("IIFE");
})();
```

---

## Uses

- Avoid global pollution
- Create private scope

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 31. Arrow Functions

## Important Differences

| Feature | Normal Function | Arrow Function |
|---|---|---|
| this | Dynamic | Lexical |
| arguments | Available | Not available |
| constructor | Yes | No |

---

## Example

```js
const add = (a, b) => a + b;
```

---

## Important Interview Points

### Arrow Functions and this

```js
const obj = {
  name: "JS",
  greet: () => {
    console.log(this.name);
  }
};

obj.greet();
```

Output:

```txt
undefined
```

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 32. this Keyword

## Answer

`this` refers to execution context.

---

## Values Depend On

- How function called
- Strict mode
- Arrow function
- Object method
- Constructor

---

## Example

```js
const obj = {
  name: "JS",
  greet() {
    console.log(this.name);
  }
};
```

---

## Important Interview Points

### Global this

Browser:

```js
window
```

Node.js:

```js
global
```

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 33. Strict Mode

## Answer

Enables stricter parsing and error handling.

---

## Example

```js
"use strict";
```

---

## Benefits

- Prevents accidental globals
- Safer code
- Better optimization

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 34. Template Literals

## Example

```js
const name = "JS";

console.log(`Hello ${name}`);
```

---

## Benefits

- String interpolation
- Multi-line strings

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 35. Destructuring

## Example

```js
const user = {
  name: "JS",
  age: 10
};

const { name, age } = user;
```

---

## Array Destructuring

```js
const [a, b] = [1, 2];
```

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 36. Spread vs Rest Operator

## Spread

Expands values.

```js
const arr2 = [...arr1];
```

---

## Rest

Collects values.

```js
function test(...args) {}
```

---

## Important Interview Points

Both use same syntax:

```txt
...
```

Context matters.

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 37. Optional Chaining

## Example

```js
user?.address?.city
```

---

## Benefits

Avoids runtime errors for missing properties.

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 38. Nullish Coalescing Operator

## Example

```js
const value = null ?? "default";
```

---

## Difference From ||

`||` treats falsy values differently.

```js
0 || 10
```

Returns:

```txt
10
```

---

```js
0 ?? 10
```

Returns:

```txt
0
```

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 39. Short Circuit Evaluation

## OR

```js
true || console.log("Hi");
```

Second expression skipped.

---

## AND

```js
false && console.log("Hi");
```

Skipped.

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 40. Pass by Value vs Pass by Reference

## Primitive

Copied by value.

---

## Objects

Reference copied.

---

## Example

```js
const obj1 = {
  name: "JS"
};

const obj2 = obj1;

obj2.name = "React";

console.log(obj1.name);
```

Output:

```txt
React
```

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 41. Shallow Copy vs Deep Copy

## Shallow Copy

Copies first level only.

```js
const copy = { ...obj };
```

---

## Deep Copy

Nested objects copied fully.

```js
structuredClone(obj);
```

---

## Important Interview Points

### JSON Deep Copy Limitation

```js
JSON.parse(JSON.stringify(obj));
```

Fails for:

- functions
- undefined
- dates
- maps
- sets

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 42. setTimeout Internals

## Important Interview Points

Even with:

```js
setTimeout(fn, 0);
```

It does NOT execute immediately.

Because callback enters queue first.

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 43. Event Loop

## Answer

Event Loop continuously checks:

- Call Stack
- Task Queues

---

## Flow

```txt
Call Stack Empty?
↓
Move Queue Task
↓
Execute
```

---

## Important Interview Points

Microtasks execute before macrotasks.

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 44. Microtask Queue vs Callback Queue

## Microtasks

- Promise.then
- queueMicrotask

---

## Macrotasks

- setTimeout
- setInterval

---

## Example

```js
console.log(1);

setTimeout(() => console.log(2));

Promise.resolve().then(() => console.log(3));

console.log(4);
```

Output:

```txt
1
4
3
2
```

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 45. Browser APIs

## Examples

- DOM APIs
- Fetch API
- setTimeout
- LocalStorage

---

## Important Interview Points

Browser APIs are NOT part of JavaScript engine.

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 46. Debouncing

## Answer

Limits function execution until delay ends.

---

## Example Use Cases

- Search input
- Resize events

---

## Example

```js
function debounce(fn, delay) {
  let timer;

  return function (...args) {
    clearTimeout(timer);

    timer = setTimeout(() => {
      fn.apply(this, args);
    }, delay);
  };
}
```

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 47. Throttling

## Answer

Limits function execution frequency.

---

## Use Cases

- Scroll events
- Mouse movement

---

## Example

```js
function throttle(fn, delay) {
  let last = 0;

  return function (...args) {
    const now = Date.now();

    if (now - last >= delay) {
      last = now;
      fn.apply(this, args);
    }
  };
}
```

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 48. Currying

## Answer

Transforming function with multiple arguments into nested single-argument functions.

---

## Example

```js
function add(a) {
  return function (b) {
    return a + b;
  };
}
```

---

## Uses

- Functional programming
- Partial application

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 49. Memoization

## Answer

Caching function results for optimization.

---

## Example

```js
function memoize(fn) {
  const cache = {};

  return function (n) {
    if (cache[n]) {
      return cache[n];
    }

    cache[n] = fn(n);

    return cache[n];
  };
}
```

---

## Uses

- Expensive calculations
- React optimization

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 50. Common JavaScript Interview Traps

## Output Based Questions

---

### Question 1

```js
console.log(typeof null);
```

Output:

```txt
object
```

---

### Question 2

```js
console.log([] == false);
```

Output:

```txt
true
```

---

### Question 3

```js
console.log(NaN === NaN);
```

Output:

```txt
false
```

---

### Question 4

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i));
}
```

Output:

```txt
3 3 3
```

---

### Question 5

```js
console.log(0.1 + 0.2 === 0.3);
```

Output:

```txt
false
```

Because floating-point precision issue.

---

# Final Tips for Interviews 🚀

## Focus Areas

- Closures
- Event Loop
- this keyword
- Hoisting
- Scope
- Promises
- Async behavior
- Type coercion

---

## Most Asked Topics

| Topic | Frequency |
|---|---|
| Closures | Very High |
| Event Loop | Very High |
| this | Very High |
| Hoisting | High |
| Async JS | Very High |
| Scope | High |

---

# Conclusion

Mastering JavaScript fundamentals deeply is extremely important because:

- React depends heavily on JavaScript concepts
- Performance optimization requires JS internals understanding
- Advanced frontend interviews focus on edge cases and internals

Strong fundamentals make advanced concepts much easier.

---

<p align="right">
<a href="#table-of-contents">⬆ Back to Top</a>
</p>