# Networking — Tricky Output Questions

## Table of Contents

1. [CORS Questions](#1-cors-questions)
2. [Fetch Questions](#2-fetch-questions)
3. [HTTP Status Code Questions](#3-http-status-code-questions)
4. [WebSocket Questions](#4-websocket-questions)
5. [async/await with Fetch Questions](#5-asyncawait-with-fetch-questions)
6. [AbortController Questions](#6-abortcontroller-questions)
7. [Advanced Networking Questions](#7-advanced-networking-questions)

---

## 1. CORS Questions

---

### Q1. Does this request trigger a CORS preflight? What is sent first?

```js
// From: https://app.com
// To:   https://api.com

fetch('https://api.com/data', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer token123'
  },
  body: JSON.stringify({ key: 'value' })
});
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
// Browser sends TWO requests (visible in DevTools > Network):
1. OPTIONS https://api.com/data     ← preflight
2. POST    https://api.com/data     ← actual request (only if preflight passes)
```

### Explanation

A preflight is triggered because this request is "non-simple":
- `Content-Type: application/json` is not one of the simple content types (`text/plain`, `multipart/form-data`, `application/x-www-form-urlencoded`)
- `Authorization` is a custom header not on the safe-list

The browser automatically sends the `OPTIONS` request first with `Access-Control-Request-Method: POST` and `Access-Control-Request-Headers: Content-Type, Authorization`. The POST only goes if the server responds with matching `Access-Control-Allow-*` headers.

</details>

---

### Q2. What will be the output when the server has no CORS headers?

```js
// From: https://myapp.com
// Server at https://api.other.com has no Access-Control-Allow-Origin header

fetch('https://api.other.com/data')
  .then(res => {
    console.log('Status:', res.status);
    return res.json();
  })
  .then(data => console.log('Data:', data))
  .catch(err => console.log('Error:', err.message));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
"Error: Failed to fetch"
```

### Explanation

The request **did reach the server** (you can verify this in server logs). However, the browser checks the response for `Access-Control-Allow-Origin` and finds nothing. It then **discards the response** and rejects the promise with a generic `TypeError: Failed to fetch`. The server's 200 response is invisible to JavaScript. CORS is enforced entirely on the browser side.

</details>

---

### Q3. What will be the output with `mode: 'no-cors'`?

```js
fetch('https://api.other.com/data', { mode: 'no-cors' })
  .then(res => {
    console.log(res.type);
    console.log(res.status);
    console.log(res.ok);
    return res.json();
  })
  .catch(err => console.log('Error:', err.message));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
"opaque"
0
false
"Error: Failed to execute 'json' on 'Response': body is null"
```

### Explanation

`no-cors` mode lets the browser send the request but returns an **opaque response** — a sealed wrapper with no readable status, no readable headers, and no readable body. `res.status` is always `0` and `res.ok` is always `false`. Calling `res.json()` or `res.text()` throws because the body is inaccessible. `no-cors` is only useful for caching opaque responses (e.g. for `<img>` tiles) via a Service Worker.

</details>

---

### Q4. What must the server's preflight response include for this request to succeed?

```js
// Client sends:
// Origin: https://dashboard.com
// Access-Control-Request-Method: DELETE
// Access-Control-Request-Headers: Content-Type, X-Custom-Header

// Which server headers are required?
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
// Required server response headers:
Access-Control-Allow-Origin: https://dashboard.com  (or *)
Access-Control-Allow-Methods: DELETE
Access-Control-Allow-Headers: Content-Type, X-Custom-Header
```

### Explanation

The server must mirror back every requested method and header in the `Access-Control-Allow-*` response headers. If any piece is missing the browser aborts the actual request. Optionally the server can add `Access-Control-Max-Age: 86400` to cache the preflight result and avoid sending it on every request.

</details>

---

## 2. Fetch Questions

---

### Q1. What will be the output when the server returns 404?

```js
fetch('/api/nonexistent')
  .then(res => {
    console.log(res.status);
    console.log(res.ok);
  })
  .catch(err => {
    console.log('Caught:', err.message);
  });
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
404
false
```

### Explanation

`fetch()` only rejects (goes to `.catch`) on **network-level failures** such as no internet connection, DNS failure, or a refused connection. HTTP error codes like 404, 500, or 403 are treated as valid responses. The promise resolves with a `Response` object where `res.ok` is `false` (because the status is not in the 200–299 range). This is the most common `fetch` interview trap.

</details>

---

### Q2. What will be logged and in what order?

```js
console.log('A');

fetch('/api/data')
  .then(res => res.json())
  .then(data => console.log('B'));

console.log('C');
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
A
C
B
```

### Explanation

`fetch` is asynchronous — it schedules the HTTP request and immediately moves on. The synchronous `console.log('C')` runs before any `.then` callback. Once the network response arrives and the JavaScript call stack is empty, the microtask queue resolves the `.then` chain and logs `B`.

</details>

---

### Q3. What will be the output when the server returns 500?

```js
fetch('/api/error-endpoint')    // server returns HTTP 500
  .then(res => console.log('then:', res.status))
  .catch(err => console.log('catch:', err.message));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
"then: 500"
```

### Explanation

Again, `fetch` does not reject on 5xx status codes. The `.then` handler runs. `res.status` is `500` and `res.ok` is `false`. The `.catch` handler is never reached. This is why a robust `fetch` wrapper always includes `if (!res.ok) throw new Error(...)`.

</details>

---

### Q4. What will be the output when a fetch is called from an HTTPS page with an HTTP URL?

```js
// Page is served over https://mysite.com
fetch('http://api.mysite.com/data')
  .then(res => console.log('Status:', res.status))
  .catch(err => console.log('Error:', err.message));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
"Error: Failed to fetch"
```

### Explanation

This is a **Mixed Content** violation. Browsers block requests from a secure HTTPS page to an insecure HTTP endpoint. The request never leaves the browser. The fetch promise rejects with a `TypeError: Failed to fetch`. The fix is to ensure all resources are loaded over HTTPS.

</details>

---

## 3. HTTP Status Code Questions

---

### Q1. The server redirects `/old` to `/new`. What does fetch see?

```js
// GET /old → 301 → GET /new → 200

fetch('/old').then(res => {
  console.log(res.status);
  console.log(res.url);
  console.log(res.redirected);
});
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
200
"https://example.com/new"
true
```

### Explanation

`fetch` follows redirects automatically by default. JavaScript only sees the **final response** after all redirects are resolved. `res.status` is `200` (the status of `/new`), `res.url` is the final URL, and `res.redirected` is `true` indicating at least one redirect happened. You can opt out with `redirect: 'manual'` or `redirect: 'error'`.

</details>

---

### Q2. What is the correct status code for each scenario?

```js
// 1. Creating a new user via POST
// 2. Deleting a resource successfully (no body returned)
// 3. Request sent with invalid data (validation failed)
// 4. Valid credentials but insufficient permissions

const scenarios = {
  created:    201,   // POST that creates a resource
  noContent:  204,   // DELETE or PUT with empty response body
  badRequest: 400,   // Malformed / invalid input
  forbidden:  403,   // Authenticated but not authorised
};

Object.entries(scenarios).forEach(([name, code]) => {
  console.log(name, '→', code);
});
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
created → 201
noContent → 204
badRequest → 400
forbidden → 403
```

### Explanation

Key distinctions:
- **201** Created — the resource was created; `Location` header often points to the new resource
- **204** No Content — success but nothing to return (common for DELETE / PUT)
- **400** Bad Request — the request itself is malformed or fails validation
- **403** Forbidden vs **401** Unauthorized — 401 means "you're not logged in"; 403 means "logged in but not allowed"

</details>

---

### Q3. What will `res.ok` return for these status codes?

```js
const codes = [200, 201, 204, 301, 400, 404, 500];

// res.ok is true if and only if status is in 200–299

codes.forEach(code => {
  const ok = code >= 200 && code <= 299;
  console.log(`${code}: ok=${ok}`);
});
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
200: ok=true
201: ok=true
204: ok=true
301: ok=false
400: ok=false
404: ok=false
500: ok=false
```

### Explanation

`res.ok` is a shorthand for `res.status >= 200 && res.status <= 299`. It is `false` for redirects (3xx), client errors (4xx), and server errors (5xx) — all of which fetch treats as resolved, not rejected. Always check `res.ok` before calling `res.json()`.

</details>

---

## 4. WebSocket Questions

---

### Q1. What will be the output and in what order?

```js
const ws = new WebSocket('wss://echo.example.com');

ws.onopen    = () => console.log('open');
ws.onmessage = (e) => console.log('message:', e.data);
ws.onclose   = () => console.log('close');

ws.onopen = () => {
  ws.send('hello');
  // Server echoes "hello" back, then closes the connection
};
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
open
message: hello
close
```

### Explanation

Events fire in order: `open` when the handshake completes, `message` when the server echoes back, `close` when the connection is terminated. Note that the second `ws.onopen` assignment **overwrites** the first — only one handler runs. If the order of assignment matters in your code, use `addEventListener('open', ...)` instead of the `.onopen` property.

</details>

---

### Q2. What will be the output?

```js
const ws = new WebSocket('wss://echo.example.com');

console.log(ws.readyState); // immediately after creation

ws.onopen = () => {
  console.log(ws.readyState); // after connection established
  ws.close();
};

ws.onclose = () => {
  console.log(ws.readyState); // after connection closed
};
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
0
1
3
```

### Explanation

`readyState` values:
- `0` = `CONNECTING` — the connection has been initiated but not yet established
- `1` = `OPEN` — the connection is open and ready to communicate
- `2` = `CLOSING` — `close()` has been called but the close handshake is not yet complete
- `3` = `CLOSED` — the connection is closed

The `CLOSING` (2) state is transient and usually not observable in synchronous logging.

</details>

---

### Q3. What happens if you call `ws.send()` before the connection opens?

```js
const ws = new WebSocket('wss://echo.example.com');

try {
  ws.send('hello before open');
} catch (e) {
  console.log(e instanceof DOMException);
  console.log(e.name);
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
true
"InvalidStateError"
```

### Explanation

Calling `send()` when `readyState` is `CONNECTING` (0) throws a `DOMException` with name `"InvalidStateError"`. The correct pattern is to call `send()` only inside the `onopen` handler or after checking `ws.readyState === WebSocket.OPEN`.

</details>

---

## 5. async/await with Fetch Questions

---

### Q1. What will be the output when the server returns 404?

```js
async function getData() {
  try {
    const res = await fetch('/api/missing');
    const data = await res.json(); // server returned {"error": "not found"}
    console.log('Got:', data);
  } catch (e) {
    console.log('Caught:', e.message);
  }
}
getData();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
"Got:" { error: "not found" }
```

### Explanation

`fetch` resolves (does not throw) for 404 responses. `await fetch(...)` successfully returns a `Response` object with `status: 404`. `await res.json()` parses the JSON body normally. The `catch` block is never reached. To handle HTTP errors, you must check `res.ok` before reading the body.

</details>

---

### Q2. What will be the output?

```js
async function fetchUser(id) {
  const res = await fetch(`/api/users/${id}`);

  if (!res.ok) {
    throw new Error(`HTTP ${res.status}`);
  }

  return res.json();
}

fetchUser(999)   // server returns 404
  .then(user => console.log('User:', user.name))
  .catch(err => console.log('Error:', err.message));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
"Error: HTTP 404"
```

### Explanation

This is the correct pattern for handling HTTP errors with `fetch`. When the server returns 404, `res.ok` is `false` and we throw an `Error`. Because `fetchUser` is an `async` function, the thrown error causes the returned promise to reject, which `.catch` then handles.

</details>

---

### Q3. What will be the output and why?

```js
async function run() {
  const [a, b] = await Promise.all([
    fetch('/api/a').then(r => r.json()),
    fetch('/api/b').then(r => r.json()),
  ]);
  console.log('a:', a.value);
  console.log('b:', b.value);
}
run();

// Both /api/a and /api/b respond in ~100ms
// Compare with sequential:
async function runSequential() {
  const a = await fetch('/api/a').then(r => r.json());
  const b = await fetch('/api/b').then(r => r.json());
  console.log('sequential done');
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
// run() — parallel, completes in ~100ms:
"a: 1"
"b: 2"

// runSequential() — sequential, completes in ~200ms:
"sequential done"
```

### Explanation

`Promise.all` fires both fetches at the same time and waits for both. Total time ≈ max(100ms, 100ms) = ~100ms. The sequential version awaits each one before starting the next, so total time ≈ 100ms + 100ms = ~200ms. Use `Promise.all` when requests are independent.

</details>

---

## 6. AbortController Questions

---

### Q1. What will be the output?

```js
const controller = new AbortController();

fetch('/api/slow', { signal: controller.signal })
  .then(res => console.log('Success:', res.status))
  .catch(err => {
    console.log(err.name);
    console.log(err instanceof DOMException);
  });

controller.abort(); // abort immediately
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
"AbortError"
true
```

### Explanation

Calling `controller.abort()` causes the fetch promise to reject with a `DOMException` named `"AbortError"`. Note that an `AbortError` is NOT a network error — it is intentional. Always check `err.name === 'AbortError'` in your catch block to distinguish user-cancelled requests from real network failures.

</details>

---

### Q2. What will be the output?

```js
const controller = new AbortController();
controller.abort(); // abort BEFORE the fetch

console.log(controller.signal.aborted); // check state

fetch('/api/data', { signal: controller.signal })
  .catch(err => console.log(err.name));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
true
"AbortError"
```

### Explanation

Once `abort()` is called, `signal.aborted` becomes `true` and stays `true` permanently — you cannot "un-abort" a signal. Any `fetch` that receives an already-aborted signal rejects **immediately** without sending a network request. Create a new `AbortController` instance for each operation you want independent cancellation control over.

</details>

---

### Q3. What will be the output when implementing a request timeout?

```js
async function fetchWithTimeout(url, ms) {
  const controller = new AbortController();
  const timerId = setTimeout(() => controller.abort(), ms);

  try {
    const res = await fetch(url, { signal: controller.signal });
    clearTimeout(timerId); // cancel timer if response arrived in time
    return await res.json();
  } catch (err) {
    if (err.name === 'AbortError') {
      console.log(`Request to ${url} timed out after ${ms}ms`);
    } else {
      console.log('Network error:', err.message);
    }
    return null;
  }
}

// Server takes 5 seconds to respond; timeout set to 2 seconds
fetchWithTimeout('/api/slow', 2000);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
"Request to /api/slow timed out after 2000ms"
```

### Explanation

The `setTimeout` fires after 2 seconds and calls `abort()`, which cancels the pending `fetch`. The catch block receives an `AbortError`. If the server had responded within 2 seconds, `clearTimeout(timerId)` would have prevented the abort. This is the standard pattern for implementing fetch timeouts — `fetch` has no built-in timeout option.

</details>

---

## 7. Advanced Networking Questions

---

### Q1. What will be the output of this XMLHttpRequest sequence?

```js
const xhr = new XMLHttpRequest();

console.log(xhr.readyState); // before open

xhr.open('GET', '/api/data');
console.log(xhr.readyState); // after open

xhr.onreadystatechange = () => {
  if (xhr.readyState === 4) {
    console.log('Done, status:', xhr.status);
  }
};

xhr.send();
// After response arrives, what sequence of readyState values fires in onreadystatechange?
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
0
1
Done, status: 200
```

### Explanation

`onreadystatechange` fires at states 2 (HEADERS_RECEIVED), 3 (LOADING), and 4 (DONE). By only logging at `readyState === 4`, we only see the final "Done" line. The full sequence of `readyState` values the handler receives is 2 → 3 → 4. State 0 (UNSENT) and 1 (OPENED) do not trigger `onreadystatechange`.

</details>

---

### Q2. What will be the output when a real network failure occurs?

```js
// Device is offline or DNS cannot resolve the host
fetch('https://definitely-not-a-real-domain-xyz.invalid/api')
  .then(res => {
    console.log('Status:', res.status);
    return res.json();
  })
  .catch(err => {
    console.log(err instanceof TypeError);
    console.log(err.message);
  });
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
true
"Failed to fetch"
```

### Explanation

A **network-level failure** (DNS error, no internet, connection refused, timeout at the TCP level) causes `fetch` to reject with a `TypeError` — the only case where `fetch` rejects on its own. The error message is `"Failed to fetch"` in Chrome or `"NetworkError when attempting to fetch resource"` in Firefox. This is distinct from an HTTP error status (404, 500), which does NOT cause rejection.

</details>

---

### Q3. What will be the output?

```js
// Demonstrating the difference between WebSocket and HTTP polling

const results = [];

// Simulate WebSocket receiving messages (server pushes)
function simulateWebSocket() {
  const messages = ['msg1', 'msg2', 'msg3'];
  messages.forEach((msg, i) => {
    setTimeout(() => {
      results.push(`WS: ${msg}`);
      if (i === messages.length - 1) {
        console.log(results.join(', '));
      }
    }, i * 10);
  });
}

simulateWebSocket();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
"WS: msg1, WS: msg2, WS: msg3"
```

### Explanation

With WebSockets, the server pushes all messages over the same open connection with minimal overhead. With HTTP polling, each message would require a separate request/response cycle plus the full HTTP header overhead. WebSockets are preferred for high-frequency, bidirectional, or server-initiated communication, while SSE is preferred for one-directional server-push at lower frequency.

</details>

---

### Q4. What will be the output?

```js
// Does fetch send cookies cross-origin by default?

fetch('https://api.otherdomain.com/user', {
  // credentials: 'omit' is the default for cross-origin
})
  .then(res => res.json())
  .then(data => console.log(data));

// Compare with:
fetch('https://api.otherdomain.com/user', {
  credentials: 'include'   // explicitly send cookies and auth headers cross-origin
});
// Note: server must also include:
// Access-Control-Allow-Credentials: true
// Access-Control-Allow-Origin: https://myapp.com  (must be explicit, not *)
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
// First fetch (default 'omit'): cookies NOT sent, server sees unauthenticated request
{ error: "unauthenticated" }

// Second fetch ('include'): cookies ARE sent, but server must have:
// Access-Control-Allow-Credentials: true
// Access-Control-Allow-Origin: https://myapp.com  (wildcard * is not allowed with credentials)
```

### Explanation

The `credentials` option controls whether cookies and `Authorization` headers are included:
- `"omit"` — never send credentials (default for cross-origin)
- `"same-origin"` — send only for same-origin requests (default when option is not set)
- `"include"` — always send credentials; requires `Access-Control-Allow-Credentials: true` on the server and an explicit (non-wildcard) `Access-Control-Allow-Origin`

</details>

---

## Final Tips

- `fetch` **does not reject** on HTTP errors (4xx, 5xx) — always check `res.ok` or `res.status`.
- CORS is a **browser restriction** — the request reaches the server, but the response is blocked on the client if headers are missing.
- A CORS preflight (`OPTIONS`) is triggered by non-simple methods, custom headers, or non-simple `Content-Type`.
- `no-cors` mode gives you an **opaque response** — status `0`, no readable body.
- `AbortError` should be distinguished from real network errors in catch blocks.
- Once an `AbortController` is aborted, create a new one — you cannot reuse it.
- WebSocket events fire in order: `open` → `message` (multiple) → `close`.
- `send()` on a WebSocket in `CONNECTING` state throws `InvalidStateError`.
- `fetch` follows redirects automatically — JavaScript sees the final 200, not the 301/302.
- For parallel independent requests, use `Promise.all([fetch(...), fetch(...)])` — not sequential `await`.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
