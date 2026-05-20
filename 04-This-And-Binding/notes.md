# This and Binding in JavaScript

## Table of Contents

1. [What is `this`?](#1-what-is-this)
2. [`this` in Global Context (browser vs Node.js)](#2-this-in-global-context-browser-vs-nodejs)
3. [`this` in Regular Functions (non-strict mode)](#3-this-in-regular-functions-non-strict-mode)
4. [`this` in Strict Mode](#4-this-in-strict-mode)
5. [`this` in Object Methods](#5-this-in-object-methods)
6. [Method Shorthand and `this`](#6-method-shorthand-and-this)
7. [`this` in Arrow Functions](#7-this-in-arrow-functions)
8. [Arrow Function vs Regular Function](#8-arrow-function-vs-regular-function)
9. [Losing `this`](#9-losing-this)
10. [call() Method](#10-call-method)
11. [apply() Method](#11-apply-method)
12. [bind() Method](#12-bind-method)
13. [call vs apply vs bind](#13-call-vs-apply-vs-bind)
14. [`this` in Constructor Functions](#14-this-in-constructor-functions)
15. [`this` in Classes](#15-this-in-classes)
16. [`this` in Event Listeners](#16-this-in-event-listeners)
17. [`this` in setTimeout (trap)](#17-this-in-settimeout-trap)
18. [`this` in Nested Functions](#18-this-in-nested-functions)
19. [`this` with Chaining](#19-this-with-chaining)
20. [Explicit Binding with Arrow Functions](#20-explicit-binding-with-arrow-functions)
21. [new Keyword and `this`](#21-new-keyword-and-this)
22. [Summary Table](#22-summary-table)

---

## 1. What is `this`?

`this` is a special keyword in JavaScript that refers to the **execution context** — the object that is currently executing the code. Its value is determined at **runtime**, not at the time the function is written, and it depends on **how** the function is called, not where it is defined (with the exception of arrow functions).

```js
function greet() {
  console.log(this);
}

const person = {
  name: "Alice",
  greet: function () {
    console.log(this.name);
  },
};

greet();         // depends on context
person.greet();  // this = person
```

### Output

```js
// greet() in non-strict mode (browser)
Window { ... }

// person.greet()
Alice
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 2. `this` in Global Context (browser vs Node.js)

At the top level (outside any function), `this` refers to the **global object**.

- **Browser:** `this === window`
- **Node.js (module):** `this === module.exports` (an empty object `{}` by default)
- **Node.js (REPL):** `this === global`

```js
// Browser
console.log(this === window); // true
console.log(this);            // Window { ... }

// Node.js (inside a module file)
console.log(this);            // {}
console.log(this === module.exports); // true

// Node.js REPL
console.log(this === global); // true
```

### Output

```js
// Browser
true
Window { ... }

// Node.js module
{}
true

// Node.js REPL
true
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 3. `this` in Regular Functions (non-strict mode)

When a regular (non-arrow) function is called **without** an explicit receiver (i.e., not as a method), `this` defaults to the **global object** (`window` in browsers, `global` in Node.js).

```js
function show() {
  console.log(this);
}

show(); // called without a receiver
```

### Output

```js
// Browser (non-strict mode)
Window { ... }

// Node.js (non-strict mode)
Object [global] { ... }
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 4. `this` in Strict Mode

In **strict mode** (`"use strict"`), `this` inside a regular function that is called without a receiver is `undefined` instead of the global object. This prevents accidental pollution of the global scope.

```js
"use strict";

function show() {
  console.log(this);
}

show();
```

### Output

```js
undefined
```

```js
"use strict";

function Person(name) {
  this.name = name; // TypeError if called without new
}

Person("Alice"); // TypeError: Cannot set properties of undefined
```

### Output

```js
// TypeError: Cannot set properties of undefined (setting 'name')
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 5. `this` in Object Methods

When a function is called **as a method of an object**, `this` refers to the **object that owns the method** (the object to the left of the dot at call time).

```js
const user = {
  name: "Bob",
  greet() {
    console.log(`Hello, I am ${this.name}`);
  },
};

user.greet();
```

### Output

```js
Hello, I am Bob
```

```js
const car = {
  brand: "Toyota",
  getBrand: function () {
    return this.brand;
  },
};

console.log(car.getBrand());
```

### Output

```js
Toyota
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 6. Method Shorthand and `this`

ES6 method shorthand (`methodName() {}`) behaves identically to a regular function in terms of `this` binding — `this` is still the calling object.

```js
const counter = {
  count: 0,
  increment() {        // shorthand method
    this.count++;
    console.log(this.count);
  },
  decrement: function () { // traditional method
    this.count--;
    console.log(this.count);
  },
};

counter.increment();
counter.increment();
counter.decrement();
```

### Output

```js
1
2
1
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 7. `this` in Arrow Functions

Arrow functions **do not have their own `this`**. They capture `this` from the **surrounding lexical (enclosing) scope** at the time they are defined. This makes them ideal for callbacks where you want to preserve the outer `this`.

```js
const obj = {
  name: "Arrow",
  regular: function () {
    console.log("regular:", this.name);
  },
  arrow: () => {
    console.log("arrow:", this.name); // this = outer (global) this
  },
};

obj.regular();
obj.arrow();
```

### Output

```js
// Browser (non-strict)
regular: Arrow
arrow: undefined
```

```js
const timer = {
  seconds: 0,
  start() {
    setInterval(() => {
      this.seconds++;           // arrow captures `this` from start()
      console.log(this.seconds);
    }, 1000);
  },
};

timer.start();
```

### Output

```js
1
2
3
// (increments every second)
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 8. Arrow Function vs Regular Function

Key differences in `this` binding between arrow and regular functions:

| Feature | Regular Function | Arrow Function |
|---|---|---|
| Own `this` | Yes | No (inherited from lexical scope) |
| `this` in method | Calling object | Outer scope |
| `this` with `call/apply/bind` | Can be changed | Cannot be changed |
| `this` in callback | Depends on call site | Inherited from enclosing function |
| Use as constructor | Yes | No (throws TypeError) |

```js
function RegularFn() {
  console.log(this); // bound to caller
}

const ArrowFn = () => {
  console.log(this); // bound to enclosing scope
};

const obj = { name: "test" };

RegularFn.call(obj); // { name: "test" }
ArrowFn.call(obj);   // still the outer this (window/global/undefined)
```

### Output

```js
{ name: 'test' }
Window { ... }  // (or global in Node.js)
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 9. Losing `this`

A common pitfall: when a method is **extracted** from an object and called as a standalone function, the binding to the original object is lost.

```js
const person = {
  name: "Charlie",
  greet() {
    console.log(`Hi, I'm ${this.name}`);
  },
};

person.greet(); // works fine

const fn = person.greet; // extract method
fn();                    // `this` is now global/undefined
```

### Output

```js
Hi, I'm Charlie
// In non-strict mode (browser):
Hi, I'm undefined
// In strict mode:
// TypeError: Cannot read properties of undefined (reading 'name')
```

```js
// Common bug in event handlers or callbacks
const timer = {
  name: "Timer",
  start() {
    setTimeout(this.tick, 1000); // this.tick is passed as a reference, loses context
  },
  tick() {
    console.log(`Tick from ${this.name}`);
  },
};

timer.start();
```

### Output

```js
Tick from undefined  // this.name is undefined (this = global/window)
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 10. call() Method

`Function.prototype.call()` invokes a function immediately with a specified `this` value and arguments passed **individually**.

**Syntax:** `fn.call(thisArg, arg1, arg2, ...)`

```js
function introduce(greeting, punctuation) {
  console.log(`${greeting}, I am ${this.name}${punctuation}`);
}

const user1 = { name: "Alice" };
const user2 = { name: "Bob" };

introduce.call(user1, "Hello", "!");
introduce.call(user2, "Hi", ".");
```

### Output

```js
Hello, I am Alice!
Hi, I am Bob.
```

```js
// Borrowing methods
const arrayLike = { 0: "a", 1: "b", 2: "c", length: 3 };
const result = Array.prototype.slice.call(arrayLike);
console.log(result);
```

### Output

```js
[ 'a', 'b', 'c' ]
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 11. apply() Method

`Function.prototype.apply()` works like `call()` but takes arguments as an **array** (or array-like object).

**Syntax:** `fn.apply(thisArg, [arg1, arg2, ...])`

```js
function introduce(greeting, punctuation) {
  console.log(`${greeting}, I am ${this.name}${punctuation}`);
}

const user = { name: "Diana" };

introduce.apply(user, ["Hey", "!"]);
```

### Output

```js
Hey, I am Diana!
```

```js
// Classic use case: spread an array into Math.max
const numbers = [3, 7, 1, 9, 4];
console.log(Math.max.apply(null, numbers));
// Modern equivalent: Math.max(...numbers)
```

### Output

```js
9
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 12. bind() Method

`Function.prototype.bind()` returns a **new function** with a permanently bound `this` value (and optionally pre-filled arguments). The new function can be called later.

**Syntax:** `const boundFn = fn.bind(thisArg, arg1, arg2, ...)`

```js
function greet(greeting) {
  console.log(`${greeting}, ${this.name}`);
}

const user = { name: "Eve" };
const greetEve = greet.bind(user, "Hello");

greetEve();       // called later
greetEve();       // can be called multiple times
```

### Output

```js
Hello, Eve
Hello, Eve
```

```js
// Fixing lost context
const counter = {
  count: 0,
  increment() {
    this.count++;
    console.log(this.count);
  },
};

const inc = counter.increment.bind(counter);
setTimeout(inc, 100);
```

### Output

```js
1
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 13. call vs apply vs bind

| Feature | call() | apply() | bind() |
|---|---|---|---|
| Invokes immediately | Yes | Yes | No (returns a new function) |
| Arguments format | Individual: `fn.call(ctx, a, b)` | Array: `fn.apply(ctx, [a, b])` | Individual (pre-filled): `fn.bind(ctx, a)` |
| Returns | Result of function | Result of function | New bound function |
| Use case | Borrow methods, set `this` inline | Spread array as args | Preserve `this` for later calls |
| Changes `this` | Once (at call time) | Once (at call time) | Permanently (for that bound fn) |

```js
function log(a, b) {
  console.log(this.name, a, b);
}
const ctx = { name: "Test" };

log.call(ctx, 1, 2);        // immediate, individual args
log.apply(ctx, [1, 2]);     // immediate, array args
const bound = log.bind(ctx, 1, 2);
bound();                    // deferred call
```

### Output

```js
Test 1 2
Test 1 2
Test 1 2
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 14. `this` in Constructor Functions

When a function is called with the `new` keyword, `this` refers to the **newly created object** (instance). The constructor implicitly returns `this` unless an explicit object is returned.

```js
function Animal(name, sound) {
  this.name = name;
  this.sound = sound;
  this.speak = function () {
    console.log(`${this.name} says ${this.sound}`);
  };
}

const dog = new Animal("Dog", "Woof");
const cat = new Animal("Cat", "Meow");

dog.speak();
cat.speak();

console.log(dog.name);
console.log(dog instanceof Animal);
```

### Output

```js
Dog says Woof
Cat says Meow
Dog
true
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 15. `this` in Classes

ES6 classes are syntactic sugar over constructor functions. `this` inside a class refers to the **instance** of the class, similar to constructor functions.

```js
class Vehicle {
  constructor(brand, speed) {
    this.brand = brand;
    this.speed = speed;
  }

  describe() {
    console.log(`${this.brand} goes at ${this.speed} km/h`);
  }
}

class Car extends Vehicle {
  constructor(brand, speed, doors) {
    super(brand, speed);     // sets this.brand and this.speed via Vehicle
    this.doors = doors;
  }

  info() {
    console.log(`${this.brand} has ${this.doors} doors`);
  }
}

const myCar = new Car("Toyota", 180, 4);
myCar.describe();
myCar.info();
```

### Output

```js
Toyota goes at 180 km/h
Toyota has 4 doors
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 16. `this` in Event Listeners

In a DOM event listener, `this` refers to the **element** that received the event (the element the listener was attached to) when using a regular function. Arrow functions inherit `this` from the surrounding scope instead.

```js
// Regular function — `this` is the clicked element
document.getElementById("btn").addEventListener("click", function () {
  console.log(this); // <button id="btn">
  this.style.color = "red";
});

// Arrow function — `this` is the outer scope (window in this case)
document.getElementById("btn").addEventListener("click", () => {
  console.log(this); // Window
});
```

### Output

```js
// Regular function click:
<button id="btn">...</button>

// Arrow function click:
Window { ... }
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 17. `this` in setTimeout (trap)

`setTimeout` calls the callback as a plain function (not as a method), so in non-strict mode `this` defaults to the global object, and in strict mode it is `undefined`. This is one of the most common `this` pitfalls.

```js
const obj = {
  name: "Timeout Trap",
  run() {
    console.log("run this:", this.name); // correct

    setTimeout(function () {
      console.log("timeout this:", this.name); // lost
    }, 100);

    setTimeout(() => {
      console.log("arrow timeout this:", this.name); // correct — arrow
    }, 200);
  },
};

obj.run();
```

### Output

```js
run this: Timeout Trap
timeout this: undefined      // (global object, name property doesn't exist)
arrow timeout this: Timeout Trap
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 18. `this` in Nested Functions

When a regular function is nested inside a method, it loses the method's `this` binding. Solutions include using an arrow function, storing `this` in a variable, or using `bind`.

```js
const obj = {
  value: 42,
  outer() {
    console.log("outer this.value:", this.value); // 42

    function inner() {
      console.log("inner this.value:", this.value); // undefined (or error in strict)
    }

    const innerArrow = () => {
      console.log("innerArrow this.value:", this.value); // 42
    };

    inner();
    innerArrow();
  },
};

obj.outer();
```

### Output

```js
outer this.value: 42
inner this.value: undefined
innerArrow this.value: 42
```

```js
// Classic self/that pattern
const obj2 = {
  value: 99,
  outer() {
    const self = this;
    function inner() {
      console.log("inner self.value:", self.value); // 99 — captured via closure
    }
    inner();
  },
};

obj2.outer();
```

### Output

```js
inner self.value: 99
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 19. `this` with Chaining

Method chaining works by returning `this` from each method, allowing multiple methods to be called sequentially on the same object.

```js
class Builder {
  constructor() {
    this.result = [];
  }

  add(item) {
    this.result.push(item);
    return this; // enables chaining
  }

  remove(item) {
    this.result = this.result.filter((i) => i !== item);
    return this;
  }

  build() {
    console.log(this.result.join(", "));
    return this;
  }
}

new Builder()
  .add("HTML")
  .add("CSS")
  .add("JS")
  .remove("CSS")
  .build();
```

### Output

```js
HTML, JS
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 20. Explicit Binding with Arrow Functions

Arrow functions **ignore** any explicit `this` provided via `call()`, `apply()`, or `bind()`. Their `this` is fixed at the time the arrow function is created and cannot be overridden.

```js
const obj = { name: "Original" };
const other = { name: "Override Attempt" };

const arrowFn = () => {
  console.log(this.name); // lexical this — outer scope
};

arrowFn.call(other);       // ignored
arrowFn.apply(other);      // ignored
const bound = arrowFn.bind(other);
bound();                   // ignored
```

### Output

```js
undefined   // (outer `this.name` is undefined in a module/strict context)
undefined
undefined
```

```js
// Compare with regular function
function regularFn() {
  console.log(this.name);
}

regularFn.call(other);  // works
```

### Output

```js
Override Attempt
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 21. new Keyword and `this`

When `new` is used to call a function, JavaScript performs four steps:
1. Creates a new empty object.
2. Sets the prototype of that object to the function's `prototype` property.
3. Executes the constructor function with `this` set to the new object.
4. Returns the new object (unless the constructor explicitly returns a different object).

```js
function Person(name, age) {
  console.log("this before assignment:", this);
  this.name = name;
  this.age = age;
  console.log("this after assignment:", this);
  // implicitly returns `this`
}

const p = new Person("Frank", 30);
console.log(p.name, p.age);
```

### Output

```js
this before assignment: Person {}
this after assignment: Person { name: 'Frank', age: 30 }
Frank 30
```

```js
// If constructor explicitly returns an object, that object is used instead of `this`
function Weird() {
  this.a = 1;
  return { b: 2 }; // overrides the default return of `this`
}

const w = new Weird();
console.log(w); // { b: 2 }, not { a: 1 }
```

### Output

```js
{ b: 2 }
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 22. Summary Table

| Context | `this` Value |
|---|---|
| Global scope (browser) | `window` |
| Global scope (Node.js module) | `{}` (module.exports) |
| Regular function (non-strict) | Global object (`window`/`global`) |
| Regular function (strict mode) | `undefined` |
| Object method | The object to the left of the dot |
| Arrow function | Inherited from enclosing lexical scope |
| `call()` / `apply()` | First argument (`thisArg`) |
| `bind()` | First argument (permanently bound) |
| Constructor (`new Fn()`) | Newly created object (instance) |
| Class constructor | Newly created instance |
| Event listener (regular fn) | The DOM element that received the event |
| Event listener (arrow fn) | Outer scope's `this` |
| `setTimeout` callback (regular fn) | Global object (non-strict) / `undefined` (strict) |
| `setTimeout` callback (arrow fn) | Inherited from enclosing scope |

---

## Final Notes

`this` is one of the most misunderstood concepts in JavaScript because its value is determined dynamically by **how** a function is invoked, not where it is defined. The four binding rules — default, implicit, explicit (`call`/`apply`/`bind`), and `new` — cover almost every scenario you will encounter. Arrow functions are the elegant modern solution to many `this`-loss problems since they capture `this` from their lexical scope at definition time and can never have it changed. Master these rules and you will confidently predict `this` in any situation.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
