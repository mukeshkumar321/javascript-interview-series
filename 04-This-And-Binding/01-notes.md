# 📘 JavaScript `this` & Binding

> Understanding `this` is one of the most important and confusing parts of JavaScript.  
> This guide covers every important concept, edge case, and interview-focused detail about `this` and binding mechanisms in JavaScript.

---

# 📑 Table of Contents

1. What is `this`?
2. Why `this` Exists
3. How `this` is Determined
4. Global Context
5. Function Context
6. Method Context
7. Object Method vs Regular Function
8. `this` in Strict Mode
9. `this` in Browser vs Node.js
10. `this` inside Arrow Functions
11. Arrow Function vs Regular Function
12. `this` in Event Listeners
13. `this` in Classes
14. `this` in Constructor Functions
15. `new` Keyword and `this`
16. Explicit Binding
17. Implicit Binding
18. Default Binding
19. Hard Binding
20. Lexical Binding
21. `call()`
22. `apply()`
23. `bind()`
24. `call()` vs `apply()` vs `bind()`
25. Losing `this`
26. Nested Functions and `this`
27. `this` in Callbacks
28. `this` in setTimeout
29. `this` in Object Destructuring
30. `this` in DOM
31. `this` with Prototype Methods
32. `this` inside IIFE
33. `this` in ES Modules
34. `this` in CommonJS
35. `super` and `this`
36. `this` in Getter & Setter
37. `this` in Static Methods
38. `this` in Closures
39. `this` in Functional Programming
40. Interview Edge Cases
41. Common Mistakes
42. Best Practices
43. Summary

---

# 1. What is `this`?

`this` is a special keyword in JavaScript that refers to the object executing the current function.

The value of `this` is determined at runtime.

Unlike many languages, JavaScript does NOT determine `this` based on where the function is written.

It depends on **how the function is called**.

---

# 2. Why `this` Exists

`this` allows functions to work with different objects dynamically.

Without `this`, we would need to reference object names directly.

## Example

```js
const user = {
  name: "John",
  greet() {
    console.log(this.name);
  }
};

user.greet();
```

Output:

```js
John
```

Here:

```js
this === user
```

---

# 3. How `this` is Determined

There are 4 major rules:

| Rule | Description |
|---|---|
| Default Binding | Simple function call |
| Implicit Binding | Function called through object |
| Explicit Binding | Using call/apply/bind |
| `new` Binding | Function called using `new` |

Priority order:

```txt
new binding > explicit binding > implicit binding > default binding
```

---

# 4. Global Context

In browser global execution context:

```js
console.log(this);
```

Output:

```js
window
```

In Node.js:

```js
console.log(this);
```

Output:

```js
{}
```

---

# 5. Function Context

## Non-Strict Mode

```js
function test() {
  console.log(this);
}

test();
```

Browser output:

```js
window
```

---

## Strict Mode

```js
"use strict";

function test() {
  console.log(this);
}

test();
```

Output:

```js
undefined
```

---

# 6. Method Context

When function is called through object:

```js
const user = {
  name: "John",
  greet() {
    console.log(this.name);
  }
};

user.greet();
```

Output:

```js
John
```

---

# 7. Object Method vs Regular Function

## Method

```js
const obj = {
  value: 10,
  show() {
    console.log(this.value);
  }
};

obj.show();
```

Output:

```js
10
```

---

## Regular Function

```js
const value = 100;

function show() {
  console.log(this.value);
}

show();
```

Browser output:

```js
100
```

---

# 8. `this` in Strict Mode

Strict mode prevents automatic global binding.

```js
"use strict";

function test() {
  console.log(this);
}

test();
```

Output:

```js
undefined
```

---

# 9. `this` in Browser vs Node.js

| Environment | Global `this` |
|---|---|
| Browser | `window` |
| Node.js | `module.exports` |
| ES Module | `undefined` |

---

# 10. `this` inside Arrow Functions

Arrow functions do NOT have their own `this`.

They inherit `this` lexically from surrounding scope.

```js
const obj = {
  name: "John",
  greet: () => {
    console.log(this.name);
  }
};

obj.greet();
```

Browser output:

```js
undefined
```

Because arrow function inherits global `this`.

---

# 11. Arrow Function vs Regular Function

| Feature | Regular Function | Arrow Function |
|---|---|---|
| Own `this` | Yes | No |
| Constructable | Yes | No |
| Arguments object | Yes | No |
| Suitable for methods | Yes | Usually No |

---

# 12. `this` in Event Listeners

```js
button.addEventListener("click", function () {
  console.log(this);
});
```

Output:

```js
button element
```

---

## Arrow Function

```js
button.addEventListener("click", () => {
  console.log(this);
});
```

Arrow function inherits outer `this`.

---

# 13. `this` in Classes

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
u.greet();
```

Output:

```js
John
```

---

# 14. `this` in Constructor Functions

```js
function User(name) {
  this.name = name;
}

const u = new User("John");

console.log(u.name);
```

Output:

```js
John
```

---

# 15. `new` Keyword and `this`

When using `new`:

1. New object is created
2. `this` points to new object
3. Prototype linked
4. Object returned automatically

---

# 16. Explicit Binding

Using:

- `call()`
- `apply()`
- `bind()`

We can manually set `this`.

---

# 17. Implicit Binding

```js
const obj = {
  name: "John",
  greet() {
    console.log(this.name);
  }
};

obj.greet();
```

`this` refers to object before dot.

---

# 18. Default Binding

```js
function test() {
  console.log(this);
}

test();
```

Non-strict:

```js
window
```

Strict:

```js
undefined
```

---

# 19. Hard Binding

```js
function greet() {
  console.log(this.name);
}

const user = { name: "John" };

const bound = greet.bind(user);

bound();
```

Output:

```js
John
```

---

# 20. Lexical Binding

Arrow functions use lexical `this`.

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

Output:

```js
John
```

---

# 21. `call()`

Calls function immediately.

```js
function greet(age) {
  console.log(this.name, age);
}

const user = { name: "John" };

greet.call(user, 25);
```

Output:

```js
John 25
```

---

# 22. `apply()`

Same as `call()` but arguments passed as array.

```js
greet.apply(user, [25]);
```

---

# 23. `bind()`

Returns new bound function.

```js
const bound = greet.bind(user);

bound(25);
```

---

# 24. `call()` vs `apply()` vs `bind()`

| Method | Executes Immediately | Arguments Format | Returns Function |
|---|---|---|---|
| call | Yes | Comma separated | No |
| apply | Yes | Array | No |
| bind | No | Comma separated | Yes |

---

# 25. Losing `this`

```js
const user = {
  name: "John",
  greet() {
    console.log(this.name);
  }
};

const fn = user.greet;

fn();
```

Output:

```js
undefined
```

Because function lost object reference.

---

# 26. Nested Functions and `this`

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

Output:

```js
undefined
```

---

## Solution Using Arrow Function

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
```

---

# 27. `this` in Callbacks

```js
const obj = {
  value: 10,
  show() {
    setTimeout(function () {
      console.log(this.value);
    }, 1000);
  }
};

obj.show();
```

Output:

```js
undefined
```

---

# 28. `this` in setTimeout

## Solution

```js
const obj = {
  value: 10,
  show() {
    setTimeout(() => {
      console.log(this.value);
    }, 1000);
  }
};

obj.show();
```

Output:

```js
10
```

---

# 29. `this` in Object Destructuring

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

Output:

```js
undefined
```

---

# 30. `this` in DOM

```js
<input onclick="console.log(this)" />
```

`this` refers to clicked DOM element.

---

# 31. `this` with Prototype Methods

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

Output:

```js
John
```

---

# 32. `this` inside IIFE

```js
(function () {
  console.log(this);
})();
```

Non-strict browser:

```js
window
```

Strict mode:

```js
undefined
```

---

# 33. `this` in ES Modules

Inside ES modules:

```js
console.log(this);
```

Output:

```js
undefined
```

---

# 34. `this` in CommonJS

Node.js CommonJS:

```js
console.log(this);
```

Output:

```js
module.exports
```

---

# 35. `super` and `this`

```js
class Animal {
  speak() {
    console.log("Animal");
  }
}

class Dog extends Animal {
  speak() {
    super.speak();
    console.log(this);
  }
}
```

`super` works with current instance `this`.

---

# 36. `this` in Getter & Setter

```js
const user = {
  first: "John",
  last: "Doe",

  get fullName() {
    return `${this.first} ${this.last}`;
  }
};

console.log(user.fullName);
```

Output:

```js
John Doe
```

---

# 37. `this` in Static Methods

```js
class User {
  static show() {
    console.log(this);
  }
}

User.show();
```

Output:

```js
[class User]
```

---

# 38. `this` in Closures

Closures and `this` are separate concepts.

Closures preserve variables.

`this` depends on call-site.

---

# 39. `this` in Functional Programming

Functional programming prefers avoiding mutable context.

Arrow functions commonly used because they avoid dynamic `this`.

---

# 40. Interview Edge Cases

---

## Edge Case 1

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

Output:

```js
undefined
```

---

## Edge Case 2

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

Output:

```js
John
```

---

## Edge Case 3

```js
function test() {
  console.log(this);
}

test.call(null);
```

Non-strict browser:

```js
window
```

Strict mode:

```js
null
```

---

## Edge Case 4

```js
const obj = {
  name: "John",
  arrow: () => {
    console.log(this.name);
  }
};

obj.arrow();
```

Output:

```js
undefined
```

---

## Edge Case 5

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

Output:

```js
John
```

Because arrow function lexically binds instance `this`.

---

# 41. Common Mistakes

| Mistake | Problem |
|---|---|
| Using arrow function as object method | Wrong `this` |
| Forgetting bind in callbacks | Lost `this` |
| Destructuring methods | Loses context |
| Confusing closure with `this` | Different concepts |

---

# 42. Best Practices

✅ Use regular functions for object methods

✅ Use arrow functions for callbacks

✅ Use `bind()` carefully

✅ Avoid relying on global `this`

✅ Prefer strict mode

✅ Understand call-site rules

---

# 43. Summary

| Scenario | `this` Value |
|---|---|
| Global browser | `window` |
| Global Node.js | `module.exports` |
| Regular function | global/undefined |
| Object method | object |
| Arrow function | inherited |
| Constructor function | new object |
| Class method | instance |
| Event listener | DOM element |
| call/apply/bind | explicitly assigned |

---

# 🎯 Final Notes

To master `this`, remember:

```txt
Do NOT ask:
"Where function is written?"

Always ask:
"How function is called?"
```

That single rule solves most `this` confusion in JavaScript.
