# DOM Manipulation — Tricky Output Questions

## Table of Contents
1. [innerHTML vs textContent Questions](#1-innerhtml-vs-textcontent-questions)
2. [DOM Traversal Questions](#2-dom-traversal-questions)
3. [Event Timing Questions (DOMContentLoaded vs load)](#3-event-timing-questions-domcontentloaded-vs-load)
4. [MutationObserver Questions](#4-mutationobserver-questions)
5. [Reflow/Repaint Questions](#5-reflowrepaint-questions)
6. [Advanced DOM Questions](#6-advanced-dom-questions)

---

## 1. innerHTML vs textContent Questions

---

### Q1. What will be the output?

```js
const div = document.createElement('div');
div.innerHTML = '<b>Hello</b> <i>World</i>';

console.log(div.textContent);
console.log(div.childNodes.length);
console.log(div.children.length);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Hello World
3
2
```

### Explanation
`textContent` strips all HTML tags and concatenates text content from all descendant text nodes. The string `'<b>Hello</b> <i>World</i>'` parses into three child nodes: a `<b>` element, a text node `" "`, and an `<i>` element. `childNodes.length` is `3` (all node types). `children.length` is `2` (element nodes only — the text node `" "` is excluded).

</details>

---

### Q2. What will be the output?

```js
const div = document.createElement('div');
div.textContent = '<b>Bold</b>';

console.log(div.innerHTML);
console.log(div.childNodes.length);
console.log(div.firstChild.nodeType);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
&lt;b&gt;Bold&lt;/b&gt;
1
3
```

### Explanation
Setting `textContent` treats the value as a plain string — it does not parse HTML. The angle brackets are HTML-escaped when read back via `innerHTML`. The div has exactly one child: a text node. Text nodes have `nodeType === 3` (`Node.TEXT_NODE`).

</details>

---

### Q3. What will be the output?

```js
const div = document.createElement('div');
const span = document.createElement('span');
span.style.display = 'none';
span.textContent = 'hidden';
div.appendChild(document.createTextNode('visible'));
div.appendChild(span);

document.body.appendChild(div);

console.log(div.textContent);
console.log(div.innerText);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
visiblehidden
visible
```

### Explanation
`textContent` returns the concatenated text of all descendant text nodes regardless of CSS visibility or display. `innerText` is aware of rendering: it only returns text that is actually visible to the user. Because `span` has `display: none`, its content is excluded from `innerText`. Note: `innerText` requires the element to be in the live document and triggers a reflow to compute styles.

</details>

---

### Q4. What will be the output?

```js
const div = document.createElement('div');
div.innerHTML = 'Before';
div.innerHTML += ' After';

console.log(div.childNodes.length);
console.log(div.textContent);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
1
Before After
```

### Explanation
`div.innerHTML += ' After'` is equivalent to `div.innerHTML = div.innerHTML + ' After'`. This **destroys and re-creates** all child nodes. First, `div.innerHTML` is read as `'Before'`, then `' After'` is concatenated, and the new string `'Before After'` is parsed back into the element, creating a single new text node. Any event listeners attached to previous child nodes are lost.

</details>

---

## 2. DOM Traversal Questions

---

### Q5. What will be the output?

```js
// HTML: <div id="parent">  <span>A</span>  <span>B</span>  </div>
// (with whitespace between elements)

const parent = document.getElementById('parent');

console.log(parent.childNodes.length);
console.log(parent.children.length);
console.log(parent.firstChild === parent.firstElementChild);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
5
2
false
```

### Explanation
The HTML has whitespace text nodes around and between the `<span>` elements. `childNodes` includes all node types: text node (space before span A), span A, text node (space between spans), span B, text node (space after span B) — total 5. `children` only counts element nodes — 2 spans. `firstChild` is the first whitespace text node, while `firstElementChild` is the first `<span>` — they are different nodes, so the comparison is `false`.

</details>

---

### Q6. What will be the output?

```js
const ul = document.createElement('ul');
ul.innerHTML = '<li>One</li><li>Two</li><li>Three</li>';

const li = ul.querySelector('li:nth-child(2)');

console.log(li.previousElementSibling.textContent);
console.log(li.nextElementSibling.textContent);
console.log(li.parentElement.tagName);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
One
Three
UL
```

### Explanation
`li:nth-child(2)` selects the second `<li>` element, which contains "Two". `previousElementSibling` points to the element immediately before it — the first `<li>` with text "One". `nextElementSibling` points to the third `<li>` with text "Three". `parentElement` is the `<ul>`.

</details>

---

### Q7. What will be the output?

```js
const div = document.createElement('div');
div.innerHTML = '<p><span>text</span></p>';

const span = div.querySelector('span');

console.log(span.parentNode === span.parentElement);
console.log(span.parentNode.tagName);
console.log(span.parentNode.parentNode.tagName);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
true
P
DIV
```

### Explanation
For regular element nodes, `parentNode` and `parentElement` return the same value — the nearest ancestor element. They differ only at the document root where `document.documentElement.parentNode` is the `document` object but `parentElement` would be `null`. The `<span>` is nested inside `<p>` which is inside `<div>`.

</details>

---

## 3. Event Timing Questions (DOMContentLoaded vs load)

---

### Q8. What will be the output order?

```js
window.addEventListener('load', () => {
  console.log('load');
});

document.addEventListener('DOMContentLoaded', () => {
  console.log('DOMContentLoaded');
});

console.log('inline script');
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
inline script
DOMContentLoaded
load
```

### Explanation
Synchronous inline script runs first. `DOMContentLoaded` fires once the browser has parsed the full HTML and built the DOM tree (but before images and stylesheets finish loading). `load` fires last, after all page resources (images, CSS, fonts, iframes) have fully loaded. This ordering is guaranteed by the browser.

</details>

---

### Q9. What will be the output?

```js
document.addEventListener('DOMContentLoaded', () => {
  console.log('1: DOMContentLoaded');
  Promise.resolve().then(() => console.log('2: microtask inside DCL'));
  setTimeout(() => console.log('3: timeout inside DCL'), 0);
});

document.addEventListener('DOMContentLoaded', () => {
  console.log('4: second DOMContentLoaded listener');
});
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
1: DOMContentLoaded
2: microtask inside DCL
4: second DOMContentLoaded listener
3: timeout inside DCL
```

### Explanation
`DOMContentLoaded` fires as a macrotask. The first listener runs: it logs `1`, schedules a microtask (Promise), and a macrotask (setTimeout). After the first listener completes, the microtask queue is drained — `2` is logged. Then the next DOMContentLoaded listener runs (still part of the same event dispatch) — `4` is logged. Finally, the setTimeout macrotask fires — `3` is logged.

</details>

---

## 4. MutationObserver Questions

---

### Q10. What will be the output?

```js
const div = document.createElement('div');
document.body.appendChild(div);

const observer = new MutationObserver((mutations) => {
  console.log('observer fired');
  console.log(mutations.length);
});

observer.observe(div, { childList: true });

div.appendChild(document.createElement('span'));
div.appendChild(document.createElement('p'));

console.log('after mutations');
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
after mutations
observer fired
2
```

### Explanation
`MutationObserver` callbacks are queued as **microtasks** and run after the current synchronous code completes. Both DOM mutations (appending `span` and `p`) happen synchronously. The synchronous code logs `'after mutations'` first. After the call stack empties, the microtask queue is drained, and the observer callback fires once with a `mutations` array containing **two** `MutationRecord` entries — one per mutation.

</details>

---

### Q11. What will be the output?

```js
const div = document.createElement('div');
div.setAttribute('data-count', '0');
document.body.appendChild(div);

const observer = new MutationObserver((mutations) => {
  for (const m of mutations) {
    console.log(m.type);
    console.log(m.attributeName);
    console.log(m.oldValue);
  }
});

observer.observe(div, {
  attributes: true,
  attributeOldValue: true
});

div.setAttribute('data-count', '1');
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
attributes
data-count
0
```

### Explanation
With `attributes: true`, the observer watches for attribute changes. With `attributeOldValue: true`, each `MutationRecord` includes the value the attribute had **before** the change. The mutation type is `"attributes"`, the attribute name is `"data-count"`, and `oldValue` is `"0"` (a string, since all attribute values are strings).

</details>

---

## 5. Reflow/Repaint Questions

---

### Q12. How many reflows does this code trigger?

```js
const box = document.getElementById('box');

// Sequence A
box.style.width = '100px';
const w = box.offsetWidth;
box.style.height = '100px';
const h = box.offsetHeight;

// Sequence B
const w2 = box.offsetWidth;
const h2 = box.offsetHeight;
box.style.width = (w2 + 10) + 'px';
box.style.height = (h2 + 10) + 'px';

console.log('Sequence A reflows: 2, Sequence B reflows: 1');
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
Sequence A reflows: 2, Sequence B reflows: 1
```

### Explanation
In Sequence A, writing `width` then immediately reading `offsetWidth` forces the browser to flush its pending style changes and perform a layout (reflow) to return an accurate value. The same happens again with `height`. That is 2 forced synchronous reflows.

In Sequence B, both reads happen before any writes. The browser can return the values from its existing layout. Then both writes happen together. The browser can batch these into a single reflow at the end of the frame. That is 1 reflow.

</details>

---

### Q13. What will be the output and what performance issue exists?

```js
const items = document.querySelectorAll('.item');

items.forEach(item => {
  const h = item.offsetHeight;    // read
  item.style.height = h + 'px';   // write
});

console.log('done');
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
done
```

### Explanation
The output itself is just `'done'`, but the code has a **layout thrashing** problem. Inside the `forEach`, each iteration reads `offsetHeight` (which forces a reflow if there are pending writes) and then writes `style.height` (which invalidates the layout). On the next iteration, reading `offsetHeight` again forces another reflow. If there are N items, this causes N forced synchronous reflows instead of 1.

The fix is to batch all reads first, then all writes:
```js
const heights = Array.from(items).map(item => item.offsetHeight);
items.forEach((item, i) => { item.style.height = heights[i] + 'px'; });
```

</details>

---

## 6. Advanced DOM Questions

---

### Q14. What will be the output?

```js
const ul = document.createElement('ul');
const items = ['a', 'b', 'c'];

items.forEach(text => {
  const li = document.createElement('li');
  li.textContent = text;
  ul.appendChild(li);
});

const liveList  = ul.getElementsByTagName('li');
const staticList = ul.querySelectorAll('li');

console.log(liveList.length);   // before
console.log(staticList.length); // before

const li = document.createElement('li');
li.textContent = 'd';
ul.appendChild(li);

console.log(liveList.length);   // after
console.log(staticList.length); // after
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
3
3
4
3
```

### Explanation
Both collections start with 3 items. After appending a fourth `<li>`, `getElementsByTagName` returns a **live** `HTMLCollection` that automatically reflects the current state of the DOM — its length becomes 4. `querySelectorAll` returns a **static** `NodeList` captured at query time — it does not update, so it remains 3.

</details>

---

### Q15. What will be the output?

```js
const original = document.createElement('div');
original.id = 'one';
original.innerHTML = '<span>child</span>';

let clickCount = 0;
original.addEventListener('click', () => clickCount++);

const deepClone   = original.cloneNode(true);
const shallowClone = original.cloneNode(false);

console.log(deepClone.id);
console.log(deepClone.children.length);
console.log(shallowClone.children.length);

// Simulate click on deep clone
deepClone.click();
console.log(clickCount);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
one
1
0
0
```

### Explanation
`cloneNode(true)` copies the element, its attributes (`id`), and all descendants. The `<span>` child is cloned, so `deepClone.children.length` is `1`. `cloneNode(false)` is shallow — no children, so `shallowClone.children.length` is `0`. Critically, `cloneNode` does **not** copy event listeners registered via `addEventListener`. Clicking `deepClone` does not increment `clickCount` on the original listener. `clickCount` remains `0`.

</details>

---

### Q16. What will be the output?

```js
const fragment = document.createDocumentFragment();
const div = document.createElement('div');
div.textContent = 'hello';
fragment.appendChild(div);

console.log(fragment.childNodes.length); // before
console.log(div.parentNode === fragment); // parent check

document.body.appendChild(fragment);

console.log(fragment.childNodes.length); // after
console.log(div.parentNode === document.body); // parent after insertion
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
1
true
0
true
```

### Explanation
Before insertion, the fragment holds one child (`div`) and is indeed its `parentNode`. When a `DocumentFragment` is appended to a real DOM node, the fragment's children are **moved** (not copied) into the target — the fragment itself is not inserted. After `appendChild(fragment)`, the fragment is empty (`childNodes.length === 0`) and `div.parentNode` is now `document.body`.

</details>

---

### Q17. What will be the output?

```js
const div = document.createElement('div');
div.innerHTML = `
  <p id="p1">First</p>
  <p id="p2">Second</p>
`;

const p1 = div.querySelector('#p1');
const p2 = div.querySelector('#p2');

div.insertBefore(p2, p1);

console.log(div.children[0].id);
console.log(div.children[1].id);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
p2
p1
```

### Explanation
`insertBefore(newNode, referenceNode)` inserts `newNode` directly before `referenceNode`. Since `p2` is already in `div`, it is **moved** (not cloned) to the position before `p1`. After the call, `p2` is the first child and `p1` is the second. Inserting an existing node always moves it — no duplicate is created.

</details>

---

## Final Tips

- `innerHTML` with user-controlled data is the number one DOM XSS attack vector; always use `textContent` or a sanitizer.
- Remember the live vs static distinction: `getElementsByClassName`/`getElementsByTagName` return live `HTMLCollection`; `querySelectorAll` returns a static `NodeList`.
- `MutationObserver` callbacks fire as microtasks — after synchronous code but before the next macrotask.
- `childNodes` includes text nodes (whitespace matters!); `children` is elements only.
- `cloneNode` never copies event listeners — only the node structure and attributes.
- A `DocumentFragment` empties itself when appended: its children are moved, not copied.
- Interleaved DOM reads and writes (layout thrashing) cause forced synchronous reflows; always batch reads before writes.
- `DOMContentLoaded` always fires before `load`. Inline scripts run before both.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
