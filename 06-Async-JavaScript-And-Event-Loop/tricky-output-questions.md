# Async JavaScript and Event Loop — Tricky Output Questions

## Table of Contents

1. [Event Loop Order Questions](#1-event-loop-order-questions)
2. [setTimeout Questions](#2-settimeout-questions)
3. [Promise Questions](#3-promise-questions)
4. [Microtask vs Macrotask Questions](#4-microtask-vs-macrotask-questions)
5. [async/await Questions](#5-asyncawait-questions)
6. [Mixed Async Questions](#6-mixed-async-questions)
7. [Advanced Event Loop Questions](#7-advanced-event-loop-questions)

---

## 1. Event Loop Order Questions

---

### Q1. What will be the output?

```js
console.log("A");
setTimeout(() => console.log("B"), 0);
console.log("C");
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

`console.log("A")` and `console.log("C")` are synchronous and run immediately in order. `setTimeout` with `0` ms schedules its callback as a **macrotask**. Macrotasks only run after all synchronous code and all microtasks have finished. So `B` logs last.

</details>

---

### Q2. What will be the output?

```js
console.log("start");

setTimeout(() => console.log("timeout 1"), 0);
setTimeout(() => console.log("timeout 2"), 0);

Promise.resolve().then(() => console.log("promise 1"));
Promise.resolve().then(() => console.log("promise 2"));

console.log("end");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
start
end
promise 1
promise 2
timeout 1
timeout 2
```

### Explanation

1. Synchronous: `start`, `end`.
2. Microtask queue drains: `promise 1`, `promise 2` (Promise callbacks are microtasks — higher priority than macrotasks).
3. Macrotask queue: `timeout 1`, `timeout 2` — picked up one at a time.

</details>

---

### Q3. What will be the output?

```js
console.log(1);

setTimeout(() => console.log(2), 0);

Promise.resolve().then(() => {
  console.log(3);
  setTimeout(() => console.log(4), 0);
});

console.log(5);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
1
5
3
2
4
```

### Explanation

1. Sync: `1`, `5`.
2. Microtask drains: the `.then` callback runs → logs `3`, then registers a new `setTimeout` for `4`.
3. Macrotask queue (in order they were registered): `2` first (registered before `4`), then `4`.

Key point: `setTimeout(4)` is registered inside a microtask, so it lands behind `setTimeout(2)` in the macrotask queue.

</details>

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 2. setTimeout Questions

---

### Q1. What will be the output?

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
3
3
3
```

### Explanation

`var` is function-scoped, so all three callbacks close over the **same** `i` variable. By the time the event loop processes the macrotasks (after the `for` loop finishes), `i` has already reached `3`. All three callbacks read `i` as `3`.

To fix this, use `let` (which creates a new binding per iteration) or an IIFE.

</details>

---

### Q2. What will be the output?

```js
setTimeout(() => console.log("A"), 100);
setTimeout(() => console.log("B"), 0);
setTimeout(() => console.log("C"), 50);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
B
C
A
```

### Explanation

Timers fire after their minimum delay has elapsed. `B` (0 ms) fires first, then `C` (50 ms), then `A` (100 ms). The macrotask queue processes them in the order their timers expire.

</details>

---

### Q3. What will be the output?

```js
setTimeout(() => {
  console.log("outer");
}, 0);

setTimeout(() => {
  console.log("inner 1");
  setTimeout(() => console.log("inner 2"), 0);
}, 0);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
outer
inner 1
inner 2
```

### Explanation

Both outer `setTimeout` callbacks are in the macrotask queue. The event loop processes them one at a time. When `inner 1` runs, it schedules another `setTimeout` — but that new callback goes to the back of the macrotask queue, so `outer` always runs before `inner 1`, and `inner 2` runs last.

</details>

---

### Q4. What will be the output?

```js
console.log("before");

setTimeout(function () {
  console.log("timeout");
}, 0);

console.log("after");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
before
after
timeout
```

### Explanation

`setTimeout(fn, 0)` does not execute `fn` immediately. It queues it as a macrotask. Synchronous code (`before`, `after`) runs first. Only when the call stack is empty does the event loop pull the macrotask and execute it.

</details>

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 3. Promise Questions

---

### Q1. What will be the output?

```js
const p = new Promise((resolve, reject) => {
  console.log("executor");
  resolve("done");
});

p.then(v => console.log(v));
console.log("after promise");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
executor
after promise
done
```

### Explanation

The Promise **executor function** runs **synchronously** — it is called immediately when the Promise is constructed. `resolve("done")` marks the Promise as fulfilled but schedules the `.then` callback as a microtask, not immediately. So `after promise` (synchronous) logs before `done` (microtask).

</details>

---

### Q2. What will be the output?

```js
Promise.resolve(1)
  .then(v => {
    console.log(v);
    return 2;
  })
  .then(v => {
    console.log(v);
    return 3;
  })
  .then(v => console.log(v));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
1
2
3
```

### Explanation

Each `.then` receives the return value of the previous `.then` callback. `Promise.resolve(1)` starts the chain with `1`, which flows through: `1` → returns `2` → returns `3`. Each step is a microtask that runs in sequence.

</details>

---

### Q3. What will be the output?

```js
Promise.resolve()
  .then(() => {
    console.log(1);
    return Promise.resolve(2);
  })
  .then(v => console.log(v));

Promise.resolve()
  .then(() => console.log(3))
  .then(() => console.log(4));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
1
3
4
2
```

### Explanation

This is one of the trickiest Promise interview questions. When a `.then` callback **returns a native Promise** (like `Promise.resolve(2)`), the spec requires an extra microtask "tick" to unwrap it (a `PromiseResolveThenableJob`).

Microtask queue trace:
- Initial queue: `[chain1-then1, chain2-then1]`
- Process `chain1-then1` → logs `1`, returns `Promise.resolve(2)` → queues 1 internal unwrapping job. Queue: `[chain2-then1, unwrap-job]`
- Process `chain2-then1` → logs `3` → queues `chain2-then2`. Queue: `[unwrap-job, chain2-then2]`
- Process `unwrap-job` → resolves outer promise with `2` → queues `chain1-then2`. Queue: `[chain2-then2, chain1-then2]`
- Process `chain2-then2` → logs `4`
- Process `chain1-then2` → logs `2`

</details>

---

### Q4. What will be the output?

```js
new Promise(resolve => {
  resolve(Promise.resolve("inner"));
}).then(v => console.log(v));

Promise.resolve().then(() => console.log("outer"));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
outer
inner
```

### Explanation

When you pass a thenable (another Promise) to `resolve()`, the engine schedules a `PromiseResolveThenableJob` microtask to call `.then()` on it. This adds an extra tick before the outer `.then(v => console.log(v))` handler can run. Meanwhile `Promise.resolve().then(() => console.log("outer"))` is a straightforward microtask with no extra delay, so it executes first.

</details>

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 4. Microtask vs Macrotask Questions

---

### Q1. What will be the output?

```js
setTimeout(() => console.log("A"), 0);
Promise.resolve().then(() => console.log("B"));
console.log("C");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
C
B
A
```

### Explanation

`C` is synchronous. `B` is a microtask (Promise.then) and runs before any macrotask. `A` is a macrotask (setTimeout) and runs last.

</details>

---

### Q2. What will be the output?

```js
setTimeout(() => {
  console.log("timeout");
  Promise.resolve().then(() => console.log("promise inside timeout"));
}, 0);

Promise.resolve().then(() => console.log("promise"));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
promise
timeout
promise inside timeout
```

### Explanation

1. `promise` is a microtask — drains before any macrotask.
2. `timeout` is a macrotask — runs next.
3. Inside the `timeout` callback, a new microtask is queued. After the macrotask completes, the microtask queue is drained again before the next macrotask: so `promise inside timeout` runs immediately after `timeout`.

</details>

---

### Q3. What will be the output?

```js
console.log("start");

setTimeout(() => console.log("macro 1"), 0);

Promise.resolve()
  .then(() => {
    console.log("micro 1");
    setTimeout(() => console.log("macro 2"), 0);
    return Promise.resolve();
  })
  .then(() => console.log("micro 2"));

setTimeout(() => console.log("macro 3"), 0);

console.log("end");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
start
end
micro 1
micro 2
macro 1
macro 3
macro 2
```

### Explanation

1. Sync: `start`, `end`. Macrotask queue at this point: `[macro 1, macro 3]`.
2. Microtask queue drains:
   - First `.then` fires → logs `micro 1`, registers `macro 2` (goes to back of macrotask queue: `[macro 1, macro 3, macro 2]`), returns `Promise.resolve()` (extra tick, but chained `.then` eventually queues).
   - Second `.then` fires → logs `micro 2`.
3. Macrotask queue processes in order: `macro 1`, `macro 3`, `macro 2`.

</details>

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 5. async/await Questions

---

### Q1. What will be the output?

```js
async function foo() {
  console.log(1);
  await Promise.resolve();
  console.log(2);
}

console.log(3);
foo();
console.log(4);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
3
1
4
2
```

### Explanation

- `3` logs before `foo()` is called.
- `foo()` runs synchronously up to `await`: logs `1`, then suspends.
- Control returns to the caller, which logs `4`.
- The microtask queue drains: `foo()` resumes and logs `2`.

</details>

---

### Q2. What will be the output?

```js
async function first() {
  console.log("first: start");
  await second();
  console.log("first: end");
}

async function second() {
  console.log("second: start");
  await Promise.resolve();
  console.log("second: end");
}

console.log("global: start");
first();
console.log("global: end");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
global: start
first: start
second: start
global: end
second: end
first: end
```

### Explanation

1. Sync: `global: start`.
2. `first()` runs sync until its `await second()` — but `second()` is called first, running sync until its own `await Promise.resolve()`: logs `first: start`, `second: start`, then suspends `second`.
3. `await second()` in `first` suspends `first` (waiting for `second`'s promise to resolve).
4. Control returns to global: `global: end`.
5. Microtask: `second` resumes → logs `second: end`, resolves its promise.
6. Microtask: `first` resumes (second's promise settled) → logs `first: end`.

</details>

---

### Q3. What will be the output?

```js
async function p() {
  console.log("A");
  await null;
  console.log("B");
  await null;
  console.log("C");
}

console.log("D");
p();
console.log("E");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
D
A
E
B
C
```

### Explanation

- Sync: `D`, then `p()` is called.
- `p()` runs: logs `A`, hits `await null` (suspends, queues microtask MT1), control returns to global.
- Sync continues: `E`.
- MT1 fires: `p()` resumes → logs `B`, hits second `await null` (suspends, queues MT2).
- MT2 fires: `p()` resumes → logs `C`.

Each `await` suspends the function for one microtask tick.

</details>

---

### Q4. What will be the output?

```js
async function fetchData() {
  return "data";
}

async function main() {
  const result = await fetchData();
  console.log(result);
}

main();
console.log("sync");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
sync
data
```

### Explanation

`main()` runs synchronously until `await fetchData()`. Even though `fetchData()` returns `"data"` immediately (wrapped as `Promise.resolve("data")`), `await` still suspends `main` and queues its continuation as a microtask. So `sync` (synchronous code after `main()`) logs before `data`.

</details>

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 6. Mixed Async Questions

---

### Q1. What will be the output?

```js
console.log("start");

setTimeout(() => console.log("setTimeout"), 0);

Promise.resolve()
  .then(() => console.log("promise 1"))
  .then(() => console.log("promise 2"));

console.log("end");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
start
end
promise 1
promise 2
setTimeout
```

### Explanation

Classic event loop ordering: synchronous first, then microtasks (all of them, including chained `.then`), then macrotasks. `promise 2` is a chained microtask — it is queued when `promise 1`'s handler returns, still before the macrotask runs.

</details>

---

### Q2. What will be the output?

```js
async function run() {
  console.log("run: start");
  setTimeout(() => console.log("run: setTimeout"), 0);
  await Promise.resolve();
  console.log("run: after await");
}

console.log("main: start");
run();
console.log("main: end");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
main: start
run: start
main: end
run: after await
run: setTimeout
```

### Explanation

1. Sync: `main: start`, then `run()` starts.
2. Inside `run()`: `run: start`, setTimeout is registered (macrotask), `await` suspends `run`.
3. Back in main: `main: end`.
4. Microtask: `run` resumes → `run: after await`.
5. Macrotask: `run: setTimeout`.

`await` creates a microtask boundary — the code after `await` always runs before any macrotask, even one registered before the `await`.

</details>

---

### Q3. What will be the output?

```js
console.log(1);

async function asyncFn() {
  console.log(2);
  await Promise.resolve();
  console.log(3);
}

setTimeout(() => console.log(4), 0);
asyncFn();
Promise.resolve().then(() => console.log(5));
console.log(6);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
1
2
6
3
5
4
```

### Explanation

1. Sync: `1`, then `asyncFn()` runs synchronously to its first `await`: logs `2`, suspends → queues MT1 (continuation of asyncFn). `setTimeout(4)` queues macrotask. `Promise.resolve().then(5)` queues MT2. Logs `6`.
2. Microtask queue: MT1 (`3`), then MT2 (`5`). (MT1 was queued first, then MT2.)
3. Macrotask: `4`.

</details>

---

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>

---

## 7. Advanced Event Loop Questions

---

### Q1. What will be the output?

```js
Promise.resolve()
  .then(() => console.log(1))
  .then(() => console.log(2))
  .then(() => console.log(3));

Promise.resolve()
  .then(() => console.log(4))
  .then(() => console.log(5))
  .then(() => console.log(6));
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
1
4
2
5
3
6
```

### Explanation

Two independent chains share the same microtask queue. After sync code, both first `.then` callbacks are already queued:

- Queue: `[chain1-then1, chain2-then1]`
- Process `chain1-then1` → logs `1` → queues `chain1-then2`. Queue: `[chain2-then1, chain1-then2]`
- Process `chain2-then1` → logs `4` → queues `chain2-then2`. Queue: `[chain1-then2, chain2-then2]`
- Process `chain1-then2` → logs `2` → queues `chain1-then3`. Queue: `[chain2-then2, chain1-then3]`
- Process `chain2-then2` → logs `5` → queues `chain2-then3`. Queue: `[chain1-then3, chain2-then3]`
- Process `chain1-then3` → logs `3`.
- Process `chain2-then3` → logs `6`.

The chains interleave because each resolved `.then` queues the **next** handler at the **back** of the queue.

</details>

---

### Q2. What will be the output?

```js
function recursiveMicrotask(n) {
  if (n === 0) return;
  Promise.resolve().then(() => {
    console.log(n);
    recursiveMicrotask(n - 1);
  });
}

setTimeout(() => console.log("setTimeout"), 0);
recursiveMicrotask(3);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
3
2
1
setTimeout
```

### Explanation

`recursiveMicrotask(3)` queues a microtask. When it runs, it logs `3` and queues another microtask for `n=2`. This cascades: each microtask queues the next. Because the microtask queue is **fully drained** before any macrotask runs, all three microtasks (`3`, `2`, `1`) execute before `setTimeout` gets a turn.

This demonstrates **microtask queue starvation**: if microtasks keep adding more microtasks, macrotasks (like rendering or timers) can be indefinitely delayed.

</details>

---

### Q3. What will be the output?

```js
async function a() {
  console.log(1);
  await b();
  console.log(2);
}

async function b() {
  console.log(3);
  await c();
  console.log(4);
}

async function c() {
  console.log(5);
}

a();
console.log(6);
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
1
3
5
6
4
2
```

### Explanation

Synchronous phase:
- `a()` runs → logs `1`, calls `b()`.
- `b()` runs → logs `3`, calls `c()`.
- `c()` runs → logs `5`, returns (resolves as `Promise.resolve(undefined)`).
- `await c()` in `b` suspends `b`, queues MT1 (b's continuation).
- `await b()` in `a` suspends `a` (waiting for `b`'s promise).
- Back in global: logs `6`.

Microtask phase:
- MT1: `b` resumes → logs `4`, `b` resolves its promise → queues MT2 (a's continuation).
- MT2: `a` resumes → logs `2`.

</details>

---

### Q4. What will be the output?

```js
async function outer() {
  console.log("outer start");

  const result = await new Promise(resolve => {
    console.log("promise executor");
    resolve("resolved value");
  });

  console.log("outer end:", result);
}

console.log("before");
outer();
console.log("after");
```

<details>
<summary><strong>Show Output & Explanation</strong></summary>

### Output

```txt
before
outer start
promise executor
after
outer end: resolved value
```

### Explanation

- `before` is synchronous.
- `outer()` runs: logs `outer start`, then evaluates the `new Promise(...)` expression.
- The Promise executor runs **synchronously**: logs `promise executor`, calls `resolve("resolved value")`.
- `await` receives the now-resolved Promise but still suspends `outer`, scheduling its continuation as a microtask.
- Back in global: logs `after`.
- Microtask fires: `outer` resumes → logs `outer end: resolved value`.

Key takeaway: the Promise **executor** is always synchronous — only the `.then`/`await` continuation is async.

</details>

---

## Final Tips

- The execution order rule is always: **sync → microtasks (all) → one macrotask → microtasks (all) → one macrotask → ...**
- `Promise.then`, `async/await` continuations, and `queueMicrotask` all go into the microtask queue.
- `setTimeout`, `setInterval`, and I/O callbacks go into the macrotask queue.
- `setTimeout(fn, 0)` is still a macrotask — it runs after all microtasks, even if a Promise is created after the `setTimeout`.
- Every time a macrotask finishes, the entire microtask queue is drained before the next macrotask runs.
- When a `.then` callback returns a **native Promise**, it takes one extra microtask tick to unwrap — this shifts subsequent handlers in that chain behind handlers of other chains already in the queue.
- The Promise executor is **always synchronous** — only the `.then`/`await` continuation is deferred.

<p align="right">
  <a href="#table-of-contents">⬆ Back to Top</a>
</p>
