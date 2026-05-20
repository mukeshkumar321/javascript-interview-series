# This and Binding — Tricky Output Questions

## Table of Contents
1. [Global `this` Questions](#1-global-this-questions)
2. [Regular Function `this` Questions](#2-regular-function-this-questions)
3. [Arrow Function `this` Questions](#3-arrow-function-this-questions)
4. [Method Context Questions](#4-method-context-questions)
5. [call/apply/bind Questions](#5-callapplybind-questions)
6. [Constructor and Class Questions](#6-constructor-and-class-questions)
7. [Event Handler Questions](#7-event-handler-questions)
8. [Advanced `this` Questions](#8-advanced-this-questions)

---

## 1. Global `this` Questions

---

### Q1. What will be the output?

```js
console.log(typeof this);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
object
```

### Explanation
In a browser at the global scope, `this` refers to the `window` object, which has type `"object"`. In a Node.js module, `this` is `module.exports` (an empty object `{}`), also of type `"object"`.

</details>

---

### Q2. What will be the output?

```js
var x = 10;

function show() {
  console.log(this.x);
}

show();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
10
```

### Explanation
`show()` is called without a receiver in non-strict mode, so `this` defaults to the global object. `var x = 10` creates a property on the global object, so `this.x` is `10`.

</details>

---

### Q3. What will be the output?

```js
"use strict";

var x = 10;

function show() {
  console.log(this);
}

show();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
undefined
```

### Explanation
In strict mode, calling a function without a receiver leaves `this` as `undefined`, not the global object. The `var x = 10` line is irrelevant here.

</details>

---

## 2. Regular Function `this` Questions

---

### Q4. What will be the output?

```js
const obj = {
  val: 5,
  getVal: function () {
    return this.val;
  },
};

const fn = obj.getVal;
console.log(fn());
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
undefined
```

### Explanation
`fn` holds a plain reference to the function. When called as `fn()` without a receiver, `this` is the global object (non-strict) or `undefined` (strict). The global object does not have a `val` property, so `this.val` is `undefined`.

</details>

---

### Q5. What will be the output?

```js
function Person(name) {
  this.name = name;
  this.greet = function () {
    console.log("Hello " + this.name);
  };
}

const p = new Person("Alice");
const greet = p.greet;
greet();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Hello undefined
```

### Explanation
`greet` is extracted from `p` and called as a standalone function. `this` reverts to the global object (non-strict), which has no `name` property, giving `undefined`.

</details>

---

### Q6. What will be the output?

```js
function outer() {
  console.log(this.name);
  function inner() {
    console.log(this.name);
  }
  inner();
}

const obj = { name: "Outer" };
obj.outer = outer;
obj.outer();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Outer
undefined
```

### Explanation
`obj.outer()` — `this` is `obj`, so `this.name` is `"Outer"`. Inside `inner()`, it is called as a plain function (no receiver), so `this` reverts to the global object, which has no `name` property.

</details>

---

### Q7. What will be the output?

```js
var name = "Global";

const obj = {
  name: "Object",
  getName: function () {
    const inner = function () {
      return this.name;
    };
    return inner();
  },
};

console.log(obj.getName());
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Global
```

### Explanation
`inner()` is a regular function called without a receiver. In non-strict mode, `this` is the global object. `var name = "Global"` sets `window.name` (in a browser), so `this.name` returns `"Global"`.

</details>

---

## 3. Arrow Function `this` Questions

---

### Q8. What will be the output?

```js
const obj = {
  name: "Arrow Test",
  arrowFn: () => {
    console.log(this.name);
  },
};

obj.arrowFn();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
undefined
```

### Explanation
Arrow functions do not have their own `this`. The arrow is defined at the object literal level where `this` is the global/module scope (not `obj`). So `this.name` is `undefined` (or `""` for `window.name` in browsers).

</details>

---

### Q9. What will be the output?

```js
const obj = {
  name: "Lexical",
  regular() {
    const arrow = () => console.log(this.name);
    arrow();
  },
};

obj.regular();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Lexical
```

### Explanation
`arrow` is defined inside `regular()`. When `obj.regular()` is called, `this` inside `regular` is `obj`. The arrow function captures that `this`, so `this.name` is `"Lexical"`.

</details>

---

### Q10. What will be the output?

```js
const obj = {
  value: 100,
  getArrow: () => () => console.log(this.value),
};

obj.getArrow()();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
undefined
```

### Explanation
Both arrow functions capture `this` from the lexical context where `getArrow` is defined (the global/module scope). Neither the outer nor inner arrow can pick up `obj` as `this`.

</details>

---

### Q11. What will be the output?

```js
function Timer() {
  this.seconds = 0;
  setInterval(() => {
    this.seconds++;
  }, 1000);
}

const t = new Timer();
setTimeout(() => console.log(t.seconds), 3100);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
3
```

### Explanation
The arrow inside `setInterval` captures `this` from the `Timer` constructor call, which is the new `t` instance. So `this.seconds++` correctly increments `t.seconds` three times in ~3 seconds.

</details>

---

### Q12. What will be the output?

```js
const a = {
  x: 1,
  getX: function () {
    return () => this.x;
  },
};

const b = { x: 2 };
const fn = a.getX();
console.log(fn.call(b));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
1
```

### Explanation
`a.getX()` returns an arrow function. That arrow captures `this` from `getX`'s execution context, which is `a` (called as `a.getX()`). Calling `.call(b)` on an arrow function is ignored — `this` stays `a`, so `this.x` is `1`.

</details>

---

## 4. Method Context Questions

---

### Q13. What will be the output?

```js
const obj = {
  num: 10,
  double() {
    return this.num * 2;
  },
};

console.log(obj.double());

const d = obj.double;
console.log(d());
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
20
NaN
```

### Explanation
`obj.double()` — `this` is `obj`, so `10 * 2 = 20`. `d()` — called without receiver, `this` is global/undefined. `undefined * 2` is `NaN`.

</details>

---

### Q14. What will be the output?

```js
const obj1 = {
  name: "obj1",
  getName() {
    return this.name;
  },
};

const obj2 = {
  name: "obj2",
  getName: obj1.getName,
};

console.log(obj1.getName());
console.log(obj2.getName());
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
obj1
obj2
```

### Explanation
The same function is referenced by both objects. `this` is determined by the calling object at runtime. `obj1.getName()` — `this` is `obj1`. `obj2.getName()` — `this` is `obj2`.

</details>

---

### Q15. What will be the output?

```js
const obj = {
  count: 0,
  inc: function () {
    this.count++;
  },
};

[1, 2, 3].forEach(obj.inc);
console.log(obj.count);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
0
```

### Explanation
`obj.inc` is passed to `forEach` as a plain function reference. Inside `forEach`, it is called without a receiver (in strict mode `this` would be `undefined`; in non-strict mode `this` would be global). Either way, `obj.count` is never incremented.

</details>

---

### Q16. What will be the output?

```js
const obj = {
  count: 0,
  inc: function () {
    this.count++;
  },
};

[1, 2, 3].forEach(obj.inc.bind(obj));
console.log(obj.count);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
3
```

### Explanation
`obj.inc.bind(obj)` creates a new function with `this` permanently bound to `obj`. Each iteration of `forEach` correctly increments `obj.count`, resulting in `3`.

</details>

---

## 5. call/apply/bind Questions

---

### Q17. What will be the output?

```js
function greet(msg) {
  console.log(msg + " " + this.name);
}

const user = { name: "Charlie" };

greet.call(user, "Hello");
greet.apply(user, ["Hi"]);
const boundGreet = greet.bind(user, "Hey");
boundGreet();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Hello Charlie
Hi Charlie
Hey Charlie
```

### Explanation
All three methods set `this` to `user`. `call` and `apply` invoke immediately (individual vs array args). `bind` returns a new function called afterwards.

</details>

---

### Q18. What will be the output?

```js
function fn() {
  console.log(this.x);
}

const obj = { x: 10 };
const bound = fn.bind(obj);
bound.call({ x: 99 });
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
10
```

### Explanation
Once a function is bound with `bind`, the `this` binding is permanent. Calling `bound.call({ x: 99 })` tries to override `this`, but the bound function ignores it and uses `obj` (`x: 10`).

</details>

---

### Q19. What will be the output?

```js
function multiply(a, b) {
  return this.factor * a * b;
}

const obj = { factor: 3 };
const double = multiply.bind(obj, 2);

console.log(double(5));
console.log(double(10));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
30
60
```

### Explanation
`bind(obj, 2)` permanently sets `this = obj` and pre-fills `a = 2`. `double(5)` → `3 * 2 * 5 = 30`. `double(10)` → `3 * 2 * 10 = 60`. This is called **partial application**.

</details>

---

### Q20. What will be the output?

```js
const obj1 = { x: 1 };
const obj2 = { x: 2 };
const obj3 = { x: 3 };

function getX() {
  return this.x;
}

const b1 = getX.bind(obj1);
const b2 = b1.bind(obj2);
const b3 = b2.bind(obj3);

console.log(b1());
console.log(b2());
console.log(b3());
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
1
1
1
```

### Explanation
Calling `bind` on an already-bound function does not override the first binding. `b1` is bound to `obj1`, and subsequent `bind` calls on `b1`/`b2`/`b3` are all ignored. Every call returns `obj1.x = 1`.

</details>

---

### Q21. What will be the output?

```js
const nums = [5, 1, 8, 3];
const max = Math.max.apply(null, nums);
console.log(max);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
8
```

### Explanation
`Math.max` doesn't use `this`, so passing `null` as the context is fine. `apply` spreads the array `[5, 1, 8, 3]` as individual arguments, equivalent to `Math.max(5, 1, 8, 3)`.

</details>

---

## 6. Constructor and Class Questions

---

### Q22. What will be the output?

```js
function Car(brand) {
  this.brand = brand;
}

const c1 = new Car("Toyota");
const c2 = Car("Honda"); // called without new

console.log(c1.brand);
console.log(c2);
console.log(typeof c2);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Toyota
undefined
undefined
```

### Explanation
`new Car("Toyota")` creates an object and returns it. `Car("Honda")` without `new` in non-strict mode runs with `this` as the global object, sets `global.brand = "Honda"`, and returns `undefined` (no explicit return). So `c2` is `undefined`.

</details>

---

### Q23. What will be the output?

```js
function Foo() {
  this.val = 1;
  return { val: 2 }; // explicit object returned
}

const f = new Foo();
console.log(f.val);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
2
```

### Explanation
When a constructor explicitly returns an **object**, that object overrides the default `this` that `new` would return. `f` is `{ val: 2 }`, not the `Foo` instance.

</details>

---

### Q24. What will be the output?

```js
class Animal {
  constructor(name) {
    this.name = name;
  }
  speak() {
    console.log(`${this.name} speaks`);
  }
}

class Dog extends Animal {
  speak() {
    super.speak();
    console.log(`${this.name} barks`);
  }
}

const d = new Dog("Rex");
d.speak();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Rex speaks
Rex barks
```

### Explanation
`d.speak()` calls `Dog`'s `speak`. Inside, `super.speak()` calls `Animal`'s `speak` with `this` still bound to `d`, so `this.name` is `"Rex"` in both cases.

</details>

---

### Q25. What will be the output?

```js
class Counter {
  count = 0;

  increment = () => {
    this.count++;
  };
}

const c = new Counter();
const inc = c.increment;
inc();
inc();
console.log(c.count);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
2
```

### Explanation
Class field arrow functions are initialized in the constructor with `this` bound to the instance. Even when `inc` is extracted and called without a receiver, `this` remains the `Counter` instance `c`, so `c.count` is incremented correctly.

</details>

---

## 7. Event Handler Questions

---

### Q26. What will be the output? (Browser environment)

```js
const btn = document.createElement("button");
btn.id = "myBtn";
btn.textContent = "Click";
document.body.appendChild(btn);

btn.addEventListener("click", function () {
  console.log(this === btn);
  console.log(this.id);
});

btn.click();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
true
myBtn
```

### Explanation
In a regular function event listener, `this` is the element the listener was attached to. `this === btn` is `true`, and `this.id` is `"myBtn"`.

</details>

---

### Q27. What will be the output? (Browser environment)

```js
const btn = document.createElement("button");
btn.id = "myBtn";
document.body.appendChild(btn);

const handler = {
  message: "clicked",
  handle: function () {
    console.log(this.message);
  },
};

btn.addEventListener("click", handler.handle);
btn.click();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
undefined
```

### Explanation
`handler.handle` is passed as a plain function reference. When the event fires, `this` is set to `btn` (the element), not `handler`. `btn.message` is `undefined`.

</details>

---

### Q28. What will be the output? (Browser environment)

```js
const btn = document.createElement("button");
document.body.appendChild(btn);

const handler = {
  message: "clicked",
  handle: function () {
    console.log(this.message);
  },
};

btn.addEventListener("click", handler.handle.bind(handler));
btn.click();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
clicked
```

### Explanation
`handler.handle.bind(handler)` creates a new function with `this` permanently bound to `handler`. Even though the browser sets the event listener's `this` to the button, `bind` overrides that, so `this.message` is `"clicked"`.

</details>

---

## 8. Advanced `this` Questions

---

### Q29. What will be the output?

```js
const obj = {
  name: "Main",
  fn: function () {
    console.log(this.name);
    return {
      name: "Inner",
      fn: function () {
        console.log(this.name);
      },
    };
  },
};

obj.fn().fn();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Main
Inner
```

### Explanation
`obj.fn()` — `this` is `obj`, logs `"Main"`, and returns a new object. `.fn()` is called on that new object, so `this` is `{ name: "Inner", fn: ... }`, logging `"Inner"`.

</details>

---

### Q30. What will be the output?

```js
function foo() {
  console.log(this.bar);
}

var bar = "global-bar";

const obj = {
  bar: "obj-bar",
  foo,
};

const { foo: extractedFoo } = obj;
extractedFoo();
obj.foo();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
global-bar
obj-bar
```

### Explanation
Destructuring `{ foo: extractedFoo } = obj` extracts the function reference, losing the object binding. `extractedFoo()` is called without a receiver, so `this` is global and `this.bar` is `"global-bar"` (set by `var bar`). `obj.foo()` sets `this` to `obj`, so `this.bar` is `"obj-bar"`.

</details>

---

### Q31. What will be the output?

```js
const obj = {
  x: 10,
  getX() {
    return this.x;
  },
};

console.log([1].map(obj.getX, obj)[0]);
console.log([1].map(obj.getX)[0]);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
10
undefined
```

### Explanation
`Array.prototype.map` accepts a second argument as the `this` context for the callback. `map(obj.getX, obj)` passes `obj` as `this`, returning `10`. `map(obj.getX)` has no `this` argument, defaulting to `undefined` (strict) or global (non-strict), giving `undefined`.

</details>

---

### Q32. What will be the output?

```js
"use strict";

function A() {
  this.val = 1;
  this.getVal = function () {
    return this.val;
  };
}

const a = new A();
const fn = a.getVal;

try {
  console.log(fn());
} catch (e) {
  console.log("Error:", e.message);
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Error: Cannot read properties of undefined (reading 'val')
```

### Explanation
In strict mode, a plain function call has `this = undefined`. `fn()` is called without a receiver, so `this` is `undefined`. Accessing `undefined.val` throws a `TypeError`.

</details>

---

### Q33. What will be the output?

```js
const obj = {
  a: 1,
  b: function () {
    return {
      a: 2,
      c: () => this.a,  // arrow captures this from b's call context
    };
  },
};

console.log(obj.b().c());
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
1
```

### Explanation
When `obj.b()` is called, `this` inside `b` is `obj`. The arrow function `c` is defined inside `b`'s execution context and captures that `this` (which is `obj`). So `this.a` inside `c` is `obj.a = 1`, even though the returned object has its own `a: 2`.

</details>

---

### Q34. What will be the output?

```js
class Greeter {
  constructor(name) {
    this.name = name;
    this.greet = this.greet.bind(this);
  }
  greet() {
    console.log(`Hello, ${this.name}`);
  }
}

const g = new Greeter("World");
const { greet } = g;
greet();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Hello, World
```

### Explanation
In the constructor, `this.greet = this.greet.bind(this)` replaces the prototype method with a bound version attached directly to the instance. Destructuring and calling `greet()` without a receiver still uses the bound `this` (the instance), so the output is correct.

</details>

---

### Q35. What will be the output?

```js
const obj = { n: 1 };

function inc() {
  this.n++;
}

inc.call(obj);
inc.call(obj);
inc.apply(obj);

console.log(obj.n);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
4
```

### Explanation
Each of the three calls sets `this` to `obj` and executes `this.n++`. Starting from `n = 1`: after first `call` → `2`, after second `call` → `3`, after `apply` → `4`.

</details>

---

### Q36. What will be the output?

```js
function outer() {
  const arrowInner = () => {
    console.log(this.val);
  };
  arrowInner.call({ val: 999 }); // attempt to override
}

outer.call({ val: 42 });
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
42
```

### Explanation
`outer` is called with `this = { val: 42 }`. `arrowInner` captures that `this` lexically. `arrowInner.call({ val: 999 })` attempts to override `this` but arrow functions ignore explicit binding — `this` remains `{ val: 42 }`, so `this.val` is `42`.

</details>

---

## Final Tips

- `this` is always determined by **how** a function is called, not where it is defined (except arrow functions).
- Arrow functions **inherit** `this` from their enclosing lexical scope and can never have `this` changed by `call`, `apply`, or `bind`.
- Extracting a method from an object loses the `this` binding — always use `bind` or an arrow wrapper when passing methods as callbacks.
- `bind` creates a **new function** with a permanently fixed `this`; calling `bind` on an already-bound function does nothing.
- In strict mode, a plain function call has `this = undefined`, not the global object — this prevents silent bugs.
- `new` always creates a fresh object and sets `this` to it inside the constructor, unless the constructor explicitly returns another object.
- `call` vs `apply`: the only difference is argument format — individual vs array. Both invoke immediately.
- Class field arrow methods (`increment = () => { ... }`) are auto-bound to the instance in the constructor, making them safe to extract.
- The second argument to `forEach`, `map`, etc. sets the `this` context for the callback — a lesser-known but useful feature.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
