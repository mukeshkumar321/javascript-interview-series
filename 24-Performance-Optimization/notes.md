# Performance Optimization in JavaScript

## Table of Contents

1. [Core Web Vitals](#1-core-web-vitals)
2. [Measuring Performance](#2-measuring-performance)
3. [Minimize Main Thread Work](#3-minimize-main-thread-work)
4. [Code Splitting](#4-code-splitting)
5. [Tree Shaking](#5-tree-shaking)
6. [Bundle Size Optimization](#6-bundle-size-optimization)
7. [Lazy Loading](#7-lazy-loading)
8. [Preload, Prefetch, and Preconnect](#8-preload-prefetch-and-preconnect)
9. [Debounce and Throttle for Performance](#9-debounce-and-throttle-for-performance)
10. [Virtual Scrolling and Windowing](#10-virtual-scrolling-and-windowing)
11. [Web Workers for CPU Tasks](#11-web-workers-for-cpu-tasks)
12. [Avoid Forced Synchronous Layout](#12-avoid-forced-synchronous-layout)
13. [Batch DOM Reads and Writes](#13-batch-dom-reads-and-writes)
14. [Event Delegation for Performance](#14-event-delegation-for-performance)
15. [Caching Strategies](#15-caching-strategies)
16. [Image Optimization](#16-image-optimization)
17. [CSS Performance Impact on JS](#17-css-performance-impact-on-js)
18. [Memory and GC Pressure](#18-memory-and-gc-pressure)
19. [Service Worker for Offline and Cache](#19-service-worker-for-offline-and-cache)
20. [Summary](#20-summary)

---

## 1. Core Web Vitals

**Core Web Vitals** are Google's standardized metrics for measuring real-world user experience. They directly affect SEO and perceived performance.

| Metric | Full Name | Measures | Good Threshold |
|---|---|---|---|
| **LCP** | Largest Contentful Paint | Loading performance | ≤ 2.5 s |
| **INP** | Interaction to Next Paint | Responsiveness | ≤ 200 ms |
| **CLS** | Cumulative Layout Shift | Visual stability | ≤ 0.1 |

```js
// Observing Core Web Vitals with the web-vitals library (or PerformanceObserver)
import { onLCP, onINP, onCLS } from "web-vitals";

onLCP(({ value }) => {
  console.log("LCP:", value.toFixed(0), "ms");
});

onINP(({ value }) => {
  console.log("INP:", value.toFixed(0), "ms");
});

onCLS(({ value }) => {
  console.log("CLS score:", value.toFixed(4));
});

// Or with raw PerformanceObserver:
new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    if (entry.entryType === "largest-contentful-paint") {
      console.log("LCP element:", entry.element?.tagName, entry.startTime.toFixed(0), "ms");
    }
  }
}).observe({ type: "largest-contentful-paint", buffered: true });
```

### Output

```js
LCP: 1820 ms
INP: 45 ms
CLS score: 0.0023
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 2. Measuring Performance

Reliable performance measurement requires the right tools at the right stage.

| Tool | Use Case |
|---|---|
| `performance.now()` | High-resolution timestamps for micro-benchmarks |
| `Performance.mark` / `measure` | Named timing marks visible in DevTools |
| Chrome DevTools → Performance tab | Full profiling: JS, layout, paint, composite |
| Lighthouse | Automated audit: Core Web Vitals, accessibility, SEO |
| `PerformanceObserver` | Observe navigation, resource, long-task entries at runtime |

```js
// 1. Micro-benchmark with performance.now()
const start = performance.now();
const data = new Array(1_000_000).fill(0).map((_, i) => i * 2);
const end = performance.now();
console.log("Map took:", (end - start).toFixed(2), "ms");

// 2. Named marks — visible in DevTools → Performance tab timeline
performance.mark("data-fetch-start");
// ... fetch data ...
performance.mark("data-fetch-end");
performance.measure("data-fetch", "data-fetch-start", "data-fetch-end");

const measures = performance.getEntriesByName("data-fetch");
console.log("Fetch duration:", measures[0].duration.toFixed(2), "ms");
```

### Output

```js
Map took: 12.45 ms
Fetch duration: 320.00 ms
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 3. Minimize Main Thread Work

The browser's main thread handles JavaScript, style calculations, layout, and paint. Keeping tasks short (under 50 ms) ensures the browser can process user input between tasks, resulting in a responsive UI.

```js
// Bad: one giant task that blocks the main thread for seconds
function processAll(items) {
  return items.map(item => heavyTransform(item));
}

// Good: break into 50ms chunks and yield between them
async function processInChunks(items, chunkSize = 100) {
  const results = [];
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    results.push(...chunk.map(item => heavyTransform(item)));
    // Yield to the browser to process input events and render frames
    await new Promise(resolve => setTimeout(resolve, 0));
    console.log(`Processed ${Math.min(i + chunkSize, items.length)} / ${items.length}`);
  }
  return results;
}

function heavyTransform(item) {
  return item * item; // simplified
}

processInChunks([1, 2, 3, 4, 5], 2).then(r => console.log("Done:", r));
```

### Output

```js
Processed 2 / 5
Processed 4 / 5
Processed 5 / 5
Done: [1, 4, 9, 16, 25]
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 4. Code Splitting

**Code splitting** divides a large JavaScript bundle into smaller chunks that are loaded on demand. This reduces the initial bundle size, improving the Time to Interactive (TTI).

Modern bundlers (webpack, Vite, Rollup) split at dynamic `import()` boundaries automatically.

```js
// Static import — always loaded, always in the main bundle
import { formatDate } from "./utils/date.js";

// Dynamic import — loaded only when needed (code splitting)
async function loadChartLibrary() {
  const { Chart } = await import("./vendor/chart.js");
  return Chart;
}

// Route-based splitting in a React-like SPA
async function navigateTo(route) {
  let Component;

  if (route === "/dashboard") {
    // dashboard.js is in a separate chunk — only fetched when user visits /dashboard
    const module = await import("./pages/Dashboard.js");
    Component = module.default;
  } else if (route === "/settings") {
    const module = await import("./pages/Settings.js");
    Component = module.default;
  }

  console.log("Loaded component:", Component?.name ?? "unknown");
}

navigateTo("/dashboard");
```

### Output

```js
Loaded component: Dashboard
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 5. Tree Shaking

**Tree shaking** is the process of removing dead code (exported but never imported) from the final bundle. It works on **ES modules** (`import`/`export`) because their dependency graph is static and analyzable at build time. CommonJS (`require`) is not tree-shakable.

```js
// math.js — named exports
export function add(a, b) { return a + b; }
export function subtract(a, b) { return a - b; }
export function multiply(a, b) { return a * b; }
export function divide(a, b) { return a / b; }

// app.js — only uses add and subtract
import { add, subtract } from "./math.js";

console.log(add(2, 3));       // 5
console.log(subtract(10, 4)); // 6

// multiply and divide are DEAD CODE — a bundler with tree shaking
// will NOT include them in the final bundle.

// Anti-pattern that prevents tree shaking:
// import * as math from "./math.js"; // imports everything — bundler can't shake
```

### Output

```js
5
6
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 6. Bundle Size Optimization

Reducing bundle size speeds up download time, parse time, and execution time. Key techniques:

1. **Minification** — remove whitespace, shorten variable names (Terser, esbuild)
2. **Compression** — gzip or Brotli at the server level (70–80% size reduction)
3. **Tree shaking** — eliminate dead code (see section 5)
4. **Vendor splitting** — separate rarely-changing vendor code for better caching
5. **Audit dependencies** — use `bundlephobia.com` before adding a package

```js
// Checking bundle stats in code — using webpack-bundle-analyzer (dev tool)
// Run: npx webpack --profile --json > stats.json
// Then: npx webpack-bundle-analyzer stats.json

// Avoiding large imports:
// Bad — imports the entire lodash library (~72 KB gzipped)
// import _ from "lodash";
// const result = _.chunk([1,2,3,4], 2);

// Good — imports only the function you need (~1 KB)
import chunk from "lodash/chunk";
const result = chunk([1, 2, 3, 4], 2);
console.log(result); // [[1, 2], [3, 4]]

// Also good — use native alternatives when possible
const result2 = [[1, 2, 3, 4].slice(0, 2), [1, 2, 3, 4].slice(2)];
console.log(result2); // [[1, 2], [3, 4]]
```

### Output

```js
[[1, 2], [3, 4]]
[[1, 2], [3, 4]]
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 7. Lazy Loading

**Lazy loading** defers the loading of non-critical resources until they are needed.

- **Images** — use the `loading="lazy"` HTML attribute or `IntersectionObserver`.
- **Modules** — use dynamic `import()` (see section 4).
- **Components** — in React: `React.lazy()` + `Suspense`.

```js
// Native lazy loading for images (browser-native, zero JS)
// <img src="hero.webp" loading="lazy" alt="Hero image" />

// IntersectionObserver-based lazy loading (broader control)
const lazyImages = document.querySelectorAll("img[data-src]");

const imageObserver = new IntersectionObserver((entries, observer) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      const img = entry.target;
      img.src = img.dataset.src;     // swap placeholder for real src
      img.removeAttribute("data-src");
      observer.unobserve(img);       // stop observing once loaded
      console.log("Loaded image:", img.src);
    }
  });
}, { rootMargin: "200px" }); // start loading 200px before visible

lazyImages.forEach(img => imageObserver.observe(img));
```

### Output

```js
Loaded image: https://example.com/photo1.webp
// (fires as each image enters the viewport)
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 8. Preload, Prefetch, and Preconnect

Resource hints tell the browser to fetch or prepare resources before they are explicitly needed.

| Hint | When to Use | Priority |
|---|---|---|
| `preload` | Critical resource needed for current page (fonts, LCP image, hero CSS) | High — fetches immediately |
| `prefetch` | Resource needed on the NEXT navigation | Low — idle-time fetch |
| `preconnect` | Establish TCP + TLS with a third-party origin early | Medium — saves handshake RTT |
| `dns-prefetch` | DNS lookup only (cheaper than preconnect) | Very low |

```js
// Adding resource hints programmatically from JavaScript
function addPreload(href, as, type) {
  const link = document.createElement("link");
  link.rel  = "preload";
  link.href = href;
  link.as   = as;
  if (type) link.type = type;
  document.head.appendChild(link);
  console.log(`Preloading ${as}: ${href}`);
}

// Preload the hero image so it is fetched in parallel with HTML parsing
addPreload("/images/hero.webp", "image");

// Preload a critical font
addPreload("/fonts/inter.woff2", "font", "font/woff2");

// Prefetch the next page's bundle when user hovers the nav link
document.querySelector("a[href='/about']")
  ?.addEventListener("mouseenter", () => {
    const link = document.createElement("link");
    link.rel  = "prefetch";
    link.href = "/bundles/about.js";
    document.head.appendChild(link);
    console.log("Prefetching about bundle");
  });
```

### Output

```js
Preloading image: /images/hero.webp
Preloading font: /fonts/inter.woff2
Prefetching about bundle   // on first hover
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 9. Debounce and Throttle for Performance

High-frequency events (scroll, resize, input, mousemove) fire dozens of times per second. Running expensive handlers on every event wastes CPU.

- **Debounce** — waits until the event stream pauses, then fires once (good for search-as-you-type).
- **Throttle** — fires at most once per interval regardless of how many events fire (good for scroll/resize).

```js
// Debounce: only fires after 300ms of silence
function debounce(fn, delay) {
  let timer;
  return function(...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}

// Throttle: fires at most once every 200ms
function throttle(fn, interval) {
  let lastTime = 0;
  return function(...args) {
    const now = Date.now();
    if (now - lastTime >= interval) {
      lastTime = now;
      fn.apply(this, args);
    }
  };
}

const handleSearch = debounce((query) => {
  console.log("Fetching results for:", query);
}, 300);

const handleScroll = throttle(() => {
  console.log("Scroll position:", window.scrollY);
}, 200);

// Simulate rapid input
handleSearch("j");
handleSearch("ja");
handleSearch("jav"); // only this one fires (300ms after last call)

// For more about debounce/throttle implementation, see 21-Utility-Functions
```

### Output

```js
Fetching results for: jav   // only after 300ms of silence
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 10. Virtual Scrolling and Windowing

Rendering thousands of DOM nodes at once is expensive. **Virtual scrolling** (windowing) renders only the items currently visible in the viewport, plus a small buffer. The total DOM node count stays constant regardless of list size.

Popular libraries: `react-window`, `react-virtual`, `@tanstack/virtual`.

```js
// Manual virtual scroll implementation concept
class VirtualList {
  constructor({ itemHeight, totalItems, containerHeight }) {
    this.itemHeight = itemHeight;
    this.totalItems = totalItems;
    this.containerHeight = containerHeight;
    this.scrollTop = 0;
  }

  get visibleRange() {
    const start = Math.floor(this.scrollTop / this.itemHeight);
    const visibleCount = Math.ceil(this.containerHeight / this.itemHeight);
    const end = Math.min(start + visibleCount + 1, this.totalItems);
    return { start, end };
  }

  get totalHeight() {
    return this.totalItems * this.itemHeight;
  }

  onScroll(scrollTop) {
    this.scrollTop = scrollTop;
    const { start, end } = this.visibleRange;
    console.log(`Rendering items ${start}–${end} of ${this.totalItems}`);
    return { start, end };
  }
}

const list = new VirtualList({ itemHeight: 40, totalItems: 100000, containerHeight: 600 });
list.onScroll(0);     // beginning
list.onScroll(4000);  // scrolled down 4000px
```

### Output

```js
Rendering items 0–16 of 100000
Rendering items 100–117 of 100000
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 11. Web Workers for CPU Tasks

Heavy computation (sorting large datasets, image processing, cryptography, compression) blocks the main thread and freezes the UI. **Web Workers** offload this work to a separate thread.

```js
// main.js
function runWorkerTask(data) {
  return new Promise((resolve, reject) => {
    const worker = new Worker("/workers/heavy-compute.js");
    worker.postMessage(data);
    worker.onmessage = (e) => {
      resolve(e.data);
      worker.terminate(); // clean up after use
    };
    worker.onerror = reject;
  });
}

runWorkerTask({ array: Array.from({ length: 500000 }, (_, i) => i) })
  .then(result => console.log("Sorted last element:", result.lastElement));

console.log("Main thread: UI stays responsive during worker computation");

// heavy-compute.js (worker file)
// self.onmessage = function({ data }) {
//   const sorted = data.array.slice().sort((a, b) => b - a); // reverse sort
//   self.postMessage({ lastElement: sorted[sorted.length - 1] });
// };
```

### Output

```js
Main thread: UI stays responsive during worker computation
Sorted last element: 0
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 12. Avoid Forced Synchronous Layout

A **Forced Synchronous Layout (FSL)** — also called layout thrashing — occurs when JavaScript reads a layout property immediately after writing a style. The browser is forced to calculate layout synchronously, blocking the current JavaScript task.

```js
// Bad — forced synchronous layout on every iteration
const boxes = document.querySelectorAll(".box");

boxes.forEach(box => {
  box.style.width = "100px";                     // write — layout dirty
  const height = box.offsetHeight;               // READ forces layout NOW
  box.style.height = height + 10 + "px";         // write again
  // Next iteration: another forced layout
});

// Good — separate all reads from all writes
const heights = [...boxes].map(box => box.offsetHeight); // all reads (one layout)
boxes.forEach((box, i) => {
  box.style.width  = "100px";
  box.style.height = heights[i] + 10 + "px";
}); // all writes (one layout at the end)

console.log("Layout thrashing avoided");
```

### Output

```js
Layout thrashing avoided
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 13. Batch DOM Reads and Writes

The `requestAnimationFrame` callback and the `fastdom` pattern provide a structured way to batch all DOM reads first, then all writes, preventing layout thrashing.

```js
// Using requestAnimationFrame to batch DOM work
function batchDOMWork(elements) {
  requestAnimationFrame(() => {
    // Phase 1: all reads (one layout calculation)
    const measurements = elements.map(el => ({
      el,
      width: el.offsetWidth,
      height: el.offsetHeight
    }));

    // Phase 2: all writes (browser batches into one reflow)
    measurements.forEach(({ el, width, height }) => {
      el.style.transform = `scale(${1 + width / 1000})`;
      el.style.opacity   = String(height > 50 ? 1 : 0.5);
    });

    console.log("DOM batch complete for", elements.length, "elements");
  });
}

// Simulate usage
const boxes = document.querySelectorAll(".box");
if (boxes.length) batchDOMWork([...boxes]);
```

### Output

```js
DOM batch complete for 12 elements   // depends on page
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 14. Event Delegation for Performance

Adding individual event listeners to hundreds of list items is expensive. **Event delegation** attaches a single listener to a common parent and uses `event.target` to identify which child was acted on.

```js
// Bad — 1000 listeners for 1000 items
function badApproach(items) {
  items.forEach(item => {
    item.addEventListener("click", (e) => {
      console.log("Clicked:", e.target.textContent);
    });
  });
}
// This creates 1000 listener objects in memory.

// Good — 1 listener on the parent
const list = document.getElementById("item-list");

list.addEventListener("click", (e) => {
  const item = e.target.closest("li"); // find the nearest <li>
  if (!item) return;
  console.log("Clicked:", item.dataset.id, item.textContent);
});

// Benefits:
// 1. Only 1 listener in memory regardless of list size
// 2. Automatically works for dynamically added items
// 3. Reducing GC pressure from hundreds of closures

console.log("Event delegation set up");
```

### Output

```js
Event delegation set up
// On click: Clicked: 42 Item 42
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 15. Caching Strategies

Effective caching avoids redundant network requests and CPU work. Multiple caching layers are available.

| Layer | Mechanism | Controlled By |
|---|---|---|
| HTTP cache | `Cache-Control`, `ETag` | Server headers |
| Service Worker cache | `Cache API` | Your JS code |
| Memory cache | `Map` / `WeakMap` in JS | Your JS code |
| localStorage / IndexedDB | Persistent client storage | Your JS code |

```js
// In-memory memoization (function-level cache)
function memoize(fn) {
  const cache = new Map();
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      console.log("Cache hit for:", key);
      return cache.get(key);
    }
    const result = fn(...args);
    cache.set(key, result);
    console.log("Cache miss — computed:", key);
    return result;
  };
}

function expensiveCalc(n) {
  // simulate heavy work
  let sum = 0;
  for (let i = 0; i <= n; i++) sum += i;
  return sum;
}

const cachedCalc = memoize(expensiveCalc);
console.log(cachedCalc(100));  // miss → computes
console.log(cachedCalc(100));  // hit → instant
console.log(cachedCalc(200));  // miss → computes

// For persistent caching with expiry, see 17-Storage-And-Caching
```

### Output

```js
Cache miss — computed: [100]
5050
Cache hit for: [100]
5050
Cache miss — computed: [200]
20100
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 16. Image Optimization

Images are often the largest resources on a page. Poor image handling is a leading cause of poor LCP scores.

```html
<!-- 1. Use modern formats: WebP is ~30% smaller than JPEG -->
<img src="photo.webp" alt="Product" />

<!-- 2. Responsive images with srcset — browser picks right size -->
<img
  src="photo-800.webp"
  srcset="photo-400.webp 400w, photo-800.webp 800w, photo-1200.webp 1200w"
  sizes="(max-width: 600px) 400px, (max-width: 1024px) 800px, 1200px"
  alt="Product"
/>

<!-- 3. Native lazy loading for off-screen images -->
<img src="below-fold.webp" loading="lazy" alt="Below fold" />

<!-- 4. Explicit width/height prevents CLS (layout shifts) -->
<img src="hero.webp" width="1200" height="630" alt="Hero" />
```

```js
// Checking image decode performance
const img = new Image();
img.src = "/images/hero.webp";

img.decode().then(() => {
  document.body.appendChild(img);
  console.log("Image decoded and appended without layout jank");
}).catch(err => {
  console.error("Image decode failed:", err);
});
```

### Output

```js
Image decoded and appended without layout jank
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 17. CSS Performance Impact on JS

CSS can indirectly affect JavaScript performance in several ways:

1. **Render-blocking CSS** delays the first paint — reduce critical CSS and load non-critical CSS asynchronously.
2. **Complex selectors** slow down style recalculations triggered by JS.
3. **Expensive properties** (`box-shadow`, `filter`, `border-radius` on large elements) slow paint.
4. **CSS animations on geometry properties** trigger reflow every frame — use `transform`/`opacity` instead.

```js
// Bad — JS reads a property that requires style recalculation
// after CSS class changes
const el = document.querySelector(".card");
el.classList.add("expanded");            // triggers style recalc
const computedHeight = el.offsetHeight;  // forces synchronous reflow

// Good — read before writing, or avoid measuring immediately after class changes
const heightBefore = el.offsetHeight;    // read first
el.classList.add("expanded");            // write

// CSS class toggle: prefer class-based animations over inline style manipulation
// Fewer specificity fights, easier to optimize with CSS transitions
el.classList.toggle("is-visible");

// Avoiding complex selectors that recalculate slowly
// Bad:  div > ul > li > a:nth-child(odd) span.highlight { }
// Good: .highlight { }

console.log("CSS-aware JS written");
```

### Output

```js
CSS-aware JS written
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 18. Memory and GC Pressure

Excessive object creation puts pressure on the garbage collector. When the GC runs, it can cause noticeable pauses (especially in older engines or on low-end devices). Reducing allocation frequency and using **object pooling** are effective mitigations.

```js
// Bad — allocating new objects every frame (60 allocations/second)
function updateParticles(particles) {
  return particles.map(p => ({
    x: p.x + p.vx,
    y: p.y + p.vy,
    vx: p.vx,
    vy: p.vy
  })); // creates 1 new object per particle per frame
}

// Good — mutate in place, avoid allocations
function updateParticlesInPlace(particles) {
  for (const p of particles) {
    p.x += p.vx;
    p.y += p.vy;
  }
  return particles;
}

const particles = Array.from({ length: 1000 }, (_, i) => ({
  x: i, y: i, vx: 0.5, vy: 0.3
}));

updateParticlesInPlace(particles);
console.log("Updated particle[0]:", particles[0].x.toFixed(1), particles[0].y.toFixed(1));

// For deep analysis of GC and memory, see 22-Memory-Management
```

### Output

```js
Updated particle[0]: 0.5 0.3
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 19. Service Worker for Offline and Cache

A **Service Worker** is a JavaScript file that runs in a separate thread and acts as a network proxy. It can intercept fetch requests and serve responses from a cache, enabling:

- Offline support
- Instant repeat-visit load times
- Granular caching strategies

```js
// Register a Service Worker from the main page
if ("serviceWorker" in navigator) {
  navigator.serviceWorker.register("/sw.js")
    .then(reg => console.log("SW registered:", reg.scope))
    .catch(err => console.error("SW registration failed:", err));
}

// sw.js (service worker file)
const CACHE_NAME = "app-v1";
const PRECACHE = ["/", "/app.js", "/styles.css", "/offline.html"];

// Install: cache critical assets
self.addEventListener("install", event => {
  event.waitUntil(
    caches.open(CACHE_NAME).then(cache => cache.addAll(PRECACHE))
  );
});

// Fetch: cache-first strategy for static assets
self.addEventListener("fetch", event => {
  event.respondWith(
    caches.match(event.request).then(cached => {
      if (cached) return cached;                  // serve from cache
      return fetch(event.request).then(response => { // fetch from network
        const toCache = response.clone();
        caches.open(CACHE_NAME).then(c => c.put(event.request, toCache));
        return response;
      });
    }).catch(() => caches.match("/offline.html")) // fallback when offline
  );
});
```

### Output

```js
SW registered: https://example.com/
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 20. Summary

| Topic | Key Point |
|---|---|
| Core Web Vitals | LCP ≤ 2.5s, INP ≤ 200ms, CLS ≤ 0.1 |
| Measuring | `performance.now()` for benchmarks; Lighthouse for audits |
| Main thread | Keep tasks < 50ms; yield with `setTimeout(fn, 0)` or `scheduler.yield()` |
| Code splitting | Dynamic `import()` loads chunks on demand |
| Tree shaking | ES module named exports allow dead code elimination |
| Bundle size | Minify + gzip/Brotli; audit with bundlephobia; avoid large imports |
| Lazy loading | `loading="lazy"` for images; dynamic `import()` for modules |
| Preload | Fetch critical resources early; prefetch for next page; preconnect for 3rd-party origins |
| Debounce/Throttle | Reduce high-frequency event handler invocations |
| Virtual scrolling | Render only visible items — constant DOM size for any list length |
| Web Workers | Offload CPU-intensive work off the main thread |
| Avoid FSL | Read layout properties before writing styles, not after |
| Batch DOM | Group all reads then all writes inside `requestAnimationFrame` |
| Event delegation | One listener on parent vs. one per child |
| Caching | Memoize functions; use HTTP cache + Service Worker + IndexedDB layers |
| Images | Use WebP, `srcset`, `loading="lazy"`, explicit dimensions for zero CLS |
| CSS on JS | Avoid triggering reflow; prefer `transform`/`opacity` for animations |
| Memory/GC | Mutate in place; use object pools for high-frequency allocations |
| Service Worker | Network proxy for offline support and cache-first strategies |

---

## Final Notes

Performance optimization is a discipline, not a single technique. Start with measurement — you cannot fix what you cannot see. Use Lighthouse to identify which Core Web Vital is failing, then use DevTools to find the specific cause. In most applications, the biggest gains come from: loading only the JavaScript needed for the current page (code splitting and lazy loading), keeping the main thread free from long tasks (chunking, Web Workers), and avoiding the read-after-write pattern that causes layout thrashing. Image optimization and a well-configured Service Worker can often halve load times on repeat visits with minimal code changes. Apply these techniques where profiling shows they matter, not everywhere — premature optimization adds complexity without measurable benefit.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
