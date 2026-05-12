# Tricky Output Questions — Closures in JavaScript

<p align="right">
  <a href="#table-of-contents">⬇ Jump to Questions</a>
</p>

---

## Table of Contents

1. [Basic Closure](#1-basic-closure)
2. [Closure with Updated Variable](#2-closure-with-updated-variable)
3. [Multiple Closures](#3-multiple-closures)
4. [Closures in Loops with var](#4-closures-in-loops-with-var)
5. [Closures in Loops with let](#5-closures-in-loops-with-let)
6. [Closure with setTimeout](#6-closure-with-settimeout)
7. [Closure and Parameter](#7-closure-and-parameter)
8. [Closure Sharing Same Reference](#8-closure-sharing-same-reference)
9. [Independent Closures](#9-independent-closures)
10. [Nested Closures](#10-nested-closures)
11. [Closure with Global Variable](#11-closure-with-global-variable)
12. [Closure with Object Mutation](#12-closure-with-object-mutation)
13. [Closure and Reassignment](#13-closure-and-reassignment)
14. [Function Factory](#14-function-factory)
15. [Closure with Default Parameter](#15-closure-with-default-parameter)
16. [Closure and Block Scope](#16-closure-and-block-scope)
17. [Closure with IIFE](#17-closure-with-iife)
18. [Closure Inside Object](#18-closure-inside-object)
19. [Closure and Memory Reference](#19-closure-and-memory-reference)
20. [Closure with Async Callback](#20-closure-with-async-callback)
21. [Closure with var in Nested Loop](#21-closure-with-var-in-nested-loop)
22. [Closure with let in Nested Loop](#22-closure-with-let-in-nested-loop)
23. [Closure and Shadowing](#23-closure-and-shadowing)
24. [Closure with Returned Object](#24-closure-with-returned-object)
25. [Closure with Function Reassignment](#25-closure-with-function-reassignment)
26. [Closure with Delayed Mutation](#26-closure-with-delayed-mutation)
27. [Closure with Array Methods](#27-closure-with-array-methods)
28. [Closure with Private Counter](#28-closure-with-private-counter)
29. [Closure and Shared State](#29-closure-and-shared-state)
30. [Deep Nested Closure](#30-deep-nested-closure)

---

## 1. Basic Closure

```js
function outer() {
  let a = 10;

  return function() {
    console.log(a);
  };
}

const fn = outer();

fn();
```

<details>
<summary>👉 Show Output</summary>

```txt
10
```

### Explanation

Returned function remembers `a`.

</details>

---

## 2. Closure with Updated Variable

```js
function outer() {
  let count = 0;

  return function() {
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
<summary>👉 Show Output</summary>

```txt
1
2
3
```

### Explanation

Closure keeps same variable reference.

</details>

---

## 3. Multiple Closures

```js
function test() {
  let value = 5;

  return [
    function() {
      value++;
      console.log(value);
    },

    function() {
      value--;
      console.log(value);
    }
  ];
}

const [inc, dec] = test();

inc();
inc();
dec();
```

<details>
<summary>👉 Show Output</summary>

```txt
6
7
6
```

### Explanation

Both closures share same `value`.

</details>

---

## 4. Closures in Loops with var

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => {
    console.log(i);
  }, 100);
}
```

<details>
<summary>👉 Show Output</summary>

```txt
3
3
3
```

### Explanation

`var` is function scoped.

Single shared variable.

</details>

---

## 5. Closures in Loops with let

```js
for (let i = 0; i < 3; i++) {
  setTimeout(() => {
    console.log(i);
  }, 100);
}
```

<details>
<summary>👉 Show Output</summary>

```txt
0
1
2
```

### Explanation

`let` creates new binding each iteration.

</details>

---

## 6. Closure with setTimeout

```js
function greet(name) {
  setTimeout(() => {
    console.log(name);
  }, 100);
}

greet("JavaScript");
```

<details>
<summary>👉 Show Output</summary>

```txt
JavaScript
```

### Explanation

Callback remembers `name`.

</details>

---

## 7. Closure and Parameter

```js
function outer(x) {
  return function(y) {
    console.log(x + y);
  };
}

const fn = outer(10);

fn(5);
```

<details>
<summary>👉 Show Output</summary>

```txt
15
```

### Explanation

Inner function remembers `x`.

</details>

---

## 8. Closure Sharing Same Reference

```js
function counter() {
  let count = 0;

  return {
    inc() {
      count++;
    },

    log() {
      console.log(count);
    }
  };
}

const c = counter();

c.inc();
c.inc();
c.log();
```

<details>
<summary>👉 Show Output</summary>

```txt
2
```

### Explanation

Both methods share same closure state.

</details>

---

## 9. Independent Closures

```js
function createCounter() {
  let count = 0;

  return function() {
    count++;
    console.log(count);
  };
}

const a = createCounter();
const b = createCounter();

a();
a();
b();
```

<details>
<summary>👉 Show Output</summary>

```txt
1
2
1
```

### Explanation

Each function has separate closure memory.

</details>

---

## 10. Nested Closures

```js
function a(x) {
  return function(y) {
    return function(z) {
      console.log(x + y + z);
    };
  };
}

a(1)(2)(3);
```

<details>
<summary>👉 Show Output</summary>

```txt
6
```

### Explanation

Each function remembers outer variables.

</details>

---

## 11. Closure with Global Variable

```js
let value = 100;

function test() {
  return function() {
    console.log(value);
  };
}

const fn = test();

value = 200;

fn();
```

<details>
<summary>👉 Show Output</summary>

```txt
200
```

### Explanation

Closures store reference, not copy.

</details>

---

## 12. Closure with Object Mutation

```js
function outer() {
  const obj = { count: 1 };

  return function() {
    obj.count++;
    console.log(obj.count);
  };
}

const fn = outer();

fn();
fn();
```

<details>
<summary>👉 Show Output</summary>

```txt
2
3
```

### Explanation

Object reference is preserved.

</details>

---

## 13. Closure and Reassignment

```js
function outer() {
  let x = 10;

  const inner = function() {
    console.log(x);
  };

  x = 20;

  return inner;
}

const fn = outer();

fn();
```

<details>
<summary>👉 Show Output</summary>

```txt
20
```

### Explanation

Closure references updated variable.

</details>

---

## 14. Function Factory

```js
function multiply(x) {
  return function(y) {
    console.log(x * y);
  };
}

const double = multiply(2);

double(5);
```

<details>
<summary>👉 Show Output</summary>

```txt
10
```

### Explanation

Closure preserves multiplier.

</details>

---

## 15. Closure with Default Parameter

```js
function outer(x = 5) {
  return function() {
    console.log(x);
  };
}

const fn = outer();

fn();
```

<details>
<summary>👉 Show Output</summary>

```txt
5
```

### Explanation

Default parameter becomes part of closure.

</details>

---

## 16. Closure and Block Scope

```js
function outer() {
  {
    let x = 50;

    return function() {
      console.log(x);
    };
  }
}

const fn = outer();

fn();
```

<details>
<summary>👉 Show Output</summary>

```txt
50
```

### Explanation

Closure preserves block scoped variable.

</details>

---

## 17. Closure with IIFE

```js
const fn = (function() {
  let count = 0;

  return function() {
    count++;
    console.log(count);
  };
})();

fn();
fn();
```

<details>
<summary>👉 Show Output</summary>

```txt
1
2
```

### Explanation

IIFE creates private scope.

</details>

---

## 18. Closure Inside Object

```js
function test() {
  let secret = "JS";

  return {
    getSecret() {
      console.log(secret);
    }
  };
}

const obj = test();

obj.getSecret();
```

<details>
<summary>👉 Show Output</summary>

```txt
JS
```

### Explanation

Method closes over `secret`.

</details>

---

## 19. Closure and Memory Reference

```js
function outer() {
  let arr = [1, 2];

  return function() {
    arr.push(3);
    console.log(arr);
  };
}

const fn = outer();

fn();
fn();
```

<details>
<summary>👉 Show Output</summary>

```txt
[1, 2, 3]
[1, 2, 3, 3]
```

### Explanation

Same array reference is reused.

</details>

---

## 20. Closure with Async Callback

```js
function test() {
  let value = 10;

  Promise.resolve().then(() => {
    console.log(value);
  });

  value = 20;
}

test();
```

<details>
<summary>👉 Show Output</summary>

```txt
20
```

### Explanation

Microtask runs later after value update.

</details>

---

## 21. Closure with var in Nested Loop

```js
for (var i = 0; i < 2; i++) {
  for (var j = 0; j < 2; j++) {
    setTimeout(() => {
      console.log(i, j);
    });
  }
}
```

<details>
<summary>👉 Show Output</summary>

```txt
2 2
2 2
2 2
2 2
```

### Explanation

Both `i` and `j` are shared.

</details>

---

## 22. Closure with let in Nested Loop

```js
for (let i = 0; i < 2; i++) {
  for (let j = 0; j < 2; j++) {
    setTimeout(() => {
      console.log(i, j);
    });
  }
}
```

<details>
<summary>👉 Show Output</summary>

```txt
0 0
0 1
1 0
1 1
```

### Explanation

Separate bindings created.

</details>

---

## 23. Closure and Shadowing

```js
let x = 1;

function outer() {
  let x = 2;

  return function() {
    console.log(x);
  };
}

const fn = outer();

fn();
```

<details>
<summary>👉 Show Output</summary>

```txt
2
```

### Explanation

Nearest scope variable is used.

</details>

---

## 24. Closure with Returned Object

```js
function test() {
  let count = 0;

  return {
    increment() {
      count++;
    },

    print() {
      console.log(count);
    }
  };
}

const obj = test();

obj.increment();
obj.increment();

obj.print();
```

<details>
<summary>👉 Show Output</summary>

```txt
2
```

### Explanation

Object methods share closure state.

</details>

---

## 25. Closure with Function Reassignment

```js
let fn;

function outer() {
  let x = 100;

  fn = function() {
    console.log(x);
  };
}

outer();

fn();
```

<details>
<summary>👉 Show Output</summary>

```txt
100
```

### Explanation

Function keeps closure after reassignment.

</details>

---

## 26. Closure with Delayed Mutation

```js
function outer() {
  let x = 1;

  setTimeout(() => {
    x = 5;
  }, 50);

  return function() {
    console.log(x);
  };
}

const fn = outer();

setTimeout(() => {
  fn();
}, 100);
```

<details>
<summary>👉 Show Output</summary>

```txt
5
```

### Explanation

Closure sees updated variable value.

</details>

---

## 27. Closure with Array Methods

```js
function test() {
  let count = 0;

  return [1, 2, 3].map(() => {
    count++;
    return count;
  });
}

console.log(test());
```

<details>
<summary>👉 Show Output</summary>

```txt
[1, 2, 3]
```

### Explanation

Callback closes over `count`.

</details>

---

## 28. Closure with Private Counter

```js
function counter() {
  let count = 0;

  return function() {
    return ++count;
  };
}

const c1 = counter();

console.log(c1());
console.log(c1());
```

<details>
<summary>👉 Show Output</summary>

```txt
1
2
```

### Explanation

Private variable persists.

</details>

---

## 29. Closure and Shared State

```js
function test() {
  let x = 0;

  return {
    a() {
      x++;
      console.log(x);
    },

    b() {
      x += 2;
      console.log(x);
    }
  };
}

const obj = test();

obj.a();
obj.b();
obj.a();
```

<details>
<summary>👉 Show Output</summary>

```txt
1
3
4
```

### Explanation

Methods share same closure variable.

</details>

---

## 30. Deep Nested Closure

```js
function a(x) {
  return function(y) {
    return function(z) {
      return function(w) {
        console.log(x + y + z + w);
      };
    };
  };
}

a(1)(2)(3)(4);
```

<details>
<summary>👉 Show Output</summary>

```txt
10
```

### Explanation

Each nested function remembers outer variables.

</details>

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>