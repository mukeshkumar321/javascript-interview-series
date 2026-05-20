# JavaScript Fundamentals

## Table of Contents

1. [JavaScript Overview](#1-javascript-overview)
2. [Primitive Data Types](#2-primitive-data-types)
3. [Reference Data Types](#3-reference-data-types)
4. [typeof Operator](#4-typeof-operator)
5. [null vs undefined](#5-null-vs-undefined)
6. [== vs === Equality](#6--vs--equality)
7. [Truthy and Falsy Values](#7-truthy-and-falsy-values)
8. [Short-Circuit Evaluation](#8-short-circuit-evaluation)
9. [Nullish Coalescing Operator](#9-nullish-coalescing-operator)
10. [Optional Chaining](#10-optional-chaining)
11. [var vs let vs const](#11-var-vs-let-vs-const)
12. [String Methods](#12-string-methods)
13. [Number Methods and Quirks](#13-number-methods-and-quirks)
14. [Math Object](#14-math-object)
15. [Template Literals](#15-template-literals)
16. [Spread Operator](#16-spread-operator)
17. [Rest Parameters](#17-rest-parameters)
18. [Destructuring](#18-destructuring)
19. [Object Shorthand and Computed Properties](#19-object-shorthand-and-computed-properties)
20. [Comma Operator](#20-comma-operator)
21. [typeof vs instanceof](#21-typeof-vs-instanceof)
22. [Garbage Collection Basics](#22-garbage-collection-basics)
23. [Summary](#23-summary)

---

## 1. JavaScript Overview

JavaScript is:

- **Interpreted** (or JIT-compiled): code runs at runtime, not compiled ahead of time into machine code.
- **Single-threaded**: one call stack, one task at a time — concurrency is handled by the event loop.
- **Dynamically typed**: variable types are determined at runtime, not at declaration time.
- **Weakly typed**: type coercion happens implicitly in many operations.

```js
// Dynamic typing — same variable can hold different types
let x = 42;
x = 'hello';
x = true;

// Weak typing — implicit coercion
console.log(1 + '2');   // '12'  — number coerced to string
console.log('5' - 3);   // 2     — string coerced to number
console.log(true + 1);  // 2     — boolean coerced to number
```

### Output

```js
'12'
2
2
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 2. Primitive Data Types

JavaScript has **7 primitive types**. Primitives are **immutable** and stored **by value**.

| Type | Example | typeof result |
|---|---|---|
| `string` | `'hello'` | `'string'` |
| `number` | `42`, `NaN`, `Infinity` | `'number'` |
| `bigint` | `9007199254740991n` | `'bigint'` |
| `boolean` | `true`, `false` | `'boolean'` |
| `null` | `null` | `'object'` (historical bug) |
| `undefined` | `undefined` | `'undefined'` |
| `symbol` | `Symbol('id')` | `'symbol'` |

```js
// Primitives are copied by value
let a = 10;
let b = a;
b = 20;
console.log(a); // 10 — a is unaffected

// Strings are immutable — character assignment is silently ignored
let str = 'hello';
str[0] = 'H';
console.log(str); // 'hello' — no change

// Symbol — each call creates a unique value
console.log(Symbol('x') === Symbol('x')); // false
```

### Output

```js
10
'hello'
false
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 3. Reference Data Types

Objects, arrays, and functions are **reference types** — stored in the heap and accessed by reference.

```js
// Objects are copied by reference
let obj1 = { name: 'Alice' };
let obj2 = obj1;   // same reference, NOT a copy
obj2.name = 'Bob';
console.log(obj1.name); // 'Bob' — obj1 was mutated via obj2

// Arrays are also reference types
let arr1 = [1, 2, 3];
let arr2 = arr1;
arr2.push(4);
console.log(arr1); // [1, 2, 3, 4]

// Shallow copy with spread — creates a new top-level reference
let obj3 = { ...obj1 };
obj3.name = 'Charlie';
console.log(obj1.name); // 'Bob' — obj1 unaffected
```

### Output

```js
'Bob'
[1, 2, 3, 4]
'Bob'
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 4. typeof Operator

`typeof` returns a **string** describing the operand's type. These results are critical for interviews — memorize every case.

```js
typeof 'hello'         // 'string'
typeof 42              // 'number'
typeof NaN             // 'number'   ← NaN is a number type!
typeof Infinity        // 'number'
typeof true            // 'boolean'
typeof undefined       // 'undefined'
typeof null            // 'object'   ← historical bug, not fixable
typeof Symbol()        // 'symbol'
typeof 42n             // 'bigint'
typeof {}              // 'object'
typeof []              // 'object'   ← arrays are objects
typeof function() {}   // 'function'
typeof class Foo {}    // 'function' ← classes are syntactic sugar over functions
typeof (() => {})      // 'function'
```

```js
// typeof on an undeclared variable does NOT throw — returns 'undefined'
console.log(typeof undeclaredVar); // 'undefined'

// typeof typeof always returns 'string' (typeof returns a string)
console.log(typeof typeof 42);     // 'string'

// Safe null check
let val = null;
console.log(typeof val === 'object' && val === null); // true
```

### Output

```js
'undefined'
'string'
true
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 5. null vs undefined

| | `null` | `undefined` |
|---|---|---|
| Meaning | Intentional absence of a value | Declared but not yet assigned |
| `typeof` | `'object'` | `'undefined'` |
| In arithmetic | coerces to `0` | coerces to `NaN` |
| `JSON.stringify` | preserved as `null` | property is omitted |
| Default param trigger | does NOT trigger | DOES trigger |

```js
// Arithmetic coercion
console.log(null + 1);      // 1   (null → 0)
console.log(undefined + 1); // NaN (undefined → NaN)

// Default parameter behavior
function greet(name = 'Guest') {
  return name;
}
console.log(greet(undefined)); // 'Guest' — triggers default
console.log(greet(null));      // null    — does NOT trigger default

// JSON serialization
console.log(JSON.stringify({ a: null, b: undefined })); // '{"a":null}'
// undefined property is completely omitted from the output

// Equality
console.log(null == undefined);  // true  — they are loosely equal
console.log(null === undefined); // false — different types
```

### Output

```js
1
NaN
'Guest'
null
'{"a":null}'
true
false
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 6. == vs === Equality

`===` (strict equality) compares **value AND type** — no coercion.
`==` (loose equality) applies the **Abstract Equality Comparison** algorithm, which coerces types.

### Key == rules to memorize

```js
null == undefined   // true  — they are only equal to each other
null == 0           // false — null never coerces in == comparisons
null == false       // false
undefined == false  // false

NaN == NaN          // false — NaN is never equal to anything, including itself

'' == 0             // true  — '' → 0
'0' == 0            // true  — '0' → 0
'' == '0'           // false — both strings, compared as-is (not equal)

[] == false         // true  — [] → '' → 0, false → 0
[] == 0             // true  — [] → '' → 0
[] == ''            // true  — [] → ''
[1] == 1            // true  — [1] → '1' → 1
```

```js
console.log(null == undefined);  // true
console.log(null === undefined); // false
console.log(NaN == NaN);         // false
console.log([] == false);        // true
console.log('' == '0');          // false
```

### Output

```js
true
false
false
true
false
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 7. Truthy and Falsy Values

**Falsy values** — there are exactly 8:

```js
false
0
-0
0n          // BigInt zero
''          // empty string (also "" and ``)
null
undefined
NaN
```

**Everything else is truthy**, including values that commonly surprise developers:

```js
'0'         // non-empty string — TRUTHY
'false'     // non-empty string — TRUTHY
[]          // empty array — TRUTHY (it's an object reference)
{}          // empty object — TRUTHY (it's an object reference)
-1          // non-zero number — TRUTHY
Infinity    // TRUTHY
```

```js
// Common interview traps
if ('0') console.log("'0' is truthy");   // prints
if ([])  console.log("[] is truthy");    // prints
if ({})  console.log("{} is truthy");    // prints

console.log(Boolean(''));    // false
console.log(Boolean('0'));   // true  — '0' is NOT the number 0
console.log(Boolean([]));    // true
console.log(Boolean(0n));    // false — BigInt zero is falsy
```

### Output

```js
'0' is truthy
[] is truthy
{} is truthy
false
true
true
false
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 8. Short-Circuit Evaluation

`&&` returns the **first falsy** operand, or the **last operand** if all are truthy.
`||` returns the **first truthy** operand, or the **last operand** if all are falsy.

The key insight: `&&` and `||` return **actual values**, not just `true`/`false`.

```js
// && examples
console.log(1 && 2 && 3);       // 3     — all truthy, returns last
console.log(1 && 0 && 3);       // 0     — 0 is first falsy
console.log(false && 'hello');   // false

// || examples
console.log(0 || '' || 'found'); // 'found' — first truthy
console.log(0 || false || null); // null    — all falsy, returns last
console.log(1 || 'never');       // 1       — stops at first truthy

// Guard pattern with &&
const user = { name: 'Alice' };
console.log(user && user.name);  // 'Alice'
console.log(null && null.name);  // null — short-circuits, no TypeError

// Default value with ||
const name = null;
console.log(name || 'Anonymous'); // 'Anonymous'
```

### Output

```js
3
0
false
'found'
null
1
'Alice'
null
'Anonymous'
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 9. Nullish Coalescing Operator

`??` returns the right-hand side **only when the left is `null` or `undefined`**.
Unlike `||`, it does NOT trigger on `0`, `''`, or `false`.

```js
// ?? vs || — the critical difference
console.log(0 || 'default');    // 'default' — 0 is falsy
console.log(0 ?? 'default');    // 0         — 0 is not null/undefined

console.log('' || 'default');   // 'default' — '' is falsy
console.log('' ?? 'default');   // ''        — '' is not null/undefined

console.log(false || 'default');  // 'default'
console.log(false ?? 'default');  // false

// ?? only triggers for null/undefined
console.log(null ?? 'default');      // 'default'
console.log(undefined ?? 'default'); // 'default'

// Chaining
const config = null;
const port = config?.port ?? 3000;
console.log(port); // 3000

// Cannot mix ?? with && or || directly (requires parentheses)
// null || undefined ?? 'val'  → SyntaxError
// (null || undefined) ?? 'val' → 'val'  (valid with parens)
```

### Output

```js
'default'
0
'default'
''
'default'
false
'default'
'default'
3000
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 10. Optional Chaining

`?.` short-circuits and returns `undefined` when the left side is `null` or `undefined`, instead of throwing a `TypeError`.

```js
const user = {
  profile: {
    address: {
      city: 'Mumbai'
    }
  }
};

// Without optional chaining — throws if profile is null
console.log(user?.profile?.address?.city);   // 'Mumbai'
console.log(user?.social?.twitter);          // undefined — no error

// With method calls
const obj = { greet: () => 'hello' };
console.log(obj.greet?.());     // 'hello'
console.log(obj.missing?.());   // undefined — no TypeError

// With array indexes
const arr = [1, 2, 3];
console.log(arr?.[0]);          // 1
console.log(null?.[0]);         // undefined

// With function parameters (optional callback pattern)
function process(callback) {
  return callback?.();
}
console.log(process());          // undefined
console.log(process(() => 42));  // 42
```

### Output

```js
'Mumbai'
undefined
'hello'
undefined
1
undefined
undefined
42
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 11. var vs let vs const

This covers declaration rules only. For scope depth, hoisting, and the Temporal Dead Zone, see `02-Scope-Hoisting`.

| | `var` | `let` | `const` |
|---|---|---|---|
| Scope | function | block | block |
| Re-declaration | allowed | SyntaxError | SyntaxError |
| Re-assignment | allowed | allowed | TypeError |
| Hoisting | hoisted, initialized as `undefined` | hoisted but in TDZ | hoisted but in TDZ |
| Global object property | yes (browser `window`) | no | no |

```js
// var allows re-declaration
var x = 1;
var x = 2;  // OK, no error

// let does not allow re-declaration
let y = 1;
// let y = 2;  // SyntaxError: Identifier 'y' has already been declared

// const must be initialized at declaration
// const z;    // SyntaxError: Missing initializer in const declaration

// const prevents re-assignment, but NOT mutation
const arr = [1, 2, 3];
arr.push(4);    // OK — mutating the contents, not reassigning
// arr = [];    // TypeError: Assignment to constant variable

const obj = { a: 1 };
obj.a = 2;      // OK — mutating a property
// obj = {};    // TypeError

console.log(arr); // [1, 2, 3, 4]
console.log(obj); // { a: 2 }
```

### Output

```js
[1, 2, 3, 4]
{ a: 2 }
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 12. String Methods

All string methods return a **new string** — the original is never modified (strings are immutable).

```js
// trim / trimStart / trimEnd
'  hello  '.trim()       // 'hello'
'  hello  '.trimStart()  // 'hello  '
'  hello  '.trimEnd()    // '  hello'

// slice(start, end) — supports negative indices
'hello'.slice(1, 3)      // 'el'
'hello'.slice(-3)        // 'llo'   ← counts from the end
'hello'.slice(1)         // 'ello'

// substring(start, end) — does NOT support negative indices
'hello'.substring(1, 3)  // 'el'
'hello'.substring(-3)    // 'hello' ← negative becomes 0

// includes / startsWith / endsWith
'hello world'.includes('world')     // true
'hello'.startsWith('he')            // true
'hello'.endsWith('lo')              // true

// replace vs replaceAll
'aabbcc'.replace('b', 'X')          // 'aaXbcc'  ← first occurrence only
'aabbcc'.replaceAll('b', 'X')       // 'aaXXcc'  ← all occurrences

// repeat / padStart / padEnd
'ha'.repeat(3)                      // 'hahaha'
'5'.padStart(4, '0')               // '0005'
'5'.padEnd(4, '-')                 // '5---'

// split
'a,b,c'.split(',')     // ['a', 'b', 'c']
'hello'.split('')      // ['h', 'e', 'l', 'l', 'o']
```

```js
console.log('hello'.slice(-3));
console.log('hello'.substring(-3));
console.log('aabbcc'.replace('b', 'X'));
console.log('5'.padStart(4, '0'));
```

### Output

```js
'llo'
'hello'
'aaXbcc'
'0005'
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 13. Number Methods and Quirks

```js
// NaN — "Not a Number", but typeof is 'number'
console.log(typeof NaN);   // 'number'
console.log(NaN === NaN);  // false — NaN is never equal to itself

// isNaN vs Number.isNaN — critical interview distinction
isNaN('hello')          // true  — coerces 'hello' to NaN, then checks
isNaN(undefined)        // true  — coerces to NaN first
isNaN('')               // false — '' coerces to 0, isNaN(0) is false!
Number.isNaN('hello')   // false — NO coercion, strict NaN check
Number.isNaN(undefined) // false — undefined is not literally NaN
Number.isNaN(NaN)       // true  — only true for actual NaN

// isFinite vs Number.isFinite
isFinite('10')          // true  — coerces '10' to 10 first
Number.isFinite('10')   // false — no coercion, '10' is not a finite number

// parseInt / parseFloat
parseInt('42px')        // 42    — parses until non-numeric character
parseInt('px42')        // NaN   — must START with a digit
parseInt('0xff', 16)    // 255   — second arg is radix
parseFloat('3.14abc')   // 3.14

// Infinity
console.log(1 / 0);          // Infinity
console.log(-1 / 0);         // -Infinity
console.log(Infinity + 1);   // Infinity

// toFixed — returns a STRING
console.log((3.14159).toFixed(2));  // '3.14'
console.log((1.005).toFixed(2));    // '1.00' — floating point imprecision!

// Floating point precision
console.log(0.1 + 0.2);            // 0.30000000000000004
console.log(0.1 + 0.2 === 0.3);    // false
```

### Output

```js
'number'
false
0.30000000000000004
false
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 14. Math Object

```js
Math.floor(4.9)     // 4   — always rounds DOWN toward -Infinity
Math.ceil(4.1)      // 5   — always rounds UP toward +Infinity
Math.round(4.5)     // 5   — rounds to nearest, ties go UP (+Infinity)
Math.round(4.49)    // 4
Math.round(-4.5)    // -4  — ties go UP, so -4.5 rounds to -4 (not -5!)

Math.trunc(4.9)     // 4   — removes decimal part, toward zero
Math.trunc(-4.9)    // -4  — compare: Math.floor(-4.9) = -5

Math.abs(-7)        // 7
Math.pow(2, 10)     // 1024
Math.sqrt(16)       // 4
Math.cbrt(27)       // 3

Math.max(1, 2, 3)   // 3
Math.min(1, 2, 3)   // 1
Math.max()          // -Infinity (identity element for max)
Math.min()          // Infinity  (identity element for min)

// Use spread to apply to an array
Math.max(...[1, 5, 3, 9, 2])  // 9

// Random integer in [min, max] inclusive
Math.floor(Math.random() * (max - min + 1)) + min;

Math.PI   // 3.141592653589793
Math.E    // 2.718281828459045
```

```js
console.log(Math.round(-4.5));  // -4
console.log(Math.floor(-4.5));  // -5
console.log(Math.trunc(-4.9));  // -4
console.log(Math.max());        // -Infinity
```

### Output

```js
-4
-5
-4
-Infinity
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 15. Template Literals

Template literals use backticks (`` ` ``) and enable interpolation, multi-line strings, and tagged templates.

```js
const name = 'Alice';
const age = 30;

// Basic interpolation
console.log(`Name: ${name}, Age: ${age}`);  // 'Name: Alice, Age: 30'

// Any expression is valid inside ${}
console.log(`Result: ${2 + 2}`);                          // 'Result: 4'
console.log(`Status: ${age >= 18 ? 'adult' : 'minor'}`); // 'Status: adult'

// Multi-line strings — newlines are preserved literally
const poem = `Line 1
Line 2
Line 3`;

// Tagged templates — function receives array of string parts + interpolated values
function highlight(strings, ...values) {
  return strings.reduce((acc, str, i) =>
    acc + str + (values[i] !== undefined ? `[${values[i]}]` : ''), '');
}
const item = 'coffee';
const price = 4.5;
console.log(highlight`I want ${item} for $${price}`);
// 'I want [coffee] for $[4.5]'

// Nested template literals
const a = 5, b = 10;
console.log(`Sum: ${`${a} + ${b}`} = ${a + b}`);
// 'Sum: 5 + 10 = 15'
```

### Output

```js
'Name: Alice, Age: 30'
'Result: 4'
'Status: adult'
'I want [coffee] for $[4.5]'
'Sum: 5 + 10 = 15'
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 16. Spread Operator

The spread operator (`...`) expands an iterable into individual elements.

```js
// Array spread
const a = [1, 2, 3];
const b = [4, 5, 6];
const combined = [...a, ...b];          // [1, 2, 3, 4, 5, 6]
const copy = [...a];                    // shallow copy

// Object spread — later keys overwrite earlier keys
const obj1 = { x: 1, y: 2 };
const obj2 = { y: 3, z: 4 };
const merged = { ...obj1, ...obj2 };   // { x: 1, y: 3, z: 4 }
const clone = { ...obj1 };             // shallow copy

// Spread in function calls
function sum(a, b, c) { return a + b + c; }
const nums = [1, 2, 3];
console.log(sum(...nums));  // 6

// String to array of characters
console.log([...'hello']);  // ['h', 'e', 'l', 'l', 'o']

// Spread is SHALLOW — nested objects are still shared
const nested = { a: { b: 1 } };
const shallowCopy = { ...nested };
shallowCopy.a.b = 99;
console.log(nested.a.b);   // 99 — both point to the same inner object!
```

### Output

```js
6
['h', 'e', 'l', 'l', 'o']
99
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 17. Rest Parameters

Rest parameters collect the remaining arguments into a real **array**. The rest parameter must always be the **last** parameter.

```js
function sum(...nums) {
  return nums.reduce((acc, n) => acc + n, 0);
}
console.log(sum(1, 2, 3, 4)); // 10

// Mixed with regular parameters
function greet(greeting, ...names) {
  return names.map(name => `${greeting}, ${name}!`);
}
console.log(greet('Hello', 'Alice', 'Bob'));
// ['Hello, Alice!', 'Hello, Bob!']

// Rest vs the arguments object
function withArguments() {
  console.log(Array.isArray(arguments)); // false — arguments is array-like, not a real array
  // arguments.map(...)  would throw TypeError
}
function withRest(...args) {
  console.log(Array.isArray(args));      // true — rest IS a real array
  args.map(x => x * 2);                 // works fine
}

// Rest in array destructuring
const [first, second, ...rest] = [1, 2, 3, 4, 5];
console.log(first);  // 1
console.log(rest);   // [3, 4, 5]
```

### Output

```js
10
['Hello, Alice!', 'Hello, Bob!']
false
true
1
[3, 4, 5]
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 18. Destructuring

### Array Destructuring

```js
// Basic
const [a, b, c] = [1, 2, 3];

// Skip elements using empty slots
const [, second, , fourth] = [1, 2, 3, 4];
console.log(second, fourth);   // 2 4

// Default values — only triggered by undefined, NOT null
const [x = 10, y = 20] = [5, undefined];
console.log(x, y);  // 5 20

// Swapping variables without a temp variable
let p = 1, q = 2;
[p, q] = [q, p];
console.log(p, q);  // 2 1
```

### Object Destructuring

```js
const user = { name: 'Alice', age: 30, city: 'Mumbai' };

// Basic
const { name, age } = user;

// Rename with colon — variable is 'fullName', not 'name'
const { name: fullName } = user;
console.log(fullName);  // 'Alice'
// console.log(name);   — still works (from the basic destructure above)

// Default values
const { role = 'admin' } = user;
console.log(role);  // 'admin' — not in user, so default is used

// Nested
const { address: { street } = {} } = { address: { street: '5th Ave' } };
console.log(street);  // '5th Ave'

// In function parameters
function display({ name, age = 0 }) {
  return `${name} is ${age}`;
}
console.log(display({ name: 'Bob', age: 25 })); // 'Bob is 25'
```

### Output

```js
2 4
5 20
2 1
'Alice'
'admin'
'5th Ave'
'Bob is 25'
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 19. Object Shorthand and Computed Properties

```js
// Property shorthand — when variable name matches the key
const name = 'Alice';
const age = 30;
const user = { name, age };  // same as { name: name, age: age }

// Method shorthand
const obj = {
  greet() {          // instead of greet: function() {}
    return 'hello';
  },
  get name() {       // getter shorthand
    return 'Alice';
  }
};

// Computed property names — dynamic keys from expressions
const key = 'color';
const theme = {
  [key]: 'blue',              // { color: 'blue' }
  [`${key}Dark`]: 'navy'      // { colorDark: 'navy' }
};
console.log(theme);
// { color: 'blue', colorDark: 'navy' }

// Computed keys in class-style factory objects
const prefix = 'get';
const api = {
  [`${prefix}Name`]() { return 'Alice'; },
  [`${prefix}Age`]()  { return 30; }
};
console.log(api.getName()); // 'Alice'
console.log(api.getAge());  // 30
```

### Output

```js
{ color: 'blue', colorDark: 'navy' }
'Alice'
30
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 20. Comma Operator

The comma operator evaluates each operand **left to right** and returns the **value of the last operand**. It rarely appears in application code but is a classic interview trick.

```js
// Comma in expression context
let x = (1, 2, 3);
console.log(x);  // 3 — only the last value is returned

// Legitimate use: for loop with multiple variables
for (let i = 0, j = 10; i < j; i++, j--) {
  // i increments and j decrements together
}

// Tricky behavior with post-increment
let a = 1;
let b = (a++, a++, a++);
// a++ returns current value, then increments
// After 3 increments: a = 4, but b = value of last a++ = 3
console.log(a);  // 4
console.log(b);  // 3

// In return (rare but valid)
function sideEffect() {
  return (console.log('side effect'), 42);
}
console.log(sideEffect());
// logs: 'side effect'
// logs: 42
```

### Output

```js
3
4
3
'side effect'
42
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 21. typeof vs instanceof

`typeof` checks the **primitive type** of a value and returns a string.
`instanceof` checks the **prototype chain** — whether an object was created by a specific constructor.

```js
// typeof works well for primitives
console.log(typeof 42);         // 'number'
console.log(typeof 'hello');    // 'string'
console.log(typeof true);       // 'boolean'

// typeof fails to distinguish object subtypes
console.log(typeof []);         // 'object'  — not 'array'!
console.log(typeof null);       // 'object'  — not 'null'!

// instanceof walks the prototype chain
console.log([] instanceof Array);    // true
console.log([] instanceof Object);   // true  — Array inherits from Object
console.log({} instanceof Object);   // true

// instanceof fails with primitives (they are not objects)
console.log(42 instanceof Number);   // false — 42 is a primitive, not new Number()
console.log('hi' instanceof String); // false

// Best practices for type checking
Array.isArray([]);                        // true  — preferred array check
Object.prototype.toString.call([]);       // '[object Array]'
Object.prototype.toString.call(null);     // '[object Null]'
Object.prototype.toString.call(/regex/);  // '[object RegExp]'
```

### Output

```js
'number'
'string'
'boolean'
'object'
'object'
true
true
true
false
false
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 22. Garbage Collection Basics

JavaScript uses **automatic memory management**. The primary algorithm is **Mark-and-Sweep**: starting from GC roots (global scope, call stack), the engine marks everything reachable. Anything unmarked is collected.

```js
// Memory is freed when no more references exist
let user = { name: 'Alice' };
user = null;  // { name: 'Alice' } object is now unreachable — eligible for GC

// Circular references are handled (unlike old reference-counting)
function createCycle() {
  let a = {};
  let b = {};
  a.ref = b;
  b.ref = a;
  // When this function exits, neither a nor b is reachable from any root
  // Mark-and-Sweep correctly collects both, despite the circular reference
}

// Common sources of memory leaks (interview question):
// 1. Uncleared setInterval / setTimeout holding references
// 2. Detached DOM nodes still referenced in JavaScript
// 3. Closures unintentionally retaining large objects in scope
// 4. Growing global variables / caches without eviction

// WeakMap and WeakSet hold WEAK references — do not prevent GC
let obj = { data: 'important' };
const weakMap = new WeakMap();
weakMap.set(obj, 'metadata');
obj = null;  // obj is eligible for GC; WeakMap entry is removed automatically
// This is why WeakMap keys MUST be objects — primitives are always accessible
```

### Output

```js
// No direct output — demonstrates memory lifecycle concepts
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 23. Summary

| Topic | Key Interview Point |
|---|---|
| Dynamic typing | Variables have no fixed type; coercion happens implicitly |
| `typeof null` | Returns `'object'` — a historical JS bug that cannot be fixed |
| `typeof NaN` | Returns `'number'` — NaN belongs to the number type |
| 7 primitives | `string`, `number`, `bigint`, `boolean`, `null`, `undefined`, `symbol` |
| `null == undefined` | `true` with `==`, `false` with `===` |
| `NaN == NaN` | Always `false` — use `Number.isNaN()` to detect NaN |
| Falsy values | `false`, `0`, `-0`, `0n`, `''`, `null`, `undefined`, `NaN` (exactly 8) |
| `'0'` and `[]` and `{}` | Always truthy — they are non-empty string and object references |
| `??` vs `\|\|` | `??` only triggers on `null`/`undefined`; `\|\|` triggers on any falsy value |
| `?.` | Returns `undefined` instead of throwing `TypeError` on null/undefined access |
| `const` | Prevents re-assignment, NOT mutation of the object's contents |
| `slice` vs `substring` | `slice` supports negative indices; `substring` treats negatives as `0` |
| `isNaN` vs `Number.isNaN` | `isNaN` coerces first (may give surprising results); `Number.isNaN` is strict |
| `0.1 + 0.2 === 0.3` | `false` — floating point representation cannot be compared directly |
| Spread shallow copy | Nested objects are still shared between original and copy |
| `typeof` undeclared | Does NOT throw — safely returns `'undefined'` |
| `typeof` undeclared | Does NOT throw — safely returns `'undefined'` |
| `instanceof` with primitives | Returns `false` — primitives are not objects |
| Comma operator | Evaluates left to right, returns the value of the last operand |
| Mark-and-Sweep GC | Handles circular references; WeakMap/WeakSet allow GC-friendly storage |

---

## Final Notes

JavaScript fundamentals form the foundation of every technical interview. The areas that trip up even experienced developers are the type coercion rules governing `==`, the complete list of exactly 8 falsy values (particularly `'0'` and `[]` being truthy), `typeof` edge cases for `null`, `NaN`, and arrays, and the distinction between `??` and `||`. Pay close attention to `Number.isNaN` versus `isNaN` — the coercion behavior of the global `isNaN` produces counter-intuitive results like `isNaN('') === false`. When preparing, always verify outputs in a REPL, because JavaScript's implicit coercions are rules to be memorized, not inferred.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
