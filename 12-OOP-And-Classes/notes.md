# OOP and Classes in JavaScript

## Table of Contents

1. [Object-Oriented Programming in JS](#1-object-oriented-programming-in-js)
2. [class Syntax](#2-class-syntax)
3. [constructor Method](#3-constructor-method)
4. [Instance Methods](#4-instance-methods)
5. [Static Methods and Properties](#5-static-methods-and-properties)
6. [extends and Inheritance](#6-extends-and-inheritance)
7. [super Keyword](#7-super-keyword)
8. [Method Overriding](#8-method-overriding)
9. [Private Fields and Methods](#9-private-fields-and-methods)
10. [Getters and Setters in Classes](#10-getters-and-setters-in-classes)
11. [Class Expressions](#11-class-expressions)
12. [Abstract-like Classes](#12-abstract-like-classes)
13. [Mixins Pattern](#13-mixins-pattern)
14. [instanceof with Classes](#14-instanceof-with-classes)
15. [class vs Constructor Functions](#15-class-vs-constructor-functions)
16. [Class Hoisting](#16-class-hoisting)
17. [Public vs Private Class Fields](#17-public-vs-private-class-fields)
18. [Static Private Fields](#18-static-private-fields)
19. [Chaining with Classes](#19-chaining-with-classes)
20. [OOP Principles in JS](#20-oop-principles-in-js)
21. [Summary Table](#21-summary-table)

---

## 1. Object-Oriented Programming in JS

JavaScript uses a **prototype-based** inheritance model, not the classical class-based model found in languages like Java or C++. Every object has an internal link to another object called its **prototype**. When a property is not found on an object, JavaScript walks up the prototype chain until it finds the property or reaches `null`.

> For deep coverage of the prototype chain, see `11-Prototypes-And-Inheritance`.

The `class` syntax introduced in ES6 is **syntactic sugar** over this prototype system — under the hood, JavaScript still uses prototypes.

```js
function Animal(name) {
  this.name = name;
}

Animal.prototype.speak = function () {
  return `${this.name} makes a sound.`;
};

const dog = new Animal("Rex");
console.log(dog.speak());                        // Rex makes a sound.
console.log(dog.__proto__ === Animal.prototype); // true
```

### Output

```js
Rex makes a sound.
true
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 2. class Syntax

`class` is syntactic sugar over constructor functions and prototypes. Internally it still creates a constructor function and assigns methods to its `prototype`. The class body is always in **strict mode**.

```js
class Animal {
  constructor(name) {
    this.name = name;
  }

  speak() {
    return `${this.name} makes a sound.`;
  }
}

const dog = new Animal("Rex");
console.log(typeof Animal);                        // function
console.log(dog.speak());                          // Rex makes a sound.
console.log(dog.__proto__ === Animal.prototype);   // true
```

### Output

```js
function
Rex makes a sound.
true
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 3. constructor Method

The `constructor` is a special method called automatically when a new instance is created with `new`. A class can have **only one** `constructor`. If omitted, a default empty constructor is provided automatically.

```js
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
}

const p = new Person("Alice", 30);
console.log(p.name); // Alice
console.log(p.age);  // 30
```

### Output

```js
Alice
30
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 4. Instance Methods

Instance methods are defined inside the class body (outside the constructor) and are placed on `ClassName.prototype`. Every instance shares the same method reference — methods are **not** copied onto each instance.

```js
class Counter {
  constructor() {
    this.count = 0;
  }

  increment() {
    this.count++;
    return this;
  }

  getValue() {
    return this.count;
  }
}

const c = new Counter();
c.increment();
c.increment();

console.log(c.getValue());                   // 2
console.log(c.hasOwnProperty("count"));      // true  (data lives on instance)
console.log(c.hasOwnProperty("increment"));  // false (method lives on prototype)
```

### Output

```js
2
true
false
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 5. Static Methods and Properties

`static` members belong to the **class itself**, not to any instance. They are called directly on the class and are useful for utility functions or factory methods. Static properties are also supported.

```js
class MathHelper {
  static PI = 3.14159;

  static square(n) {
    return n * n;
  }

  static cube(n) {
    return n * n * n;
  }
}

console.log(MathHelper.PI);         // 3.14159
console.log(MathHelper.square(4));  // 16
console.log(MathHelper.cube(3));    // 27

const m = new MathHelper();
console.log(m.square);              // undefined (static is not on instances)
```

### Output

```js
3.14159
16
27
undefined
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 6. extends and Inheritance

`extends` sets up a prototype chain between a child class and a parent class. The child inherits all instance methods from the parent. The child class **must** call `super()` before accessing `this` inside its constructor.

```js
class Animal {
  constructor(name) {
    this.name = name;
  }

  speak() {
    return `${this.name} makes a sound.`;
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name);        // must call super() before using this
    this.breed = breed;
  }

  bark() {
    return `${this.name} barks!`;
  }
}

const d = new Dog("Rex", "Labrador");
console.log(d.speak());           // Rex makes a sound.
console.log(d.bark());            // Rex barks!
console.log(d instanceof Dog);    // true
console.log(d instanceof Animal); // true
```

### Output

```js
Rex makes a sound.
Rex barks!
true
true
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 7. super Keyword

`super` serves two distinct purposes:

- **In a constructor**: `super(args)` calls the parent class constructor.
- **In a method**: `super.methodName()` calls the parent's version of that method.

```js
class Shape {
  constructor(color) {
    this.color = color;
  }

  describe() {
    return `A ${this.color} shape.`;
  }
}

class Circle extends Shape {
  constructor(color, radius) {
    super(color);          // calls Shape constructor
    this.radius = radius;
  }

  describe() {
    const base = super.describe();  // calls Shape.describe()
    return `${base} It is a circle with radius ${this.radius}.`;
  }
}

const c = new Circle("red", 5);
console.log(c.describe());
// A red shape. It is a circle with radius 5.
```

### Output

```js
A red shape. It is a circle with radius 5.
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 8. Method Overriding

A child class can redefine a method from the parent. The child's version takes precedence when called on a child instance. Use `super.method()` inside the override to also invoke the parent logic.

```js
class Vehicle {
  startEngine() {
    return "Engine started.";
  }
}

class ElectricCar extends Vehicle {
  startEngine() {
    return "Silent start — electric motor on.";
  }
}

class HybridCar extends Vehicle {
  startEngine() {
    const base = super.startEngine();
    return `${base} + Electric boost activated.`;
  }
}

const v = new Vehicle();
const e = new ElectricCar();
const h = new HybridCar();

console.log(v.startEngine()); // Engine started.
console.log(e.startEngine()); // Silent start — electric motor on.
console.log(h.startEngine()); // Engine started. + Electric boost activated.
```

### Output

```js
Engine started.
Silent start — electric motor on.
Engine started. + Electric boost activated.
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 9. Private Fields and Methods

Private fields use the `#` prefix and are **truly private** — enforced by the JavaScript engine. They cannot be accessed outside the class body, not even in subclasses. Private methods follow the same syntax.

```js
class BankAccount {
  #balance = 0;           // private field

  constructor(initialBalance) {
    this.#balance = initialBalance;
  }

  #validate(amount) {     // private method
    return amount > 0 && amount <= this.#balance;
  }

  withdraw(amount) {
    if (this.#validate(amount)) {
      this.#balance -= amount;
      return `Withdrew ${amount}. Balance: ${this.#balance}`;
    }
    return "Invalid withdrawal.";
  }

  get balance() {
    return this.#balance;
  }
}

const acc = new BankAccount(100);
console.log(acc.withdraw(40)); // Withdrew 40. Balance: 60
console.log(acc.balance);      // 60
// console.log(acc.#balance);  // SyntaxError: Private field '#balance' must be declared in an enclosing class
```

### Output

```js
Withdrew 40. Balance: 60
60
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 10. Getters and Setters in Classes

`get` and `set` define computed properties. They look like regular property access to the caller but execute logic internally. Combine them with private fields for controlled access.

```js
class Temperature {
  #celsius;

  constructor(celsius) {
    this.#celsius = celsius;
  }

  get fahrenheit() {
    return this.#celsius * 1.8 + 32;
  }

  set fahrenheit(f) {
    this.#celsius = (f - 32) / 1.8;
  }

  get celsius() {
    return this.#celsius;
  }
}

const temp = new Temperature(100);
console.log(temp.fahrenheit); // 212

temp.fahrenheit = 32;
console.log(temp.celsius);    // 0
```

### Output

```js
212
0
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 11. Class Expressions

Classes can be defined as expressions — either anonymous or named. Named class expressions have the class name visible **only inside the class body**, not in the outer scope.

```js
// Anonymous class expression
const Animal = class {
  constructor(name) {
    this.name = name;
  }
  speak() {
    return `${this.name} speaks.`;
  }
};

// Named class expression
const Dog = class DogClass {
  getName() {
    return DogClass.name; // "DogClass" — visible inside the class only
  }
};

const a = new Animal("Cat");
console.log(a.speak());    // Cat speaks.

const d = new Dog();
console.log(d.getName());  // DogClass

// console.log(DogClass);  // ReferenceError: DogClass is not defined
```

### Output

```js
Cat speaks.
DogClass
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 12. Abstract-like Classes

JavaScript has no `abstract` keyword. The standard workaround is to use `new.target` in the constructor to prevent direct instantiation, and throw an error in methods that must be overridden.

```js
class AbstractShape {
  constructor() {
    if (new.target === AbstractShape) {
      throw new Error("AbstractShape cannot be instantiated directly.");
    }
  }

  area() {
    throw new Error("Method 'area()' must be implemented.");
  }
}

class Rectangle extends AbstractShape {
  constructor(w, h) {
    super();        // new.target is Rectangle here, so no error
    this.w = w;
    this.h = h;
  }

  area() {
    return this.w * this.h;
  }
}

// new AbstractShape();  // Error: AbstractShape cannot be instantiated directly.

const r = new Rectangle(4, 5);
console.log(r.area()); // 20
```

### Output

```js
20
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 13. Mixins Pattern

JavaScript does not support multiple inheritance. The **mixin pattern** uses higher-order functions that take a base class and return an extended class, allowing behavior from multiple sources to be composed.

```js
const Serializable = (Base) => class extends Base {
  serialize() {
    return JSON.stringify(this);
  }
};

const Validatable = (Base) => class extends Base {
  validate() {
    return Object.keys(this).every(k => this[k] !== null);
  }
};

class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }
}

class EnhancedUser extends Serializable(Validatable(User)) {}

const u = new EnhancedUser("Alice", "alice@example.com");
console.log(u.validate());   // true
console.log(u.serialize());  // {"name":"Alice","email":"alice@example.com"}
```

### Output

```js
true
{"name":"Alice","email":"alice@example.com"}
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 14. instanceof with Classes

`instanceof` checks whether `Constructor.prototype` exists anywhere in an object's prototype chain. Because `extends` sets up the chain, a child instance is also `instanceof` the parent.

```js
class Animal {}
class Dog extends Animal {}
class Cat extends Animal {}

const d = new Dog();

console.log(d instanceof Dog);     // true
console.log(d instanceof Animal);  // true  (Dog's chain includes Animal.prototype)
console.log(d instanceof Cat);     // false (Cat's prototype is not in d's chain)
console.log(d instanceof Object);  // true  (everything inherits from Object)
```

### Output

```js
true
true
false
true
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 15. class vs Constructor Functions

ES6 `class` and the older constructor function pattern produce equivalent prototype structures but differ in several important ways.

```js
// Constructor Function
function PersonFn(name) {
  this.name = name;
}
PersonFn.prototype.greet = function () {
  return `Hi, I'm ${this.name}`;
};

// ES6 Class
class PersonClass {
  constructor(name) {
    this.name = name;
  }
  greet() {
    return `Hi, I'm ${this.name}`;
  }
}

console.log(typeof PersonFn);    // function
console.log(typeof PersonClass); // function (class is a function under the hood)

// Classes enforce the use of new:
try {
  PersonClass("Alice");
} catch (e) {
  console.log(e.message);
  // Class constructor PersonClass cannot be invoked without 'new'
}
```

### Output

```js
function
function
Class constructor PersonClass cannot be invoked without 'new'
```

| Feature | Constructor Function | ES6 Class |
|---|---|---|
| Hoisting | Fully hoisted | Not initialized (TDZ) |
| `new` required | Optional (risky in non-strict) | Enforced by engine |
| Strict mode | Optional | Always strict |
| `super` | Manual via `call`/`apply` | Built-in keyword |
| Static members | Manually assigned on function | `static` keyword |
| Private fields | Convention (`_name`) | True `#name` enforcement |
| Method enumerable | Enumerable by default | Non-enumerable |

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 16. Class Hoisting

Function declarations are fully hoisted. Classes are hoisted in name only — they remain in the **Temporal Dead Zone (TDZ)** until the declaration is evaluated, causing a `ReferenceError` if used before.

```js
// Function constructor — works before declaration in source order
const p1 = new PersonFn("Alice");
console.log(p1.name); // Alice

function PersonFn(name) {
  this.name = name;
}

// Class — ReferenceError if used before declaration
try {
  const p2 = new PersonClass("Bob");
} catch (e) {
  console.log(e.message);
  // Cannot access 'PersonClass' before initialization
}

class PersonClass {
  constructor(name) {
    this.name = name;
  }
}

const p3 = new PersonClass("Charlie");
console.log(p3.name); // Charlie
```

### Output

```js
Alice
Cannot access 'PersonClass' before initialization
Charlie
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 17. Public vs Private Class Fields

**Public fields** are declared in the class body and become own properties on every instance. **Private fields** use `#` and are inaccessible outside the class. Both can have initializer values.

```js
class Config {
  host = "localhost";       // public field
  port = 3000;              // public field
  #apiKey = "secret-123";   // private field

  getKey() {
    return this.#apiKey;
  }
}

const cfg = new Config();
console.log(cfg.host);      // localhost
console.log(cfg.port);      // 3000
console.log(cfg.getKey());  // secret-123
console.log(cfg.apiKey);    // undefined  (no public property named apiKey)
// console.log(cfg.#apiKey); // SyntaxError
```

### Output

```js
localhost
3000
secret-123
undefined
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 18. Static Private Fields

Combining `static` and `#` creates a **private static field** — state that is private to the class and shared across all instances. Subclasses cannot access it either.

```js
class IdGenerator {
  static #nextId = 1;

  static generate() {
    return IdGenerator.#nextId++;
  }
}

console.log(IdGenerator.generate()); // 1
console.log(IdGenerator.generate()); // 2
console.log(IdGenerator.generate()); // 3

// console.log(IdGenerator.#nextId); // SyntaxError
```

### Output

```js
1
2
3
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 19. Chaining with Classes

The **fluent interface** pattern (method chaining) is implemented by returning `this` from each method. Every method call returns the same instance, allowing chained invocations.

```js
class QueryBuilder {
  #query = "";

  select(fields) {
    this.#query += `SELECT ${fields} `;
    return this;
  }

  from(table) {
    this.#query += `FROM ${table} `;
    return this;
  }

  where(condition) {
    this.#query += `WHERE ${condition}`;
    return this;
  }

  build() {
    return this.#query.trim();
  }
}

const sql = new QueryBuilder()
  .select("*")
  .from("users")
  .where("age > 18")
  .build();

console.log(sql); // SELECT * FROM users WHERE age > 18
```

### Output

```js
SELECT * FROM users WHERE age > 18
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 20. OOP Principles in JS

JavaScript supports all four OOP pillars through classes and closures.

**Encapsulation** — bundling data and methods; restricting direct access using private fields.

```js
class Wallet {
  #balance = 0;
  deposit(n) { this.#balance += n; }
  get balance() { return this.#balance; }
}

const w = new Wallet();
w.deposit(50);
console.log(w.balance); // 50
```

**Inheritance** — a child class reuses and extends parent behavior.

```js
class Animal { speak() { return "..."; } }
class Dog extends Animal { speak() { return "Woof!"; } }

console.log(new Dog().speak()); // Woof!
```

**Polymorphism** — the same interface behaves differently depending on the concrete type.

```js
class Shape { area() { return 0; } }

class Circle extends Shape {
  constructor(r) { super(); this.r = r; }
  area() { return (Math.PI * this.r ** 2).toFixed(2); }
}

class Square extends Shape {
  constructor(s) { super(); this.s = s; }
  area() { return (this.s ** 2).toFixed(2); }
}

const shapes = [new Circle(5), new Square(4)];
shapes.forEach(s => console.log(s.area()));
// 78.54
// 16.00
```

**Abstraction** — hiding complexity, exposing only a clean interface (see Section 12 for `new.target` guard pattern).

### Output

```js
50
Woof!
78.54
16.00
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 21. Summary Table

| Feature | Key Point |
|---|---|
| `class` syntax | Syntactic sugar over prototype-based inheritance |
| `constructor` | Invoked automatically on `new`; only one allowed per class |
| Instance methods | Stored on `prototype`; shared by all instances; non-enumerable |
| `static` members | Belong to the class itself, not to instances |
| `extends` | Sets up prototype chain between child and parent |
| `super()` | Calls parent constructor; required before `this` in child |
| `super.method()` | Calls parent's version of an overridden method |
| Private `#field` | Truly private; enforced by engine; inaccessible in subclasses |
| Getter / Setter | Computed property access using `get` / `set` |
| Class expressions | Class as a value — named or anonymous |
| `new.target` | Refers to the constructor originally called with `new` |
| Abstract-like | Use `new.target === Base` check to prevent direct instantiation |
| Mixins | Higher-order class composition for multi-source behavior |
| `instanceof` | Checks if `Constructor.prototype` is in the object's chain |
| Class hoisting | TDZ applies — cannot use a class before its declaration |
| `#field` vs `_field` | True engine-level privacy vs mere naming convention |
| Static private | `static #field` — private state shared at class level |
| Method chaining | Return `this` to enable fluent interfaces |
| OOP pillars | Encapsulation, Inheritance, Polymorphism, Abstraction |

---

## Final Notes

JavaScript's class syntax makes OOP approachable and readable, but the underlying prototype model is always present. Private fields (`#`) provide real encapsulation — not just convention. Understanding the distinction between instance members (on `prototype`) and static members (on the class itself), how `super` works in both constructors and methods, and the TDZ behavior of class declarations will consistently appear in JavaScript interviews at all levels.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
