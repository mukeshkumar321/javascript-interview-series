# Tricky Output Questions — JavaScript Fundamentals 🚀

# Table of Contents

1. [var vs let Scope](#1-var-vs-let-scope)
2. [Hoisting with var](#2-hoisting-with-var)
3. [Temporal Dead Zone](#3-temporal-dead-zone)
4. [Function Declaration vs Function Expression](#4-function-declaration-vs-function-expression)
5. [Closures Inside Loops](#5-closures-inside-loops)
6. [setTimeout with var](#6-settimeout-with-var)
7. [setTimeout with let](#7-settimeout-with-let)
8. [Arrow Function and this](#8-arrow-function-and-this)
9. [Normal Function and this](#9-normal-function-and-this)
10. [Implicit Type Coercion](#10-implicit-type-coercion)
11. [Equality Operators](#11-equality-operators)
12. [Array + Array](#12-array--array)
13. [Object + Object](#13-object--object)
14. [Truthy and Falsy Values](#14-truthy-and-falsy-values)
15. [NaN Special Behavior](#15-nan-special-behavior)
16. [typeof null](#16-typeof-null)
17. [parseInt Weirdness](#17-parseint-weirdness)
18. [Floating Point Precision](#18-floating-point-precision)
19. [delete Operator](#19-delete-operator)
20. [Object Reference Behavior](#20-object-reference-behavior)
21. [Shallow Copy vs Deep Copy](#21-shallow-copy-vs-deep-copy)
22. [Array Holes](#22-array-holes)
23. [map vs forEach Return](#23-map-vs-foreach-return)
24. [Promise Execution Order](#24-promise-execution-order)
25. [Microtask vs Macrotask](#25-microtask-vs-macrotask)
26. [Async Await Execution](#26-async-await-execution)
27. [Promise.resolve().then() Chain](#27-promiseresolvethen-chain)
28. [Event Loop Deep Understanding](#28-event-loop-deep-understanding)
29. [Call by Value vs Reference](#29-call-by-value-vs-reference)
30. [Destructuring Defaults](#30-destructuring-defaults)

---

# 1. var vs let Scope

## Question

```js
{
  var a = 10;
  let b = 20;
}

console.log(a);
console.log(b);
```

<details>
<summary>📌 Show Output</summary>

```js
10
ReferenceError: b is not defined
```

</details>

<details>
<summary>💡 Show Explanation</summary>

### `var`

- Function scoped
- Ignores block scope

### `let`

- Block scoped
- Exists only inside `{}`

So:

- `a` is accessible outside block
- `b` is not accessible outside block

</details>

---

# 2. Hoisting with var

## Question

```js
console.log(a);

var a = 10;
```

<details>
<summary>📌 Show Output</summary>

```js
undefined
```

</details>

<details>
<summary>💡 Show Explanation</summary>

Internally:

```js
var a;

console.log(a);

a = 10;
```

- Declaration hoisted
- Initialization not hoisted

</details>

---

# 3. Temporal Dead Zone

## Question

```js
console.log(a);

let a = 10;
```

<details>
<summary>📌 Show Output</summary>

```js
ReferenceError
```

</details>

<details>
<summary>💡 Show Explanation</summary>

`let` and `const` are hoisted but remain inside:

# Temporal Dead Zone (TDZ)

until initialization.

</details>

---

# 4. Function Declaration vs Function Expression

## Question

```js
sayHi();

function sayHi() {
  console.log("Hello");
}
```

<details>
<summary>📌 Show Output</summary>

```js
Hello
```

</details>

---

## Question

```js
sayHi();

var sayHi = function () {
  console.log("Hello");
};
```

<details>
<summary>📌 Show Output</summary>

```js
TypeError: sayHi is not a function
```

</details>

<details>
<summary>💡 Show Explanation</summary>

### Function Declaration

Entire function hoisted.

### Function Expression

Only variable declaration hoisted.

</details>

---

# 5. Closures Inside Loops

## Question

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => {
    console.log(i);
  }, 1000);
}
```

<details>
<summary>📌 Show Output</summary>

```js
3
3
3
```

</details>

<details>
<summary>💡 Show Explanation</summary>

All callbacks share SAME `i`.

After loop:

```js
i = 3
```

</details>

---

# 6. setTimeout with var

## Question

```js
for (var i = 0; i < 3; i++) {
  setTimeout(function () {
    console.log(i);
  });
}
```

<details>
<summary>📌 Show Output</summary>

```js
3
3
3
```

</details>

<details>
<summary>💡 Show Explanation</summary>

Callbacks execute after loop completion.

</details>

---

# 7. setTimeout with let

## Question

```js
for (let i = 0; i < 3; i++) {
  setTimeout(() => {
    console.log(i);
  });
}
```

<details>
<summary>📌 Show Output</summary>

```js
0
1
2
```

</details>

<details>
<summary>💡 Show Explanation</summary>

`let` creates new binding per iteration.

</details>

---

# 8. Arrow Function and this

## Question

```js
const obj = {
  name: "JS",
  greet: () => {
    console.log(this.name);
  },
};

obj.greet();
```

<details>
<summary>📌 Show Output</summary>

```js
undefined
```

</details>

<details>
<summary>💡 Show Explanation</summary>

Arrow functions inherit `this` from outer scope.

</details>

---

# 9. Normal Function and this

## Question

```js
const obj = {
  name: "JS",
  greet() {
    console.log(this.name);
  },
};

obj.greet();
```

<details>
<summary>📌 Show Output</summary>

```js
JS
```

</details>

<details>
<summary>💡 Show Explanation</summary>

Normal methods get `this` from caller object.

</details>

---

# 10. Implicit Type Coercion

## Question

```js
console.log("5" + 1);
console.log("5" - 1);
console.log(true + 1);
console.log(false + 1);
```

<details>
<summary>📌 Show Output</summary>

```js
51
4
2
1
```

</details>

<details>
<summary>💡 Show Explanation</summary>

- `+` can concatenate
- `-` converts to numbers

</details>

---

# 11. Equality Operators

## Question

```js
console.log(0 == false);
console.log(0 === false);
console.log("" == false);
console.log(null == undefined);
```

<details>
<summary>📌 Show Output</summary>

```js
true
false
true
true
```

</details>

<details>
<summary>💡 Show Explanation</summary>

- `==` does coercion
- `===` strict comparison

</details>

---

# 12. Array + Array

## Question

```js
console.log([] + []);
console.log([1] + [2]);
```

<details>
<summary>📌 Show Output</summary>

```js

12
```

</details>

<details>
<summary>💡 Show Explanation</summary>

Arrays convert into strings before concatenation.

</details>

---

# 13. Object + Object

## Question

```js
console.log({} + {});
```

<details>
<summary>📌 Show Output</summary>

```js
"[object Object][object Object]"
```

</details>

<details>
<summary>💡 Show Explanation</summary>

Objects convert using `toString()`.

</details>

---

# 14. Truthy and Falsy Values

## Question

```js
if ("0") {
  console.log("Truthy");
}

if ([]) {
  console.log("Truthy");
}
```

<details>
<summary>📌 Show Output</summary>

```js
Truthy
Truthy
```

</details>

<details>
<summary>💡 Show Explanation</summary>

Falsy values:

```js
false
0
""
null
undefined
NaN
```

Everything else is truthy.

</details>

---

# 15. NaN Special Behavior

## Question

```js
console.log(NaN === NaN);
console.log(typeof NaN);
```

<details>
<summary>📌 Show Output</summary>

```js
false
number
```

</details>

<details>
<summary>💡 Show Explanation</summary>

`NaN` is special floating-point value.

</details>

---

# 16. typeof null

## Question

```js
console.log(typeof null);
```

<details>
<summary>📌 Show Output</summary>

```js
object
```

</details>

<details>
<summary>💡 Show Explanation</summary>

Historical JavaScript bug.

</details>

---

# 17. parseInt Weirdness

## Question

```js
console.log(parseInt("10+2"));
console.log(parseInt("a10"));
console.log(parseInt("10a"));
```

<details>
<summary>📌 Show Output</summary>

```js
10
NaN
10
```

</details>

<details>
<summary>💡 Show Explanation</summary>

`parseInt` stops at invalid character.

</details>

---

# 18. Floating Point Precision

## Question

```js
console.log(0.1 + 0.2);
```

<details>
<summary>📌 Show Output</summary>

```js
0.30000000000000004
```

</details>

<details>
<summary>💡 Show Explanation</summary>

IEEE 754 floating-point precision issue.

</details>

---

# 19. delete Operator

## Question

```js
const obj = {
  name: "JS",
};

delete obj.name;

console.log(obj);
```

<details>
<summary>📌 Show Output</summary>

```js
{}
```

</details>

---

## Question

```js
var a = 10;

delete a;

console.log(a);
```

<details>
<summary>📌 Show Output</summary>

```js
10
```

</details>

<details>
<summary>💡 Show Explanation</summary>

`delete` removes object properties, not variables.

</details>

---

# 20. Object Reference Behavior

## Question

```js
const a = { name: "JS" };
const b = a;

b.name = "React";

console.log(a.name);
```

<details>
<summary>📌 Show Output</summary>

```js
React
```

</details>

<details>
<summary>💡 Show Explanation</summary>

Objects are copied by reference.

</details>

---

# 21. Shallow Copy vs Deep Copy

## Question

```js
const a = {
  user: {
    name: "JS",
  },
};

const b = { ...a };

b.user.name = "React";

console.log(a.user.name);
```

<details>
<summary>📌 Show Output</summary>

```js
React
```

</details>

<details>
<summary>💡 Show Explanation</summary>

Spread operator creates shallow copy.

</details>

---

# 22. Array Holes

## Question

```js
const arr = [1, , 3];

console.log(arr.length);
console.log(arr[1]);
```

<details>
<summary>📌 Show Output</summary>

```js
3
undefined
```

</details>

<details>
<summary>💡 Show Explanation</summary>

Sparse arrays contain empty slots.

</details>

---

# 23. map vs forEach Return

## Question

```js
const result = [1, 2, 3].forEach((n) => n * 2);

console.log(result);
```

<details>
<summary>📌 Show Output</summary>

```js
undefined
```

</details>

---

## Question

```js
const result = [1, 2, 3].map((n) => n * 2);

console.log(result);
```

<details>
<summary>📌 Show Output</summary>

```js
[2, 4, 6]
```

</details>

<details>
<summary>💡 Show Explanation</summary>

- `forEach` returns undefined
- `map` returns new array

</details>

---

# 24. Promise Execution Order

## Question

```js
console.log(1);

Promise.resolve().then(() => {
  console.log(2);
});

console.log(3);
```

<details>
<summary>📌 Show Output</summary>

```js
1
3
2
```

</details>

<details>
<summary>💡 Show Explanation</summary>

Promise callbacks go to microtask queue.

</details>

---

# 25. Microtask vs Macrotask

## Question

```js
setTimeout(() => {
  console.log("timeout");
});

Promise.resolve().then(() => {
  console.log("promise");
});

console.log("sync");
```

<details>
<summary>📌 Show Output</summary>

```js
sync
promise
timeout
```

</details>

<details>
<summary>💡 Show Explanation</summary>

Priority:

1. Sync
2. Microtasks
3. Macrotasks

</details>

---

# 26. Async Await Execution

## Question

```js
async function test() {
  console.log(1);

  await Promise.resolve();

  console.log(2);
}

test();

console.log(3);
```

<details>
<summary>📌 Show Output</summary>

```js
1
3
2
```

</details>

<details>
<summary>💡 Show Explanation</summary>

Code after `await` becomes microtask.

</details>

---

# 27. Promise.resolve().then() Chain

## Question

```js
Promise.resolve()
  .then(() => {
    console.log(1);
  })
  .then(() => {
    console.log(2);
  });

console.log(3);
```

<details>
<summary>📌 Show Output</summary>

```js
3
1
2
```

</details>

<details>
<summary>💡 Show Explanation</summary>

`.then()` chain executes asynchronously.

</details>

---

# 28. Event Loop Deep Understanding

## Question

```js
console.log("Start");

setTimeout(() => {
  console.log("Timeout");
}, 0);

Promise.resolve().then(() => {
  console.log("Promise");
});

console.log("End");
```

<details>
<summary>📌 Show Output</summary>

```js
Start
End
Promise
Timeout
```

</details>

<details>
<summary>💡 Show Explanation</summary>

Microtasks execute before macrotasks.

</details>

---

# 29. Call by Value vs Reference

## Question

```js
let a = 10;
let b = a;

b = 20;

console.log(a);
```

<details>
<summary>📌 Show Output</summary>

```js
10
```

</details>

---

## Question

```js
let obj1 = { value: 10 };
let obj2 = obj1;

obj2.value = 20;

console.log(obj1.value);
```

<details>
<summary>📌 Show Output</summary>

```js
20
```

</details>

<details>
<summary>💡 Show Explanation</summary>

Primitives copy by value.

Objects copy by reference.

</details>

---

# 30. Destructuring Defaults

## Question

```js
const { name = "Guest" } = {
  name: undefined,
};

console.log(name);
```

<details>
<summary>📌 Show Output</summary>

```js
Guest
```

</details>

<details>
<summary>💡 Show Explanation</summary>

Default works only for `undefined`.

</details>

---

# End Notes 🚀

These questions are heavily asked in:

- JavaScript Interviews
- React Interviews
- Frontend Interviews
- Full Stack Interviews
- Machine Coding Rounds

Mastering these improves:

- Debugging skills
- JS fundamentals
- Event loop understanding
- Closure concepts
- Async behavior knowledge

---