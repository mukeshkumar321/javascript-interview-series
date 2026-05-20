# Objects — Tricky Output Questions

## Table of Contents

1. [Property Descriptor Questions](#1-property-descriptor-questions)
2. [freeze / seal / preventExtensions Questions](#2-freeze--seal--preventextensions-questions)
3. [Shallow vs Deep Copy Questions](#3-shallow-vs-deep-copy-questions)
4. [Destructuring Questions](#4-destructuring-questions)
5. [Reference vs Value Questions](#5-reference-vs-value-questions)
6. [for...in Questions](#6-forin-questions)
7. [Getter / Setter Questions](#7-getter--setter-questions)
8. [Advanced Object Questions](#8-advanced-object-questions)

---

## 1. Property Descriptor Questions

---

### Q1. What will be the output?

```js
const obj = {};
Object.defineProperty(obj, "x", {
  value: 10,
  writable: false,
  enumerable: true,
  configurable: false,
});

obj.x = 99;
console.log(obj.x);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
10
```

### Explanation

`writable: false` makes the property read-only. In non-strict mode the assignment `obj.x = 99` silently fails — no error is thrown but the value does not change. In strict mode (`"use strict"`), a `TypeError` would be thrown. The logged value is still `10`.

</details>

---

### Q2. What will be the output?

```js
const obj = {};
Object.defineProperty(obj, "secret", {
  value: 42,
  enumerable: false,
  writable: true,
  configurable: true,
});

console.log(obj.secret);
console.log(Object.keys(obj));
console.log("secret" in obj);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
42
[]
true
```

### Explanation

`enumerable: false` hides the property from `Object.keys`, `Object.values`, `Object.entries`, and `for...in`. However, the property still exists on the object — direct access (`obj.secret`) and the `in` operator both find it. Non-enumerable means "hidden from iteration", not "hidden from existence checks".

</details>

---

### Q3. What will be the output?

```js
const obj = {};
Object.defineProperty(obj, "id", {
  value: 1,
  configurable: false,
});

try {
  Object.defineProperty(obj, "id", { value: 2 });
} catch (e) {
  console.log(e.message);
}

console.log(obj.id);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
Cannot redefine property: id
1
```

### Explanation

`configurable: false` prevents the property descriptor from being changed and the property from being deleted. Attempting to redefine it with `Object.defineProperty` throws a `TypeError`. The value remains `1`.

</details>

---

## 2. freeze / seal / preventExtensions Questions

---

### Q1. What will be the output?

```js
const obj = Object.freeze({ a: 1, b: { c: 2 } });

obj.a = 99;
obj.b.c = 99;

console.log(obj.a);
console.log(obj.b.c);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
1
99
```

### Explanation

`Object.freeze` is **shallow**. The top-level property `a` cannot be modified — the assignment silently fails. However, the nested object `obj.b` is a separate reference that is **not** frozen. Mutations on `obj.b.c` succeed normally. To deep-freeze, you must recursively freeze all nested objects.

</details>

---

### Q2. What will be the output?

```js
const obj = Object.seal({ a: 1, b: 2 });

obj.a = 99;
obj.c = 3;
delete obj.b;

console.log(obj.a);
console.log(obj.c);
console.log(obj.b);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
99
undefined
2
```

### Explanation

`Object.seal` allows modification of existing properties but prevents adding or deleting properties. `obj.a = 99` succeeds. `obj.c = 3` silently fails — no new property is added. `delete obj.b` silently fails — sealed properties are non-configurable, so `obj.b` remains `2`.

</details>

---

### Q3. What will be the output?

```js
const obj = Object.preventExtensions({ x: 1, y: 2 });

obj.x = 99;
obj.z = 3;
delete obj.y;

console.log(obj.x);
console.log(obj.z);
console.log(obj.y);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
99
undefined
undefined
```

### Explanation

`Object.preventExtensions` only prevents **adding** new properties. Existing properties can still be modified and deleted. `obj.x = 99` succeeds. `obj.z = 3` fails silently — no new property. `delete obj.y` succeeds — `y` is removed. Both `obj.z` and `obj.y` are `undefined` but for different reasons: `z` was never added, `y` was deleted.

</details>

---

## 3. Shallow vs Deep Copy Questions

---

### Q1. What will be the output?

```js
const original = { score: 100, data: { wins: 5 } };
const copy = Object.assign({}, original);

copy.score = 0;
copy.data.wins = 0;

console.log(original.score);
console.log(original.data.wins);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
100
0
```

### Explanation

`Object.assign` creates a shallow copy. The primitive `score` is copied by value — mutating `copy.score` does not affect `original.score`. The nested object `data` is copied by reference — both `copy.data` and `original.data` point to the same object in memory, so `copy.data.wins = 0` also changes `original.data.wins`.

</details>

---

### Q2. What will be the output?

```js
const a = { x: [1, 2, 3] };
const b = { ...a };

b.x.push(4);

console.log(a.x);
console.log(b.x);
console.log(a.x === b.x);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
[1, 2, 3, 4]
[1, 2, 3, 4]
true
```

### Explanation

The spread operator `{ ...a }` performs a shallow copy. The property `x` holds an array (which is an object), so only the reference is copied. `a.x` and `b.x` point to the same array. Mutating the array via either reference affects both. `a.x === b.x` confirms they are the same object.

</details>

---

### Q3. What will be the output?

```js
const obj = {
  name: "Alice",
  fn: function () { return "hello"; },
  value: undefined,
  score: NaN,
};

const deep = JSON.parse(JSON.stringify(obj));

console.log(deep.name);
console.log(deep.fn);
console.log(deep.value);
console.log(deep.score);
console.log(Object.keys(deep));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
Alice
undefined
undefined
null
['name', 'score']
```

### Explanation

`JSON.stringify` silently drops properties whose values are functions (`fn`) or `undefined` (`value`) — those keys do not appear in the JSON string. `NaN` is serialized as `null` because JSON has no `NaN` representation. After parsing, `deep.fn` and `deep.value` are `undefined` because those keys were dropped; `deep.score` is `null`.

</details>

---

## 4. Destructuring Questions

---

### Q1. What will be the output?

```js
const { a: x = 10, b: y = 20 } = { a: 5 };

console.log(x);
console.log(y);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
5
20
```

### Explanation

`{ a: x = 10 }` renames `a` to `x` and sets a default of `10`. Since `a` exists with value `5`, the default is not used — `x` is `5`. `{ b: y = 20 }` renames `b` to `y` with default `20`. Since `b` does not exist in the object, the default kicks in — `y` is `20`.

</details>

---

### Q2. What will be the output?

```js
const obj = { p: 1, q: 2, r: 3 };
const { p, ...rest } = obj;

console.log(p);
console.log(rest);
console.log(obj);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
1
{ q: 2, r: 3 }
{ p: 1, q: 2, r: 3 }
```

### Explanation

Rest in destructuring (`...rest`) collects all remaining own enumerable properties that were not explicitly destructured. `p` is extracted, so `rest` gets `{ q: 2, r: 3 }`. The original `obj` is not mutated — destructuring never modifies the source.

</details>

---

### Q3. What will be the output?

```js
const data = {
  user: {
    name: "Alice",
    address: { city: "NY" },
  },
};

const { user: { name, address: { city }, phone = "N/A" } } = data;

console.log(name);
console.log(city);
console.log(phone);

try {
  console.log(user);
} catch (e) {
  console.log(e.message);
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
Alice
NY
N/A
user is not defined
```

### Explanation

In nested destructuring, intermediate keys like `user` and `address` are not declared as variables — they are only used as paths to navigate the structure. Only `name`, `city`, and `phone` are declared. `phone` uses its default `"N/A"` because the property does not exist. Accessing `user` directly throws a `ReferenceError`.

</details>

---

## 5. Reference vs Value Questions

---

### Q1. What will be the output?

```js
let a = { val: 1 };
let b = a;

b.val = 99;
b = { val: 0 };

console.log(a.val);
console.log(b.val);
console.log(a === b);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
99
0
false
```

### Explanation

Initially `b = a` makes both variables point to the same object. `b.val = 99` mutates that shared object, so `a.val` is also `99`. Then `b = { val: 0 }` **reassigns** `b` to a completely new object — `a` still points to the original. After reassignment, `a` and `b` are different objects, so `a === b` is `false`.

</details>

---

### Q2. What will be the output?

```js
function mutate(obj) {
  obj.x = 100;
  obj = { x: 999 };
}

const o = { x: 1 };
mutate(o);
console.log(o.x);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
100
```

### Explanation

JavaScript passes object references by value. Inside `mutate`, `obj` starts as a copy of the reference pointing to the same object as `o`. `obj.x = 100` mutates the shared object — this affects `o`. Then `obj = { x: 999 }` reassigns the local parameter `obj` to a new object — this only changes the local binding and has no effect on `o`. So `o.x` is `100`.

</details>

---

## 6. for...in Questions

---

### Q1. What will be the output?

```js
const parent = { inherited: true };
const child = Object.create(parent);
child.own = true;

for (const key in child) {
  console.log(key);
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
own
inherited
```

### Explanation

`for...in` iterates over all enumerable string-keyed properties — both own and inherited. `own` is an own property of `child`. `inherited` comes from `parent`, which sits in `child`'s prototype chain. Both are enumerable, so both are logged. Own properties are listed before inherited ones.

</details>

---

### Q2. What will be the output?

```js
const obj = { a: 1, b: 2, c: 3 };

Object.defineProperty(obj, "hidden", {
  value: 99,
  enumerable: false,
});

for (const key in obj) {
  console.log(key);
}

console.log(obj.hidden);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
a
b
c
99
```

### Explanation

`for...in` only iterates enumerable properties. `hidden` was defined with `enumerable: false`, so it is skipped by `for...in`. However, the property still exists on the object and is accessible via direct access (`obj.hidden`).

</details>

---

### Q3. What will be the output?

```js
const obj = { a: 1, b: 2 };
const results = [];

for (const key in obj) {
  results.push(key);
}

results.push("c");
console.log(Object.keys(obj));
console.log(results);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
['a', 'b']
['a', 'b', 'c']
```

### Explanation

`for...in` builds `results` with `['a', 'b']`. Pushing `'c'` into `results` only modifies the `results` array — it does not affect the original `obj`. `Object.keys(obj)` still returns `['a', 'b']`.

</details>

---

## 7. Getter / Setter Questions

---

### Q1. What will be the output?

```js
const obj = {
  _count: 0,
  get count() {
    return this._count;
  },
  set count(value) {
    if (value >= 0) this._count = value;
  },
};

obj.count = 5;
obj.count = -1;
console.log(obj.count);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
5
```

### Explanation

The setter validates that the value is non-negative before updating `_count`. `obj.count = 5` passes validation — `_count` becomes `5`. `obj.count = -1` fails the `>= 0` check — the setter does nothing, so `_count` stays at `5`. Reading `obj.count` calls the getter which returns `_count`.

</details>

---

### Q2. What will be the output?

```js
const obj = {
  get x() {
    return 42;
  },
};

obj.x = 100;
console.log(obj.x);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
42
```

### Explanation

There is a getter for `x` but no setter. In non-strict mode, the assignment `obj.x = 100` silently fails — no error is thrown and the getter is unchanged. Reading `obj.x` still invokes the getter which returns `42`. In strict mode, assigning to a getter-only property throws a `TypeError`.

</details>

---

### Q3. What will be the output?

```js
const person = {
  _name: "Alice",
  get name() {
    console.log("getter called");
    return this._name;
  },
  set name(val) {
    console.log("setter called");
    this._name = val.trim();
  },
};

const n = person.name;
person.name = "  Bob  ";
console.log(person._name);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
getter called
setter called
Bob
```

### Explanation

Accessing `person.name` triggers the getter, which logs "getter called" and returns `"Alice"`. Assigning `person.name = "  Bob  "` triggers the setter, which logs "setter called" and stores the trimmed value `"Bob"` in `_name`. The getter and setter are transparent: they look like a plain property from the outside but execute logic invisibly.

</details>

---

## 8. Advanced Object Questions

---

### Q1. What will be the output?

```js
const sym1 = Symbol("key");
const sym2 = Symbol("key");
const obj = {
  [sym1]: "symbol1 value",
  name: "visible",
};

console.log(sym1 === sym2);
console.log(obj[sym1]);
console.log(obj[sym2]);
console.log(Object.keys(obj));
console.log(Object.getOwnPropertySymbols(obj).length);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
false
symbol1 value
undefined
['name']
1
```

### Explanation

Every call to `Symbol()` creates a **unique** symbol, even if the description string is the same. `sym1 === sym2` is `false`. `obj[sym1]` accesses the correct property. `obj[sym2]` is `undefined` because `sym2` is a different symbol with no property. `Object.keys` excludes symbol-keyed properties. `Object.getOwnPropertySymbols` returns only symbol-keyed properties — there is one (`sym1`).

</details>

---

### Q2. What will be the output?

```js
const obj = { a: 1, b: 2 };
const copy = Object.assign({}, obj);
const spread = { ...obj };

obj.a = 99;

console.log(copy.a);
console.log(spread.a);
console.log(obj.a);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
1
1
99
```

### Explanation

Both `Object.assign({}, obj)` and `{ ...obj }` create shallow copies at the time of the call. The property `a` has a primitive value (`1`), so it is copied by value. Later mutating `obj.a = 99` affects only `obj` — `copy.a` and `spread.a` remain `1`. Both techniques behave identically for top-level primitive properties.

</details>

---

### Q3. What will be the output?

```js
const obj = {};

Object.defineProperty(obj, "readOnly", {
  get() { return "constant"; },
  configurable: false,
});

console.log(obj.readOnly);
obj.readOnly = "changed";
console.log(obj.readOnly);

try {
  Object.defineProperty(obj, "readOnly", { value: "new" });
} catch (e) {
  console.log(e.message);
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
constant
constant
Cannot redefine property: readOnly
```

### Explanation

A property defined with only a `get` accessor (no `set`) is read-only — assigning to it silently fails in non-strict mode. `configurable: false` prevents the descriptor from being redefined, so the `Object.defineProperty` call throws a `TypeError`. The property remains unchanged throughout.

</details>

---

### Q4. What will be the output?

```js
const target = { a: 1, b: 2 };
const source1 = { b: 10, c: 3 };
const source2 = { c: 30, d: 4 };

const result = Object.assign(target, source1, source2);

console.log(result === target);
console.log(result.b);
console.log(result.c);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
true
10
30
```

### Explanation

`Object.assign` merges sources left to right into `target`, **mutating** it and returning the same reference. When the same key exists in multiple sources, the last source wins. `b` exists in `target` (`1`) and `source1` (`10`) — `source1` overwrites giving `10`. `c` exists in `source1` (`3`) and `source2` (`30`) — `source2` overwrites giving `30`. `result === target` is `true` because `Object.assign` returns the mutated target, not a new object.

</details>

---

## Final Tips

- **Property descriptors** control whether a property can be read, written, deleted, or iterated — understanding them is essential for library code and defensive programming.
- **`Object.freeze` is shallow** — a frozen object can still have its nested objects mutated. Deep-freeze requires a recursive approach.
- **Shallow copy pitfall** — `{ ...obj }` and `Object.assign` copy only top-level properties. Nested objects are shared by reference.
- **`JSON.stringify` traps** — silently drops `undefined` and functions; converts `NaN`/`Infinity` to `null`; throws on circular references.
- **`for...in` and inherited props** — always use `hasOwnProperty` guard if you only want own properties.
- **Getters without setters** silently fail on assignment in non-strict mode — this is a common source of confusing bugs.
- **Symbols** are unique by definition — two `Symbol("same")` calls produce different symbols that will never collide as property keys.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
