# JavaScript Fundamentals

> Core JavaScript concepts every developer must understand before moving to advanced topics.

---

# Table of Contents

1. [Introduction](#introduction)
2. [What is JavaScript?](#what-is-javascript)
3. [History of JavaScript](#history-of-javascript)
4. [JavaScript vs ECMAScript](#javascript-vs-ecmascript)
5. [Features of JavaScript](#features-of-javascript)
6. [How JavaScript Runs](#how-javascript-runs)
7. [JavaScript Engine](#javascript-engine)
8. [Variables](#variables)
9. [Data Types](#data-types)
10. [Operators](#operators)
11. [Type Conversion](#type-conversion)
12. [Truthy and Falsy Values](#truthy-and-falsy-values)
13. [Control Flow](#control-flow)
14. [Functions Basics](#functions-basics)
15. [Template Literals](#template-literals)
16. [Comments in JavaScript](#comments-in-javascript)
17. [Strict Mode](#strict-mode)
18. [Conclusion](#conclusion)

---

# Introduction

JavaScript is one of the most widely used programming languages in the world.

It is mainly used to build:

- Interactive websites
- Web applications
- Mobile applications
- Backend services
- Desktop applications

JavaScript works directly inside browsers and is also used outside browsers using environments like Node.js.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# What is JavaScript?

JavaScript is a:

- High-level programming language
- Dynamically typed language
- Interpreted language
- Lightweight scripting language
- Multi-paradigm language

JavaScript allows developers to:

- Add interactivity to web pages
- Handle user events
- Manipulate HTML and CSS
- Communicate with servers
- Build full-stack applications

Example:

```js
console.log("Hello JavaScript");
```

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# History of JavaScript

JavaScript was created by **Brendan Eich** in **1995**.

Initial names:

1. Mocha
2. LiveScript
3. JavaScript

JavaScript was developed in just **10 days**.

Later, JavaScript was standardized under:

- ECMAScript (ES)

Important versions:

| Version | Features |
|---|---|
| ES5 | Strict mode, JSON support |
| ES6 | let/const, arrow functions, classes |
| ES7+ | async/await, optional chaining |

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# JavaScript vs ECMAScript

| JavaScript | ECMAScript |
|---|---|
| Programming language | Specification/standard |
| Used by developers | Defines rules for JS |
| Runs in browsers | Defines language features |

ECMAScript defines the standard, while JavaScript is the implementation of that standard.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Features of JavaScript

## 1. Dynamic Typing

Variable types can change during execution.

```js
let value = 10;

value = "Hello";
```

---

## 2. Lightweight

JavaScript is designed to execute quickly inside browsers.

---

## 3. Cross Platform

Runs on:

- Browsers
- Servers
- Mobile devices
- Desktop applications

---

## 4. Event Driven

JavaScript reacts to user interactions like:

- Clicks
- Keyboard input
- Mouse events

---

## 5. Interpreted Language

JavaScript code executes directly without manual compilation.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# How JavaScript Runs

When JavaScript code executes:

1. Browser reads the code
2. JavaScript engine parses the code
3. Code gets converted into machine-readable instructions
4. Browser executes the code

Basic Flow:

```text
Code → Parse → Execute
```

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# JavaScript Engine

A JavaScript engine is responsible for executing JavaScript code.

Popular engines:

| Engine | Browser |
|---|---|
| V8 | Google Chrome |
| SpiderMonkey | Firefox |
| JavaScriptCore | Safari |

## Responsibilities of JS Engine

- Parsing code
- Optimizing code
- Executing code
- Managing memory

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Variables

Variables store data values.

JavaScript provides three keywords:

| Keyword | Scope | Reassign | Redeclare |
|---|---|---|---|
| var | Function | Yes | Yes |
| let | Block | Yes | No |
| const | Block | No | No |

Examples:

```js
var city = "Mumbai";

let age = 25;

const country = "India";
```

## Naming Rules

- Cannot start with numbers
- Cannot use reserved keywords
- Case-sensitive
- Can contain `_` and `$`

Valid:

```js
let userName;
let $price;
let _count;
```

Invalid:

```js
let 1name;
let var;
```

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Data Types

JavaScript data types are divided into two categories.

## Primitive Data Types

- String
- Number
- Boolean
- Undefined
- Null
- Symbol
- BigInt

Examples:

```js
let name = "John";

let age = 25;

let isAdmin = true;
```

---

## Non-Primitive Data Types

- Object
- Array
- Function

Example:

```js
const user = {
  name: "John",
  age: 25
};
```

---

## typeof Operator

Used to check data types.

```js
typeof "Hello"; // string

typeof 10; // number

typeof true; // boolean
```

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Operators

Operators perform operations on values.

## Arithmetic Operators

```js
+
-
*
/
%
**
```

Example:

```js
console.log(10 + 5);
```

---

## Comparison Operators

```js
==
===
!=
!==
>
<
>=
<=
```

Example:

```js
console.log(10 === 10);
```

---

## Logical Operators

```js
&&
||
!
```

Example:

```js
console.log(true && false);
```

---

## Assignment Operators

```js
=
+=
-=
*=
/=
```

Example:

```js
let x = 10;

x += 5;
```

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Type Conversion

JavaScript automatically or manually converts data types.

## Implicit Conversion

Automatic conversion by JavaScript.

```js
"5" + 1; // "51"
```

---

## Explicit Conversion

Manual conversion by developers.

```js
Number("10");

String(100);

Boolean(1);
```

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Truthy and Falsy Values

Values that become `false` in boolean context are called falsy values.

## Falsy Values

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
if ("Hello") {
  console.log("Truthy");
}
```

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Control Flow

Control flow determines program execution order.

## if else

```js
let age = 18;

if (age >= 18) {
  console.log("Adult");
} else {
  console.log("Minor");
}
```

---

## switch

```js
let day = 1;

switch (day) {
  case 1:
    console.log("Monday");
    break;

  default:
    console.log("Invalid");
}
```

---

## Loops

### for Loop

```js
for (let i = 0; i < 5; i++) {
  console.log(i);
}
```

### while Loop

```js
let i = 0;

while (i < 5) {
  console.log(i);
  i++;
}
```

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Functions Basics

Functions are reusable blocks of code.

## Function Declaration

```js
function greet() {
  console.log("Hello");
}

greet();
```

---

## Function with Parameters

```js
function add(a, b) {
  return a + b;
}

console.log(add(2, 3));
```

---

## Function Expression

```js
const greet = function () {
  console.log("Hello");
};
```

---

## Arrow Function

```js
const greet = () => {
  console.log("Hello");
};
```

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Template Literals

Template literals use backticks `` ` ` ``.

Example:

```js
const name = "John";

console.log(`Hello ${name}`);
```

Benefits:

- String interpolation
- Multi-line strings

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Comments in JavaScript

## Single Line Comment

```js
// This is a comment
```

---

## Multi Line Comment

```js
/*
  Multi-line
  comment
*/
```

Comments improve code readability.

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

- Prevents accidental mistakes
- Improves security
- Throws better errors

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# Conclusion

JavaScript fundamentals are the building blocks of advanced JavaScript concepts.

Strong understanding of fundamentals helps in:

- Problem solving
- Writing clean code
- Understanding frameworks
- Cracking interviews

Practice consistently and build small projects to strengthen concepts.

---

# Next Topics

- Scope & Hoisting
- Closures
- this Keyword
- Event Loop
- Promises
- Async/Await
- Objects & Prototypes

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>