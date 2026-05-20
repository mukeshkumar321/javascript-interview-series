# Prototypes and Inheritance in JavaScript

## Table of Contents

1. [What is a Prototype?](#1-what-is-a-prototype)
2. [[[Prototype]] and __proto__](#2-prototype-and-__proto__)
3. [prototype vs __proto__](#3-prototype-vs-__proto__)
4. [Prototype Chain](#4-prototype-chain)
5. [Object.prototype — Top of the Chain](#5-objectprototype--top-of-the-chain)
6. [Object.create()](#6-objectcreate)
7. [Constructor Functions and prototype](#7-constructor-functions-and-prototype)
8. [The new Keyword — 4 Internal Steps](#8-the-new-keyword--4-internal-steps)
9. [instanceof Operator](#9-instanceof-operator)
10. [Object.getPrototypeOf vs __proto__](#10-objectgetprototypeof-vs-__proto__)
11. [hasOwnProperty](#11-hasownproperty)
12. [Prototype Pollution](#12-prototype-pollution)
13. [Shadowing Prototype Properties](#13-shadowing-prototype-properties)
14. [Object.setPrototypeOf](#14-objectsetprototypeof)
15. [Prototypal vs Classical Inheritance](#15-prototypal-vs-classical-inheritance)
16. [Inheriting from Constructor Functions](#16-inheriting-from-constructor-functions)
17. [Function.prototype.call / apply for Inheritance](#17-functionprototypecall--apply-for-inheritance)
18. [Object.create for Inheritance](#18-objectcreate-for-inheritance)
19. [Prototype and Performance](#19-prototype-and-performance)
20. [Summary](#20-summary)

---

## 1. What is a Prototype?

Every JavaScript object has an internal link to another object called its **prototype**. When you access a property that does not exist on an object, JavaScript automatically looks up the prototype chain until it finds the property or reaches `null`.

This mechanism is called **prototypal delegation** — properties and methods are not copied into each object, they are looked up on shared prototype objects.

```js
const arr = [1, 2, 3];

// Arrays inherit methods from Array.prototype
console.log(typeof arr.map);        // function — defined on Array.prototype
console.log(typeof arr.toString);   // function — defined on Object.prototype

console.log(arr.__proto__ === Array.prototype);  // true
```

### Output

```js
// function
// function
// true
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 2. [[Prototype]] and __proto__

Every object has an internal slot called `[[Prototype]]`. In browsers and Node.js, this is exposed as the `__proto__` accessor property (defined on `Object.prototype`). It is a non-standard but widely supported way to read and write the prototype link directly.

```js
const animal = { legs: 4 };
const dog = { breed: "Labrador" };

// Manually set prototype
dog.__proto__ = animal;

console.log(dog.breed);  // own property
console.log(dog.legs);   // from prototype — looked up via [[Prototype]]
console.log(dog.hasOwnProperty("legs")); // false — it is inherited
```

### Output

```js
// Labrador
// 4
// false
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 3. prototype vs __proto__

These two look similar but are entirely different things.

| | `prototype` | `__proto__` |
|---|---|---|
| **What it is** | A regular property on functions | Internal `[[Prototype]]` link on every object |
| **Who has it** | Functions only | Every object |
| **Purpose** | Used as `[[Prototype]]` for objects created with `new` | The actual prototype link of an object |

```js
function Person(name) {
  this.name = name;
}

const alice = new Person("Alice");

// Person is a function — it has a .prototype property
console.log(typeof Person.prototype);   // object

// alice's [[Prototype]] IS Person.prototype
console.log(alice.__proto__ === Person.prototype); // true

// Person itself is a function-object — its [[Prototype]] is Function.prototype
console.log(Person.__proto__ === Function.prototype); // true
```

### Output

```js
// object
// true
// true
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 4. Prototype Chain

When you access a property on an object, JavaScript searches:
1. The object's own properties
2. The prototype (`__proto__`) of the object
3. The prototype of the prototype
4. … until `null` is reached (end of the chain)

```js
const grandparent = { a: "grandparent" };
const parent = Object.create(grandparent);
parent.b = "parent";
const child = Object.create(parent);
child.c = "child";

// chain: child --> parent --> grandparent --> Object.prototype --> null

console.log(child.c);  // own property
console.log(child.b);  // found on parent
console.log(child.a);  // found on grandparent
console.log(child.toString()); // found on Object.prototype

console.log(child.hasOwnProperty("a")); // false — inherited
console.log(child.hasOwnProperty("c")); // true  — own
```

### Output

```js
// child
// parent
// grandparent
// [object Object]
// false
// true
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 5. Object.prototype — Top of the Chain

All plain objects ultimately inherit from `Object.prototype`. Its prototype is `null` — the absolute end of every chain. This is where built-in methods like `toString`, `hasOwnProperty`, and `valueOf` come from.

```js
// The top
console.log(Object.getPrototypeOf(Object.prototype)); // null

// Object.prototype provides common methods to all objects
console.log(Object.prototype.hasOwnProperty("toString"));     // true
console.log(Object.prototype.hasOwnProperty("hasOwnProperty")); // true

// A plain object inherits these
const obj = {};
console.log(obj.toString());             // [object Object]
console.log(obj.hasOwnProperty("x"));   // false
```

### Output

```js
// null
// true
// true
// [object Object]
// false
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 6. Object.create()

`Object.create(proto)` creates a new object with `proto` as its `[[Prototype]]`. Pass `null` to create an object with no prototype at all (useful for pure hash maps).

```js
const animal = {
  speak() {
    return `${this.name} makes a sound.`;
  },
};

const dog = Object.create(animal);
dog.name = "Rex";

console.log(dog.speak()); // delegates to animal.speak
console.log(Object.getPrototypeOf(dog) === animal); // true
console.log(dog.hasOwnProperty("speak")); // false — speak is on animal

// Object.create(null) — no prototype
const map = Object.create(null);
map.key = "value";
console.log(map.toString); // undefined — no Object.prototype in chain
```

### Output

```js
// Rex makes a sound.
// true
// false
// undefined
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 7. Constructor Functions and prototype

Every function automatically gets a `prototype` property — an object with a `constructor` back-reference. When you create an instance with `new`, that instance's `[[Prototype]]` is set to the constructor's `prototype` object. Methods added to `prototype` are shared by all instances (not copied).

```js
function Car(brand) {
  this.brand = brand; // own property per instance
}

// Add method to the shared prototype
Car.prototype.drive = function () {
  return `${this.brand} is driving.`;
};

const car1 = new Car("Toyota");
const car2 = new Car("Honda");

console.log(car1.drive());    // Toyota is driving.
console.log(car2.drive());    // Honda is driving.

// Both instances share the SAME drive function
console.log(car1.drive === car2.drive); // true

console.log(car1.hasOwnProperty("drive")); // false — on prototype
console.log(car1.hasOwnProperty("brand")); // true  — own property
```

### Output

```js
// Toyota is driving.
// Honda is driving.
// true
// false
// true
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 8. The new Keyword — 4 Internal Steps

When you call `new Constructor(args)`, JavaScript performs these four steps internally:

1. Create a new empty object `{}`
2. Set the new object's `[[Prototype]]` to `Constructor.prototype`
3. Call `Constructor` with `this` bound to the new object
4. Return the new object — **unless** the constructor explicitly returns a different object

```js
function Person(name) {
  this.name = name;
}

// Simulating what `new` does internally:
function myNew(Constructor, ...args) {
  const obj = Object.create(Constructor.prototype); // steps 1 & 2
  const result = Constructor.apply(obj, args);       // step 3
  return result instanceof Object ? result : obj;    // step 4
}

const alice = myNew(Person, "Alice");

console.log(alice.name);              // Alice
console.log(alice instanceof Person); // true
console.log(Object.getPrototypeOf(alice) === Person.prototype); // true
```

### Output

```js
// Alice
// true
// true
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 9. instanceof Operator

`obj instanceof Constructor` returns `true` if `Constructor.prototype` appears anywhere in `obj`'s prototype chain. It checks the **chain**, not just the immediate prototype.

```js
function Animal(name) {
  this.name = name;
}

function Dog(name) {
  Animal.call(this, name);
}
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

const rex = new Dog("Rex");

console.log(rex instanceof Dog);    // true — Dog.prototype in chain
console.log(rex instanceof Animal); // true — Animal.prototype in chain
console.log(rex instanceof Object); // true — Object.prototype in chain

// instanceof uses the current .prototype reference
function Foo() {}
const obj = new Foo();
Foo.prototype = {}; // replace prototype after creation
console.log(obj instanceof Foo); // false — chain no longer matches
```

### Output

```js
// true
// true
// true
// false
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 10. Object.getPrototypeOf vs __proto__

`Object.getPrototypeOf(obj)` is the **standard** ES5 method for reading the `[[Prototype]]`. `__proto__` is a legacy accessor that also works but is considered deprecated in favor of the static method.

```js
function Foo() {}
const obj = new Foo();

// Both return the same result
console.log(Object.getPrototypeOf(obj) === Foo.prototype); // true
console.log(obj.__proto__ === Foo.prototype);               // true

// Prefer Object.getPrototypeOf in production code
const arr = [];
console.log(Object.getPrototypeOf(arr) === Array.prototype); // true

// Full chain
console.log(Object.getPrototypeOf(Object.getPrototypeOf(arr)) === Object.prototype); // true
console.log(Object.getPrototypeOf(Object.prototype)); // null
```

### Output

```js
// true
// true
// true
// true
// null
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 11. hasOwnProperty

`hasOwnProperty(key)` returns `true` only if the property exists **directly** on the object — it does not traverse the prototype chain. Use it to distinguish own properties from inherited ones.

```js
function Vehicle(type) {
  this.type = type;
}
Vehicle.prototype.move = function () { return "moving"; };

const car = new Vehicle("car");

console.log(car.hasOwnProperty("type")); // true  — set by constructor
console.log(car.hasOwnProperty("move")); // false — lives on prototype

// Safe usage when hasOwnProperty might be overridden
const obj = Object.create(null); // no prototype
obj.key = "value";

// obj.hasOwnProperty is undefined — use this pattern instead:
console.log(Object.prototype.hasOwnProperty.call(obj, "key")); // true
```

### Output

```js
// true
// false
// true
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 12. Prototype Pollution

Prototype pollution is a security vulnerability where an attacker injects properties into `Object.prototype`, causing those properties to appear on every plain object in the application.

```js
// Direct prototype pollution
const malicious = {};
malicious.__proto__.isAdmin = true;

const normalUser = {};
const anotherObject = {};

console.log(normalUser.isAdmin);  // true — Object.prototype was polluted!
console.log(anotherObject.isAdmin); // true — affects ALL objects

// Vulnerable merge function — never use in production
function unsafeMerge(target, source) {
  for (const key in source) {
    target[key] = source[key]; // copies __proto__ properties!
  }
}
```

### Output

```js
// true
// true
```

**Prevention strategies:**
- Validate and sanitize external input before merging into objects
- Use `Object.create(null)` for data maps (no prototype to pollute)
- Use `hasOwnProperty` checks in merge functions
- Use `structuredClone` or trusted deep-copy libraries

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 13. Shadowing Prototype Properties

When you assign a property on an instance that already exists on the prototype, you create an **own property** that **shadows** (hides) the prototype one. The prototype property is not modified.

```js
function Animal() {}
Animal.prototype.legs = 4;

const snake = new Animal();
const dog = new Animal();

console.log(snake.legs); // 4 — from prototype
console.log(dog.legs);   // 4 — from prototype

// Add own property to snake — shadows the prototype property
snake.legs = 0;

console.log(snake.legs);          // 0 — own property (shadows prototype)
console.log(dog.legs);            // 4 — prototype unchanged
console.log(Animal.prototype.legs); // 4 — prototype unchanged
console.log(snake.hasOwnProperty("legs")); // true — now an own property
```

### Output

```js
// 4
// 4
// 0
// 4
// 4
// true
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 14. Object.setPrototypeOf

`Object.setPrototypeOf(obj, proto)` changes the `[[Prototype]]` of an existing object. While it works, it is **strongly discouraged** in hot code paths because changing a prototype after object creation causes JavaScript engines to de-optimize the object.

```js
const animal = { breathes: true };
const fish = { swimsInWater: true };

// Set fish's prototype to animal
Object.setPrototypeOf(fish, animal);

console.log(fish.breathes);      // true  — inherited
console.log(fish.swimsInWater);  // true  — own
console.log(Object.getPrototypeOf(fish) === animal); // true

// Verify chain
console.log(fish instanceof Object); // true — Object.prototype still in chain
```

### Output

```js
// true
// true
// true
// true
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 15. Prototypal vs Classical Inheritance

| Aspect | Prototypal (JavaScript) | Classical (Java, C++) |
|---|---|---|
| Mechanism | Objects link to other objects | Classes define blueprints; instances are copies |
| Inheritance | Delegation through prototype chain | Classes extend classes |
| Runtime flexibility | Prototype can be changed at runtime | Class hierarchy is fixed at compile time |
| Copying | Properties are NOT copied — looked up | Properties ARE copied into instances |
| ES6 `class` | Syntactic sugar over prototypes | True class-based system |

JavaScript uses **delegation**: if an object does not have a property, it delegates the lookup to its prototype. The property is never copied; it lives in one place and is shared by all objects that link to it.

```js
// Prototypal: objects directly inherit from objects
const base = { greet() { return "hello"; } };
const instance = Object.create(base);

console.log(instance.greet());          // hello — delegated
console.log(instance.hasOwnProperty("greet")); // false — never copied
```

### Output

```js
// hello
// false
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 16. Inheriting from Constructor Functions

Before ES6 classes, inheritance between constructor functions was set up manually by wiring the prototype chain.

```js
function Animal(name) {
  this.name = name;
}
Animal.prototype.speak = function () {
  return `${this.name} makes a noise.`;
};

function Dog(name) {
  Animal.call(this, name); // 1. inherit own properties from Animal
}

// 2. set up prototype chain: Dog.prototype --> Animal.prototype
Dog.prototype = Object.create(Animal.prototype);

// 3. restore the constructor reference (gets lost in step 2)
Dog.prototype.constructor = Dog;

// 4. add Dog-specific methods
Dog.prototype.bark = function () {
  return `${this.name} barks.`;
};

const d = new Dog("Rex");

console.log(d.speak());           // Rex makes a noise.
console.log(d.bark());            // Rex barks.
console.log(d instanceof Dog);    // true
console.log(d instanceof Animal); // true
console.log(d.constructor === Dog); // true
```

### Output

```js
// Rex makes a noise.
// Rex barks.
// true
// true
// true
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 17. Function.prototype.call / apply for Inheritance

`call` and `apply` are used inside a child constructor to invoke the parent constructor with the child's `this`, effectively "borrowing" the parent's property initialization logic.

```js
function Shape(color) {
  this.color = color;
}
Shape.prototype.describe = function () {
  return `A ${this.color} shape.`;
};

function Circle(color, radius) {
  Shape.call(this, color); // borrow Shape's constructor
  this.radius = radius;
}

// Wire prototype chain
Circle.prototype = Object.create(Shape.prototype);
Circle.prototype.constructor = Circle;

Circle.prototype.area = function () {
  return (Math.PI * this.radius ** 2).toFixed(2);
};

const c = new Circle("red", 5);

console.log(c.color);     // red     — set by Shape.call
console.log(c.radius);    // 5       — set by Circle
console.log(c.describe()); // A red shape.
console.log(c.area());    // 78.54
```

### Output

```js
// red
// 5
// A red shape.
// 78.54
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 18. Object.create for Inheritance

`Object.create` can set up inheritance chains without any constructor functions — a purely object-based approach.

```js
const vehicle = {
  init(type, wheels) {
    this.type = type;
    this.wheels = wheels;
    return this;
  },
  describe() {
    return `I am a ${this.type} with ${this.wheels} wheels.`;
  },
};

const car = Object.create(vehicle).init("car", 4);
const bike = Object.create(vehicle).init("bike", 2);

console.log(car.describe());   // I am a car with 4 wheels.
console.log(bike.describe());  // I am a bike with 2 wheels.

// car and bike share the describe method (not copied)
console.log(car.describe === bike.describe); // true
console.log(Object.getPrototypeOf(car) === vehicle); // true
```

### Output

```js
// I am a car with 4 wheels.
// I am a bike with 2 wheels.
// true
// true
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 19. Prototype and Performance

Long prototype chains can hurt performance. Each property lookup that fails on the current object must walk up the chain, adding overhead for each step.

**Key points:**
- Property lookups on long chains are slower — engines cannot always cache deeply inherited properties as efficiently.
- `Object.setPrototypeOf` on an existing object triggers expensive de-optimization — always set up the prototype chain at creation time.
- Frequent access to a deeply inherited method can be faster if you cache it in a local variable.

```js
// Bad pattern — O(n) chain length for lookup
function createDeepChain(depth) {
  let obj = {};
  for (let i = 0; i < depth; i++) {
    obj = Object.create(obj);
  }
  return obj;
}

const deepObj = createDeepChain(1000);
deepObj.data = "found";

// Accessing 'data' requires walking 1000 levels up the chain
console.log(deepObj.data); // found — but very slow in hot paths

// Cache inherited methods in a local variable for repeated calls
const arr = [1, 2, 3];
const push = Array.prototype.push; // cached
push.call(arr, 4);
console.log(arr); // [1, 2, 3, 4]
```

### Output

```js
// found
// [1, 2, 3, 4]
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 20. Summary

| Concept | Description |
|---|---|
| `prototype` | Property on functions; becomes `[[Prototype]]` of instances created with `new` |
| `__proto__` | Accessor exposing the `[[Prototype]]` link on any object |
| `prototype` vs `__proto__` | `prototype` is on functions; `__proto__` is on instances |
| Prototype chain | Property lookup walks: object → prototype → … → `Object.prototype` → `null` |
| `Object.create(p)` | Creates object with `p` as its `[[Prototype]]` |
| `Object.create(null)` | Creates object with no prototype (no inherited methods) |
| `new F()` | Creates object, sets `[[Prototype]]` to `F.prototype`, calls `F` with `this` |
| `instanceof` | Checks if `Constructor.prototype` is anywhere in the object's chain |
| `Object.getPrototypeOf` | Standard way to read `[[Prototype]]` (prefer over `__proto__`) |
| `Object.setPrototypeOf` | Mutates `[[Prototype]]`; hurts performance — avoid in hot paths |
| `hasOwnProperty` | Returns `true` only for own (not inherited) properties |
| Shadowing | Assigning an own property that hides (but does not modify) a prototype property |
| Prototype pollution | Injecting into `Object.prototype`; affects every plain object in the app |
| Prototypal inheritance | Delegation — objects link to other objects; properties are not copied |
| Chain end | `Object.prototype.__proto__ === null` |

---

## Final Notes

JavaScript's prototype system is one of its most distinctive features. While ES6 `class` syntax provides a cleaner, more familiar surface, the underlying mechanism is still prototype delegation — understanding it helps you debug tricky inheritance issues, avoid prototype pollution vulnerabilities, and write performant code. Always set up prototype chains at construction time, prefer `Object.getPrototypeOf` over `__proto__`, use `hasOwnProperty` when you need to distinguish own from inherited properties, and keep chains short in performance-sensitive code.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
