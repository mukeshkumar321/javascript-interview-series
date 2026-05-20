# Event System — Tricky Output Questions

## Table of Contents
1. [Bubbling and Capturing Questions](#1-bubbling-and-capturing-questions)
2. [target vs currentTarget Questions](#2-target-vs-currenttarget-questions)
3. [stopPropagation Questions](#3-stoppropagation-questions)
4. [Event Delegation Questions](#4-event-delegation-questions)
5. [Custom Event Questions](#5-custom-event-questions)
6. [Event Listener Options Questions](#6-event-listener-options-questions)
7. [Advanced Event Questions](#7-advanced-event-questions)

---

## 1. Bubbling and Capturing Questions

---

### Q1. What will be the output when the inner button is clicked?

```js
// HTML: <div id="outer"><div id="middle"><button id="inner">Click</button></div></div>

document.getElementById('outer').addEventListener('click', () => console.log('outer bubble'));
document.getElementById('middle').addEventListener('click', () => console.log('middle bubble'));
document.getElementById('inner').addEventListener('click', () => console.log('inner bubble'));

document.getElementById('outer').addEventListener('click', () => console.log('outer capture'), true);
document.getElementById('middle').addEventListener('click', () => console.log('middle capture'), true);
document.getElementById('inner').addEventListener('click', () => console.log('inner capture'), true);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
outer capture
middle capture
inner capture
inner bubble
middle bubble
outer bubble
```

### Explanation
Event propagation has three phases: capture (top-down), target, bubble (bottom-up). Capture listeners (`true` / `{ capture: true }`) run first, traveling from `outer` down to `inner`. On the target element (`inner`), capture and bubble listeners both run in registration order. Then the event bubbles back up: `middle` bubble, then `outer` bubble.

</details>

---

### Q2. What will be the output?

```js
// HTML: <div id="parent"><button id="child">Click</button></div>

document.getElementById('parent').addEventListener('click', () => {
  console.log('parent capture');
}, { capture: true });

document.getElementById('parent').addEventListener('click', () => {
  console.log('parent bubble');
});

document.getElementById('child').addEventListener('click', () => {
  console.log('child listener 1');
});

document.getElementById('child').addEventListener('click', () => {
  console.log('child listener 2');
}, { capture: true });
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
parent capture
child listener 1
child listener 2
parent bubble
```

### Explanation
Capture phase runs first: `parent capture` fires. On the **target element** (`child`), the phase is "AT_TARGET" — capture and bubble listeners are both considered target-phase listeners and run **in registration order**, regardless of their `capture` flag. `child listener 1` was registered first, so it runs first. `child listener 2` second. Then the bubble phase: `parent bubble`.

</details>

---

### Q3. What will be the output?

```js
document.body.addEventListener('click', (e) => {
  console.log('body:', e.eventPhase);
}, true);

document.getElementById('btn').addEventListener('click', (e) => {
  console.log('btn:', e.eventPhase);
});

// eventPhase values: 1=CAPTURING, 2=AT_TARGET, 3=BUBBLING
// Assume btn is a direct child of body
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
body: 1
btn: 2
```

### Explanation
`event.eventPhase` reports the current phase of event propagation. When the `body` capture listener runs, the event is in the capture phase (`1`). When the `btn` listener runs, the event is at the target element (`2`). Because `body` has no bubble listener for this event, no `3` is logged. The numeric values are `Event.CAPTURING_PHASE = 1`, `Event.AT_TARGET = 2`, `Event.BUBBLING_PHASE = 3`.

</details>

---

## 2. target vs currentTarget Questions

---

### Q4. What will be the output when clicking the span?

```js
// HTML: <div id="box"><span id="label">Click me</span></div>

document.getElementById('box').addEventListener('click', (e) => {
  console.log('target:', e.target.tagName);
  console.log('currentTarget:', e.currentTarget.tagName);
  console.log('same?', e.target === e.currentTarget);
});
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
target: SPAN
currentTarget: DIV
same? false
```

### Explanation
The click originates on the `<span>` — that is `event.target`. The listener is attached to the `<div>`, which is `event.currentTarget`. They are different elements, so the comparison is `false`. `currentTarget` always refers to the element the listener is registered on, while `target` is the actual origin of the event.

</details>

---

### Q5. What will be the output?

```js
// HTML: <div id="box"><span id="label">Click me</span></div>

document.getElementById('box').addEventListener('click', (e) => {
  console.log('handler 1 currentTarget:', e.currentTarget.id);
});

document.getElementById('box').addEventListener('click', (e) => {
  console.log('handler 2 currentTarget:', e.currentTarget.id);
});

// After ALL handlers fire, check currentTarget asynchronously
document.getElementById('box').addEventListener('click', (e) => {
  const capturedEvent = e;
  setTimeout(() => {
    console.log('async currentTarget:', capturedEvent.currentTarget);
  }, 0);
});
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
handler 1 currentTarget: box
handler 2 currentTarget: box
async currentTarget: null
```

### Explanation
`event.currentTarget` is set to the element whose listener is running and is reset to `null` once the event dispatch is complete. Accessing it synchronously inside a handler works fine. But capturing the event object and reading `currentTarget` later in a `setTimeout` returns `null` because dispatch is long over. If you need the value later, save `e.currentTarget` in a variable, not the event object itself.

</details>

---

## 3. stopPropagation Questions

---

### Q6. What will be the output when the button is clicked?

```js
// HTML: <div id="outer"><button id="inner">Click</button></div>

document.getElementById('outer').addEventListener('click', () => {
  console.log('outer');
});

document.getElementById('inner').addEventListener('click', (e) => {
  e.stopPropagation();
  console.log('inner 1');
});

document.getElementById('inner').addEventListener('click', () => {
  console.log('inner 2');
});
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
inner 1
inner 2
```

### Explanation
`stopPropagation()` prevents the event from **traveling to other elements** but does NOT prevent other listeners on the **same element** from running. Both `inner 1` and `inner 2` listeners are on the same element — they both run. `outer` never fires because the event stopped propagating after the target phase.

</details>

---

### Q7. What will be the output?

```js
// HTML: <div id="outer"><button id="inner">Click</button></div>

document.getElementById('outer').addEventListener('click', () => {
  console.log('outer');
});

document.getElementById('inner').addEventListener('click', (e) => {
  e.stopImmediatePropagation();
  console.log('inner 1');
});

document.getElementById('inner').addEventListener('click', () => {
  console.log('inner 2'); // will this run?
});
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
inner 1
```

### Explanation
`stopImmediatePropagation()` does two things: it stops propagation to ancestor elements (like `stopPropagation`) AND it prevents any remaining listeners on the **same element** from firing. `inner 2` was registered after `inner 1`, so it is blocked. `outer` is also blocked. Only `inner 1` runs.

</details>

---

### Q8. What will be the output?

```js
// HTML: <div id="outer"><button id="inner">Click</button></div>

document.getElementById('outer').addEventListener('click', (e) => {
  e.stopPropagation();
  console.log('outer capture');
}, { capture: true });

document.getElementById('inner').addEventListener('click', () => {
  console.log('inner'); // will this run?
});

document.getElementById('outer').addEventListener('click', () => {
  console.log('outer bubble'); // will this run?
});
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
outer capture
```

### Explanation
`stopPropagation()` stops propagation in **both directions**. When called during the capture phase on `outer`, the event stops there and never reaches `inner` (no target phase), and never bubbles back up. The `outer bubble` listener and `inner` listener both never fire.

</details>

---

## 4. Event Delegation Questions

---

### Q9. What will be the output when clicking the third list item?

```js
// HTML:
// <ul id="list">
//   <li>One</li>
//   <li>Two</li>
//   <li><span>Three inner</span></li>
// </ul>

document.getElementById('list').addEventListener('click', (e) => {
  console.log('target tag:', e.target.tagName);
  console.log('closest li text:', e.target.closest('li').textContent.trim());
});
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
target tag: SPAN
closest li text: Three inner
```

### Explanation
When you click on the text inside the `<span>`, `event.target` is the `<span>`, not the `<li>`. This is a common delegation pitfall — clicking a nested child sets `target` to that child. `closest('li')` walks up the DOM tree from `event.target` until it finds the first matching `<li>` ancestor (or returns `null` if none found), which correctly identifies the list item regardless of how deeply nested the click target is.

</details>

---

### Q10. What will be the output?

```js
// HTML: <div id="container"><button class="btn">A</button></div>

const container = document.getElementById('container');

container.addEventListener('click', (e) => {
  if (e.target.classList.contains('btn')) {
    console.log('button clicked:', e.target.textContent);
  }
});

// Dynamically add another button
const newBtn = document.createElement('button');
newBtn.className = 'btn';
newBtn.textContent = 'B';
container.appendChild(newBtn);

// Now click button "B"
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
button clicked: B
```

### Explanation
Event delegation works for dynamically added elements because the listener is on the **parent container**, not the individual buttons. When `newBtn` (button B) is added and clicked, the event bubbles up to `container`. The delegation check `e.target.classList.contains('btn')` is true, so the handler fires. This demonstrates the key advantage of delegation over attaching listeners to individual elements.

</details>

---

## 5. Custom Event Questions

---

### Q11. What will be the output?

```js
const div = document.createElement('div');
document.body.appendChild(div);

div.addEventListener('myEvent', (e) => {
  console.log('received:', e.detail.msg);
  console.log('bubbles:', e.bubbles);
});

document.body.addEventListener('myEvent', (e) => {
  console.log('body caught it');
});

div.dispatchEvent(new CustomEvent('myEvent', {
  detail: { msg: 'hello' },
  bubbles: false
}));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
received: hello
bubbles: false
```

### Explanation
The custom event is dispatched on `div` with `bubbles: false`. The listener on `div` fires and logs the detail. Because `bubbles` is `false`, the event does not propagate to `document.body`, so the body listener never fires. If `bubbles: true` were set, `'body caught it'` would also be logged.

</details>

---

### Q12. What will be the output?

```js
const el = document.createElement('div');
document.body.appendChild(el);

el.addEventListener('test', (e) => {
  console.log('listener 1');
  console.log('cancelable:', e.cancelable);
  const prevented = e.preventDefault(); // try to cancel it
  console.log('defaultPrevented:', e.defaultPrevented);
});

const event = new CustomEvent('test', { cancelable: true, bubbles: false });
const notCancelled = el.dispatchEvent(event);

console.log('dispatchEvent returned:', notCancelled);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
listener 1
cancelable: true
defaultPrevented: true
dispatchEvent returned: false
```

### Explanation
`dispatchEvent` returns `false` if the event was `cancelable` and `preventDefault()` was called, otherwise `true`. Here the event is cancelable and `preventDefault()` is called inside the listener — so `defaultPrevented` becomes `true` and `dispatchEvent` returns `false`. This return value pattern mirrors how `element.click()` or form submission can be intercepted.

</details>

---

## 6. Event Listener Options Questions

---

### Q13. What will be the output after three clicks?

```js
const btn = document.getElementById('btn');
let count = 0;

btn.addEventListener('click', () => {
  count++;
  console.log('count:', count);
}, { once: true });

btn.click();
btn.click();
btn.click();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
count: 1
```

### Explanation
`{ once: true }` causes the listener to be automatically removed after it fires the first time. The first `btn.click()` fires the listener (count becomes 1) and removes it. The second and third `btn.click()` calls find no listener registered, so nothing happens. The final count is 1.

</details>

---

### Q14. What will be the output?

```js
const btn = document.getElementById('btn');

function handler() {
  console.log('fired');
}

btn.addEventListener('click', handler);
btn.addEventListener('click', handler);     // duplicate
btn.addEventListener('click', handler, true); // capture — different listener

btn.click();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
fired
fired
```

### Explanation
`addEventListener` ignores duplicate registrations of the exact same listener (same function reference, same event type, same capture flag). The second call with the same `handler` and default `capture: false` is silently ignored — only one bubble-phase listener is registered. However, the third call adds a **capture-phase** listener (different `capture` value = different listener slot). So there are two unique listeners: one capture and one bubble. On click: capture fires first ("fired"), then bubble fires ("fired").

</details>

---

### Q15. What will be the output?

```js
const btn = document.getElementById('btn');

btn.addEventListener('click', (e) => {
  console.log('passive listener');
  try {
    e.preventDefault();
  } catch (err) {
    console.log('error:', err.message);
  }
}, { passive: true });

btn.click();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
passive listener
error: Unable to preventDefault inside passive event listener invocation.
```

### Explanation
When a listener is registered as `passive: true`, it signals to the browser that `preventDefault()` will never be called. If you call it anyway, browsers throw a `TypeError` (or log a warning in some environments). The `passive` flag is primarily for scroll/touch performance — it lets the browser start scrolling immediately without waiting for the listener to complete. Calling `preventDefault` inside a passive listener is a contract violation.

</details>

---

## 7. Advanced Event Questions

---

### Q16. What will be the output order?

```js
document.getElementById('btn').addEventListener('click', () => {
  console.log('1: sync start');

  setTimeout(() => console.log('4: setTimeout'), 0);

  Promise.resolve().then(() => console.log('3: microtask'));

  console.log('2: sync end');
});

// Button is clicked programmatically
document.getElementById('btn').click();

console.log('5: after click()');
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
1: sync start
2: sync end
3: microtask
5: after click()
4: setTimeout
```

### Explanation
`element.click()` (programmatic click) is **synchronous** — the event listener runs inline, as if it were a direct function call. So `1` and `2` log immediately. After the listener returns, the microtask queue is drained: `3` logs. Then execution returns to the script after `.click()`, so `5` logs. Finally the `setTimeout` macrotask fires: `4` logs.

Note: this is different from a real user click, which is a macrotask. Programmatic `.click()` and `.dispatchEvent()` are synchronous.

</details>

---

### Q17. What will be the output?

```js
// HTML: <form id="form"><input id="input" type="text"></form>

document.getElementById('form').addEventListener('submit', (e) => {
  console.log('submit fired');
  e.preventDefault();
  console.log('default prevented:', e.defaultPrevented);
});

document.getElementById('form').addEventListener('submit', (e) => {
  console.log('second submit listener');
  console.log('already prevented:', e.defaultPrevented);
});
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
submit fired
default prevented: true
second submit listener
already prevented: true
```

### Explanation
`preventDefault()` sets `event.defaultPrevented` to `true` on the shared event object. Both listeners receive the **same event object**. The first listener calls `preventDefault()` and `defaultPrevented` becomes `true`. When the second listener runs, `e.defaultPrevented` is already `true`. Calling `preventDefault()` in the first listener does not stop the second listener from running — propagation is not affected.

</details>

---

### Q18. What will be the output?

```js
const outer = document.getElementById('outer');
const inner = document.getElementById('inner');

outer.addEventListener('click', (e) => {
  console.log('outer', e.target === inner ? 'target=inner' : 'target=outer');
  e.stopPropagation();
}, { capture: true });

outer.addEventListener('click', (e) => {
  console.log('outer bubble');
});

inner.addEventListener('click', (e) => {
  console.log('inner');
});

inner.click();
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output
```txt
outer target=inner
```

### Explanation
`inner.click()` triggers a click on `inner` which bubbles. During the capture phase, `outer`'s capture listener runs and calls `stopPropagation()`. The event never reaches `inner` (no target phase), and because propagation is stopped, the `outer bubble` listener also never fires. The target of the event is `inner` (where `click()` was called) even though the listener fires on `outer` during the capture phase.

</details>

---

## Final Tips

- **Capture vs bubble order**: capture phase runs top-down first; at the target element, listeners run in registration order regardless of phase; bubble runs bottom-up last.
- **`stopPropagation` vs `stopImmediatePropagation`**: `stopPropagation` stops travel to other elements; `stopImmediatePropagation` also stops other listeners on the same element.
- **`target` vs `currentTarget`**: `target` never changes during propagation; `currentTarget` updates to whichever element's listener is running and becomes `null` after dispatch.
- **`currentTarget` in async code**: save `e.currentTarget` to a variable immediately — it becomes `null` after the synchronous handler returns.
- **Delegation pitfall**: `event.target` may be a deeply nested child; always use `closest()` to find the intended ancestor.
- **Duplicate listeners**: registering the same function with the same capture flag is silently ignored — only one is registered.
- **`once: true`**: the listener is automatically removed after the first invocation.
- **Programmatic `.click()`**: synchronous — the listener runs inline and resolves before the next line of calling code; real user clicks are macrotasks.
- **`preventDefault` does not stop propagation** — these are independent operations on the same event.
- **Custom events**: `bubbles` defaults to `false`; `dispatchEvent` returns `false` if the event was cancelable and `preventDefault` was called.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
