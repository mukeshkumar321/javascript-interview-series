# Performance Optimization — Tricky Output Questions

## Table of Contents
1. [Rendering Performance Questions](#1-rendering-performance-questions)
2. [Code Splitting Questions](#2-code-splitting-questions)
3. [Layout Thrashing Questions](#3-layout-thrashing-questions)
4. [Web Worker Questions](#4-web-worker-questions)
5. [Core Web Vitals Questions](#5-core-web-vitals-questions)
6. [Advanced Performance Questions](#6-advanced-performance-questions)

---

## 1. Rendering Performance Questions

---

### Q1. What will be the output order?

```js
console.log("A");

setTimeout(() => console.log("B - setTimeout"), 0);

requestAnimationFrame(() => console.log("C - rAF"));

Promise.resolve().then(() => console.log("D - microtask"));

console.log("E");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
A
E
D - microtask
B - setTimeout
C - rAF
```

### Explanation
Synchronous code runs first: A, then E.

`Promise.resolve().then` is a **microtask** — it runs immediately after the current synchronous task, before any macrotasks. D fires next.

`setTimeout(fn, 0)` is a **macrotask** (timer). It runs after all microtasks. B fires next.

`requestAnimationFrame` fires just before the **next browser repaint**. In a real browser, it typically fires after setTimeout(0) when the browser is ready to paint. C fires last.

Note: in a Node.js environment, `requestAnimationFrame` does not exist. In browsers, the exact order of rAF vs. setTimeout(0) depends on the browser and current frame timing.

</details>

---

### Q2. How many times will the browser run layout in this code?

```js
const list = document.getElementById("list");

for (let i = 0; i < 200; i++) {
  const li = document.createElement("li");
  li.textContent = `Item ${i}`;
  list.appendChild(li); // layout invalidated on each append
}

console.log("Items added:", list.children.length);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Items added: 200
```

### Explanation
The browser does **not** run layout 200 times synchronously here. `appendChild` marks the layout as dirty but does not immediately recalculate it. JavaScript continues without waiting. The browser batches all 200 inserts and runs **one layout pass** before the next paint.

However, if you read a layout property (`offsetHeight`, `getBoundingClientRect`) inside the loop after each `appendChild`, that would force 200 synchronous layouts.

Fix for even better performance: use `DocumentFragment` to batch all DOM insertions into one:
```js
const frag = document.createDocumentFragment();
for (let i = 0; i < 200; i++) {
  const li = document.createElement("li");
  li.textContent = `Item ${i}`;
  frag.appendChild(li);
}
list.appendChild(frag); // single DOM mutation
```

</details>

---

### Q3. What is the output? Does the animation choice affect performance?

```js
const el = document.querySelector(".box");

// Animation A: using left (triggers layout)
function animateWithLeft() {
  let pos = 0;
  function step() {
    el.style.left = pos + "px"; // layout + paint + composite every frame
    pos += 1;
    if (pos < 300) requestAnimationFrame(step);
  }
  requestAnimationFrame(step);
  console.log("animating with left");
}

// Animation B: using transform (composite only)
function animateWithTransform() {
  let pos = 0;
  function step() {
    el.style.transform = `translateX(${pos}px)`; // composite only
    pos += 1;
    if (pos < 300) requestAnimationFrame(step);
  }
  requestAnimationFrame(step);
  console.log("animating with transform");
}

animateWithTransform();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
animating with transform
```

### Explanation
`animateWithLeft` changes the `left` CSS property. Since `left` affects geometry, it triggers a **full reflow + repaint + composite** every frame (~60 times per second). This is extremely expensive and will cause dropped frames on complex pages.

`animateWithTransform` uses `transform: translateX(...)`. This property is handled entirely by the **compositor thread** on the GPU — it bypasses layout and paint. Result: smooth 60fps even if the main thread is busy with JavaScript.

Always prefer `transform` and `opacity` for animations.

</details>

---

## 2. Code Splitting Questions

---

### Q4. What will be the output? What happens to the bundle size?

```js
// Without code splitting — all modules bundled together
import { heavyChartLibrary } from "./chart-library.js"; // always loaded
import { heavyTableLibrary } from "./table-library.js"; // always loaded

console.log("App loaded");

// With code splitting — loaded only when needed
async function showChart() {
  const { heavyChartLibrary } = await import("./chart-library.js");
  console.log("Chart library loaded on demand");
}

async function showTable() {
  const { heavyTableLibrary } = await import("./table-library.js");
  console.log("Table library loaded on demand");
}

showChart();
console.log("Main thread not blocked waiting for chart library");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
App loaded
Main thread not blocked waiting for chart library
Chart library loaded on demand
```

### Explanation
With static `import`, `heavyChartLibrary` and `heavyTableLibrary` are included in the initial bundle — the user downloads them even if they never view a chart or table.

With dynamic `import()`, the chart library chunk is only fetched when `showChart()` is called. `import()` returns a **Promise** — it is non-blocking. The main thread logs `"Main thread not blocked"` immediately. When the chunk loads, the `.then` continuation runs and logs `"Chart library loaded on demand"`.

In a typical app with multiple large libraries, code splitting can reduce the initial bundle by 50–80%.

</details>

---

### Q5. Will tree shaking remove the unused export?

```js
// utils.js
export function formatCurrency(amount) {
  return `$${amount.toFixed(2)}`;
}

export function formatDate(date) {
  return date.toISOString().split("T")[0];
}

export function unusedHelper() {
  return "never called anywhere";
}

// main.js
import { formatCurrency } from "./utils.js";

console.log(formatCurrency(42.5));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
$42.50
```

### Explanation
At runtime the output is `$42.50`. At **build time**, a bundler with tree shaking (webpack, Rollup, Vite) will see that `formatDate` and `unusedHelper` are exported but never imported anywhere in the application. Both are removed from the final bundle.

This works because ES module `import`/`export` is **static** — the dependency graph is known before execution. CommonJS `require()` is dynamic (can be inside `if` statements, functions) and therefore cannot be reliably tree-shaken.

</details>

---

## 3. Layout Thrashing Questions

---

### Q6. How many forced synchronous layouts does this code trigger?

```js
const items = document.querySelectorAll(".item");

items.forEach(item => {
  item.style.width = "100px";          // write
  const h = item.offsetHeight;         // read  → forces layout
  item.style.height = h * 2 + "px";   // write
});

console.log("done:", items.length, "elements processed");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
done: 8 elements processed   // depends on how many .item elements exist
```

### Explanation
**One forced synchronous layout per iteration** — `items.length` total reflows. The pattern is:

1. `style.width = "100px"` — invalidates layout (marks dirty)
2. `item.offsetHeight` — the layout is dirty, so the browser **must flush it synchronously** to return an accurate value (forced reflow #1)
3. `style.height = ...` — invalidates layout again
4. Next iteration: same cycle repeats

Fix:
```js
const heights = [...items].map(el => el.offsetHeight); // read phase: 1 layout
items.forEach((el, i) => {
  el.style.width  = "100px";
  el.style.height = heights[i] * 2 + "px";
}); // write phase: 1 lazy layout before paint
```

</details>

---

### Q7. What is the output? Is there a performance problem?

```js
const container = document.getElementById("container");

function updateLayout() {
  const width  = container.offsetWidth;  // forced layout
  const height = container.offsetHeight; // forced layout (2nd time!)

  container.style.padding = width > 800 ? "20px" : "10px";
  container.style.margin  = height > 600 ? "auto" : "0";

  return { width, height };
}

const dims = updateLayout();
console.log("Container:", dims.width, "x", dims.height);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Container: 1024 x 768   // depends on actual container size
```

### Explanation
There are **two forced synchronous layouts** in `updateLayout`. Reading `container.offsetWidth` triggers one layout flush. Reading `container.offsetHeight` immediately after triggers **another** flush (because after the first read, no write occurred, but the engine still needs to re-evaluate on the second read if the layout is dirty from the previous style change... in this case no style was changed between the two reads, so a smart engine may only flush once).

More precisely: if the first read happens when layout is clean, both reads can be served from the cached layout (one flush). But if any style is mutated between the reads, a second flush occurs.

Fix: read all needed properties **before** any writes:
```js
const width  = container.offsetWidth;
const height = container.offsetHeight;
// now write
container.style.padding = width > 800 ? "20px" : "10px";
container.style.margin  = height > 600 ? "auto" : "0";
```

</details>

---

### Q8. What does this code output, and what is the performance fix?

```js
const elements = document.querySelectorAll("p");
const snapshot = [];

// Reading into a snapshot first
elements.forEach(el => {
  snapshot.push(el.getBoundingClientRect().height);
});

// Then writing
elements.forEach((el, i) => {
  el.style.minHeight = snapshot[i] + 10 + "px";
});

console.log("First paragraph min-height set to:", snapshot[0] + 10, "px");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
First paragraph min-height set to: 28 px   // depends on line-height
```

### Explanation
This is the **correct** pattern. All `getBoundingClientRect` reads happen in the first loop — one layout flush at the start. All style writes happen in the second loop — the browser batches them into one deferred layout + paint before the next frame.

Compare with the naive version (read + write in the same loop), which would trigger one forced layout per element. The snapshot approach reduces `n` forced layouts to 1, which is a significant speedup for long lists.

</details>

---

## 4. Web Worker Questions

---

### Q9. What will be the output order?

```js
// Assume a Blob-based worker inline for this example
const code = `
  self.onmessage = function(e) {
    let sum = 0;
    for (let i = 0; i < e.data; i++) sum += i;
    self.postMessage(sum);
  };
`;
const blob = new Blob([code], { type: "text/javascript" });
const worker = new Worker(URL.createObjectURL(blob));

console.log("1 - before postMessage");
worker.postMessage(100);
console.log("2 - after postMessage");

worker.onmessage = (e) => {
  console.log("3 - worker result:", e.data);
  worker.terminate();
};

console.log("4 - end of script");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
1 - before postMessage
2 - after postMessage
4 - end of script
3 - worker result: 4950
```

### Explanation
`worker.postMessage(100)` is **non-blocking** — it schedules a message to the worker and returns immediately. The main thread continues synchronously: 1, 2, 4. When the worker finishes computing (on its own thread) and calls `self.postMessage(sum)`, the `worker.onmessage` handler fires as an asynchronous event callback — after all synchronous code completes. 3 is logged last.

Sum of 0..99 = `99 * 100 / 2 = 4950`.

</details>

---

### Q10. Can a Web Worker access the DOM? What is the output?

```js
const code = `
  try {
    document.title = "Modified from worker";
    self.postMessage("DOM access succeeded");
  } catch (e) {
    self.postMessage("Error: " + e.message);
  }
`;
const blob = new Blob([code], { type: "text/javascript" });
const worker = new Worker(URL.createObjectURL(blob));

worker.onmessage = (e) => {
  console.log("Worker says:", e.data);
  worker.terminate();
};
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Worker says: Error: document is not defined
```

### Explanation
Web Workers run in a **completely separate global scope** that has no access to `document`, `window`, `localStorage`, or any DOM API. This is intentional — DOM manipulation must happen only on the main thread to avoid concurrency issues with the rendering engine. Workers have access to `fetch`, `XMLHttpRequest`, `WebSocket`, `IndexedDB`, `crypto`, `console`, and `setTimeout`, but not the DOM.

</details>

---

## 5. Core Web Vitals Questions

---

### Q11. Which code pattern hurts CLS (Cumulative Layout Shift)?

```js
// Pattern A: image without explicit dimensions
// <img src="banner.jpg" alt="Banner" />
// When the image loads, it pushes other content down → CLS

// Pattern B: image with explicit dimensions
// <img src="banner.jpg" width="1200" height="400" alt="Banner" />
// Browser reserves space before image loads → no CLS

// Simulating a layout shift in JavaScript
const container = document.querySelector(".ad-slot");

// Bad: injecting content that shifts existing content
setTimeout(() => {
  const banner = document.createElement("div");
  banner.style.height = "250px";
  banner.style.background = "lightblue";
  document.body.insertBefore(banner, container); // SHIFTS content below
  console.log("Layout shift caused");
}, 2000); // fires after user started reading

// Good: reserve space upfront
const placeholder = document.createElement("div");
placeholder.style.height = "250px"; // pre-reserved space
placeholder.style.background = "transparent";
document.body.insertBefore(placeholder, container);

setTimeout(() => {
  placeholder.style.background = "lightblue"; // fills reserved space
  console.log("No layout shift — space was reserved");
}, 2000);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
No layout shift — space was reserved
```

### Explanation
**CLS** measures unexpected layout shifts — when elements move because content around them is inserted or resized. Injecting a 250px banner after the user starts reading pushes everything below it down, causing a high CLS score.

The fix: reserve the space upfront with a placeholder of the same size. When the real content arrives, it fills the pre-reserved space without shifting anything. `width`/`height` attributes on `<img>` tags serve the same purpose.

</details>

---

### Q12. What causes poor INP (Interaction to Next Paint)?

```js
document.getElementById("save-btn").addEventListener("click", (e) => {
  // Synchronous heavy work on the main thread
  const start = Date.now();

  // Simulating 400ms of heavy synchronous processing
  while (Date.now() - start < 400) {}

  // Update the UI
  document.getElementById("status").textContent = "Saved!";
  console.log("Save complete");
});

// INP measures: click event → next paint
// 400ms of blocking work → INP ≈ 400ms (well above the 200ms threshold)
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Save complete
```

### Explanation
**INP** (Interaction to Next Paint) measures the time from a user interaction (click, keypress) to when the browser can next paint an updated frame. The 400ms `while` loop blocks the main thread for the entire duration — the browser cannot paint the `"Saved!"` status update until the loop completes. This results in an INP of ~400ms, far above the "good" threshold of 200ms.

Fix: move the heavy work to a **Web Worker** and update the UI only when the result arrives. Or break the work into chunks with `setTimeout(fn, 0)` / `scheduler.yield()` so the browser can paint between chunks.

</details>

---

## 6. Advanced Performance Questions

---

### Q13. What is the output? What performance pattern does this illustrate?

```js
function memoize(fn) {
  const cache = new Map();
  return function(...args) {
    const key = args.join(",");
    if (cache.has(key)) return cache.get(key);
    const result = fn(...args);
    cache.set(key, result);
    return result;
  };
}

let callCount = 0;

const slowFib = memoize(function fib(n) {
  callCount++;
  if (n <= 1) return n;
  return slowFib(n - 1) + slowFib(n - 2);
});

console.log(slowFib(10));
console.log("Call count:", callCount);
console.log(slowFib(10)); // second call — should hit cache
console.log("Call count after second call:", callCount);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
55
Call count: 11
55
Call count after second call: 11
```

### Explanation
`slowFib(10)` computes Fibonacci recursively. With memoization, each value `fib(0)` through `fib(10)` is computed exactly **once** — 11 calls total (n=0 through n=10). Without memoization, the naive recursive Fibonacci would make 177 calls for `fib(10)`.

The second call to `slowFib(10)` hits the top-level cache immediately — `callCount` does not increase. This demonstrates how memoization converts O(2^n) repeated computation into O(n) unique computations.

</details>

---

### Q14. What is the output? What performance concern does this highlight?

```js
const results = [];

function addItem(n) {
  results.push({ id: n, data: new Array(1000).fill(n) });
}

for (let i = 0; i < 10000; i++) {
  addItem(i);
}

console.log("Items:", results.length);
console.log("First item id:", results[0].id);

// Clearing the cache
results.length = 0; // truncate the array
console.log("After clear:", results.length);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Items: 10000
First item id: 0
After clear: 0
```

### Explanation
10,000 objects each holding a 1,000-element array are pushed into `results`. Total allocations: ~10 million array slots plus 10,000 wrapper objects — significant GC pressure.

`results.length = 0` is an efficient way to truncate an array in place — it avoids creating a new array (unlike `results = []`, which creates a new reference and leaves the old array for GC). After truncation, all 10,000 objects are eligible for garbage collection.

Performance concern: if this pattern runs repeatedly (e.g., refreshing a list every second), the constant allocation and collection of millions of objects keeps the GC busy and can cause periodic pauses. Fix: reuse objects (object pooling) or use typed arrays where data is numeric.

</details>

---

### Q15. What will be the output? What rendering optimization does this implement?

```js
class EventDelegator {
  constructor(container) {
    this.container = container;
    this.handlers  = new Map();

    container.addEventListener("click", (e) => {
      const target = e.target.closest("[data-action]");
      if (!target) return;
      const action = target.dataset.action;
      const handler = this.handlers.get(action);
      if (handler) handler(target, e);
    });
  }

  on(action, handler) {
    this.handlers.set(action, handler);
    return this;
  }
}

// Usage
const delegator = new EventDelegator(document.getElementById("app") || document.body);

delegator
  .on("delete", (el) => console.log("Delete:", el.dataset.id))
  .on("edit",   (el) => console.log("Edit:",   el.dataset.id))
  .on("view",   (el) => console.log("View:",   el.dataset.id));

// Simulate a click on a hypothetical button:
// <button data-action="edit" data-id="42">Edit</button>
const fakeEvent = { target: { closest: (s) => ({ dataset: { action: "edit", id: "42" } }) } };
const action = fakeEvent.target.closest("[data-action]").dataset.action;
const handler = delegator.handlers.get(action);
if (handler) handler(fakeEvent.target.closest("[data-action]"), fakeEvent);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Edit: 42
```

### Explanation
This implements **event delegation** — a single listener on the container handles all clicks for every `[data-action]` element inside it, regardless of how many there are or whether they are added dynamically later.

Performance benefit: instead of attaching N listeners (one per button/link), there is exactly **1 listener** on the container. Memory usage is constant regardless of list size. Dynamically added elements with `data-action` attributes automatically work without re-registering listeners.

The `closest("[data-action]")` call traverses up the DOM tree from the click target to find the nearest ancestor (or self) with a `data-action` attribute, correctly handling clicks on child elements inside a button.

</details>

---

### Q16. What will be the output? What resource loading optimization is shown?

```js
// Simulating dynamic resource hinting based on user behavior
function prefetchOnHover(anchorEl) {
  let prefetched = false;

  anchorEl.addEventListener("mouseenter", () => {
    if (prefetched) return;
    prefetched = true;

    const link = document.createElement("link");
    link.rel  = "prefetch";
    link.href = anchorEl.href;
    document.head.appendChild(link);
    console.log("Prefetching:", anchorEl.href);
  });
}

// Hypothetical anchor element
const anchor = {
  href: "https://example.com/page2.js",
  addEventListener: (event, fn) => {
    if (event === "mouseenter") fn(); // simulate immediate hover
  }
};

prefetchOnHover(anchor);
prefetchOnHover(anchor); // second hover — should NOT prefetch again
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Prefetching: https://example.com/page2.js
```

### Explanation
The `prefetched` flag ensures the `<link rel="prefetch">` element is only added **once** per anchor, even if the user mouses over multiple times. Without the flag, hovering 10 times would inject 10 identical `<link>` tags, causing 10 redundant network requests.

The prefetch hint tells the browser to fetch `page2.js` at **low priority** during idle time, so it is ready in the cache when the user navigates to the next page. This can make subsequent page loads feel instant.

</details>

---

## Final Tips

- `requestAnimationFrame` fires before the next repaint and pauses in hidden tabs — always use it for visual animations, never `setInterval`.
- Reading `offsetWidth`, `offsetHeight`, `getBoundingClientRect`, `scrollTop`, or `getComputedStyle` after any DOM mutation causes a **forced synchronous layout**. Read first, write after.
- `transform` and `opacity` are the only CSS properties that animate on the GPU compositor without triggering layout or paint — use them for smooth 60fps animations.
- Web Workers cannot access the DOM but can run any computation-heavy code. Always use them for tasks taking more than 50ms.
- Tree shaking only works with **ES module static imports** — `import { fn } from "./utils"`. CommonJS `require()` is not tree-shakable.
- Code splitting with dynamic `import()` is non-blocking — it returns a Promise and does not freeze the main thread while the chunk downloads.
- Memoization trades memory for speed — cache results of pure functions to avoid re-computation.
- CLS is caused by content injected after page load without reserving space — always pre-size containers for dynamic content.
- INP measures the time from interaction to next paint — keep every event handler's main-thread work under 50ms.
- Event delegation reduces memory usage from O(n) listeners to O(1) and automatically covers dynamically added elements.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
