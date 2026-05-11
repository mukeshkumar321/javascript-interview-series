# Closures in JavaScript

# Table of Contents

1. [What is a Closure?](#1-what-is-a-closure)
2. [Why Closures Exist](#2-why-closures-exist)
3. [Lexical Scope](#3-lexical-scope)
4. [Lexical Environment](#4-lexical-environment)
5. [How Closures Work Internally](#5-how-closures-work-internally)
6. [Basic Closure Example](#6-basic-closure-example)
7. [Closure with Returned Function](#7-closure-with-returned-function)
8. [Closure Scope Chain](#8-closure-scope-chain)
9. [Data Hiding & Encapsulation](#9-data-hiding--encapsulation)
10. [Function Factory](#10-function-factory)
11. [Currying Using Closures](#11-currying-using-closures)
12. [Closures in Loops](#12-closures-in-loops)
13. [Closures with setTimeout](#13-closures-with-settimeout)
14. [Closures in Event Listeners](#14-closures-in-event-listeners)
15. [Module Pattern](#15-module-pattern)
16. [Memoization](#16-memoization)
17. [Closures and Memory Management](#17-closures-and-memory-management)
18. [Garbage Collection and Closures](#18-garbage-collection-and-closures)
19. [Advantages of Closures](#19-advantages-of-closures)
20. [Disadvantages of Closures](#20-disadvantages-of-closures)
21. [Common Closure Mistakes](#21-common-closure-mistakes)
22. [Closures vs Scope](#22-closures-vs-scope)
23. [Closures vs Objects](#23-closures-vs-objects)
24. [Real World Use Cases](#24-real-world-use-cases)
25. [Important Interview Points](#25-important-interview-points)
26. [Summary](#26-summary)

---

# 1. What is a Closure?

A closure is a combination of:

- A function
- Its lexical environment

In simple words:

> A closure allows a function to access variables from its outer scope even after the outer function has finished execution.

---

## Example

```js
function outer() {
  let username = "Dilkhush";

  function inner() {
    console.log(username);
  }

  return inner;
}

const fn = outer();

fn();
```

---

## Output

```txt
Dilkhush
```

---

## Why?

Even though `outer()` has finished execution, the `inner()` function still remembers `username`.

That memory preservation is called a closure.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 2. Why Closures Exist

JavaScript functions are first-class citizens.

Functions can:

- Be stored in variables
- Be passed as arguments
- Be returned from functions

Because functions can survive outside their original scope, JavaScript preserves variables needed by those functions.

Closures make this possible.

---

## Without Closures

Returned functions would lose access to outer variables.

Closures solve this problem.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 3. Lexical Scope

Closures are based on lexical scope.

Lexical scope means:

> Scope is determined by where code is written.

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

---

## Output

```txt
10
```

`inner()` can access `a` because it is lexically inside `outer()`.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 4. Lexical Environment

Every execution context has:

- Local Memory
- Reference to outer environment

Together these form the lexical environment.

---

## Structure

```txt
Lexical Environment
    ↓
Local Variables
    +
Reference to Parent Lexical Environment
```

Closures preserve this environment.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 5. How Closures Work Internally

When a function is returned:

- JavaScript keeps the variables alive
- Variables are not destroyed
- Inner function maintains reference to them

---

## Example

```js
function counter() {
  let count = 0;

  return function () {
    count++;
    console.log(count);
  };
}

const increment = counter();

increment();
increment();
increment();
```

---

## Output

```txt
1
2
3
```

---

## Internal Working

`count` is preserved because returned function still references it.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 6. Basic Closure Example

```js
function greeting(message) {
  return function(name) {
    console.log(`${message} ${name}`);
  };
}

const sayHello = greeting("Hello");

sayHello("Dilkhush");
```

---

## Output

```txt
Hello Dilkhush
```

The inner function remembers `message`.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 7. Closure with Returned Function

```js
function outer() {
  let data = "Secret";

  return function() {
    console.log(data);
  };
}

const fn = outer();

fn();
```

---

## Output

```txt
Secret
```

Returned function forms closure over `data`.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 8. Closure Scope Chain

Closures can access:

1. Own variables
2. Parent variables
3. Global variables

---

## Example

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

---

## Output

```txt
Inner
Outer
Global
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 9. Data Hiding & Encapsulation

Closures can create private variables.

---

## Example

```js
function bankAccount() {
  let balance = 1000;

  return {
    deposit(amount) {
      balance += amount;
      console.log(balance);
    },

    withdraw(amount) {
      balance -= amount;
      console.log(balance);
    }
  };
}

const account = bankAccount();

account.deposit(500);
account.withdraw(200);

console.log(account.balance);
```

---

## Output

```txt
1500
1300
undefined
```

`balance` cannot be directly accessed.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 10. Function Factory

Closures help create reusable customized functions.

---

## Example

```js
function multiply(x) {
  return function(y) {
    return x * y;
  };
}

const double = multiply(2);
const triple = multiply(3);

console.log(double(5));
console.log(triple(5));
```

---

## Output

```txt
10
15
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 11. Currying Using Closures

```js
function add(a) {
  return function(b) {
    return function(c) {
      return a + b + c;
    };
  };
}

console.log(add(1)(2)(3));
```

---

## Output

```txt
6
```

Closures preserve `a` and `b`.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 12. Closures in Loops

This is one of the most important interview topics.

---

# Problem with `var`

```js
for (var i = 1; i <= 3; i++) {
  setTimeout(() => {
    console.log(i);
  }, 1000);
}
```

---

## Output

```txt
4
4
4
```

---

## Why?

- `var` is function scoped
- Same `i` is shared
- Loop finishes first
- `i` becomes `4`

---

# Solution Using `let`

```js
for (let i = 1; i <= 3; i++) {
  setTimeout(() => {
    console.log(i);
  }, 1000);
}
```

---

## Output

```txt
1
2
3
```

`let` creates new binding for every iteration.

---

# Solution Using Closure

```js
for (var i = 1; i <= 3; i++) {
  function close(x) {
    setTimeout(() => {
      console.log(x);
    }, 1000);
  }

  close(i);
}
```

---

## Output

```txt
1
2
3
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 13. Closures with setTimeout

```js
function greet(name) {
  setTimeout(() => {
    console.log(`Hello ${name}`);
  }, 1000);
}

greet("Dilkhush");
```

---

## Output

```txt
Hello Dilkhush
```

The callback remembers `name`.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 14. Closures in Event Listeners

```js
function attachEvent() {
  let count = 0;

  document
    .getElementById("btn")
    .addEventListener("click", function() {
      count++;
      console.log(count);
    });
}
```

Each click remembers previous `count`.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 15. Module Pattern

Before ES6 modules, closures were used for modules.

---

## Example

```js
const counterModule = (function() {
  let count = 0;

  return {
    increment() {
      count++;
    },

    getCount() {
      return count;
    }
  };
})();

counterModule.increment();

console.log(counterModule.getCount());
```

---

## Output

```txt
1
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 16. Memoization

Closures help store cache.

---

## Example

```js
function memoizedAdd() {
  let cache = {};

  return function(num) {
    if (cache[num]) {
      console.log("Cached");
      return cache[num];
    }

    console.log("Calculated");

    cache[num] = num + 10;

    return cache[num];
  };
}

const add = memoizedAdd();

console.log(add(5));
console.log(add(5));
```

---

## Output

```txt
Calculated
15

Cached
15
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 17. Closures and Memory Management

Closures keep variables alive in memory.

---

## Example

```js
function hugeMemory() {
  let largeArray = new Array(1000000).fill("🔥");

  return function() {
    console.log(largeArray[0]);
  };
}

const data = hugeMemory();
```

`largeArray` remains in memory because closure references it.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 18. Garbage Collection and Closures

Unused memory is automatically cleaned.

But variables referenced by closures are not garbage collected.

---

## Example

```js
let fn = hugeMemory();

fn = null;
```

Now memory becomes collectible.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 19. Advantages of Closures

- Data privacy
- Encapsulation
- Function factories
- Currying
- Memoization
- Maintaining state
- Useful in async programming

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 20. Disadvantages of Closures

- Increased memory usage
- Possible memory leaks
- Harder debugging
- Retained unnecessary references

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 21. Common Closure Mistakes

---

## 1. Using `var` in loops

```js
for (var i = 0; i < 5; i++) {
  setTimeout(() => console.log(i));
}
```

---

## 2. Retaining unnecessary memory

```js
function test() {
  let bigData = new Array(1000000);

  return function() {
    console.log(bigData);
  };
}
```

---

## 3. Assuming variables are copied

Closures store references, not copies.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 22. Closures vs Scope

| Scope | Closure |
|---|---|
| Determines accessibility | Remembers variables |
| Created during parsing | Created during function creation |
| Static structure | Runtime behavior |

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 23. Closures vs Objects

| Closures | Objects |
|---|---|
| Data privacy | Public properties |
| Functional approach | OOP approach |
| Memory efficient for small cases | Better for large structured data |

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 24. Real World Use Cases

Closures are heavily used in:

- React Hooks
- Event handlers
- Timers
- Debouncing
- Throttling
- Memoization
- State management
- Currying
- Module patterns
- API wrappers

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 25. Important Interview Points

---

## 1. Do closures copy variables?

No.

Closures store references.

---

## 2. Are closures only created when functions return?

No.

Any inner function accessing outer variables creates closure.

---

## 3. Can closures access updated values?

Yes.

```js
function outer() {
  let count = 0;

  return function() {
    count++;
    console.log(count);
  };
}
```

---

## 4. Do arrow functions create closures?

Yes.

Arrow functions also form closures.

---

## 5. Are closures memory efficient?

Not always.

Improper usage can increase memory usage.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

# 26. Summary

- Closures allow functions to remember outer variables
- Closures depend on lexical scope
- Closures preserve lexical environment
- Used for encapsulation and state management
- Commonly used in async JavaScript
- Closures can cause memory leaks if misused

---

# Final Definition

> A closure is a function bundled together with its lexical environment, allowing it to access outer scope variables even after the outer function has completed execution.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>