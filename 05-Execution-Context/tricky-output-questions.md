# 🚀 JavaScript Execution Context — Tricky Output Questions

# 📚 Table of Contents

- [1. Global Execution Context](#1-global-execution-context)
- [2. Memory Creation Phase](#2-memory-creation-phase)
- [3. Code Execution Phase](#3-code-execution-phase)
- [4. Hoisting in Execution Context](#4-hoisting-in-execution-context)
- [5. Function Execution Context](#5-function-execution-context)
- [6. Scope Chain](#6-scope-chain)
- [7. Lexical Environment](#7-lexical-environment)
- [8. Variable Environment](#8-variable-environment)
- [9. Temporal Dead Zone](#9-temporal-dead-zone)
- [10. Call Stack](#10-call-stack)
- [11. Nested Execution Context](#11-nested-execution-context)
- [12. `this` Binding](#12-this-binding)
- [13. Closures and Execution Context](#13-closures-and-execution-context)
- [14. Strict Mode](#14-strict-mode)
- [15. Browser vs Node.js](#15-browser-vs-nodejs)
- [16. Mixed Concept Questions](#16-mixed-concept-questions)

---

# 1. Global Execution Context

---

## Question 1

```js
console.log(a);

var a = 10;
```

<details>
<summary>✅ Output</summary>

```txt
undefined
```

</details>

<details>
<summary>🧠 Explanation</summary>

During memory creation phase:

```txt
a -> undefined
```

So `a` exists before execution.

</details>

---

## Question 2

```js
console.log(a);
let a = 10;
```

<details>
<summary>✅ Output</summary>

```txt
ReferenceError
```

</details>

<details>
<summary>🧠 Explanation</summary>

`let` is hoisted but remains inside Temporal Dead Zone.

</details>

---

## Question 3

```js
console.log(this);
```

<details>
<summary>✅ Output</summary>

### Browser

```txt
window object
```

### Node.js

```txt
{}
```

or

```txt
module.exports
```

</details>

---

# 2. Memory Creation Phase

---

## Question 4

```js
console.log(a);
console.log(b);

var a = 1;
var b = 2;
```

<details>
<summary>✅ Output</summary>

```txt
undefined
undefined
```

</details>

---

## Question 5

```js
console.log(test);

function test() {}
```

<details>
<summary>✅ Output</summary>

```txt
[Function: test]
```

</details>

<details>
<summary>🧠 Explanation</summary>

Function declarations are fully hoisted.

</details>

---

## Question 6

```js
console.log(test);

var test = function () {};
```

<details>
<summary>✅ Output</summary>

```txt
undefined
```

</details>

<details>
<summary>🧠 Explanation</summary>

Only variable declaration is hoisted.

Function expression is not initialized during memory phase.

</details>

---

# 3. Code Execution Phase

---

## Question 7

```js
var a = 10;

console.log(a);

a = 20;

console.log(a);
```

<details>
<summary>✅ Output</summary>

```txt
10
20
```

</details>

---

## Question 8

```js
var a = 10;

function test() {
  console.log(a);
}

test();

var a = 20;
```

<details>
<summary>✅ Output</summary>

```txt
10
```

</details>

---

# 4. Hoisting in Execution Context

---

## Question 9

```js
foo();

function foo() {
  console.log("Hello");
}
```

<details>
<summary>✅ Output</summary>

```txt
Hello
```

</details>

---

## Question 10

```js
foo();

var foo = function () {
  console.log("Hello");
};
```

<details>
<summary>✅ Output</summary>

```txt
TypeError: foo is not a function
```

</details>

<details>
<summary>🧠 Explanation</summary>

During memory phase:

```txt
foo -> undefined
```

Then:

```js
foo()
```

becomes:

```js
undefined()
```

</details>

---

# 5. Function Execution Context

---

## Question 11

```js
function a() {
  console.log("A");
}

function b() {
  a();
  console.log("B");
}

b();
```

<details>
<summary>✅ Output</summary>

```txt
A
B
```

</details>

---

## Question 12

```js
function test() {
  var a = 10;

  console.log(a);
}

test();

console.log(a);
```

<details>
<summary>✅ Output</summary>

```txt
10
ReferenceError
```

</details>

---

# 6. Scope Chain

---

## Question 13

```js
var a = 1;

function outer() {
  var b = 2;

  function inner() {
    var c = 3;

    console.log(a, b, c);
  }

  inner();
}

outer();
```

<details>
<summary>✅ Output</summary>

```txt
1 2 3
```

</details>

---

## Question 14

```js
var a = 10;

function test() {
  console.log(a);

  var a = 20;
}

test();
```

<details>
<summary>✅ Output</summary>

```txt
undefined
```

</details>

<details>
<summary>🧠 Explanation</summary>

Inside function:

```txt
a -> undefined
```

Local variable shadows global variable.

</details>

---

# 7. Lexical Environment

---

## Question 15

```js
function outer() {
  let a = 10;

  function inner() {
    console.log(a);
  }

  return inner;
}

const fn = outer();

fn();
```

<details>
<summary>✅ Output</summary>

```txt
10
```

</details>

---

## Question 16

```js
let a = 1;

function x() {
  let a = 2;

  function y() {
    console.log(a);
  }

  y();
}

x();
```

<details>
<summary>✅ Output</summary>

```txt
2
```

</details>

---

# 8. Variable Environment

---

## Question 17

```js
var a = 1;

function test() {
  var a = 2;

  console.log(a);
}

test();

console.log(a);
```

<details>
<summary>✅ Output</summary>

```txt
2
1
```

</details>

---

# 9. Temporal Dead Zone

---

## Question 18

```js
{
  console.log(a);

  let a = 10;
}
```

<details>
<summary>✅ Output</summary>

```txt
ReferenceError
```

</details>

---

## Question 19

```js
{
  let a = 10;

  console.log(a);
}
```

<details>
<summary>✅ Output</summary>

```txt
10
```

</details>

---

## Question 20

```js
console.log(a);

const a = 100;
```

<details>
<summary>✅ Output</summary>

```txt
ReferenceError
```

</details>

---

# 10. Call Stack

---

## Question 21

```js
function one() {
  two();
  console.log("One");
}

function two() {
  three();
  console.log("Two");
}

function three() {
  console.log("Three");
}

one();
```

<details>
<summary>✅ Output</summary>

```txt
Three
Two
One
```

</details>

---

## Question 22

```js
function a() {
  console.log("A");
}

function b() {
  a();
}

function c() {
  b();
}

c();
```

<details>
<summary>✅ Output</summary>

```txt
A
```

</details>

---

# 11. Nested Execution Context

---

## Question 23

```js
function a() {
  console.log("A");

  function b() {
    console.log("B");
  }

  b();
}

a();
```

<details>
<summary>✅ Output</summary>

```txt
A
B
```

</details>

---

# 12. `this` Binding

---

## Question 24

```js
function test() {
  console.log(this);
}

test();
```

<details>
<summary>✅ Output</summary>

### Browser

```txt
window
```

### Strict Mode

```txt
undefined
```

</details>

---

## Question 25

```js
const obj = {
  name: "JS",
  show() {
    console.log(this.name);
  }
};

obj.show();
```

<details>
<summary>✅ Output</summary>

```txt
JS
```

</details>

---

## Question 26

```js
const obj = {
  name: "JS",
  show: () => {
    console.log(this.name);
  }
};

obj.show();
```

<details>
<summary>✅ Output</summary>

```txt
undefined
```

</details>

<details>
<summary>🧠 Explanation</summary>

Arrow functions don't have their own `this`.

</details>

---

# 13. Closures and Execution Context

---

## Question 27

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

<details>
<summary>✅ Output</summary>

```txt
1
2
3
```

</details>

---

## Question 28

```js
function test() {
  var a = 10;

  return function () {
    console.log(a);
  };
}

const fn = test();

fn();
```

<details>
<summary>✅ Output</summary>

```txt
10
```

</details>

---

# 14. Strict Mode

---

## Question 29

```js
"use strict";

function test() {
  console.log(this);
}

test();
```

<details>
<summary>✅ Output</summary>

```txt
undefined
```

</details>

---

## Question 30

```js
"use strict";

a = 10;

console.log(a);
```

<details>
<summary>✅ Output</summary>

```txt
ReferenceError
```

</details>

---

# 15. Browser vs Node.js

---

## Question 31

```js
var a = 10;

console.log(window.a);
```

<details>
<summary>✅ Output</summary>

### Browser

```txt
10
```

### Node.js

```txt
ReferenceError
```

</details>

---

## Question 32

```js
console.log(this);
```

<details>
<summary>✅ Output</summary>

### Browser

```txt
window
```

### Node.js

```txt
{}
```

</details>

---

# 16. Mixed Concept Questions

---

## Question 33

```js
var x = 1;

function a() {
  console.log(x);

  var x = 2;
}

a();
```

<details>
<summary>✅ Output</summary>

```txt
undefined
```

</details>

---

## Question 34

```js
var x = 1;

function a() {
  console.log(x);
}

function b() {
  var x = 10;

  a();
}

b();
```

<details>
<summary>✅ Output</summary>

```txt
1
```

</details>

<details>
<summary>🧠 Explanation</summary>

JavaScript uses lexical scope, not dynamic scope.

</details>

---

## Question 35

```js
function outer() {
  let x = 10;

  return function inner() {
    console.log(x);
  };
}

const fn1 = outer();
const fn2 = outer();

fn1();
fn2();
```

<details>
<summary>✅ Output</summary>

```txt
10
10
```

</details>

---

## Question 36

```js
var a = 10;

(function () {
  console.log(a);

  var a = 20;
})();
```

<details>
<summary>✅ Output</summary>

```txt
undefined
```

</details>

---

## Question 37

```js
let a = 10;

{
  console.log(a);

  let a = 20;
}
```

<details>
<summary>✅ Output</summary>

```txt
ReferenceError
```

</details>

---

## Question 38

```js
function test(a, b) {
  console.log(a);
  console.log(b);
}

test(1);
```

<details>
<summary>✅ Output</summary>

```txt
1
undefined
```

</details>

---

## Question 39

```js
function test() {
  console.log(a);

  if (true) {
    var a = 10;
  }
}

test();
```

<details>
<summary>✅ Output</summary>

```txt
undefined
```

</details>

---

## Question 40

```js
var a = 1;

function test() {
  console.log(a);

  a = 10;

  console.log(a);

  var a = 100;

  console.log(a);
}

test();

console.log(a);
```

<details>
<summary>✅ Output</summary>

```txt
undefined
10
100
1
```

</details>

<details>
<summary>🧠 Explanation</summary>

Inside function:

```txt
var a -> undefined
```

Local variable shadows global variable.

</details>

---