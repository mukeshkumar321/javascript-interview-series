# Browser APIs — Tricky Output Questions

## Table of Contents
1. [Fetch Questions](#1-fetch-questions)
2. [Web Worker Questions](#2-web-worker-questions)
3. [Service Worker Questions](#3-service-worker-questions)
4. [Intersection Observer Questions](#4-intersection-observer-questions)
5. [requestAnimationFrame Questions](#5-requestanimationframe-questions)
6. [History API Questions](#6-history-api-questions)
7. [Advanced Browser API Questions](#7-advanced-browser-api-questions)

---

## 1. Fetch Questions

---

### Q1. What will be the output if the server returns HTTP 404?

```js
fetch('/api/missing')
  .then(response => {
    console.log('resolved:', response.ok);
    console.log('status:', response.status);
    if (!response.ok) {
      throw new Error(`HTTP error ${response.status}`);
    }
    return response.json();
  })
  .then(data => {
    console.log('data:', data);
  })
  .catch(err => {
    console.log('caught:', err.message);
  });
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
resolved: false
status: 404
caught: HTTP error 404
```

### Explanation
A critical Fetch gotcha: the `fetch` promise only **rejects** on network failure (no connection, DNS error, etc.). An HTTP 404 or 500 response is still a valid response from the network — the promise **resolves** successfully. `response.ok` is `false` for any status outside 200–299. You must manually check `response.ok` and throw if needed to enter the `catch` handler.

</details>

---

### Q2. What will be the output of this async/await fetch?

```js
async function getData() {
  try {
    const res = await fetch('/api/data');
    const json = await res.json();
    console.log('got data');
    return json;
  } catch (e) {
    console.log('error:', e.message);
  }
}

const result = getData();
console.log('type:', typeof result);
console.log('is promise:', result instanceof Promise);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
type: object
is promise: true
got data
```

### Explanation
An `async` function **always returns a Promise**, even if you don't explicitly `return` a Promise. The call `getData()` returns a Promise immediately — before any `await` resolves. The synchronous code after the call runs first, logging `type: object` and `is promise: true`. The `async` function body continues asynchronously, and `'got data'` is logged after the fetch and JSON parsing complete.

</details>

---

### Q3. What will be the output?

```js
const controller = new AbortController();

fetch('/api/slow', { signal: controller.signal })
  .then(res => res.json())
  .then(data => console.log('success:', data))
  .catch(err => {
    console.log('name:', err.name);
    console.log('message:', err.message);
  });

// Abort immediately
controller.abort();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
name: AbortError
message: The operation was aborted.
```

### Explanation
`AbortController` allows you to cancel a fetch request. When `controller.abort()` is called, the fetch promise rejects with a `DOMException` where `name` is `"AbortError"`. Unlike a network error or server error, an abort produces this specific named error type so you can distinguish it: `if (err.name === 'AbortError') { /* user cancelled */ }`.

</details>

---

### Q4. What will be the output?

```js
async function run() {
  const [r1, r2] = await Promise.all([
    fetch('/api/users'),
    fetch('/api/posts')
  ]);

  const [users, posts] = await Promise.all([
    r1.json(),
    r2.json()
  ]);

  console.log('users:', users.length);
  console.log('posts:', posts.length);
}

run();
console.log('after run()');
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
after run()
users: 10
posts: 50
```

### Explanation
`run()` is an async function — it returns a Promise immediately. `console.log('after run()')` executes synchronously before any of the awaited operations complete. Both fetches run in parallel via `Promise.all` (not sequentially, which would be slower). The second `Promise.all` parses both JSON bodies in parallel. Results log after all network and parsing is done.

</details>

---

## 2. Web Worker Questions

---

### Q5. What will be the output order?

```js
// main.js
const worker = new Worker('worker.js');

console.log('1: before postMessage');

worker.postMessage('ping');

console.log('2: after postMessage');

worker.onmessage = (e) => {
  console.log('3: main received:', e.data);
};

// worker.js:
// self.onmessage = (e) => {
//   self.postMessage('pong');
// };
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
1: before postMessage
2: after postMessage
3: main received: pong
```

### Explanation
`postMessage` is non-blocking — it queues the message for the worker but does not wait for a response. The main thread continues synchronously, logging `1` and `2`. The worker processes the message in its own thread, calls `postMessage('pong')`, and that message arrives in the main thread's message queue. The `onmessage` handler fires asynchronously, logging `3`. Worker communication is always asynchronous from the main thread's perspective.

</details>

---

### Q6. What is wrong with this code and what will happen?

```js
const worker = new Worker('worker.js');

worker.onmessage = (e) => {
  console.log('result:', e.data);
};

// Try to access DOM from worker (inside worker.js):
// document.getElementById('output').textContent = 'done'; // ???
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
// Inside worker.js this throws:
// ReferenceError: document is not defined
```

### Explanation
Web Workers run in a completely separate global context (`DedicatedWorkerGlobalScope`) that does NOT have access to `document`, `window`, or any DOM APIs. Attempting to access `document` inside a worker throws a `ReferenceError`. Workers can use: `fetch`, `XMLHttpRequest`, `console`, `setTimeout`, `setInterval`, `IndexedDB`, `crypto`, `postMessage`, `importScripts`, and `WebSockets`. For UI updates, the worker must `postMessage` the result back to the main thread.

</details>

---

## 3. Service Worker Questions

---

### Q7. What will be logged and in what order when a page loads for the first time with a service worker?

```js
// main.js
console.log('1: page script starts');

navigator.serviceWorker.register('/sw.js')
  .then(reg => console.log('2: SW registered'))
  .catch(err => console.log('SW failed'));

console.log('3: after register()');

navigator.serviceWorker.ready.then(() => {
  console.log('4: SW is active and ready');
});
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
1: page script starts
3: after register()
2: SW registered
4: SW is active and ready
```

### Explanation
Synchronous code runs first: `1` and `3`. `register()` returns a Promise that resolves asynchronously once registration succeeds — `2` logs as a microtask. `navigator.serviceWorker.ready` is a Promise that resolves once an active service worker is controlling the page — this may take a bit longer (install + activate phases), so `4` logs last. On a first visit the page is not controlled by the SW until the next navigation, but `ready` resolves once the SW is activated.

</details>

---

### Q8. What is the correct service worker fetch handler for a cache-first strategy?

```js
// sw.js — which implementation is correct for "cache first, then network"?

// Option A:
self.addEventListener('fetch', (e) => {
  e.respondWith(
    caches.match(e.request)
      .then(cached => cached || fetch(e.request))
  );
});

// Option B:
self.addEventListener('fetch', (e) => {
  fetch(e.request)
    .then(response => {
      caches.open('v1').then(cache => cache.put(e.request, response));
      return response;
    });
  // e.respondWith not called
});
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Option A is correct.
Option B has two bugs: e.respondWith() is never called, and the response is consumed before caching.
```

### Explanation
Option A correctly implements cache-first: check the cache, return the cached response if found, otherwise fall back to the network. Option B has two bugs: (1) `e.respondWith()` is never called, so the browser falls through to its default fetch behavior. (2) `fetch(e.request)` returns the response body which can only be consumed once — passing it to both `cache.put` and `return response` would require `response.clone()`. The correct network-then-cache pattern needs `const clone = response.clone(); cache.put(req, clone); return response;`.

</details>

---

## 4. Intersection Observer Questions

---

### Q9. What will be the output?

```js
// Assume #box is currently NOT visible in the viewport

const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    console.log('isIntersecting:', entry.isIntersecting);
  });
}, { threshold: 0 });

observer.observe(document.getElementById('box'));

console.log('observer attached');
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
observer attached
isIntersecting: false
```

### Explanation
`IntersectionObserver` fires its callback **immediately upon observation** with the initial intersection state, even if nothing has changed. The synchronous `console.log('observer attached')` runs first. Then the observer callback fires asynchronously with the element's current state — since `#box` is not visible, `isIntersecting` is `false`. This initial notification allows you to set initial state without needing a separate check.

</details>

---

### Q10. What will be the output if `threshold` is set to `[0, 0.5, 1]`?

```js
const observer = new IntersectionObserver((entries) => {
  console.log('ratio:', entries[0].intersectionRatio.toFixed(1));
}, { threshold: [0, 0.5, 1] });

observer.observe(document.getElementById('img'));

// Simulate: element goes from fully hidden to fully visible
// Crossing 0% -> fires at 0
// Crossing 50% -> fires at 0.5
// Crossing 100% -> fires at 1
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
ratio: 0.0
ratio: 0.5
ratio: 1.0
```

### Explanation
The `threshold` array defines the intersection ratios at which the callback fires. With `[0, 0.5, 1]`, the callback fires three times as the element scrolls from hidden to fully visible: once at crossing 0% visibility, once at 50%, and once at 100%. This allows granular control — for example, you could trigger a fade-in at 10% visible and a full-quality image load at 100% visible.

</details>

---

## 5. requestAnimationFrame Questions

---

### Q11. What will be the output order?

```js
console.log('1');

setTimeout(() => console.log('2: setTimeout'), 0);

requestAnimationFrame(() => console.log('3: rAF'));

Promise.resolve().then(() => console.log('4: microtask'));

console.log('5');
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
1
5
4: microtask
3: rAF
2: setTimeout
```

### Explanation
Execution order: synchronous code runs first (`1`, `5`). Microtasks (Promise callbacks) drain before any macrotask (`4`). `requestAnimationFrame` callbacks run before the next paint — typically before `setTimeout(fn, 0)` macrotasks. `setTimeout(fn, 0)` has a minimum delay and runs last as a macrotask. Note: the exact order of rAF vs setTimeout can vary across browsers and environments, but in a browser rAF is tied to the rendering cycle and generally fires before a 0ms timeout.

</details>

---

### Q12. What is the problem with this animation code?

```js
function badAnimation() {
  let x = 0;
  setInterval(() => {
    x++;
    document.getElementById('box').style.left = x + 'px';
  }, 16);
}

function goodAnimation() {
  let x = 0;
  function frame() {
    x++;
    document.getElementById('box').style.left = x + 'px';
    if (x < 300) requestAnimationFrame(frame);
  }
  requestAnimationFrame(frame);
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
// badAnimation: jittery, may cause missed or doubled frames
// goodAnimation: smooth, 60fps synchronized
```

### Explanation
`setInterval(fn, 16)` problems: (1) 16ms is an approximation — the actual browser refresh rate may differ (e.g. 90Hz, 120Hz screens). (2) The interval does not pause when the tab is hidden, wasting CPU. (3) The timer is not synchronized with the browser's repaint cycle, causing visual tearing or jitter. `requestAnimationFrame` automatically: runs before each paint (synchronized with display refresh), pauses when the tab is hidden (saving battery), and receives a high-precision timestamp for smooth time-based animations.

</details>

---

## 6. History API Questions

---

### Q13. What will be the output?

```js
console.log('initial path:', location.pathname);

history.pushState({ page: 1 }, '', '/page1');
console.log('after push 1:', location.pathname);
console.log('state 1:', history.state.page);

history.pushState({ page: 2 }, '', '/page2');
console.log('after push 2:', location.pathname);

history.replaceState({ page: 99 }, '', '/page2');
console.log('after replace:', history.state.page);

console.log('length:', history.length);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
initial path: /
after push 1: /page1
state 1: 1
after push 2: /page2
after replace: 99
length: 3
```

### Explanation
`pushState` adds a new entry to the history stack and changes the URL shown in the address bar without reloading. `replaceState` modifies the current entry without adding a new one. After two `pushState` calls and one `replaceState`, the history stack has 3 entries: the original `/`, `/page1`, and `/page2` (modified in place). `history.state` always returns the state of the **current** entry, which after `replaceState` is `{ page: 99 }`.

</details>

---

### Q14. What will be the output?

```js
window.addEventListener('popstate', (e) => {
  console.log('popstate fired, state:', e.state);
});

history.pushState({ step: 'A' }, '', '/a');
history.pushState({ step: 'B' }, '', '/b');
history.pushState({ step: 'C' }, '', '/c');

history.go(-2); // go back 2 steps
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
popstate fired, state: { step: 'A' }
```

### Explanation
`history.go(-2)` navigates two steps back in the session history. Starting from `/c` (state C), going back 2 steps lands on `/a` (state A). The `popstate` event fires when the active history entry changes due to `back()`, `forward()`, or `go()`. Critically, `pushState` and `replaceState` do **NOT** fire `popstate` — only user navigation (browser back/forward buttons) and programmatic `history.go()` / `back()` / `forward()` do.

</details>

---

## 7. Advanced Browser API Questions

---

### Q15. What will be the output?

```js
const url = new URL('https://shop.example.com/items?color=red&size=L&size=XL#top');

console.log(url.origin);
console.log(url.pathname);
console.log(url.searchParams.get('color'));
console.log(url.searchParams.getAll('size'));
console.log(url.searchParams.has('price'));
console.log(url.hash);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
https://shop.example.com
/items
red
[ 'L', 'XL' ]
false
#top
```

### Explanation
`url.origin` combines protocol and host. `searchParams.get()` returns the first value for a key. `searchParams.getAll()` returns an array of all values for a key — useful for repeated parameters like `size=L&size=XL`. `searchParams.has()` returns a boolean. The `hash` includes the `#` prefix. All these are read-only unless you mutate through the `searchParams` or `url` object properties directly.

</details>

---

### Q16. What will be the output?

```js
const t0 = performance.now();

// Simulate some work
let sum = 0;
for (let i = 0; i < 1e7; i++) sum += i;

const t1 = performance.now();

console.log('sum:', sum);
console.log('elapsed > 0:', (t1 - t0) > 0);
console.log('elapsed is integer:', Number.isInteger(t1 - t0));
console.log('Date.now() precision (ms):', typeof Date.now() === 'number');
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
sum: 49999995000000
elapsed > 0: true
elapsed is integer: false
Date.now() precision (ms): true
```

### Explanation
`performance.now()` returns a floating-point number with sub-millisecond precision (e.g. `12.345678`). The difference between two calls is therefore a float, not an integer. `Date.now()` returns an integer millisecond timestamp — it has only millisecond precision and is affected by system clock adjustments. For profiling code, `performance.now()` is the right choice. Browsers may round `performance.now()` values slightly for security (to prevent Spectre-style attacks), but it still offers microsecond-range precision.

</details>

---

### Q17. What will be the output?

```js
// Assume two tabs open on the same origin

// Tab 1:
const ch = new BroadcastChannel('sync');

ch.onmessage = (e) => {
  console.log('Tab1 got:', e.data.type);
};

ch.postMessage({ type: 'HEARTBEAT' });

// Tab 2 (same origin):
const ch2 = new BroadcastChannel('sync');

ch2.onmessage = (e) => {
  console.log('Tab2 got:', e.data.type);
};

// What does Tab 1 see? What does Tab 2 see?
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Tab 2 sees: Tab2 got: HEARTBEAT
Tab 1 does NOT receive its own message
```

### Explanation
`BroadcastChannel` sends messages to **all other** contexts connected to the same channel name on the same origin — but NOT to the sender itself. Tab 1 posts `HEARTBEAT` — Tab 2's `onmessage` fires. Tab 1's own `onmessage` does not fire. This is by design: it prevents a tab from receiving its own messages in an infinite loop. If you need to react to your own broadcast, handle the action locally before calling `postMessage`.

</details>

---

### Q18. What will be the output and why is this code dangerous?

```js
async function loadUserData(userId) {
  const res = await fetch(`/api/users/${userId}`);
  const data = await res.json();
  console.log('body consumed once:', data.name);

  // Try to read body again
  try {
    const data2 = await res.json();
    console.log('second read:', data2.name);
  } catch (e) {
    console.log('error:', e.message);
  }
}

loadUserData(1);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
body consumed once: Alice
error: Failed to execute 'json' on 'Response': body stream already read
```

### Explanation
A `Response` body is a **readable stream** that can only be consumed once. After `res.json()` is called and awaited, the stream is fully read and closed. Calling `.json()` (or `.text()`, `.blob()`, etc.) a second time throws a `TypeError` because the stream is already consumed. To read the body multiple times, use `res.clone()` before reading: `const clone = res.clone(); const data = await res.json(); const data2 = await clone.json();`.

</details>

---

## Final Tips

- `fetch` never rejects on HTTP error status (4xx, 5xx) — always check `response.ok` or `response.status`.
- `async` functions always return a Promise, even if they only do synchronous work.
- Abort a fetch with `AbortController` — the rejection has `err.name === 'AbortError'`.
- Web Workers cannot access `document` or `window` — any DOM manipulation must be `postMessage`'d back to the main thread.
- `BroadcastChannel` does not deliver messages to the sender — only to other contexts.
- A `Response` body stream can only be read once — use `.clone()` to read it multiple times.
- `pushState` / `replaceState` do not fire `popstate` — only navigation via back/forward/`go()` does.
- `performance.now()` returns a float with sub-millisecond precision; `Date.now()` returns an integer in milliseconds.
- `IntersectionObserver` fires immediately on `observe()` with the current intersection state.
- `requestAnimationFrame` pauses when the tab is hidden and is synchronized with the display refresh rate — `setInterval` is not.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
