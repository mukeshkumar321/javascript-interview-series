# Functions in JavaScript

## Table of Contents

1. [Function Declaration vs Expression vs Arrow Function](#1-function-declaration-vs-expression-vs-arrow-function)
2. [First-Class Functions](#2-first-class-functions)
3. [Higher-Order Functions](#3-higher-order-functions)
4. [IIFE (Immediately Invoked Function Expression)](#4-iife-immediately-invoked-function-expression)
5. [Pure Functions](#5-pure-functions)
6. [Default Parameters](#6-default-parameters)
7. [Rest Parameters](#7-rest-parameters)
8. [Arguments Object](#8-arguments-object)
9. [Named Function Expressions](#9-named-function-expressions)
10. [Function Length Property](#10-function-length-property)
11. [Currying](#11-currying)
12. [Partial Application](#12-partial-application)
13. [Function Composition](#13-function-composition)
14. [Memoization](#14-memoization)
15. [Generator Functions](#15-generator-functions)
16. [Iterator Protocol](#16-iterator-protocol)
17. [Recursive Functions and Stack](#17-recursive-functions-and-stack)
18. [Tail Call Optimization](#18-tail-call-optimization)
19. [call, apply, and bind](#19-call-apply-and-bind)
20. [Functions are Objects](#20-functions-are-objects)
21. [Summary Table](#21-summary-table)

---

## 1. Function Declaration vs Expression vs Arrow Function

JavaScript has three main ways to define a function. They differ in hoisting behavior, `this` binding, and syntax.

```js
// Function Declaration — hoisted completely
function greet(name) {
  return `Hello, ${name}`;
}

// Function Expression — variable is hoisted, but assignment is not
const greetExpr = function(name) {
  return `Hello, ${name}`;
};

// Arrow Function — no own `this`, no `arguments`, cannot be used as constructor
const greetArrow = (name) => `Hello, ${name}`;

console.log(greet("Alice"));
console.log(greetExpr("Bob"));
console.log(greetArrow("Carol"));
```

### Output

```js
Hello, Alice
Hello, Bob
Hello, Carol
```

**Key differences table:**

| Feature | Declaration | Expression | Arrow |
|---|---|---|---|
| Hoisted | Yes (fully) | Partially (var) / No (let/const) | Partially (var) / No (let/const) |
| `this` binding | Dynamic | Dynamic | Lexical (inherited) |
| `arguments` object | Yes | Yes | No |
| `new` constructor | Yes | Yes | No |
| Named in stack trace | Yes | Only if named | Usually anonymous |
| Suitable for methods | Yes | Yes | No (loses `this`) |

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 2. First-Class Functions

In JavaScript, functions are **first-class citizens** — they can be stored in variables, passed as arguments, returned from other functions, and assigned as object properties.

```js
// Stored in a variable
const add = (a, b) => a + b;

// Passed as an argument
function apply(fn, x, y) {
  return fn(x, y);
}
console.log(apply(add, 3, 4)); // 7

// Returned from a function
function makeMultiplier(factor) {
  return function(num) {
    return num * factor;
  };
}
const double = makeMultiplier(2);
console.log(double(5)); // 10

// Stored in an array or object
const ops = { add, double };
console.log(ops.add(2, 3)); // 5
```

### Output

```js
7
10
5
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 3. Higher-Order Functions

A **higher-order function** is a function that either takes one or more functions as arguments, or returns a function as its result.

```js
// Takes a function as argument
function repeat(n, action) {
  for (let i = 0; i < n; i++) {
    action(i);
  }
}
repeat(3, console.log); // 0, 1, 2

// Returns a function
function multiplier(factor) {
  return (number) => number * factor;
}
const triple = multiplier(3);
console.log(triple(7)); // 21

// Built-in HOFs: map, filter, reduce
const numbers = [1, 2, 3, 4, 5];
const evens = numbers.filter(n => n % 2 === 0);
console.log(evens); // [2, 4]
```

### Output

```js
0
1
2
21
[2, 4]
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 4. IIFE (Immediately Invoked Function Expression)

An **IIFE** is a function that is defined and called immediately. It creates its own scope, which was widely used before ES6 modules to avoid polluting the global scope.

```js
// Standard IIFE syntax
(function() {
  const message = "I run immediately";
  console.log(message);
})();

// Arrow function IIFE
(() => {
  console.log("Arrow IIFE");
})();

// IIFE with arguments
(function(x, y) {
  console.log(x + y);
})(10, 20);

// IIFE returning a value
const result = (function() {
  return 42;
})();
console.log(result);
```

### Output

```js
I run immediately
Arrow IIFE
30
42
```

**Why use IIFE?**
- Avoid variable name collisions in global scope.
- Initialize modules or configurations once.
- Create private scope for variables.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 5. Pure Functions

A **pure function** always returns the same output for the same input and produces no side effects (does not modify external state).

```js
// Pure function — same input always gives same output
function add(a, b) {
  return a + b;
}
console.log(add(2, 3)); // 5
console.log(add(2, 3)); // 5 — always the same

// Impure function — modifies external state (side effect)
let counter = 0;
function increment() {
  counter++; // side effect: modifying external variable
  return counter;
}
console.log(increment()); // 1
console.log(increment()); // 2 — different result each call

// Impure function — relies on external state
const TAX = 0.1;
function calculateTotal(price) {
  return price + price * TAX; // depends on external TAX
}
```

### Output

```js
5
5
1
2
```

**Properties of pure functions:**
- No mutation of input arguments.
- No I/O (no console.log, fetch, etc.).
- No access to or modification of global variables.
- Referentially transparent (can replace call with its return value).

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 6. Default Parameters

Default parameters allow you to assign fallback values to function parameters when no argument is passed or `undefined` is passed.

```js
function greet(name = "Guest", greeting = "Hello") {
  return `${greeting}, ${name}!`;
}

console.log(greet());               // Hello, Guest!
console.log(greet("Alice"));        // Hello, Alice!
console.log(greet("Bob", "Hi"));    // Hi, Bob!
console.log(greet(undefined, "Hey")); // Hey, Guest!
console.log(greet(null, "Hey"));    // Hey, null!

// Default parameters can reference earlier parameters
function createBox(width = 10, height = width) {
  return { width, height };
}
console.log(createBox());       // { width: 10, height: 10 }
console.log(createBox(5));      // { width: 5, height: 5 }
console.log(createBox(5, 20));  // { width: 5, height: 20 }
```

### Output

```js
Hello, Guest!
Hello, Alice!
Hi, Bob!
Hey, Guest!
Hey, null!
{ width: 10, height: 10 }
{ width: 5, height: 5 }
{ width: 5, height: 20 }
```

> `undefined` triggers the default; `null` does not.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 7. Rest Parameters

The **rest parameter** (`...args`) collects all remaining arguments into a real array. It must be the last parameter in the function signature.

```js
function sum(...numbers) {
  return numbers.reduce((acc, n) => acc + n, 0);
}
console.log(sum(1, 2, 3));       // 6
console.log(sum(1, 2, 3, 4, 5)); // 15

// Rest with other parameters
function log(level, ...messages) {
  console.log(`[${level}]`, messages.join(", "));
}
log("INFO", "Server started", "Port 3000");
// [INFO] Server started, Port 3000

// Rest parameter is a real Array (has all Array methods)
function first(...args) {
  return args[0];
}
console.log(first(10, 20, 30)); // 10
```

### Output

```js
6
15
[INFO] Server started, Port 3000
10
```

**Rest vs Spread:**
| | Rest | Spread |
|---|---|---|
| Used in | Function parameter list | Function calls, array/object literals |
| Creates | Array from arguments | Expands iterable into elements |

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 8. Arguments Object

The `arguments` object is an **array-like** (not a real array) object available inside regular functions and function expressions. It contains all arguments passed to the function.

```js
function showArgs() {
  console.log(arguments);
  console.log(arguments[0]);
  console.log(arguments.length);
  // arguments is array-like but NOT a real array:
  // console.log(arguments.map(...)) — TypeError
}
showArgs(1, 2, 3);

// Convert to array
function toArray() {
  return Array.from(arguments);
}
console.log(toArray(10, 20, 30)); // [10, 20, 30]

// Arrow functions do NOT have their own arguments object
const arrow = () => {
  // console.log(arguments); // ReferenceError in strict mode
};

function outer() {
  const inner = () => {
    console.log(arguments[0]); // refers to outer's arguments
  };
  inner();
}
outer(99); // 99
```

### Output

```js
[Arguments] { '0': 1, '1': 2, '2': 3 }
1
3
[10, 20, 30]
99
```

> Prefer **rest parameters** in modern code. `arguments` is a legacy feature.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 9. Named Function Expressions

A **named function expression** is a function expression with an explicit name. The name is local to the function body and is useful for recursion and better stack traces.

```js
// Anonymous function expression
const factorial1 = function(n) {
  return n <= 1 ? 1 : n * factorial1(n - 1); // relies on outer variable
};

// Named function expression — name is scoped to the function body
const factorial2 = function fact(n) {
  return n <= 1 ? 1 : n * fact(n - 1); // uses internal name 'fact'
};

console.log(factorial2(5)); // 120

// The internal name is not accessible outside
// console.log(fact); // ReferenceError

// Named NFE shows better name in stack traces
const divide = function safeDivide(a, b) {
  if (b === 0) throw new Error("Division by zero");
  return a / b;
};
console.log(divide.name); // safeDivide
```

### Output

```js
120
safeDivide
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 10. Function Length Property

The `.length` property of a function returns the **number of declared parameters** (before any rest parameter or default parameter).

```js
function noParams() {}
function twoParams(a, b) {}
function withDefault(a, b = 10) {}
function withRest(a, ...rest) {}
function mixed(a, b = 5, ...rest) {}

console.log(noParams.length);   // 0
console.log(twoParams.length);  // 2
console.log(withDefault.length); // 1 — b has a default, not counted
console.log(withRest.length);    // 1 — rest not counted
console.log(mixed.length);       // 1 — only a is counted

// Arrow functions also have .length
const arrow = (x, y) => x + y;
console.log(arrow.length); // 2
```

### Output

```js
0
2
1
1
1
2
```

> `.length` counts parameters up to (but not including) the first default or rest parameter.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 11. Currying

**Currying** transforms a function that takes multiple arguments into a sequence of functions, each accepting one argument. This enables partial application and function reuse.

```js
// Non-curried
function add(a, b) {
  return a + b;
}

// Manually curried
function curriedAdd(a) {
  return function(b) {
    return a + b;
  };
}
const add5 = curriedAdd(5);
console.log(add5(3));  // 8
console.log(add5(10)); // 15

// Generic curry utility
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn(...args);
    }
    return function(...moreArgs) {
      return curried(...args, ...moreArgs);
    };
  };
}

const multiply = (a, b, c) => a * b * c;
const curriedMultiply = curry(multiply);
console.log(curriedMultiply(2)(3)(4)); // 24
console.log(curriedMultiply(2, 3)(4)); // 24
console.log(curriedMultiply(2)(3, 4)); // 24
```

### Output

```js
8
15
24
24
24
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 12. Partial Application

**Partial application** fixes some arguments of a function and returns a new function that accepts the remaining arguments. Unlike currying (one argument at a time), partial application can fix any number of arguments at once.

```js
function partial(fn, ...presetArgs) {
  return function(...laterArgs) {
    return fn(...presetArgs, ...laterArgs);
  };
}

function greet(greeting, name, punctuation) {
  return `${greeting}, ${name}${punctuation}`;
}

const sayHello = partial(greet, "Hello");
console.log(sayHello("Alice", "!")); // Hello, Alice!
console.log(sayHello("Bob", ".")); // Hello, Bob.

const sayHelloToAlice = partial(greet, "Hello", "Alice");
console.log(sayHelloToAlice("?")); // Hello, Alice?

// Using Function.prototype.bind for partial application
function multiply(a, b) {
  return a * b;
}
const double = multiply.bind(null, 2);
console.log(double(5));  // 10
console.log(double(9));  // 18
```

### Output

```js
Hello, Alice!
Hello, Bob.
Hello, Alice?
10
18
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 13. Function Composition

**Function composition** combines two or more functions so the output of one becomes the input of the next. It reads right-to-left in the standard mathematical convention.

```js
// compose: applies right-to-left
const compose = (...fns) => (x) => fns.reduceRight((v, fn) => fn(v), x);

// pipe: applies left-to-right (easier to read for sequential steps)
const pipe = (...fns) => (x) => fns.reduce((v, fn) => fn(v), x);

const double = x => x * 2;
const addOne = x => x + 1;
const square = x => x * x;

const transform = compose(square, addOne, double);
// double(3) = 6 → addOne(6) = 7 → square(7) = 49
console.log(transform(3)); // 49

const pipeline = pipe(double, addOne, square);
// double(3) = 6 → addOne(6) = 7 → square(7) = 49
console.log(pipeline(3)); // 49

// Two-function compose
const composeTwoFns = (f, g) => (x) => f(g(x));
const shoutReversed = composeTwoFns(
  str => str.toUpperCase(),
  str => str.split("").reverse().join("")
);
console.log(shoutReversed("hello")); // OLLEH
```

### Output

```js
49
49
OLLEH
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 14. Memoization

**Memoization** is an optimization technique that caches the results of expensive function calls so that the same computation is not repeated for the same input.

```js
function memoize(fn) {
  const cache = new Map();
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      console.log("Cache hit:", key);
      return cache.get(key);
    }
    const result = fn(...args);
    cache.set(key, result);
    return result;
  };
}

function expensiveAdd(a, b) {
  // Simulate expensive computation
  return a + b;
}

const memoAdd = memoize(expensiveAdd);
console.log(memoAdd(2, 3)); // 5
console.log(memoAdd(2, 3)); // Cache hit: [2,3] → 5
console.log(memoAdd(4, 5)); // 9

// Memoized fibonacci
function fib(n) {
  if (n <= 1) return n;
  return fib(n - 1) + fib(n - 2);
}
const memoFib = memoize(fib);
console.log(memoFib(10)); // 55
```

### Output

```js
5
Cache hit: [2,3]
5
9
55
```

> Memoization trades memory for speed. It works best for pure functions with expensive, repeated computations. See the Closures section for more on how closures enable memoization.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 15. Generator Functions

A **generator function** (`function*`) returns a **Generator** object. Execution pauses at each `yield` expression and resumes when `.next()` is called, making generators useful for lazy evaluation and custom iteration.

```js
function* counter(start = 1) {
  while (true) {
    yield start++;
  }
}

const gen = counter(1);
console.log(gen.next()); // { value: 1, done: false }
console.log(gen.next()); // { value: 2, done: false }
console.log(gen.next()); // { value: 3, done: false }

// Finite generator
function* range(start, end, step = 1) {
  for (let i = start; i <= end; i += step) {
    yield i;
  }
}

for (const num of range(1, 5)) {
  process.stdout.write(num + " ");
}
// 1 2 3 4 5

// yield* delegates to another iterable
function* concat(a, b) {
  yield* a;
  yield* b;
}
console.log([...concat([1, 2], [3, 4])]); // [1, 2, 3, 4]
```

### Output

```js
{ value: 1, done: false }
{ value: 2, done: false }
{ value: 3, done: false }
1 2 3 4 5
[1, 2, 3, 4]
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 16. Iterator Protocol

An object is an **iterator** if it has a `.next()` method that returns `{ value, done }`. An object is **iterable** if it has a `[Symbol.iterator]()` method that returns an iterator. Generators satisfy both protocols.

```js
// Custom iterable object
const range = {
  from: 1,
  to: 3,
  [Symbol.iterator]() {
    let current = this.from;
    const last = this.to;
    return {
      next() {
        if (current <= last) {
          return { value: current++, done: false };
        }
        return { value: undefined, done: true };
      }
    };
  }
};

for (const num of range) {
  console.log(num);
}
// 1, 2, 3

console.log([...range]); // [1, 2, 3]

// Manual iterator usage
const iter = range[Symbol.iterator]();
console.log(iter.next()); // { value: 1, done: false }
console.log(iter.next()); // { value: 2, done: false }
console.log(iter.next()); // { value: 3, done: false }
console.log(iter.next()); // { value: undefined, done: true }
```

### Output

```js
1
2
3
[1, 2, 3]
{ value: 1, done: false }
{ value: 2, done: false }
{ value: 3, done: false }
{ value: undefined, done: true }
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 17. Recursive Functions and Stack

A **recursive function** calls itself until it reaches a **base case**. Each call adds a new frame to the call stack. Too many recursive calls without a base case cause a **stack overflow**.

```js
// Factorial — classic recursion
function factorial(n) {
  if (n <= 1) return 1;      // base case
  return n * factorial(n - 1); // recursive case
}
console.log(factorial(5)); // 120

// Fibonacci (naive — exponential time)
function fib(n) {
  if (n <= 1) return n;
  return fib(n - 1) + fib(n - 2);
}
console.log(fib(7)); // 13

// Flatten nested array recursively
function flatten(arr) {
  return arr.reduce((flat, item) =>
    Array.isArray(item) ? flat.concat(flatten(item)) : flat.concat(item),
  []);
}
console.log(flatten([1, [2, [3, [4]]]])); // [1, 2, 3, 4]

// Mutual recursion
function isEven(n) { return n === 0 ? true : isOdd(n - 1); }
function isOdd(n)  { return n === 0 ? false : isEven(n - 1); }
console.log(isEven(4)); // true
console.log(isOdd(3));  // true
```

### Output

```js
120
13
[1, 2, 3, 4]
true
true
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 18. Tail Call Optimization

A **tail call** is when a function's last action is a call to another function. In **strict mode**, ES6 specifies that engines **may** eliminate the stack frame when a tail call is detected, preventing stack overflow for deeply recursive functions.

```js
"use strict";

// NOT tail-recursive — multiplication happens after the recursive call
function factorialNormal(n) {
  if (n <= 1) return 1;
  return n * factorialNormal(n - 1); // n * ... is pending — not a tail call
}

// Tail-recursive — accumulator pattern
function factorialTCO(n, acc = 1) {
  if (n <= 1) return acc;
  return factorialTCO(n - 1, n * acc); // last action is the recursive call
}

console.log(factorialNormal(5)); // 120
console.log(factorialTCO(5));    // 120

// Tail-recursive sum
function sumTCO(n, acc = 0) {
  if (n === 0) return acc;
  return sumTCO(n - 1, acc + n);
}
console.log(sumTCO(100)); // 5050
```

### Output

```js
120
120
5050
```

> TCO is defined in the ES6 spec but is only reliably implemented in Safari. In V8/Node.js, TCO is not fully supported. The pattern still matters for conceptual clarity and code organization.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 19. call, apply, and bind

`call`, `apply`, and `bind` are methods on `Function.prototype` that control the `this` context when invoking a function. For detailed `this` binding rules, see the **04-This-And-Binding** section.

```js
function introduce(greeting, punctuation) {
  console.log(`${greeting}, I am ${this.name}${punctuation}`);
}

const person = { name: "Alice" };

// call — invokes immediately, arguments passed individually
introduce.call(person, "Hello", "!");
// Hello, I am Alice!

// apply — invokes immediately, arguments passed as array
introduce.apply(person, ["Hi", "."]);
// Hi, I am Alice.

// bind — returns a new function with this bound, does not invoke
const boundIntroduce = introduce.bind(person, "Hey");
boundIntroduce("?");
// Hey, I am Alice?

// Common use case: borrowing array methods for array-like objects
function showArgs() {
  const args = Array.prototype.slice.call(arguments);
  console.log(args);
}
showArgs(1, 2, 3); // [1, 2, 3]
```

### Output

```js
Hello, I am Alice!
Hi, I am Alice.
Hey, I am Alice?
[1, 2, 3]
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 20. Functions are Objects

In JavaScript, functions are objects of type `Function`. They inherit from `Function.prototype` and can have properties and methods attached to them.

```js
function greet(name) {
  return `Hello, ${name}`;
}

// Built-in properties
console.log(typeof greet);        // function
console.log(greet instanceof Function); // true
console.log(greet.name);          // greet
console.log(greet.length);        // 1

// Functions have a prototype property (used in constructor patterns)
console.log(typeof greet.prototype); // object

// You can add custom properties to functions
function counter() {
  counter.count++;
  return counter.count;
}
counter.count = 0;

console.log(counter()); // 1
console.log(counter()); // 2
console.log(counter.count); // 2

// Function.prototype methods available on all functions
console.log(greet.toString().slice(0, 20)); // function greet(name) {
console.log(greet.call(null, "Bob")); // Hello, Bob
```

### Output

```js
function
true
greet
1
object
1
2
2
function greet(name) {
Hello, Bob
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 21. Summary Table

| Concept | Key Point |
|---|---|
| Function Declaration | Fully hoisted; `function foo() {}` |
| Function Expression | Not hoisted (with `let`/`const`); `const foo = function() {}` |
| Arrow Function | Lexical `this`; no `arguments`; cannot be `new`-ed |
| First-Class Functions | Functions can be stored, passed, and returned like any value |
| Higher-Order Functions | Takes or returns functions; e.g., `map`, `filter`, `reduce` |
| IIFE | Runs immediately; creates isolated scope; `(function(){})()` |
| Pure Function | Same input → same output; no side effects |
| Default Parameters | `undefined` triggers default; `null` does not |
| Rest Parameters | `...args` collects remaining args into a real Array |
| `arguments` object | Array-like; only in regular functions; legacy feature |
| Named Function Expression | Name scoped inside body; useful for recursion and stack traces |
| Function `.length` | Count of params before first default/rest |
| Currying | `f(a, b, c)` → `f(a)(b)(c)` |
| Partial Application | Fix some args, return function for the rest |
| Function Composition | `compose(f, g)(x)` = `f(g(x))`; right-to-left |
| Memoization | Cache results for repeated inputs |
| Generator (`function*`) | Pauses at `yield`; returns `{ value, done }` from `.next()` |
| Iterator Protocol | Object with `.next()` returning `{ value, done }` |
| Recursive Functions | Self-calling; needs base case to avoid stack overflow |
| Tail Call Optimization | Last action is recursive call; may avoid new stack frame |
| `call` / `apply` / `bind` | Manually set `this`; `call`/`apply` invoke, `bind` returns new fn |
| Functions are Objects | Have `.name`, `.length`, `.prototype`, can hold properties |

---

## Final Notes

Functions are the backbone of JavaScript. Understanding the differences between declarations, expressions, and arrow functions is fundamental for writing correct code and passing interviews. Beyond syntax, mastering concepts like currying, composition, and generators unlocks a functional programming style that leads to more testable and reusable code. Generators and the iterator protocol form the foundation for `async/await` under the hood. When studying functions, always cross-reference closures (03), `this` binding (04), and execution context (05) — these topics are deeply intertwined.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
