# Scope & Hoisting in JavaScript

## Table of Contents

1. [What is Scope?](#1-what-is-scope)
2. [Why Scope Exists](#2-why-scope-exists)
3. [Types of Scope](#3-types-of-scope)
4. [Global Scope](#4-global-scope)
5. [Function Scope](#5-function-scope)
6. [Block Scope](#6-block-scope)
7. [Lexical Scope](#7-lexical-scope)
8. [Scope Chain](#8-scope-chain)
9. [Nested Scope](#9-nested-scope)
10. [Variable Shadowing](#10-variable-shadowing)
11. [Illegal Shadowing](#11-illegal-shadowing)
12. [Temporal Dead Zone (TDZ)](#12-temporal-dead-zone-tdz)
13. [What is Hoisting?](#13-what-is-hoisting)
14. [Hoisting with var](#14-hoisting-with-var)
15. [Hoisting with let & const](#15-hoisting-with-let--const)
16. [Function Hoisting](#16-function-hoisting)
17. [Function Expression Hoisting](#17-function-expression-hoisting)
18. [Arrow Function Hoisting](#18-arrow-function-hoisting)
19. [Class Hoisting](#19-class-hoisting)
20. [Hoisting Priority](#20-hoisting-priority)
21. [Scope in Loops](#21-scope-in-loops)
22. [Closures and Scope](#22-closures-and-scope)
23. [Strict Mode & Scope](#23-strict-mode--scope)
24. [Global Object Behavior](#24-global-object-behavior)
25. [Execution Context and Scope](#25-execution-context-and-scope)
26. [Memory Creation Phase](#26-memory-creation-phase)
27. [Common Interview Edge Cases](#27-common-interview-edge-cases)
28. [Best Practices](#28-best-practices)
29. [Summary](#29-summary)

---

## 1. What is Scope?

Scope determines:

- Where variables can be accessed
- Where functions can be accessed
- Lifetime of variables
- Visibility of identifiers

In simple words:

> Scope controls accessibility of variables and functions in different parts of code.

```js
let name = "JavaScript";

function greet() {
  console.log(name);
}

greet();
```

### Output

```js
JavaScript
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 2. Why Scope Exists

Scope helps:

- Avoid variable conflicts
- Protect data
- Prevent accidental modifications
- Organize memory efficiently
- Improve security

Without scope, everything would collide globally.

```js
var count = 1;
var count = 2;
var count = 1000;
```

Every reassignment overwrites previous value.

---

## 3. Types of Scope

JavaScript has mainly:

| Scope Type | Description |
|---|---|
| Global Scope | Accessible everywhere |
| Function Scope | Accessible only inside function |
| Block Scope | Accessible only inside block |
| Lexical Scope | Inner scope accesses outer scope |

---

## 4. Global Scope

Variables declared outside all functions/blocks belong to global scope.

```js
let city = "Mumbai";

function showCity() {
  console.log(city);
}

showCity();
```

### Output

```js
Mumbai
```

---

### Global Variables Problem

Too many global variables can:

- Pollute memory
- Cause naming conflicts
- Create bugs

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 5. Function Scope

Variables declared inside a function are only accessible inside that function.

```js
function test() {
  let age = 25;
  console.log(age);
}

test();
console.log(age);
```

### Output

```js
25
ReferenceError: age is not defined
```

---

### var is Function Scoped

```js
function demo() {
  var x = 10;
}

console.log(x);
```

### Output

```js
ReferenceError: x is not defined
```

---

## 6. Block Scope

A block means:

```js
{
}
```

Variables declared using `let` and `const` are block scoped.

```js
{
  let a = 10;
  const b = 20;
}

console.log(a);
console.log(b);
```

### Output

```js
ReferenceError: a is not defined
ReferenceError: b is not defined
```

---

### var Ignores Block Scope

```js
{
  var x = 100;
}

console.log(x);
```

### Output

```js
100
```

---

### if Block

```js
if (true) {
  let message = "Hello";
}

console.log(message);
```

### Output

```js
ReferenceError: message is not defined
```

---

### for Loop Block Scope

```js
for (let i = 0; i < 3; i++) {}

console.log(i);
```

### Output

```js
ReferenceError: i is not defined
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 7. Lexical Scope

Lexical means:

> Scope is determined by where code is written physically.

Inner functions can access outer variables.

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

### Output

```js
10
```

---

### Reverse is NOT Possible

Outer function cannot access inner variables.

```js
function outer() {
  function inner() {
    let secret = "hidden";
  }

  console.log(secret);
}

outer();
```

### Output

```js
ReferenceError: secret is not defined
```

---

## 8. Scope Chain

JavaScript searches variables in order:

1. Current scope
2. Parent scope
3. Global scope

This process is called Scope Chain.

```js
let globalVar = "Global";

function outer() {
  let outerVar = "Outer";

  function inner() {
    let innerVar = "Inner";

    console.log(innerVar);
    console.log(outerVar);
    console.log(globalVar);
  }

  inner();
}

outer();
```

### Output

```js
Inner
Outer
Global
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 9. Nested Scope

Functions inside functions create nested scope.

```js
function one() {
  let a = 1;

  function two() {
    let b = 2;

    function three() {
      let c = 3;

      console.log(a, b, c);
    }

    three();
  }

  two();
}

one();
```

### Output

```js
1 2 3
```

---

## 10. Variable Shadowing

Inner variable hides outer variable.

```js
let name = "Global";

function test() {
  let name = "Local";
  console.log(name);
}

test();
```

### Output

```js
Local
```

---

### Shadowing Example

```js
let a = 1;

function demo() {
  let a = 2;

  {
    let a = 3;
    console.log(a);
  }

  console.log(a);
}

demo();
console.log(a);
```

### Output

```js
3
2
1
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 11. Illegal Shadowing

Cannot shadow `let` with `var` in same scope.

```js
let a = 10;

{
  var a = 20;
}
```

### Output

```js
SyntaxError: Identifier 'a' has already been declared
```

---

### Valid Shadowing

```js
var a = 10;

{
  let a = 20;
}

console.log(a);
```

### Output

```js
10
```

---

## 12. Temporal Dead Zone (TDZ)

TDZ is the time between:

- entering scope
- and variable initialization

Accessing variable during TDZ causes error.

```js
console.log(a);

let a = 10;
```

### Output

```js
ReferenceError: Cannot access 'a' before initialization
```

---

### Why TDZ Exists

To prevent accidental access before initialization.

```js
console.log(pi);

const pi = 3.14;
```

### Output

```js
ReferenceError: Cannot access 'pi' before initialization
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 13. What is Hoisting?

Hoisting means:

> JavaScript moves declarations to the top during memory creation phase.

Only declarations are hoisted, not initializations.

---

## 14. Hoisting with var

```js
console.log(a);

var a = 10;
```

Internally becomes:

```js
var a;

console.log(a);

a = 10;
```

### Output

```js
undefined
```

---

### Important

`var` gets initialized with `undefined`.

---

## 15. Hoisting with let & const

```js
console.log(a);

let a = 10;
```

### Output

```js
ReferenceError: Cannot access 'a' before initialization
```

---

### Reason

They are hoisted but kept inside TDZ.

```js
const a;
```

### Output

```js
SyntaxError: Missing initializer in const declaration
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 16. Function Hoisting

Function declarations are fully hoisted.

```js
greet();

function greet() {
  console.log("Hello");
}
```

### Output

```js
Hello
```

---

## 17. Function Expression Hoisting

```js
sayHi();

var sayHi = function () {
  console.log("Hi");
};
```

### Output

```js
TypeError: sayHi is not a function
```

---

### Why?

Internally:

```js
var sayHi = undefined;

sayHi();
```

`undefined` is not callable.

---

## 18. Arrow Function Hoisting

```js
hello();

const hello = () => {
  console.log("Hello");
};
```

### Output

```js
ReferenceError: Cannot access 'hello' before initialization
```

---

## 19. Class Hoisting

Classes are hoisted but stay in TDZ.

```js
const obj = new Person();

class Person {}
```

### Output

```js
ReferenceError: Cannot access 'Person' before initialization
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 20. Hoisting Priority

Priority order:

1. Function declarations
2. Variable declarations

```js
var a = 1;

function a() {
  console.log("hello");
}

console.log(a);
```

### Output

```js
1
```

---

## 21. Scope in Loops

### var in Loops

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => {
    console.log(i);
  }, 100);
}
```

### Output

```js
3
3
3
```

---

### let in Loops

```js
for (let i = 0; i < 3; i++) {
  setTimeout(() => {
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

## 22. Closures and Scope

Closures happen because of lexical scope.

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
fn();
```

### Output

```js
1
2
3
```

---

## 23. Strict Mode & Scope

```js
"use strict";

x = 10;
```

### Output

```js
ReferenceError: x is not defined
```

---

### Without Strict Mode

```js
x = 10;

console.log(x);
```

### Output

```js
10
```

---

## 24. Global Object Behavior

In browser:

```js
var a = 10;

console.log(window.a);
```

### Output

```js
10
```

---

### let and const

```js
let b = 20;

console.log(window.b);
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

## 25. Execution Context and Scope

Execution context contains:

- Variable Environment
- Scope Chain
- this keyword

---

### Phases

1. Memory Creation Phase
2. Code Execution Phase

---

## 26. Memory Creation Phase

Before execution:

- variables stored
- functions stored
- scope chain created

```js
console.log(a);

var a = 5;

function test() {}
```

Memory phase creates:

```js
a = undefined
test = function
```

### Output

```js
undefined
```

---

## 27. Common Interview Edge Cases

### Edge Case 1

```js
var a = 1;

function test() {
  console.log(a);

  var a = 2;
}

test();
```

### Output

```js
undefined
```

---

### Edge Case 2

```js
let a = 1;

{
  console.log(a);

  let a = 2;
}
```

### Output

```js
ReferenceError: Cannot access 'a' before initialization
```

---

### Edge Case 3

```js
function test() {
  console.log(a);

  if (true) {
    var a = 10;
  }
}

test();
```

### Output

```js
undefined
```

---

### Edge Case 4

```js
function test() {
  console.log(a);

  if (true) {
    let a = 10;
  }
}

test();
```

### Output

```js
ReferenceError: Cannot access 'a' before initialization
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 28. Best Practices

### Prefer let and const

Avoid `var`.

```js
let x = 10;
const y = 20;
```

---

### Minimize Global Variables

Too many globals create bugs.

---

### Use const by Default

```js
const PI = 3.14;
```

---

### Keep Scope Small

Smaller scope improves maintainability.

---

### Avoid Variable Shadowing

Can create confusion.

---

## 29. Summary

| Topic | Key Point |
|---|---|
| Scope | Controls accessibility |
| Global Scope | Accessible everywhere |
| Function Scope | var is function scoped |
| Block Scope | let/const are block scoped |
| Lexical Scope | Inner accesses outer |
| Scope Chain | JS searches parent scopes |
| Hoisting | Declarations moved to top |
| var Hoisting | Initialized with undefined |
| let/const Hoisting | In Temporal Dead Zone |
| Function Hoisting | Fully hoisted |
| Function Expression | Hoisted as var (undefined) |
| Class Hoisting | In Temporal Dead Zone |
| TDZ | Block-level scoped variables inaccessible before init |
| Variable Shadowing | Inner hides outer variable |
| Illegal Shadowing | Cannot shadow let with var |
| Closures | Functions remember scope |

---

## Final Notes

Understanding Scope & Hoisting is critical because:

- Closures depend on scope
- Async behavior depends on scope
- Memory optimization depends on scope
- Most tricky interview questions come from hoisting and scope behavior

Mastering these topics makes advanced JavaScript much easier.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
