# Scope & Hoisting — Tricky Output Questions

## Table of Contents

1. [Global Scope Questions](#1-global-scope-questions)
2. [Function Scope Questions](#2-function-scope-questions)
3. [Block Scope Questions](#3-block-scope-questions)
4. [Temporal Dead Zone Questions](#4-temporal-dead-zone-questions)
5. [Hoisting Questions](#5-hoisting-questions)
6. [Function Hoisting Questions](#6-function-hoisting-questions)
7. [Shadowing Questions](#7-shadowing-questions)
8. [Loop Scope Questions](#8-loop-scope-questions)
9. [Execution Context Questions](#9-execution-context-questions)
10. [Advanced Edge Cases](#10-advanced-edge-cases)

---

## 1. Global Scope Questions

---

### Q1. What will be the output?

```js
var a = 10;

function test() {
  console.log(a);
}

test();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
10
```

### Explanation

Global variables are accessible inside functions.

</details>

---

### Q2. What will be the output?

```js
let a = 20;

{
  console.log(a);
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
20
```

### Explanation

Block can access outer scope variables.

</details>

---

### Q3. What will be the output?

```js
var a = 10;

{
  var a = 20;
}

console.log(a);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
20
```

### Explanation

`var` is NOT block scoped.

</details>

---

## 2. Function Scope Questions

---

### Q4. What will be the output?

```js
function test() {
  var x = 100;
}

console.log(x);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
ReferenceError
```

### Explanation

`x` is function scoped.

</details>

---

### Q5. What will be the output?

```js
function test() {
  if (true) {
    var a = 10;
  }

  console.log(a);
}

test();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
10
```

### Explanation

`var` ignores block scope.

</details>

---

### Q6. What will be the output?

```js
function test() {
  if (true) {
    let a = 10;
  }

  console.log(a);
}

test();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
ReferenceError
```

### Explanation

`let` is block scoped.

</details>

---

## 3. Block Scope Questions

---

### Q7. What will be the output?

```js
{
  let a = 10;
  const b = 20;
}

console.log(a);
console.log(b);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
ReferenceError
ReferenceError
```

### Explanation

Both `let` and `const` are block scoped.

</details>

---

### Q8. What will be the output?

```js
{
  var x = 50;
}

console.log(x);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
50
```

### Explanation

`var` escapes block scope.

</details>

---

### Q9. What will be the output?

```js
let a = 1;

{
  let a = 2;

  {
    let a = 3;

    console.log(a);
  }

  console.log(a);
}

console.log(a);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
3
2
1
```

### Explanation

Each block creates separate scope.

</details>

---

## 4. Temporal Dead Zone Questions

---

### Q10. What will be the output?

```js
console.log(a);

let a = 10;
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
ReferenceError
```

### Explanation

`a` is inside TDZ before initialization.

</details>

---

### Q11. What will be the output?

```js
console.log(a);

var a = 10;
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
undefined
```

### Explanation

`var` gets initialized with `undefined`.

</details>

---

### Q12. What will be the output?

```js
{
  console.log(a);

  let a = 100;
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
ReferenceError
```

### Explanation

TDZ exists inside block scope too.

</details>

---

## 5. Hoisting Questions

---

### Q13. What will be the output?

```js
console.log(a);

var a = 5;

console.log(a);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
undefined
5
```

### Explanation

Declaration is hoisted, assignment is not.

</details>

---

### Q14. What will be the output?

```js
var a;

console.log(a);

a = 10;
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
undefined
```

</details>

---

### Q15. What will be the output?

```js
console.log(a);

const a = 100;
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
ReferenceError
```

</details>

---

## 6. Function Hoisting Questions

---

### Q16. What will be the output?

```js
greet();

function greet() {
  console.log("Hello");
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
Hello
```

### Explanation

Function declarations are fully hoisted.

</details>

---

### Q17. What will be the output?

```js
sayHi();

var sayHi = function () {
  console.log("Hi");
};
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
TypeError
```

### Explanation

`sayHi` becomes `undefined`.

</details>

---

### Q18. What will be the output?

```js
hello();

let hello = function () {
  console.log("Hello");
};
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
ReferenceError
```

### Explanation

`hello` is inside TDZ.

</details>

---

### Q19. What will be the output?

```js
test();

const test = () => {
  console.log("Arrow");
};
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
ReferenceError
```

</details>

---

## 7. Shadowing Questions

---

### Q20. What will be the output?

```js
let a = 10;

function test() {
  let a = 20;

  console.log(a);
}

test();

console.log(a);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
20
10
```

### Explanation

Inner variable shadows outer variable.

</details>

---

### Q21. What will be the output?

```js
var a = 10;

{
  let a = 20;

  console.log(a);
}

console.log(a);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
20
10
```

</details>

---

### Q22. What will be the output?

```js
let a = 10;

{
  console.log(a);

  let a = 20;
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
ReferenceError
```

### Explanation

Local `a` enters TDZ.

</details>

---

## 8. Loop Scope Questions

---

### Q23. What will be the output?

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => {
    console.log(i);
  }, 100);
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
3
3
3
```

### Explanation

All callbacks share same `i`.

</details>

---

### Q24. What will be the output?

```js
for (let i = 0; i < 3; i++) {
  setTimeout(() => {
    console.log(i);
  }, 100);
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
0
1
2
```

### Explanation

`let` creates new binding every iteration.

</details>

---

### Q25. What will be the output?

```js
for (var i = 1; i <= 3; i++) {
  ((j) => {
    setTimeout(() => {
      console.log(j);
    }, 100);
  })(i);
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
1
2
3
```

### Explanation

IIFE creates separate closure.

</details>

---

## 9. Execution Context Questions

---

### Q26. What will be the output?

```js
var x = 1;

function test() {
  console.log(x);

  var x = 2;
}

test();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
undefined
```

### Explanation

Local `x` is hoisted inside function.

</details>

---

### Q27. What will be the output?

```js
var x = 1;

function test() {
  console.log(x);

  x = 2;
}

test();

console.log(x);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
1
2
```

### Explanation

No local variable exists, so global variable is modified.

</details>

---

### Q28. What will be the output?

```js
function test() {
  console.log(a);

  var a = 10;

  console.log(a);
}

test();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
undefined
10
```

</details>

---

## 10. Advanced Edge Cases

---

### Q29. What will be the output?

```js
var a = 10;

function a() {
  console.log("Hello");
}

console.log(typeof a);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
number
```

### Explanation

Variable assignment overrides function reference.

</details>

---

### Q30. What will be the output?

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
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
undefined
```

### Explanation

`var a` is hoisted to function scope.

</details>

---

### Q31. What will be the output?

```js
function test() {
  console.log(a);

  if (true) {
    let a = 10;
  }
}

test();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
ReferenceError
```

### Explanation

`a` inside block enters TDZ.

</details>

---

### Q32. What will be the output?

```js
let a = 10;

function test() {
  console.log(a);

  let a = 20;
}

test();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
ReferenceError
```

### Explanation

Inner `a` shadows outer `a` and stays in TDZ.

</details>

---

### Q33. What will be the output?

```js
{
  function greet() {
    console.log("Hello");
  }
}

greet();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
Behavior may differ between environments
```

### Explanation

Block-level function declarations behave differently in browsers and strict mode.

</details>

---

### Q34. What will be the output?

```js
var a = 1;

function outer() {
  console.log(a);

  var a = 2;

  function inner() {
    console.log(a);
  }

  inner();
}

outer();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
undefined
2
```

### Explanation

Local `a` is hoisted inside `outer`.

</details>

---

### Q35. What will be the output?

```js
console.log(typeof x);

var x = 10;
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
undefined
```

### Explanation

`x` exists during hoisting with value `undefined`.

</details>

---

## Final Tips

For Scope & Hoisting output questions, always analyze:

1. Global vs Local Scope
2. `var` vs `let` vs `const`
3. Block Scope
4. Function Scope
5. Temporal Dead Zone (TDZ)
6. Hoisting Rules
7. Function Declaration vs Expression
8. Variable Shadowing
9. Scope Chain
10. Async Loop Behavior

These questions are extremely common in:

- JavaScript Interviews
- Frontend Interviews
- React Interviews
- Machine Coding Rounds
- Senior JavaScript Interviews

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>