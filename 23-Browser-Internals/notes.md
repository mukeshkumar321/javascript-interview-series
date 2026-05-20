# Browser Internals in JavaScript

## Table of Contents

1. [How Browsers Work](#1-how-browsers-work)
2. [JavaScript Engine — V8 Overview](#2-javascript-engine--v8-overview)
3. [Parsing — Tokenizing and AST](#3-parsing--tokenizing-and-ast)
4. [JIT Compilation](#4-jit-compilation)
5. [Ignition and Turbofan in V8](#5-ignition-and-turbofan-in-v8)
6. [Hidden Classes](#6-hidden-classes)
7. [Inline Caching](#7-inline-caching)
8. [Critical Rendering Path](#8-critical-rendering-path)
9. [DOM Construction](#9-dom-construction)
10. [CSSOM Construction](#10-cssom-construction)
11. [Render Tree](#11-render-tree)
12. [Layout (Reflow)](#12-layout-reflow)
13. [Paint](#13-paint)
14. [Composite](#14-composite)
15. [Layers and GPU Acceleration](#15-layers-and-gpu-acceleration)
16. [will-change CSS Property](#16-will-change-css-property)
17. [requestAnimationFrame](#17-requestanimationframe)
18. [Main Thread vs Worker Thread](#18-main-thread-vs-worker-thread)
19. [Long Tasks and Main Thread Blocking](#19-long-tasks-and-main-thread-blocking)
20. [Summary](#20-summary)

---

## 1. How Browsers Work

A modern browser is composed of several major components working together:

| Component | Role |
|---|---|
| **User Interface** | Address bar, tabs, back/forward buttons |
| **Browser Engine** | Coordinates UI and rendering engine |
| **Rendering Engine** | Parses HTML/CSS, builds the visual display (Blink in Chrome) |
| **JavaScript Engine** | Parses and executes JavaScript (V8 in Chrome/Node.js) |
| **Networking** | HTTP/S requests, caching |
| **Data Storage** | localStorage, sessionStorage, IndexedDB, cookies |
| **UI Backend** | Draws widgets like combo boxes and windows |

The **rendering engine** and **JavaScript engine** communicate tightly — JavaScript can manipulate the DOM, which triggers the rendering pipeline.

```js
// From JS perspective, this is what triggers the browser pipeline
document.querySelector("h1").textContent = "Hello";
// 1. JS Engine executes this line
// 2. DOM is mutated
// 3. Rendering engine re-runs layout → paint → composite as needed
console.log("DOM updated");
```

### Output

```js
DOM updated
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 2. JavaScript Engine — V8 Overview

**V8** is Google's open-source JavaScript (and WebAssembly) engine, written in C++. It powers Chrome, Edge, Node.js, and Deno.

Key responsibilities:
- Parse JavaScript source code into an **Abstract Syntax Tree (AST)**
- Compile AST to bytecode via the **Ignition** interpreter
- Identify "hot" code paths and compile them to highly optimized machine code via the **Turbofan** compiler
- Manage the **heap** (object allocation and garbage collection)
- Handle **just-in-time (JIT)** compilation feedback loops

```js
// This is how V8 "sees" your code at a high level:
// 1. Source text   →  Tokenizer  →  Token stream
// 2. Token stream  →  Parser     →  AST
// 3. AST           →  Ignition   →  Bytecode (interpreted quickly)
// 4. Bytecode      →  Turbofan   →  Optimized machine code (for hot paths)

function add(a, b) {
  return a + b;
}

// Called many times with numbers — Turbofan optimizes for (number, number)
for (let i = 0; i < 1_000_000; i++) {
  add(i, i + 1);
}

console.log(add(2, 3)); // fast path — optimized machine code
```

### Output

```js
5
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 3. Parsing — Tokenizing and AST

Before any JavaScript executes, V8 must **parse** the source text:

1. **Tokenizing (Lexing)** — the source string is broken into a stream of meaningful tokens: keywords (`function`, `let`), identifiers, literals, operators, punctuation.
2. **Parsing** — the token stream is consumed by the parser to produce an **Abstract Syntax Tree (AST)** — a tree structure representing the program's grammar.

```js
// Source code:
const x = 2 + 3;

// Simplified token stream:
// [CONST] [IDENTIFIER:"x"] [ASSIGN] [NUMBER:2] [PLUS] [NUMBER:3] [SEMI]

// Simplified AST:
// VariableDeclaration
//   └─ VariableDeclarator (id: "x")
//       └─ BinaryExpression (+)
//           ├─ Literal (2)
//           └─ Literal (3)

// You can inspect the real AST at https://astexplorer.net
console.log(x); // 5
```

### Output

```js
5
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 4. JIT Compilation

**Just-In-Time (JIT)** compilation is a hybrid strategy: code starts interpreted (fast startup), and frequently executed ("hot") code paths are compiled to native machine code at runtime (fast execution).

| Strategy | Startup | Peak Speed | Used By |
|---|---|---|---|
| Pure Interpreter | Fast | Slow | Old JS engines |
| Ahead-of-Time (AOT) | Slow | Very fast | C, Rust, Go |
| JIT | Fast | Very fast | V8, SpiderMonkey |

```js
// JIT deoptimization example — changing a variable's type forces
// V8 to throw away the optimized code and re-optimize

function square(n) {
  return n * n;
}

// V8 sees numbers → optimizes square for (number) → (number)
for (let i = 0; i < 100000; i++) square(i);

// Passing a string "deoptimizes" the function:
// V8 throws away the specialized code and generates a generic version
console.log(square("3")); // "33" — string repetition, not math!
console.log(square(4));   // 16   — re-optimized for numbers again
```

### Output

```js
33
16
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 5. Ignition and Turbofan in V8

V8 uses a **two-tier** compilation pipeline:

- **Ignition** — the bytecode interpreter. It compiles AST to compact bytecode and runs it immediately. It also collects **type feedback** (which types are passed to each function).
- **Turbofan** — the optimizing compiler. When Ignition identifies "hot" functions (called many times), Turbofan uses the collected type feedback to produce highly optimized machine code.

If type assumptions are violated later (a "deoptimization"), Turbofan discards the optimized code and falls back to Ignition-interpreted bytecode.

```js
// Demonstrating type stability for V8 optimization
function multiply(a, b) {
  return a * b; // Turbofan assumes a, b are always numbers
}

// Warm up the function with numbers (triggers Turbofan optimization)
for (let i = 0; i < 50000; i++) multiply(i, 2);

// Stable types → fast optimized code
console.log(multiply(10, 20)); // 200

// Passing a different type forces deoptimization
console.log(multiply(null, 5)); // 0 (null coerced to 0, but deopt happens)
console.log(multiply(10, 20)); // 200 (re-optimized)
```

### Output

```js
200
0
200
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 6. Hidden Classes

JavaScript objects are dynamic — properties can be added or deleted at any time. To make property access as fast as statically-typed languages, V8 creates an internal concept called a **hidden class** (also called a "shape" or "map").

Every object starts with a hidden class. When you add a property, V8 transitions to a new hidden class. Objects with the same shape share the same hidden class, enabling fast property lookups via a constant offset rather than a hash table lookup.

```js
// Both objects follow the same property-addition order → same hidden class
// V8 can use the same optimized lookup for both
const p1 = { x: 1, y: 2 };
const p2 = { x: 3, y: 4 };

// Different order → different hidden classes → slower access
const p3 = { x: 1 };
p3.y = 2; // same result, same final shape — V8 transitions p3's hidden class

// Bad: adds properties in different orders per object — breaks shared hidden class
function makeBad(condition) {
  const obj = {};
  if (condition) {
    obj.a = 1;
    obj.b = 2;
  } else {
    obj.b = 2; // b added before a
    obj.a = 1;
  }
  return obj;
}

// Good: always initialize properties in the same order
function makeGood() {
  return { a: 0, b: 0 }; // consistent shape
}

console.log(p1.x + p2.x); // 4
```

### Output

```js
4
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 7. Inline Caching

**Inline caching (IC)** is a V8 optimization that caches the result of a property lookup the first time it happens at a given call site. On subsequent calls, V8 checks if the object has the same hidden class and, if so, uses the cached offset directly — skipping the full lookup.

| IC State | Description |
|---|---|
| **Uninitialized** | No call seen yet |
| **Monomorphic** | One shape seen — fully optimized |
| **Polymorphic** | 2–4 shapes seen — some overhead |
| **Megamorphic** | 5+ shapes seen — falls back to hash lookup |

```js
// Monomorphic — same shape every time → optimal
function getX(obj) {
  return obj.x;
}

const a = { x: 1 };
const b = { x: 2 };
const c = { x: 3 };

// All { x: number } objects have the same hidden class
console.log(getX(a)); // 1 — IC becomes monomorphic, fast
console.log(getX(b)); // 2 — uses cached offset
console.log(getX(c)); // 3 — uses cached offset

// Polymorphic — different shapes
const d = { x: 4, y: 5 };      // different hidden class
console.log(getX(d)); // 4 — IC becomes polymorphic
```

### Output

```js
1
2
3
4
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 8. Critical Rendering Path

The **Critical Rendering Path (CRP)** is the sequence of steps the browser performs to convert HTML, CSS, and JavaScript into pixels on the screen:

1. Parse HTML → build **DOM**
2. Parse CSS → build **CSSOM**
3. Combine DOM + CSSOM → build **Render Tree**
4. **Layout** (calculate geometry)
5. **Paint** (fill pixels)
6. **Composite** (send layers to GPU)

Optimizing the CRP means minimizing the work and data needed before the first meaningful pixel is shown.

```js
// JS can block the CRP — a <script> tag without async/defer
// pauses HTML parsing until the script downloads and executes

// Bad:
// <script src="heavy.js"></script>  ← blocks parser

// Good:
// <script src="heavy.js" defer></script>    ← runs after HTML parsed
// <script src="analytics.js" async></script> ← runs as soon as downloaded

// In JS, avoid measuring layout immediately after a DOM mutation:
const el = document.querySelector(".box");
el.style.width = "100px";                    // schedules style recalculation
const width = el.getBoundingClientRect().width; // FORCES synchronous layout
console.log(width); // 100
```

### Output

```js
100
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 9. DOM Construction

When the browser receives the HTML, the **HTML parser** processes it byte-by-byte, converting the markup into a tree of **DOM nodes**. This process:

- Is **incremental** — the browser starts rendering before the full HTML arrives.
- Is **blocked** by synchronous `<script>` tags — JS execution pauses the parser.
- Produces a **Document Object Model (DOM)** — a live, in-memory tree representation.

```js
// You can observe DOM construction interactivity in JS:
document.addEventListener("DOMContentLoaded", () => {
  // Fires when HTML has been fully parsed and DOM is ready
  // CSS, images, and subframes may still be loading
  const headings = document.querySelectorAll("h1");
  console.log("DOM ready, h1 count:", headings.length);
});

window.addEventListener("load", () => {
  // Fires when everything (images, CSS, fonts) is fully loaded
  console.log("All resources loaded");
});
```

### Output

```js
DOM ready, h1 count: 1   // (depends on the page)
All resources loaded
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 10. CSSOM Construction

In parallel with DOM construction, the browser parses all CSS and builds the **CSS Object Model (CSSOM)** — a tree of style rules. Like the DOM, it can be accessed and manipulated from JavaScript.

CSS is **render-blocking** — the browser will not render anything until the CSSOM is complete (to avoid a flash of unstyled content). This means large CSS files delay the first render.

```js
// Accessing CSSOM from JavaScript
const sheets = document.styleSheets;
console.log("Number of stylesheets:", sheets.length);

// Reading a computed style (forces CSSOM to be fully built)
const el = document.querySelector("body");
const computedStyle = window.getComputedStyle(el);
console.log("Body font-size:", computedStyle.fontSize);

// Dynamically adding a CSS rule via CSSOM
const style = document.createElement("style");
document.head.appendChild(style);
style.sheet.insertRule("body { background: #f0f0f0; }", 0);
```

### Output

```js
Number of stylesheets: 1       // depends on page
Body font-size: 16px           // depends on browser defaults
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 11. Render Tree

The **Render Tree** is built by combining the DOM and CSSOM. It contains only the nodes that will be **visually rendered** — nodes with `display: none` are excluded; pseudo-elements like `::before` are included.

Each node in the render tree (called a **render object** or **layout object**) knows its computed visual styles.

```js
// display:none elements are NOT in the render tree
// visibility:hidden elements ARE in the render tree (they occupy space)

const hidden1 = document.createElement("div");
hidden1.style.display = "none";
hidden1.textContent = "Invisible — not in render tree";
document.body.appendChild(hidden1);

const hidden2 = document.createElement("div");
hidden2.style.visibility = "hidden";
hidden2.textContent = "Hidden — still in render tree";
document.body.appendChild(hidden2);

console.log("display:none offsetHeight:", hidden1.offsetHeight);     // 0
console.log("visibility:hidden offsetHeight:", hidden2.offsetHeight); // > 0
```

### Output

```js
display:none offsetHeight: 0
visibility:hidden offsetHeight: 18   // typical line-height
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 12. Layout (Reflow)

**Layout** (also called **reflow**) is the process where the browser calculates the exact position and size of every render tree node. It runs:

- After the initial render tree is built
- Whenever geometry-affecting properties change

**Triggers for reflow:**
- Adding/removing/modifying DOM elements
- Changing CSS properties: `width`, `height`, `margin`, `padding`, `top`, `left`, `font-size`, etc.
- Reading layout properties (`offsetWidth`, `getBoundingClientRect`, `scrollTop`, etc.) after a mutation

```js
// Reading layout properties triggers SYNCHRONOUS layout (forced reflow)
const box = document.getElementById("box");

// Bad — layout thrashing: write then immediately read forces sync reflow
box.style.width = "200px";
const width = box.offsetWidth; // forces layout NOW
box.style.height = width + "px"; // another write after a read → another reflow

// Good — batch reads before writes
const currentWidth = box.offsetWidth;  // read first
box.style.width = currentWidth + 10 + "px"; // write after
box.style.height = currentWidth + "px";     // another write — only one reflow

console.log("Width read:", currentWidth);
```

### Output

```js
Width read: 190   // depends on actual box width
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 13. Paint

**Paint** is the process of filling in the pixels. The browser traverses the render tree and draws the visual content of each node: text, colors, borders, shadows, images.

Paint is **expensive for large areas** and is triggered by changes to visual properties that do not affect geometry: `color`, `background-color`, `border-color`, `box-shadow`, `outline`, etc.

```js
// CSS properties and their rendering cost:
// Layout (most expensive):  width, height, margin, padding, top, left, font-size
// Paint (medium):           color, background, border-color, box-shadow
// Composite (cheapest):     transform, opacity

// Animating color triggers paint every frame (slow)
function animateColor(el) {
  let hue = 0;
  function frame() {
    hue = (hue + 1) % 360;
    el.style.backgroundColor = `hsl(${hue}, 100%, 50%)`; // paint every frame
    requestAnimationFrame(frame);
  }
  requestAnimationFrame(frame);
}

// Animating transform/opacity triggers only compositing (fast)
function animateFast(el) {
  let pos = 0;
  function frame() {
    pos = (pos + 1) % 300;
    el.style.transform = `translateX(${pos}px)`; // composite only — no paint
    requestAnimationFrame(frame);
  }
  requestAnimationFrame(frame);
}
```

### Output

```js
// Visual only — no console output
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 14. Composite

**Compositing** is the final step: the browser assembles multiple painted layers into the final image and sends it to the GPU to be drawn on screen.

When an element is promoted to its own **compositor layer** (via `transform`, `opacity`, `will-change`, or `position: fixed`), changes to that element only require the compositor to re-assemble layers — **no layout or paint is needed**. This makes animations on those properties very smooth.

```js
// Checking whether an element is on its own layer:
// In Chrome DevTools → Layers panel, you can see each compositor layer

// Forcing layer promotion for smooth animation
const el = document.querySelector(".animated");

// This animation runs on the GPU compositor thread — 60 fps even if
// the main thread is busy
el.style.transition = "transform 0.3s ease";
el.style.transform = "translateX(100px)";

// Contrast: this animation requires paint on every frame
el.style.transition = "left 0.3s ease"; // triggers layout + paint
el.style.left = "100px";

console.log("animation started");
```

### Output

```js
animation started
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 15. Layers and GPU Acceleration

The browser maintains a **layer tree**. Some elements are promoted to their own compositing layer and rendered by the GPU:

- Elements with `will-change: transform` or `will-change: opacity`
- Elements with 3D transforms (`translate3d`, `scale3d`)
- Elements with `position: fixed`
- Canvas elements
- Video elements

GPU compositing allows those layers to animate without touching layout or paint, enabling 60+ fps animations.

```js
// Checking layer count in Chrome:
// DevTools → Rendering → (enable) Layer borders

// Promoting an element to a GPU layer explicitly
const card = document.querySelector(".card");
card.style.willChange = "transform"; // browser allocates a GPU layer

// Moving or fading the card now happens entirely on the GPU
function animateCard() {
  card.style.transform = "scale(1.05)";
  card.style.opacity = "0.9";
}

// Clean up the will-change hint after animation completes
card.addEventListener("transitionend", () => {
  card.style.willChange = "auto"; // release GPU layer
});

console.log("GPU layer created for card");
```

### Output

```js
GPU layer created for card
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 16. will-change CSS Property

`will-change` is a CSS hint that tells the browser **in advance** that an element will be animated, so the browser can prepare (allocate a compositor layer, pre-rasterize, etc.) before the animation starts.

```js
// Applying will-change at the right time

const button = document.querySelector(".animated-btn");

// Add before animation (on hover, before click, etc.)
button.addEventListener("mouseenter", () => {
  button.style.willChange = "transform, opacity";
});

// Remove after animation completes — holding layers wastes GPU memory
button.addEventListener("mouseleave", () => {
  button.style.willChange = "auto";
});

// Common mistake: setting will-change: transform on everything
// This wastes GPU memory and can HURT performance

// Good rule: use will-change only for elements you KNOW will animate,
// and only for the duration of the animation
console.log("will-change applied on hover");
```

### Output

```js
will-change applied on hover
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 17. requestAnimationFrame

`requestAnimationFrame` (rAF) schedules a callback to run **just before the browser's next repaint**, synchronized with the display's refresh rate (typically 60 fps = ~16.67 ms per frame). This ensures animations are smooth and avoids wasted work on hidden tabs.

```js
// Smooth animation loop using requestAnimationFrame
const box = document.querySelector(".box");
let startTime = null;
const duration = 1000; // 1 second
const distance = 300;  // pixels

function animate(timestamp) {
  if (!startTime) startTime = timestamp;
  const elapsed = timestamp - startTime;
  const progress = Math.min(elapsed / duration, 1); // clamp 0..1

  box.style.transform = `translateX(${progress * distance}px)`;

  if (progress < 1) {
    requestAnimationFrame(animate); // request next frame
  } else {
    console.log("Animation complete");
  }
}

requestAnimationFrame(animate);

// Why NOT use setInterval for animations:
// setInterval(callback, 16) fires at 16ms regardless of the actual
// refresh rate and can fire while the tab is hidden, wasting CPU.
// rAF pauses automatically in hidden tabs and aligns with the display.
```

### Output

```js
Animation complete   // after ~1 second
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 18. Main Thread vs Worker Thread

The browser's **main thread** handles almost everything: JavaScript execution, DOM manipulation, layout, paint, and event handling. This means long JavaScript tasks can **block rendering** and make the page unresponsive.

**Web Workers** run JavaScript on a separate thread, completely isolated from the DOM. They communicate with the main thread via `postMessage`.

```js
// main.js — main thread
const worker = new Worker("worker.js");

// Send data to worker for heavy processing
worker.postMessage({ numbers: Array.from({ length: 1000000 }, (_, i) => i) });

worker.onmessage = (event) => {
  console.log("Result from worker:", event.data.sum);
  // UI remains responsive while worker processes
};

// worker.js (separate file — no DOM access)
// self.onmessage = (event) => {
//   const { numbers } = event.data;
//   const sum = numbers.reduce((acc, n) => acc + n, 0);
//   self.postMessage({ sum });
// };

console.log("Worker started — main thread is not blocked");
```

### Output

```js
Worker started — main thread is not blocked
Result from worker: 499999500000
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 19. Long Tasks and Main Thread Blocking

A **long task** is any main-thread task that takes longer than **50 ms** (Chrome's threshold). Long tasks delay user interaction responses, causing perceived jank.

Common causes: synchronous JSON parsing of large data, heavy DOM manipulation, synchronous XHR, large blocking scripts.

```js
// Detecting long tasks with the PerformanceObserver API
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    console.log(`Long task detected: ${entry.duration.toFixed(1)}ms`);
  }
});
observer.observe({ type: "longtask", buffered: true });

// Simulating a long task — BLOCKS the main thread
function blockingWork() {
  const start = performance.now();
  while (performance.now() - start < 200) {
    // busy-waiting for 200ms — no frames can render during this time
  }
  console.log("Blocking work done");
}

// Fix: break work into chunks with scheduler.yield() or setTimeout
async function chunkedWork(items) {
  for (let i = 0; i < items.length; i++) {
    doWorkOn(items[i]);
    if (i % 100 === 0) {
      await new Promise(resolve => setTimeout(resolve, 0)); // yield to browser
    }
  }
  console.log("Chunked work done — main thread yielded regularly");
}

function doWorkOn(item) { /* process one item */ }
```

### Output

```js
Long task detected: 200.0ms
Blocking work done
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 20. Summary

| Topic | Key Point |
|---|---|
| Browser components | JS engine + rendering engine + networking + storage |
| V8 | Parses, compiles, optimizes, and GCs JavaScript |
| Tokenizing | Source → token stream |
| AST | Token stream → syntax tree used for compilation |
| JIT | Starts interpreted; hot paths compiled to machine code |
| Ignition | Bytecode interpreter; collects type feedback |
| Turbofan | Optimizing compiler; uses feedback; deoptimizes on type change |
| Hidden classes | V8's internal representation of object shape; enables fast property access |
| Inline caching | Caches property lookup result at a call site; monomorphic = fastest |
| Critical rendering path | HTML → DOM → CSSOM → Render Tree → Layout → Paint → Composite |
| DOM construction | HTML parsed into node tree; blocked by synchronous scripts |
| CSSOM | CSS parsed into style tree; render-blocking |
| Render tree | DOM + CSSOM, excluding hidden nodes |
| Layout / Reflow | Calculates geometry; triggered by geometry CSS changes or layout reads |
| Paint | Fills pixels; triggered by visual CSS changes (color, background) |
| Composite | GPU assembles layers; transform/opacity only cost this step |
| Layers | Promoted elements animate on the GPU without layout/paint |
| will-change | Hint to browser to pre-allocate a GPU layer |
| requestAnimationFrame | Schedules work before next repaint; pauses in hidden tabs |
| Main thread | Handles JS + DOM + layout + paint — avoid blocking it |
| Long tasks | > 50ms tasks cause jank; break them up or use Web Workers |

---

## Final Notes

Understanding browser internals transforms the way you write JavaScript. When you know that changing `width` triggers a full layout-paint-composite cycle while changing `transform` only triggers compositing, you make better decisions about which CSS properties to animate. When you understand that V8's Turbofan deoptimizes when a function receives unexpected types, you write type-stable code. The critical rendering path shows you why render-blocking scripts and large CSS files hurt your first contentful paint, and why `defer`/`async` attributes and code splitting matter. Combine this knowledge with requestAnimationFrame for smooth 60fps animations, Web Workers for CPU-intensive tasks, and the PerformanceObserver API for detecting long tasks, and you can build experiences that are fast by design rather than fast by accident.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
