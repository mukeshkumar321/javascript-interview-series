# ES6+ Features in JavaScript

## Table of Contents

1. [let and const](#1-let-and-const)
2. [Arrow Functions](#2-arrow-functions)
3. [Template Literals](#3-template-literals)
4. [Destructuring](#4-destructuring)
5. [Spread and Rest](#5-spread-and-rest)
6. [Default Parameters](#6-default-parameters)
7. [Modules](#7-modules)
8. [Symbol](#8-symbol)
9. [Map and Set](#9-map-and-set)
10. [WeakMap and WeakSet](#10-weakmap-and-weakset)
11. [Proxy and Reflect](#11-proxy-and-reflect)
12. [Generators and Iterators](#12-generators-and-iterators)
13. [for...of vs for...in](#13-forof-vs-forin)
14. [Promise Enhancements](#14-promise-enhancements)
15. [Optional Chaining (?.)](#15-optional-chaining-)
16. [Nullish Coalescing (??)](#16-nullish-coalescing-)
17. [Logical Assignment Operators](#17-logical-assignment-operators)
18. [Numeric Separators](#18-numeric-separators)
19. [Object.fromEntries](#19-objectfromentries)
20. [Array at() Method](#20-array-at-method)
21. [String at() Method](#21-string-at-method)
22. [structuredClone()](#22-structuredclone)
23. [globalThis](#23-globalthis)
24. [Summary Table](#24-summary-table)

---

## 1. let and const

`let` and `const` are block-scoped and subject to the **Temporal Dead Zone (TDZ)**. `const` prevents reassignment but does not make objects immutable. For full coverage of hoisting and scope, see `02-Scope-Hoisting`.

```js
// TDZ example
try {
  console.log(x);  // ReferenceError
  let x = 5;
} catch (e) {
  console.log(e.message);
  // Cannot access 'x' before initialization
}

// const with objects — contents are mutable
const obj = { a: 1 };
obj.a = 2;           // OK — property mutation allowed
// obj = {};         // TypeError — reassignment not allowed
console.log(obj.a);  // 2
```

### Output

```js
Cannot access 'x' before initialization
2
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 2. Arrow Functions

Arrow functions provide a concise syntax and do not have their own `this`, `arguments`, `super`, or `prototype`. For deep coverage of how `this` behaves with arrow functions, see `04-This-And-Binding`. For generator functions and other function types, see `08-Functions`.

```js
const add = (a, b) => a + b;
console.log(add(2, 3));   // 5

const square = n => n * n;
console.log(square(4));   // 16

// No own 'this' — arrow inherits from enclosing scope
function Timer() {
  this.seconds = 0;
  setInterval(() => {
    this.seconds++;  // 'this' is Timer instance, not global
  }, 1000);
}
```

### Output

```js
5
16
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 3. Template Literals

Template literals use backticks and support embedded expressions (`${expr}`), multi-line strings, and **tagged templates** where a function processes the template parts.

```js
// Tagged template
function highlight(strings, ...values) {
  return strings.reduce((result, str, i) => {
    return result + str + (values[i] !== undefined ? `[${values[i]}]` : "");
  }, "");
}

const name = "Alice";
const score = 95;
console.log(highlight`Name: ${name}, Score: ${score}`);
// Name: [Alice], Score: [95]

// String.raw — a built-in tag that gives the raw string (no escape processing)
console.log(String.raw`Line1\nLine2`);
// Line1\nLine2  (the \n is NOT converted to a newline)

// Nested template literals
const items = ["a", "b", "c"];
const html = `<ul>${items.map(i => `<li>${i}</li>`).join("")}</ul>`;
console.log(html);
// <ul><li>a</li><li>b</li><li>c</li></ul>
```

### Output

```js
Name: [Alice], Score: [95]
Line1\nLine2
<ul><li>a</li><li>b</li><li>c</li></ul>
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 4. Destructuring

Destructuring supports defaults, renaming, nested patterns, skipping elements, and rest collection. For cross-reference with advanced patterns in arrays and objects, see `01-Fundamentals` and `09-10` (Arrays/Objects).

```js
// Object destructuring: rename + default
const { a: renamed = 10, b: { c: nested } = {} } = { a: 5, b: { c: 99 } };
console.log(renamed);  // 5
console.log(nested);   // 99

// Array destructuring: skip + default
const [, second, , fourth = "default"] = [1, 2, 3];
console.log(second);   // 2
console.log(fourth);   // default

// Swapping variables
let x = 1, y = 2;
[x, y] = [y, x];
console.log(x, y);     // 2 1

// Function parameter destructuring with defaults
function greet({ name = "World", greeting = "Hello" } = {}) {
  return `${greeting}, ${name}!`;
}
console.log(greet({ name: "Alice" }));  // Hello, Alice!
console.log(greet());                   // Hello, World!
```

### Output

```js
5
99
2
default
2 1
Hello, Alice!
Hello, World!
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 5. Spread and Rest

**Spread** (`...`) expands an iterable into individual elements. **Rest** (`...`) collects remaining elements into an array or object. Spread creates **shallow** copies.

```js
// Spread for shallow merging — nested objects are still shared
const original = { x: 1, y: { z: 2 } };
const copy = { ...original, w: 3 };
copy.y.z = 99;
console.log(original.y.z); // 99 (shallow copy — nested reference is shared)
console.log(copy.w);        // 3

// Rest in object destructuring
const { x: first, ...rest } = { x: 1, y: 2, z: 3 };
console.log(first); // 1
console.log(rest);  // { y: 2, z: 3 }

// Rest in function parameters
function sum(first, ...others) {
  return others.reduce((acc, n) => acc + n, first);
}
console.log(sum(1, 2, 3, 4)); // 10
```

### Output

```js
99
3
1
{ y: 2, z: 3 }
10
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 6. Default Parameters

Default parameters allow function arguments to have fallback values when `undefined` is passed. For detailed coverage, see `08-Functions`.

```js
function greet(name = "World", greeting = "Hello") {
  return `${greeting}, ${name}!`;
}

console.log(greet());                    // Hello, World!
console.log(greet("Alice"));             // Hello, Alice!
console.log(greet(undefined, "Hi"));     // Hi, World! (undefined triggers default)
console.log(greet(null, "Hey"));         // Hey, null  (null does NOT trigger default)
```

### Output

```js
Hello, World!
Hello, Alice!
Hi, World!
Hey, null
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 7. Modules

ES Modules provide a **static** module system with named exports, default exports, live bindings, and dynamic imports. Modules are always in strict mode.

```js
// ---- math.js ----
export const add = (a, b) => a + b;
export const sub = (a, b) => a - b;
export default function multiply(a, b) { return a * b; }

// ---- utils.js ----
export { add as sum } from "./math.js"; // re-export with rename

// ---- app.js ----
import multiply, { add, sub } from "./math.js"; // default + named
import * as math from "./math.js";              // namespace import

console.log(add(2, 3));        // 5
console.log(sub(5, 2));        // 3
console.log(multiply(4, 3));   // 12
console.log(math.add(1, 1));   // 2

// Dynamic import — returns a Promise
async function loadMath() {
  const { add } = await import("./math.js");
  console.log(add(10, 5));    // 15
}
```

### Output

```js
5
3
12
2
15
```

> **Key facts about ES Modules:**
> - Imports are **live bindings** — if the exported value changes, the import reflects the change.
> - Modules are executed only **once**, even if imported multiple times.
> - `import` declarations are **hoisted** and evaluated before the rest of the module.
> - Dynamic `import()` is the only way to conditionally or lazily load a module.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 8. Symbol

`Symbol` is a primitive type that produces a **guaranteed unique** value each time it is called. Symbols are commonly used as unique object keys and for defining custom behaviors via **well-known symbols**.

```js
// Every Symbol is unique
const s1 = Symbol("id");
const s2 = Symbol("id");
console.log(s1 === s2);  // false
console.log(typeof s1);  // symbol

// Symbol as a unique object key
const KEY = Symbol("key");
const obj = { [KEY]: "private-value", key: "public-value" };
console.log(obj[KEY]);                          // private-value
console.log(obj.key);                           // public-value
console.log(Object.keys(obj));                  // ["key"]  (Symbol key omitted)
console.log(JSON.stringify(obj));               // {"key":"public-value"}  (Symbol omitted)

// Well-known symbol: Symbol.iterator
class Range {
  constructor(start, end) { this.start = start; this.end = end; }

  [Symbol.iterator]() {
    let current = this.start;
    const end = this.end;
    return {
      next() {
        return current <= end
          ? { value: current++, done: false }
          : { done: true, value: undefined };
      }
    };
  }
}

console.log([...new Range(1, 4)]); // [1, 2, 3, 4]
```

### Output

```js
false
symbol
private-value
public-value
[ 'key' ]
{"key":"public-value"}
[ 1, 2, 3, 4 ]
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 9. Map and Set

**`Map`** is a key-value collection that accepts any type as a key and preserves insertion order. **`Set`** stores unique values of any type in insertion order. Both differ significantly from plain objects and arrays.

```js
// Map — any key type, ordered
const map = new Map();
const objKey = { id: 1 };
map.set(objKey, "object key value");
map.set(1, "number key");
map.set("1", "string key");

console.log(map.get(objKey));  // object key value
console.log(map.get(1));       // number key
console.log(map.get("1"));     // string key  (1 and "1" are different keys)
console.log(map.size);         // 3

// Set — unique values
const set = new Set([1, 2, 2, 3, 3, 3]);
console.log(set.size);         // 3
console.log([...set]);         // [1, 2, 3]

// Map vs Object: Object coerces all keys to strings
const obj = {};
obj[1] = "num";
obj["1"] = "str";
console.log(Object.keys(obj).length); // 1  (1 and "1" are the same key!)
console.log(obj[1]);                  // str
```

### Output

```js
object key value
number key
string key
3
3
[ 1, 2, 3 ]
1
str
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 10. WeakMap and WeakSet

`WeakMap` and `WeakSet` hold **weak references** to their keys/values (which must be objects). They are **not iterable**, have no `size` property, and allow garbage collection of their entries when the object is no longer referenced elsewhere.

```js
// WeakMap — keys must be objects
let user = { name: "Alice" };
const wm = new WeakMap();
wm.set(user, { role: "admin" });

console.log(wm.has(user));   // true
console.log(wm.get(user));   // { role: 'admin' }

user = null; // user object can now be garbage collected
// wm.size      // undefined — WeakMap has no size
// [...wm]      // TypeError — not iterable

// WeakMap as private data store
const _private = new WeakMap();

class Person {
  constructor(name, age) {
    _private.set(this, { name, age });
  }
  greet() {
    const { name, age } = _private.get(this);
    return `Hi, I'm ${name}, ${age} years old.`;
  }
}

const p = new Person("Bob", 25);
console.log(p.greet());              // Hi, I'm Bob, 25 years old.
console.log(Object.keys(p).length);  // 0  (no public own properties)

// WeakSet
const seen = new WeakSet();
const nodeA = { id: 1 };
seen.add(nodeA);
console.log(seen.has(nodeA)); // true
console.log(seen.size);       // undefined
```

### Output

```js
true
{ role: 'admin' }
Hi, I'm Bob, 25 years old.
0
true
undefined
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 11. Proxy and Reflect

`Proxy` wraps an object and intercepts operations (get, set, delete, has, etc.) via **traps**. `Reflect` provides the default behavior counterpart for each trap and is used to forward operations cleanly.

```js
// Proxy with get and set traps
const handler = {
  get(target, prop, receiver) {
    return prop in target
      ? Reflect.get(target, prop, receiver)
      : `Property "${prop}" not found`;
  },
  set(target, prop, value) {
    if (typeof value !== "number") {
      throw new TypeError("Only numbers allowed");
    }
    return Reflect.set(target, prop, value);
  }
};

const proxy = new Proxy({}, handler);
proxy.x = 42;
console.log(proxy.x);        // 42
console.log(proxy.missing);  // Property "missing" not found

try {
  proxy.y = "hello";
} catch (e) {
  console.log(e.message);    // Only numbers allowed
}

// Proxy with has trap — customize 'in' operator
const range = new Proxy({ min: 1, max: 10 }, {
  has(target, prop) {
    const num = Number(prop);
    return num >= target.min && num <= target.max;
  }
});

console.log(5 in range);   // true
console.log(11 in range);  // false
```

### Output

```js
42
Property "missing" not found
Only numbers allowed
true
false
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 12. Generators and Iterators

Generator functions (`function*`) produce lazy sequences using `yield`. They return an iterator. For deeper coverage of generator patterns and async generators, see `08-Functions`.

```js
function* count(n) {
  for (let i = 1; i <= n; i++) {
    yield i;
  }
}

const gen = count(3);
console.log(gen.next()); // { value: 1, done: false }
console.log(gen.next()); // { value: 2, done: false }
console.log(gen.next()); // { value: 3, done: false }
console.log(gen.next()); // { value: undefined, done: true }

// Generators are iterable
console.log([...count(5)]); // [1, 2, 3, 4, 5]
```

### Output

```js
{ value: 1, done: false }
{ value: 2, done: false }
{ value: 3, done: false }
{ value: undefined, done: true }
[ 1, 2, 3, 4, 5 ]
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 13. for...of vs for...in

`for...in` iterates **enumerable property keys** (including inherited ones) as strings. `for...of` iterates **iterable values** (arrays, strings, Maps, Sets, generators). Do not use `for...in` to iterate arrays.

```js
const arr = [10, 20, 30];
arr.extra = "hello";           // adding a custom property

const forInKeys = [];
const forOfVals = [];

for (const k in arr) forInKeys.push(k);    // iterates enumerable keys
for (const v of arr) forOfVals.push(v);    // iterates values

console.log(forInKeys); // ["0", "1", "2", "extra"]  (extra key included!)
console.log(forOfVals); // [10, 20, 30]              (only array values)

// for...of on a string iterates Unicode code points
const chars = [];
for (const c of "hello") chars.push(c);
console.log(chars); // ['h', 'e', 'l', 'l', 'o']

// for...in on an object with inherited props
function Base() { this.own = 1; }
Base.prototype.inherited = 2;
const obj = new Base();
const keys = [];
for (const k in obj) keys.push(k);
console.log(keys); // ["own", "inherited"]
```

### Output

```js
[ '0', '1', '2', 'extra' ]
[ 10, 20, 30 ]
[ 'h', 'e', 'l', 'l', 'o' ]
[ 'own', 'inherited' ]
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 14. Promise Enhancements

ES2017–ES2020 added several `Promise` combinators. For full coverage of `async/await`, `Promise.all`, `Promise.race`, `Promise.allSettled`, and `Promise.any`, see `06-Async-Await-Promises`.

| Method | Resolves when | Rejects when |
|---|---|---|
| `Promise.all` | All promises resolve | Any one rejects |
| `Promise.allSettled` | All promises settle (resolve or reject) | Never |
| `Promise.race` | First promise settles | First promise rejects |
| `Promise.any` | First promise resolves | All promises reject |

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 15. Optional Chaining (?.)

`?.` short-circuits to `undefined` if the left-hand side is `null` or `undefined`, preventing `TypeError` on deeply nested property access. Works on properties, method calls, and array-index access.

```js
const user = {
  profile: {
    address: { city: "NYC" },
    getName() { return "Alice"; }
  }
};

console.log(user?.profile?.address?.city);   // NYC
console.log(user?.settings?.theme);          // undefined (short-circuits at settings)
console.log(user?.profile?.getName?.());     // Alice
console.log(user?.profile?.missing?.());     // undefined (short-circuits at missing)

const arr = [1, 2, 3];
console.log(arr?.[0]);   // 1
console.log(null?.[0]);  // undefined
```

### Output

```js
NYC
undefined
Alice
undefined
1
undefined
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 16. Nullish Coalescing (??)

`??` returns the right-hand side only when the left-hand side is **`null` or `undefined`** — unlike `||`, which triggers on any falsy value (`0`, `""`, `false`).

```js
console.log(null ?? "default");      // default
console.log(undefined ?? "default"); // default
console.log(0 ?? "default");         // 0    (0 is not null/undefined)
console.log("" ?? "default");        // ""   ("" is not null/undefined)
console.log(false ?? "default");     // false

// Comparison with || (OR)
console.log(0 || "fallback");        // fallback (0 is falsy)
console.log("" || "fallback");       // fallback ("" is falsy)
console.log(false || "fallback");    // fallback (false is falsy)
```

### Output

```js
default
default
0

false
fallback
fallback
fallback
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 17. Logical Assignment Operators

Logical assignment operators combine a logical operation with assignment in a single expression. They **short-circuit** — the right-hand side is only evaluated if the assignment will actually happen.

```js
// ??= (nullish assignment): assign only if left side is null or undefined
let a = null;
let b = 0;
a ??= "assigned";
b ??= "assigned";
console.log(a); // assigned  (null triggers)
console.log(b); // 0         (0 is not null/undefined — no assignment)

// ||= (OR assignment): assign only if left side is falsy
let c = 0;
let d = 5;
c ||= 99;
d ||= 99;
console.log(c); // 99  (0 is falsy)
console.log(d); // 5   (5 is truthy — no assignment)

// &&= (AND assignment): assign only if left side is truthy
let e = 1;
let f = 0;
let sideEffects = 0;
e &&= (++sideEffects, 42);
f &&= (++sideEffects, 42);
console.log(e);           // 42  (1 is truthy — assignment ran)
console.log(f);           // 0   (0 is falsy — right side was NOT evaluated)
console.log(sideEffects); // 1   (only one side effect ran)
```

### Output

```js
assigned
0
99
5
42
0
1
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 18. Numeric Separators

Numeric separators (`_`) can be placed inside numeric literals for readability. They have **no effect on the value** and are stripped at parse time.

```js
const million  = 1_000_000;
const bytes    = 0xFF_FF_FF;
const bits     = 0b1010_0001;
const fraction = 1_234.567_89;

console.log(million);   // 1000000
console.log(bytes);     // 16777215
console.log(bits);      // 161
console.log(fraction);  // 1234.56789

// Separators are not allowed at the start, end, or next to a decimal point
// 1_.0  // SyntaxError
// _1    // treated as identifier, not a number
```

### Output

```js
1000000
16777215
161
1234.56789
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 19. Object.fromEntries

`Object.fromEntries()` converts an iterable of `[key, value]` pairs into an object. It is the inverse of `Object.entries()` and also works directly with `Map` instances.

```js
// From an array of pairs
const entries = [["name", "Alice"], ["age", 30]];
console.log(Object.fromEntries(entries));
// { name: 'Alice', age: 30 }

// From a Map
const map = new Map([["a", 1], ["b", 2]]);
console.log(Object.fromEntries(map));
// { a: 1, b: 2 }

// Transforming object values — a clean pattern
const prices = { apple: 1.5, banana: 0.5, cherry: 3.0 };
const doubled = Object.fromEntries(
  Object.entries(prices).map(([k, v]) => [k, v * 2])
);
console.log(doubled);
// { apple: 3, banana: 1, cherry: 6 }
```

### Output

```js
{ name: 'Alice', age: 30 }
{ a: 1, b: 2 }
{ apple: 3, banana: 1, cherry: 6 }
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 20. Array at() Method

`Array.prototype.at()` accepts both positive and **negative** indices. Negative indices count from the end of the array (`-1` is the last element). This eliminates the need for `arr[arr.length - 1]`.

```js
const arr = [10, 20, 30, 40, 50];

console.log(arr.at(0));    // 10   (first element)
console.log(arr.at(2));    // 30
console.log(arr.at(-1));   // 50   (last element)
console.log(arr.at(-2));   // 40   (second to last)
console.log(arr.at(10));   // undefined (out of bounds)

// Clean last-element access
const last = arr.at(-1);
console.log(last === arr[arr.length - 1]); // true
```

### Output

```js
10
30
50
40
undefined
true
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 21. String at() Method

`String.prototype.at()` works identically to `Array.prototype.at()` — it accepts negative indices and returns the character at that position. It is Unicode-safe for BMP characters.

```js
const str = "Hello";

console.log(str.at(0));    // H
console.log(str.at(-1));   // o   (last character)
console.log(str.at(-2));   // l

// String length vs for...of with emoji (Unicode code points)
const emoji = "hi 👋";
console.log(emoji.length);          // 5  (emoji is 2 UTF-16 code units)
console.log([...emoji].length);     // 4  (for...of counts code points)
console.log(emoji.at(-1));          // (half of emoji — use [...str].at(-1) for code point)
console.log([...emoji].at(-1));     // 👋 (correct last code point)
```

### Output

```js
H
o
l
5
4
👋
```

> Note: `str.at(-1)` on a string containing multi-unit emoji characters returns the last UTF-16 code unit, which may be half of a surrogate pair. Use `[...str].at(-1)` to get the last Unicode code point safely.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 22. structuredClone()

`structuredClone()` creates a **deep clone** of a value using the Structured Clone Algorithm. It correctly handles nested objects, arrays, `Map`, `Set`, `Date`, `RegExp`, and circular references — unlike spread or `JSON.parse/stringify`.

```js
const original = {
  name: "Alice",
  address: { city: "NYC" },
  hobbies: ["reading", "coding"],
  created: new Date("2024-01-01")
};

const clone = structuredClone(original);

// Modifying clone does not affect original
clone.address.city = "LA";
clone.hobbies.push("gaming");

console.log(original.address.city);    // NYC  (unaffected)
console.log(original.hobbies.length);  // 2    (unaffected)
console.log(clone.created instanceof Date); // true  (Date preserved correctly)

// JSON.parse/stringify loses the Date type
const jsonClone = JSON.parse(JSON.stringify(original));
console.log(jsonClone.created instanceof Date); // false (becomes a string)
```

### Output

```js
NYC
2
true
false
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 23. globalThis

`globalThis` provides a **universal** reference to the global object regardless of the environment (browser, Node.js, Web Worker). It eliminates the need for environment-specific checks like `window`, `global`, or `self`.

```js
// Works in any environment:
console.log(typeof globalThis);       // object
console.log(globalThis === globalThis); // true

// Storing a value on the global object
globalThis.appVersion = "1.0.0";
console.log(appVersion);              // 1.0.0

// Environment detection
const isBrowser = typeof globalThis.window !== "undefined";
const isNode    = typeof globalThis.process !== "undefined";
console.log(typeof isBrowser); // boolean
console.log(typeof isNode);    // boolean
```

### Output

```js
object
true
1.0.0
boolean
boolean
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 24. Summary Table

| Feature | Key Point |
|---|---|
| `let` / `const` | Block-scoped; TDZ applies; `const` prevents reassignment only |
| Arrow functions | No own `this`, `arguments`, `super`, or `prototype` |
| Template literals | Backtick strings; tagged templates; `String.raw` |
| Destructuring | Defaults, renaming, nesting, rest, and parameter patterns |
| Spread `...` | Expands iterables; creates shallow copies |
| Rest `...` | Collects remaining args or properties |
| ES Modules | Static `import`/`export`; live bindings; `import()` for dynamic load |
| `Symbol` | Unique primitive; unique keys; well-known symbols (iterator, etc.) |
| `Map` | Any key type; ordered; `size` property; iterable |
| `Set` | Unique values; ordered; iterable |
| `WeakMap` | Object-keyed; weak refs; not iterable; no `size` |
| `WeakSet` | Object values; weak refs; not iterable; no `size` |
| `Proxy` | Intercepts object operations (get, set, has, delete, etc.) |
| `Reflect` | Mirror of Proxy traps; invokes default behavior |
| Generators | `function*` + `yield`; lazy sequences; full iterator protocol |
| `for...of` | Iterates **values** of any iterable |
| `for...in` | Iterates **enumerable keys** (own + inherited) |
| `?.` | Optional chaining; short-circuits on `null`/`undefined` |
| `??` | Nullish coalescing; fallback only for `null`/`undefined` |
| `&&=`, `\|\|=`, `??=` | Logical assignment with short-circuit evaluation |
| Numeric separators | `1_000_000`; readability only; no runtime effect |
| `Object.fromEntries` | Converts `[k,v]` pairs or `Map` to plain object |
| `Array.at()` / `String.at()` | Negative index support; cleaner end-access |
| `structuredClone()` | True deep clone; handles `Map`, `Set`, `Date`, circular refs |
| `globalThis` | Universal global object reference across all environments |

---

## Final Notes

ES6 and the subsequent annual releases have fundamentally changed how JavaScript is written. The most interview-critical features are modules (live bindings, tree-shaking), `Symbol` (especially well-known symbols for custom iteration and `instanceof`), `Proxy`/`Reflect` (used heavily in reactivity systems like Vue 3), logical assignment operators (common in configuration code), and `structuredClone` (a replacement for `JSON.parse(JSON.stringify(x))`). Knowing not just the syntax but the subtle differences — `??` vs `||`, `WeakMap` vs `Map`, `for...of` vs `for...in` — is what separates a solid JavaScript engineer in an interview.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
