# Storage and Caching — Tricky Output Questions

## Table of Contents

1. [localStorage vs sessionStorage Questions](#1-localstorage-vs-sessionstorage-questions)
2. [Cookie Questions](#2-cookie-questions)
3. [Storage API Questions](#3-storage-api-questions)
4. [HTTP Caching Questions](#4-http-caching-questions)
5. [Advanced Storage Questions](#5-advanced-storage-questions)

---

## 1. localStorage vs sessionStorage Questions

---

### Q1. What will be the output?

```js
localStorage.setItem('count', 5);
const count = localStorage.getItem('count');

console.log(count + 1);
console.log(typeof count);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
"51"
"string"
```

### Explanation

`localStorage.setItem()` always converts the value to a string. `getItem()` always returns a string. So `count` is `"5"`, not `5`. The expression `"5" + 1` performs string concatenation and produces `"51"`, not `6`. Always convert back with `Number()` or `parseInt()` when doing arithmetic.

</details>

---

### Q2. What will be the output?

```js
console.log(localStorage.getItem('nonexistent'));
console.log(localStorage.getItem('nonexistent') === null);
console.log(localStorage.getItem('nonexistent') === undefined);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
null
true
false
```

### Explanation

`getItem()` returns `null` (not `undefined`) when the key does not exist. This is an important distinction — checking `=== undefined` will always be `false` for missing keys.

</details>

---

### Q3. Tab A sets sessionStorage. Tab B opens the same URL. What will Tab B log?

```js
// Tab A:
sessionStorage.setItem('user', 'Alice');

// Tab B opens the same URL:
console.log(sessionStorage.getItem('user'));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
null
```

### Explanation

`sessionStorage` is scoped to the **individual tab**. Each tab gets its own completely isolated sessionStorage. Tab B has never set 'user', so it gets `null`. This contrasts with `localStorage`, where Tab B would have read `"Alice"`.

</details>

---

### Q4. What will be the output?

```js
localStorage.setItem('theme', 'light');
localStorage.setItem('theme', 'dark');

console.log(localStorage.length);
console.log(localStorage.getItem('theme'));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
1
"dark"
```

### Explanation

Setting a key that already exists **updates** the value — it does not create a new entry. `length` stays at `1` because there is still only one key named `'theme'`, and its value is now `"dark"`.

</details>

---

## 2. Cookie Questions

---

### Q1. What will be the output?

```js
document.cookie = "x=1";
document.cookie = "y=2";
document.cookie = "z=3";

console.log(document.cookie);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
"x=1; y=2; z=3"
```

### Explanation

Unlike localStorage, **assigning to `document.cookie` does not replace the entire cookie string**. Each assignment adds or updates a single cookie. After three assignments, all three cookies are present and reading `document.cookie` returns them all as one semicolon-separated string.

</details>

---

### Q2. What will be the output?

```js
document.cookie = "name=Alice";
document.cookie = "name=Bob";

console.log(document.cookie);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
"name=Bob"
```

### Explanation

When you set a cookie with the same **name** and **path** as an existing one, the value is overwritten. Both assignments use the default path `/`, so the second assignment updates `name` to `"Bob"`.

</details>

---

### Q3. The server set a cookie with `HttpOnly`. What will be the output?

```js
// Server response header: Set-Cookie: secret=abc123; HttpOnly

document.cookie = "visible=yes";

console.log(document.cookie.includes('secret'));
console.log(document.cookie.includes('visible'));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
false
true
```

### Explanation

`HttpOnly` cookies are **inaccessible to JavaScript**. They are sent by the browser to the server on every request, but `document.cookie` simply does not include them. This is a security feature that prevents XSS attacks from stealing session tokens.

</details>

---

## 3. Storage API Questions

---

### Q1. What will be the output?

```js
const user = { name: 'Alice', role: 'admin' };
localStorage.setItem('user', user);

console.log(localStorage.getItem('user'));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
"[object Object]"
```

### Explanation

When you pass an object to `setItem`, JavaScript calls `.toString()` on it. The default `toString()` for plain objects returns `"[object Object]"`. This is almost never what you want. Always use `JSON.stringify(user)` before storing.

</details>

---

### Q2. What will be the output?

```js
localStorage.setItem('val', undefined);

console.log(localStorage.getItem('val'));
console.log(localStorage.getItem('val') === undefined);
console.log(localStorage.getItem('val') === 'undefined');
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
"undefined"
false
true
```

### Explanation

`setItem` calls `.toString()` on the value. `undefined.toString()` produces the string `"undefined"`. So the stored value is the four-character string `"undefined"`. `getItem` then returns that string, which is `=== 'undefined'` (string) but not `=== undefined` (the JS primitive).

</details>

---

### Q3. What will be the output?

```js
localStorage.clear();
localStorage.setItem('n', 0);

for (let i = 0; i < 3; i++) {
  localStorage.setItem('n', localStorage.getItem('n') + 1);
}

console.log(localStorage.getItem('n'));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
"0111"
```

### Explanation

`getItem` always returns a string. Each iteration:
- i=0: `"0" + 1` → `"01"` (string concatenation)
- i=1: `"01" + 1` → `"011"`
- i=2: `"011" + 1` → `"0111"`

The `+` operator sees a string on the left and coerces the number `1` to `"1"`. Fix: use `Number(localStorage.getItem('n')) + 1`.

</details>

---

### Q4. What will be the output?

```js
// The key 'data' has never been set

const val = localStorage.getItem('data');
console.log(val);
console.log(JSON.parse(val));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
null
null
```

### Explanation

`getItem` returns `null` for a missing key. `JSON.parse(null)` does **not** throw — it returns `null` because `null` is valid JSON. This can hide bugs: you may expect to get an object back but silently receive `null` instead.

</details>

---

## 4. HTTP Caching Questions

---

### Q1. The server returns a `304 Not Modified`. What does JavaScript see?

```js
// Browser has a cached copy with ETag "etag-abc"
// Server returns 304 (ETag still matches)

fetch('/api/data', {
  headers: { 'If-None-Match': '"etag-abc"' }
}).then((res) => {
  console.log(res.status);
  console.log(res.ok);
});
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
200
true
```

### Explanation

The browser handles 304 responses **transparently**. It intercepts the 304, retrieves the cached body, and presents a synthetic `200` response to JavaScript. Your `fetch()` code never sees a 304 status.

</details>

---

### Q2. What is the difference between `Cache-Control: no-store` and `Cache-Control: no-cache`?

```js
// Response A has: Cache-Control: no-store
// Response B has: Cache-Control: no-cache

// On the next request for each resource, what does the browser do?

// Resource A (no-store):
// Browser never stored the response — always makes a full network request

// Resource B (no-cache):
// Browser stored the response but MUST revalidate (If-None-Match / If-Modified-Since)
// before using it — server can return 304 to serve cached copy

console.log('no-store: never cached, always fresh from server');
console.log('no-cache: cached, but revalidate every time before use');
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
"no-store: never cached, always fresh from server"
"no-cache: cached, but revalidate every time before use"
```

### Explanation

`no-store` means the response is **never written to any cache**. `no-cache` is misleading — it actually **allows caching** but requires a round-trip to the server for revalidation before serving the cached copy. If the server returns 304, the cached version is used.

</details>

---

### Q3. A static image has `Cache-Control: public, max-age=31536000, immutable`. The user loads the page today and again six months later. What happens on the second load?

```js
// Simulating what the browser does:
fetch('/static/logo.a1b2c3.png', { cache: 'force-cache' })
  .then((res) => {
    console.log(res.status);  // 200 (from disk cache)
    console.log(res.headers.get('Cache-Control'));
    // "public, max-age=31536000, immutable"
  });
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
200
"public, max-age=31536000, immutable"
```

### Explanation

Six months (~15.8 million seconds) is less than `max-age=31536000` (one year). The `immutable` directive additionally tells the browser never to revalidate even after expiry. The image is served directly from the disk cache with zero network round-trips. This is why fingerprinted filenames (content hashing) are paired with long `max-age`.

</details>

---

## 5. Advanced Storage Questions

---

### Q1. Does the `storage` event fire in the same tab that made the change?

```js
let fired = false;

window.addEventListener('storage', () => {
  fired = true;
});

localStorage.setItem('key', 'value');
console.log(fired);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
false
```

### Explanation

The `storage` event fires in **other** tabs and windows of the same origin — it never fires in the tab that made the change. This is by design: the tab that writes already knows what it wrote. If you need to react to a storage change in the same tab, you must call your own handler directly.

</details>

---

### Q2. What will be the output when localStorage is full?

```js
try {
  // Attempt to store 10 MB (well above the ~5 MB limit)
  localStorage.setItem('bigData', 'x'.repeat(10 * 1024 * 1024));
} catch (e) {
  console.log(e.name);
  console.log(e instanceof DOMException);
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
"QuotaExceededError"
true
```

### Explanation

When the storage quota is exceeded, `setItem` throws a `DOMException` with the name `"QuotaExceededError"`. Always wrap localStorage writes in a `try/catch` in production code — especially for large values — because the quota can also be much lower in private/incognito mode.

</details>

---

### Q3. What will be the output?

```js
localStorage.setItem('flag', true);   // boolean
localStorage.setItem('score', 42);    // number

const flag  = localStorage.getItem('flag');
const score = localStorage.getItem('score');

console.log(flag  === true);         // comparing string to boolean
console.log(flag  === 'true');       // comparing string to string
console.log(score === 42);           // comparing string to number
console.log(parseInt(score) === 42); // properly converted
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
false
true
false
true
```

### Explanation

`setItem` converts every value to a string via `.toString()`. `true` becomes `"true"` and `42` becomes `"42"`. Strict equality (`===`) does not coerce types, so `"true" === true` is `false` and `"42" === 42` is `false`. Always convert retrieved values back to their expected types.

</details>

---

### Q4. What will be the output?

```js
// Storage event cross-tab scenario
// Tab A runs:
window.addEventListener('storage', (e) => {
  console.log('Tab A heard:', e.key, e.newValue);
});
localStorage.setItem('msg', 'hello');

// Tab B (same origin) runs:
window.addEventListener('storage', (e) => {
  console.log('Tab B heard:', e.key, e.newValue);
});
localStorage.setItem('reply', 'world');
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
// In Tab A (after Tab B sets 'reply'):
"Tab A heard: reply world"

// In Tab B (after Tab A sets 'msg'):
"Tab B heard: msg hello"

// Neither tab hears its OWN setItem call
```

### Explanation

The `storage` event is a **cross-tab broadcast**. Tab A hears Tab B's write, and Tab B hears Tab A's write — but each tab is silent to its own writes. This makes it useful for cross-tab state synchronisation (e.g., logging out a user in all open tabs).

</details>

---

## Final Tips

- `localStorage` and `sessionStorage` store **strings only** — always convert numbers and objects explicitly.
- `getItem` returns **`null`** for missing keys, never `undefined`.
- `document.cookie` assignment **adds** a cookie, it does not replace the whole string.
- `HttpOnly` cookies are completely invisible to JavaScript.
- The `storage` event fires in **other tabs**, not the one that wrote.
- `JSON.parse(null)` silently returns `null` — validate before parsing.
- Always wrap `localStorage` writes in `try/catch` to handle `QuotaExceededError`.
- `Cache-Control: no-cache` does **not** mean "don't cache" — it means "cache but always revalidate".
- A 304 response is **transparent to JavaScript** — `fetch` presents it as a 200.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
