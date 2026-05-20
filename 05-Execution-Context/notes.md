# Execution Context in JavaScript

## Table of Contents

1. [What is Execution Context?](#1-what-is-execution-context)
2. [Types of Execution Context](#2-types-of-execution-context)
3. [Global Execution Context](#3-global-execution-context)
4. [Function Execution Context](#4-function-execution-context)
5. [Call Stack](#5-call-stack)
6. [Stack Overflow](#6-stack-overflow)
7. [Memory Creation Phase (Creation Phase)](#7-memory-creation-phase-creation-phase)
8. [Code Execution Phase](#8-code-execution-phase)
9. [Variable Environment](#9-variable-environment)
10. [Lexical Environment](#10-lexical-environment)
11. [Outer Environment Reference](#11-outer-environment-reference)
12. [Scope Chain in Execution Context](#12-scope-chain-in-execution-context)
13. [Execution Context Lifecycle](#13-execution-context-lifecycle)
14. [How Functions Are Called (push/pop on call stack)](#14-how-functions-are-called-pushpop-on-call-stack)
15. [Nested Function Calls](#15-nested-function-calls)
16. [Execution Context and Closures](#16-execution-context-and-closures)
17. [`this` in Execution Context](#17-this-in-execution-context)
18. [eval() and Execution Context](#18-eval-and-execution-context)
19. [Visual Representation of Call Stack](#19-visual-representation-of-call-stack)
20. [Summary Table](#20-summary-table)

---

## 1. What is Execution Context?

An **Execution Context (EC)** is the environment in which JavaScript code is evaluated and executed. Every time JavaScript runs any code — global code, a function, or `eval` — it creates a corresponding execution context that tracks:

- The variables and functions available (Variable/Lexical Environment)
- The value of `this`
- A reference to the outer scope (outer environment)

Think of an execution context as a "box" that wraps all the information needed to run a particular piece of code.

```js
var globalVar = "I am global";

function greet(name) {
  var message = "Hello, " + name;
  console.log(message);
}

greet("World");
```

### Output

```js
Hello, World
```

When this script runs:
1. A **Global EC** is created for `var globalVar` and the function declaration `greet`.
2. When `greet("World")` is called, a **Function EC** is created for it.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 2. Types of Execution Context

JavaScript has three types of execution context:

| Type | When Created | Notes |
|---|---|---|
| **Global EC** | Once, when the script starts | One per script/module; creates the global object and `this` |
| **Function EC** | Every time a function is **called** | A new EC is created for each invocation |
| **Eval EC** | When `eval()` is called | Rarely used; executes code string in its own context |

```js
var x = 1; // Global EC

function outer() {    // Function EC created when called
  var y = 2;
  function inner() { // Another Function EC created when called
    var z = 3;
    console.log(x, y, z);
  }
  inner();
}

outer();
```

### Output

```js
1 2 3
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 3. Global Execution Context

The **Global Execution Context** (Global EC) is the default, outermost context. It is created before any code runs and performs two important tasks:

1. Creates the **global object** (`window` in browsers, `global` in Node.js).
2. Sets `this` to the global object.

It also goes through the two phases (memory creation + code execution) for all top-level code.

```js
var a = 10;
let b = 20;

function add(x, y) {
  return x + y;
}

console.log(add(a, b));
```

### Output

```js
30
```

During the Global EC's **memory phase**, `a` is initialized to `undefined`, `b` is placed in the temporal dead zone, and `add` is hoisted as a full function. During the **execution phase**, actual values are assigned and `add(10, 20)` is called.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 4. Function Execution Context

A **Function Execution Context** is created every time a function is **invoked** (not when it is defined). Each function call gets its own fresh EC containing its local variables, arguments, and its own `this` value.

```js
function multiply(a, b) {
  var result = a * b;
  console.log(result);
}

multiply(3, 4); // EC 1 created and destroyed
multiply(5, 6); // EC 2 created and destroyed
```

### Output

```js
12
30
```

Each call creates an independent EC. Variables inside one call do not interfere with another. After the function returns, its EC is popped off the call stack and garbage collected (unless a closure holds a reference).

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 5. Call Stack

The **Call Stack** is a LIFO (Last In, First Out) data structure that JavaScript uses to track which execution context is currently running. When a function is called, its EC is **pushed** onto the stack. When the function returns, its EC is **popped** off.

```js
function third() {
  console.log("In third");
}

function second() {
  console.log("In second - before third");
  third();
  console.log("In second - after third");
}

function first() {
  console.log("In first - before second");
  second();
  console.log("In first - after second");
}

first();
```

### Output

```js
In first - before second
In second - before third
In third
In second - after third
In first - after second
```

Call stack sequence:
```
[Global EC]
[Global EC] → [first EC]
[Global EC] → [first EC] → [second EC]
[Global EC] → [first EC] → [second EC] → [third EC]
[Global EC] → [first EC] → [second EC]   (third popped)
[Global EC] → [first EC]                 (second popped)
[Global EC]                              (first popped)
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 6. Stack Overflow

A **Stack Overflow** occurs when the call stack grows beyond its maximum size. This typically happens with **unbounded recursion** — a function that calls itself without a proper base case, causing an infinite chain of EC pushes.

```js
function recurse() {
  return recurse(); // no base case — infinite recursion
}

try {
  recurse();
} catch (e) {
  console.log(e instanceof RangeError); // true
  console.log(e.message);
}
```

### Output

```js
true
Maximum call stack size exceeded
```

```js
// Correct recursion with a base case
function factorial(n) {
  if (n <= 1) return 1;          // base case — stops recursion
  return n * factorial(n - 1);   // recursive call
}

console.log(factorial(5));
```

### Output

```js
120
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 7. Memory Creation Phase (Creation Phase)

Before any code in an execution context runs, JavaScript first goes through the **Memory Creation Phase** (also called the Creation Phase). During this phase:

- **`var` declarations** are hoisted and initialized to `undefined`.
- **Function declarations** are hoisted in their entirety (name + body).
- **`let` and `const` declarations** are hoisted but placed in the **Temporal Dead Zone (TDZ)** — accessing them before their declaration throws a `ReferenceError`.
- The value of `this` is determined.

```js
console.log(a);     // undefined (var hoisted)
console.log(b);     // ReferenceError (let in TDZ)
console.log(fn());  // "hello" (function declaration hoisted)

var a = 5;
let b = 10;

function fn() {
  return "hello";
}
```

### Output

```js
undefined
// ReferenceError: Cannot access 'b' before initialization
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 8. Code Execution Phase

After the memory creation phase, JavaScript enters the **Code Execution Phase**, where it runs the code line by line, assigning actual values to variables and invoking functions.

```js
var x;           // Memory phase: x = undefined
var y;           // Memory phase: y = undefined

x = 10;          // Execution phase: x = 10
y = 20;          // Execution phase: y = 20

console.log(x + y); // 30
```

### Output

```js
30
```

```js
// Demonstrating both phases
var num = 5;
console.log(num);   // 5 — value assigned before this line runs

function square(n) {
  var result = n * n;  // local var, memory phase inside function EC
  return result;
}

console.log(square(num)); // 25
```

### Output

```js
5
25
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 9. Variable Environment

The **Variable Environment** is a component of the execution context that stores all the `var` bindings and function declarations for that context. It is the memory store associated with the current EC.

- In the Global EC, it holds global `var` declarations and function declarations.
- In a Function EC, it holds the local `var` declarations, function declarations, and the `arguments` object.

```js
var globalA = "global";

function demo() {
  var localB = "local";
  console.log(globalA); // accessible via scope chain
  console.log(localB);  // accessible from this EC's variable environment
}

demo();
console.log(typeof localB); // "undefined" — not in global variable environment
```

### Output

```js
global
local
undefined
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 10. Lexical Environment

A **Lexical Environment** is a structure that holds identifier-to-value bindings for the current scope. It consists of two parts:

1. **Environment Record** — the actual storage of local variable and function bindings.
2. **Outer Environment Reference** — a pointer to the parent lexical environment.

In modern JavaScript (ES6+), `let` and `const` live in the lexical environment, while `var` lives in the variable environment. Together they form the full scope available to the code.

```js
let outerVal = "outer";

function outer() {
  let innerVal = "inner";

  function inner() {
    // inner's Lexical Environment: { innerVal: "inner" }
    // outer reference → outer's LE: { outerVal: "outer" }
    // outer reference → global LE: { outerVal: "outer" }
    console.log(outerVal); // found in outer's LE
    console.log(innerVal); // found in inner's own LE
  }

  inner();
}

outer();
```

### Output

```js
outer
inner
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 11. Outer Environment Reference

Each execution context (except the Global EC) has an **Outer Environment Reference** — a link to the lexical environment of its **enclosing scope** at the time the function was **defined** (not called). This chain of references forms the **scope chain**.

```js
var level = "global";

function first() {
  var level = "first";

  function second() {
    var level = "second";
    console.log(level); // "second" — found in own LE
  }

  function third() {
    // no local `level`
    console.log(level); // "first" — found via outer ref to first's LE
  }

  second();
  third();
}

first();
```

### Output

```js
second
first
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 12. Scope Chain in Execution Context

The **scope chain** is the series of outer environment references that connects each execution context back to the global context. When JavaScript looks up a variable, it walks this chain from innermost to outermost until it finds the variable or reaches the global EC (and throws a `ReferenceError` if not found).

Note: Full scope and hoisting details are covered in `02-Scope-Hoisting`. This section focuses on how the scope chain is physically implemented via execution contexts and lexical environment references.

```js
const A = "A";

function outerFn() {
  const B = "B";

  function middleFn() {
    const C = "C";

    function innerFn() {
      // Scope chain lookup order: innerFn LE → middleFn LE → outerFn LE → Global LE
      console.log(A, B, C);
    }

    innerFn();
  }

  middleFn();
}

outerFn();
```

### Output

```js
A B C
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 13. Execution Context Lifecycle

Every execution context goes through the same lifecycle:

1. **Creation Phase**
   - Variable Environment set up (`var` → `undefined`, functions → full definition, `let`/`const` → TDZ)
   - Lexical Environment set up
   - `this` value determined
2. **Execution Phase**
   - Code runs line by line
   - Variables receive actual values
   - Functions are invoked (creating new ECs)
3. **Destruction Phase**
   - EC is popped off the call stack
   - Local variables become eligible for garbage collection
   - Exception: closures retain a reference to the surrounding LE

```js
function lifecycle() {
  // Creation phase: result = undefined
  console.log(result); // undefined (hoisted)
  var result = "done";
  // Execution phase: result = "done"
  console.log(result); // done
  // Destruction: EC popped, result GC'd
}

lifecycle();
```

### Output

```js
undefined
done
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 14. How Functions Are Called (push/pop on call stack)

Every function invocation causes a new EC to be **pushed** onto the call stack. When the function finishes (returns or reaches its end), its EC is **popped**. The engine then resumes execution in the EC that is now on top of the stack.

```js
function a() {
  console.log("a start");
  b();
  console.log("a end");
}

function b() {
  console.log("b start");
  c();
  console.log("b end");
}

function c() {
  console.log("c");
}

a();
```

### Output

```js
a start
b start
c
b end
a end
```

Call stack trace:
```
push: Global EC
push: a EC
push: b EC
push: c EC
pop:  c EC (c returns)
pop:  b EC (b returns)
pop:  a EC (a returns)
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 15. Nested Function Calls

When functions are nested and called, each invocation creates its own EC. Each EC has its own memory space for local variables — they do not interfere with each other, even if functions share the same variable names.

```js
function add(a, b) {
  var sum = a + b;
  return sum;
}

function compute() {
  var x = add(2, 3); // EC for add(2,3) created and destroyed
  var y = add(4, 5); // EC for add(4,5) created and destroyed
  console.log(x, y, x + y);
}

compute();
```

### Output

```js
5 9 14
```

Each call to `add` gets its own EC with its own `a`, `b`, and `sum` variables. They are completely isolated.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 16. Execution Context and Closures

A **closure** is formed when an inner function retains a reference to the **lexical environment** of its outer function's EC — even after the outer function has returned and its EC has been popped off the call stack.

The inner function's outer environment reference keeps the outer EC's variable bindings alive in memory.

```js
function makeCounter() {
  let count = 0; // Lives in makeCounter's lexical environment

  return function increment() {
    count++; // increment retains a reference to makeCounter's LE
    console.log(count);
  };
}

const counter = makeCounter(); // makeCounter's EC is popped, but `count` lives on
counter(); // 1
counter(); // 2
counter(); // 3
```

### Output

```js
1
2
3
```

After `makeCounter()` returns, its EC is destroyed, but `count` is preserved because `increment` holds a reference to the lexical environment that contains it.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 17. `this` in Execution Context

The value of `this` is determined **during the creation phase** of an execution context and depends on how the code is invoked:

- **Global EC:** `this` is the global object (`window`/`global`) or `module.exports` in a Node.js module.
- **Function EC (regular call):** `this` is the global object (non-strict) or `undefined` (strict).
- **Function EC (method call):** `this` is the object to the left of the dot.
- **Function EC (`new` call):** `this` is the newly created instance.

```js
// Global EC
console.log(typeof this); // "object"

function regularFn() {
  "use strict";
  console.log(this); // undefined
}

const obj = {
  method() {
    console.log(this === obj); // true
  },
};

regularFn();
obj.method();
```

### Output

```js
object
undefined
true
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 18. eval() and Execution Context

`eval()` takes a string and executes it as JavaScript code. Depending on where it is called, it creates a new execution context (or re-uses the current one) and has access to the surrounding scope. Using `eval` is strongly discouraged in production code due to security risks and performance implications.

```js
var x = 10;

function demo() {
  var y = 20;
  eval("console.log(x + y)"); // access both outer and local variables
  eval("var z = 30");          // z is added to demo's variable environment
  console.log(z);
}

demo();
```

### Output

```js
30
30
```

```js
// eval in strict mode gets its own EC and cannot modify outer variables
"use strict";

function strictDemo() {
  eval("var local = 99");
  console.log(typeof local); // "undefined" — eval has its own scope in strict mode
}

strictDemo();
```

### Output

```js
undefined
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 19. Visual Representation of Call Stack

The following traces the call stack for a typical nested function scenario, showing which ECs are active at each step.

```js
function multiply(a, b) {
  return a * b;
}

function square(n) {
  return multiply(n, n);
}

function printSquare(n) {
  var result = square(n);
  console.log(result);
}

printSquare(4);
```

### Output

```js
16
```

**Call Stack Visualization:**

```
Step 1: Script starts
┌─────────────┐
│  Global EC  │  ← top of stack (printSquare, square, multiply defined)
└─────────────┘

Step 2: printSquare(4) called
┌──────────────────┐
│  printSquare EC  │  ← pushed
├──────────────────┤
│    Global EC     │
└──────────────────┘

Step 3: square(4) called inside printSquare
┌──────────────────┐
│    square EC     │  ← pushed
├──────────────────┤
│  printSquare EC  │
├──────────────────┤
│    Global EC     │
└──────────────────┘

Step 4: multiply(4, 4) called inside square
┌──────────────────┐
│   multiply EC    │  ← pushed
├──────────────────┤
│    square EC     │
├──────────────────┤
│  printSquare EC  │
├──────────────────┤
│    Global EC     │
└──────────────────┘

Step 5: multiply returns 16 → popped
Step 6: square returns 16 → popped
Step 7: printSquare logs 16, returns → popped
Step 8: Only Global EC remains
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 20. Summary Table

| Concept | Description |
|---|---|
| Execution Context (EC) | Environment containing variables, `this`, and outer reference for running code |
| Global EC | Created once on script start; holds global variables and sets `this` to global object |
| Function EC | Created on every function **call**; holds local vars, arguments, own `this` |
| Eval EC | Created by `eval()`; has access to surrounding scope (non-strict) |
| Call Stack | LIFO stack that tracks the currently active ECs |
| Stack Overflow | Error from unbounded recursion exhausting the call stack |
| Memory Creation Phase | Phase where `var` → `undefined`, functions hoisted fully, `let`/`const` → TDZ |
| Code Execution Phase | Phase where code runs line by line and variables get actual values |
| Variable Environment | Storage for `var` bindings and function declarations in a given EC |
| Lexical Environment | Environment record + outer reference; storage for `let`/`const` and scope chain link |
| Outer Environment Reference | Pointer to the enclosing lexical environment (set at **definition** time) |
| Scope Chain | Chain of outer references from inner EC to global EC used for variable lookup |
| Closure | Inner function retaining access to outer function's LE after outer EC is destroyed |
| `this` in EC | Determined during creation phase based on how the function was invoked |
| eval() | Executes code strings; discouraged due to security/perf issues |

---

## Final Notes

The execution context is the invisible engine behind every line of JavaScript you write. Understanding that code runs in two phases — memory creation and execution — immediately explains hoisting. Understanding the call stack explains why recursive functions can overflow and why asynchronous code (callbacks, promises) defers to the event loop instead of blocking the stack. Lexical environments and their outer references are the physical mechanism behind closures, which are one of JavaScript's most powerful features. Mastering execution context gives you a mental model that makes all other JavaScript concepts — scope, hoisting, closures, `this`, and async — fall neatly into place.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
