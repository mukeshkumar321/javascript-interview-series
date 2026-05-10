# JavaScript Fundamentals — Tricky Output Questions

> Carefully selected tricky JavaScript output questions focused only on fundamentals.

---

# Table of Contents

1. [Variables](#variables)
2. [Data Types](#data-types)
3. [Type Coercion](#type-coercion)
4. [Operators](#operators)
5. [Truthy and Falsy](#truthy-and-falsy)
6. [Comparison Operators](#comparison-operators)
7. [Functions Basics](#functions-basics)
8. [Control Flow](#control-flow)
9. [Loops](#loops)
10. [Template Literals](#template-literals)

---

# Variables

---

## 1. var Redeclaration

```js
var a = 10;

var a = 20;

console.log(a);
```

<details>
<summary>Show Output</summary>

```js
20
```

### Explanation

`var` allows redeclaration.

Second declaration overrides the first value.

</details>

---

## 2. let Reassignment

```js
let a = 10;

a = 30;

console.log(a);
```

<details>
<summary>Show Output</summary>

```js
30
```

### Explanation

`let` allows reassignment but not redeclaration.

</details>

---

## 3. const Object Modification

```js
const user = {
  name: "John"
};

user.name = "Sam";

console.log(user.name);
```

<details>
<summary>Show Output</summary>

```js
Sam
```

### Explanation

`const` prevents reassignment of the variable reference.

Object properties can still be modified.

</details>

---

## 4. Undefined Variable

```js
let a;

console.log(a);
```

<details>
<summary>Show Output</summary>

```js
undefined
```

### Explanation

Variable declared without value gets `undefined`.

</details>

---

# Data Types

---

## 5. typeof null

```js
console.log(typeof null);
```

<details>
<summary>Show Output</summary>

```js
object
```

### Explanation

This is a historical JavaScript bug.

`null` is actually a primitive value.

</details>

---

## 6. typeof NaN

```js
console.log(typeof NaN);
```

<details>
<summary>Show Output</summary>

```js
number
```

### Explanation

`NaN` means "Not a Number" but its type is still `number`.

</details>

---

## 7. Array Type

```js
console.log(typeof []);
```

<details>
<summary>Show Output</summary>

```js
object
```

### Explanation

Arrays are special kinds of objects in JavaScript.

</details>

---

## 8. Function Type

```js
console.log(typeof function () {});
```

<details>
<summary>Show Output</summary>

```js
function
```

### Explanation

Functions are callable objects with special type `"function"`.

</details>

---

# Type Coercion

---

## 9. String + Number

```js
console.log("5" + 1);
```

<details>
<summary>Show Output</summary>

```js
51
```

### Explanation

`+` operator converts number into string when one operand is string.

</details>

---

## 10. String - Number

```js
console.log("5" - 1);
```

<details>
<summary>Show Output</summary>

```js
4
```

### Explanation

`-` operator converts string into number.

</details>

---

## 11. Boolean Addition

```js
console.log(true + true);
```

<details>
<summary>Show Output</summary>

```js
2
```

### Explanation

`true` becomes `1`.

`1 + 1 = 2`

</details>

---

## 12. null Addition

```js
console.log(null + 1);
```

<details>
<summary>Show Output</summary>

```js
1
```

### Explanation

`null` converts to `0`.

</details>

---

## 13. undefined Addition

```js
console.log(undefined + 1);
```

<details>
<summary>Show Output</summary>

```js
NaN
```

### Explanation

`undefined` converts to `NaN`.

</details>

---

## 14. Empty String Addition

```js
console.log("" + 1 + 2);
```

<details>
<summary>Show Output</summary>

```js
12
```

### Explanation

Left-to-right evaluation:

```js
"" + 1 = "1"

"1" + 2 = "12"
```

</details>

---

## 15. Numeric Addition First

```js
console.log(1 + 2 + "");
```

<details>
<summary>Show Output</summary>

```js
3
```

### Explanation

```js
1 + 2 = 3

3 + "" = "3"
```

Console displays string `"3"`.

</details>

---

# Operators

---

## 16. Exponent Operator

```js
console.log(2 ** 3);
```

<details>
<summary>Show Output</summary>

```js
8
```

### Explanation

`**` means power operator.

```js
2 × 2 × 2 = 8
```

</details>

---

## 17. Modulus Operator

```js
console.log(10 % 3);
```

<details>
<summary>Show Output</summary>

```js
1
```

### Explanation

Returns remainder after division.

</details>

---

## 18. Division by Zero

```js
console.log(10 / 0);
```

<details>
<summary>Show Output</summary>

```js
Infinity
```

### Explanation

JavaScript returns `Infinity` instead of error.

</details>

---

## 19. Invalid Math

```js
console.log(0 / 0);
```

<details>
<summary>Show Output</summary>

```js
NaN
```

### Explanation

Result is mathematically undefined.

</details>

---

# Truthy and Falsy

---

## 20. Empty Array

```js
if ([]) {
  console.log("Truthy");
}
```

<details>
<summary>Show Output</summary>

```js
Truthy
```

### Explanation

Empty arrays are truthy.

</details>

---

## 21. Empty Object

```js
if ({}) {
  console.log("Truthy");
}
```

<details>
<summary>Show Output</summary>

```js
Truthy
```

### Explanation

Empty objects are truthy.

</details>

---

## 22. Empty String

```js
if ("") {
  console.log("Hello");
} else {
  console.log("Empty");
}
```

<details>
<summary>Show Output</summary>

```js
Empty
```

### Explanation

Empty string is falsy.

</details>

---

# Comparison Operators

---

## 23. Loose Equality

```js
console.log(5 == "5");
```

<details>
<summary>Show Output</summary>

```js
true
```

### Explanation

`==` performs type coercion.

</details>

---

## 24. Strict Equality

```js
console.log(5 === "5");
```

<details>
<summary>Show Output</summary>

```js
false
```

### Explanation

`===` checks both value and type.

</details>

---

## 25. null vs undefined

```js
console.log(null == undefined);
```

<details>
<summary>Show Output</summary>

```js
true
```

### Explanation

Special loose equality rule in JavaScript.

</details>

---

## 26. Strict null Comparison

```js
console.log(null === undefined);
```

<details>
<summary>Show Output</summary>

```js
false
```

### Explanation

Different data types.

</details>

---

## 27. NaN Comparison

```js
console.log(NaN == NaN);
```

<details>
<summary>Show Output</summary>

```js
false
```

### Explanation

`NaN` is never equal to itself.

</details>

---

# Functions Basics

---

## 28. Function Return

```js
function test() {
  return;
}

console.log(test());
```

<details>
<summary>Show Output</summary>

```js
undefined
```

### Explanation

Functions without return value return `undefined`.

</details>

---

## 29. Missing Parameters

```js
function add(a, b) {
  console.log(a + b);
}

add(2);
```

<details>
<summary>Show Output</summary>

```js
NaN
```

### Explanation

Missing parameter becomes `undefined`.

```js
2 + undefined = NaN
```

</details>

---

## 30. Extra Parameters

```js
function greet(name) {
  console.log(name);
}

greet("John", "Sam");
```

<details>
<summary>Show Output</summary>

```js
John
```

### Explanation

Extra arguments are ignored unless explicitly used.

</details>

---

# Control Flow

---

## 31. if Condition

```js
if (0) {
  console.log("Yes");
} else {
  console.log("No");
}
```

<details>
<summary>Show Output</summary>

```js
No
```

### Explanation

`0` is falsy.

</details>

---

## 32. switch Strict Comparison

```js
let value = "1";

switch (value) {
  case 1:
    console.log("Number");
    break;

  case "1":
    console.log("String");
    break;
}
```

<details>
<summary>Show Output</summary>

```js
String
```

### Explanation

`switch` uses strict comparison (`===`).

</details>

---

# Loops

---

## 33. Loop Variable

```js
for (let i = 0; i < 3; i++) {
  console.log(i);
}
```

<details>
<summary>Show Output</summary>

```js
0
1
2
```

### Explanation

Loop runs while condition is true.

</details>

---

## 34. Infinite Loop

```js
for (;;) {
  console.log("Hello");
}
```

<details>
<summary>Show Output</summary>

```js
Infinite Loop
```

### Explanation

All conditions are omitted, so loop never stops.

</details>

---

# Template Literals

---

## 35. Template Literal Output

```js
const name = "John";

console.log(`Hello ${name}`);
```

<details>
<summary>Show Output</summary>

```js
Hello John
```

### Explanation

Template literals allow variable interpolation.

</details>

---

# End

Practice these questions multiple times to strengthen your JavaScript fundamentals.