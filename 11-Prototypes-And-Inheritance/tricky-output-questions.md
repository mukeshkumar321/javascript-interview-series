# Prototypes and Inheritance — Tricky Output Questions

## Table of Contents

1. [Prototype Chain Questions](#1-prototype-chain-questions)
2. [instanceof Questions](#2-instanceof-questions)
3. [new Keyword Questions](#3-new-keyword-questions)
4. [hasOwnProperty Questions](#4-hasownproperty-questions)
5. [Object.create Questions](#5-objectcreate-questions)
6. [Prototype Mutation Questions](#6-prototype-mutation-questions)
7. [Advanced Prototype Questions](#7-advanced-prototype-questions)

---

## 1. Prototype Chain Questions

---

### Q1. What will be the output?

```js
function Foo() {}
Foo.prototype.x = 10;

const obj = new Foo();
console.log(obj.x);

obj.x = 20;
console.log(obj.x);

delete obj.x;
console.log(obj.x);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
10
20
10
```

### Explanation

Initially `obj` has no own property `x`, so lookup delegates to `Foo.prototype` — returns `10`. `obj.x = 20` creates an **own** property on `obj` that shadows the prototype property — now `obj.x` returns `20`. `delete obj.x` removes the own property only. The prototype property is untouched, so `obj.x` falls back to `Foo.prototype.x` which is `10` again.

</details>

---

### Q2. What will be the output?

```js
const a = {};
const b = Object.create(a);
const c = Object.create(b);

a.greet = "hello";

console.log(c.greet);
console.log(c.hasOwnProperty("greet"));
console.log(b.hasOwnProperty("greet"));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
hello
false
false
```

### Explanation

The prototype chain is `c → b → a → Object.prototype → null`. `c.greet` is not found on `c` or `b`, but is found on `a` — returns `"hello"`. Neither `c` nor `b` have `greet` as an own property — both `hasOwnProperty` calls return `false`. Properties added to `a` after `b` and `c` were created are still accessible via delegation.

</details>

---

### Q3. What will be the output?

```js
const obj = { a: 1 };
console.log(obj.toString());
console.log(obj.hasOwnProperty("toString"));
console.log(Object.getPrototypeOf(obj) === Object.prototype);
console.log(Object.getPrototypeOf(Object.prototype));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
[object Object]
false
true
null
```

### Explanation

`obj.toString()` is found on `Object.prototype` — the top of the chain for plain objects. `hasOwnProperty("toString")` is `false` because `toString` is inherited, not own. `Object.getPrototypeOf(obj)` is `Object.prototype` for any plain object literal. `Object.getPrototypeOf(Object.prototype)` is `null` — the absolute end of every prototype chain.

</details>

---

## 2. instanceof Questions

---

### Q1. What will be the output?

```js
function A() {}
function B() {}

B.prototype = Object.create(A.prototype);
B.prototype.constructor = B;

const obj = new B();

console.log(obj instanceof B);
console.log(obj instanceof A);
console.log(obj instanceof Object);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
true
true
true
```

### Explanation

`instanceof` walks up the prototype chain looking for the constructor's `.prototype` object. `obj`'s chain is: `obj → B.prototype → A.prototype → Object.prototype → null`. `B.prototype` is in the chain — `instanceof B` is `true`. `A.prototype` is also in the chain — `instanceof A` is `true`. `Object.prototype` is always in the chain — `instanceof Object` is `true`.

</details>

---

### Q2. What will be the output?

```js
function Foo() {}
const obj = new Foo();

Foo.prototype = {};

console.log(obj instanceof Foo);
console.log(obj.constructor === Foo);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
false
true
```

### Explanation

`instanceof` checks if the **current** `Foo.prototype` is in the object's chain. After `Foo.prototype = {}`, the prototype was replaced with a new empty object. `obj` was created with the **old** prototype, so `instanceof Foo` returns `false` — the new `Foo.prototype` is not in `obj`'s chain. However, `obj.constructor` still delegates to the old prototype (which it holds an `__proto__` reference to), and the old prototype had `constructor === Foo` — so `obj.constructor === Foo` is `true`.

</details>

---

### Q3. What will be the output?

```js
const arr = [1, 2, 3];
const obj = {};
const fn = function () {};

console.log(arr instanceof Array);
console.log(arr instanceof Object);
console.log(obj instanceof Function);
console.log(fn instanceof Function);
console.log(fn instanceof Object);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
true
true
false
true
true
```

### Explanation

`arr`'s chain is `arr → Array.prototype → Object.prototype → null`, so `instanceof Array` and `instanceof Object` are both `true`. `obj` is a plain object — `Function.prototype` is not in its chain, so `instanceof Function` is `false`. Functions have the chain `fn → Function.prototype → Object.prototype → null`, making `instanceof Function` and `instanceof Object` both `true`.

</details>

---

## 3. new Keyword Questions

---

### Q1. What will be the output?

```js
function Person(name) {
  this.name = name;
  return { name: "Override" };
}

const p = new Person("Alice");
console.log(p.name);
console.log(p instanceof Person);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
Override
false
```

### Explanation

Step 4 of `new`: if the constructor explicitly returns an **object**, `new` returns that object instead of the newly created `this`. `Person` returns `{ name: "Override" }` — a plain object with no link to `Person.prototype`. So `p.name` is `"Override"` and `p instanceof Person` is `false` because `Person.prototype` is not in the chain of the returned object.

</details>

---

### Q2. What will be the output?

```js
function Counter() {
  this.count = 0;
}

Counter.prototype.increment = function () {
  this.count++;
};

const a = new Counter();
const b = new Counter();

a.increment();
a.increment();
b.increment();

console.log(a.count);
console.log(b.count);
console.log(a.increment === b.increment);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
2
1
true
```

### Explanation

Each `new Counter()` call creates a separate instance with its own `count` property initialized to `0`. `increment` is defined on `Counter.prototype` and is **shared** between instances — `a.increment === b.increment` is `true`. However, when `increment` is called, `this` refers to the specific instance. So `a.count` and `b.count` are tracked independently.

</details>

---

### Q3. What will be the output?

```js
function Foo() {
  this.value = 42;
  return 99; // returns a primitive, not an object
}

const obj = new Foo();
console.log(obj.value);
console.log(obj instanceof Foo);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
42
true
```

### Explanation

`new` only overrides the return value if the constructor returns an **object**. A primitive return value (like `99`) is ignored — `new` falls back to returning `this`. So `obj` is the newly created instance with `value: 42`, and `obj instanceof Foo` is `true` because its `[[Prototype]]` is `Foo.prototype`.

</details>

---

## 4. hasOwnProperty Questions

---

### Q1. What will be the output?

```js
const parent = { a: 1 };
const child = Object.create(parent);
child.b = 2;

console.log(child.hasOwnProperty("a"));
console.log(child.hasOwnProperty("b"));
console.log("a" in child);
console.log("b" in child);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
false
true
true
true
```

### Explanation

`child.b = 2` creates an own property on `child`. `a` lives on `parent` (the prototype), so `hasOwnProperty("a")` is `false`. The `in` operator searches the entire prototype chain — both `"a"` and `"b"` are found somewhere in the chain, so both return `true`. `hasOwnProperty` distinguishes own from inherited; `in` does not.

</details>

---

### Q2. What will be the output?

```js
const obj = {
  hasOwnProperty: function () {
    return "I am overridden!";
  },
};

console.log(obj.hasOwnProperty("anything"));
console.log(Object.prototype.hasOwnProperty.call(obj, "hasOwnProperty"));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
I am overridden!
true
```

### Explanation

`obj` defines its own `hasOwnProperty` property that shadows `Object.prototype.hasOwnProperty`. Calling `obj.hasOwnProperty("anything")` invokes the overridden version which always returns the string. To reliably use the original method, borrow it via `Object.prototype.hasOwnProperty.call(obj, key)`. The second call correctly checks whether `obj` has its own property named `"hasOwnProperty"` — it does, so the result is `true`.

</details>

---

## 5. Object.create Questions

---

### Q1. What will be the output?

```js
const proto = {
  greet() {
    return `Hello, I am ${this.name}`;
  },
};

const obj = Object.create(proto);
obj.name = "Alice";

console.log(obj.greet());
console.log(obj.hasOwnProperty("greet"));
console.log(obj.hasOwnProperty("name"));
console.log(Object.getPrototypeOf(obj) === proto);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
Hello, I am Alice
false
true
true
```

### Explanation

`Object.create(proto)` creates a new object whose `[[Prototype]]` is `proto`. `greet` is on `proto`, not on `obj` itself — `hasOwnProperty("greet")` is `false`. `name` was added directly to `obj` — `hasOwnProperty("name")` is `true`. When `greet()` is called, `this` is `obj`, so `this.name` is `"Alice"`.

</details>

---

### Q2. What will be the output?

```js
const obj = Object.create(null);
obj.name = "test";

console.log(obj.name);
console.log(obj.toString);
console.log(obj instanceof Object);

try {
  obj.hasOwnProperty("name");
} catch (e) {
  console.log(e.message);
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
test
undefined
false
obj.hasOwnProperty is not a function
```

### Explanation

`Object.create(null)` creates an object with **no prototype** — the chain is just `obj → null`. Own properties like `name` are still accessible. `obj.toString` is `undefined` because there is no `Object.prototype` in the chain. `instanceof Object` is `false` for the same reason. Calling `obj.hasOwnProperty` throws a `TypeError` because that method doesn't exist — use `Object.prototype.hasOwnProperty.call(obj, "name")` instead.

</details>

---

## 6. Prototype Mutation Questions

---

### Q1. What will be the output?

```js
function Dog() {}
const d1 = new Dog();

Dog.prototype.bark = function () {
  return "woof";
};

const d2 = new Dog();

console.log(d1.bark());
console.log(d2.bark());
console.log(d1.bark === d2.bark);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
woof
woof
true
```

### Explanation

`d1` was created before `bark` was added to `Dog.prototype`, but that doesn't matter. `d1.__proto__` is a **live reference** to the same `Dog.prototype` object. When `bark` is added to that object, it becomes accessible from `d1` immediately. Both instances delegate to the same prototype, so `d1.bark === d2.bark` is `true`.

</details>

---

### Q2. What will be the output?

```js
function Foo() {}
Foo.prototype.x = 1;

const a = new Foo();

Foo.prototype.x = 2;
console.log(a.x);

Foo.prototype = { x: 3 };
console.log(a.x);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
2
2
```

### Explanation

After `Foo.prototype.x = 2`, `a.x` delegates to the same `Foo.prototype` object which now has `x: 2`. After `Foo.prototype = { x: 3 }`, the `Foo.prototype` variable points to a **new** object. But `a.__proto__` still holds a reference to the **old** prototype object where `x` was set to `2`. The reassignment of `Foo.prototype` does not change the prototype chain of already-created instances.

</details>

---

### Q3. What will be the output?

```js
const obj = { a: 1 };
Object.setPrototypeOf(obj, { b: 2, c: 3 });

console.log(obj.a);
console.log(obj.b);
console.log(obj.hasOwnProperty("b"));
console.log(Object.keys(obj));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
1
2
false
['a']
```

### Explanation

`Object.setPrototypeOf` replaces `obj`'s prototype with a new object `{ b: 2, c: 3 }`. Own property `a` is unaffected. `obj.b` is found via the new prototype. `hasOwnProperty("b")` is `false` because `b` is inherited. `Object.keys` returns only own enumerable properties — just `['a']`.

</details>

---

## 7. Advanced Prototype Questions

---

### Q1. What will be the output?

```js
function Animal(name) {
  this.name = name;
}
Animal.prototype.speak = function () {
  return `${this.name} speaks.`;
};

function Dog(name) {
  Animal.call(this, name);
}
Dog.prototype = Object.create(Animal.prototype);
// Note: Dog.prototype.constructor is NOT restored

const d = new Dog("Rex");

console.log(d.speak());
console.log(d.constructor === Dog);
console.log(d.constructor === Animal);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
Rex speaks.
false
true
```

### Explanation

`Dog.prototype = Object.create(Animal.prototype)` correctly wires the chain but replaces the old `Dog.prototype` object — and with it, the `constructor` property that pointed to `Dog`. The new `Dog.prototype` inherits `constructor` from `Animal.prototype`, which points to `Animal`. So `d.constructor === Dog` is `false` and `d.constructor === Animal` is `true`. Always restore: `Dog.prototype.constructor = Dog` after setting up the chain.

</details>

---

### Q2. What will be the output?

```js
function Shape(color) {
  this.color = color;
}
Shape.prototype.describe = function () {
  return `color: ${this.color}`;
};

function Box(color, size) {
  Shape.call(this, color);
  this.size = size;
}
Box.prototype = Object.create(Shape.prototype);
Box.prototype.constructor = Box;

const b = new Box("blue", 10);

console.log(b.color);
console.log(b.size);
console.log(b.describe());
console.log(b instanceof Box);
console.log(b instanceof Shape);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
blue
10
color: blue
true
true
```

### Explanation

`Shape.call(this, color)` inside the `Box` constructor runs `Shape`'s initialization with `Box`'s `this`, setting `this.color = color`. `this.size = size` adds the own `size` property. The prototype chain `b → Box.prototype → Shape.prototype → Object.prototype → null` makes `describe` accessible and both `instanceof` checks pass.

</details>

---

### Q3. What will be the output?

```js
function Foo() {}
console.log(typeof Foo.prototype);
console.log(Foo.prototype.constructor === Foo);

const obj = new Foo();
console.log(obj.constructor === Foo);
console.log(obj.hasOwnProperty("constructor"));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
object
true
true
false
```

### Explanation

Every function's `prototype` is an object (`typeof Foo.prototype` is `"object"`). By default, `Foo.prototype.constructor` is a back-reference to `Foo` itself. `obj.constructor === Foo` is `true` because `obj` delegates the lookup to `Foo.prototype`, which has `constructor: Foo`. `obj.hasOwnProperty("constructor")` is `false` — `constructor` is inherited from `Foo.prototype`, not an own property on `obj`.

</details>

---

### Q4. What will be the output?

```js
Object.prototype.extra = "polluted";

const obj = { a: 1 };
const arr = [1, 2];

console.log(obj.extra);
console.log(arr.extra);

for (const key in obj) {
  console.log(key);
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
polluted
polluted
a
extra
```

### Explanation

Adding a property to `Object.prototype` makes it accessible on **every** plain object and array (since `arr.__proto__.__proto__ === Object.prototype`). Both `obj.extra` and `arr.extra` return `"polluted"`. The `for...in` loop over `obj` enumerates own properties (`a`) and then inherited enumerable properties, so `extra` from `Object.prototype` also appears. This is why mutating `Object.prototype` in any real application is dangerous and must be avoided.

</details>

---

## Final Tips

- **Prototype chain lookup is live** — properties added to a prototype after instances are created are immediately accessible via those instances.
- **Replacing `prototype`** (not mutating) breaks the `instanceof` check and loses the `constructor` back-reference for existing instances.
- **Always restore `constructor`** after `Dog.prototype = Object.create(Parent.prototype)` — otherwise `instance.constructor` points to the parent.
- **`instanceof` vs `constructor`** — `instanceof` is more reliable than checking `.constructor` because `.constructor` can be reassigned.
- **`new` with object return** — if a constructor explicitly returns an object, `new` returns that object, not `this`. The result will fail `instanceof` checks.
- **`Object.create(null)`** produces objects with no prototype — avoid calling `hasOwnProperty` directly on them; borrow from `Object.prototype` instead.
- **Never mutate `Object.prototype`** — it pollutes every object in the runtime and is a critical security and correctness issue.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
