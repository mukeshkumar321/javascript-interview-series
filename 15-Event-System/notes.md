# Event System in JavaScript

## Table of Contents

1. [What are Events?](#1-what-are-events)
2. [addEventListener vs onclick](#2-addeventlistener-vs-onclick)
3. [Event Object](#3-event-object)
4. [target vs currentTarget](#4-target-vs-currenttarget)
5. [Event Bubbling](#5-event-bubbling)
6. [Event Capturing](#6-event-capturing)
7. [stopPropagation vs stopImmediatePropagation](#7-stoppropagation-vs-stopimmediatepropagation)
8. [preventDefault](#8-preventdefault)
9. [Event Delegation](#9-event-delegation)
10. [Custom Events](#10-custom-events)
11. [once Option in addEventListener](#11-once-option-in-addeventlistener)
12. [passive Event Listeners](#12-passive-event-listeners)
13. [Keyboard Events](#13-keyboard-events)
14. [Mouse Events](#14-mouse-events)
15. [mouseenter vs mouseover](#15-mouseenter-vs-mouseover)
16. [Focus Events](#16-focus-events)
17. [Input Events](#17-input-events)
18. [Event Loop and Events](#18-event-loop-and-events)
19. [Pointer Events](#19-pointer-events)
20. [Touch Events](#20-touch-events)
21. [Summary](#21-summary)

---

## 1. What are Events?

Events are signals fired by the browser (or your own code) when something happens — a user click, a key press, a network response, a DOM change. JavaScript responds to events by attaching **event listeners** — functions that are called when the event fires.

```js
// Most basic event listener
document.getElementById('btn').addEventListener('click', function (event) {
  console.log('clicked!');
  console.log(event.type);        // "click"
  console.log(event.target.id);  // "btn"
});
```

### Output

```js
"clicked!"
"click"
"btn"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 2. addEventListener vs onclick

| Feature | `addEventListener` | `onclick` (property) |
|---|---|---|
| Multiple listeners | Yes — each call adds a listener | No — only one; next assignment overwrites |
| Removal | `removeEventListener` | Set to `null` |
| Capture phase | Yes (`{ capture: true }`) | No |
| Options | `once`, `passive`, `signal` | None |
| Inline HTML | No | Via `onclick="..."` attribute |

```js
const btn = document.getElementById('btn');

// addEventListener — both run
btn.addEventListener('click', () => console.log('first'));
btn.addEventListener('click', () => console.log('second'));

// onclick — only last assignment survives
btn.onclick = () => console.log('alpha');
btn.onclick = () => console.log('beta');  // overwrites alpha

// After a single click:
// first
// second
// beta
```

### Output

```js
"first"
"second"
"beta"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 3. Event Object

Every listener receives an `Event` object (or a subclass like `MouseEvent`, `KeyboardEvent`). Key properties:

| Property / Method | Description |
|---|---|
| `event.type` | Event name (e.g. `"click"`) |
| `event.target` | Element that fired the event |
| `event.currentTarget` | Element the listener is attached to |
| `event.bubbles` | Whether the event bubbles |
| `event.cancelable` | Whether `preventDefault` has effect |
| `event.timeStamp` | Time since page load (ms) |
| `event.preventDefault()` | Prevents default browser action |
| `event.stopPropagation()` | Stops bubbling/capturing |
| `event.stopImmediatePropagation()` | Stops propagation + other listeners on same element |

```js
document.getElementById('link').addEventListener('click', (e) => {
  console.log(e.type);          // "click"
  console.log(e.bubbles);       // true
  console.log(e.cancelable);    // true
  e.preventDefault();            // stop navigation
  console.log('navigation prevented');
});
```

### Output

```js
"click"
true
true
"navigation prevented"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 4. target vs currentTarget

`event.target` is the element that originally triggered the event. `event.currentTarget` is the element whose listener is currently running. They differ when events bubble.

```js
// HTML: <div id="outer"><button id="inner">Click</button></div>

document.getElementById('outer').addEventListener('click', (e) => {
  console.log('target:', e.target.id);        // "inner" (where click happened)
  console.log('currentTarget:', e.currentTarget.id); // "outer" (where listener lives)
});
```

### Output

```js
"target: inner"
"currentTarget: outer"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 5. Event Bubbling

After an event fires on a target element, it **bubbles up** through each ancestor in the DOM tree, triggering matching listeners on the way. Most DOM events bubble (`click`, `keydown`, `input`); a few do not (`focus`, `blur`, `load`).

```js
// HTML: <div id="grandparent"><div id="parent"><button id="child">Click</button></div></div>

['grandparent', 'parent', 'child'].forEach(id => {
  document.getElementById(id).addEventListener('click', () => {
    console.log(id);
  });
});

// Click the button — logs in order from target to root
```

### Output

```js
"child"
"parent"
"grandparent"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 6. Event Capturing

By passing `{ capture: true }` (or `true` as the third argument), a listener runs during the **capture phase** — traveling top-down from `window` to the target — before bubbling. Capture listeners fire before bubble listeners.

```js
// HTML: <div id="outer"><button id="inner">Click</button></div>

document.getElementById('outer').addEventListener('click', () => {
  console.log('outer CAPTURE');
}, { capture: true });

document.getElementById('inner').addEventListener('click', () => {
  console.log('inner bubble');
});

document.getElementById('outer').addEventListener('click', () => {
  console.log('outer bubble');
});

// Click the inner button
```

### Output

```js
"outer CAPTURE"
"inner bubble"
"outer bubble"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 7. stopPropagation vs stopImmediatePropagation

`stopPropagation()` prevents the event from traveling further up (or down) the DOM — but other listeners on the **same element** still fire.

`stopImmediatePropagation()` also prevents other listeners on the **same element** from running.

```js
const btn = document.getElementById('btn');

btn.addEventListener('click', (e) => {
  console.log('listener 1');
  e.stopImmediatePropagation(); // blocks listener 2 AND parent listener
});

btn.addEventListener('click', () => {
  console.log('listener 2'); // never runs
});

document.getElementById('parent').addEventListener('click', () => {
  console.log('parent');     // never runs
});

// Click btn
```

### Output

```js
"listener 1"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 8. preventDefault

`preventDefault()` cancels the browser's default action for an event without stopping propagation. Common uses: prevent form submission, prevent link navigation, prevent default drag-and-drop behavior.

```js
// Prevent form submission
document.getElementById('myForm').addEventListener('submit', (e) => {
  e.preventDefault();
  console.log('form submitted via JS, page did not reload');
});

// Prevent link navigation
document.getElementById('myLink').addEventListener('click', (e) => {
  e.preventDefault();
  console.log('link click handled, navigation blocked');
});

// Check if it worked
document.getElementById('myLink').addEventListener('click', (e) => {
  console.log('defaultPrevented:', e.defaultPrevented); // true
});
```

### Output

```js
"link click handled, navigation blocked"
"defaultPrevented: true"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 9. Event Delegation

Instead of attaching listeners to every individual child element, attach **one listener to a common parent**. The event bubbles up from whichever child was clicked, and you identify the target via `event.target`. This is more memory-efficient and automatically handles dynamically added children.

```js
// HTML: <ul id="list"><li data-id="1">One</li><li data-id="2">Two</li></ul>

document.getElementById('list').addEventListener('click', (e) => {
  const li = e.target.closest('li');
  if (!li) return;
  console.log('clicked:', li.dataset.id);
});

// Dynamically added item still works — no new listener needed
const li3 = document.createElement('li');
li3.dataset.id = '3';
li3.textContent = 'Three';
document.getElementById('list').appendChild(li3);

// Click "Three" -> logs: "clicked: 3"
```

### Output

```js
"clicked: 3"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 10. Custom Events

`CustomEvent` lets you create and dispatch your own named events, optionally carrying data via the `detail` property. Use `dispatchEvent` to fire them.

```js
const el = document.getElementById('widget');

el.addEventListener('user:login', (e) => {
  console.log('user logged in:', e.detail.username);
  console.log('bubbles:', e.bubbles);
});

const event = new CustomEvent('user:login', {
  detail: { username: 'alice' },
  bubbles: true,
  cancelable: true
});

el.dispatchEvent(event);
```

### Output

```js
"user logged in: alice"
"bubbles: true"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 11. once Option in addEventListener

Passing `{ once: true }` automatically removes the listener after it fires the first time. No need to call `removeEventListener` manually.

```js
const btn = document.getElementById('btn');
let count = 0;

btn.addEventListener('click', () => {
  count++;
  console.log('clicked, count:', count);
}, { once: true });

btn.click(); // fires
btn.click(); // listener already removed — nothing happens
btn.click(); // nothing

console.log('final count:', count); // 1
```

### Output

```js
"clicked, count: 1"
"final count: 1"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 12. passive Event Listeners

Marking a listener `{ passive: true }` tells the browser that `preventDefault()` will never be called inside it. This allows the browser to start scrolling immediately without waiting for the listener to finish, improving scroll performance on touch devices.

```js
// Without passive — browser waits to see if preventDefault is called
window.addEventListener('touchstart', handler);

// With passive — browser can scroll immediately
window.addEventListener('touchstart', handler, { passive: true });

function handler(e) {
  // e.preventDefault(); // would throw a console warning if passive: true
  console.log('touch started');
}

// Note: Chrome sets touchstart/touchmove as passive by default since Chrome 51
```

### Output

```js
"touch started"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 13. Keyboard Events

Three keyboard events exist. `keypress` is deprecated — use `keydown` or `keyup`.

| Event | Fires | Repeats on hold? |
|---|---|---|
| `keydown` | When key is pressed down | Yes |
| `keyup` | When key is released | No |
| `keypress` | (deprecated) When printable key pressed | Yes |

```js
document.addEventListener('keydown', (e) => {
  console.log('key:', e.key);       // "A", "Enter", "ArrowLeft"
  console.log('code:', e.code);     // "KeyA", "Enter", "ArrowLeft"
  console.log('ctrlKey:', e.ctrlKey); // true if Ctrl held
});

// Pressing Ctrl + A:
// key: "a"   (what the key produces)
// code: "KeyA"  (physical key location)
// ctrlKey: true
```

### Output

```js
"key: a"
"code: KeyA"
"ctrlKey: true"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 14. Mouse Events

| Event | Description |
|---|---|
| `click` | Single left-click (mousedown + mouseup on same element) |
| `dblclick` | Two rapid clicks |
| `mousedown` | Mouse button pressed |
| `mouseup` | Mouse button released |
| `mousemove` | Mouse moves over element |
| `mouseenter` | Pointer enters element (does not bubble) |
| `mouseleave` | Pointer leaves element (does not bubble) |
| `mouseover` | Pointer enters element or its children (bubbles) |
| `mouseout` | Pointer leaves element or its children (bubbles) |
| `contextmenu` | Right-click / context menu key |

```js
const box = document.getElementById('box');

box.addEventListener('click', (e) => {
  console.log('clientX:', e.clientX); // viewport coordinates
  console.log('button:', e.button);   // 0=left, 1=middle, 2=right
});
```

### Output

```js
"clientX: 150"
"button: 0"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 15. mouseenter vs mouseover

The key difference is **bubbling**. `mouseover` bubbles — it fires when the pointer enters the element **or any of its descendants**. `mouseenter` does not bubble — it fires only when the pointer enters the element itself.

```js
// HTML: <div id="outer"><div id="inner">hover</div></div>

document.getElementById('outer').addEventListener('mouseover', () => {
  console.log('outer mouseover'); // fires when entering outer OR inner
});

document.getElementById('outer').addEventListener('mouseenter', () => {
  console.log('outer mouseenter'); // fires only when entering outer
});

// Moving mouse from outside directly onto the inner div:
// mouseover fires (bubbles up from inner to outer)
// mouseenter fires once (entering outer)

// Moving mouse from outer into inner:
// mouseover fires again (re-entering inner triggers bubble to outer)
// mouseenter does NOT fire again (pointer never left outer)
```

### Output

```js
"outer mouseenter"
"outer mouseover"
// then moving from outer into inner:
"outer mouseover"
// mouseenter does NOT fire again
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 16. Focus Events

| Event | Bubbles? | Description |
|---|---|---|
| `focus` | No | Element receives focus |
| `blur` | No | Element loses focus |
| `focusin` | Yes | Element receives focus (bubbles) |
| `focusout` | Yes | Element loses focus (bubbles) |

Use `focusin`/`focusout` when you need delegation (because `focus`/`blur` don't bubble).

```js
// HTML: <form id="form"><input id="name"><input id="email"></form>

// Using focusin for delegation — one listener on the form
document.getElementById('form').addEventListener('focusin', (e) => {
  console.log('focused:', e.target.id);
});

document.getElementById('form').addEventListener('focusout', (e) => {
  console.log('blurred:', e.target.id);
});

// Tab through inputs: focused: name -> blurred: name -> focused: email
```

### Output

```js
"focused: name"
"blurred: name"
"focused: email"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 17. Input Events

| Event | Fires on | Bubbles? |
|---|---|---|
| `input` | Every value change (typing, paste, delete) | Yes |
| `change` | When value is committed (blur for text, toggle for checkbox) | Yes |
| `submit` | Form submission attempt | Yes |

```js
const input = document.getElementById('search');

input.addEventListener('input', (e) => {
  console.log('input:', e.target.value); // fires on every keystroke
});

input.addEventListener('change', (e) => {
  console.log('change:', e.target.value); // fires when user leaves field
});

// Typing "ab" then clicking away:
// input: a
// input: ab
// change: ab
```

### Output

```js
"input: a"
"input: ab"
"change: ab"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 18. Event Loop and Events

User interaction events (click, keydown) and timer callbacks are placed in the **macrotask queue**. After each macrotask completes, the browser drains all microtasks (Promises, `queueMicrotask`), then may render, then picks the next macrotask.

```js
document.getElementById('btn').addEventListener('click', () => {
  console.log('1: click handler');

  Promise.resolve().then(() => {
    console.log('2: microtask');
  });

  setTimeout(() => {
    console.log('3: setTimeout');
  }, 0);

  console.log('4: end of click handler');
});

// Click the button — output order:
```

### Output

```js
"1: click handler"
"4: end of click handler"
"2: microtask"
"3: setTimeout"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 19. Pointer Events

Pointer Events unify mouse, touch, and stylus input into a single event model. They are the preferred API for cross-device interaction.

| Event | Description |
|---|---|
| `pointerdown` | Any pointer pressed |
| `pointerup` | Any pointer released |
| `pointermove` | Any pointer moved |
| `pointerenter` | Pointer enters element (no bubbling) |
| `pointerleave` | Pointer leaves element (no bubbling) |
| `pointercancel` | Pointer cancelled (e.g. scroll takes over) |

```js
const canvas = document.getElementById('canvas');

canvas.addEventListener('pointerdown', (e) => {
  console.log('pointerType:', e.pointerType); // "mouse", "touch", "pen"
  console.log('pointerId:', e.pointerId);     // unique ID per active pointer
  console.log('pressure:', e.pressure);       // 0 to 1
});
```

### Output

```js
"pointerType: mouse"
"pointerId: 1"
"pressure: 0.5"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 20. Touch Events

Touch Events are a mobile-specific API. Each event has a `touches` list (all current touches) and `targetTouches` (touches on this element) and `changedTouches` (touches that changed for this event).

```js
document.addEventListener('touchstart', (e) => {
  console.log('touches:', e.touches.length);
  const touch = e.touches[0];
  console.log('clientX:', touch.clientX);
  console.log('clientY:', touch.clientY);
  e.preventDefault(); // prevent scroll (only works if not passive)
});

document.addEventListener('touchmove', (e) => {
  console.log('moving');
});

document.addEventListener('touchend', (e) => {
  console.log('changedTouches:', e.changedTouches.length);
});
```

### Output

```js
"touches: 1"
"clientX: 200"
"clientY: 350"
"moving"
"changedTouches: 1"
```

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 21. Summary

| Topic | Key Point |
|---|---|
| `addEventListener` | Allows multiple listeners; supports `capture`, `once`, `passive` |
| `onclick` property | Only one listener; last assignment wins |
| `event.target` | Original source element of the event |
| `event.currentTarget` | Element whose listener is currently executing |
| Bubbling | Events travel up from target to root; most events bubble |
| Capturing | Events travel down from root to target; opt-in with `capture: true` |
| `stopPropagation` | Stops travel; other listeners on same element still run |
| `stopImmediatePropagation` | Stops travel AND other listeners on same element |
| `preventDefault` | Cancels browser default action; does not stop propagation |
| Event Delegation | One parent listener handles all current and future children |
| `CustomEvent` | Create own events with `detail` payload |
| `once: true` | Listener auto-removes after first fire |
| `passive: true` | Promise no `preventDefault` — allows immediate scroll |
| `keydown` / `keyup` | `key` = character/name, `code` = physical key |
| `mouseenter` | Does not bubble; only fires when entering the exact element |
| `mouseover` | Bubbles; fires when entering element or any descendant |
| `focus` / `blur` | Do not bubble; use `focusin`/`focusout` for delegation |
| `input` event | Fires on every character change |
| `change` event | Fires on committed value change |
| Events in event loop | Events are macrotasks; microtasks run between them |
| Pointer Events | Unified API for mouse, touch, and pen |

---

## Final Notes

The JavaScript event system rewards a mental model built around three things: the propagation phases (capture, target, bubble), the distinction between `target` and `currentTarget`, and the performance patterns (delegation, `passive`, `once`). In interviews, questions about bubbling order, `stopPropagation` vs `stopImmediatePropagation`, and `mouseenter` vs `mouseover` are especially common. Understanding `preventDefault` does not stop propagation, and that event listeners are macrotasks with microtasks draining between them, will set you apart from surface-level candidates.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
