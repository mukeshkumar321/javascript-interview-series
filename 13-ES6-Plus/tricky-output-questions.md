# ES6+ Features — Tricky Output Questions

## Table of Contents

1. [Module Questions](#1-module-questions)
2. [Symbol Questions](#2-symbol-questions)
3. [Map and Set Questions](#3-map-and-set-questions)
4. [WeakMap and WeakSet Questions](#4-weakmap-and-weakset-questions)
5. [Proxy Questions](#5-proxy-questions)
6. [Logical Assignment Questions](#6-logical-assignment-questions)
7. [for...of vs for...in Questions](#7-forof-vs-forin-questions)
8. [Advanced ES6+ Questions](#8-advanced-es6-questions)

---

## 1. Module Questions

---

### Q1. What will be the output? (conceptual — live bindings)

```js
// counter.js
export let count = 0;
export function increment() {
  count++;
}

// main.js
import { count, increment } from "./counter.js";

console.log(count);   // A
increment();
increment();
console.log(count);   // B
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
A: 0
B: 2
```

### Explanation
ES module imports are **live bindings** — they reflect the current value of the exported variable. Unlike CommonJS `require()`, which copies the value at import time, ES module `count` in `main.js` stays in sync with `counter.js`'s `count`. After two `increment()` calls, `count` is `2` when read in `main.js`.

</details>

---

### Q2. What will happen? (default vs named export conflict)

```js
// lib.js
export const value = 10;
export default function getValue() { return value; }

// app.js — which line is valid?
import getValue from "./lib.js";          // (A)
import { getValue } from "./lib.js";      // (B)
import { value } from "./lib.js";         // (C)
import defaultExport from "./lib.js";     // (D)
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
(A) — valid: imports the default export as 'getValue'
(B) — SyntaxError / undefined: no named export called 'getValue' exists
(C) — valid: imports the named export 'value' (value = 10)
(D) — valid: imports the default export, renamed to 'defaultExport'
```

### Explanation
A module's **default export** is imported without braces. The function in `lib.js` is exported as `default`, not as a named export called `getValue`. Line (B) tries to import a **named** export called `getValue`, which does not exist — this either throws a SyntaxError in strict ES module environments or results in `undefined`. Lines (A) and (D) both correctly import the default export under different local names.

</details>

---

### Q3. What will be the output?

```js
// moduleA.js
console.log("moduleA executing");
export const x = 1;

// main.js
import { x } from "./moduleA.js";
import { x as y } from "./moduleA.js";

console.log(x);
console.log(y);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
moduleA executing
1
1
```

### Explanation
ES modules are **evaluated only once**, regardless of how many times they are imported. Even though `moduleA.js` is imported twice (once as `x`, once as `x` renamed to `y`), the module body executes only once. Both `x` and `y` are live bindings to the same exported `x` variable, so both read `1`.

</details>

---

## 2. Symbol Questions

---

### Q1. What will be the output?

```js
const s1 = Symbol("foo");
const s2 = Symbol("foo");

console.log(s1 === s2);
console.log(s1 == s2);
console.log(s1.toString() === s2.toString());
console.log(s1.description === s2.description);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
false
false
true
true
```

### Explanation
Every call to `Symbol()` creates a **unique** symbol, even if the description string is identical. `s1 === s2` and `s1 == s2` are both `false`. However, `toString()` returns `"Symbol(foo)"` for both, and `.description` returns `"foo"` for both — so those string comparisons are `true`. The description is purely a label for debugging and has no effect on uniqueness.

</details>

---

### Q2. What will be the output?

```js
const id = Symbol("id");
const user = {
  name: "Alice",
  [id]: 123
};

console.log(Object.keys(user));
console.log(Object.getOwnPropertyNames(user));
console.log(Object.getOwnPropertySymbols(user));
console.log(JSON.stringify(user));
console.log(user[id]);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
[ 'name' ]
[ 'name' ]
[ Symbol(id) ]
{"name":"Alice"}
123
```

### Explanation
Symbol-keyed properties are invisible to `Object.keys()`, `Object.getOwnPropertyNames()`, `for...in`, and `JSON.stringify()`. They are only visible via `Object.getOwnPropertySymbols()` or `Reflect.ownKeys()`. This makes Symbols useful for adding metadata to objects without polluting their public key space.

</details>

---

### Q3. What will be the output?

```js
const s1 = Symbol.for("shared");
const s2 = Symbol.for("shared");
const s3 = Symbol("shared");       // NOT in global registry

console.log(s1 === s2);
console.log(s1 === s3);
console.log(Symbol.keyFor(s1));
console.log(Symbol.keyFor(s3));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
true
false
shared
undefined
```

### Explanation
`Symbol.for(key)` looks up — or creates — a symbol in the **global symbol registry**. Multiple calls with the same key return the exact same symbol. `Symbol("shared")` bypasses the registry and always creates a new unique symbol. `Symbol.keyFor()` only works for symbols registered via `Symbol.for()` — it returns `undefined` for locally created symbols.

</details>

---

### Q4. What will be the output?

```js
class Collection {
  #items = [];

  add(item) {
    this.#items.push(item);
    return this;
  }

  [Symbol.iterator]() {
    let i = 0;
    const items = this.#items;
    return {
      next() {
        return i < items.length
          ? { value: items[i++], done: false }
          : { value: undefined, done: true };
      }
    };
  }
}

const col = new Collection();
col.add("a").add("b").add("c");

console.log([...col]);
for (const item of col) {
  process.stdout.write(item + " ");
}
console.log();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
[ 'a', 'b', 'c' ]
a b c 
```

### Explanation
Implementing `[Symbol.iterator]()` makes any object **iterable**. The spread operator and `for...of` both call `[Symbol.iterator]()` to get an iterator, then call `.next()` repeatedly until `done: true`. The method chaining (`add` returns `this`) works because each `add` call returns the same `Collection` instance.

</details>

---

## 3. Map and Set Questions

---

### Q1. What will be the output?

```js
const map = new Map();
map.set(1, "number one");
map.set("1", "string one");
map.set(true, "boolean true");

console.log(map.get(1));
console.log(map.get("1"));
console.log(map.get(true));
console.log(map.size);

const obj = {};
obj[1] = "num";
obj["1"] = "override";
obj[true] = "bool";
console.log(Object.keys(obj).length);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
number one
string one
boolean true
3
2
```

### Explanation
`Map` uses **SameValueZero** equality for keys, so `1`, `"1"`, and `true` are three distinct keys. Plain objects coerce all keys to strings: `1` → `"1"` and `"1"` → `"1"` are the same key (second write overwrites the first). `true` coerces to `"true"`, which is a different key from `"1"`. So the object has 2 keys: `"1"` and `"true"`.

</details>

---

### Q2. What will be the output?

```js
const set = new Set([3, 1, 4, 1, 5, 9, 2, 6, 5, 3]);

console.log(set.size);
console.log([...set][0]);

set.delete(9);
console.log(set.has(9));
console.log(set.has(3));

const sorted = [...set].sort((a, b) => a - b);
console.log(sorted);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
7
3
false
true
[ 1, 2, 3, 4, 5, 6 ]
```

### Explanation
`Set` keeps only **unique values** in **insertion order**. Duplicates (`1`, `5`, `3` appearing twice each) are silently ignored. The unique values in insertion order are `[3, 1, 4, 5, 9, 2, 6]` — size is `7` and the first element is `3`. After `delete(9)`, `has(9)` is `false`. Spreading and sorting gives `[1, 2, 3, 4, 5, 6]`.

</details>

---

### Q3. What will be the output?

```js
const map = new Map()
  .set("a", 1)
  .set("b", 2)
  .set("c", 3);

for (const [key, val] of map) {
  console.log(`${key}: ${val}`);
}

console.log([...map.keys()]);
console.log([...map.values()]);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
a: 1
b: 2
c: 3
[ 'a', 'b', 'c' ]
[ 1, 2, 3 ]
```

### Explanation
`Map.prototype.set()` returns the Map instance, so calls can be chained. `for...of` on a Map iterates `[key, value]` pairs in **insertion order**, allowing destructuring directly in the loop header. `.keys()` and `.values()` return iterators that can be spread into arrays.

</details>

---

## 4. WeakMap and WeakSet Questions

---

### Q1. What will be the output?

```js
const wm = new WeakMap();
const obj = { name: "test" };

wm.set(obj, 42);
console.log(wm.has(obj));
console.log(wm.get(obj));

// Attempt to iterate or get size
console.log(wm.size);
console.log(typeof wm[Symbol.iterator]);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
true
42
undefined
undefined
```

### Explanation
`WeakMap` has no `size` property (it is `undefined`, not `0`) and no `[Symbol.iterator]` method, so it cannot be iterated. This is by design — if WeakMap were iterable, iterating it could prevent garbage collection of its keys. `has()` and `get()` work normally when the key object is still referenced.

</details>

---

### Q2. What will be the output?

```js
const cache = new WeakMap();

function process(obj) {
  if (cache.has(obj)) {
    console.log("cache hit:", cache.get(obj));
    return;
  }
  const result = obj.value * 2;
  cache.set(obj, result);
  console.log("computed:", result);
}

const data = { value: 21 };
process(data);
process(data);
process({ value: 21 });  // different object reference
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
computed: 42
cache hit: 42
computed: 42
```

### Explanation
`WeakMap` keys are compared by **object identity** (reference), not by value. The first call computes and caches the result for `data`. The second call hits the cache because it uses the same `data` reference. The third call passes a new `{ value: 21 }` — a different object in memory — so `cache.has()` returns `false` and the value is recomputed.

</details>

---

### Q3. What will be the output?

```js
const ws = new WeakSet();
const a = { id: 1 };
const b = { id: 2 };

ws.add(a);
ws.add(a);   // duplicate — no effect
ws.add(b);

console.log(ws.has(a));
console.log(ws.has(b));
console.log(ws.has({ id: 1 })); // new object, different reference

ws.delete(b);
console.log(ws.has(b));
console.log(ws.size);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
true
true
false
false
undefined
```

### Explanation
`WeakSet` only stores **unique object references**. Adding `a` twice has no effect. `{ id: 1 }` creates a **new** object in memory — it is not the same reference as `a`, so `has({ id: 1 })` returns `false`. After `delete(b)`, `has(b)` returns `false`. `ws.size` is `undefined` because `WeakSet` has no `size` property.

</details>

---

## 5. Proxy Questions

---

### Q1. What will be the output?

```js
const handler = {
  get(target, prop) {
    return prop in target ? target[prop] : 37;
  }
};

const p = new Proxy({ a: 1, b: undefined }, handler);

console.log(p.a);
console.log(p.b);
console.log(p.c);
console.log("a" in p);
console.log("c" in p);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
1
undefined
37
true
false
```

### Explanation
The `get` trap fires for every property access. `p.a` returns `1` (exists in target). `p.b` returns `undefined` — the property **exists** in the target (with value `undefined`), so `prop in target` is `true` and `target[prop]` (which is `undefined`) is returned. `p.c` triggers the fallback `37` since `"c"` is not a key in target. The `in` operator checks the actual target object directly (no `has` trap defined), so `"a" in p` is `true` and `"c" in p` is `false`.

</details>

---

### Q2. What will be the output?

```js
const validator = {
  set(target, prop, value) {
    if (prop === "age") {
      if (!Number.isInteger(value) || value < 0 || value > 150) {
        throw new RangeError(`Invalid age: ${value}`);
      }
    }
    target[prop] = value;
    return true;
  }
};

const person = new Proxy({}, validator);
person.name = "Alice";
person.age = 30;
console.log(person.name);
console.log(person.age);

try {
  person.age = 200;
} catch (e) {
  console.log(e.message);
}

console.log(person.age);  // unchanged after failed set
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Alice
30
Invalid age: 200
30
```

### Explanation
The `set` trap intercepts all property assignments. `name` has no special validation and is set normally. `age = 30` passes validation. `age = 200` fails the range check and throws before `target[prop] = value` runs — so the original value `30` is preserved. The `set` trap must return `true` to signal success; returning `false` in strict mode would throw a `TypeError`.

</details>

---

### Q3. What will be the output?

```js
let callCount = 0;

const handler = {
  get(target, prop, receiver) {
    callCount++;
    return Reflect.get(target, prop, receiver);
  }
};

const obj = new Proxy({ x: 10, y: 20 }, handler);

const sum = obj.x + obj.y;
console.log(sum);
console.log(callCount);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
30
2
```

### Explanation
Every property access on `obj` triggers the `get` trap. `obj.x` and `obj.y` each trigger it once, incrementing `callCount` to `2`. `Reflect.get(target, prop, receiver)` forwards to the original property access, returning `10` and `20`. This pattern is useful for logging, performance monitoring, or reactive systems like Vue 3.

</details>

---

### Q4. What will be the output?

```js
const range = new Proxy({ min: 1, max: 10 }, {
  has(target, prop) {
    const num = Number(prop);
    return !isNaN(num) && num >= target.min && num <= target.max;
  }
});

console.log(1 in range);
console.log(5 in range);
console.log(10 in range);
console.log(11 in range);
console.log("min" in range);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
true
true
true
false
false
```

### Explanation
The `has` trap overrides the `in` operator. The custom logic checks if the left-hand side, when converted to a number, falls within `[min, max]`. `1`, `5`, and `10` are all within `[1, 10]`, so `true`. `11` exceeds `max`, so `false`. `"min"` converts to `NaN` via `Number("min")`, so `isNaN(NaN)` is `true` and the whole expression returns `false`.

</details>

---

## 6. Logical Assignment Questions

---

### Q1. What will be the output?

```js
let a = null;
let b = undefined;
let c = 0;
let d = "";
let e = false;

a ??= "filled";
b ??= "filled";
c ??= "filled";
d ??= "filled";
e ??= "filled";

console.log(a);
console.log(b);
console.log(c);
console.log(d);
console.log(e);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
filled
filled
0

false
```

### Explanation
`??=` assigns only when the current value is `null` or `undefined`. `null` and `undefined` trigger the assignment. `0`, `""`, and `false` are falsy but are **not** `null` or `undefined`, so they are left unchanged. This is the critical difference between `??=` and `||=`.

</details>

---

### Q2. What will be the output?

```js
let a = 0;
let b = 1;
let c = 0;
let d = 1;

a ||= 99;
b ||= 99;
c &&= 99;
d &&= 99;

console.log(a);
console.log(b);
console.log(c);
console.log(d);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
99
1
0
99
```

### Explanation
`||=` assigns when the left side is **falsy**: `a` (which is `0`) gets `99`; `b` (which is `1`, truthy) keeps `1`. `&&=` assigns when the left side is **truthy**: `c` (which is `0`, falsy) keeps `0` — right side is not even evaluated; `d` (which is `1`, truthy) gets overwritten with `99`.

</details>

---

### Q3. What will be the output?

```js
let sideEffects = 0;

function compute(label) {
  sideEffects++;
  return label;
}

let x = null;
let y = 0;
let z = 1;

x ??= compute("x-computed");
y ||= compute("y-computed");
z &&= compute("z-computed");

console.log(x);
console.log(y);
console.log(z);
console.log(sideEffects);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
x-computed
y-computed
z-computed
3
```

### Explanation
All three assignments trigger their right-hand sides: `x` is `null` (triggers `??=`), `y` is `0` (falsy, triggers `||=`), `z` is `1` (truthy, triggers `&&=`). Each call to `compute()` increments `sideEffects`, so `sideEffects` ends at `3`. This confirms that all three assignments were fully evaluated.

</details>

---

### Q4. What will be the output?

```js
const config = {};

config.timeout ??= 3000;
config.retries ??= 5;
config.retries ??= 10;   // already set — should NOT re-assign

console.log(config.timeout);
console.log(config.retries);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
3000
5
```

### Explanation
The first `??=` on `config.timeout` assigns `3000` because `config.timeout` is `undefined`. The first `??=` on `config.retries` assigns `5` for the same reason. The second `??=` on `config.retries` does **not** assign `10` because `config.retries` is now `5`, which is neither `null` nor `undefined`. This pattern is ideal for safe default initialization.

</details>

---

## 7. for...of vs for...in Questions

---

### Q1. What will be the output?

```js
const arr = [10, 20, 30];
arr.label = "my-array";

const forInKeys = [];
const forOfVals = [];

for (const k in arr) forInKeys.push(k);
for (const v of arr) forOfVals.push(v);

console.log(forInKeys);
console.log(forOfVals);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
[ '0', '1', '2', 'label' ]
[ 10, 20, 30 ]
```

### Explanation
`for...in` iterates all **enumerable property keys** of the object, including non-index keys like `"label"`. Array indices (`"0"`, `"1"`, `"2"`) and `"label"` are all enumerable own properties. `for...of` uses the array's `[Symbol.iterator]` which only yields the indexed **values** — it ignores non-numeric properties entirely.

</details>

---

### Q2. What will be the output?

```js
const str = "hi 👋";

let forOfCount = 0;
for (const char of str) {
  forOfCount++;
}

console.log(str.length);
console.log(forOfCount);
console.log([...str].length);
console.log([...str].at(-1));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
5
4
4
👋
```

### Explanation
The wave emoji 👋 (U+1F44B) is encoded as a **surrogate pair** in UTF-16, occupying 2 code units. `str.length` counts UTF-16 code units, so `h`+`i`+` `+`👋`(×2) = 5. `for...of` and the spread operator iterate **Unicode code points** — they correctly treat the emoji as 1 character, so the count is 4. `[...str].at(-1)` returns the last code point, which is the whole emoji.

</details>

---

### Q3. What will be the output?

```js
function Animal(name) {
  this.name = name;
}
Animal.prototype.type = "animal";

const dog = new Animal("Rex");
dog.breed = "Lab";

const ownKeys = [];
const allKeys = [];

for (const k in dog) allKeys.push(k);
Object.keys(dog).forEach(k => ownKeys.push(k));

console.log(allKeys);
console.log(ownKeys);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
[ 'name', 'breed', 'type' ]
[ 'name', 'breed' ]
```

### Explanation
`for...in` iterates **own and inherited enumerable properties** in the prototype chain. `dog` has own properties `name` and `breed`, and inherits the enumerable `type` from `Animal.prototype`. `Object.keys()` returns **only own enumerable** properties, so `type` is excluded. Use `hasOwnProperty()` or `Object.keys()` when you want to exclude inherited properties.

</details>

---

### Q4. What will be the output?

```js
const map = new Map([["a", 1], ["b", 2], ["c", 3]]);
const set = new Set([10, 20, 30]);

const mapResults = [];
const setResults = [];

for (const [k, v] of map) mapResults.push(`${k}=${v}`);
for (const v of set) setResults.push(v);

// Can we use for...in on Map?
const forInOnMap = [];
for (const k in map) forInOnMap.push(k);

console.log(mapResults);
console.log(setResults);
console.log(forInOnMap);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
[ 'a=1', 'b=2', 'c=3' ]
[ 10, 20, 30 ]
[]
```

### Explanation
`Map` and `Set` implement `[Symbol.iterator]`, so `for...of` works on them. A `Map` yields `[key, value]` pairs which can be destructured. `for...in` iterates **enumerable string-keyed properties** of an object — `Map` and `Set` store their data internally, not as enumerable object properties, so `for...in` on a `Map` yields an empty result.

</details>

---

## 8. Advanced ES6+ Questions

---

### Q1. What will be the output?

```js
const original = { a: 1, b: { c: 2 } };

const shallow = { ...original };
const deep = structuredClone(original);

shallow.b.c = 10;   // mutates shared nested ref
deep.b.c = 20;      // only mutates deep clone

console.log(original.b.c);
console.log(shallow.b.c);
console.log(deep.b.c);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
10
10
20
```

### Explanation
Spread (`...`) creates a **shallow copy** — top-level properties are copied by value, but nested objects are copied by reference. Mutating `shallow.b.c` also mutates `original.b.c` because they share the same `b` object. `structuredClone()` creates a **deep copy** — the nested `b` object is fully cloned and independent. Mutating `deep.b.c` has no effect on `original`.

</details>

---

### Q2. What will be the output?

```js
const obj = {
  user: {
    profile: {
      getName() { return "Alice"; }
    }
  }
};

console.log(obj?.user?.profile?.getName?.());
console.log(obj?.account?.profile?.getName?.());
console.log(obj?.user?.profile?.missing?.());
console.log(obj?.user?.profile?.getName?.call(null));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Alice
undefined
undefined
Alice
```

### Explanation
`?.()` is optional method call syntax — it calls the method only if the preceding expression is not `null`/`undefined`. `obj?.account` is `undefined`, so the entire chain short-circuits to `undefined`. `obj?.user?.profile?.missing` is `undefined`, so `?.()` short-circuits. `?.call(null)` is equivalent to a normal call using `Function.prototype.call`; the method runs with `this = null` but still returns `"Alice"` since it doesn't use `this`.

</details>

---

### Q3. What will be the output?

```js
const config = {
  timeout: 0,
  retries: false,
  name: "",
  level: null,
  debug: undefined
};

const timeout  = config.timeout  ?? 5000;
const retries  = config.retries  ?? 3;
const name     = config.name     ?? "default";
const level    = config.level    ?? "info";
const debug    = config.debug    ?? true;

console.log(timeout);
console.log(retries);
console.log(name);
console.log(level);
console.log(debug);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
0
false

info
true
```

### Explanation
`??` only provides the fallback for `null` and `undefined`. `0`, `false`, and `""` are falsy but are **not** `null` or `undefined`, so they pass through unchanged. `null` and `undefined` trigger the fallback values `"info"` and `true` respectively. This makes `??` far safer than `||` for configuration objects where `0`, `false`, or `""` are valid values.

</details>

---

### Q4. What will be the output?

```js
const prices = { apple: 1.5, banana: 0.75, cherry: 3.0 };

// Double all prices
const doubled = Object.fromEntries(
  Object.entries(prices).map(([k, v]) => [k, v * 2])
);

// Filter entries where value > 2
const expensive = Object.fromEntries(
  Object.entries(prices).filter(([, v]) => v > 2)
);

console.log(doubled);
console.log(expensive);

// Round-trip: Object → Map → filter → Object
const map = new Map(Object.entries(prices));
map.delete("banana");
const filtered = Object.fromEntries(map);
console.log(filtered);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
{ apple: 3, banana: 1.5, cherry: 6 }
{ cherry: 3 }
{ apple: 1.5, cherry: 3 }
```

### Explanation
`Object.entries()` converts an object to `[key, value]` pairs, which can be chained with array methods (`map`, `filter`), then reconstructed with `Object.fromEntries()`. This is a clean functional pipeline for object transformation. `Object.fromEntries()` also accepts a `Map` directly, enabling a Map → transform → Object round-trip pattern.

</details>

---

### Q5. What will be the output?

```js
function* fibonacci() {
  let [a, b] = [0, 1];
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

const gen = fibonacci();
const first6 = [];

for (const n of gen) {
  first6.push(n);
  if (first6.length === 6) break;
}

console.log(first6);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
[ 0, 1, 1, 2, 3, 5 ]
```

### Explanation
The generator defines an **infinite** sequence (no termination condition). It is safe to iterate because `for...of` can break early — when `break` is called, it sends a `return` signal to the generator, which terminates it cleanly. The first 6 Fibonacci numbers are `0, 1, 1, 2, 3, 5`. Generators are the idiomatic way to create lazy infinite sequences in JavaScript.

</details>

---

## Final Tips

- ES module imports are **live bindings** — always reflect the current exported value, unlike CommonJS `require()`.
- `Symbol()` is always unique; `Symbol.for()` uses a global registry for shared symbols.
- `Map` preserves insertion order and accepts any key type; plain objects coerce all keys to strings.
- `WeakMap` / `WeakSet` have no `size` and are not iterable — by design, to allow garbage collection.
- Proxy `get` traps fire even for `undefined` property reads; check `prop in target` to distinguish.
- `??=` is strictly nullish (`null`/`undefined`); `||=` is falsy (`0`, `""`, `false` also trigger it).
- `for...in` includes inherited enumerable properties; prefer `Object.keys()` for own-only iteration.
- `structuredClone()` is the modern deep-clone solution — it handles `Date`, `Map`, `Set`, and circular references, unlike `JSON.parse(JSON.stringify(...))`.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
