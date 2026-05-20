# DOM Manipulation in JavaScript

## Table of Contents

1. [What is the DOM?](#1-what-is-the-dom)
2. [getElementById / querySelector / querySelectorAll](#2-getelementbyid--queryselector--queryselectorall)
3. [getElementsByClassName / getElementsByTagName](#3-getelementsbyclassname--getelementsbytagname)
4. [Creating Elements](#4-creating-elements)
5. [Appending Elements](#5-appending-elements)
6. [Removing Elements](#6-removing-elements)
7. [Modifying Elements](#7-modifying-elements)
8. [innerHTML vs textContent (Security)](#8-innerhtml-vs-textcontent-security)
9. [Attribute Manipulation](#9-attribute-manipulation)
10. [Class Manipulation](#10-class-manipulation)
11. [Style Manipulation](#11-style-manipulation)
12. [DOM Traversal](#12-dom-traversal)
13. [DocumentFragment](#13-documentfragment)
14. [Cloning Elements](#14-cloning-elements)
15. [Reflow and Repaint](#15-reflow-and-repaint)
16. [Batch DOM Updates](#16-batch-dom-updates)
17. [Virtual DOM Concept](#17-virtual-dom-concept)
18. [DOMContentLoaded vs load](#18-domcontentloaded-vs-load)
19. [MutationObserver](#19-mutationobserver)
20. [Summary](#20-summary)

---

## 1. What is the DOM?

The **Document Object Model (DOM)** is a tree-structured, in-memory representation of an HTML document. The browser parses HTML and creates a live object graph where every element, attribute, and text node is a JavaScript object you can read and modify. Changes to the DOM are reflected immediately in the rendered page.

```js
// The browser gives you the root via:
console.log(typeof document);         // "object"
console.log(document.nodeType);       // 9  (DOCUMENT_NODE)
console.log(document.documentElement.tagName); // "HTML"
```

### Output

```js
"object"
9
"HTML"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 2. getElementById / querySelector / querySelectorAll

`getElementById` returns a single element by its `id`. `querySelector` returns the **first** element matching a CSS selector. `querySelectorAll` returns a **static** `NodeList` of all matches.

```js
// Assume: <p id="msg" class="text">Hello</p>

const byId   = document.getElementById('msg');
const first  = document.querySelector('.text');
const all    = document.querySelectorAll('p');

console.log(byId === first);   // true  — same element
console.log(all instanceof NodeList); // true
console.log(all.length);       // 1
```

### Output

```js
true
true
1
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 3. getElementsByClassName / getElementsByTagName

Both methods return an **HTMLCollection**, which is **live** — it updates automatically when the DOM changes. This is the key difference from `querySelectorAll` which returns a static `NodeList`.

```js
// Assume: <ul><li class="item">A</li><li class="item">B</li></ul>

const items = document.getElementsByClassName('item');
console.log(items.length); // 2

// Add a new matching element
const li = document.createElement('li');
li.className = 'item';
document.querySelector('ul').appendChild(li);

console.log(items.length); // 3  — live collection updated automatically
```

### Output

```js
2
3
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 4. Creating Elements

Use `document.createElement(tagName)` to create a new element node and `document.createTextNode(text)` to create a text node. The created nodes are not in the document until you append them.

```js
const div  = document.createElement('div');
div.id     = 'box';

const text = document.createTextNode('Hello DOM');
div.appendChild(text);

console.log(div.outerHTML);          // <div id="box">Hello DOM</div>
console.log(div.isConnected);        // false — not yet in the document
```

### Output

```js
"<div id=\"box\">Hello DOM</div>"
false
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 5. Appending Elements

Several methods exist to insert nodes at different positions.

| Method | Position |
|---|---|
| `appendChild(node)` | Last child (returns the node) |
| `append(...nodes)` | Last child, accepts strings too (returns undefined) |
| `prepend(...nodes)` | First child |
| `insertBefore(new, ref)` | Before a reference child |
| `insertAdjacentHTML(pos, html)` | Relative to element by position string |

```js
const ul = document.createElement('ul');

ul.append('First item as text'); // text node shorthand
const li = document.createElement('li');
li.textContent = 'Second';
ul.appendChild(li);

const first = document.createElement('li');
first.textContent = 'Zero';
ul.prepend(first);

console.log(ul.children.length); // 2  (text nodes not counted in .children)
console.log(ul.childNodes.length); // 3  (text node + li + li)
```

### Output

```js
2
3
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 6. Removing Elements

`removeChild` is the older pattern; `remove()` is the modern, simpler method.

```js
// Old way
const parent = document.getElementById('list');
const child  = document.getElementById('item1');
parent.removeChild(child); // returns the removed node

// Modern way
document.getElementById('item2').remove(); // no return value needed

// Check if still connected
console.log(child.isConnected);  // false
```

### Output

```js
false
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 7. Modifying Elements

Three properties let you read and write element content:

| Property | Parses HTML? | Shows hidden text? | Use case |
|---|---|---|---|
| `innerHTML` | Yes | No | Read/write HTML markup |
| `textContent` | No | Yes | Plain text, all nodes |
| `innerText` | No | No | Visible rendered text |

```js
const div = document.createElement('div');
div.innerHTML = '<b>Bold</b> text';

console.log(div.innerHTML);     // "<b>Bold</b> text"
console.log(div.textContent);   // "Bold text"
console.log(div.innerText);     // "Bold text"

// textContent vs innerText difference
const hidden = document.createElement('span');
hidden.style.display = 'none';
hidden.textContent = 'secret';
div.appendChild(hidden);

console.log(div.textContent);  // "Bold textsecret"
console.log(div.innerText);    // "Bold text"  (hidden element excluded)
```

### Output

```js
"<b>Bold</b> text"
"Bold text"
"Bold text"
"Bold textsecret"
"Bold text"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 8. innerHTML vs textContent (Security)

Setting `innerHTML` with untrusted user input is an **XSS vulnerability** because the browser parses and executes injected scripts. Always use `textContent` for plain text or sanitize before using `innerHTML`.

```js
// DANGEROUS — never do this with user input
const userInput = '<img src=x onerror="alert(\'XSS\')">';
// document.body.innerHTML = userInput; // executes the onerror handler!

// SAFE — textContent treats everything as plain text
const div = document.createElement('div');
div.textContent = userInput;
console.log(div.innerHTML);
// Outputs escaped HTML — no script execution

// Also safe: DOMPurify library
// div.innerHTML = DOMPurify.sanitize(userInput);
```

### Output

```js
"&lt;img src=x onerror=&quot;alert('XSS')&quot;&gt;"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 9. Attribute Manipulation

Use the attribute API to read, write, remove, and check attributes. The `dataset` property provides clean access to `data-*` attributes.

```js
const btn = document.createElement('button');

btn.setAttribute('type', 'submit');
btn.setAttribute('data-id', '42');
btn.setAttribute('disabled', '');

console.log(btn.getAttribute('type'));     // "submit"
console.log(btn.hasAttribute('disabled')); // true

btn.removeAttribute('disabled');
console.log(btn.hasAttribute('disabled')); // false

// dataset
console.log(btn.dataset.id);               // "42"
btn.dataset.role = 'admin';
console.log(btn.getAttribute('data-role')); // "admin"
```

### Output

```js
"submit"
true
false
"42"
"admin"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 10. Class Manipulation

`classList` provides a clean API for managing CSS classes without manually parsing `className` strings.

```js
const el = document.createElement('div');
el.className = 'box';

el.classList.add('active', 'visible');
console.log(el.className);             // "box active visible"

el.classList.remove('visible');
console.log(el.classList.contains('active'));  // true
console.log(el.classList.contains('visible')); // false

el.classList.toggle('dark');
console.log(el.classList.contains('dark'));    // true

el.classList.toggle('dark');
console.log(el.classList.contains('dark'));    // false

el.classList.replace('active', 'inactive');
console.log(el.className);             // "box inactive"
```

### Output

```js
"box active visible"
true
false
true
false
"box inactive"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 11. Style Manipulation

`element.style` sets **inline styles** and only reflects styles set directly on the element. `getComputedStyle` returns the final resolved style including stylesheets.

```js
const div = document.createElement('div');
document.body.appendChild(div);

div.style.color       = 'red';
div.style.fontSize    = '16px';
div.style.display     = 'block';

console.log(div.style.color);           // "red"
console.log(div.style.backgroundColor); // ""  (not set inline)

const computed = getComputedStyle(div);
console.log(computed.display);          // "block"
// computed.color would be "rgb(255, 0, 0)"
```

### Output

```js
"red"
""
"block"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 12. DOM Traversal

Navigate the DOM tree using relationship properties. Note the distinction between properties that include all node types (including text nodes) and those restricted to element nodes.

| Property | Includes Text Nodes? |
|---|---|
| `parentNode` | Yes |
| `parentElement` | No (elements only) |
| `childNodes` | Yes |
| `children` | No |
| `firstChild` | Yes |
| `firstElementChild` | No |
| `lastChild` | Yes |
| `lastElementChild` | No |
| `nextSibling` | Yes |
| `nextElementSibling` | No |
| `previousSibling` | Yes |
| `previousElementSibling` | No |

```js
// <div id="parent"><span>A</span><span>B</span></div>
const parent = document.getElementById('parent');

console.log(parent.childNodes.length);       // 2 (spans, no whitespace in this case)
console.log(parent.children.length);         // 2
console.log(parent.firstElementChild.textContent);  // "A"
console.log(parent.lastElementChild.textContent);   // "B"

const spanA = parent.firstElementChild;
console.log(spanA.nextElementSibling.textContent);  // "B"
console.log(spanA.parentElement.id);                // "parent"
```

### Output

```js
2
2
"A"
"B"
"B"
"parent"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 13. DocumentFragment

A `DocumentFragment` is a lightweight, off-document container. Appending many nodes to a fragment and then inserting the fragment causes only **one reflow** instead of one per node.

```js
const fragment = document.createDocumentFragment();

for (let i = 1; i <= 3; i++) {
  const li = document.createElement('li');
  li.textContent = `Item ${i}`;
  fragment.appendChild(li);
}

// fragment itself is not part of the document yet
console.log(fragment.childNodes.length); // 3

const ul = document.getElementById('list');
ul.appendChild(fragment); // single DOM insertion — one reflow

// fragment is now empty after insertion
console.log(fragment.childNodes.length); // 0
```

### Output

```js
3
0
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 14. Cloning Elements

`cloneNode(deep)` creates a copy of a node. Pass `true` to perform a **deep clone** (copies all descendants); `false` for a **shallow clone** (element only, no children).

```js
const original = document.createElement('div');
original.id = 'box';
original.innerHTML = '<span>child</span>';
original.addEventListener('click', () => console.log('clicked'));

const shallow = original.cloneNode(false);
const deep    = original.cloneNode(true);

console.log(shallow.id);                    // "box"
console.log(shallow.children.length);       // 0  (no children)
console.log(deep.children.length);          // 1  (span cloned)
console.log(deep.firstElementChild.tagName); // "SPAN"

// Event listeners are NOT cloned
// deep.click() would NOT log "clicked"
```

### Output

```js
"box"
0
1
"SPAN"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 15. Reflow and Repaint

**Reflow** (layout): the browser recalculates positions and sizes of elements. It is expensive because it can cascade to ancestor and descendant elements.

**Repaint**: the browser redraws pixels without recalculating layout (e.g., changing `color` or `background`). Cheaper than reflow.

**Triggers reflow:**
- Reading layout properties: `offsetWidth`, `offsetHeight`, `getBoundingClientRect()`, `scrollTop`
- Changing geometry: `width`, `height`, `padding`, `margin`, `font-size`
- Adding/removing DOM nodes that affect layout
- `window.getComputedStyle()`

**Triggers repaint only:**
- `color`, `background-color`, `box-shadow` (without geometry change)

```js
// BAD: forced synchronous layout (read-write-read-write interleaving)
const box = document.getElementById('box');
box.style.width = '100px';       // write
const w = box.offsetWidth;       // read  — forces reflow
box.style.height = w + 'px';     // write
const h = box.offsetHeight;      // read  — forces reflow again

// GOOD: batch reads first, then writes
const w2 = box.offsetWidth;      // read
const h2 = box.offsetHeight;     // read
box.style.width  = w2 + 10 + 'px'; // write
box.style.height = h2 + 10 + 'px'; // write (single reflow)
```

### Output

```js
// No console output — demonstrates the pattern
// Bad pattern: 2 forced reflows
// Good pattern: 1 reflow
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 16. Batch DOM Updates

Two main strategies to minimize layout thrashing: read-then-write batching and `DocumentFragment`.

```js
// Strategy 1: Read all, then write all
const elements = document.querySelectorAll('.box');
const widths = Array.from(elements).map(el => el.offsetWidth); // batch reads

widths.forEach((w, i) => {
  elements[i].style.width = (w * 2) + 'px'; // batch writes
});

// Strategy 2: DocumentFragment (see section 13)

// Strategy 3: requestAnimationFrame for visual updates
function updateUI() {
  requestAnimationFrame(() => {
    document.getElementById('counter').textContent = Date.now();
  });
}
```

### Output

```js
// No console.log — demonstrates pattern
// widths array filled in one pass, writes in one pass = one reflow
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 17. Virtual DOM Concept

The Virtual DOM is an in-memory JavaScript object representation of the real DOM, popularised by React. Instead of directly manipulating the expensive real DOM on every state change, frameworks:

1. Render a new virtual tree on state change
2. **Diff** the new virtual tree against the previous one
3. Calculate the minimal set of real DOM changes needed
4. Apply only those changes (**reconciliation**)

```js
// Simplified concept (not real React code)
const oldVNode = { type: 'div', props: { id: 'app' }, children: ['Hello'] };
const newVNode = { type: 'div', props: { id: 'app' }, children: ['Hello World'] };

function diff(oldNode, newNode) {
  // Only the text child changed — only update that text node
  if (oldNode.children[0] !== newNode.children[0]) {
    return [{ type: 'UPDATE_TEXT', value: newNode.children[0] }];
  }
  return [];
}

console.log(diff(oldVNode, newVNode));
// [{ type: 'UPDATE_TEXT', value: 'Hello World' }]
```

### Output

```js
[{ type: "UPDATE_TEXT", value: "Hello World" }]
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 18. DOMContentLoaded vs load

| Event | Fires when | Typical use |
|---|---|---|
| `DOMContentLoaded` | HTML parsed, DOM ready (no images/stylesheets) | Run JS that queries the DOM |
| `load` | All resources fully loaded (images, CSS, fonts) | Read image dimensions, measure layout |

```js
document.addEventListener('DOMContentLoaded', () => {
  console.log('DOM ready');
  // Safe to query elements here
  const h1 = document.querySelector('h1');
  console.log(h1 !== null); // true
});

window.addEventListener('load', () => {
  console.log('All resources loaded');
  // Image dimensions are reliable here
  const img = document.querySelector('img');
  console.log(img.naturalWidth > 0); // true
});

// Order is always: DOMContentLoaded fires before load
```

### Output

```js
"DOM ready"
true
"All resources loaded"
true
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 19. MutationObserver

`MutationObserver` watches for changes in the DOM tree and invokes a callback asynchronously (microtask queue after the current script). It is the modern replacement for the deprecated `MutationEvents`.

```js
const target = document.createElement('div');
document.body.appendChild(target);

const observer = new MutationObserver((mutations) => {
  for (const mutation of mutations) {
    console.log(mutation.type);          // "childList" or "attributes"
    console.log(mutation.addedNodes.length);
  }
});

observer.observe(target, {
  childList: true,   // watch for added/removed children
  attributes: true,  // watch for attribute changes
  subtree: true      // observe all descendants too
});

// Trigger mutations
const span = document.createElement('span');
target.appendChild(span); // triggers callback

// Later, stop observing
// observer.disconnect();
```

### Output

```js
"childList"
1
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 20. Summary

| Topic | Key Point |
|---|---|
| DOM | In-memory tree of HTML; live JavaScript objects |
| `querySelector` | Returns first match; static result |
| `getElementsByClassName` | Returns live HTMLCollection |
| `createElement` | Creates node not yet in document |
| `appendChild` vs `append` | `append` accepts strings; `appendChild` returns node |
| `innerHTML` | Parses HTML — XSS risk with user input |
| `textContent` | Plain text; includes hidden elements |
| `innerText` | Visible rendered text only |
| `dataset` | Clean API for `data-*` attributes |
| `classList` | `add`, `remove`, `toggle`, `contains`, `replace` |
| `getComputedStyle` | Resolves all CSS sources; `element.style` is inline only |
| `childNodes` vs `children` | `childNodes` includes text nodes; `children` elements only |
| `DocumentFragment` | Batch inserts with single reflow |
| `cloneNode(true)` | Deep clone; event listeners not copied |
| Reflow | Geometry change — expensive; batch writes after reads |
| Repaint | Visual change only — cheaper than reflow |
| Virtual DOM | Diff + reconcile to minimize real DOM ops |
| `DOMContentLoaded` | DOM parsed; fires before `load` |
| `load` | All resources ready |
| `MutationObserver` | Async DOM change watcher; replaces MutationEvents |

---

## Final Notes

DOM Manipulation is one of the most tested areas in JavaScript interviews because it sits at the intersection of the language and the browser platform. The most important distinctions to memorise are: `innerHTML` vs `textContent` (security and parsing), live `HTMLCollection` vs static `NodeList`, and the performance pattern of batching reads before writes to avoid forced synchronous layouts. Understanding `DOMContentLoaded` vs `load` and the role of `MutationObserver` demonstrates depth beyond basic DOM queries.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
