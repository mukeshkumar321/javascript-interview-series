# Storage and Caching in JavaScript

## Table of Contents

1. [Web Storage Overview](#1-web-storage-overview)
2. [localStorage](#2-localstorage)
3. [sessionStorage](#3-sessionstorage)
4. [localStorage vs sessionStorage](#4-localstorage-vs-sessionstorage)
5. [Cookies](#5-cookies)
6. [localStorage vs Cookies](#6-localstorage-vs-cookies)
7. [localStorage API](#7-localstorage-api)
8. [Storing Objects in localStorage](#8-storing-objects-in-localstorage)
9. [localStorage Events](#9-localstorage-events)
10. [IndexedDB](#10-indexeddb)
11. [IndexedDB vs localStorage](#11-indexeddb-vs-localstorage)
12. [Cache API](#12-cache-api)
13. [HTTP Caching Headers](#13-http-caching-headers)
14. [Cache-Control Directives](#14-cache-control-directives)
15. [Browser Memory Cache vs Disk Cache](#15-browser-memory-cache-vs-disk-cache)
16. [Service Worker Cache](#16-service-worker-cache)
17. [Stale-While-Revalidate Strategy](#17-stale-while-revalidate-strategy)
18. [Storage Limits](#18-storage-limits)
19. [Privacy Mode and Storage](#19-privacy-mode-and-storage)
20. [Summary](#20-summary)

---

## 1. Web Storage Overview

The browser provides several client-side storage mechanisms. Each has different scope, lifetime, capacity, and access style. Choosing the right one depends on whether the server needs the data, how much you need to store, and whether it should persist across sessions.

| Mechanism | Scope | Persistence | Capacity | Access |
|---|---|---|---|---|
| localStorage | Origin | Until cleared | ~5 MB | Synchronous |
| sessionStorage | Tab + Origin | Until tab closes | ~5 MB | Synchronous |
| Cookies | Domain | Configurable | ~4 KB | Sync + HTTP |
| IndexedDB | Origin | Until cleared | Hundreds of MB | Asynchronous |
| Cache API | Origin | Until cleared | Large (disk-based) | Asynchronous |

```js
// Check available storage quota for IndexedDB and Cache API
navigator.storage.estimate().then(({ quota, usage }) => {
  console.log('Quota:', quota);
  console.log('Usage:', usage);
});
```

### Output

```js
Quota: 439804651520
Usage: 12345
```

---

## 2. localStorage

`localStorage` stores key-value pairs as **strings** and persists data even after the browser is closed and reopened. It is scoped to the origin (`protocol + hostname + port`).

- Capacity: ~5 MB per origin
- Values are always strings — numbers and objects must be converted
- Synchronous: every call blocks the main thread

```js
localStorage.setItem('theme', 'dark');
localStorage.setItem('count', 42);       // stored as string "42"

console.log(localStorage.getItem('theme'));           // "dark"
console.log(localStorage.getItem('count'));           // "42" (not a number)
console.log(typeof localStorage.getItem('count'));    // "string"
```

### Output

```js
"dark"
"42"
"string"
```

---

## 3. sessionStorage

`sessionStorage` has the identical API to `localStorage` but data is scoped to the **current browser tab and session**. When the tab is closed the data is gone.

- Opening the same URL in a new tab creates a **separate** sessionStorage
- Reloading the page does NOT clear sessionStorage
- Duplicating a tab copies the current sessionStorage snapshot at duplication time

```js
sessionStorage.setItem('pageVisited', 'true');
sessionStorage.setItem('scrollPosition', '450');

console.log(sessionStorage.getItem('pageVisited'));    // "true"
console.log(sessionStorage.getItem('scrollPosition')); // "450"
console.log(sessionStorage.length);                    // 2
// Closing this tab clears everything above
```

### Output

```js
"true"
"450"
2
```

---

## 4. localStorage vs sessionStorage

Both objects implement the same `Storage` interface. The only difference is lifetime and tab scope.

| Feature | localStorage | sessionStorage |
|---|---|---|
| Persistence | Until explicitly cleared | Until tab/window closes |
| Scope | All tabs of same origin | Current tab only |
| Survives page reload | Yes | Yes |
| Survives browser restart | Yes | No |
| Shared across tabs | Yes | No |
| `storage` event fired | Yes (in other tabs) | No |
| Capacity | ~5 MB | ~5 MB |

```js
// localStorage is shared: Tab B can read what Tab A wrote
localStorage.setItem('shared', 'hello from Tab A');

// sessionStorage is isolated: Tab B always sees null for Tab A's keys
sessionStorage.setItem('private', 'only this tab');

// In a new tab opening the same origin:
console.log(localStorage.getItem('shared'));    // "hello from Tab A"
console.log(sessionStorage.getItem('private')); // null
```

### Output

```js
// In the ORIGINAL tab:
"hello from Tab A"
null                 // sessionStorage of THIS tab has no "private" key

// In a NEW tab:
"hello from Tab A"   // localStorage shared
null                 // sessionStorage is empty in the new tab
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 5. Cookies

Cookies are small pieces of data sent back and forth with every matching HTTP request. They are set via `document.cookie` in JavaScript or via the `Set-Cookie` response header from the server.

Key cookie attributes:
- **name=value** — the actual data
- **domain** — which domains the cookie is sent to
- **path** — URL path restriction (defaults to `/`)
- **expires / max-age** — when the cookie expires; omitting both makes it a session cookie
- **HttpOnly** — blocks JavaScript access; readable only by the server
- **Secure** — only sent over HTTPS
- **SameSite** — controls cross-site sending: `Strict`, `Lax`, or `None`

```js
// Set a cookie that expires in 1 hour
document.cookie = "username=Alice; path=/; max-age=3600; SameSite=Lax";

// Reading — all non-HttpOnly cookies as a single semicolon-separated string
console.log(document.cookie); // "username=Alice"

// Adding another cookie (assignment ADDS, it does not replace the whole string)
document.cookie = "theme=dark; path=/";
console.log(document.cookie); // "username=Alice; theme=dark"

// Deleting a cookie: set max-age=0 with same name + path
document.cookie = "username=Alice; max-age=0; path=/";
console.log(document.cookie); // "theme=dark"
```

### Output

```js
"username=Alice"
"username=Alice; theme=dark"
"theme=dark"
```

---

## 6. localStorage vs Cookies

| Feature | localStorage | Cookies |
|---|---|---|
| Capacity | ~5 MB | ~4 KB per cookie |
| Sent with HTTP requests | No | Yes (automatically) |
| Accessible via JavaScript | Yes | Yes (unless HttpOnly) |
| Expiry control | None (manual clear) | Yes (expires / max-age) |
| HttpOnly support | No | Yes |
| Secure flag | No | Yes |
| Cross-subdomain access | No | Yes (via domain attribute) |
| Created by | JavaScript | JavaScript or Server |
| Best use case | Client-only UI state | Auth tokens, server-needed state |

```js
// localStorage: good for client-only preferences
localStorage.setItem('preferredLanguage', 'en');

// Cookies: use when the server needs the value on every request
// (HttpOnly cookies cannot be set from JavaScript — only by the server)
document.cookie = "sessionId=xyz123; path=/; SameSite=Strict";

console.log(localStorage.getItem('preferredLanguage')); // "en"
console.log(document.cookie);                           // "sessionId=xyz123"
```

### Output

```js
"en"
"sessionId=xyz123"
```

---

## 7. localStorage API

The `localStorage` object provides a straightforward synchronous key-value API.

| Method / Property | Description |
|---|---|
| `setItem(key, value)` | Store a value (coerced to string) |
| `getItem(key)` | Retrieve a value; returns `null` if key does not exist |
| `removeItem(key)` | Delete a specific key |
| `clear()` | Delete all keys |
| `key(index)` | Return the key name at the given numeric index |
| `length` | Number of keys currently stored |

```js
localStorage.setItem('a', '1');
localStorage.setItem('b', '2');
localStorage.setItem('c', '3');

console.log(localStorage.length);        // 3
console.log(localStorage.getItem('b'));  // "2"
console.log(localStorage.getItem('z'));  // null  (missing key)
console.log(localStorage.key(0));        // "a"   (order varies by browser)

localStorage.removeItem('a');
console.log(localStorage.length);        // 2

localStorage.clear();
console.log(localStorage.length);        // 0
```

### Output

```js
3
"2"
null
"a"
2
0
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 8. Storing Objects in localStorage

`localStorage` only holds strings. Storing an object directly calls `.toString()` on it, producing the useless `"[object Object]"`. Always use `JSON.stringify()` to save and `JSON.parse()` to retrieve.

Important limitations of `JSON.stringify`:
- Functions are silently dropped
- `undefined` values on object properties are silently dropped
- `Date` objects are converted to ISO strings (not restored as Date on parse)

```js
// Wrong — stores "[object Object]"
const user = { name: 'Alice', age: 25 };
localStorage.setItem('user', user);
console.log(localStorage.getItem('user')); // "[object Object]"

// Correct
localStorage.setItem('user', JSON.stringify(user));
const retrieved = JSON.parse(localStorage.getItem('user'));
console.log(retrieved.name);    // "Alice"
console.log(typeof retrieved);  // "object"

// Functions and undefined are silently dropped
const obj = { name: 'Bob', greet: () => 'Hi', val: undefined };
localStorage.setItem('obj', JSON.stringify(obj));
console.log(localStorage.getItem('obj')); // '{"name":"Bob"}'
```

### Output

```js
"[object Object]"
"Alice"
"object"
'{"name":"Bob"}'
```

---

## 9. localStorage Events

The `storage` event fires on **other tabs and windows** of the same origin when a localStorage key is added, changed, or removed. It does **not** fire in the tab that made the change. It never fires for sessionStorage.

```js
// ---- Tab A: listen for changes ----
window.addEventListener('storage', (event) => {
  console.log('Key changed:', event.key);
  console.log('Old value:', event.oldValue);
  console.log('New value:', event.newValue);
  console.log('Same storage?', event.storageArea === localStorage);
  console.log('Source URL:', event.url);
});

// ---- Tab B: trigger the event ----
localStorage.setItem('theme', 'dark');
// Tab A receives the storage event; Tab B gets nothing
```

### Output

```js
// Output in Tab A when Tab B calls setItem('theme', 'dark'):
Key changed: theme
Old value: null
New value: dark
Same storage? true
Source URL: https://example.com/page
```

---

## 10. IndexedDB

IndexedDB is a low-level, asynchronous, transactional database in the browser. It is designed for large amounts of structured data (objects, files, blobs) and supports indexes and complex queries.

- Uses **object stores** (similar to tables)
- All operations are **asynchronous** (event-based or via the `idb` Promise wrapper)
- **Transactions** ensure data integrity
- Accessible from Web Workers

```js
const request = indexedDB.open('MyDatabase', 1);

// Runs when the database is created or version number increases
request.onupgradeneeded = (event) => {
  const db = event.target.result;
  const store = db.createObjectStore('users', { keyPath: 'id' });
  store.createIndex('name', 'name', { unique: false });
  console.log('Database upgraded');
};

request.onsuccess = (event) => {
  const db = event.target.result;
  const tx = db.transaction('users', 'readwrite');
  const store = tx.objectStore('users');

  store.add({ id: 1, name: 'Alice', age: 25 });

  const getReq = store.get(1);
  getReq.onsuccess = () => {
    console.log(getReq.result.name); // "Alice"
  };
};
```

### Output

```js
"Database upgraded"
"Alice"
```

---

## 11. IndexedDB vs localStorage

| Feature | localStorage | IndexedDB |
|---|---|---|
| Data types | Strings only | Any structured data |
| Storage limit | ~5 MB | Up to ~50% of disk space |
| API style | Synchronous | Asynchronous |
| Transactions | No | Yes |
| Indexes / queries | No | Yes |
| Works in Web Workers | No | Yes |
| Best use case | Small key-value preferences | Large or complex offline data |

```js
// localStorage: simple, small data
localStorage.setItem('sidebarOpen', 'true');

// IndexedDB: complex structured data (e.g. thousands of records)
// indexedDB.open('AppDB', 1) — see section 10 for full example
console.log('Use IndexedDB for offline-first apps with large datasets');
```

### Output

```js
"Use IndexedDB for offline-first apps with large datasets"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 12. Cache API

The Cache API is a Promise-based storage mechanism for `Request`/`Response` pairs, designed to work alongside Service Workers to cache network resources for offline use.

```js
// Open (or create) a named cache
caches.open('my-cache-v1').then((cache) => {
  // Cache a single URL
  cache.add('/styles/main.css');

  // Cache multiple URLs at once
  cache.addAll(['/index.html', '/app.js']);

  // Store a custom Response manually
  const req = new Request('/api/config');
  const res = new Response(JSON.stringify({ theme: 'dark' }), {
    headers: { 'Content-Type': 'application/json' }
  });
  cache.put(req, res);

  console.log('Resources cached');
});

// Retrieve a cached response
caches.match('/styles/main.css').then((response) => {
  if (response) {
    console.log('Cache hit, status:', response.status);
  } else {
    console.log('Cache miss — fetching from network');
  }
});
```

### Output

```js
"Resources cached"
"Cache hit, status: 200"
```

---

## 13. HTTP Caching Headers

HTTP caching headers let the server tell the browser how (and whether) to cache a response. The browser uses these on the next request to decide whether to serve from cache or hit the network.

| Header | Direction | Purpose |
|---|---|---|
| `Cache-Control` | Response (& Request) | Primary caching directive |
| `Expires` | Response | Legacy absolute expiry date |
| `ETag` | Response | Opaque version identifier for the resource |
| `Last-Modified` | Response | Date the resource was last changed |
| `If-None-Match` | Request | Sends the cached ETag to check if still valid |
| `If-Modified-Since` | Request | Sends a date to check if resource changed |

```js
// Inspect caching headers from a fetch response
fetch('/api/data').then((response) => {
  console.log(response.headers.get('Cache-Control'));  // "max-age=3600"
  console.log(response.headers.get('ETag'));           // '"d41d8cd9"'
  console.log(response.headers.get('Last-Modified'));  // "Wed, 01 Jan 2025 00:00:00 GMT"
  console.log(response.headers.get('Expires'));        // "Thu, 02 Jan 2025 00:00:00 GMT"
});
```

### Output

```js
"max-age=3600"
'"d41d8cd9"'
"Wed, 01 Jan 2025 00:00:00 GMT"
"Thu, 02 Jan 2025 00:00:00 GMT"
```

---

## 14. Cache-Control Directives

`Cache-Control` is the primary tool for controlling browser and CDN caching. It can contain multiple comma-separated directives.

| Directive | Meaning |
|---|---|
| `no-store` | Never cache — always fetch fresh; no storage at all |
| `no-cache` | Store the response but revalidate with server before every use |
| `max-age=N` | Cache the response for N seconds from the time of the request |
| `s-maxage=N` | Override max-age for shared caches (CDNs/proxies) only |
| `private` | Only the end-user's browser may cache (not CDNs) |
| `public` | Any cache (browser, CDN, proxy) may store the response |
| `must-revalidate` | After max-age expires, must revalidate before serving stale copy |
| `immutable` | Content will never change; skip revalidation even after max-age |

```js
// Long-lived static asset (fingerprinted filename)
fetch('/static/app.a1b2c3.js').then((res) => {
  console.log(res.headers.get('Cache-Control'));
  // "public, max-age=31536000, immutable"
  // Browser caches for 1 year and never revalidates
});

// Sensitive user data
fetch('/user/profile').then((res) => {
  console.log(res.headers.get('Cache-Control'));
  // "private, no-cache"
  // Stored only in user's browser; always revalidated before use
});
```

### Output

```js
"public, max-age=31536000, immutable"
"private, no-cache"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 15. Browser Memory Cache vs Disk Cache

When a browser caches resources it uses two main tiers automatically:

- **Memory Cache** — Stored in RAM. Extremely fast. Lives only for the current page session. Cleared when the tab is closed or the page is refreshed in some cases. Used for resources loaded within the current page navigation.
- **Disk Cache** — Stored on the hard drive. Persists across browser restarts. Slower than memory cache but retains data long-term based on `Cache-Control` headers.

The browser decides which tier to use — you cannot control this from JavaScript. You can observe it in the DevTools **Network** tab under the **Size** column ("from memory cache" / "from disk cache").

```js
// Cache busting: append a version query parameter to force a fresh network request
const version = '2.1.0';
fetch(`/static/app.js?v=${version}`).then((res) => {
  console.log(res.headers.get('Cache-Control')); // "public, max-age=86400"
  console.log('Response served from network — not from any cache');
});
```

### Output

```js
"public, max-age=86400"
"Response served from network — not from any cache"
```

---

## 16. Service Worker Cache

A Service Worker is a background script that runs separately from the page and can intercept all outgoing network requests. By combining a Service Worker with the Cache API you can serve cached responses even when the device is offline. Full Service Worker coverage is in **16-Browser-APIs**.

```js
// service-worker.js — Cache-First strategy
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then((cachedResponse) => {
      if (cachedResponse) {
        console.log('Serving from cache:', event.request.url);
        return cachedResponse;
      }
      // Not in cache: fetch from network and store the response
      return fetch(event.request).then((networkResponse) => {
        const clone = networkResponse.clone();
        caches.open('dynamic-v1').then((cache) => cache.put(event.request, clone));
        return networkResponse;
      });
    })
  );
});
```

### Output

```js
// First visit (no cache yet): network request made, response stored
// Subsequent visits (cache hit):
"Serving from cache: https://example.com/app.js"
```

---

## 17. Stale-While-Revalidate Strategy

Stale-While-Revalidate (SWR) returns a **cached (possibly stale) response immediately** and simultaneously fetches a fresh copy in the background to update the cache. This gives the user a fast response while keeping data up to date.

Can be set via:
1. **HTTP header**: `Cache-Control: max-age=60, stale-while-revalidate=3600`
2. **Service Worker**: manually implement the pattern

```js
// service-worker.js — Stale-While-Revalidate
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.open('swr-v1').then(async (cache) => {
      const cachedResponse = await cache.match(event.request);

      // Fetch in background and update the cache silently
      const fetchPromise = fetch(event.request).then((networkResponse) => {
        cache.put(event.request, networkResponse.clone());
        return networkResponse;
      });

      // Immediately return cache hit; fall back to network if no cache
      return cachedResponse || fetchPromise;
    })
  );
});

// Equivalent via HTTP header (set on the server):
// Cache-Control: max-age=60, stale-while-revalidate=3600
// — Serve fresh for 60s; after that serve stale and revalidate in background for up to 3600s
```

### Output

```js
// First visit: served from network (cache empty)
// Subsequent visits within 60s: served fresh from cache
// After 60s: stale cache returned instantly, fresh fetch happens silently in background
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 18. Storage Limits

Storage limits are enforced **per origin**. Values vary by browser and available disk space.

| Mechanism | Typical Limit |
|---|---|
| localStorage | ~5 MB per origin |
| sessionStorage | ~5 MB per origin |
| Cookies | ~4 KB per cookie; ~50–180 cookies per domain |
| IndexedDB | Dynamic; often up to 50–60% of available disk |
| Cache API | Dynamic; shares the same quota pool as IndexedDB |

```js
// Check current usage and available quota (IndexedDB + Cache API)
navigator.storage.estimate().then(({ quota, usage }) => {
  const usedMB = (usage / 1024 / 1024).toFixed(2);
  const quotaGB = (quota / 1024 / 1024 / 1024).toFixed(2);
  console.log(`Used: ${usedMB} MB`);
  console.log(`Quota: ${quotaGB} GB`);
});

// Request persistent storage to prevent eviction under disk pressure
navigator.storage.persist().then((granted) => {
  console.log('Persistent storage granted:', granted);
});
```

### Output

```js
"Used: 0.23 MB"
"Quota: 100.00 GB"
"Persistent storage granted: true"
```

---

## 19. Privacy Mode and Storage

In **private / incognito mode** the browser creates an isolated session. Most storage works during the session but is destroyed when all private windows are closed.

| Mechanism | Behaviour in Private Mode |
|---|---|
| localStorage | Works, cleared when last incognito window closes |
| sessionStorage | Works normally (per-tab, cleared on close) |
| Cookies | Session cookies only; persistent cookies discarded on close |
| IndexedDB | Works, cleared on close |
| Cache API | Works, cleared on close |
| `navigator.storage.persist()` | Always returns `false` |

```js
// Detecting incognito is not reliably possible, but you can detect storage failure
try {
  localStorage.setItem('_test', '1');
  localStorage.removeItem('_test');
  console.log('Storage is available');
} catch (e) {
  // SecurityError in some browsers when storage is blocked
  console.log('Storage unavailable:', e.name);
}

// navigator.storage.persist() returning false is one soft signal
navigator.storage.persist().then((granted) => {
  if (!granted) {
    console.log('Storage is not persistent — possible incognito or user denied');
  }
});
```

### Output

```js
"Storage is available"
"Storage is not persistent — possible incognito or user denied"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 20. Summary

| Topic | Key Point |
|---|---|
| localStorage | Persistent, ~5 MB, strings only, per-origin, synchronous |
| sessionStorage | Same API as localStorage, per-tab, cleared when tab closes |
| Cookies | ~4 KB, sent automatically with HTTP, HttpOnly / Secure / SameSite attributes |
| localStorage vs Cookies | localStorage for client-only data; cookies when the server needs the value |
| localStorage API | setItem, getItem, removeItem, clear, key(n), length |
| Storing objects | Always JSON.stringify to save and JSON.parse to retrieve |
| Storage events | Fires in OTHER tabs only, not the tab that made the change |
| IndexedDB | Asynchronous, structured data, large quota, supports indexes and transactions |
| Cache API | Promise-based, stores Request/Response pairs, used with Service Workers |
| Cache-Control | no-store, no-cache, max-age, private, public, immutable |
| ETag / Last-Modified | Enable conditional requests to avoid re-downloading unchanged resources |
| Memory vs Disk cache | Memory: fast, session-only; Disk: persistent, survives restarts |
| Stale-While-Revalidate | Return stale response immediately while refreshing in the background |
| Storage limits | localStorage ~5 MB; IndexedDB / Cache up to ~50% of available disk |
| Privacy mode | All storage cleared when the last incognito window closes |

---

## Final Notes

Understanding storage and caching is essential for building fast, reliable web applications. Use `localStorage` for lightweight client-side preferences, cookies when the server needs the value on each request, and `IndexedDB` for complex offline data. Combine the Cache API with Service Workers to build offline-first experiences. Always handle storage failures gracefully — quota can be exceeded, storage can be blocked in private mode, and older browsers may not support newer APIs.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
