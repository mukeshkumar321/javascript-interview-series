# 🧠 JavaScript `this` & Binding — Tricky Output Questions

> These questions are designed to test deep understanding of:
>
> - `this`
> - binding rules
> - arrow functions
> - strict mode
> - call/apply/bind
> - constructor behavior
> - callbacks
> - class methods
> - lexical scope vs dynamic binding
>
> Try predicting the output before opening the answer.

---

# 📑 Table of Contents

1. [Global Context](#1-global-context)
2. [Regular Functions](#2-regular-functions)
3. [Object Methods](#3-object-methods)
4. [Arrow Functions](#4-arrow-functions)
5. [Nested Functions](#5-nested-functions)
6. [call/apply/bind](#6-callapplybind)
7. [Constructor Functions](#7-constructor-functions)
8. [Classes](#8-classes)
9. [Event Loop & Callbacks](#9-event-loop--callbacks)
10. [Strict Mode](#10-strict-mode)
11. [Destructuring](#11-destructuring)
12. [Prototype & Inheritance](#12-prototype--inheritance)
13. [Mixed Edge Cases](#13-mixed-edge-cases)

---

# 1. Global Context

---

## Question 1

```js
console.log(this);
```

<details>
<summary>✅ Output</summary>

Browser:

```js
window
```

Node.js:

```js
{}
```

</details>

---

## Question 2

```js
var name = "John";

function show() {
  console.log(this.name);
}

show();
```

<details>
<summary>✅ Output</summary>

Browser (non-strict):

```js
John
```

Because regular function uses default binding.

</details>

---

## Question 3

```js
"use strict";

var name = "John";

function show() {
  console.log(this);
}

show();
```

<details>
<summary>✅ Output</summary>

```js
undefined
```

Strict mode disables default global binding.

</details>

---

# 2. Regular Functions

---

## Question 4

```js
function test() {
  console.log(this);
}

test.call(5);
```

<details>
<summary>✅ Output</summary>

```js
[Number: 5]
```

Primitive gets boxed into object wrapper.

</details>

---

## Question 5

```js
function test() {
  console.log(typeof this);
}

test.call("hello");
```

<details>
<summary>✅ Output</summary>

```js
object
```

String primitive becomes String object.

</details>

---

# 3. Object Methods

---

## Question 6

```js
const user = {
  name: "John",
  greet() {
    console.log(this.name);
  }
};

user.greet();
```

<details>
<summary>✅ Output</summary>

```js
John
```

</details>

---

## Question 7

```js
const user = {
  name: "John",
  greet() {
    console.log(this);
  }
};

const fn = user.greet;

fn();
```

<details>
<summary>✅ Output</summary>

Browser non-strict:

```js
window
```

Strict mode:

```js
undefined
```

Function lost object reference.

</details>

---

## Question 8

```js
const user = {
  name: "John",
  greet: function () {
    return this.name;
  }
};

console.log((user.greet)());
```

<details>
<summary>✅ Output</summary>

```js
John
```

Parentheses do NOT break binding.

</details>

---

## Question 9

```js
const user = {
  name: "John",
  greet() {
    console.log(this.name);
  }
};

(user.greet = user.greet)();
```

<details>
<summary>✅ Output</summary>

```js
undefined
```

Assignment breaks implicit binding.

</details>

---

# 4. Arrow Functions

---

## Question 10

```js
const user = {
  name: "John",
  greet: () => {
    console.log(this.name);
  }
};

user.greet();
```

<details>
<summary>✅ Output</summary>

Browser:

```js
undefined
```

Arrow functions inherit outer `this`.

</details>

---

## Question 11

```js
const obj = {
  name: "John",
  greet() {
    const inner = () => {
      console.log(this.name);
    };

    inner();
  }
};

obj.greet();
```

<details>
<summary>✅ Output</summary>

```js
John
```

Arrow function lexically captures `this`.

</details>

---

## Question 12

```js
const obj = {
  value: 10,
  regular: function () {
    console.log(this.value);
  },
  arrow: () => {
    console.log(this.value);
  }
};

obj.regular();
obj.arrow();
```

<details>
<summary>✅ Output</summary>

```js
10
undefined
```

</details>

---

# 5. Nested Functions

---

## Question 13

```js
const obj = {
  name: "John",
  greet() {
    function inner() {
      console.log(this.name);
    }

    inner();
  }
};

obj.greet();
```

<details>
<summary>✅ Output</summary>

```js
undefined
```

Nested regular function gets default binding.

</details>

---

## Question 14

```js
const obj = {
  name: "John",
  greet() {
    const self = this;

    function inner() {
      console.log(self.name);
    }

    inner();
  }
};

obj.greet();
```

<details>
<summary>✅ Output</summary>

```js
John
```

Classic `self = this` pattern.

</details>

---

# 6. call/apply/bind

---

## Question 15

```js
function greet(age) {
  console.log(this.name, age);
}

const user = { name: "John" };

greet.call(user, 25);
```

<details>
<summary>✅ Output</summary>

```js
John 25
```

</details>

---

## Question 16

```js
function greet(age) {
  console.log(this.name, age);
}

const user = { name: "John" };

greet.apply(user, [30]);
```

<details>
<summary>✅ Output</summary>

```js
John 30
```

</details>

---

## Question 17

```js
function greet() {
  console.log(this.name);
}

const user1 = { name: "John" };
const user2 = { name: "Alice" };

const bound = greet.bind(user1);

bound.call(user2);
```

<details>
<summary>✅ Output</summary>

```js
John
```

`bind()` creates hard binding.

</details>

---

## Question 18

```js
const obj = {
  name: "John"
};

function show() {
  console.log(this.name);
}

const bound = show.bind(obj);

new bound();
```

<details>
<summary>✅ Output</summary>

```js
undefined
```

`new` binding overrides bind binding.

</details>

---

# 7. Constructor Functions

---

## Question 19

```js
function User(name) {
  this.name = name;
}

const u = User("John");

console.log(name);
```

<details>
<summary>✅ Output</summary>

Browser non-strict:

```js
John
```

Without `new`, `this` becomes global object.

</details>

---

## Question 20

```js
"use strict";

function User(name) {
  this.name = name;
}

User("John");
```

<details>
<summary>✅ Output</summary>

```js
TypeError
```

`this` is undefined in strict mode.

</details>

---

## Question 21

```js
function User(name) {
  this.name = name;

  return {
    name: "Override"
  };
}

const u = new User("John");

console.log(u.name);
```

<details>
<summary>✅ Output</summary>

```js
Override
```

Explicit object return overrides constructed object.

</details>

---

# 8. Classes

---

## Question 22

```js
class User {
  constructor(name) {
    this.name = name;
  }

  greet() {
    console.log(this.name);
  }
}

const u = new User("John");

const fn = u.greet;

fn();
```

<details>
<summary>✅ Output</summary>

```js
TypeError
```

Class methods are always strict mode.

</details>

---

## Question 23

```js
class User {
  name = "John";

  greet = () => {
    console.log(this.name);
  };
}

const u = new User();

const fn = u.greet;

fn();
```

<details>
<summary>✅ Output</summary>

```js
John
```

Arrow function captures instance `this`.

</details>

---

# 9. Event Loop & Callbacks

---

## Question 24

```js
const obj = {
  value: 10,
  show() {
    setTimeout(function () {
      console.log(this.value);
    }, 0);
  }
};

obj.show();
```

<details>
<summary>✅ Output</summary>

Browser:

```js
undefined
```

Callback loses object binding.

</details>

---

## Question 25

```js
const obj = {
  value: 10,
  show() {
    setTimeout(() => {
      console.log(this.value);
    }, 0);
  }
};

obj.show();
```

<details>
<summary>✅ Output</summary>

```js
10
```

Arrow function preserves lexical `this`.

</details>

---

# 10. Strict Mode

---

## Question 26

```js
"use strict";

function test() {
  console.log(this);
}

test.call(null);
```

<details>
<summary>✅ Output</summary>

```js
null
```

Strict mode preserves exact value.

</details>

---

## Question 27

```js
function test() {
  console.log(this);
}

test.call(null);
```

<details>
<summary>✅ Output</summary>

Browser non-strict:

```js
window
```

`null` converts to global object.

</details>

---

# 11. Destructuring

---

## Question 28

```js
const user = {
  name: "John",
  greet() {
    console.log(this.name);
  }
};

const { greet } = user;

greet();
```

<details>
<summary>✅ Output</summary>

```js
undefined
```

Destructuring removes object context.

</details>

---

# 12. Prototype & Inheritance

---

## Question 29

```js
function User(name) {
  this.name = name;
}

User.prototype.greet = function () {
  console.log(this.name);
};

const u = new User("John");

u.greet();
```

<details>
<summary>✅ Output</summary>

```js
John
```

</details>

---

## Question 30

```js
function User(name) {
  this.name = name;
}

User.prototype.greet = () => {
  console.log(this.name);
};

const u = new User("John");

u.greet();
```

<details>
<summary>✅ Output</summary>

```js
undefined
```

Arrow function should NOT be used in prototype methods.

</details>

---

# 13. Mixed Edge Cases

---

## Question 31

```js
const obj = {
  name: "John",
  greet() {
    return function () {
      console.log(this.name);
    };
  }
};

obj.greet()();
```

<details>
<summary>✅ Output</summary>

```js
undefined
```

Returned function loses binding.

</details>

---

## Question 32

```js
const obj = {
  name: "John",
  greet() {
    return () => {
      console.log(this.name);
    };
  }
};

obj.greet()();
```

<details>
<summary>✅ Output</summary>

```js
John
```

Arrow function captures outer `this`.

</details>

---

## Question 33

```js
const obj = {
  name: "John",
  greet() {
    console.log(this.name);
  }
};

setTimeout(obj.greet, 0);
```

<details>
<summary>✅ Output</summary>

```js
undefined
```

Passing method reference loses binding.

</details>

---

## Question 34

```js
const obj = {
  name: "John",
  greet() {
    console.log(this.name);
  }
};

setTimeout(obj.greet.bind(obj), 0);
```

<details>
<summary>✅ Output</summary>

```js
John
```

`bind()` permanently fixes `this`.

</details>

---

## Question 35

```js
const obj = {
  name: "John",
  greet: function () {
    console.log(this.name);

    const arrow = () => {
      console.log(this.name);
    };

    arrow();
  }
};

obj.greet();
```

<details>
<summary>✅ Output</summary>

```js
John
John
```

Arrow function inherits method `this`.

</details>

---

# 🎯 Final Interview Rule

To solve ANY `this` question:

---

## Step 1

Check HOW function is called.

---

## Step 2

Apply priority order:

```txt
new binding
↓
explicit binding
↓
implicit binding
↓
default binding
```

---

## Step 3

Check if function is arrow function.

If yes:

```txt
Ignore all binding rules.
Use lexical this.
```

---

# 🚀 Golden Rule

```txt
this depends on CALL-SITE,
not where function is written.
```
