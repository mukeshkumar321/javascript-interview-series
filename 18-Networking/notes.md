# Networking in JavaScript

## Table of Contents

1. [HTTP vs HTTPS](#1-http-vs-https)
2. [HTTP Methods](#2-http-methods)
3. [HTTP Status Codes](#3-http-status-codes)
4. [HTTP Headers](#4-http-headers)
5. [CORS — Cross-Origin Resource Sharing](#5-cors--cross-origin-resource-sharing)
6. [CORS Preflight Request](#6-cors-preflight-request)
7. [Same-Origin Policy](#7-same-origin-policy)
8. [XMLHttpRequest](#8-xmlhttprequest)
9. [Fetch API](#9-fetch-api)
10. [Request Modes](#10-request-modes)
11. [Fetch with async/await and Error Handling](#11-fetch-with-asyncawait-and-error-handling)
12. [AbortController](#12-abortcontroller)
13. [WebSockets](#13-websockets)
14. [WebSocket vs HTTP](#14-websocket-vs-http)
15. [Server-Sent Events](#15-server-sent-events)
16. [Long Polling vs Short Polling](#16-long-polling-vs-short-polling)
17. [HTTP/1.1 vs HTTP/2 vs HTTP/3](#17-http11-vs-http2-vs-http3)
18. [REST API Principles](#18-rest-api-principles)
19. [GraphQL Basics](#19-graphql-basics)
20. [REST vs GraphQL](#20-rest-vs-graphql)
21. [Summary](#21-summary)

---

## 1. HTTP vs HTTPS

**HTTP** (HyperText Transfer Protocol) is a plain-text application layer protocol. **HTTPS** adds a **TLS/SSL** layer that encrypts the connection between client and server.

| Feature | HTTP | HTTPS |
|---|---|---|
| Encryption | None | TLS/SSL |
| Default port | 80 | 443 |
| Data in transit | Visible to attackers (MITM) | Encrypted |
| Required for | — | Service Workers, Geolocation, camera/mic APIs |
| SEO / Browser trust | Lower | Higher (padlock icon) |

```js
// Mixing HTTP and HTTPS causes a Mixed Content error in browsers
// HTTPS page loading an HTTP resource is blocked by the browser

fetch('http://api.example.com/data')     // Called from https://mysite.com
  .then(res => console.log(res.status))
  .catch(err => console.log(err.message));
```

### Output

```js
"Failed to fetch"
// Mixed Content: The page at 'https://mysite.com' was loaded over HTTPS,
// but requested an insecure resource 'http://api.example.com/data'.
```

---

## 2. HTTP Methods

HTTP methods (also called verbs) define the **intended action** on a resource.

| Method | Purpose | Body | Idempotent | Safe |
|---|---|---|---|---|
| `GET` | Read a resource | No | Yes | Yes |
| `POST` | Create a resource | Yes | No | No |
| `PUT` | Replace a resource entirely | Yes | Yes | No |
| `PATCH` | Partially update a resource | Yes | No | No |
| `DELETE` | Remove a resource | Optional | Yes | No |
| `HEAD` | Like GET but returns headers only | No | Yes | Yes |
| `OPTIONS` | List allowed methods / CORS preflight | No | Yes | Yes |

```js
// GET: retrieve
fetch('/api/users/1');

// POST: create
fetch('/api/users', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ name: 'Alice' })
});

// DELETE: remove
fetch('/api/users/1', { method: 'DELETE' });

// Checking allowed methods via OPTIONS
fetch('/api/users', { method: 'OPTIONS' }).then(res => {
  console.log(res.headers.get('Allow'));
});
```

### Output

```js
// OPTIONS response header example:
"GET, POST, PUT, PATCH, DELETE, OPTIONS"
```

---

## 3. HTTP Status Codes

Status codes tell the client what happened to the request.

| Range | Category | Common Examples |
|---|---|---|
| 2xx | Success | 200 OK, 201 Created, 204 No Content |
| 3xx | Redirection | 301 Moved Permanently, 302 Found, 304 Not Modified |
| 4xx | Client Error | 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 409 Conflict, 422 Unprocessable |
| 5xx | Server Error | 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable |

```js
fetch('/api/resource').then((res) => {
  console.log(res.status);  // e.g. 200

  // res.ok is true for 200-299, false for anything else
  console.log(res.ok);      // true for 2xx

  if (res.status === 401) {
    console.log('Not authenticated — redirect to login');
  }
  if (res.status === 404) {
    console.log('Resource not found');
  }
});
```

### Output

```js
200
true
```

---

## 4. HTTP Headers

Headers are key-value metadata sent with requests and responses. They control content negotiation, authentication, caching, CORS, and more.

| Header | Direction | Purpose |
|---|---|---|
| `Content-Type` | Both | Media type of the body (e.g., `application/json`) |
| `Accept` | Request | Media types the client can handle |
| `Authorization` | Request | Credentials (e.g., `Bearer <token>`) |
| `Origin` | Request | The origin making the request (set by browser for cross-origin) |
| `Access-Control-Allow-Origin` | Response | Allows a specific origin (CORS) |
| `Access-Control-Allow-Methods` | Response | Allowed HTTP methods (CORS) |
| `Access-Control-Allow-Headers` | Response | Allowed request headers (CORS) |
| `Access-Control-Allow-Credentials` | Response | Whether credentials (cookies) are allowed |

```js
fetch('/api/data', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Accept': 'application/json',
    'Authorization': 'Bearer eyJhbGciOiJIUzI1NiJ9...'
  },
  body: JSON.stringify({ key: 'value' })
}).then((res) => {
  console.log(res.headers.get('Content-Type'));
  console.log(res.headers.get('Access-Control-Allow-Origin'));
});
```

### Output

```js
"application/json"
"https://myapp.com"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 5. CORS — Cross-Origin Resource Sharing

CORS is a browser security mechanism that controls which **cross-origin** requests are allowed. When your JavaScript on `https://app.com` calls `https://api.example.com`, the server at `api.example.com` must explicitly permit it.

The server grants permission by including `Access-Control-Allow-Origin` in its response. If the header is absent or does not match the requesting origin, the browser **blocks** the response (even though the request was sent and received).

```js
// From https://app.com, calling a different origin
fetch('https://api.example.com/data')
  .then(res => {
    // Browser checks: does the response have the right CORS headers?
    console.log(res.headers.get('Access-Control-Allow-Origin'));
    return res.json();
  })
  .then(data => console.log(data))
  .catch(err => console.log('CORS error:', err.message));

// If server includes: Access-Control-Allow-Origin: https://app.com
// — the request succeeds

// If server includes no CORS headers:
// — the browser blocks the response; .catch fires
```

### Output

```js
// When server allows the origin:
"https://app.com"
{ /* response data */ }

// When server does not allow:
"CORS error: Failed to fetch"
```

---

## 6. CORS Preflight Request

A **preflight** is an automatic `OPTIONS` request the browser sends before a "non-simple" cross-origin request to check whether the server accepts it.

A request is **non-simple** (triggers preflight) when it uses:
- Methods other than GET, HEAD, or POST
- Custom headers (e.g., `Authorization`)
- `Content-Type` other than `text/plain`, `multipart/form-data`, or `application/x-www-form-urlencoded`

```js
// This POST with JSON body and Authorization header triggers a preflight

fetch('https://api.example.com/users', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',     // non-simple Content-Type
    'Authorization': 'Bearer token123'       // custom header
  },
  body: JSON.stringify({ name: 'Alice' })
});

// Browser first sends:
// OPTIONS https://api.example.com/users
// Origin: https://app.com
// Access-Control-Request-Method: POST
// Access-Control-Request-Headers: Content-Type, Authorization

// Server must respond with (for the POST to proceed):
// Access-Control-Allow-Origin: https://app.com
// Access-Control-Allow-Methods: POST
// Access-Control-Allow-Headers: Content-Type, Authorization
```

### Output

```js
// Developer Tools > Network shows TWO requests:
// 1. OPTIONS /users  — preflight
// 2. POST    /users  — actual request (only if preflight succeeds)
```

---

## 7. Same-Origin Policy

The **Same-Origin Policy (SOP)** is a fundamental browser security rule: a script running on origin A may not freely read resources from origin B. Two URLs share the same origin only if their **scheme, host, and port** all match.

| URL A | URL B | Same Origin? |
|---|---|---|
| `https://example.com/a` | `https://example.com/b` | Yes |
| `https://example.com` | `http://example.com` | No (different scheme) |
| `https://example.com` | `https://api.example.com` | No (different host) |
| `https://example.com:443` | `https://example.com:8443` | No (different port) |

```js
// SOP allows sending requests cross-origin but blocks reading the response
// CORS is the mechanism that selectively relaxes the read restriction

// This will SEND but the browser will BLOCK the response if no CORS headers:
fetch('https://other-origin.com/data')
  .then(res => res.json())        // blocked here if CORS not configured
  .catch(err => console.log('Blocked by SOP / CORS:', err.message));
```

### Output

```js
"Blocked by SOP / CORS: Failed to fetch"
```

---

## 8. XMLHttpRequest

`XMLHttpRequest` (XHR) is the original browser API for making HTTP requests, introduced before `fetch`. It is event-based and more verbose. You will encounter it in legacy codebases and interview questions.

```js
const xhr = new XMLHttpRequest();

console.log(xhr.readyState); // 0 — UNSENT

xhr.open('GET', '/api/data');
console.log(xhr.readyState); // 1 — OPENED

xhr.onreadystatechange = () => {
  // States: 0 UNSENT, 1 OPENED, 2 HEADERS_RECEIVED, 3 LOADING, 4 DONE
  if (xhr.readyState === 4) {
    if (xhr.status === 200) {
      console.log('Response:', JSON.parse(xhr.responseText));
    } else {
      console.log('Error:', xhr.status);
    }
  }
};

xhr.send();
```

### Output

```js
0
1
// When response arrives:
"Response:" { /* parsed JSON */ }
```

---

## 9. Fetch API

`fetch` is the modern Promise-based API for HTTP requests. It is covered in detail in **16-Browser-APIs**. Key points for context:

- Returns a `Promise<Response>`
- Does **not** reject on HTTP error status codes (4xx, 5xx) — only on network failure
- Use `res.ok` or check `res.status` manually

```js
// Basic fetch — detail is in 16-Browser-APIs
fetch('/api/users')
  .then(res => {
    console.log(res.status); // 200
    console.log(res.ok);     // true
    return res.json();
  })
  .then(data => console.log('Users:', data))
  .catch(err => console.log('Network error:', err.message));
```

### Output

```js
200
true
"Users:" [{ id: 1, name: "Alice" }]
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 10. Request Modes

The `mode` option of a `fetch` call controls how cross-origin requests are handled.

| Mode | Behaviour |
|---|---|
| `"cors"` | Default for cross-origin requests. CORS headers required on the server. |
| `"same-origin"` | Request fails immediately if the URL is a different origin. |
| `"no-cors"` | Request is sent but the response is **opaque**: status is `0`, body is unreadable. |
| `"navigate"` | Used by the browser for top-level navigation (not for manual use). |

```js
// no-cors: request goes through but you cannot read the response
fetch('https://other-origin.com/image.png', { mode: 'no-cors' })
  .then(res => {
    console.log(res.type);   // "opaque"
    console.log(res.status); // 0
    console.log(res.ok);     // false
  });

// same-origin: rejects immediately for cross-origin URLs
fetch('https://other-origin.com/data', { mode: 'same-origin' })
  .catch(err => console.log('Error:', err.message));
```

### Output

```js
"opaque"
0
false
"Error: Failed to fetch"
```

---

## 11. Fetch with async/await and Error Handling

`fetch` does not throw on HTTP errors. A robust pattern is to check `res.ok` and throw manually, then catch both HTTP errors and network errors in one `catch` block.

```js
async function getUser(id) {
  try {
    const res = await fetch(`/api/users/${id}`);

    if (!res.ok) {
      // Throw so the catch block handles HTTP errors too
      throw new Error(`HTTP Error: ${res.status} ${res.statusText}`);
    }

    const user = await res.json();
    console.log('User:', user.name);
    return user;
  } catch (err) {
    // Handles both network errors and our thrown HTTP errors
    console.log('Failed:', err.message);
    return null;
  }
}

getUser(1);   // Existing user
getUser(999); // Returns 404
```

### Output

```js
// For getUser(1) — server returns 200:
"User: Alice"

// For getUser(999) — server returns 404:
"Failed: HTTP Error: 404 Not Found"
```

---

## 12. AbortController

`AbortController` lets you cancel an in-flight `fetch` request. This is useful for search-as-you-type (cancel previous requests) and for implementing timeouts.

```js
const controller = new AbortController();
const { signal } = controller;

fetch('/api/slow-data', { signal })
  .then(res => res.json())
  .then(data => console.log('Data:', data))
  .catch(err => {
    if (err.name === 'AbortError') {
      console.log('Request was cancelled');
    } else {
      console.log('Network error:', err.message);
    }
  });

// Cancel the request after 3 seconds
const timeoutId = setTimeout(() => controller.abort(), 3000);

// Or cancel immediately:
// controller.abort();
```

### Output

```js
// If response arrives before 3 seconds:
"Data:" { /* response data */ }

// If 3 seconds pass before response:
"Request was cancelled"
```

---

## 13. WebSockets

A WebSocket provides a **persistent, full-duplex** (bidirectional) connection between client and server over a single TCP connection. After the initial HTTP handshake upgrade, both sides can send messages at any time with very low overhead.

Use cases: live chat, real-time dashboards, multiplayer games, collaborative editors.

```js
const ws = new WebSocket('wss://chat.example.com/socket');

ws.onopen = () => {
  console.log('Connection opened');
  ws.send(JSON.stringify({ type: 'join', room: 'general' }));
};

ws.onmessage = (event) => {
  const msg = JSON.parse(event.data);
  console.log('Received:', msg);
};

ws.onerror = (event) => {
  console.log('WebSocket error');
};

ws.onclose = (event) => {
  console.log('Connection closed, code:', event.code);
};
```

### Output

```js
"Connection opened"
"Received:" { type: "message", text: "Hello!", user: "Bob" }
"Connection closed, code: 1000"
```

---

## 14. WebSocket vs HTTP

| Feature | HTTP | WebSocket |
|---|---|---|
| Connection | New per request (HTTP/1.1) or multiplexed (HTTP/2) | Single persistent TCP connection |
| Communication | Client-initiated only (request/response) | Full-duplex: both sides can send anytime |
| Overhead | Headers on every request/response | Minimal framing after handshake |
| Use case | REST APIs, static resources | Chat, live feeds, gaming, collaboration |
| Protocol | `http://` / `https://` | `ws://` / `wss://` |
| Browser API | `fetch`, `XMLHttpRequest` | `WebSocket` |

```js
// WebSocket readyState constants
const ws = new WebSocket('wss://echo.example.com');
console.log(ws.readyState); // 0 — WebSocket.CONNECTING

ws.onopen = () => {
  console.log(ws.readyState); // 1 — WebSocket.OPEN
  ws.close();
};

ws.onclose = () => {
  console.log(ws.readyState); // 3 — WebSocket.CLOSED
};

// States: 0=CONNECTING, 1=OPEN, 2=CLOSING, 3=CLOSED
```

### Output

```js
0
1
3
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 15. Server-Sent Events

**Server-Sent Events (SSE)** allow the server to push updates to the client over a persistent HTTP connection. Unlike WebSockets, SSE is **unidirectional** (server to client only) and uses plain HTTP, making it simpler to implement and compatible with HTTP/2 multiplexing.

```js
// Client side
const evtSource = new EventSource('/api/live-updates');

evtSource.onopen = () => {
  console.log('SSE connection opened');
};

evtSource.onmessage = (event) => {
  console.log('Update received:', event.data);
};

// Named events (server sends: event: stockPrice\ndata: 150.25\n\n)
evtSource.addEventListener('stockPrice', (event) => {
  console.log('Stock price:', event.data);
});

evtSource.onerror = () => {
  console.log('SSE error — browser will auto-reconnect');
  evtSource.close(); // or let it retry automatically
};
```

### Output

```js
"SSE connection opened"
"Update received: { status: 'ok' }"
"Stock price: 150.25"
```

---

## 16. Long Polling vs Short Polling

When WebSockets or SSE are not available, polling is a fallback strategy to simulate real-time updates.

| Strategy | How it works | Latency | Server load |
|---|---|---|---|
| Short Polling | Client repeats requests at a fixed interval | High (interval delay) | High (many requests) |
| Long Polling | Client sends a request; server holds it open until data is available, then responds; client immediately re-requests | Low | Medium |

```js
// Short polling — request every 2 seconds
function shortPoll() {
  setInterval(async () => {
    const res = await fetch('/api/notifications');
    const data = await res.json();
    if (data.length > 0) {
      console.log('New notifications:', data);
    }
  }, 2000);
}

// Long polling — server holds response until data available
async function longPoll() {
  while (true) {
    const res = await fetch('/api/wait-for-update'); // server waits up to 30s
    const data = await res.json();
    console.log('Update:', data);
    // Immediately re-request
  }
}
```

### Output

```js
// Short poll (every 2 seconds):
"New notifications:" [{ id: 1, text: "You have a new message" }]

// Long poll (whenever server has data):
"Update:" { event: "message", payload: "Hello" }
```

---

## 17. HTTP/1.1 vs HTTP/2 vs HTTP/3

Each version improves the performance and efficiency of web communication.

| Feature | HTTP/1.1 | HTTP/2 | HTTP/3 |
|---|---|---|---|
| Multiplexing | No (head-of-line blocking) | Yes (multiple streams on one TCP connection) | Yes |
| Transport | TCP | TCP | QUIC (UDP-based) |
| Header compression | None | HPACK | QPACK |
| Server push | No | Yes | Yes |
| TLS required | No | De facto required | Built-in (always encrypted) |
| Connection reuse | Keep-Alive (limited) | Single multiplexed connection | QUIC streams |

```js
// You cannot directly choose HTTP version from JavaScript —
// the browser and server negotiate it automatically.
// You can observe the protocol in DevTools > Network > Protocol column.

fetch('/api/data').then(res => {
  // No standard API to read HTTP version from fetch response
  console.log(res.status); // 200
  // Check DevTools Network tab — it will show "h2" for HTTP/2 or "h3" for HTTP/3
});
```

### Output

```js
200
// DevTools Network tab: Protocol column shows "h2" or "h3"
```

---

## 18. REST API Principles

REST (Representational State Transfer) is an architectural style for designing networked APIs. A RESTful API organises resources as URLs and uses HTTP methods to express operations.

Six constraints define REST:
1. **Client-Server** — UI and data are separated
2. **Stateless** — Each request is self-contained; no server-side session
3. **Cacheable** — Responses must define whether they can be cached
4. **Uniform Interface** — Resource-based URLs, HTTP verbs, standard representations
5. **Layered System** — Client does not know if it talks to a proxy or the real server
6. **Code on Demand** (optional) — Server can send executable code

```js
// RESTful resource design for a Users API:
// GET    /users         — list all users
// POST   /users         — create a user
// GET    /users/:id     — get one user
// PUT    /users/:id     — replace a user
// PATCH  /users/:id     — partially update a user
// DELETE /users/:id     — delete a user

fetch('/users/42')
  .then(res => res.json())
  .then(user => console.log(user));

fetch('/users', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ name: 'Alice', email: 'alice@example.com' })
}).then(res => {
  console.log(res.status); // 201 Created
});
```

### Output

```js
{ id: 42, name: "Bob", email: "bob@example.com" }
201
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 19. GraphQL Basics

GraphQL is a query language for APIs that lets the client specify **exactly what data it needs**. All requests go to a single endpoint (usually `POST /graphql`).

- **Query** — read data
- **Mutation** — write/modify data
- **Subscription** — real-time updates over WebSocket

```js
// GraphQL query — ask for exactly the fields you need
const query = `
  query GetUser($id: ID!) {
    user(id: $id) {
      id
      name
      email
    }
  }
`;

fetch('/graphql', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    query,
    variables: { id: '42' }
  })
})
  .then(res => res.json())
  .then(({ data, errors }) => {
    if (errors) {
      console.log('GraphQL errors:', errors);
    } else {
      console.log('User:', data.user.name);
    }
  });
```

### Output

```js
"User: Alice"
```

---

## 20. REST vs GraphQL

| Feature | REST | GraphQL |
|---|---|---|
| Endpoints | Many (`/users`, `/posts`, etc.) | Single (`/graphql`) |
| Over-fetching | Common — server decides shape | None — client specifies fields |
| Under-fetching | Common — multiple requests for related data | None — nest related data in one query |
| HTTP methods | GET, POST, PUT, PATCH, DELETE | Usually just POST |
| Real-time | Via SSE or WebSocket (separate setup) | Subscriptions built in |
| Caching | Straightforward (HTTP cache by URL) | More complex (query-level caching) |
| Type system | No built-in | Strongly typed schema |
| Learning curve | Low | Medium |

```js
// REST: may need two requests to get user + their posts
const user  = await fetch('/users/1').then(r => r.json());
const posts = await fetch('/users/1/posts').then(r => r.json());
console.log(user.name, '—', posts.length, 'posts');

// GraphQL: get user + posts in one query
const { data } = await fetch('/graphql', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ query: `{ user(id:1){ name posts{ title } } }` })
}).then(r => r.json());
console.log(data.user.name, '—', data.user.posts.length, 'posts');
```

### Output

```js
"Alice — 5 posts"
"Alice — 5 posts"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 21. Summary

| Topic | Key Point |
|---|---|
| HTTP vs HTTPS | HTTPS encrypts traffic with TLS; required for Service Workers and sensitive APIs |
| HTTP Methods | GET=read, POST=create, PUT=replace, PATCH=partial update, DELETE=remove |
| Status Codes | 2xx=success, 3xx=redirect, 4xx=client error, 5xx=server error |
| HTTP Headers | Content-Type, Authorization, Accept control request/response metadata |
| CORS | Server must include `Access-Control-Allow-Origin` to allow cross-origin reads |
| Preflight | OPTIONS request sent automatically before non-simple cross-origin requests |
| Same-Origin Policy | Browser blocks cross-origin response reads; CORS relaxes this |
| XMLHttpRequest | Legacy event-based API; `readyState` goes 0→1→2→3→4 |
| Fetch API | Promise-based; does NOT reject on 4xx/5xx — check `res.ok` manually |
| Request Modes | `cors` (default), `same-origin` (restrict), `no-cors` (opaque response) |
| AbortController | Cancel in-flight fetch; rejects with `AbortError` |
| WebSockets | Persistent, full-duplex, low-overhead; `ws://` / `wss://` |
| SSE | Server-push only, over HTTP, auto-reconnect; simpler than WebSockets |
| Long vs Short Polling | Long polling holds the connection for lower latency vs fixed-interval short poll |
| HTTP/2 | Multiplexing on single connection eliminates HTTP/1.1 head-of-line blocking |
| REST | Resource-based URLs, HTTP verbs, stateless, predictable |
| GraphQL | Single endpoint, client-specified fields, no over/under-fetching |

---

## Final Notes

Networking is one of the most heavily tested areas in JavaScript interviews because it touches async programming, security (CORS, SOP), performance (caching, HTTP/2), and real-time communication (WebSockets, SSE). Remember the two most common traps: `fetch` does not reject on HTTP errors — you must check `res.ok`; and CORS is enforced by the **browser**, not the server — the request still reaches the server but the response is blocked on the client side if headers are missing.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
