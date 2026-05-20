# JavaScript Fundamentals — Tricky Output Questions

## Table of Contents

1. [typeof Questions](#1-typeof-questions)
2. [null vs undefined Questions](#2-null-vs-undefined-questions)
3. [== vs === Questions](#3--vs--questions)
4. [Truthy and Falsy Questions](#4-truthy-and-falsy-questions)
5. [Short-Circuit Questions](#5-short-circuit-questions)
6. [NaN and Number Questions](#6-nan-and-number-questions)
7. [String Questions](#7-string-questions)
8. [Spread and Destructuring Questions](#8-spread-and-destructuring-questions)
9. [Operator Precedence Questions](#9-operator-precedence-questions)
10. [Advanced Edge Cases](#10-advanced-edge-cases)

---

## 1. typeof Questions

---

### Q1. What will be the output?

```js
console.log(typeof null);
console.log(typeof undefined);
console.log(typeof NaN);
console.log(typeof Infinity);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
'object'
'undefined'
'number'
'number'
```

### Explanation

`typeof null === 'object'` is a historical bug in JavaScript that has never been fixed to avoid breaking existing code. `NaN` and `Infinity` both belong to the `number` type — they are special numeric values, not a separate type.

</details>

---

### Q2. What will be the output?

```js
console.log(typeof []);
console.log(typeof {});
console.log(typeof function() {});
console.log(typeof class Foo {});
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
'object'
'object'
'function'
'function'
```

### Explanation

Arrays are objects in JavaScript, so `typeof []` is `'object'`. Regular functions and classes both return `'function'`. Classes are syntactic sugar over constructor functions — they are still functions under the hood. There is no `'array'` or `'class'` result from `typeof`.

</details>

---

### Q3. What will be the output?

```js
console.log(typeof typeof 42);
console.log(typeof typeof undefined);
console.log(typeof typeof []);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
'string'
'string'
'string'
```

### Explanation

`typeof` always returns a **string**. So `typeof 42` is `'number'` (a string), and then `typeof 'number'` is `'string'`. No matter what the inner operand is, `typeof (typeof x)` always evaluates to `'string'`.

</details>

---

### Q4. What will be the output?

```js
console.log(typeof undeclaredVariable);
// console.log(undeclaredVariable);  // What would this do?
console.log(typeof Symbol());
console.log(typeof 42n);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
'undefined'
'symbol'
'bigint'
```

### Explanation

`typeof` is the **only** operator that can be safely used on an undeclared variable without throwing a `ReferenceError`. It returns `'undefined'`. Accessing `undeclaredVariable` directly (the commented line) would throw `ReferenceError: undeclaredVariable is not defined`. `Symbol()` and BigInt literals (`42n`) have their own dedicated `typeof` results: `'symbol'` and `'bigint'`.

</details>

---

### Q5. What will be the output?

```js
const fn = () => {};
const gen = function* () {};
const asyncFn = async function() {};

console.log(typeof fn);
console.log(typeof gen);
console.log(typeof asyncFn);
console.log(typeof class {});
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
'function'
'function'
'function'
'function'
```

### Explanation

Arrow functions, generator functions, async functions, and classes all return `'function'` from `typeof`. The `typeof` operator does not distinguish between these different kinds of callable objects — they all share the `'function'` result.

</details>

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 2. null vs undefined Questions

---

### Q6. What will be the output?

```js
console.log(null + 1);
console.log(undefined + 1);
console.log(null + undefined);
console.log(null + null);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
1
NaN
NaN
0
```

### Explanation

In arithmetic, `null` coerces to `0` and `undefined` coerces to `NaN`. So: `null + 1` = `0 + 1` = `1`, `undefined + 1` = `NaN + 1` = `NaN`, `null + undefined` = `0 + NaN` = `NaN`, and `null + null` = `0 + 0` = `0`.

</details>

---

### Q7. What will be the output?

```js
function test(value = 'default') {
  return value;
}

console.log(test(undefined));
console.log(test(null));
console.log(test(0));
console.log(test(''));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
'default'
null
0
''
```

### Explanation

Default parameter values are only triggered when the argument is `undefined` (or omitted entirely). `null`, `0`, and `''` are all valid values that do NOT trigger the default — they are passed through as-is. This is a key difference between `null` and `undefined` that frequently appears in interviews.

</details>

---

### Q8. What will be the output?

```js
console.log(JSON.stringify({ a: 1, b: undefined, c: null, d: function() {} }));
console.log(JSON.stringify([1, undefined, null, function() {}]));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
'{"a":1,"c":null}'
'[1,null,null,null]'
```

### Explanation

`JSON.stringify` handles `undefined` and functions differently depending on context. In **objects**, properties whose values are `undefined` or functions are **omitted entirely**. In **arrays**, `undefined` and functions are converted to `null` to preserve the array's index structure. `null` is preserved in both cases.

</details>

---

### Q9. What will be the output?

```js
console.log(null > 0);
console.log(null == 0);
console.log(null >= 0);
console.log(null < 1);
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

This is one of the most notorious JavaScript quirks. Relational operators (`>`, `<`, `>=`, `<=`) convert `null` to a number (`0`), so `null >= 0` becomes `0 >= 0 = true` and `null < 1` becomes `0 < 1 = true`. However, the equality operator `==` uses a **different algorithm** — `null` only equals `undefined` (and nothing else), so `null == 0` is `false`. The results appear contradictory but follow two completely separate rules.

</details>

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 3. == vs === Questions

---

### Q10. What will be the output?

```js
console.log([] == false);
console.log([] == 0);
console.log([] == '');
console.log({} == false);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
true
true
true
false
```

### Explanation

For `[] == false`: boolean `false` becomes `0`, then `[]` is coerced via `ToPrimitive` to `''`, then to `0`. So `0 == 0` → `true`. Same chain for `[] == 0`. For `[] == ''`: `[]` becomes `''`, then `'' == ''` → `true`. For `{} == false`: `false` becomes `0`, `{}` via `ToPrimitive` becomes `'[object Object]'`, then `NaN`. `NaN == 0` → `false`.

</details>

---

### Q11. What will be the output?

```js
console.log(null == undefined);
console.log(null == false);
console.log(null == 0);
console.log(undefined == false);
console.log(undefined == 0);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
true
false
false
false
false
```

### Explanation

The Abstract Equality Comparison algorithm has a special rule: `null` and `undefined` are **only loosely equal to each other** and to nothing else (other than themselves). `null == false`, `null == 0`, `undefined == false`, and `undefined == 0` are all `false`. This is the safest and most idiomatic null check: `value == null` catches both `null` AND `undefined`.

</details>

---

### Q12. What will be the output?

```js
console.log('' == 0);
console.log('0' == 0);
console.log('' == '0');
console.log(0 == '0');
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
true
true
false
true
```

### Explanation

When `==` compares a string to a number, the string is converted to a number: `''` → `0` (true), `'0'` → `0` (true). But `'' == '0'` compares two strings directly — they are not equal as strings. This triangle of `'' == 0`, `'0' == 0`, but `'' != '0'` is a classic example of why `==` is non-transitive.

</details>

---

### Q13. What will be the output?

```js
console.log(NaN == NaN);
console.log(NaN === NaN);
console.log(NaN != NaN);
console.log(Number.isNaN(NaN));
console.log(isNaN(NaN));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
false
false
true
true
true
```

### Explanation

`NaN` is the **only value in JavaScript that is not equal to itself** — both `==` and `===` return `false` when comparing `NaN` to anything, including `NaN`. Therefore `NaN != NaN` is `true`. To check for `NaN`, use `Number.isNaN()` (strict, no coercion) or `isNaN()` (coerces first, more permissive).

</details>

---

### Q14. What will be the output?

```js
console.log([1] == 1);
console.log([1, 2] == '1,2');
console.log([[]] == 0);
console.log([0] == false);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
true
true
true
true
```

### Explanation

All four comparisons trigger object-to-primitive conversion. `[1]` → `'1'` → `1`; `[1,2]` → `'1,2'` (via `.toString()`); `[[]]` → `''` → `0`; `[0]` → `'0'` → `0`, and `false` → `0`. These results are why `===` should always be preferred.

</details>

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 4. Truthy and Falsy Questions

---

### Q15. What will be the output?

```js
console.log(Boolean(''));
console.log(Boolean('0'));
console.log(Boolean(0));
console.log(Boolean(-0));
console.log(Boolean(0n));
console.log(Boolean([]));
console.log(Boolean({}));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
false
true
false
false
false
true
true
```

### Explanation

The 8 falsy values are: `false`, `0`, `-0`, `0n`, `''`, `null`, `undefined`, `NaN`. Everything else is truthy. `'0'` is a **non-empty string**, so it is truthy. `[]` and `{}` are object references — they are always truthy regardless of being "empty".

</details>

---

### Q16. What will be the output?

```js
if ([]) {
  console.log('A');
} else {
  console.log('B');
}

if ([] == false) {
  console.log('C');
} else {
  console.log('D');
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
A
C
```

### Explanation

This is a famous JavaScript trap. In the `if` statement, `[]` is **truthy** (object reference → always truthy), so `'A'` is logged. In the `==` comparison, type coercion kicks in: `[]` becomes `''` becomes `0`, and `false` becomes `0`, so `0 == 0` is `true`, and `'C'` is logged. An empty array is truthy in boolean context but loosely equal to `false` through coercion — two completely different mechanisms.

</details>

---

### Q17. What will be the output?

```js
console.log(!!null);
console.log(!!undefined);
console.log(!!NaN);
console.log(!!0);
console.log(!!'');
console.log(!!'hello');
console.log(!![]);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
false
false
false
false
false
true
true
```

### Explanation

`!!` (double negation) is a common idiom to convert any value to its boolean equivalent. The first `!` coerces to boolean and negates; the second `!` negates again. All falsy values produce `false`, all truthy values produce `true`. `[]` is truthy so `!![]` is `true`.

</details>

---

### Q18. What will be the output?

```js
const values = [0, '', null, undefined, NaN, false, -0, 0n];
console.log(values.filter(Boolean).length);

const moreValues = ['0', [], {}, -1, Infinity, ' '];
console.log(moreValues.filter(Boolean).length);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
0
6
```

### Explanation

`filter(Boolean)` removes all falsy values. The first array contains **all 8 falsy values** — nothing passes the filter, so length is `0`. The second array contains `'0'` (non-empty string), `[]` (object), `{}` (object), `-1` (non-zero number), `Infinity` (truthy), and `' '` (a space, which is a non-empty string) — all 6 are truthy, so all pass.

</details>

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 5. Short-Circuit Questions

---

### Q19. What will be the output?

```js
console.log(0 || '' || null || undefined || 'found' || 'second');
console.log(1 && 2 && 3);
console.log(1 && 0 && 3);
console.log(null && undefined);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
'found'
3
0
null
```

### Explanation

`||` returns the first truthy value, or the last value if all are falsy. `0`, `''`, `null`, and `undefined` are all falsy, so `'found'` is returned (the second truthy value stops evaluation). `&&` returns the first falsy value, or the last value if all are truthy. In `1 && 0 && 3`: `0` is falsy so it is returned and `3` is never evaluated. In `null && undefined`: `null` is falsy so it is returned immediately.

</details>

---

### Q20. What will be the output?

```js
let a = 0;
let b = 0;

const result1 = a++ || ++b;
console.log(a, b, result1);

const result2 = a++ && ++b;
console.log(a, b, result2);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
1 1 1
2 2 2
```

### Explanation

First expression: `a++` evaluates to `0` (post-increment, returns before incrementing), so `a` becomes `1`. `0` is falsy, so `||` evaluates the right side: `++b` increments `b` to `1` and returns `1`. `result1 = 1`. Second expression: `a++` evaluates to `1` (post-increment), so `a` becomes `2`. `1` is truthy, so `&&` evaluates the right side: `++b` increments `b` to `2` and returns `2`. `result2 = 2`.

</details>

---

### Q21. What will be the output?

```js
console.log(0 ?? 'fallback');
console.log(null ?? 'fallback');
console.log(undefined ?? 'fallback');
console.log(false ?? 'fallback');
console.log('' ?? 'fallback');
console.log(NaN ?? 'fallback');
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
0
'fallback'
'fallback'
false
''
NaN
```

### Explanation

`??` (nullish coalescing) returns the right side **only when the left is `null` or `undefined`**. It does NOT trigger for `0`, `false`, `''`, or `NaN` — these are all valid non-null values. This is the critical difference from `||`, which would return `'fallback'` for all of these. Use `??` when `0` or `''` are meaningful values.

</details>

---

### Q22. What will be the output?

```js
const user = null;
const name = user?.profile?.name ?? 'Anonymous';
console.log(name);

const count = 0;
const display1 = count || 'No items';
const display2 = count ?? 'No items';
console.log(display1);
console.log(display2);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
'Anonymous'
'No items'
0
```

### Explanation

`user?.profile?.name` short-circuits to `undefined` because `user` is `null`. Then `undefined ?? 'Anonymous'` returns `'Anonymous'`. For `count = 0`: `count || 'No items'` returns `'No items'` because `0` is falsy. But `count ?? 'No items'` returns `0` because `??` only triggers on `null`/`undefined`, and `0` is neither. This demonstrates why `??` is safer for numeric values.

</details>

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 6. NaN and Number Questions

---

### Q23. What will be the output?

```js
console.log(isNaN('hello'));
console.log(isNaN(''));
console.log(isNaN(undefined));
console.log(isNaN(null));
console.log(Number.isNaN('hello'));
console.log(Number.isNaN(undefined));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
true
false
true
false
false
false
```

### Explanation

`isNaN` coerces its argument to a number first, then checks. `Number('')` = `0`, so `isNaN('')` is `false`. `Number(undefined)` = `NaN`, so `isNaN(undefined)` is `true`. `Number(null)` = `0`, so `isNaN(null)` is `false`. `Number.isNaN` does **no coercion** — it only returns `true` for the actual `NaN` value. `'hello'` and `undefined` are not `NaN` the value, so both return `false`.

</details>

---

### Q24. What will be the output?

```js
console.log(parseInt('-4.9'));
console.log(Math.floor(-4.9));
console.log(Math.trunc(-4.9));
console.log(Math.ceil(-4.9));
console.log(Math.round(-4.5));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
-4
-5
-4
-4
-4
```

### Explanation

For negative numbers these functions diverge: `parseInt` removes the decimal (toward zero) → `-4`. `Math.floor` always rounds **toward -Infinity** → `-5`. `Math.trunc` removes the decimal (toward zero) → `-4`. `Math.ceil` rounds toward `+Infinity` → `-4` (since `-4` is greater than `-4.9`). `Math.round(-4.5)` rounds toward `+Infinity` for `.5` ties → `-4` (not `-5`).

</details>

---

### Q25. What will be the output?

```js
console.log(0.1 + 0.2);
console.log(0.1 + 0.2 === 0.3);
console.log(1 / 0);
console.log(-1 / 0);
console.log(0 / 0);
console.log(typeof (1 / 0));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
0.30000000000000004
false
Infinity
-Infinity
NaN
'number'
```

### Explanation

`0.1 + 0.2` results in `0.30000000000000004` due to IEEE 754 binary floating-point representation — these fractions cannot be represented exactly in binary. Division by `0` for a positive number gives `Infinity`, negative gives `-Infinity`, and `0/0` gives `NaN`. Crucially, `typeof Infinity` and `typeof NaN` are both `'number'` — they are special numeric values, not a separate type.

</details>

---

### Q26. What will be the output?

```js
console.log(parseInt('10', 2));
console.log(parseInt('10', 8));
console.log(parseInt('10', 16));
console.log(parseInt('ff', 16));
console.log(parseInt('08'));
console.log(parseInt('0x1A'));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
2
8
16
255
8
26
```

### Explanation

The second argument to `parseInt` is the **radix** (base). `'10'` in base 2 = 2, in base 8 = 8, in base 16 = 16. `'ff'` in base 16 = 255. `parseInt('08')` defaults to base 10, so the result is `8`. `parseInt('0x1A')` recognizes the `0x` prefix and automatically uses base 16: `1*16 + 10 = 26`.

</details>

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 7. String Questions

---

### Q27. What will be the output?

```js
console.log('hello'.slice(1, 3));
console.log('hello'.slice(-3));
console.log('hello'.slice(-3, -1));
console.log('hello'.substring(1, 3));
console.log('hello'.substring(-3));
console.log('hello'.substring(3, 1));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
'el'
'llo'
'll'
'el'
'hello'
'el'
```

### Explanation

`slice` supports negative indices (counting from end) and never swaps arguments. `slice(-3)` means start at index `5 - 3 = 2` → `'llo'`. `slice(-3, -1)` = indices `2` to `4` (exclusive) → `'ll'`. `substring` treats negative values as `0` (so `substring(-3)` = `substring(0)` = full string), and it **swaps arguments** when `start > end` (so `substring(3, 1)` = `substring(1, 3)` = `'el'`).

</details>

---

### Q28. What will be the output?

```js
const str = 'hello world hello';
console.log(str.replace('hello', 'hi'));
console.log(str.replaceAll('hello', 'hi'));
console.log(str.replace(/hello/g, 'hi'));
console.log('  hello  '.trim());
console.log('  hello  '.trimStart());
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
'hi world hello'
'hi world hi'
'hi world hi'
'hello'
'hello  '
```

### Explanation

`replace` with a string argument only replaces the **first occurrence**. `replaceAll` and `replace` with a global regex (`/g` flag) replace all occurrences. `trim()` removes whitespace from both ends. `trimStart()` removes from the beginning only, so trailing spaces remain. Strings are immutable — all these methods return new strings.

</details>

---

### Q29. What will be the output?

```js
let str = 'hello';
str[0] = 'H';
console.log(str);
console.log(str.length);

console.log('5'.padStart(5, '0'));
console.log('hello'.padStart(3, '*'));
console.log('ha'.repeat(0));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
'hello'
5
'00005'
'hello'
''
```

### Explanation

Strings are **immutable** in JavaScript — assigning to an index is silently ignored in non-strict mode (or throws in strict mode), leaving the string unchanged. `padStart(5, '0')` adds `'0'`s to reach length 5. If the string is already longer or equal to the target length (`'hello'.padStart(3, '*')`), it is returned unchanged. `'ha'.repeat(0)` returns an empty string.

</details>

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 8. Spread and Destructuring Questions

---

### Q30. What will be the output?

```js
const original = { a: 1, b: { c: 2 } };
const copy = { ...original };

copy.a = 99;
copy.b.c = 99;

console.log(original.a);
console.log(original.b.c);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
1
99
```

### Explanation

Spread creates a **shallow copy** — only the top-level properties are copied. `copy.a = 99` changes the copy's own `a` property without affecting the original (primitives are copied by value). But `copy.b` is the **same object reference** as `original.b` — there is no deep copy. Mutating `copy.b.c` therefore mutates `original.b.c` as well.

</details>

---

### Q31. What will be the output?

```js
const [a = 1, b = 2, c = 3] = [10, undefined, null];
console.log(a);
console.log(b);
console.log(c);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
10
2
null
```

### Explanation

Default values in destructuring are **only triggered when the value is `undefined`**, not for any other falsy value. `a = 10` (explicit value provided). `b = 2` (the value is `undefined`, so the default `2` is used). `c = null` (the value is `null`, which is NOT `undefined`, so the default `3` is NOT used — `null` is returned as-is).

</details>

---

### Q32. What will be the output?

```js
const { a: x, b: y = 10 } = { a: 1 };
console.log(x);
console.log(y);
// console.log(a);  // What would this do?
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
1
10
```

### Explanation

In object destructuring, `{ a: x }` means "read property `a`, assign it to a variable named `x`". The variable created is `x`, NOT `a`. Therefore `console.log(a)` would throw `ReferenceError: a is not defined`. The `b` property does not exist on the source object, so the default `10` is used and assigned to `y`.

</details>

---

### Q33. What will be the output?

```js
const obj1 = { a: 1, b: 2 };
const obj2 = { b: 3, c: 4 };
const merged = { ...obj1, ...obj2 };
console.log(merged);

const { a, ...rest } = merged;
console.log(a);
console.log(rest);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
{ a: 1, b: 3, c: 4 }
1
{ b: 3, c: 4 }
```

### Explanation

When spreading objects with duplicate keys, **later spreads overwrite earlier ones**. `obj2.b = 3` overwrites `obj1.b = 2`. The rest operator in object destructuring collects all remaining own enumerable properties not explicitly destructured. `a` is extracted separately, and `rest` gets `{ b: 3, c: 4 }`.

</details>

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 9. Operator Precedence Questions

---

### Q34. What will be the output?

```js
console.log(1 + '2' + 3);
console.log(1 + 2 + '3');
console.log(+'3' + +'4');
console.log('5' - 3);
console.log('5' * '3');
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
'123'
'33'
7
2
15
```

### Explanation

The `+` operator is left-associative. `1 + '2'` → `'12'` (number coerced to string), then `'12' + 3` → `'123'`. In `1 + 2 + '3'`: `1 + 2` = `3` (both numbers), then `3 + '3'` = `'33'`. `+'3'` uses the **unary plus**, which converts to number: `3 + 4 = 7`. `-`, `*`, `/` always coerce both operands to numbers (unlike `+` which has the string case), so `'5' - 3 = 2` and `'5' * '3' = 15`.

</details>

---

### Q35. What will be the output?

```js
console.log(3 > 2 > 1);
console.log(1 < 2 < 3);
console.log(3 > 2 > 0);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
false
true
true
```

### Explanation

Comparison operators are **left-associative**. `3 > 2 > 1`: first `3 > 2` = `true`, then `true > 1` = `1 > 1` = `false`. `1 < 2 < 3`: first `1 < 2` = `true`, then `true < 3` = `1 < 3` = `true`. `3 > 2 > 0`: `true > 0` = `1 > 0` = `true`. Chaining comparison operators like `a < b < c` does NOT work as a mathematical range check — it always reduces the left side to a boolean (0 or 1) first.

</details>

---

### Q36. What will be the output?

```js
console.log(2 ** 3 ** 2);
console.log((2 ** 3) ** 2);
console.log(2 ** 10);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
512
64
1024
```

### Explanation

The exponentiation operator `**` is **right-associative** — it evaluates right to left. `2 ** 3 ** 2` = `2 ** (3 ** 2)` = `2 ** 9` = `512`. With explicit left-grouping: `(2 ** 3) ** 2` = `8 ** 2` = `64`. This is different from most other binary operators which are left-associative. `2 ** 10` = `1024`.

</details>

---

### Q37. What will be the output?

```js
let x = (5, 10, 15);
console.log(x);

function test() {
  return 1, 2, 3;
}
console.log(test());

let y = 1;
let z = (y++, y++, y);
console.log(y);
console.log(z);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
15
3
3
3
```

### Explanation

The **comma operator** evaluates each operand left to right and returns the value of the last one. `(5, 10, 15)` returns `15`. In the `return` statement, `return 1, 2, 3` returns `3` (last value). In `(y++, y++, y)`: `y` starts at `1`. First `y++` returns `1`, increments to `2`. Second `y++` returns `2`, increments to `3`. Then `y` is `3` (already a pure read, no increment). `z = 3`.

</details>

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 10. Advanced Edge Cases

---

### Q38. What will be the output?

```js
console.log([] + []);
console.log([] + {});
console.log({} + []);
console.log(+[]);
console.log(+{});
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
''
'[object Object]'
'[object Object]'
0
NaN
```

### Explanation

`[] + []`: both arrays convert to `''` (empty string), so `'' + ''` = `''`. `[] + {}`: `[]` → `''`, `{}` → `'[object Object]'`, so `'' + '[object Object]'` = `'[object Object]'`. `{} + []` as an **expression** (e.g., in a console, right-hand of assignment): `{}` → `'[object Object]'`, `[]` → `''`, result is `'[object Object]'`. Note: as a **statement** (at the start of a line), `{}` is treated as an empty block and `+[]` = `0`. `+[]`: unary plus on `[]` → `+''` = `0`. `+{}`: unary plus on `{}` → `+'[object Object]'` = `NaN`.

</details>

---

### Q39. What will be the output?

```js
console.log(Number(null));
console.log(Number(undefined));
console.log(Number(''));
console.log(Number('  '));
console.log(Number(true));
console.log(Number(false));
console.log(Number([]));
console.log(Number({}));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
0
NaN
0
0
1
0
0
NaN
```

### Explanation

`Number(null)` = `0`. `Number(undefined)` = `NaN`. Empty string `''` and whitespace-only string `'  '` both convert to `0` (whitespace is trimmed first). `true` = `1`, `false` = `0`. `Number([])`: array → `''` → `0`. `Number({})`: object → `'[object Object]'` → `NaN`. These coercion rules underlie all of JavaScript's implicit type conversions.

</details>

---

### Q40. What will be the output?

```js
const obj = {
  value: 42,
  [Symbol.toPrimitive](hint) {
    if (hint === 'number') return 10;
    if (hint === 'string') return 'ten';
    return true;
  }
};

console.log(+obj);
console.log(`${obj}`);
console.log(obj + '');
console.log(obj == true);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
10
'ten'
'true'
true
```

### Explanation

When `Symbol.toPrimitive` is defined, JavaScript calls it with a `hint` argument: `'number'` for numeric contexts (unary `+`), `'string'` for template literals, and `'default'` for `+` with a string (or `==` comparison). `+obj` uses `'number'` hint → `10`. `${obj}` uses `'string'` hint → `'ten'`. `obj + ''` uses `'default'` hint → `true` → `'true'`. `obj == true`: `obj` with `'default'` hint returns `true`, and `true == true` → `true`.

</details>

---

### Q41. What will be the output?

```js
function checkType(val) {
  if (val == null) {
    return 'nullish';
  }
  return typeof val;
}

console.log(checkType(null));
console.log(checkType(undefined));
console.log(checkType(0));
console.log(checkType(''));
console.log(checkType(false));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
'nullish'
'nullish'
'number'
'string'
'boolean'
```

### Explanation

`val == null` is a practical idiomatic pattern that catches **both `null` and `undefined`** in one check. This works because of the special equality rule: `null == undefined` is `true`, and `null` and `undefined` are not `==` to any other value. `0`, `''`, and `false` are falsy but NOT `== null`, so they fall through to the `typeof` branch.

</details>

---

### Q42. What will be the output?

```js
const arr = [1, 2, 3];
const [first, ...rest] = arr;
rest.push(4);

console.log(arr);
console.log(rest);
console.log(first);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
[1, 2, 3]
[2, 3, 4]
1
```

### Explanation

The rest element in array destructuring (`...rest`) collects remaining items into a **new array** — it is not a reference to the tail of the original array. Pushing `4` into `rest` does not affect `arr`. This is different from a reference copy. `first = 1`, `rest = [2, 3]` (initially), then becomes `[2, 3, 4]` after the push. `arr` remains `[1, 2, 3]`.

</details>

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## Final Tips

- **`typeof null === 'object'`** — memorize this as a historical bug; the safe null check is `val === null`.
- **`typeof` on undeclared variables** does not throw — it returns `'undefined'`. This is the only safe use.
- **`NaN !== NaN`** — always use `Number.isNaN()` (strict) rather than `isNaN()` (coerces first).
- **`isNaN('')` is `false`** — `''` coerces to `0`, which is not NaN. Use `Number.isNaN` to avoid surprises.
- **`null == undefined` is `true`** but `null` is not equal to `0`, `false`, or `''` via `==`.
- **`null >= 0` is `true`** but `null == 0` is `false` — relational and equality operators use different coercion rules.
- **`[]` and `{}` are always truthy**, but `[] == false` is `true` via coercion — different mechanisms entirely.
- **`??` vs `||`** — use `??` when `0`, `''`, or `false` are valid values you want to preserve.
- **Default params only trigger on `undefined`**, not `null`. This is a common source of subtle bugs.
- **Spread is shallow** — nested objects are still shared references; always deep clone when mutating nested data.
- **Destructuring rename syntax**: `const { a: b } = obj` creates `b`, not `a`. Accessing `a` will throw.
- **`3 > 2 > 1` is `false`** — chained comparisons don't work as expected; the result of `3 > 2` (`true` = `1`) is compared to `1`.
- **`2 ** 3 ** 2` is `512`** — exponentiation is right-associative: `2 ** (3 ** 2)`.
- **`'' + null`** is `'null'` and **`'' + undefined`** is `'undefined'` — in string concatenation, `null` and `undefined` become their string representations.
- **Always use `===`** in production code — use `==` only when you intentionally want the `null == undefined` shortcut.

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
