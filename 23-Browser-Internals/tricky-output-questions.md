# Browser Internals — Tricky Output Questions

## Table of Contents
1. [Browser Rendering Questions](#1-browser-rendering-questions)
2. [Reflow vs Repaint Questions](#2-reflow-vs-repaint-questions)
3. [JIT Optimization Questions](#3-jit-optimization-questions)
4. [Main Thread Questions](#4-main-thread-questions)
5. [V8 Optimization Questions](#5-v8-optimization-questions)

---

## 1. Browser Rendering Questions

---

### Q1. What will be the output and order of logs?

```js
console.log("1 - script start");

document.addEventListener("DOMContentLoaded", () => {
  console.log("3 - DOMContentLoaded");
});

window.addEventListener("load", () => {
  console.log("4 - load");
});

console.log("2 - script end");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
1 - script start
2 - script end
3 - DOMContentLoaded
4 - load
```

### Explanation
Synchronous code runs first (1, 2). After the HTML parser finishes building the DOM, `DOMContentLoaded` fires (3). Only after all external resources (images, stylesheets, subframes) finish loading does `load` fire (4). `DOMContentLoaded` always fires before `load`.

</details>

---

### Q2. What will the height of the element be — and why does the order matter?

```js
const div = document.createElement("div");
div.style.height = "100px";

// Reading height BEFORE appending to DOM
const heightBefore = div.offsetHeight;

document.body.appendChild(div);

// Reading height AFTER appending to DOM
const heightAfter = div.offsetHeight;

console.log("Before append:", heightBefore);
console.log("After append:", heightAfter);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Before append: 0
After append: 100
```

### Explanation
`offsetHeight` requires the element to be part of the **live document** and have layout calculated. Before the element is appended, it is detached — the browser cannot calculate its layout, so `offsetHeight` returns `0`. After `appendChild`, the element is in the render tree and layout runs, returning the correct `100px`.

</details>

---

### Q3. What will be the output and what rendering step is triggered?

```js
const el = document.querySelector(".box");
el.style.display = "none";

console.log("display:none set");

const h = el.offsetHeight;

console.log("offsetHeight:", h);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
display:none set
offsetHeight: 0
```

### Explanation
When `display: none` is applied, the element is removed from the render tree entirely. It has no box in the layout, so its `offsetHeight` is `0`. Reading `offsetHeight` still triggers a **synchronous layout flush** (forced reflow) to get an up-to-date value — but since the element has no layout box, the result is `0`.

</details>

---

### Q4. What fires first — DOMContentLoaded or the inline script at the bottom of body?

```html
<body>
  <script>console.log("A - inline script");</script>
  <script>
    document.addEventListener("DOMContentLoaded", () => console.log("C - DCL"));
  </script>
  <script>console.log("B - second inline script");</script>
</body>
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
A - inline script
B - second inline script
C - DCL
```

### Explanation
Inline scripts are executed **synchronously** as the HTML parser encounters them. The parser runs them in document order — A first, B second. `DOMContentLoaded` fires after all synchronous scripts have run and the full DOM is parsed. So C fires after A and B.

</details>

---

## 2. Reflow vs Repaint Questions

---

### Q5. How many reflows does this code trigger?

```js
const box = document.getElementById("box");

// Operation 1
box.style.width = "200px";

// Operation 2
const w = box.offsetWidth;

// Operation 3
box.style.height = w + "px";

// Operation 4
box.style.background = "red";

console.log("Width:", w);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Width: 200
```

### Explanation
**Two reflows** are triggered:

1. After `box.style.width = "200px"`, the browser marks the layout as dirty but does not run layout immediately (lazy).
2. `box.offsetWidth` forces a **synchronous reflow** to resolve the pending dirty layout and return an accurate value. This is reflow #1.
3. `box.style.height = w + "px"` marks layout dirty again.
4. `box.style.background = "red"` does NOT trigger reflow (background does not affect geometry). It only schedules a repaint.
5. The browser runs reflow #2 before the next paint to resolve the height change.

The read between two writes (`write → read → write`) is the classic **layout thrashing** pattern.

</details>

---

### Q6. Which properties trigger layout (reflow), which trigger only paint, and which trigger only composite?

```js
const el = document.querySelector(".item");

// A
el.style.width = "100px";
// B
el.style.color = "red";
// C
el.style.transform = "translateX(50px)";
// D
el.style.opacity = "0.5";
// E
el.style.padding = "10px";
// F
el.style.boxShadow = "0 0 10px black";

console.log("styles applied");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
styles applied
```

### Explanation
| Change | Pipeline cost |
|---|---|
| A: `width` | Layout + Paint + Composite |
| B: `color` | Paint + Composite (no geometry change) |
| C: `transform` | Composite only (GPU layer) |
| D: `opacity` | Composite only (GPU layer) |
| E: `padding` | Layout + Paint + Composite |
| F: `box-shadow` | Paint + Composite |

`transform` and `opacity` are the "free" properties for animation because they only require compositing — the GPU handles them without touching the main thread.

</details>

---

### Q7. This loop reads and writes layout properties alternately. How many reflows does it cause?

```js
const items = document.querySelectorAll(".item");

// Slow version
for (let i = 0; i < items.length; i++) {
  const w = items[i].offsetWidth;    // read  — forces reflow
  items[i].style.width = w + 10 + "px"; // write — marks dirty
}

console.log("done");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
done
```

### Explanation
**`items.length` reflows** are triggered — one per iteration. Each iteration reads `offsetWidth` (which forces a synchronous layout flush to get the current value) and then writes `width` (marking the layout dirty again). The next read forces another flush, and so on.

Fix: separate reads from writes:
```js
// Fast version — only 1 reflow
const widths = [...items].map(el => el.offsetWidth); // all reads first
items.forEach((el, i) => { el.style.width = widths[i] + 10 + "px"; }); // all writes
```

</details>

---

## 3. JIT Optimization Questions

---

### Q8. What is the output? Why might this function deoptimize?

```js
function add(a, b) {
  return a + b;
}

console.log(add(1, 2));       // A
console.log(add("x", "y"));   // B
console.log(add(3, 4));       // C
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
3
xy
7
```

### Explanation
(A) `add(1, 2)` → `3`. V8 sees two numbers. After a few calls with numbers, Turbofan optimizes `add` for `(number, number)`.

(B) `add("x", "y")` → `"xy"`. Strings violate the assumed types. Turbofan **deoptimizes** the function — it discards the machine-code version and falls back to Ignition's bytecode. `+` with strings performs concatenation.

(C) `add(3, 4)` → `7`. After the deoptimization, V8 may re-optimize for numbers again, but a deoptimization has still occurred (extra overhead).

**Lesson:** always call functions with consistent argument types to keep them in their optimized (Turbofan) state.

</details>

---

### Q9. What will be the output? What does V8 do with this shape-changing object?

```js
function makePoint(x, y) {
  const p = {};
  p.x = x;
  p.y = y;
  return p;
}

const a = makePoint(1, 2);
const b = makePoint(3, 4);

a.z = 10; // adds a new property — hidden class transition

console.log(a.x, a.y, a.z); // A
console.log(b.x, b.y);       // B
console.log(a.constructor === b.constructor); // C
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
1 2 10
3 4
true
```

### Explanation
`a` and `b` start with the same hidden class (shape: `{x, y}`). When `a.z = 10` is added, `a` **transitions** to a new hidden class (shape: `{x, y, z}`). Now `a` and `b` have different hidden classes. Inline caches that were monomorphic on `{x, y}` become polymorphic, reducing optimization.

`a.constructor === b.constructor` is `true` — both are plain `Object` instances. Hidden classes are an internal V8 concept, not exposed to JavaScript.

</details>

---

## 4. Main Thread Questions

---

### Q10. What will be the output order?

```js
console.log("A");

setTimeout(() => console.log("B"), 0);

Promise.resolve().then(() => console.log("C"));

requestAnimationFrame(() => console.log("D"));

console.log("E");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
A
E
C
B
D
```

### Explanation
- `A` and `E` are synchronous — logged immediately in order.
- `Promise.resolve().then(...)` schedules a **microtask**. Microtasks run after the current synchronous task completes, before any macrotasks. `C` is logged next.
- `setTimeout(fn, 0)` schedules a **macrotask**. It runs after microtasks. `B` is logged after `C`.
- `requestAnimationFrame` is scheduled before the **next browser repaint**, which happens after the current task queue is processed. `D` is logged last (in a browser environment).

Note: `D` may appear before or after `B` depending on timing in some environments, but in a real browser, rAF fires before the next paint and after the current task queue.

</details>

---

### Q11. This code blocks the main thread. What is the UI effect?

```js
document.getElementById("run-btn").addEventListener("click", () => {
  const start = Date.now();

  // Blocking the main thread for 3 seconds
  while (Date.now() - start < 3000) {}

  console.log("Done — 3 seconds of blocking");
});
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Done — 3 seconds of blocking
// (after the while loop completes)
```

### Explanation
The `while` loop runs entirely on the **main thread**, occupying it for 3 seconds. During this time:
- The browser cannot process any events (clicks, keystrokes)
- No frames can be rendered (the page appears frozen/janky)
- No other JavaScript (timers, promises) can execute

This is a classic **long task**. The fix is to break the work into smaller chunks and yield control back to the browser between chunks using `setTimeout(fn, 0)`, `scheduler.yield()`, or move the work to a **Web Worker**.

</details>

---

### Q12. What is the output and which thread runs the worker code?

```js
// Assume this runs in a browser with Blob-based worker creation
const workerCode = `
  self.onmessage = function(e) {
    const result = e.data.reduce((sum, n) => sum + n, 0);
    self.postMessage(result);
  };
`;

const blob = new Blob([workerCode], { type: "application/javascript" });
const worker = new Worker(URL.createObjectURL(blob));

worker.postMessage([1, 2, 3, 4, 5]);
worker.onmessage = (e) => console.log("Sum:", e.data);

console.log("Main thread continues immediately");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Main thread continues immediately
Sum: 15
```

### Explanation
The `worker.postMessage` call is **non-blocking**. The main thread immediately logs `"Main thread continues immediately"` without waiting for the worker to finish. The worker runs on a **separate OS thread** — it does not block the main thread. When the worker finishes computing the sum (`15`), it calls `self.postMessage(15)`, which delivers the message back to the main thread, triggering `worker.onmessage` and logging `"Sum: 15"`.

</details>

---

## 5. V8 Optimization Questions

---

### Q13. What is the output? What optimization problem does this code illustrate?

```js
function processShape(shape) {
  return shape.x + shape.y;
}

const circle  = { x: 1, y: 2, radius: 5 };    // hidden class A
const rect    = { x: 3, y: 4, width: 6, height: 8 }; // hidden class B
const point   = { x: 5, y: 6 };               // hidden class C

console.log(processShape(circle));
console.log(processShape(rect));
console.log(processShape(point));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
3
7
11
```

### Explanation
Each object has a **different hidden class** (different shape). When `processShape` is called with all three, the inline cache at `shape.x` and `shape.y` becomes **polymorphic** (3 shapes → transitions toward megamorphic). A monomorphic IC (one shape) would be fastest. A megamorphic IC (5+ shapes) falls back to a generic hash-table lookup.

Fix for hot code: keep all shapes consistent. If `x` and `y` are always present, define all objects with at least `{ x, y }` as the first two properties so they share a hidden class prefix.

</details>

---

### Q14. What is the output? Will Turbofan optimize this function, and why not?

```js
function sumArray(arr) {
  let total = 0;
  for (let i = 0; i < arr.length; i++) {
    total += arr[i];
  }
  return total;
}

console.log(sumArray([1, 2, 3, 4, 5]));       // A — all numbers (SMI array)
console.log(sumArray([1, 2, "3", 4, 5]));      // B — mixed types
console.log(sumArray([1.1, 2.2, 3.3]));        // C — floating point numbers
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
15
3345
6.6000000000000005
```

### Explanation
(A) `[1,2,3,4,5]` — V8 stores this as a **SMI (small integer) array**, the most optimized array kind. Turbofan compiles a tight numeric loop. Result: `15`.

(B) `[1, 2, "3", 4, 5]` — a **mixed array** defeats type specialization. `total` starts at `0`. After `0+1+2=3`, the addition `3 + "3"` hits a string and switches to concatenation: `"33" + 4 = "334"`, `"334" + 5 = "3345"`. Result: `"3345"`.

(C) `[1.1, 2.2, 3.3]` — V8 uses a **DOUBLE array** (heap numbers). Still fast, but slightly less than SMI. Result: `6.600000000000001` (floating-point precision).

</details>

---

### Q15. What is the output? What V8 concept does this illustrate?

```js
const obj = {};

// Assigning properties in consistent order
obj.name = "Alice";
obj.age  = 30;
obj.role = "dev";

// Deleting a property
delete obj.age;

// Adding it back
obj.age = 31;

console.log(obj.name); // A
console.log(obj.age);  // B
console.log(Object.keys(obj).join(", ")); // C
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Alice
31
name, role, age
```

### Explanation
(A) and (B) are straightforward property reads.

(C) `Object.keys` returns properties in insertion order. After `delete obj.age`, the property is removed. When `obj.age = 31` re-adds it, it is appended in the current insertion order — after `role`. This is why `age` appears last.

V8 internals: `delete` causes a **hidden class transition to a different shape** that is not shared with objects that never had that property deleted. Re-adding the property creates yet another transition. This is why `delete` is considered a "deoptimization" in performance-critical code — it breaks the shared hidden class chain and can push V8 from a fast path (C++ fixed-offset property access) to a **dictionary mode** (hash map) for that object.

</details>

---

## Final Tips

- The browser rendering pipeline order is: DOM → CSSOM → Render Tree → Layout → Paint → Composite.
- `DOMContentLoaded` fires when HTML is parsed; `load` fires when all external resources are loaded.
- Reading layout properties (`offsetWidth`, `getBoundingClientRect`) after DOM writes forces a **synchronous reflow**. Batch reads before writes.
- Only `transform` and `opacity` are truly "free" to animate — they skip layout and paint, running on the GPU compositor.
- V8 deoptimizes functions that receive inconsistent types across calls — write **type-stable** code.
- `delete` on an object property can push that object into **dictionary mode**, harming property access performance.
- `requestAnimationFrame` callbacks run just before the next repaint; `setTimeout(fn, 0)` is a macrotask that runs after microtasks; `Promise.then` is a microtask that runs before macrotasks.
- Long tasks (> 50 ms) block the main thread — use Web Workers or chunk the work with `setTimeout` / `scheduler.yield()`.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
