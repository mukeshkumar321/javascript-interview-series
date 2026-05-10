# JavaScript Fundamentals

JavaScript Fundamentals are the core building blocks of the language.  
Before learning advanced topics like closures, async programming, React, Node.js, or performance optimization, every developer should have a strong understanding of these basics.

This section covers:

- Variables and Data Types
- Scope and Hoisting
- Operators
- Functions
- Arrays and Objects
- Loops and Conditions
- Type Conversion
- Truthy & Falsy Values
- Equality Comparisons
- Basic Memory Concepts
- Execution Context Basics

---

# Table of Contents

- [Why JavaScript Fundamentals Matter](#why-javascript-fundamentals-matter)
- [JavaScript Basics](#javascript-basics)
- [JavaScript Engine](#javascript-engine)
- [Is JavaScript Compiled or Interpreted?](#is-javascript-compiled-or-interpreted)
- [Strict Mode](#strict-mode)
- [Variables in JavaScript](#variables-in-javascript)
- [Difference Between var, let, and const](#difference-between-var-let-and-const)
- [Data Types](#data-types)
- [typeof Operator](#typeof-operator)
- [Hoisting](#hoisting)
- [Temporal Dead Zone (TDZ)](#temporal-dead-zone-tdz)
- [Scope](#scope)
- [Functions in JavaScript](#functions-in-javascript)
- [Parameters vs Arguments](#parameters-vs-arguments)
- [Operators](#operators)
- [== vs ===](#-vs-)
- [Truthy and Falsy Values](#truthy-and-falsy-values)
- [Conditionals](#conditionals)
- [Loops](#loops)
- [Arrays](#arrays)
- [Objects](#objects)
- [Destructuring](#destructuring)
- [Spread Operator](#spread-operator)
- [Rest Operator](#rest-operator)
- [Type Conversion](#type-conversion)
- [null vs undefined](#null-vs-undefined)
- [NaN](#nan)
- [Template Literals](#template-literals)
- [Optional Chaining](#optional-chaining)
- [Nullish Coalescing Operator](#nullish-coalescing-operator)
- [Execution Context](#execution-context-basic-idea)
- [Stack and Heap Memory](#stack-and-heap-memory)
- [Primitive vs Reference Types](#primitive-vs-reference-types)
- [Pass by Value vs Pass by Reference](#pass-by-value-vs-pass-by-reference)
- [Naming Conventions](#naming-conventions)
- [Best Practices](#best-practices)
- [Common Beginner Mistakes](#common-beginner-mistakes)
- [Summary](#summary)

---

# Why JavaScript Fundamentals Matter

Strong fundamentals help developers:

- Write clean and predictable code
- Debug issues faster
- Understand advanced concepts easily
- Perform better in interviews
- Build scalable frontend and backend applications

Most advanced JavaScript interview questions are built on top of fundamental concepts.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# JavaScript Basics

JavaScript is a:

- High-level programming language
- Interpreted language
- Single-threaded language
- Dynamically typed language
- Prototype-based language

JavaScript runs in:

- Browsers
- Servers using Node.js
- Mobile applications
- Desktop applications

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# JavaScript Engine

JavaScript code is executed by JavaScript engines.

Popular engines:

| Engine | Platform |
|---|---|
| V8 | Chrome, Node.js |
| SpiderMonkey | Firefox |
| JavaScriptCore | Safari |

The engine:

1. Parses code
2. Compiles code
3. Executes code
4. Performs memory management

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Is JavaScript Compiled or Interpreted?

JavaScript is technically both compiled and interpreted.

Modern JavaScript engines:

1. Parse the code
2. Compile it into machine code using Just-In-Time (JIT) compilation
3. Execute it immediately

This improves performance significantly.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Strict Mode

Strict mode helps write safer JavaScript.

```js
"use strict";
```

Benefits:

- Prevents accidental global variables
- Throws more errors
- Improves code quality
- Makes debugging easier

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Variables in JavaScript

Variables are used to store data.

JavaScript provides 3 ways to declare variables:

```js
var name = "John";
let age = 25;
const country = "India";
```

---

# Difference Between var, let, and const

| Feature | var | let | const |
|---|---|---|---|
| Scope | Function Scoped | Block Scoped | Block Scoped |
| Re-declaration | Allowed | Not Allowed | Not Allowed |
| Re-assignment | Allowed | Allowed | Not Allowed |
| Hoisted | Yes | Yes (TDZ) | Yes (TDZ) |
| Preferred Usage | Avoid | Use | Use by default |

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Data Types

JavaScript has two categories of data types.

## Primitive Data Types

These are immutable values.

```js
String
Number
Boolean
Undefined
Null
BigInt
Symbol
```

Example:

```js
const name = "JavaScript";
const age = 25;
const isDeveloper = true;
```

---

## Non-Primitive Data Types

These are reference types.

```js
Object
Array
Function
Date
Map
Set
```

Example:

```js
const user = {
  name: "John",
  age: 25
};
```

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# typeof Operator

Used to check data type.

```js
typeof "Hello"; // string
typeof 10; // number
typeof true; // boolean
typeof undefined; // undefined
typeof null; // object (historical bug)
typeof {}; // object
typeof []; // object
```

To properly check arrays:

```js
Array.isArray([]); // true
```

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Hoisting

Hoisting is JavaScript's default behavior of moving declarations to the top of their scope during compilation.

Example:

```js
console.log(a);

var a = 10;
```

Internally behaves like:

```js
var a;

console.log(a);

a = 10;
```

Output:

```js
undefined
```

---

# Temporal Dead Zone (TDZ)

Variables declared with `let` and `const` exist in a Temporal Dead Zone until initialization.

```js
console.log(a);

let a = 10;
```

Output:

```js
ReferenceError
```

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Scope

Scope determines where variables can be accessed.

## Global Scope

```js
const name = "John";
```

Accessible everywhere.

---

## Function Scope

```js
function test() {
  var age = 20;
}
```

Accessible only inside function.

---

## Block Scope

```js
{
  let city = "Mumbai";
}
```

Accessible only inside block.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Functions in JavaScript

Functions are reusable blocks of code.

## Function Declaration

```js
function greet() {
  return "Hello";
}
```

---

## Function Expression

```js
const greet = function () {
  return "Hello";
};
```

---

## Arrow Function

```js
const greet = () => {
  return "Hello";
};
```

Short version:

```js
const greet = () => "Hello";
```

---

# Parameters vs Arguments

```js
function add(a, b) {
  return a + b;
}

add(10, 20);
```

| Term | Value |
|---|---|
| Parameters | a, b |
| Arguments | 10, 20 |

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Operators

## Arithmetic Operators

```js
+
-
*
/
%
**
```

---

## Comparison Operators

```js
>
<
>=
<=
==
===
!=
!==
```

---

## Logical Operators

```js
&&
||
!
```

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# == vs ===

## Double Equals (`==`)

Performs type conversion before comparison.

```js
5 == "5"; // true
```

---

## Triple Equals (`===`)

Checks both value and type.

```js
5 === "5"; // false
```

Always prefer `===`.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Truthy and Falsy Values

## Falsy Values

Only these values are falsy:

```js
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

Example:

```js
if ("hello") {
  console.log("Truthy");
}
```

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Conditionals

## if Statement

```js
if (age >= 18) {
  console.log("Adult");
}
```

---

## if...else

```js
if (age >= 18) {
  console.log("Adult");
} else {
  console.log("Minor");
}
```

---

## Ternary Operator

```js
const result = age >= 18 ? "Adult" : "Minor";
```

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Loops

## for Loop

```js
for (let i = 0; i < 5; i++) {
  console.log(i);
}
```

---

## while Loop

```js
let i = 0;

while (i < 5) {
  console.log(i);
  i++;
}
```

---

## for...of

Used for iterable values.

```js
const arr = [1, 2, 3];

for (const value of arr) {
  console.log(value);
}
```

---

## for...in

Used for object keys.

```js
const user = {
  name: "John",
  age: 25
};

for (const key in user) {
  console.log(key);
}
```

Avoid using `for...in` directly on arrays because it iterates over keys.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Arrays

Arrays store multiple values.

```js
const fruits = ["Apple", "Banana", "Mango"];
```

---

## Common Array Methods

```js
push()
pop()
shift()
unshift()
map()
filter()
find()
reduce()
slice()
splice()
```

Example:

```js
const numbers = [1, 2, 3];

const doubled = numbers.map(num => num * 2);
```

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Objects

Objects store data in key-value pairs.

```js
const user = {
  name: "John",
  age: 25
};
```

Access values:

```js
user.name;
user["age"];
```

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Destructuring

## Array Destructuring

```js
const [a, b] = [1, 2];
```

---

## Object Destructuring

```js
const user = {
  name: "John",
  age: 25
};

const { name, age } = user;
```

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Spread Operator

```js
const arr1 = [1, 2];
const arr2 = [...arr1, 3, 4];
```

Objects:

```js
const user = {
  name: "John"
};

const updatedUser = {
  ...user,
  age: 25
};
```

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Rest Operator

```js
function sum(...numbers) {
  return numbers.reduce((acc, curr) => acc + curr, 0);
}
```

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Type Conversion

## Explicit Conversion

```js
Number("10");
String(100);
Boolean(1);
```

---

## Implicit Conversion (Coercion)

```js
"5" + 1; // "51"
"5" - 1; // 4
true + 1; // 2
```

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# null vs undefined

| null | undefined |
|---|---|
| Intentional absence of value | Variable declared but not assigned |
| Type is object | Type is undefined |

Example:

```js
let a;
console.log(a); // undefined

let b = null;
```

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# NaN

NaN means "Not a Number".

```js
Number("Hello"); // NaN
```

Check using:

```js
Number.isNaN(value);
```

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Template Literals

```js
const name = "John";

console.log(`Hello ${name}`);
```

Benefits:

- Multi-line strings
- String interpolation
- Cleaner syntax

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Optional Chaining

Prevents errors when accessing nested properties.

```js
const user = {
  profile: {
    name: "John"
  }
};

console.log(user?.profile?.name);
```

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Nullish Coalescing Operator

Returns right-side value only for `null` or `undefined`.

```js
const value = null ?? "Default";
```

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Execution Context (Basic Idea)

JavaScript creates an execution context to run code.

Two major phases:

1. Memory Creation Phase
2. Execution Phase

Understanding execution context helps in:

- Hoisting
- Closures
- Scope Chain
- `this` keyword

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Stack and Heap Memory

## Stack Memory

Stores:

- Primitive values
- Function calls

---

## Heap Memory

Stores:

- Objects
- Arrays
- Functions

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Primitive vs Reference Types

## Primitive

Copied by value.

```js
let a = 10;
let b = a;

b = 20;

console.log(a); // 10
```

---

## Reference Type

Copied by reference.

```js
const obj1 = {
  name: "John"
};

const obj2 = obj1;

obj2.name = "Doe";

console.log(obj1.name); // Doe
```

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Pass by Value vs Pass by Reference

## Pass by Value

Primitive values are copied independently.

```js
let a = 10;
let b = a;

b = 20;

console.log(a); // 10
```

---

## Pass by Reference

Objects are copied using references.

```js
const user1 = {
  name: "John"
};

const user2 = user1;

user2.name = "Doe";

console.log(user1.name); // Doe
```

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Important Note About const

`const` prevents reassignment, but object properties can still be modified.

```js
const user = {
  name: "John"
};

user.name = "Doe"; // Allowed
```

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Naming Conventions

## Variables

Use camelCase.

```js
const firstName = "John";
```

---

## Constants

Constants are often written in uppercase.

```js
const API_URL = "https://example.com";
```

---

## Classes

Use PascalCase.

```js
class UserProfile {}
```

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Best Practices

- Prefer `const` by default
- Use `let` when reassignment is needed
- Avoid `var`
- Always use `===`
- Write small reusable functions
- Use meaningful variable names
- Avoid global variables
- Keep code readable

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Common Beginner Mistakes

- Using `==` instead of `===`
- Confusing `null` and `undefined`
- Mutating objects accidentally
- Misunderstanding scope
- Forgetting return statements
- Overusing global variables
- Ignoring async behavior

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Summary

JavaScript fundamentals are the foundation of everything in JavaScript.

Mastering these concepts helps in:

- Frontend Development
- React
- Node.js
- Backend Development
- System Design
- Performance Optimization
- JavaScript Interviews

A strong understanding of fundamentals makes advanced JavaScript significantly easier to learn.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>