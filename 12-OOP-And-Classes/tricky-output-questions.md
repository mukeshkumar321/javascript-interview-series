# OOP and Classes — Tricky Output Questions

## Table of Contents

1. [Class Declaration Questions](#1-class-declaration-questions)
2. [Inheritance and super Questions](#2-inheritance-and-super-questions)
3. [Static vs Instance Questions](#3-static-vs-instance-questions)
4. [Private Field Questions](#4-private-field-questions)
5. [Method Override Questions](#5-method-override-questions)
6. [class vs Function Constructor Questions](#6-class-vs-function-constructor-questions)
7. [Advanced Class Questions](#7-advanced-class-questions)

---

## 1. Class Declaration Questions

---

### Q1. What will be the output?

```js
try {
  const p = new Person("Alice");
  console.log(p.name);
} catch (e) {
  console.log(e.message);
}

class Person {
  constructor(name) {
    this.name = name;
  }
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Cannot access 'Person' before initialization
```

### Explanation
Unlike function declarations, class declarations are **not hoisted** in a usable way. The class exists in the Temporal Dead Zone (TDZ) from the start of the scope until the declaration is reached. Accessing it before the declaration throws a `ReferenceError`.

</details>

---

### Q2. What will be the output?

```js
class Strict {
  test() {
    return this;
  }
}

const t = new Strict();
const fn = t.test;      // extract method from instance

console.log(fn() === undefined); // ?
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
true
```

### Explanation
The class body always runs in **strict mode**. In strict mode, when a regular function is called without a receiver (not as a method), `this` is `undefined` — not the global object. Extracting `t.test` and calling it as `fn()` loses the receiver, so `this` inside `test()` is `undefined`.

</details>

---

### Q3. What will be the output?

```js
const Foo = class Bar {
  static test() {
    return Bar.name;
  }
};

console.log(Foo.name);
console.log(Foo.test());

try {
  console.log(Bar.name);
} catch (e) {
  console.log("ReferenceError");
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Bar
Bar
ReferenceError
```

### Explanation
In a **named class expression**, the name (`Bar`) is only visible **inside the class body**. Outside the class, only the variable `Foo` exists. `Foo.name` returns `"Bar"` because the `.name` property of a function/class reflects its internal name. Accessing `Bar` in the outer scope throws a `ReferenceError`.

</details>

---

### Q4. What will be the output?

```js
class A {}
class B extends A {}

console.log(typeof A);
console.log(A.prototype.constructor === A);
console.log(Object.getPrototypeOf(B) === A);
console.log(Object.getPrototypeOf(B.prototype) === A.prototype);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
function
true
true
true
```

### Explanation
`class` syntax creates a regular function under the hood, so `typeof A` is `"function"`. The `prototype.constructor` property always points back to the class itself. With `extends`, there are two prototype links: `Object.getPrototypeOf(B) === A` (the class inherits static members) and `Object.getPrototypeOf(B.prototype) === A.prototype` (instances inherit instance methods).

</details>

---

## 2. Inheritance and super Questions

---

### Q1. What will be the output?

```js
class Animal {
  constructor(name) {
    this.name = name;
  }
}

class Dog extends Animal {
  constructor(name) {
    // super(name) is missing!
    this.name = name;
  }
}

try {
  const d = new Dog("Rex");
} catch (e) {
  console.log(e.constructor.name);
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
ReferenceError
```

### Explanation
In a derived class constructor, you **must** call `super()` before accessing `this`. Until `super()` is called, `this` is uninitialized. Attempting to use `this.name = name` before `super()` throws a `ReferenceError: Must call super constructor in derived class before accessing 'this'`.

</details>

---

### Q2. What will be the output?

```js
class A {
  hello() { return "A"; }
}

class B extends A {
  hello() { return super.hello() + "B"; }
}

class C extends B {
  hello() { return super.hello() + "C"; }
}

console.log(new C().hello());
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
ABC
```

### Explanation
`super.hello()` in `C` calls `B.prototype.hello()`, which in turn calls `super.hello()` (i.e., `A.prototype.hello()`). The call chain resolves as: `A.hello()` returns `"A"`, `B.hello()` appends `"B"` to get `"AB"`, and `C.hello()` appends `"C"` to get `"ABC"`.

</details>

---

### Q3. What will be the output?

```js
class Parent {
  static greet() {
    return "Hello from Parent";
  }
}

class Child extends Parent {
  static greet() {
    return super.greet() + " and Child";
  }
}

console.log(Child.greet());
console.log(Parent.greet());
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Hello from Parent and Child
Hello from Parent
```

### Explanation
`super` also works in **static methods** to call the parent class's static method. `Child.greet()` calls `Parent.greet()` via `super.greet()` and appends its own string. The parent's own static method is unaffected.

</details>

---

### Q4. What will be the output?

```js
class Base {
  constructor() {
    this.type = "base";
  }
}

class Derived extends Base {
  constructor() {
    super();
    this.type = "derived";
  }
}

const d = new Derived();
console.log(d.type);
console.log(d instanceof Derived);
console.log(d instanceof Base);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
derived
true
true
```

### Explanation
`super()` runs `Base`'s constructor first, setting `this.type = "base"`. Then Derived's constructor continues and **overwrites** `this.type` to `"derived"`. The final value is `"derived"`. The instance is `instanceof` both `Derived` and `Base` because `extends` sets up the full prototype chain.

</details>

---

## 3. Static vs Instance Questions

---

### Q1. What will be the output?

```js
class Animal {
  static type = "Animal";

  static describe() {
    return `I am a ${this.type}`;
  }
}

class Dog extends Animal {
  static type = "Dog";
}

console.log(Animal.describe());
console.log(Dog.describe());
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
I am a Animal
I am a Dog
```

### Explanation
In a static method, `this` refers to the **class on which the method was called**. `Animal.describe()` has `this === Animal`, so `this.type` is `"Animal"`. `Dog.describe()` is inherited, but `this === Dog` when called on `Dog`, so `this.type` resolves to `Dog`'s own static `type`, which is `"Dog"`.

</details>

---

### Q2. What will be the output?

```js
class Counter {
  static count = 0;

  constructor() {
    Counter.count++;
  }
}

new Counter();
new Counter();
const c = new Counter();

console.log(Counter.count);
console.log(c.count);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
3
undefined
```

### Explanation
`Counter.count` is a **static property** — it lives on the class, not on instances. Each `new Counter()` increments it via `Counter.count++`. After 3 instantiations, `Counter.count` is `3`. Accessing `c.count` on an instance returns `undefined` because static properties are not accessible as instance properties.

</details>

---

### Q3. What will be the output?

```js
class Foo {
  instanceMethod() {
    return "instance";
  }
  static staticMethod() {
    return "static";
  }
}

const f = new Foo();

console.log(f.instanceMethod());
console.log(Foo.staticMethod());
console.log(f.staticMethod);
console.log(Foo.instanceMethod);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
instance
static
undefined
undefined
```

### Explanation
Instance methods are on `Foo.prototype` — accessible via instances but not via the class directly. Static methods are on `Foo` itself — accessible via the class but not via instances. Accessing either in the wrong direction yields `undefined`.

</details>

---

## 4. Private Field Questions

---

### Q1. What will be the output?

```js
class Secret {
  #value = 42;

  getValue() {
    return this.#value;
  }
}

const s = new Secret();

console.log(s.getValue());
console.log(s.value);
console.log("#value" in s);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
42
undefined
false
```

### Explanation
`s.getValue()` works because it accesses `#value` from **inside** the class. `s.value` is `undefined` because there is no public property named `value`. `"#value" in s` is `false` — the `in` operator checks string-keyed properties; private fields use a different internal slot that is not visible to `in` with a string key.

</details>

---

### Q2. What will be the output?

```js
class Base {
  #x = 10;

  getX() {
    return this.#x;
  }
}

class Derived extends Base {
  getPrivate() {
    return this.getX();   // must use inherited public method
  }
}

const d = new Derived();
console.log(d.getPrivate());
console.log(d.getX());
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
10
10
```

### Explanation
Private fields are **not inherited**. `Derived` cannot access `#x` directly — doing so would be a `SyntaxError`. However, `Derived` inherits the **public method** `getX()` which internally reads `#x`. Both `d.getPrivate()` and `d.getX()` ultimately call `Base.prototype.getX()`, which has access to `#x`, returning `10`.

</details>

---

### Q3. What will be the output?

```js
class HasPrivate {
  #secret = true;

  static isInstance(obj) {
    return #secret in obj;  // ergonomic brand check
  }
}

const h = new HasPrivate();
console.log(HasPrivate.isInstance(h));
console.log(HasPrivate.isInstance({}));
console.log(HasPrivate.isInstance(new HasPrivate()));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
true
false
true
```

### Explanation
The `#field in obj` syntax (ergonomic brand check) is the correct way to check whether an object has a given private field. It returns `true` only for instances of `HasPrivate` that have `#secret` initialized. A plain `{}` object has no private fields, so it returns `false`. This pattern is the safest alternative to `instanceof` when dealing with private state.

</details>

---

## 5. Method Override Questions

---

### Q1. What will be the output?

```js
class Animal {
  sound() { return "..."; }
}

class Dog extends Animal {
  sound() { return "Woof"; }
}

class Cat extends Animal {
  sound() { return "Meow"; }
}

const animals = [new Animal(), new Dog(), new Cat()];
animals.forEach(a => console.log(a.sound()));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
...
Woof
Meow
```

### Explanation
This demonstrates **polymorphism**. Even though `animals` is typed as `Animal[]`, each element carries its own concrete class's `sound()` method. JavaScript method lookup starts from the instance's actual prototype chain, so `Dog` and `Cat` instances use their own overridden methods.

</details>

---

### Q2. What will be the output?

```js
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  toString() {
    return `(${this.x}, ${this.y})`;
  }
}

const p = new Point(3, 4);

console.log(String(p));
console.log(`${p}`);
console.log(p + "");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
(3, 4)
(3, 4)
(3, 4)
```

### Explanation
All three produce the same result because all three coerce `p` to a string. `String(p)` explicitly calls `toString()`. Template literals call `toString()` during interpolation. The `+` operator with an empty string triggers type coercion, which also calls `toString()`. Overriding `toString()` in a class customizes all three behaviors.

</details>

---

### Q3. What will be the output?

```js
class A {
  method() { return "A"; }
}

class B extends A {
  method() { return "B"; }
}

class C extends A {
  // Does NOT override method
}

const b = new B();
const c = new C();

console.log(b.method());
console.log(c.method());
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
B
A
```

### Explanation
`B` overrides `method()`, so `b.method()` calls `B.prototype.method()` returning `"B"`. `C` does not override `method()`, so JavaScript walks up `C`'s prototype chain: `C.prototype` → `A.prototype`. It finds `method` on `A.prototype`, returning `"A"`. `B`'s override has no effect on `C`'s chain.

</details>

---

## 6. class vs Function Constructor Questions

---

### Q1. What will be the output?

```js
class Foo {}

try {
  Foo();
} catch (e) {
  console.log(e instanceof TypeError);
  console.log(e.message.includes("new"));
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
true
true
```

### Explanation
Classes **must** be invoked with `new`. Calling a class as a regular function throws a `TypeError` with a message containing the word "new" (e.g., `"Class constructor Foo cannot be invoked without 'new'"`). This is a key difference from constructor functions, which can be called without `new` in non-strict mode.

</details>

---

### Q2. What will be the output?

```js
function FnConstructor(val) {
  this.val = val;
  return { extra: true };  // returns a non-primitive object
}

class ClassConstructor {
  constructor(val) {
    this.val = val;
    return { extra: true }; // same behavior with classes
  }
}

const f = new FnConstructor(1);
const c = new ClassConstructor(2);

console.log(f.val);
console.log(f.extra);
console.log(c.val);
console.log(c.extra);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
undefined
true
undefined
true
```

### Explanation
When a constructor explicitly returns a **non-primitive object**, that object replaces the newly created instance entirely. The `this` object (with `val`) is discarded. This behavior is identical for both constructor functions and ES6 classes. If a primitive (like a number) were returned instead, it would be ignored and `this` would be returned.

</details>

---

### Q3. What will be the output?

```js
class Foo {
  constructor() {
    this.x = 10;
    return 42;  // returning a primitive
  }
}

const f = new Foo();
console.log(f.x);
console.log(f instanceof Foo);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
10
true
```

### Explanation
When a constructor returns a **primitive value**, the return is silently ignored and `this` is returned instead. So `f` is the normally constructed instance with `x = 10`. This contrasts with Q2 above — only returning a non-primitive object overrides `this`.

</details>

---

## 7. Advanced Class Questions

---

### Q1. What will be the output?

```js
class Base {
  constructor() {
    console.log(new.target.name);
  }
}

class Derived extends Base {
  constructor() {
    super();
  }
}

new Base();
new Derived();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Base
Derived
```

### Explanation
`new.target` inside a constructor refers to the **constructor that was directly called with `new`**. When `new Base()` is called, `new.target` is `Base`. When `new Derived()` is called and `super()` runs the `Base` constructor, `new.target` is still `Derived` — the originally invoked constructor. This is the mechanism used to implement abstract-like classes.

</details>

---

### Q2. What will be the output?

```js
class Even {
  static [Symbol.hasInstance](num) {
    return typeof num === "number" && num % 2 === 0;
  }
}

console.log(2 instanceof Even);
console.log(3 instanceof Even);
console.log(4 instanceof Even);
console.log("4" instanceof Even);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
true
false
true
false
```

### Explanation
`Symbol.hasInstance` is a well-known symbol that customizes the behavior of `instanceof`. When defined as a static method, it receives the left-hand side of `instanceof` as its argument. Here it returns `true` only for numbers that are even. `"4"` is a string, not a number, so it returns `false`.

</details>

---

### Q3. What will be the output?

```js
class Builder {
  constructor() {
    this.result = [];
  }

  add(n) {
    this.result.push(n);
    return this;
  }

  double() {
    this.result = this.result.map(x => x * 2);
    return this;
  }

  get() {
    return this.result;
  }
}

console.log(
  new Builder().add(1).add(2).double().add(3).get()
);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
[ 2, 4, 3 ]
```

### Explanation
Each method returns `this`, enabling method chaining. The sequence: `add(1)` → `[1]`, `add(2)` → `[1, 2]`, `double()` → `[2, 4]`, `add(3)` → `[2, 4, 3]`. `get()` returns the array without returning `this`, ending the chain.

</details>

---

### Q4. What will be the output?

```js
class Circle {
  #radius = 0;

  get radius() { return this.#radius; }

  set radius(r) {
    if (r < 0) throw new RangeError("Radius must be non-negative");
    this.#radius = r;
  }

  get area() {
    return (Math.PI * this.#radius ** 2).toFixed(2);
  }
}

const c = new Circle();
c.radius = 5;
console.log(c.radius);
console.log(c.area);

try {
  c.radius = -1;
} catch (e) {
  console.log(e.message);
}

console.log(c.radius); // radius unchanged after failed set
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
5
78.54
Radius must be non-negative
5
```

### Explanation
The setter validates the input before writing to `#radius`. A valid assignment (`5`) succeeds. `area` is a getter-only computed property — `Math.PI * 25 ≈ 78.54`. When `-1` is passed to the setter, the `RangeError` is thrown **before** `#radius` is modified, so `c.radius` remains `5` after the failed assignment.

</details>

---

## Final Tips

- Classes are **not hoisted** — always declare before use to avoid TDZ errors.
- The class body is always **strict mode** — standalone method calls have `this === undefined`.
- `super()` must come before `this` in any derived class constructor — no exceptions.
- Private fields (`#`) are enforced at the syntax level — not just a naming convention.
- `static` methods inherited by subclasses have `this` pointing to the **subclass**, not the parent.
- `new.target` in a constructor always points to the **outermost** `new`-called constructor.
- `Symbol.hasInstance` lets you fully customize `instanceof` behavior.
- Method chaining (`return this`) is a clean pattern for builder-style APIs.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
