# 🚀 JavaScript Execution Context

Execution Context is one of the most important concepts in JavaScript.

Almost everything in JavaScript happens inside an execution context.

Understanding execution context helps you understand:

- How JavaScript code runs
- How variables are stored
- How functions execute
- How `this` works
- How closures work internally
- How hoisting actually works
- How the call stack operates

---

# 📚 Table of Contents

- [1. What is Execution Context?](#1-what-is-execution-context)
- [2. Why Execution Context Exists](#2-why-execution-context-exists)
- [3. Types of Execution Context](#3-types-of-execution-context)
- [4. Global Execution Context (GEC)](#4-global-execution-context-gec)
- [5. Function Execution Context (FEC)](#5-function-execution-context-fec)
- [6. Eval Execution Context](#6-eval-execution-context)
- [7. Components of Execution Context](#7-components-of-execution-context)
- [8. Memory Creation Phase](#8-memory-creation-phase)
- [9. Code Execution Phase](#9-code-execution-phase)
- [10. JavaScript Execution Flow](#10-javascript-execution-flow)
- [11. Variable Environment](#11-variable-environment)
- [12. Lexical Environment](#12-lexical-environment)
- [13. Scope Chain](#13-scope-chain)
- [14. Outer Environment Reference](#14-outer-environment-reference)
- [15. Thread of Execution](#15-thread-of-execution)
- [16. Call Stack](#16-call-stack)
- [17. Execution Context Lifecycle](#17-execution-context-lifecycle)
- [18. Synchronous Nature of JavaScript](#18-synchronous-nature-of-javascript)
- [19. Single Threaded Execution](#19-single-threaded-execution)
- [20. Function Invocation Process](#20-function-invocation-process)
- [21. Nested Execution Context](#21-nested-execution-context)
- [22. Execution Context vs Scope](#22-execution-context-vs-scope)
- [23. Execution Context vs Call Stack](#23-execution-context-vs-call-stack)
- [24. Execution Context and Hoisting](#24-execution-context-and-hoisting)
- [25. Execution Context and Closures](#25-execution-context-and-closures)
- [26. Execution Context and `this`](#26-execution-context-and-this)
- [27. Re-execution of Functions](#27-re-execution-of-functions)
- [28. Memory Allocation in Context](#28-memory-allocation-in-context)
- [29. Temporal Dead Zone in Context](#29-temporal-dead-zone-in-context)
- [30. Strict Mode Behavior](#30-strict-mode-behavior)
- [31. Browser vs Node.js Execution Context](#31-browser-vs-nodejs-execution-context)
- [32. Internal Representation](#32-internal-representation)
- [33. Common Misconceptions](#33-common-misconceptions)
- [34. Real-World Example](#34-real-world-example)
- [35. Deep Execution Flow Example](#35-deep-execution-flow-example)
- [36. Important Interview Points](#36-important-interview-points)
- [37. Summary](#37-summary)

---

# 1. What is Execution Context?

Execution Context is the environment where JavaScript code is evaluated and executed.

It contains everything needed to run code:

- Variables
- Functions
- Scope information
- `this` value
- References to outer scopes

Think of it as:

> A container where JavaScript code runs.

Every time JavaScript executes code, it creates an execution context.

---

# 2. Why Execution Context Exists

JavaScript needs a structured environment to:

- Store variables
- Track function calls
- Manage scope
- Determine `this`
- Execute code line by line

Without execution context:

- Variables couldn't exist
- Functions couldn't execute
- Scope couldn't work
- Closures wouldn't exist

---

# 3. Types of Execution Context

JavaScript has three types of execution contexts:

| Type | Description |
|------|-------------|
| Global Execution Context | Created for global code |
| Function Execution Context | Created whenever a function is invoked |
| Eval Execution Context | Created inside `eval()` |

---

# 4. Global Execution Context (GEC)

The Global Execution Context is created when JavaScript starts executing the file.

It is created only once.

---

## Example

```js
console.log("Start");
```

Before executing this code:

JavaScript creates:

- Global object
- `this`
- Memory space
- Scope chain

---

## Browser

In browsers:

```js
this === window // true
```

Global object:

```js
window
```

---

## Node.js

In Node.js:

```js
this !== global
```

Global object:

```js
global
```

---

# 5. Function Execution Context (FEC)

Whenever a function is called:

A new execution context is created.

---

## Example

```js
function greet() {
  console.log("Hello");
}

greet();
```

Steps:

1. Global execution context created
2. Function stored in memory
3. `greet()` invoked
4. New function execution context created
5. Function executes
6. Context removed from stack

---

# 6. Eval Execution Context

Created when code runs inside:

```js
eval()
```

Example:

```js
eval("console.log('Hi')");
```

Rarely used in modern JavaScript.

Avoid using `eval()`.

---

# 7. Components of Execution Context

Every execution context contains:

| Component | Purpose |
|-----------|---------|
| Memory Component | Stores variables/functions |
| Code Component | Executes code |
| Lexical Environment | Scope handling |
| Variable Environment | Variable storage |
| `this` Binding | Value of `this` |

---

# 8. Memory Creation Phase

Also called:

- Creation Phase
- Hoisting Phase

Before code executes:

JavaScript scans the code.

---

## During this phase

### Variables

```js
var a = 10;
```

Stored as:

```js
a: undefined
```

---

### Functions

Entire function stored in memory.

```js
function test() {}
```

Stored completely.

---

### let and const

Allocated memory but remain uninitialized.

They stay inside:

```txt
Temporal Dead Zone (TDZ)
```

---

## Example

```js
console.log(a);

var a = 10;
```

Memory phase:

```txt
a: undefined
```

Execution phase:

```txt
a = 10
```

---

# 9. Code Execution Phase

After memory creation:

JavaScript starts executing code line by line.

---

## Example

```js
var a = 10;

console.log(a);
```

Execution:

```txt
a = 10
print 10
```

---

# 10. JavaScript Execution Flow

JavaScript executes code in two phases:

| Phase | Work |
|------|------|
| Memory Creation Phase | Allocate memory |
| Code Execution Phase | Execute code |

---

## Example

```js
var a = 5;

function test() {
  console.log("Hello");
}

test();
```

---

## Memory Phase

```txt
a -> undefined
test -> function definition
```

---

## Execution Phase

```txt
a = 5
test() invoked
```

---

# 11. Variable Environment

Stores:

- Variables
- Function declarations

---

## Example

```js
var a = 10;

function test() {}
```

Variable environment contains:

```txt
a
test
```

---

# 12. Lexical Environment

Lexical Environment determines:

- Scope
- Accessibility of variables/functions

It contains:

- Local memory
- Reference to outer lexical environment

---

## Example

```js
function outer() {
  let a = 10;

  function inner() {
    console.log(a);
  }

  inner();
}

outer();
```

`inner()` accesses `a` using lexical environment.

---

# 13. Scope Chain

JavaScript searches variables in:

1. Current scope
2. Outer scope
3. Parent scope
4. Global scope

This process is called:

```txt
Scope Chain
```

---

## Example

```js
let a = 10;

function outer() {
  function inner() {
    console.log(a);
  }

  inner();
}

outer();
```

Search order:

```txt
inner -> outer -> global
```

---

# 14. Outer Environment Reference

Every lexical environment keeps reference to outer environment.

---

## Example

```js
function a() {
  function b() {
    function c() {
      console.log("Hi");
    }
  }
}
```

References:

```txt
c -> b -> a -> global
```

---

# 15. Thread of Execution

JavaScript executes one line at a time.

This is called:

```txt
Thread of Execution
```

---

# 16. Call Stack

Call Stack manages execution contexts.

Also called:

- Execution Stack
- Program Stack
- Runtime Stack

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
  console.log("Done");
}

one();
```

---

## Stack Flow

```txt
Global()
one()
two()
three()
```

After completion:

```txt
three removed
two removed
one removed
global remains
```

---

# 17. Execution Context Lifecycle

Lifecycle:

1. Creation
2. Execution
3. Destruction

---

## Example

```js
function test() {
  console.log("Hello");
}

test();
```

Lifecycle:

```txt
Create context
Execute code
Destroy context
```

---

# 18. Synchronous Nature of JavaScript

JavaScript is synchronous by default.

Meaning:

One operation executes at a time.

---

## Example

```js
console.log(1);
console.log(2);
console.log(3);
```

Output:

```txt
1
2
3
```

---

# 19. Single Threaded Execution

JavaScript has:

```txt
One Call Stack
```

Thus:

```txt
Single Threaded Language
```

Only one task executes at a time.

---

# 20. Function Invocation Process

When function is called:

1. New execution context created
2. Pushed into call stack
3. Executes
4. Removed after completion

---

## Example

```js
function greet() {
  console.log("Hello");
}

greet();
```

---

# 21. Nested Execution Context

Functions can create nested contexts.

---

## Example

```js
function a() {
  function b() {
    console.log("B");
  }

  b();
}

a();
```

Contexts:

```txt
Global -> a -> b
```

---

# 22. Execution Context vs Scope

| Execution Context | Scope |
|------------------|------|
| Runtime concept | Lexical concept |
| Created during execution | Defined during writing code |
| Stores execution info | Determines accessibility |

---

# 23. Execution Context vs Call Stack

| Execution Context | Call Stack |
|------------------|------------|
| Environment of execution | Structure managing contexts |
| Created per execution | Stores contexts |

---

# 24. Execution Context and Hoisting

Hoisting happens during:

```txt
Memory Creation Phase
```

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

Because:

```txt
a -> undefined
```

during creation phase.

---

# 25. Execution Context and Closures

Closures depend on lexical environment.

---

## Example

```js
function outer() {
  let count = 0;

  return function inner() {
    count++;
    console.log(count);
  };
}

const fn = outer();

fn();
fn();
```

Output:

```txt
1
2
```

Why?

Because `inner()` remembers outer lexical environment.

---

# 26. Execution Context and `this`

`this` is determined during execution context creation.

---

## Global Context

Browser:

```js
console.log(this === window);
```

---

## Function Context

```js
function test() {
  console.log(this);
}

test();
```

Depends on:

- Strict mode
- Invocation style

---

# 27. Re-execution of Functions

Every function call creates a brand new execution context.

---

## Example

```js
function test() {
  let a = 0;
  a++;
  console.log(a);
}

test();
test();
```

Output:

```txt
1
1
```

Each call gets separate memory.

---

# 28. Memory Allocation in Context

Memory allocation differs for:

| Keyword | Behavior |
|---------|----------|
| var | Initialized with undefined |
| let | TDZ |
| const | TDZ |

---

# 29. Temporal Dead Zone in Context

TDZ exists between:

```txt
Memory allocation
and
Initialization
```

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

# 30. Strict Mode Behavior

Strict mode changes `this`.

---

## Example

```js
"use strict";

function test() {
  console.log(this);
}

test();
```

Output:

```txt
undefined
```

---

# 31. Browser vs Node.js Execution Context

| Feature | Browser | Node.js |
|---------|----------|---------|
| Global Object | window | global |
| Top-level `this` | window | module.exports |
| Environment | Browser APIs | Node APIs |

---

# 32. Internal Representation

Execution Context internally stores:

```txt
LexicalEnvironment
VariableEnvironment
ThisBinding
```

---

# 33. Common Misconceptions

---

## Misconception 1

### "Hoisting moves code"

Wrong.

Only declarations are stored during memory phase.

---

## Misconception 2

### "`let` is not hoisted"

Wrong.

`let` is hoisted but stays inside TDZ.

---

## Misconception 3

### "Call stack and execution context are same"

Wrong.

Call stack stores execution contexts.

---

# 34. Real-World Example

```js
var a = 10;

function outer() {
  var b = 20;

  function inner() {
    var c = 30;

    console.log(a, b, c);
  }

  inner();
}

outer();
```

---

## Execution Flow

### Global Context

```txt
a -> undefined
outer -> function
```

---

### Execute

```txt
a = 10
outer() called
```

---

### outer Context

```txt
b -> undefined
inner -> function
```

---

### inner Context

```txt
c -> undefined
```

---

### Scope Chain

```txt
inner -> outer -> global
```

---

# 35. Deep Execution Flow Example

```js
var x = 1;

function a() {
  var y = 2;

  function b() {
    var z = 3;

    console.log(x, y, z);
  }

  b();
}

a();
```

---

## Global Context

```txt
x -> undefined
a -> function
```

---

## Execution

```txt
x = 1
a() called
```

---

## a() Context

```txt
y -> undefined
b -> function
```

---

## b() Context

```txt
z -> undefined
```

---

## Variable Lookup

```txt
z -> local
y -> parent scope
x -> global scope
```

---

# 36. Important Interview Points

---

## Q1. What are the phases of execution context?

Two phases:

1. Memory Creation Phase
2. Code Execution Phase

---

## Q2. What is stored during memory creation?

- Variables
- Functions
- Scope references
- `this`

---

## Q3. Why does `var` print undefined?

Because during memory phase:

```txt
var -> undefined
```

---

## Q4. Why does `let` throw ReferenceError?

Because it stays inside:

```txt
Temporal Dead Zone
```

---

## Q5. Is JavaScript synchronous?

Yes.

JavaScript is synchronous and single-threaded by default.

---

## Q6. Does every function call create a new execution context?

Yes.

Every invocation creates a fresh context.

---

## Q7. What manages execution contexts?

```txt
Call Stack
```

---

## Q8. What happens after function execution?

Its execution context is removed from call stack.

---

# 37. Summary

- JavaScript runs inside execution contexts
- Global context is created first
- Every function call creates a new context
- Execution occurs in two phases:
  - Memory Creation
  - Code Execution
- Call stack manages contexts
- Lexical environment enables closures
- Scope chain handles variable lookup
- JavaScript is synchronous and single-threaded
- `this` is determined during context creation
- Closures work because lexical environments persist

---