# Objects in JavaScript

## Table of Contents

1. [Object Creation](#1-object-creation)
2. [Property Access](#2-property-access)
3. [Property Descriptors](#3-property-descriptors)
4. [Object.defineProperty](#4-objectdefineproperty)
5. [Object.freeze vs Object.seal vs Object.preventExtensions](#5-objectfreeze-vs-objectseal-vs-objectpreventextensions)
6. [Object.keys, Object.values, Object.entries](#6-objectkeys-objectvalues-objectentries)
7. [Object.assign — Shallow Merge](#7-objectassign--shallow-merge)
8. [Spread Operator with Objects](#8-spread-operator-with-objects)
9. [Shallow Copy vs Deep Copy](#9-shallow-copy-vs-deep-copy)
10. [JSON.stringify / JSON.parse](#10-jsonstringify--jsonparse)
11. [Computed Property Names](#11-computed-property-names)
12. [Property Shorthand](#12-property-shorthand)
13. [Getter and Setter](#13-getter-and-setter)
14. [Object Destructuring](#14-object-destructuring)
15. [Nested Destructuring](#15-nested-destructuring)
16. [for...in Loop](#16-forin-loop)
17. [hasOwnProperty vs in Operator](#17-hasownproperty-vs-in-operator)
18. [Object Comparison](#18-object-comparison)
19. [Optional Chaining with Objects](#19-optional-chaining-with-objects)
20. [Nullish Coalescing with Objects](#20-nullish-coalescing-with-objects)
21. [Symbol as Property Key](#21-symbol-as-property-key)
22. [Summary](#22-summary)

---

## 1. Object Creation

JavaScript provides four main ways to create objects: object literal, `Object.create()`, constructor functions, and classes (covered in detail in topic 12).

```js
// 1. Object Literal — most common
const person = { name: "Alice", age: 30 };

// 2. Object.create() — sets prototype explicitly
const proto = { greet() { return `Hi, I'm ${this.name}`; } };
const obj = Object.create(proto);
obj.name = "Bob";
console.log(obj.greet()); // Hi, I'm Bob

// 3. Constructor Function
function Car(brand) {
  this.brand = brand;
}
const myCar = new Car("Toyota");
console.log(myCar.brand); // Toyota

// 4. Class syntax (syntactic sugar over prototype)
class Animal {
  constructor(type) {
    this.type = type;
  }
}
const dog = new Animal("Dog");
console.log(dog.type); // Dog
```

### Output

```js
// Hi, I'm Bob
// Toyota
// Dog
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 2. Property Access

Use **dot notation** for known, valid identifiers. Use **bracket notation** for dynamic keys, keys with special characters, or variable-based access.

```js
const user = { name: "Alice", "first-name": "Alice" };

console.log(user.name);          // dot notation
console.log(user["name"]);       // bracket notation — same result
console.log(user["first-name"]); // required for keys with special chars

const key = "name";
console.log(user[key]);          // dynamic key lookup
```

### Output

```js
// Alice
// Alice
// Alice
// Alice
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 3. Property Descriptors

Every object property has a descriptor object with four attributes.

| Attribute | Type | Default (literal) | Description |
|---|---|---|---|
| `value` | any | the value itself | The property's value |
| `writable` | boolean | `true` | Can the value be changed? |
| `enumerable` | boolean | `true` | Shows in `for...in`, `Object.keys`? |
| `configurable` | boolean | `true` | Can the descriptor be changed or the property deleted? |

```js
const obj = { name: "Alice" };

console.log(Object.getOwnPropertyDescriptor(obj, "name"));
```

### Output

```js
// { value: 'Alice', writable: true, enumerable: true, configurable: true }
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 4. Object.defineProperty

`Object.defineProperty` gives precise control over a property's descriptor. Any attribute not specified defaults to `false` / `undefined` when defining a new property this way (unlike literals where defaults are `true`).

```js
const user = { firstName: "John" };

Object.defineProperty(user, "lastName", {
  value: "Doe",
  writable: true,
  enumerable: false,   // hidden from Object.keys / for...in
  configurable: true,
});

console.log(user.lastName);       // accessible directly
console.log(Object.keys(user));   // lastName is hidden

Object.defineProperty(user, "id", {
  value: 42,
  writable: false,     // read-only
  enumerable: true,
  configurable: false, // cannot be redefined or deleted
});

user.id = 99;          // silently fails in non-strict; TypeError in strict
console.log(user.id);
```

### Output

```js
// Doe
// ['firstName']
// 42
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 5. Object.freeze vs Object.seal vs Object.preventExtensions

All three restrict objects in different ways, and all three are **shallow** — nested objects are not affected.

| Method | Add New Props | Delete Props | Modify Existing Props | Shallow Only? |
|---|---|---|---|---|
| `Object.freeze` | No | No | No | Yes |
| `Object.seal` | No | No | Yes | Yes |
| `Object.preventExtensions` | No | Yes | Yes | Yes |

```js
// --- Object.freeze ---
const frozen = Object.freeze({ a: 1, b: { c: 2 } });
frozen.a = 99;    // silently ignored
frozen.d = 4;     // silently ignored
console.log(frozen.a); // 1  (unchanged)
console.log(frozen.b.c = 99); // 99 — nested object is NOT frozen!

// --- Object.seal ---
const sealed = Object.seal({ a: 1 });
sealed.a = 99;    // OK — can modify
sealed.b = 2;     // silently ignored — can't add
delete sealed.a;  // silently ignored — can't delete
console.log(sealed.a); // 99
console.log(sealed.b); // undefined

// --- Object.preventExtensions ---
const noExt = Object.preventExtensions({ x: 1, y: 2 });
noExt.x = 99;    // OK — can modify
noExt.z = 3;     // silently ignored — can't add
delete noExt.y;  // OK — can delete
console.log(noExt.x); // 99
console.log(noExt.y); // undefined (was deleted)
console.log(noExt.z); // undefined (was never added)
```

### Output

```js
// 1
// 99
// 99
// undefined
// 99
// undefined
// undefined
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 6. Object.keys, Object.values, Object.entries

These three static methods iterate over **own enumerable** string-keyed properties only. Inherited and non-enumerable properties are excluded.

```js
const user = { name: "Alice", age: 30, city: "NY" };

console.log(Object.keys(user));    // property names
console.log(Object.values(user));  // property values
console.log(Object.entries(user)); // [key, value] pairs
```

### Output

```js
// ['name', 'age', 'city']
// ['Alice', 30, 'NY']
// [['name', 'Alice'], ['age', 30], ['city', 'NY']]
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 7. Object.assign — Shallow Merge

`Object.assign(target, ...sources)` copies own enumerable properties from sources into target. It **mutates** and returns the target. The copy is **shallow**: nested objects are copied by reference.

```js
const target = { a: 1 };
const source = { b: 2, c: { d: 3 } };

const result = Object.assign(target, source);

console.log(result);         // target is mutated
console.log(target === result); // true — same object

// Shallow: nested object c is shared
result.c.d = 99;
console.log(source.c.d);    // 99 — original is affected
```

### Output

```js
// { a: 1, b: 2, c: { d: 3 } }
// true
// 99
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 8. Spread Operator with Objects

The spread operator `{ ...obj }` creates a new object with copies of own enumerable string-keyed properties. Like `Object.assign`, it is a **shallow copy** — nested objects share the same reference.

```js
const original = { a: 1, b: { c: 2 } };
const copy = { ...original };

copy.a = 99;        // primitive — does NOT affect original
copy.b.c = 99;      // object reference — DOES affect original

console.log(original.a);   // 1  (primitive, unaffected)
console.log(original.b.c); // 99 (reference shared)

// Merging with spread
const defaults = { theme: "light", lang: "en" };
const userPrefs = { lang: "fr" };
const config = { ...defaults, ...userPrefs };
console.log(config); // { theme: 'light', lang: 'fr' }
```

### Output

```js
// 1
// 99
// { theme: 'light', lang: 'fr' }
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 9. Shallow Copy vs Deep Copy

A **shallow copy** copies the top-level properties but nested objects still share references with the original. A **deep copy** creates a fully independent clone at all levels.

```js
// Shallow copy — nested objects share reference
const obj1 = { a: 1, nested: { x: 10 } };
const shallow = { ...obj1 };
shallow.nested.x = 999;
console.log(obj1.nested.x); // 999 — original affected

// Deep copy using JSON (most common for plain objects)
const obj2 = { a: 1, nested: { x: 10 } };
const deep = JSON.parse(JSON.stringify(obj2));
deep.nested.x = 999;
console.log(obj2.nested.x); // 10 — original unaffected

// Deep copy using structuredClone (modern, supports more types)
const obj3 = { a: 1, nested: { x: 10 } };
const deepClone = structuredClone(obj3);
deepClone.nested.x = 999;
console.log(obj3.nested.x); // 10 — original unaffected
```

### Output

```js
// 999
// 10
// 10
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 10. JSON.stringify / JSON.parse

`JSON.stringify` + `JSON.parse` is a common deep copy technique, but it has important limitations.

**Gotchas:**
- Functions are **dropped** (omitted from the result)
- `undefined` values are **dropped**
- `NaN` and `Infinity` become `null`
- `Date` objects become **strings** (not restored as Date on parse)
- Circular references throw a `TypeError`

```js
const obj = {
  name: "Alice",
  score: NaN,
  city: undefined,
  greet: function () { return "hello"; },
  created: new Date("2024-01-01"),
};

const json = JSON.stringify(obj);
console.log(json);

const parsed = JSON.parse(json);
console.log(parsed);
console.log(typeof parsed.created); // string, not Date

// Circular reference — throws
const circular = {};
circular.self = circular;
try {
  JSON.stringify(circular);
} catch (e) {
  console.log(e.message);
}
```

### Output

```js
// '{"name":"Alice","score":null,"created":"2024-01-01T00:00:00.000Z"}'
// { name: 'Alice', score: null, created: '2024-01-01T00:00:00.000Z' }
// string
// Converting circular structure to JSON
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 11. Computed Property Names

Square bracket syntax `[expression]` inside an object literal lets you use a dynamic value as the property name.

```js
const key = "name";
const prefix = "get";
const field = "score";

const obj = {
  [key]: "Alice",          // dynamic key from variable
  [`${prefix}Name`]() {   // dynamic method name using template literal
    return this.name;
  },
  [`min${field.charAt(0).toUpperCase() + field.slice(1)}`]: 0,
};

console.log(obj.name);        // Alice
console.log(obj.getName());   // Alice
console.log(obj.minScore);    // 0
```

### Output

```js
// Alice
// Alice
// 0
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 12. Property Shorthand

When a variable name matches the desired property name, you can omit the value — JavaScript uses the variable's value automatically.

```js
const name = "Alice";
const age = 30;
const city = "NY";

// Old way
const userOld = { name: name, age: age, city: city };

// Shorthand
const user = { name, age, city };

console.log(user); // { name: 'Alice', age: 30, city: 'NY' }

// Also works with method shorthand
const calc = {
  value: 0,
  increment() {       // shorthand for increment: function() {}
    this.value++;
  },
};

calc.increment();
console.log(calc.value); // 1
```

### Output

```js
// { name: 'Alice', age: 30, city: 'NY' }
// 1
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 13. Getter and Setter

Getters (`get`) and setters (`set`) define properties that look like data properties but execute functions when read or written. They are defined with `get` and `set` keywords.

```js
const person = {
  _firstName: "John",
  _lastName: "Doe",

  get fullName() {
    return `${this._firstName} ${this._lastName}`;
  },

  set fullName(value) {
    const parts = value.split(" ");
    this._firstName = parts[0];
    this._lastName = parts[1];
  },
};

console.log(person.fullName); // read via getter

person.fullName = "Jane Smith"; // write via setter
console.log(person._firstName); // Jane
console.log(person._lastName);  // Smith

// Getter with no setter — assignment silently fails in non-strict
const circle = {
  radius: 5,
  get area() {
    return Math.PI * this.radius ** 2;
  },
};

console.log(circle.area.toFixed(2)); // computed on every access
```

### Output

```js
// John Doe
// Jane
// Smith
// 78.54
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 14. Object Destructuring

Destructuring extracts properties from an object into distinct variables. Supports renaming and default values in a single expression.

```js
const user = { name: "Alice", age: 30 };

// Basic
const { name, age } = user;
console.log(name); // Alice
console.log(age);  // 30

// Rename: { originalKey: newVariableName }
const { name: userName } = user;
console.log(userName); // Alice

// Default value — used only when the property is undefined
const { city = "Unknown", age: userAge = 0 } = user;
console.log(city);    // Unknown (not in user)
console.log(userAge); // 30 (exists, default ignored)

// Rest in destructuring
const { name: n, ...rest } = user;
console.log(n);    // Alice
console.log(rest); // { age: 30 }
```

### Output

```js
// Alice
// 30
// Alice
// Unknown
// 30
// Alice
// { age: 30 }
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 15. Nested Destructuring

You can destructure nested objects by using the same syntax recursively.

```js
const data = {
  user: {
    name: "Alice",
    address: {
      city: "New York",
      zip: "10001",
    },
  },
};

// Destructure nested — note: { user: { name, address: { city } } }
const {
  user: {
    name,
    address: { city, zip },
  },
} = data;

console.log(name); // Alice
console.log(city); // New York
console.log(zip);  // 10001

// Note: 'user' and 'address' are NOT declared as variables here
// They are only used as paths. The actual variables are name, city, zip.

// With defaults at nested level
const { user: { phone = "N/A" } } = data;
console.log(phone); // N/A
```

### Output

```js
// Alice
// New York
// 10001
// N/A
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 16. for...in Loop

`for...in` iterates over **all enumerable** string-keyed properties — both **own** and **inherited** (up the prototype chain). This is a common source of bugs.

```js
function Animal(type) {
  this.type = type;
}
Animal.prototype.sound = "generic"; // inherited enumerable property

const dog = new Animal("dog");
dog.name = "Rex";

// Iterates own AND inherited enumerable properties
for (const key in dog) {
  console.log(key);
}

console.log("--- own only ---");

// Best practice: filter to own properties
for (const key in dog) {
  if (dog.hasOwnProperty(key)) {
    console.log(key);
  }
}
```

### Output

```js
// type
// name
// sound
// --- own only ---
// type
// name
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 17. hasOwnProperty vs in Operator

| Check | `hasOwnProperty` | `in` operator |
|---|---|---|
| Own properties | Yes | Yes |
| Inherited properties | No | Yes |

```js
function Car(brand) {
  this.brand = brand;
}
Car.prototype.wheels = 4;

const car = new Car("Toyota");

console.log("brand" in car);               // true  (own)
console.log("wheels" in car);              // true  (inherited)
console.log("color" in car);               // false (doesn't exist)

console.log(car.hasOwnProperty("brand"));  // true  (own)
console.log(car.hasOwnProperty("wheels")); // false (inherited, not own)
```

### Output

```js
// true
// true
// false
// true
// false
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 18. Object Comparison

Objects are **reference types**. Two separately created objects with identical content are NOT equal because they point to different memory addresses.

```js
const a = { x: 1 };
const b = { x: 1 };
const c = a; // c points to the SAME object as a

console.log(a === b); // false — different references
console.log(a === c); // true  — same reference

// Mutation via shared reference
c.x = 99;
console.log(a.x); // 99 — a and c are the same object

// Value comparison — use JSON.stringify (for simple flat objects)
const p = { x: 1 };
const q = { x: 1 };
console.log(JSON.stringify(p) === JSON.stringify(q)); // true
```

### Output

```js
// false
// true
// 99
// true
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 19. Optional Chaining with Objects

The optional chaining operator `?.` short-circuits and returns `undefined` instead of throwing a `TypeError` when accessing a property on `null` or `undefined`.

```js
const user = {
  profile: {
    address: {
      city: "New York",
    },
  },
};

console.log(user?.profile?.address?.city);     // New York
console.log(user?.contact?.phone);              // undefined (no error)
console.log(user?.contact?.phone?.toString()); // undefined (chain stops)

// Optional chaining with methods
console.log(user?.profile?.getName?.());       // undefined (method missing)

// Useful with nullable data
function getCity(user) {
  return user?.profile?.address?.city ?? "Unknown";
}

console.log(getCity(null));  // Unknown
console.log(getCity(user));  // New York
```

### Output

```js
// New York
// undefined
// undefined
// undefined
// Unknown
// New York
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 20. Nullish Coalescing with Objects

The nullish coalescing operator `??` returns the right-hand side value only when the left side is `null` or `undefined`. Unlike `||`, it does **not** treat `0`, `""`, or `false` as falsy.

```js
const config = {
  timeout: 0,       // falsy but intentional
  retries: null,    // explicitly null
  verbose: false,   // falsy but intentional
};

// ?? — only triggers on null / undefined
console.log(config.timeout ?? 5000);     // 0   (0 is NOT null/undefined)
console.log(config.retries ?? 3);        // 3   (null triggers fallback)
console.log(config.verbose ?? true);     // false (false is NOT null/undefined)
console.log(config.missing ?? "default"); // default (property doesn't exist)

// Compare with || — treats all falsy as missing
console.log(config.timeout || 5000);    // 5000 (0 is falsy — bug!)
console.log(config.verbose || true);    // true  (false is falsy — bug!)
```

### Output

```js
// 0
// 3
// false
// default
// 5000
// true
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 21. Symbol as Property Key

`Symbol` creates a unique, non-string key. Symbol-keyed properties are excluded from `Object.keys`, `Object.values`, `Object.entries`, and `for...in`. They are accessible via `Object.getOwnPropertySymbols`.

```js
const id = Symbol("id");
const role = Symbol("role");

const user = {
  name: "Alice",
  [id]: 123,
  [role]: "admin",
};

console.log(user[id]);   // access by symbol reference
console.log(user.name);  // still accessible normally

// Symbols are hidden from standard enumeration
console.log(Object.keys(user));    // ['name'] — no symbols
console.log(Object.values(user));  // ['Alice'] — no symbols

for (const key in user) {
  console.log(key); // only 'name'
}

// Access symbols explicitly
console.log(Object.getOwnPropertySymbols(user)); // [Symbol(id), Symbol(role)]
```

### Output

```js
// 123
// Alice
// ['name']
// ['Alice']
// name
// [Symbol(id), Symbol(role)]
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 22. Summary

| Topic | Key Point |
|---|---|
| Object creation | Literal, `Object.create()`, constructor function, class |
| Dot vs bracket | Bracket required for dynamic keys or special-char keys |
| Property descriptor | `value`, `writable`, `enumerable`, `configurable` |
| `Object.defineProperty` | Precise control over descriptors; unset attributes default to `false` |
| `Object.freeze` | No add, delete, or modify (shallow) |
| `Object.seal` | No add or delete; can modify (shallow) |
| `Object.preventExtensions` | No add; can delete and modify (shallow) |
| `Object.keys/values/entries` | Own enumerable string-keyed properties only |
| `Object.assign` | Shallow merge; mutates and returns target |
| Spread `{ ...obj }` | Shallow copy; does not mutate original |
| Shallow vs deep copy | Shallow shares nested refs; deep creates independent clone |
| `JSON.stringify/parse` | Drops functions and `undefined`; `NaN` → `null`; fails on circular refs |
| Computed property names | `[expression]` as key in object literal |
| Property shorthand | `{ name }` instead of `{ name: name }` |
| Getter / setter | `get`/`set` keywords; look like properties, execute functions |
| Destructuring | Extract values with rename and default support |
| `for...in` | Own + inherited enumerable; use `hasOwnProperty` to filter |
| `hasOwnProperty` | Only own; `in` also checks inherited |
| Object comparison | `{} === {}` is `false`; objects compared by reference |
| Optional chaining `?.` | Returns `undefined` instead of throwing on null access |
| Nullish coalescing `??` | Fallback only for `null`/`undefined`, not `0`, `""`, `false` |
| Symbol key | Unique key; hidden from `Object.keys`, `for...in`, `entries` |

---

## Final Notes

Objects are the foundation of JavaScript. Master property descriptors and `Object.defineProperty` for access control in libraries and frameworks. Always be aware of the difference between shallow and deep copies to avoid subtle bugs when mutating nested data. Leverage modern syntax — destructuring, optional chaining, nullish coalescing, and shorthand properties — to write cleaner, more expressive code. Finally, prefer `Object.keys` or `for...of Object.entries` over `for...in` in production code to avoid accidentally iterating over inherited properties.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
