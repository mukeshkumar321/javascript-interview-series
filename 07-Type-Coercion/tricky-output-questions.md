# Type Coercion — Tricky Output Questions

## Table of Contents

1. [String Coercion Questions](#1-string-coercion-questions)
2. [Number Coercion Questions](#2-number-coercion-questions)
3. [Boolean Coercion Questions](#3-boolean-coercion-questions)
4. [Abstract Equality (==) Questions](#4-abstract-equality--questions)
5. [+ Operator Questions](#5--operator-questions)
6. [Object to Primitive Questions](#6-object-to-primitive-questions)
7. [Mixed Operator Questions](#7-mixed-operator-questions)
8. [Advanced Coercion Questions](#8-advanced-coercion-questions)

---

## 1. String Coercion Questions

---

### Q1. What will be the output?

```js
console.log(String(null));
console.log(String(undefined));
console.log(String(true));
console.log(String(false));
console.log(String(0));
console.log(String(-0));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
null
undefined
true
false
0
0
```

### Explanation

`String()` converts each value directly to its string representation. The notable edge case is `String(-0)` — even though `-0` is a distinct value in JavaScript, `String(-0)` returns `"0"` (the negative sign is lost). Compare with `JSON.stringify(-0)` which also returns `"0"`.

</details>

---

### Q2. What will be the output?

```js
console.log(`${null}`);
console.log(`${undefined}`);
console.log(`${[1, 2, 3]}`);
console.log(`${{ a: 1 }}`);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
null
undefined
1,2,3
[object Object]
```

### Explanation

Template literals use a "string" hint for `ToPrimitive`. For arrays, `toString()` joins elements with commas: `[1,2,3].toString()` → `"1,2,3"`. For plain objects, `toString()` returns `"[object Object]"`. `null` and `undefined` are converted to their string names (`"null"`, `"undefined"`).

</details>

---

### Q3. What will be the output?

```js
console.log(1 + 2 + "3");
console.log("1" + 2 + 3);
console.log("3" + (1 + 2));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
33
123
33
```

### Explanation

`+` is left-associative, so it evaluates left to right:
- `1 + 2 + "3"` → `(1 + 2) + "3"` → `3 + "3"` → `"33"` (number + string = string concat)
- `"1" + 2 + 3` → `("1" + 2) + 3` → `"12" + 3` → `"123"` (once a string is in the chain, all subsequent `+` becomes concatenation)
- `"3" + (1 + 2)` → `"3" + 3` → `"33"` (parentheses force numeric add first)

</details>

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 2. Number Coercion Questions

---

### Q1. What will be the output?

```js
console.log(Number(""));
console.log(Number("  "));
console.log(Number("  123  "));
console.log(Number("123abc"));
console.log(Number(null));
console.log(Number(undefined));
console.log(Number(true));
console.log(Number(false));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
0
0
123
NaN
0
NaN
1
0
```

### Explanation

- `""` and `"  "` (whitespace-only): `Number()` trims whitespace first, leaving `""` → `0`.
- `"  123  "`: trimmed to `"123"` → `123`.
- `"123abc"`: contains non-numeric characters after trimming → `NaN`. (Unlike `parseInt` which stops at the first non-digit.)
- `null` → `0`, `undefined` → `NaN` (the asymmetry is a common interview trap).
- `true` → `1`, `false` → `0`.

</details>

---

### Q2. What will be the output?

```js
console.log(+[]);
console.log(+[3]);
console.log(+[1, 2]);
console.log(+{});
console.log(+null);
console.log(+undefined);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
0
3
NaN
NaN
0
NaN
```

### Explanation

The unary `+` calls `Number()` on the operand:
- `+[]`: `[].toString()` → `""` → `Number("")` → `0`.
- `+[3]`: `[3].toString()` → `"3"` → `Number("3")` → `3`.
- `+[1,2]`: `[1,2].toString()` → `"1,2"` → `Number("1,2")` → `NaN`.
- `+{}`: `{}.toString()` → `"[object Object]"` → `NaN`.
- `+null` → `0`, `+undefined` → `NaN`.

</details>

---

### Q3. What will be the output?

```js
console.log(parseInt("10.5"));
console.log(parseFloat("10.5"));
console.log(parseInt("10abc"));
console.log(parseInt("abc10"));
console.log(parseInt("0x10"));
console.log(parseInt("010"));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
10
10.5
10
NaN
16
10
```

### Explanation

- `parseInt("10.5")`: stops at the decimal point → `10`.
- `parseInt("10abc")`: stops at first non-digit `a` → `10`.
- `parseInt("abc10")`: first character is not a digit → `NaN`.
- `parseInt("0x10")`: recognizes hex prefix `0x` → `16`.
- `parseInt("010")`: in modern JS (ES5+) without a radix, this is decimal `10`. Always pass the radix (`parseInt("010", 10)`) to be explicit.

</details>

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 3. Boolean Coercion Questions

---

### Q1. What will be the output?

```js
console.log(Boolean("0"));
console.log(Boolean("false"));
console.log(Boolean([]));
console.log(Boolean({}));
console.log(Boolean(0));
console.log(Boolean(""));
console.log(Boolean(new Boolean(false)));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
true
true
true
true
false
false
true
```

### Explanation

The six falsy values are: `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, `NaN`. **Everything else is truthy.** Common traps:
- `"0"` is a non-empty string → `true`.
- `"false"` is a non-empty string → `true`.
- `[]` and `{}` are object references → `true` (even if "empty").
- `new Boolean(false)` is an **object** wrapping `false` → the object reference is truthy even though the wrapped value is falsy.

</details>

---

### Q2. What will be the output?

```js
if ("false") {
  console.log("truthy");
} else {
  console.log("falsy");
}

if (0) {
  console.log("truthy");
} else {
  console.log("falsy");
}

if ([]) {
  console.log("truthy");
} else {
  console.log("falsy");
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
truthy
falsy
truthy
```

### Explanation

- `"false"` is a non-empty string → truthy.
- `0` is one of the six falsy values → falsy.
- `[]` is an object reference → truthy, even though `[] == false` is `true` via `==`. The boolean context and `==` use different coercion rules.

</details>

---

### Q3. What will be the output?

```js
console.log(!!"");
console.log(!!"0");
console.log(!!null);
console.log(!!undefined);
console.log(!!NaN);
console.log(!!0);
console.log(!![]);
console.log(!!{});
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
false
true
true
```

### Explanation

Double negation (`!!`) is the idiomatic way to convert any value to its boolean equivalent. The first `!` converts to boolean and negates, the second `!` negates back. The result is identical to `Boolean(value)`.

</details>

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 4. Abstract Equality (==) Questions

---

### Q1. What will be the output?

```js
console.log(null == undefined);
console.log(null == 0);
console.log(null == false);
console.log(null == "");
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
false
```

### Explanation

The `==` spec says: if either operand is `null` or `undefined`, the result is `true` only if the **other operand is also** `null` or `undefined`. Any other comparison involving `null`/`undefined` returns `false` — even with falsy values like `0`, `false`, or `""`. This is the most important `==` rule to memorize.

</details>

---

### Q2. What will be the output?

```js
console.log("" == false);
console.log("0" == false);
console.log("" == 0);
console.log("0" == 0);
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

Step-by-step for each:
- `"" == false`: `false` → `0`, `""` → `0`. `0 == 0` → `true`.
- `"0" == false`: `false` → `0`, `"0"` → `0`. `0 == 0` → `true`.
- `"" == 0`: `""` → `0`. `0 == 0` → `true`.
- `"0" == 0`: `"0"` → `0`. `0 == 0` → `true`.

Notice the transitivity violation: `"" == false` (true), `"0" == false` (true), but `"" == "0"` is `false` (same type, different strings). `==` is NOT transitive.

</details>

---

### Q3. What will be the output?

```js
console.log([] == false);
console.log([] == 0);
console.log([] == "");
console.log([] == ![]);
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

- `[] == false`: `false` → `0`. `[]` is an object → `ToPrimitive([])` → `[].toString()` → `""` → `Number("")` → `0`. `0 == 0` → `true`.
- `[] == 0`: `ToPrimitive([])` → `""` → `0`. `0 == 0` → `true`.
- `[] == ""`: `ToPrimitive([])` → `""`. Same type and same value → `true`.
- `[] == ![]`: `![]` evaluates first — `[]` is truthy, so `![]` = `false`. Now `[] == false` → `true` (same as first case).

`[] == ![]` is one of the most famous JavaScript quirks — both sides involve `[]` yet they are "equal" via `==`.

</details>

---

### Q4. What will be the output?

```js
console.log(NaN == NaN);
console.log(NaN === NaN);
console.log(NaN != NaN);
console.log(Number.isNaN(NaN));
console.log(isNaN("hello"));
console.log(Number.isNaN("hello"));
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
false
```

### Explanation

`NaN` is the only JavaScript value that is **not equal to itself** — both `==` and `===` return `false`. To check for `NaN`, use `Number.isNaN()` which does not coerce its argument. `isNaN("hello")` returns `true` because it converts `"hello"` to `Number("hello")` = `NaN` first — a source of bugs. `Number.isNaN("hello")` returns `false` because `"hello"` is literally not `NaN`.

</details>

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 5. + Operator Questions

---

### Q1. What will be the output?

```js
console.log([] + []);
console.log([] + {});
console.log({} + []);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt

[object Object]
[object Object]
```

### Explanation

- `[] + []`: `ToPrimitive([])` → `""`. `"" + ""` → `""` (empty string — the output line appears blank).
- `[] + {}`: `ToPrimitive([])` → `""`. `ToPrimitive({})` → `"[object Object]"`. `"" + "[object Object]"` → `"[object Object]"`.
- `{} + []` inside `console.log()` (expression context): `ToPrimitive({})` → `"[object Object]"`. `"[object Object]" + ""` → `"[object Object]"`.

Note: if `{} + []` is a **statement** (typed directly in the browser console), `{}` is parsed as an empty block and the expression becomes `+[]` = `0`.

</details>

---

### Q2. What will be the output?

```js
console.log(true + true);
console.log(true + false);
console.log(false + false);
console.log(true + 1);
console.log(false + 1);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
2
1
0
2
1
```

### Explanation

Neither operand is a string, so `+` performs numeric addition. Booleans are coerced to numbers: `true` → `1`, `false` → `0`. The results follow from simple arithmetic.

</details>

---

### Q3. What will be the output?

```js
console.log(null + 1);
console.log(undefined + 1);
console.log(null + null);
console.log(undefined + undefined);
console.log(null + undefined);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
1
NaN
0
NaN
NaN
```

### Explanation

- `null + 1`: `null` → `0`. `0 + 1` → `1`.
- `undefined + 1`: `undefined` → `NaN`. `NaN + 1` → `NaN`.
- `null + null`: `0 + 0` → `0`.
- `undefined + undefined`: `NaN + NaN` → `NaN`.
- `null + undefined`: `0 + NaN` → `NaN`.

`null` coerces to `0` in arithmetic but `undefined` coerces to `NaN` — another asymmetry to remember.

</details>

---

### Q4. What will be the output?

```js
console.log(+"");
console.log(+" ");
console.log(+"0");
console.log(+"1e2");
console.log(+"Infinity");
console.log(+"0x10");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
0
0
0
100
Infinity
16
```

### Explanation

The unary `+` applies `Number()` conversion:
- `+""` and `+" "` → empty/whitespace strings → `0`.
- `+"0"` → `0`.
- `+"1e2"` → scientific notation is valid → `100`.
- `+"Infinity"` → JavaScript recognizes the literal string `"Infinity"` → `Infinity`.
- `+"0x10"` → hex literal string is recognized → `16`.

</details>

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 6. Object to Primitive Questions

---

### Q1. What will be the output?

```js
const obj = {
  valueOf() { return 10; },
  toString() { return "hello"; }
};

console.log(obj + "");
console.log(`${obj}`);
console.log(obj * 2);
console.log(obj > 5);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
10
hello
20
true
```

### Explanation

- `obj + ""`: hint is `"default"` (same as `"number"` for plain objects) → `valueOf()` → `10` → `10 + ""` → `"10"`.
- `` `${obj}` ``: hint is `"string"` → `toString()` → `"hello"`.
- `obj * 2`: hint is `"number"` → `valueOf()` → `10` → `10 * 2` → `20`.
- `obj > 5`: hint is `"number"` → `valueOf()` → `10` → `10 > 5` → `true`.

</details>

---

### Q2. What will be the output?

```js
const obj = {
  [Symbol.toPrimitive](hint) {
    console.log("hint:", hint);
    if (hint === "number") return 42;
    if (hint === "string") return "forty-two";
    return true;
  }
};

console.log(+obj);
console.log(`${obj}`);
console.log(obj + "");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
hint: number
42
hint: string
forty-two
hint: default
true
```

### Explanation

`Symbol.toPrimitive` receives the hint as a string argument and takes priority over `valueOf`/`toString`. The unary `+` uses `"number"` hint. Template literals use `"string"` hint. The `+` operator with a string operand uses `"default"` hint. The `"default"` handler returns `true` (a boolean), which is then concatenated with `""` → `"true"`.

</details>

---

### Q3. What will be the output?

```js
console.log([] == ![]);
console.log({} == !{});
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
true
false
```

### Explanation

- `[] == ![]`: `![]` evaluates first. `[]` is truthy → `![]` = `false`. So `[] == false`. Now: `false` → `0`, `ToPrimitive([])` → `""` → `0`. `0 == 0` → `true`.
- `{} == !{}`: `!{}` evaluates first. `{}` is truthy → `!{}` = `false`. So `{} == false`. Now: `false` → `0`, `ToPrimitive({})` → `"[object Object]"` → `Number("[object Object]")` → `NaN`. `NaN == 0` → `false`.

Same pattern, different result — because `ToPrimitive({})` produces a non-numeric string, which coerces to `NaN`, and `NaN` is never equal to anything.

</details>

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 7. Mixed Operator Questions

---

### Q1. What will be the output?

```js
console.log(1 - "1");
console.log(1 * "2");
console.log("6" / "2");
console.log("7" % "3");
console.log("abc" - 1);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
0
2
3
1
NaN
```

### Explanation

`-`, `*`, `/`, `%` always coerce both operands to numbers — there is no string-concatenation behavior for these operators. `"6"` → `6`, `"2"` → `2`, `"1"` → `1`, `"abc"` → `NaN`. Any arithmetic with `NaN` produces `NaN`.

</details>

---

### Q2. What will be the output?

```js
console.log("5" + 3);
console.log("5" - 3);
console.log("5" * "3");
console.log("5" > 3);
console.log("10" > "9");
console.log("10" > 9);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
53
2
15
true
false
true
```

### Explanation

- `"5" + 3`: string on the left → concatenation → `"53"`.
- `"5" - 3`: `-` forces numeric → `5 - 3` = `2`.
- `"5" * "3"`: both strings → both converted to numbers → `15`.
- `"5" > 3`: one side is a number → numeric comparison → `5 > 3` → `true`.
- `"10" > "9"`: **both** sides are strings → lexicographic comparison → `"1"` < `"9"` → `false`. This is a classic sorting bug.
- `"10" > 9`: number on the right → numeric comparison → `10 > 9` → `true`.

</details>

---

### Q3. What will be the output?

```js
console.log(null > 0);
console.log(null < 0);
console.log(null == 0);
console.log(null >= 0);
console.log(null <= 0);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
false
false
false
true
true
```

### Explanation

This is one of JavaScript's most confusing coercion quirks:
- `null > 0`: `null` → `0`, `0 > 0` → `false`.
- `null < 0`: `null` → `0`, `0 < 0` → `false`.
- `null == 0`: **special `==` rule** — `null` only equals `null`/`undefined` → `false`.
- `null >= 0`: `null` → `0`, `0 >= 0` → `true`.
- `null <= 0`: `null` → `0`, `0 <= 0` → `true`.

`>=` and `<=` use numeric coercion but `==` uses its own special algorithm. So `null >= 0` is `true` while `null == 0` is `false` — mathematically inconsistent.

</details>

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 8. Advanced Coercion Questions

---

### Q1. What will be the output?

```js
console.log(0 == "0");
console.log(0 == "");
console.log("0" == "");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
true
true
false
```

### Explanation

This demonstrates that `==` is **not transitive**:
- `0 == "0"`: `"0"` → `0`. `0 == 0` → `true`.
- `0 == ""`: `""` → `0`. `0 == 0` → `true`.
- `"0" == ""`: both are strings → compare as strings → `"0"` ≠ `""` → `false`.

So `0 == "0"` and `0 == ""` are both `true`, but `"0" == ""` is `false`. Transitivity (`a == b` and `b == c` implies `a == c`) does not hold for `==`.

</details>

---

### Q2. What will be the output?

```js
const a = [];
const b = [];

console.log(a == b);
console.log(a === b);
console.log(a == false);
console.log(a == 0);
console.log(a == "");
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

- `a == b` and `a === b`: Both operands are objects. Object comparison (both `==` and `===`) is by **reference**, not value. `a` and `b` are different array instances → `false`.
- `a == false`: One side is an object, the other is a primitive. `ToPrimitive(a)` → `""`. `false` → `0`. `""` → `0`. `0 == 0` → `true`.
- `a == 0`: `ToPrimitive([])` → `""` → `0`. `0 == 0` → `true`.
- `a == ""`: `ToPrimitive([])` → `""`. Both strings, same value → `true`.

An empty array is truthy in boolean context (`if (a)` → true) but `a == false` is also true — seemingly contradictory until you understand the different coercion rules used.

</details>

---

### Q3. What will be the output?

```js
console.log(typeof NaN);
console.log(typeof null);
console.log(typeof undefined);
console.log(typeof []);
console.log(typeof {});
console.log(typeof function(){});
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
number
object
undefined
object
object
function
```

### Explanation

Classic `typeof` quirks:
- `typeof NaN` → `"number"` (NaN is a numeric value meaning "invalid number").
- `typeof null` → `"object"` — a long-standing JavaScript bug from 1995 that was never fixed for backward-compatibility reasons.
- `typeof []` → `"object"` — arrays are objects in JavaScript. Use `Array.isArray()` to check for arrays.
- `typeof function(){}` → `"function"` — functions get their own `typeof` result even though they are technically objects.

</details>

---

### Q4. What will be the output?

```js
console.log([] + []);
console.log(+[]);
console.log(+{});
console.log([] - []);
console.log([3] - [1]);
console.log([1, 2] - [1]);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt

0
NaN
0
2
NaN
```

### Explanation

- `[] + []`: `""` + `""` = `""` (empty string — line appears blank in output).
- `+[]`: `+""` = `0`.
- `+{}`: `+"[object Object]"` = `NaN`.
- `[] - []`: `ToNumber([])` = `ToNumber("")` = `0`. `0 - 0` = `0`.
- `[3] - [1]`: `ToNumber([3])` = `ToNumber("3")` = `3`. `ToNumber([1])` = `1`. `3 - 1` = `2`.
- `[1,2] - [1]`: `ToNumber([1,2])` = `ToNumber("1,2")` = `NaN`. `NaN - 1` = `NaN`.

The `-` operator always coerces to numbers — arrays convert via `toString()` first, which joins elements with commas.

</details>

---

### Q5. What will be the output?

```js
console.log(false == "false");
console.log(false == "0");
console.log(false == 0);
console.log(false == "");
console.log(false == null);
console.log(false == undefined);
console.log(false == NaN);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
false
true
true
true
false
false
false
```

### Explanation

When one operand is `false` (a boolean), the `==` algorithm converts it to a number first (`false` → `0`), then restarts:
- `false == "false"`: `false` → `0`. Now `0 == "false"` → `"false"` → `NaN`. `0 == NaN` → `false`.
- `false == "0"`: `false` → `0`. `"0"` → `0`. `0 == 0` → `true`.
- `false == 0`: `false` → `0`. `0 == 0` → `true`.
- `false == ""`: `false` → `0`. `""` → `0`. `0 == 0` → `true`.
- `false == null`: `false` → `0`. `0 == null` — but now null uses the special rule: null only equals null/undefined → `false`.
- `false == undefined`: same as above → `false`.
- `false == NaN`: `false` → `0`. `0 == NaN` → `false` (NaN is never equal to anything).

</details>

---

## Final Tips

- Always know the **six falsy values**: `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, `NaN`. Everything else is truthy.
- `null == undefined` is `true` but `null` does **not** equal anything else via `==`.
- The `+` operator concatenates if **either** operand is a string; `-`, `*`, `/`, `%` always force numeric conversion.
- `==` is **not transitive** — `0 == ""` and `0 == "0"` are both `true`, but `"" == "0"` is `false`.
- `[] == false` is `true` via `==`, but `[]` is truthy in a boolean context — they use different rules.
- `NaN !== NaN` — use `Number.isNaN()` to check for `NaN`, never `==/===`.
- For object-to-primitive: the `"default"` hint (used by `+` and `==`) behaves like `"number"` for most objects — `valueOf()` is tried first.
- In production code, always prefer `===` over `==` and explicit conversion functions (`Number()`, `String()`, `Boolean()`) over relying on implicit coercion.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
