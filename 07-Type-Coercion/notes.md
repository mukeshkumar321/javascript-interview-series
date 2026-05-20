# Type Coercion in JavaScript

## Table of Contents

1. [What is Type Coercion?](#1-what-is-type-coercion)
2. [Implicit vs Explicit Coercion](#2-implicit-vs-explicit-coercion)
3. [String Conversion](#3-string-conversion)
4. [Number Conversion](#4-number-conversion)
5. [Boolean Conversion](#5-boolean-conversion)
6. [The + Operator](#6-the--operator)
7. [The - * / % Operators](#7-the-----operators)
8. [Comparison Operators with Coercion](#8-comparison-operators-with-coercion)
9. [Abstract Equality (==) Algorithm](#9-abstract-equality--algorithm)
10. [null == undefined (Special Case)](#10-null--undefined-special-case)
11. [null == 0 (False)](#11-null--0-false)
12. [NaN Comparisons](#12-nan-comparisons)
13. [Object to Primitive Conversion](#13-object-to-primitive-conversion)
14. [Array Coercion](#14-array-coercion)
15. [Boolean Context with Objects](#15-boolean-context-with-objects)
16. [Explicit Coercion Best Practices](#16-explicit-coercion-best-practices)
17. [Summary Table](#17-summary-table)

---

## 1. What is Type Coercion?

**Type coercion** is the automatic or manual conversion of a value from one type to another. JavaScript is dynamically typed — variables have no fixed type — so the engine frequently converts values to make operations work. This can produce surprising results if you are not aware of the rules.

```js
console.log(1 + "2");    // "12"  — number coerced to string
console.log("5" - 2);    // 3     — string coerced to number
console.log(true + 1);   // 2     — boolean coerced to number
console.log(null + 1);   // 1     — null coerced to 0
```

### Output

```js
12
3
2
1
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 2. Implicit vs Explicit Coercion

**Implicit coercion** happens automatically when the engine needs to resolve a type mismatch (e.g., using `+` with a string and a number). **Explicit coercion** is performed intentionally by the developer using built-in functions.

```js
// Implicit — engine decides the type conversion
console.log("3" - 1);    // 2
console.log(!!0);         // false
console.log(1 == "1");    // true

// Explicit — developer intentionally converts
console.log(Number("3") - 1);   // 2
console.log(Boolean(0));         // false
console.log(String(42));         // "42"
```

### Output

```js
2
false
true
2
false
42
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 3. String Conversion

Convert to string using `String()`, `.toString()`, or template literals. Each behaves slightly differently with edge-case values.

```js
// String() — handles null/undefined without throwing
console.log(String(123));       // "123"
console.log(String(-0));        // "0"
console.log(String(true));      // "true"
console.log(String(null));      // "null"
console.log(String(undefined)); // "undefined"
console.log(String([1, 2, 3])); // "1,2,3"
console.log(String({}));        // "[object Object]"

// .toString() — throws on null/undefined
console.log((42).toString());   // "42"
console.log((255).toString(16)); // "ff"  (base 16)

// Template literals — use String() rules internally
console.log(`${null}`);         // "null"
console.log(`${[1, 2]}`);       // "1,2"
```

### Output

```js
123
0
true
null
undefined
1,2,3
[object Object]
42
ff
null
1,2
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 4. Number Conversion

Convert to number using `Number()`, `parseInt()`, `parseFloat()`, or the unary `+` operator.

```js
// Number() — strict conversion
console.log(Number(""));         // 0   — empty string is 0
console.log(Number("  123  "));  // 123 — trims whitespace
console.log(Number("123abc"));   // NaN — non-numeric characters
console.log(Number(null));       // 0
console.log(Number(undefined));  // NaN
console.log(Number(true));       // 1
console.log(Number(false));      // 0
console.log(Number([]));         // 0   — [] → "" → 0
console.log(Number([3]));        // 3   — [3] → "3" → 3
console.log(Number([1, 2]));     // NaN — [1,2] → "1,2" → NaN

// parseInt / parseFloat — parse from left, stop at first non-numeric char
console.log(parseInt("10px"));   // 10
console.log(parseFloat("3.14em")); // 3.14
console.log(parseInt("abc"));    // NaN

// Unary + — equivalent to Number()
console.log(+"7");               // 7
console.log(+true);              // 1
console.log(+null);              // 0
console.log(+undefined);         // NaN
```

### Output

```js
0
123
NaN
0
NaN
1
0
0
3
NaN
10
3.14
NaN
7
1
0
NaN
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 5. Boolean Conversion

Convert to boolean using `Boolean()` or the double-negation `!!` operator. The rules are simple: there are exactly **six falsy values** — everything else is truthy.

```js
// The six falsy values
console.log(Boolean(false));      // false
console.log(Boolean(0));          // false
console.log(Boolean(-0));         // false
console.log(Boolean(0n));         // false — BigInt zero
console.log(Boolean(""));         // false
console.log(Boolean(null));       // false
console.log(Boolean(undefined));  // false
console.log(Boolean(NaN));        // false

// Everything else is truthy — including these common surprises
console.log(Boolean("0"));        // true — non-empty string
console.log(Boolean([]));         // true — empty array
console.log(Boolean({}));         // true — empty object
console.log(Boolean("false"));    // true — non-empty string
```

### Output

```js
false
false
false
false
false
false
false
false
true
true
true
true
```

> For a complete truthy/falsy reference and practical examples see `01-Fundamentals/notes.md`.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 6. The + Operator

The `+` operator is **overloaded** — it performs string concatenation when at least one operand is a string, and numeric addition otherwise. The engine calls `ToPrimitive` on objects before deciding.

```js
// Addition when both are numbers
console.log(1 + 2);          // 3

// Concatenation when either is a string
console.log(1 + "2");        // "12"
console.log("2" + 1);        // "21"

// Left-to-right evaluation matters
console.log(1 + 2 + "3");    // "33"  — 1+2=3, then 3+"3"="33"
console.log("1" + 2 + 3);    // "123" — "1"+2="12", then "12"+3="123"

// boolean / null / undefined coerced to number when the other side is a number
console.log(true + 1);       // 2
console.log(false + 1);      // 1
console.log(null + 1);       // 1
console.log(undefined + 1);  // NaN

// Object/array — ToPrimitive is called first
console.log([] + []);        // ""                — both → ""
console.log([] + {});        // "[object Object]" — "" + "[object Object]"
console.log({} + []);        // "[object Object]" — expression context
```

### Output

```js
3
12
21
33
123
2
1
1
NaN

[object Object]
[object Object]
```

> Note: `{} + []` at the **start of a statement** (not inside an expression) is parsed as an empty block followed by `+[]`, which equals `0`. Inside `console.log()` it is always in expression context and evaluates to `"[object Object]"`.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 7. The - * / % Operators

The arithmetic operators `-`, `*`, `/`, and `%` **always coerce both operands to numbers**. There is no string concatenation shortcut for these operators.

```js
console.log("6" - 2);     // 4    — "6" → 6
console.log("6" * "2");   // 12   — both → numbers
console.log("6" / "2");   // 3
console.log("7" % "3");   // 1

console.log(true - 1);    // 0    — true → 1
console.log(null - 1);    // -1   — null → 0
console.log("" - 1);      // -1   — "" → 0
console.log("abc" - 1);   // NaN  — "abc" → NaN

console.log([] - []);     // 0    — [] → "" → 0, so 0 - 0
console.log([3] - [1]);   // 2    — "3" → 3, "1" → 1
```

### Output

```js
4
12
3
1
0
-1
-1
NaN
0
2
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 8. Comparison Operators with Coercion

The `<`, `>`, `<=`, `>=` operators coerce both operands to numbers unless **both** are strings (in which case lexicographic comparison is used).

```js
console.log("10" > 9);      // true  — "10" → 10, 10 > 9
console.log("10" > "9");    // false — lexicographic: "1" < "9"
console.log(null > 0);      // false — null → 0, 0 > 0 is false
console.log(null == 0);     // false — special rule (see section 10)
console.log(null >= 0);     // true  — null → 0, 0 >= 0 is true (confusing!)
console.log(undefined > 0); // false — NaN comparisons always false
console.log(undefined < 0); // false
console.log(undefined == 0); // false
```

### Output

```js
true
false
false
false
true
false
false
false
```

> The `null >= 0` being `true` while `null == 0` is `false` is a well-known JavaScript quirk: `>=` uses numeric coercion (null → 0), while `==` has a special rule that `null` only equals `undefined`.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 9. Abstract Equality (==) Algorithm

The `==` operator uses the **Abstract Equality Comparison** algorithm which applies type coercion. The steps (simplified from the spec):

1. If types are the same → use strict equality (`===`).
2. `null == undefined` → `true` (and vice versa).
3. `null` or `undefined` compared to anything else → `false`.
4. If one operand is a **number** and the other is a **string** → convert string to number, compare.
5. If one operand is a **boolean** → convert it to number, restart.
6. If one operand is an **object** and the other is a string, number, or symbol → call `ToPrimitive` on the object, restart.
7. Otherwise → `false`.

```js
console.log(1 == "1");      // true  — "1" → 1
console.log(1 == true);     // true  — true → 1
console.log(0 == false);    // true  — false → 0
console.log(0 == "");       // true  — "" → 0
console.log("" == false);   // true  — false → 0, "" → 0
console.log("1" == true);   // true  — true → 1, "1" → 1
console.log(null == undefined); // true — special rule
console.log(null == 0);     // false — special rule (null only equals undefined)
```

### Output

```js
true
true
true
true
true
true
true
false
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 10. null == undefined (Special Case)

`null == undefined` evaluates to `true` by a special rule in the Abstract Equality algorithm. Neither value is coerced to a number or string — they are simply defined as equal to each other and to nothing else.

```js
console.log(null == undefined);  // true
console.log(undefined == null);  // true  — symmetric
console.log(null === undefined); // false — different types

// null/undefined do NOT equal any other falsy value
console.log(null == 0);         // false
console.log(null == false);     // false
console.log(null == "");        // false
console.log(undefined == 0);   // false
console.log(undefined == false); // false
```

### Output

```js
true
true
false
false
false
false
false
false
```

This rule is commonly used for null-checking: `if (value == null)` catches both `null` and `undefined` in a single check.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 11. null == 0 (False)

Despite `null` being converted to `0` in arithmetic operations, `null == 0` is **false**. The `==` algorithm has a special gate: if either operand is `null` or `undefined`, the result is `true` only if the other operand is also `null` or `undefined`.

```js
// Arithmetic: null → 0
console.log(null + 0);    // 0
console.log(null * 5);    // 0
console.log(null > -1);   // true  — null → 0, 0 > -1

// Abstract equality: null only equals undefined
console.log(null == 0);   // false
console.log(null == "");  // false
console.log(null == false); // false
console.log(null >= 0);   // true  — >= uses numeric coercion (not == rules)
```

### Output

```js
0
0
true
false
false
false
true
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 12. NaN Comparisons

`NaN` (Not a Number) is the result of an undefined numeric operation. It has the unique property of **not being equal to itself** — `NaN !== NaN` is the only value in JavaScript for which `x !== x` is true.

```js
console.log(NaN == NaN);          // false — NaN is never equal to anything
console.log(NaN === NaN);         // false
console.log(NaN != NaN);          // true
console.log(NaN !== NaN);         // true

// How to actually check for NaN
console.log(isNaN(NaN));          // true
console.log(isNaN("hello"));      // true  — coerces first: Number("hello") = NaN
console.log(Number.isNaN(NaN));   // true
console.log(Number.isNaN("hello")); // false — no coercion, "hello" is not NaN

// How NaN is produced
console.log(0 / 0);               // NaN
console.log(parseInt("abc"));     // NaN
console.log(Math.sqrt(-1));       // NaN
```

### Output

```js
false
false
true
true
true
true
true
false
NaN
NaN
NaN
```

> Always prefer `Number.isNaN()` over `isNaN()` — the latter coerces its argument first, which can produce false positives.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 13. Object to Primitive Conversion

When an object participates in a context that requires a primitive (number, string, or default), the engine calls the internal `ToPrimitive` operation with a **hint**:

- `"number"` hint (arithmetic, comparison) → tries `valueOf()` first, then `toString()`
- `"string"` hint (template literals, `String()`) → tries `toString()` first, then `valueOf()`
- `"default"` hint (`+`, `==`) → behaves like `"number"` for most objects

You can override this with `Symbol.toPrimitive`.

```js
const obj = {
  valueOf() { return 10; },
  toString() { return "hello"; }
};

console.log(obj + "");    // "10"   — default hint → valueOf() → 10 → "10" + "" = "10"
console.log(`${obj}`);   // "hello" — string hint → toString() → "hello"
console.log(obj * 2);    // 20      — number hint → valueOf() → 10 × 2 = 20
console.log(obj > 5);    // true    — number hint → valueOf() → 10 > 5

// Symbol.toPrimitive gives full control
const smart = {
  [Symbol.toPrimitive](hint) {
    if (hint === "number") return 42;
    if (hint === "string") return "forty-two";
    return true; // default hint
  }
};

console.log(+smart);      // 42
console.log(`${smart}`);  // "forty-two"
console.log(smart + "");  // "true"  — default → true → "true"
```

### Output

```js
10
hello
20
true
42
forty-two
true
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 14. Array Coercion

Arrays follow the same `ToPrimitive` rules as other objects. `Array.prototype.toString()` joins elements with commas (same as `.join(",")`). An empty array becomes an empty string.

```js
// Array to string
console.log(String([]));         // ""
console.log(String([1, 2, 3]));  // "1,2,3"
console.log(String([null]));     // ""   — null/undefined elements become ""
console.log(String([1, null, 3])); // "1,,3"

// Array to number
console.log(Number([]));         // 0    — [] → "" → 0
console.log(Number([3]));        // 3    — [3] → "3" → 3
console.log(Number([1, 2]));     // NaN  — [1,2] → "1,2" → NaN

// The famous + operator with arrays/objects
console.log([] + []);            // ""                 — "" + "" = ""
console.log([] + {});            // "[object Object]"  — "" + "[object Object]"
console.log({} + []);            // "[object Object]"  — expression context
console.log([] - []);            // 0                  — 0 - 0 = 0
console.log([3] - [1]);          // 2                  — 3 - 1 = 2
```

### Output

```js

1,2,3

1,,3
0
3
NaN

[object Object]
[object Object]
0
2
```

> `{} + []` evaluated as a statement (top-level in the console, not inside an expression) would be `0`, because `{}` is treated as an empty block and `+[]` is `0`. Inside `console.log()` it is always an expression.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 15. Boolean Context with Objects

In a boolean context (e.g., `if`, `while`, `||`, `&&`, `!`), **all objects are truthy** — including empty arrays `[]` and empty objects `{}`. This is independent of their string or numeric value.

```js
// All objects are truthy in boolean context
if ([])  console.log("[] is truthy");     // runs
if ({})  console.log("{} is truthy");     // runs
if (new Boolean(false)) console.log("new Boolean(false) is truthy"); // runs!

// Equality with false uses coercion (different rules from boolean context)
console.log([] == false);   // true   — [] → "" → 0, false → 0
console.log(Boolean([]));   // true   — boolean context: objects are truthy
console.log(!!{});          // true

// Practical gotcha
const arr = [];
if (arr) {
  console.log("arr is truthy — object reference");
}
if (arr == false) {
  console.log("arr == false — because ToPrimitive gives ''");
}
```

### Output

```js
[] is truthy
{} is truthy
new Boolean(false) is truthy
true
true
true
arr is truthy — object reference
arr == false — because ToPrimitive gives ''
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 16. Explicit Coercion Best Practices

Relying on implicit coercion makes code harder to read and debug. Prefer explicit conversions so intent is obvious.

```js
// String conversion — prefer String() or template literals
const num = 42;
const str1 = String(num);     // explicit — preferred
const str2 = num + "";        // implicit — avoid
const str3 = `${num}`;        // template literal — fine for interpolation

// Number conversion — prefer Number() or unary +
const input = "3.14";
const n1 = Number(input);     // explicit — preferred
const n2 = +input;            // unary + — acceptable shorthand
const n3 = input * 1;         // implicit — avoid

// Boolean conversion — prefer Boolean() or !!
const val = "";
const b1 = Boolean(val);      // explicit — preferred
const b2 = !!val;             // double-negation — widely accepted

// Parsing integers and floats from strings
console.log(parseInt("100px", 10));   // 100  — always provide radix
console.log(parseFloat("3.14rem"));   // 3.14

// Equality — prefer === over ==
console.log(1 === 1);   // true  — no coercion
console.log(1 == "1");  // true  — implicit coercion: surprising
```

### Output

```js
100
3.14
true
true
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 17. Summary Table

| Concept | Key Point |
|---|---|
| Type coercion | Automatic or manual conversion between types |
| Implicit coercion | Engine converts automatically (e.g., `"5" - 1` → `4`) |
| Explicit coercion | Developer converts intentionally (`Number()`, `String()`, `Boolean()`) |
| String conversion | `String(null)` → `"null"`, `String(undefined)` → `"undefined"` |
| Number conversion | `Number("")` → `0`, `Number(null)` → `0`, `Number(undefined)` → `NaN` |
| Falsy values | `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, `NaN` — exactly 8 |
| All objects are truthy | `[]`, `{}`, `new Boolean(false)` — all truthy in boolean context |
| `+` operator | String concat if either operand is a string; numeric add otherwise |
| `-` `*` `/` `%` | Always convert operands to numbers — no string shortcut |
| `<` `>` `<=` `>=` | Numeric comparison unless both sides are strings |
| `==` algorithm | Applies coercion according to specific rules — prefer `===` |
| `null == undefined` | `true` — they are defined as equal to each other by the spec |
| `null == 0` | `false` — null only equals null/undefined via `==` |
| `null` in arithmetic | Coerced to `0` — so `null + 1` is `1` |
| `NaN == NaN` | `false` — NaN is the only value not equal to itself |
| `Number.isNaN` | Does not coerce — use this instead of `isNaN()` |
| `ToPrimitive` "number" | `valueOf()` first, then `toString()` |
| `ToPrimitive` "string" | `toString()` first, then `valueOf()` |
| `Symbol.toPrimitive` | Override all three hints with one method |
| `[] + []` | `""` — both arrays coerce to empty string |
| `[] + {}` | `"[object Object]"` |
| `[] == false` | `true` — `[]` → `""` → `0`, `false` → `0` |

---

## Final Notes

Type coercion is one of the most frequently misunderstood areas of JavaScript and a staple of interview questions. The two most important rules to internalize are: (1) the `+` operator prefers strings when either side is a string, while all other arithmetic operators force numeric conversion; and (2) abstract equality (`==`) follows a multi-step algorithm involving type coercion — `null` and `undefined` are special cases, and objects are converted via `ToPrimitive`. In production code, almost always prefer strict equality (`===`) and explicit conversion functions to avoid coercion surprises.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
