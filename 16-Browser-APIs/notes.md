# Browser APIs in JavaScript

## Table of Contents

1. [Fetch API](#1-fetch-api)
2. [Fetch vs XMLHttpRequest](#2-fetch-vs-xmlhttprequest)
3. [Headers, Request, Response Objects](#3-headers-request-response-objects)
4. [Web Workers](#4-web-workers)
5. [Service Workers](#5-service-workers)
6. [Web Worker vs Service Worker](#6-web-worker-vs-service-worker)
7. [Intersection Observer](#7-intersection-observer)
8. [MutationObserver (brief)](#8-mutationobserver-brief)
9. [ResizeObserver](#9-resizeobserver)
10. [requestAnimationFrame](#10-requestanimationframe)
11. [requestIdleCallback](#11-requestidlecallback)
12. [Geolocation API](#12-geolocation-api)
13. [Clipboard API](#13-clipboard-api)
14. [History API](#14-history-api)
15. [URL API](#15-url-api)
16. [Broadcast Channel API](#16-broadcast-channel-api)
17. [Web Notifications API](#17-web-notifications-api)
18. [Performance API](#18-performance-api)
19. [console Methods](#19-console-methods)
20. [Summary](#20-summary)

---

## 1. Fetch API

The Fetch API is the modern, Promise-based way to make HTTP requests. `fetch()` returns a `Promise` that resolves to a `Response` object. The promise only rejects on network failure — a 4xx or 5xx HTTP status does **not** reject the promise, so you must check `response.ok`.

```js
fetch('https://api.example.com/users/1')
  .then(response => {
    console.log(response.status);   // 200
    console.log(response.ok);       // true  (status 200–299)
    return response.json();         // another Promise
  })
  .then(data => {
    console.log(data.name);         // "Alice"
  })
  .catch(err => {
    console.log('Network error:', err.message);
  });

// POST request
fetch('https://api.example.com/users', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ name: 'Bob' })
});
```

### Output

```js
200
true
"Alice"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 2. Fetch vs XMLHttpRequest

| Feature | Fetch | XMLHttpRequest |
|---|---|---|
| API style | Promise-based | Callback-based |
| Streaming | Yes (`response.body`) | No |
| Progress events | No (use streams) | Yes (`onprogress`) |
| Abort | `AbortController` | `xhr.abort()` |
| Cookies | `credentials` option | `withCredentials` |
| Service Worker | Yes (interceptable) | No |
| JSONP | No | Workaround needed |
| Error handling | Manual `response.ok` check | `onerror` / status codes |
| Browser support | Modern browsers | All browsers |

```js
// Equivalent requests

// Fetch
const res = await fetch('/api/data');
if (!res.ok) throw new Error(`HTTP ${res.status}`);
const data = await res.json();

// XMLHttpRequest
const xhr = new XMLHttpRequest();
xhr.open('GET', '/api/data');
xhr.onload = () => {
  if (xhr.status >= 200 && xhr.status < 300) {
    const data = JSON.parse(xhr.responseText);
  }
};
xhr.onerror = () => console.log('network error');
xhr.send();
```

### Output

```js
// Both retrieve the same data — Fetch is cleaner and Promise-based
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 3. Headers, Request, Response Objects

`Headers`, `Request`, and `Response` are the building blocks of the Fetch API. You can construct them explicitly for full control.

```js
// Headers — iterable key-value store
const headers = new Headers({
  'Content-Type': 'application/json',
  'X-Auth': 'token123'
});

console.log(headers.get('content-type')); // "application/json" (case-insensitive)
console.log(headers.has('x-auth'));       // true

headers.set('X-Auth', 'newtoken');
headers.delete('X-Auth');

// Request — full request descriptor
const req = new Request('https://api.example.com/data', {
  method: 'GET',
  headers,
  credentials: 'include'
});

console.log(req.url);     // "https://api.example.com/data"
console.log(req.method);  // "GET"

// Response — returned by fetch()
const mockResponse = new Response(JSON.stringify({ ok: true }), {
  status: 200,
  headers: { 'Content-Type': 'application/json' }
});

console.log(mockResponse.status); // 200
console.log(mockResponse.ok);     // true
```

### Output

```js
"application/json"
true
"https://api.example.com/data"
"GET"
200
true
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 4. Web Workers

A **Web Worker** runs JavaScript in a background thread, separate from the main UI thread. It cannot access the DOM but can perform CPU-intensive computations. Communication between the main thread and worker uses `postMessage` / `onmessage`.

```js
// main.js
const worker = new Worker('worker.js');

worker.postMessage({ num: 40 });

worker.onmessage = (e) => {
  console.log('result:', e.data); // result: 102334155
};

worker.onerror = (e) => {
  console.log('error:', e.message);
};

// To stop the worker:
// worker.terminate();

// ---- worker.js ----
// function fib(n) { return n <= 1 ? n : fib(n-1) + fib(n-2); }
// self.onmessage = (e) => {
//   self.postMessage(fib(e.data.num));
// };
```

### Output

```js
"result: 102334155"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 5. Service Workers

A **Service Worker** is a special worker that acts as a **network proxy** between the browser and the network. It runs in a separate thread with no DOM access and has a lifecycle (install, activate, fetch). Primary use cases: offline caching, background sync, push notifications.

```js
// Register a service worker
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js')
    .then(reg => {
      console.log('SW registered, scope:', reg.scope);
    })
    .catch(err => {
      console.log('SW registration failed:', err);
    });
}

// ---- sw.js ----
// self.addEventListener('install', (e) => {
//   e.waitUntil(
//     caches.open('v1').then(cache => cache.addAll(['/index.html', '/app.js']))
//   );
// });
//
// self.addEventListener('fetch', (e) => {
//   e.respondWith(
//     caches.match(e.request).then(cached => cached || fetch(e.request))
//   );
// });
```

### Output

```js
"SW registered, scope: https://example.com/"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 6. Web Worker vs Service Worker

| Feature | Web Worker | Service Worker |
|---|---|---|
| Purpose | Offload CPU work | Network proxy / caching |
| Lifecycle | Tied to creating page | Independent; persists |
| Scope | One page | Origin-wide |
| DOM access | No | No |
| Network access | `fetch()` | Full intercept via `fetch` event |
| Multiple instances | Yes | One per scope (shared) |
| Offline support | No | Yes (Cache API) |
| Push notifications | No | Yes |
| Background sync | No | Yes |
| Created by | `new Worker(url)` | `navigator.serviceWorker.register()` |

```js
// Web Worker — direct message channel
const w = new Worker('heavy.js');
w.postMessage('start');
w.onmessage = e => console.log(e.data);

// Service Worker — registered once, controls all pages in scope
navigator.serviceWorker.register('/sw.js');
// From then on, ALL fetch requests on the origin pass through sw.js
```

### Output

```js
// Web Worker result from heavy.js computation
// Service Worker intercepts all network requests
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 7. Intersection Observer

`IntersectionObserver` fires a callback when a target element enters or leaves a specified viewport (or ancestor element). Common uses: lazy loading images, infinite scroll, triggering animations on scroll.

```js
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    console.log('target:', entry.target.id);
    console.log('isIntersecting:', entry.isIntersecting);
    console.log('intersectionRatio:', entry.intersectionRatio);

    if (entry.isIntersecting) {
      // Lazy load image
      entry.target.src = entry.target.dataset.src;
      observer.unobserve(entry.target); // stop watching once loaded
    }
  });
}, {
  root: null,          // viewport
  rootMargin: '0px',
  threshold: 0.1       // fire when 10% visible
});

// Observe all lazy images
document.querySelectorAll('img[data-src]').forEach(img => {
  observer.observe(img);
});
```

### Output

```js
"target: hero-image"
"isIntersecting: true"
"intersectionRatio: 0.5"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 8. MutationObserver (brief)

`MutationObserver` watches for DOM tree changes and fires a callback asynchronously. Covered in full detail in the DOM Manipulation section; here is a quick reference.

```js
const observer = new MutationObserver(mutations => {
  mutations.forEach(m => console.log(m.type, m.addedNodes.length));
});

observer.observe(document.body, { childList: true, subtree: true });

// When any element is added to the body or its descendants, the callback fires
document.body.appendChild(document.createElement('div')); // "childList" 1

observer.disconnect(); // stop observing
```

### Output

```js
"childList" 1
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 9. ResizeObserver

`ResizeObserver` fires when an element's size changes. Unlike `window.resize`, it works per-element and does not cause infinite loops if you change size inside the callback (the browser skips infinite cycles).

```js
const box = document.getElementById('box');

const ro = new ResizeObserver(entries => {
  for (const entry of entries) {
    const { width, height } = entry.contentRect;
    console.log(`box resized: ${width} x ${height}`);
  }
});

ro.observe(box);

// When box changes size (e.g. user resizes window or JS changes its dimensions):
// box resized: 400 x 200

ro.unobserve(box); // stop watching this element
// ro.disconnect();  // stop watching all elements
```

### Output

```js
"box resized: 400 x 200"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 10. requestAnimationFrame

`requestAnimationFrame` (rAF) schedules a callback to run **before the next browser repaint**, synchronized with the display refresh rate (typically 60fps = ~16ms intervals). Use it for smooth animations instead of `setTimeout`.

```js
let start = null;
const box = document.getElementById('box');

function animate(timestamp) {
  if (!start) start = timestamp;
  const elapsed = timestamp - start;

  box.style.transform = `translateX(${Math.min(elapsed / 5, 200)}px)`;

  if (elapsed < 1000) {
    requestAnimationFrame(animate); // schedule next frame
  } else {
    console.log('animation done');
  }
}

const id = requestAnimationFrame(animate);

// To cancel before it fires:
// cancelAnimationFrame(id);
```

### Output

```js
// (runs ~60 times per second for 1 second)
"animation done"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 11. requestIdleCallback

`requestIdleCallback` schedules work during the browser's **idle periods** — when it has nothing urgent to do. The callback receives a `deadline` object with `timeRemaining()` so you can break work into chunks.

```js
function processQueue(deadline) {
  while (deadline.timeRemaining() > 0 && tasks.length > 0) {
    const task = tasks.shift();
    task();
  }

  if (tasks.length > 0) {
    requestIdleCallback(processQueue); // continue in next idle period
  }
}

const tasks = [
  () => console.log('task 1'),
  () => console.log('task 2'),
  () => console.log('task 3')
];

requestIdleCallback(processQueue, { timeout: 2000 }); // max wait 2s

// Output (when browser is idle):
// task 1
// task 2
// task 3
```

### Output

```js
"task 1"
"task 2"
"task 3"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 12. Geolocation API

The `Geolocation` API provides access to the device's physical location. It requires explicit user permission and is only available in secure contexts (HTTPS).

```js
if ('geolocation' in navigator) {
  navigator.geolocation.getCurrentPosition(
    (position) => {
      console.log('lat:', position.coords.latitude);
      console.log('lon:', position.coords.longitude);
      console.log('accuracy (m):', position.coords.accuracy);
    },
    (error) => {
      console.log('error code:', error.code);
      // 1 = PERMISSION_DENIED
      // 2 = POSITION_UNAVAILABLE
      // 3 = TIMEOUT
    },
    { timeout: 5000, maximumAge: 60000, enableHighAccuracy: true }
  );

  // Continuous watching
  const watchId = navigator.geolocation.watchPosition(handler);
  // navigator.geolocation.clearWatch(watchId);
}
```

### Output

```js
"lat: 51.5074"
"lon: -0.1278"
"accuracy (m): 20"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 13. Clipboard API

The modern `navigator.clipboard` API provides async read/write access to the system clipboard. Requires user permission for reading; writing is generally allowed during user gestures.

```js
// Write to clipboard
async function copyText(text) {
  try {
    await navigator.clipboard.writeText(text);
    console.log('copied!');
  } catch (err) {
    console.log('copy failed:', err.message);
  }
}

// Read from clipboard
async function pasteText() {
  try {
    const text = await navigator.clipboard.readText();
    console.log('clipboard:', text);
  } catch (err) {
    console.log('read failed:', err.message); // permission denied
  }
}

copyText('Hello World').then(() => pasteText());
```

### Output

```js
"copied!"
"clipboard: Hello World"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 14. History API

The History API lets you manipulate the browser's session history without full page reloads. Essential for Single Page Applications (SPAs).

| Method | Description |
|---|---|
| `history.pushState(state, title, url)` | Add new entry; changes URL, no reload |
| `history.replaceState(state, title, url)` | Replace current entry |
| `history.back()` | Go back one page |
| `history.forward()` | Go forward one page |
| `history.go(n)` | Go n steps (negative = back) |
| `popstate` event | Fires when navigating via back/forward |

```js
// SPA navigation
history.pushState({ page: 'about' }, '', '/about');
console.log(location.pathname);  // "/about"
console.log(history.state);      // { page: "about" }

history.pushState({ page: 'contact' }, '', '/contact');
console.log(location.pathname);  // "/contact"

// Handle browser back button
window.addEventListener('popstate', (e) => {
  console.log('navigated to state:', e.state);
  // e.state = { page: 'about' }
});

history.back(); // triggers popstate
```

### Output

```js
"/about"
{ page: "about" }
"/contact"
"navigated to state: { page: 'about' }"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 15. URL API

The `URL` constructor parses and manipulates URLs. `URLSearchParams` provides an easy way to work with query strings.

```js
const url = new URL('https://example.com/search?q=hello&page=2#results');

console.log(url.hostname);    // "example.com"
console.log(url.pathname);    // "/search"
console.log(url.hash);        // "#results"
console.log(url.search);      // "?q=hello&page=2"

const params = url.searchParams;
console.log(params.get('q'));     // "hello"
console.log(params.get('page'));  // "2"

params.set('page', '3');
params.append('lang', 'en');
console.log(url.toString());
// "https://example.com/search?q=hello&page=3&lang=en#results"
```

### Output

```js
"example.com"
"/search"
"#results"
"?q=hello&page=2"
"hello"
"2"
"https://example.com/search?q=hello&page=3&lang=en#results"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 16. Broadcast Channel API

`BroadcastChannel` enables communication between multiple browsing contexts (tabs, windows, iframes) from the same origin. All contexts on the same named channel receive messages.

```js
// In Tab 1
const channel = new BroadcastChannel('app-channel');

channel.onmessage = (e) => {
  console.log('Tab 1 received:', e.data);
};

// In Tab 2
const channel2 = new BroadcastChannel('app-channel');
channel2.postMessage({ type: 'USER_LOGGED_IN', user: 'alice' });

// Tab 1 output:
// Tab 1 received: { type: 'USER_LOGGED_IN', user: 'alice' }

// Close the channel when done
channel.close();
```

### Output

```js
"Tab 1 received: { type: 'USER_LOGGED_IN', user: 'alice' }"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 17. Web Notifications API

The Notifications API displays system-level notifications outside the browser tab. Requires explicit user permission.

```js
async function showNotification() {
  const permission = await Notification.requestPermission();
  console.log('permission:', permission); // "granted", "denied", "default"

  if (permission === 'granted') {
    const notification = new Notification('New Message', {
      body: 'You have a new message from Alice',
      icon: '/avatar.png',
      tag: 'msg-1'  // replaces previous notification with same tag
    });

    notification.onclick = () => {
      console.log('notification clicked');
      window.focus();
    };

    // Auto-close after 5 seconds
    setTimeout(() => notification.close(), 5000);
  }
}

showNotification();
```

### Output

```js
"permission: granted"
// System notification appears
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 18. Performance API

The Performance API gives fine-grained timing information. `performance.now()` returns a high-resolution timestamp (sub-millisecond precision). Marks and measures let you profile specific code sections.

```js
// High-resolution timing
const t0 = performance.now();
for (let i = 0; i < 1e6; i++) {} // some work
const t1 = performance.now();
console.log(`Loop took ${(t1 - t0).toFixed(2)}ms`);

// Custom marks and measures
performance.mark('start-parse');
// ... parse some data ...
performance.mark('end-parse');

performance.measure('parse-duration', 'start-parse', 'end-parse');

const [measure] = performance.getEntriesByName('parse-duration');
console.log(`Parse: ${measure.duration.toFixed(2)}ms`);

// Navigation timing
const nav = performance.getEntriesByType('navigation')[0];
console.log('DOM interactive:', nav.domInteractive);
console.log('Load event:', nav.loadEventEnd);
```

### Output

```js
"Loop took 2.45ms"
"Parse: 0.83ms"
"DOM interactive: 312"
"Load event: 450"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 19. console Methods

Beyond `console.log`, the browser console exposes several useful debugging methods.

```js
// Timing
console.time('fetch');
// ... some async operation ...
console.timeEnd('fetch'); // "fetch: 45.23ms"

// Grouping
console.group('User data');
console.log('name:', 'Alice');
console.log('age:', 30);
console.groupEnd();

// Table — renders arrays/objects as a table
const users = [
  { name: 'Alice', age: 30 },
  { name: 'Bob',   age: 25 }
];
console.table(users);

// Assertion — only logs if false
console.assert(1 === 2, 'Math is broken'); // "Assertion failed: Math is broken"
console.assert(1 === 1, 'This will not print'); // silent

// Counting
console.count('loop'); // loop: 1
console.count('loop'); // loop: 2
console.countReset('loop');
console.count('loop'); // loop: 1

// Stack trace
console.trace('where am I?');
```

### Output

```js
"fetch: 45.23ms"
// Collapsed group "User data" with name and age
// Rendered table with name/age columns
"Assertion failed: Math is broken"
"loop: 1"
"loop: 2"
"loop: 1"
// Stack trace output
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 20. Summary

| API | Key Point |
|---|---|
| Fetch | Promise-based; does NOT reject on 4xx/5xx; check `response.ok` |
| Fetch vs XHR | Fetch is cleaner, stream-capable, Service Worker-interceptable |
| Headers | Case-insensitive key-value store |
| Response | `response.body` is a stream; `.json()`, `.text()` return Promises |
| Web Worker | Background thread for CPU work; no DOM access |
| Service Worker | Network proxy; caching; offline; push; lives beyond page lifetime |
| IntersectionObserver | Fires when element enters/leaves viewport; great for lazy loading |
| ResizeObserver | Per-element resize notification; no infinite loop issue |
| rAF | Pre-repaint callback; 60fps synchronized; use for animations |
| requestIdleCallback | Run work in idle time; `deadline.timeRemaining()` for chunking |
| Geolocation | Async; requires HTTPS and user permission |
| Clipboard API | Async read/write; write OK on user gesture; read needs permission |
| History API | `pushState` / `replaceState` for SPA routing; `popstate` for back/forward |
| URL API | Parse and build URLs; `URLSearchParams` for query strings |
| BroadcastChannel | Cross-tab messaging on the same origin |
| Notifications API | System-level notifications; requires `Notification.requestPermission()` |
| Performance API | `performance.now()` for high-res timing; marks and measures for profiling |
| console methods | `time`, `timeEnd`, `group`, `table`, `assert`, `count`, `trace` |

---

## Final Notes

Browser APIs bridge JavaScript and the full capability of the browser platform. The Fetch API and Workers are the most interview-critical: understand that `fetch` never rejects on HTTP error status codes, that Web Workers are for CPU-bound tasks while Service Workers are for network-bound caching and offline scenarios, and that `requestAnimationFrame` is the correct tool for animations (not `setTimeout`). For performance measurement, `performance.now()` provides microsecond precision compared to `Date.now()`. The observer APIs (`IntersectionObserver`, `ResizeObserver`, `MutationObserver`) follow a consistent instantiate-observe-callback pattern and should be your go-to over scroll event listeners or polling.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
