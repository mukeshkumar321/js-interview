# Chapter 8: Event Loop & Async JavaScript

> **Target:** Frontend Developer (4+ years) | Interview-focused, practical explanations

---

## Table of Contents

- [JavaScript Concurrency Model](#javascript-concurrency-model)
- [Timers & Callbacks](#timers--callbacks)
- [Promises](#promises)
- [Async/Await](#asyncawait)
- [Advanced Async Patterns](#advanced-async-patterns)

---

## JavaScript Concurrency Model

<details>
<summary><strong>1. How does JavaScript, being single-threaded, handle asynchronous operations?</strong></summary>

JavaScript has a single call stack — it can only do one thing at a time. Asynchronous work is offloaded to **browser/Node APIs** (Web APIs), which run in separate threads. When complete, callbacks are queued back and the event loop picks them up.

```
Your Code → Call Stack (single thread)
                ↓ (async call e.g. fetch, setTimeout)
           Web APIs (multi-threaded, provided by browser)
                ↓ (when complete)
           Callback/Microtask Queue
                ↓ (when stack is empty)
           Event Loop picks up next task
```

This is why `setTimeout` and `fetch` don't block the main thread — the *waiting* happens in a separate browser thread, while JavaScript keeps running.

</details>

---

<details>
<summary><strong>2. What is the Event Loop and why is it needed?</strong></summary>

The **Event Loop** is a mechanism that continuously checks: *"Is the call stack empty? If yes, is there anything in the queues?"* If so, it moves the next task onto the stack.

```
while (true) {
  if (callStack.isEmpty()) {
    // drain ALL microtasks first
    while (microtaskQueue.hasItems()) {
      callStack.push(microtaskQueue.dequeue());
    }
    // then one macrotask
    if (taskQueue.hasItems()) {
      callStack.push(taskQueue.dequeue());
    }
  }
}
```

**Why it's needed:** Without it, async callbacks would have no mechanism to re-enter the JS runtime after their background work completes.

</details>

---

<details>
<summary><strong>3. What are the main components of the Event Loop architecture?</strong></summary>

```
┌─────────────────────────┐
│       Call Stack         │ ← JS executes here
└────────────┬────────────┘
             │ async calls
┌────────────▼────────────┐
│  Web APIs / Node APIs    │ ← setTimeout, fetch, I/O (browser-managed)
└────────────┬────────────┘
             │ complete
    ┌────────┴────────┐
    ▼                 ▼
┌──────────┐  ┌─────────────────┐
│  Task    │  │  Microtask      │
│  Queue   │  │  Queue          │
│(macrotask│  │ (Promises,      │
│ setTimeout│  │  queueMicrotask)│
│  I/O)    │  └────────────────┘
└──────────┘         ↑ drained FIRST
         Event Loop picks next
```

- **Call Stack** — where JS executes synchronously
- **Web APIs** — browser-managed: `setTimeout`, `fetch`, `DOM events`
- **Task Queue (Macrotask)** — `setTimeout`, `setInterval`, I/O, UI events
- **Microtask Queue** — `Promise.then`, `queueMicrotask`, `MutationObserver`

</details>

---

<details>
<summary><strong>4 & 5. What is the difference between the Callback Queue and Microtask Queue? What is the execution order?</strong></summary>

**Execution order per event loop tick:**
1. Run all synchronous code (call stack drains)
2. Run **all** microtasks (Promise callbacks, `queueMicrotask`) — repeat until queue empty
3. Run **one** macrotask (setTimeout, setInterval, I/O)
4. Render (if needed)
5. Repeat

```js
console.log('1 - sync');

setTimeout(() => console.log('4 - macrotask'), 0);

Promise.resolve()
  .then(() => console.log('2 - microtask 1'))
  .then(() => console.log('3 - microtask 2'));

console.log('1b - sync');

// Output: 1 - sync → 1b - sync → 2 - microtask 1 → 3 - microtask 2 → 4 - macrotask
```

> **Key rule:** Microtasks drain completely before the next macrotask runs. This means if a microtask schedules another microtask, the new one also runs before any macrotask.

</details>

---

<details>
<summary><strong>6. What are microtasks and macrotasks? Give examples.</strong></summary>

| | Microtasks | Macrotasks |
|--|-----------|------------|
| **Examples** | `Promise.then/catch/finally`, `queueMicrotask`, `MutationObserver` | `setTimeout`, `setInterval`, `setImmediate`, I/O callbacks, UI events |
| **Priority** | High — run before next macrotask | Low — run one per tick |
| **Queue drain** | All at once | One per tick |

```js
// Microtasks (Promise)
Promise.resolve().then(() => console.log('micro'));

// Macrotasks (setTimeout)
setTimeout(() => console.log('macro'), 0);

// queueMicrotask — explicit microtask scheduling
queueMicrotask(() => console.log('explicit micro'));

// Order: 'explicit micro' → 'micro' → 'macro'
```

</details>

---

<details>
<summary><strong>7. Why are Promise callbacks executed before <code>setTimeout()</code> callbacks?</strong></summary>

Promises use the **Microtask Queue**, which has higher priority than the **Task Queue** (macrotasks). After every task (including the initial script), the engine drains all microtasks before picking the next macrotask.

```js
setTimeout(() => console.log('timeout'), 0);
Promise.resolve().then(() => console.log('promise'));
console.log('sync');

// Output: sync → promise → timeout
// Even though setTimeout was registered first, Promise wins
```

This design ensures that promise chains resolve in a predictable, tight sequence — important for maintaining data consistency before the browser can render or handle other events.

</details>

---

## Timers & Callbacks

<details>
<summary><strong>8. What happens internally when <code>setTimeout()</code> is executed?</strong></summary>

1. `setTimeout(fn, delay)` is called — JS hands the callback + timer to the **Web API**
2. JS continues executing (non-blocking)
3. After `delay` ms, the Web API moves `fn` to the **Task Queue**
4. Event Loop picks it up when the call stack is empty

```js
console.log('start');

setTimeout(() => console.log('timeout'), 1000);

console.log('end');

// start → end → (1 second later) → timeout
```

**The timer doesn't run in JavaScript** — it runs in the browser's timer API. JS just registers it and moves on.

</details>

---

<details>
<summary><strong>9. Why does <code>setTimeout(fn, 0)</code> not execute immediately?</strong></summary>

Even with `0` delay, the callback goes through the Web API → Task Queue → Event Loop cycle. It runs only after:
1. All current synchronous code finishes
2. All pending microtasks are drained

```js
console.log('1');
setTimeout(() => console.log('3'), 0);
Promise.resolve().then(() => console.log('2 - micro'));
console.log('1b');
// 1 → 1b → 2 - micro → 3
```

**Minimum delay:** Browsers enforce a minimum of ~4ms for nested `setTimeout` calls (per spec). The `0` is a floor request, not a guarantee.

**Practical use:** `setTimeout(fn, 0)` is used to defer execution until after the current synchronous block and pending microtasks — useful for deferring non-critical UI updates.

</details>

---

<details>
<summary><strong>10. What is Callback Hell and why is it a problem?</strong></summary>

Callback Hell (the "pyramid of doom") is deeply nested callbacks for sequential async operations — hard to read, reason about, and maintain.

```js
// ❌ Callback Hell
getUser(userId, function(err, user) {
  if (err) return handleError(err);
  getOrders(user.id, function(err, orders) {
    if (err) return handleError(err);
    getOrderDetails(orders[0].id, function(err, details) {
      if (err) return handleError(err);
      updateUI(details);
    });
  });
});
```

**Problems:**
- Hard to read (pyramid shape grows right)
- Error handling repeated at every level
- Difficult to add logic or reorder operations
- Not composable — callbacks can't be returned or passed around easily

</details>

---

<details>
<summary><strong>11. What are common techniques used to avoid Callback Hell?</strong></summary>

```js
// 1. Named functions (flatten nesting)
function handleDetails(err, details) { if (err) throw err; updateUI(details); }
function handleOrders(err, orders)   { getOrderDetails(orders[0].id, handleDetails); }
getUser(id, (err, user) => getOrders(user.id, handleOrders));

// 2. Promises (chain instead of nest)
getUser(userId)
  .then(user => getOrders(user.id))
  .then(orders => getOrderDetails(orders[0].id))
  .then(details => updateUI(details))
  .catch(handleError);

// 3. Async/Await (reads like synchronous)
async function loadUserData(userId) {
  try {
    const user    = await getUser(userId);
    const orders  = await getOrders(user.id);
    const details = await getOrderDetails(orders[0].id);
    updateUI(details);
  } catch (err) {
    handleError(err);
  }
}
```

</details>

---

## Promises

<details>
<summary><strong>12. What are Promises and what problems do they solve?</strong></summary>

A **Promise** is an object representing the eventual result (or failure) of an async operation. It provides a cleaner API than callbacks for async control flow.

```js
// Creating a Promise
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    Math.random() > 0.5 ? resolve('success') : reject(new Error('fail'));
  }, 1000);
});

// Consuming
promise
  .then(result => console.log(result))
  .catch(err   => console.error(err))
  .finally(()  => console.log('done'));
```

**Problems Promises solve:**
- Inversion of control (callbacks hand control to the callee — promises don't)
- Better error propagation (one `.catch` at the end)
- Composability — promises can be returned, passed, and combined

</details>

---

<details>
<summary><strong>13. What are the three states of a Promise?</strong></summary>

| State | Meaning | Terminal? |
|-------|---------|-----------|
| **Pending** | Initial state — not yet settled | No |
| **Fulfilled** | Completed successfully | Yes |
| **Rejected** | Failed with an error | Yes |

```js
const p = new Promise(resolve => setTimeout(resolve, 1000));
// Initially: pending
// After 1s:  fulfilled

// Once settled (fulfilled or rejected), a Promise CANNOT change state
const resolved = Promise.resolve(42);  // immediately fulfilled
const rejected = Promise.reject('err'); // immediately rejected
```

A Promise that is either fulfilled or rejected is called **settled**. Settled promises are immutable.

</details>

---

<details>
<summary><strong>14. How do <code>.then()</code>, <code>.catch()</code>, and <code>.finally()</code> work?</strong></summary>

```js
promise
  .then(
    value => { /* runs on fulfill */  return newValue; },
    error => { /* optional: runs on reject */ }
  )
  .catch(error => { /* runs on any rejection up the chain */ })
  .finally(() => { /* always runs, regardless of outcome */ });
```

**Key behaviors:**
- `.then()` returns a **new Promise** — enables chaining
- If `.then()` throws, the next `.catch()` catches it
- `.catch(fn)` is shorthand for `.then(undefined, fn)`
- `.finally()` receives no value — used for cleanup (hide spinner, close connection)

```js
fetch('/api/data')
  .then(res => res.json())         // parse
  .then(data => processData(data)) // use
  .catch(err => showError(err))    // handle any error in chain
  .finally(() => hideLoader());    // always clean up
```

</details>

---

<details>
<summary><strong>15 & 16. How does Promise Chaining work? What is Promise Error Propagation?</strong></summary>

Each `.then()` returns a new promise. The value returned from a `.then()` callback becomes the resolved value of the next promise in the chain.

```js
Promise.resolve(1)
  .then(n => n + 1)    // 2
  .then(n => n * 10)   // 20
  .then(n => {
    throw new Error('oops'); // triggers catch
    return n + 1;
  })
  .then(n => console.log('never reached'))
  .catch(err => console.log(err.message)); // 'oops'
```

**Error propagation:** A rejection skips all `.then()` handlers until a `.catch()` is found. After `.catch()` handles the error (by returning normally), subsequent `.then()` calls resume normally.

```js
Promise.reject('error')
  .then(() => console.log('skipped'))
  .then(() => console.log('skipped'))
  .catch(e  => { console.log('caught:', e); return 'recovered'; })
  .then(v   => console.log('resumed:', v)); // 'resumed: recovered'
```

</details>

---

<details>
<summary><strong>17. What happens when a Promise is resolved with another Promise?</strong></summary>

When you resolve a Promise with another Promise, JavaScript **adopts the state** of the inner promise — it "unwraps" it automatically.

```js
const inner = new Promise(resolve => setTimeout(() => resolve(42), 1000));
const outer = Promise.resolve(inner);

outer.then(val => console.log(val)); // 42 (after 1s) — not the Promise object

// Same behavior in .then()
Promise.resolve(1)
  .then(() => Promise.resolve(99)) // returning a promise
  .then(val => console.log(val));  // 99 — automatically unwrapped
```

This enables transparent async chaining — you can return a promise from `.then()` and the chain waits for it.

</details>

---

<details>
<summary><strong>18. What are <code>Promise.all()</code>, <code>Promise.allSettled()</code>, <code>Promise.race()</code>, and <code>Promise.any()</code>?</strong></summary>

| Method | Resolves when | Rejects when | Use case |
|--------|--------------|-------------|---------|
| `Promise.all()` | **All** fulfill | **Any** rejects | All required |
| `Promise.allSettled()` | **All** settle (any outcome) | Never | Report all results |
| `Promise.race()` | **First** settles (fulfill or reject) | First rejects | Timeout pattern |
| `Promise.any()` | **First** fulfills | **All** reject (AggregateError) | First success |

```js
const p1 = fetch('/api/users');
const p2 = fetch('/api/orders');
const p3 = fetch('/api/products');

// All must succeed
const [users, orders, products] = await Promise.all([p1, p2, p3]);

// Report results regardless of failures
const results = await Promise.allSettled([p1, p2, p3]);
results.forEach(r => {
  if (r.status === 'fulfilled') use(r.value);
  else logError(r.reason);
});

// Timeout pattern with race
const timeout = new Promise((_, reject) =>
  setTimeout(() => reject(new Error('timeout')), 5000));
const data = await Promise.race([fetch('/api/data'), timeout]);

// Try multiple sources — first success wins
const fastData = await Promise.any([cdn1, cdn2, cdn3]);
```

</details>

---

<details>
<summary><strong>19. How is a Promise executed internally?</strong></summary>

The **executor function** passed to `new Promise()` runs **synchronously** and immediately. Only the resolution callbacks (`.then`, `.catch`) are scheduled as microtasks.

```js
console.log('1 - before');

const p = new Promise((resolve) => {
  console.log('2 - executor runs sync'); // runs right now
  resolve('value');
  console.log('3 - after resolve');      // still runs
});

p.then(v => console.log('5 - then:', v)); // scheduled as microtask

console.log('4 - after new Promise');

// Output: 1 → 2 → 3 → 4 → 5
```

**Internally:**
1. `new Promise(executor)` — executor runs synchronously, creating the promise object
2. `resolve(value)` — marks the promise as fulfilled, queues `.then` callbacks as microtasks
3. `.then(fn)` — if promise is already settled, `fn` is queued immediately as a microtask; if pending, `fn` is stored to be queued when settled
4. Microtasks run after the current synchronous block finishes

> **Key insight:** The executor is sync, but the callbacks are always async (microtasks). A `.then()` callback never runs synchronously, even on an already-resolved promise.

</details>

---

<details>
<summary><strong>20. What is Promise Flattening (Promise Resolution Procedure)?</strong></summary>

When a `.then()` callback returns a **thenable** (anything with a `.then` method), the Promise engine doesn't wrap it as a value — it **adopts its state** instead. This is the Promise Resolution Procedure (PRP), defined in the spec.

```js
// Returning a plain value — wraps it
Promise.resolve(1)
  .then(() => 42)
  .then(v => console.log(v)); // 42

// Returning a Promise — flattened (not nested)
Promise.resolve(1)
  .then(() => Promise.resolve(42))
  .then(v => console.log(v)); // 42 — NOT Promise<42>

// Nested promises are automatically unwrapped
Promise.resolve(
  Promise.resolve(
    Promise.resolve('deep')
  )
).then(v => console.log(v)); // 'deep' — fully flattened
```

**Why it matters:** Without flattening, chaining async operations would produce `Promise<Promise<Promise<value>>>`. Flattening keeps chains flat and composable.

**Thenable duck-typing:** Any object with a `.then` method is treated as a promise-like, not just native Promises — this enables interoperability between different promise libraries.

```js
// Custom thenable
const thenable = {
  then(resolve) { resolve(100); }
};
Promise.resolve(thenable).then(v => console.log(v)); // 100
```

</details>

---

<details>
<summary><strong>21. What is the difference between returning a value and returning a Promise from <code>.then()</code>?</strong></summary>

```js
// Returning a plain value — next .then() runs in the next microtask tick
Promise.resolve()
  .then(() => 42)           // wraps 42 in a resolved promise
  .then(v => console.log(v)); // 42

// Returning a Promise — chain WAITS for that promise to settle
Promise.resolve()
  .then(() => new Promise(resolve => setTimeout(() => resolve(42), 1000)))
  .then(v => console.log(v)); // 42 — but after 1 second
```

| | Return a value | Return a Promise |
|--|---------------|-----------------|
| **Next `.then()` timing** | Next microtask tick | When the returned promise settles |
| **Chain behavior** | Passes value through | Waits, then unwraps |
| **Use case** | Transform data | Trigger another async operation |

**Practical implication:** This is what makes async chaining work cleanly — each step can optionally hand off to another async operation and the chain waits automatically.

```js
fetch('/api/user')
  .then(res => res.json())              // returns Promise — chain waits
  .then(user => user.name.toUpperCase()) // returns value — instant
  .then(name => console.log(name));
```

</details>

---

<details>
<summary><strong>22. Can a Promise be settled more than once?</strong></summary>

**No.** Once a Promise is settled (fulfilled or rejected), calling `resolve` or `reject` again is silently ignored. The state transition is one-way and permanent.

```js
const p = new Promise((resolve, reject) => {
  resolve('first');
  resolve('second'); // ignored
  reject('error');   // ignored
});

p.then(v => console.log(v)); // 'first' — only the first resolve counts
```

**Why this matters:**
- Guarantees predictability — a promise always delivers the same value
- Safe to pass a promise to multiple consumers (each gets the same result)
- Avoids race conditions where both resolve and reject might be called

```js
// Common gotcha — async code calling resolve twice
function fetchWithFallback(url) {
  return new Promise((resolve, reject) => {
    fetch(url)
      .then(res => resolve(res))
      .catch(() => {
        resolve(fallbackData); // safe — if fetch already resolved, this is ignored
      });
  });
}
```

</details>

---

<details>
<summary><strong>23. How do <code>Promise.resolve()</code> and <code>Promise.reject()</code> work?</strong></summary>

**`Promise.resolve(value)`** — creates an already-fulfilled promise, with special handling for thenables:

```js
// Plain value — wraps in a fulfilled promise
Promise.resolve(42).then(v => console.log(v)); // 42

// Existing Promise — returns it AS-IS (no wrapping)
const p = Promise.resolve(42);
Promise.resolve(p) === p; // true — same reference

// Thenable — adopts its eventual value
Promise.resolve({ then: resolve => resolve(99) })
  .then(v => console.log(v)); // 99
```

**`Promise.reject(reason)`** — always wraps in a rejected promise, even if passed a Promise:

```js
// Always rejects — even if passed a Promise
const p = Promise.resolve(42);
Promise.reject(p).catch(v => console.log(v)); // logs the Promise object, not 42

// Common use — create a pre-rejected promise for testing or short-circuit
function mustBeLoggedIn() {
  if (!user) return Promise.reject(new Error('Not authenticated'));
  return fetchUserData();
}
```

> **Key difference:** `Promise.resolve` is "smart" about thenables and existing promises. `Promise.reject` is dumb — it always rejects with exactly what you pass it.

</details>

---

<details>
<summary><strong>24. How do you create your own Promise-based APIs?</strong></summary>

Wrap callback-based or event-based APIs in `new Promise()` — exposing a clean promise interface to consumers.

```js
// 1. Promisify a callback-based API
function readFile(path) {
  return new Promise((resolve, reject) => {
    fs.readFile(path, 'utf8', (err, data) => {
      if (err) reject(err);
      else resolve(data);
    });
  });
}

// 2. Promisify a timeout
function delay(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}
await delay(1000); // pause for 1 second

// 3. Promisify DOM events
function waitForClick(element) {
  return new Promise(resolve => {
    element.addEventListener('click', resolve, { once: true });
  });
}
await waitForClick(button);
console.log('Button was clicked!');

// 4. Cancellable promise with AbortController
function fetchWithAbort(url, signal) {
  return new Promise((resolve, reject) => {
    signal.addEventListener('abort', () => reject(new DOMException('Aborted', 'AbortError')));
    fetch(url).then(resolve).catch(reject);
  });
}
```

**Best practices:**
- Always handle both the success and error paths
- Avoid the "explicit promise construction antipattern" — if a function already returns a promise, don't wrap it again
- Use `{ once: true }` for event listeners to avoid memory leaks

```js
// ❌ Antipattern — unnecessary wrapping
function getData() {
  return new Promise((resolve, reject) => {
    fetch('/api').then(resolve).catch(reject); // just return fetch() directly
  });
}

// ✅
function getData() {
  return fetch('/api');
}
```

</details>

---

## Async/Await

<details>
<summary><strong>25. What is <code>async/await</code> and what problems does it solve?</strong></summary>

`async/await` is syntactic sugar over Promises, making async code read and reason about like synchronous code.

```js
// Promise chain
function loadData() {
  return fetch('/api/user')
    .then(res => res.json())
    .then(user => fetch(`/api/orders/${user.id}`))
    .then(res => res.json());
}

// Async/await — same logic, much clearer
async function loadData() {
  const userRes = await fetch('/api/user');
  const user    = await userRes.json();
  const ordersRes = await fetch(`/api/orders/${user.id}`);
  return ordersRes.json();
}
```

**What `async` does:** Makes the function always return a Promise (wraps the return value).  
**What `await` does:** Pauses execution of the async function until the Promise resolves — without blocking the thread.

</details>

---

<details>
<summary><strong>26. What happens internally when JavaScript encounters an <code>await</code> statement?</strong></summary>

1. The `async` function pauses at `await`
2. The Promise being awaited is registered
3. The function yields control back to the caller / event loop
4. The rest of the async function is scheduled as a microtask when the promise resolves

```js
async function foo() {
  console.log('A');
  const result = await Promise.resolve('B');
  console.log(result); // runs as microtask after current sync code
  console.log('C');
}

foo();
console.log('D');

// Output: A → D → B → C
```

`await` doesn't block the thread — it suspends only the current async function, returning control to the caller immediately.

</details>

---

<details>
<summary><strong>27. Why can <code>await</code> only be used inside an <code>async</code> function?</strong></summary>

`await` works by **suspending the current function** and resuming it later as a microtask. This requires the function to be transformed into a state machine by the JS engine — which only happens when a function is marked `async`.

A regular function has no mechanism to pause mid-execution and resume. The `async` keyword opts the function into this transformation.

```js
// ❌ SyntaxError — await outside async function
function getData() {
  const data = await fetch('/api'); // SyntaxError
}

// ✅
async function getData() {
  const data = await fetch('/api'); // valid
}

// Top-level await — allowed in ES modules (.mjs or type="module")
// (the module itself is treated as an async context)
const data = await fetch('/api'); // valid at top level in a module
```

**Under the hood:** The JS engine rewrites `async` functions as generator-like state machines. Each `await` is a yield point. Without `async`, there's no state machine, so `await` has nowhere to pause.

> **Interview note:** Top-level `await` is valid in ES modules (supported in modern browsers and Node.js v14.8+), which is why you can use `await` at the top level in a `.mjs` file.

</details>

---

<details>
<summary><strong>28. What happens when you <code>await</code> a non-Promise value?</strong></summary>

The value is **implicitly wrapped** in `Promise.resolve()` first. It still causes a microtask tick — the code after `await` is always async, even for non-promises.

```js
async function example() {
  console.log('A');
  const x = await 42; // same as await Promise.resolve(42)
  console.log('B', x);
}

example();
console.log('C');

// Output: A → C → B 42
// Even though 42 isn't async, the microtask tick still defers 'B'
```

**Practical implication:**

```js
// These are equivalent:
const a = await 42;
const b = await Promise.resolve(42);

// Both defer the continuation to a microtask
// Both produce the same value
```

> **Why it matters:** You can safely `await` a value that *might* be a Promise or might be a plain value — `await` handles both cases correctly. This is useful when a function sometimes returns a promise and sometimes a cached plain value.

```js
async function getUser(id) {
  const cached = cache.get(id);
  return await cached ?? fetchUser(id); // works whether cached is a value or promise
}
```

</details>

---

<details>
<summary><strong>29. Does <code>await</code> block JavaScript execution?</strong></summary>

**No.** `await` only pauses the current `async` function — it does not block the call stack or the event loop. Other code continues to run while the awaited promise is pending.

```js
async function slowTask() {
  console.log('slow: start');
  await delay(2000); // pauses slowTask, not the whole program
  console.log('slow: done');
}

async function fastTask() {
  console.log('fast: start');
  await delay(100);
  console.log('fast: done');
}

slowTask();
fastTask();

// Output:
// slow: start
// fast: start
// fast: done   (after ~100ms)
// slow: done   (after ~2000ms)
```

**Contrast with truly blocking code:**

```js
// ❌ This DOES block — synchronous busy-wait
function blockingDelay(ms) {
  const end = Date.now() + ms;
  while (Date.now() < end) {} // freezes the entire thread
}

// ✅ This does NOT block — async suspension
async function asyncDelay(ms) {
  await new Promise(resolve => setTimeout(resolve, ms));
}
```

> `await` is cooperative multitasking — the function voluntarily yields control and is resumed later. The thread is free for other work in between.

</details>

---

<details>
<summary><strong>30. How is error handling done with <code>async/await</code>?</strong></summary>

```js
// 1. try/catch — most common
async function fetchUser(id) {
  try {
    const res = await fetch(`/api/users/${id}`);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return await res.json();
  } catch (err) {
    console.error('Failed:', err.message);
    return null; // graceful fallback
  }
}

// 2. .catch() on the returned Promise
const user = await fetchUser(1).catch(err => null);

// 3. Helper to avoid try/catch everywhere
async function to(promise) {
  try {
    return [null, await promise];
  } catch (err) {
    return [err, null];
  }
}
const [err, data] = await to(fetchUser(1));
if (err) handleError(err);
```

> **Common Mistake:** Forgetting to handle rejections in async functions — unhandled rejections crash Node.js processes and cause silent failures in browsers.

</details>

---

<details>
<summary><strong>31. Sequential vs Parallel execution using async/await.</strong></summary>

**Sequential** — each `await` waits for the previous to finish before starting the next. Total time = sum of all durations.

```js
async function sequential() {
  const user    = await fetchUser();    // wait ~200ms
  const orders  = await fetchOrders();  // wait ~300ms after user finishes
  const reviews = await fetchReviews(); // wait ~100ms after orders finishes
  // Total: ~600ms
}
```

**Parallel** — fire all requests simultaneously, then collect results. Total time = slowest operation.

```js
async function parallel() {
  const [user, orders, reviews] = await Promise.all([
    fetchUser(),    // ~200ms ┐
    fetchOrders(),  // ~300ms ├ all running at the same time
    fetchReviews(), // ~100ms ┘
  ]);
  // Total: ~300ms (the slowest one)
}
```

**Start parallel, await individually** — fires requests simultaneously but accesses results in order:

```js
async function startAll() {
  const pUser    = fetchUser();    // fires immediately
  const pOrders  = fetchOrders();  // fires immediately
  const pReviews = fetchReviews(); // fires immediately

  const user    = await pUser;    // waits for user
  const orders  = await pOrders;  // likely already done
  const reviews = await pReviews; // likely already done
}
```

> **Interview note:** The most impactful async performance win is usually switching sequential `await` chains to `Promise.all` for independent operations. Always ask: "do these operations depend on each other's results?" If not, run them in parallel.

</details>

---

<details>
<summary><strong>32. What is the difference between <code>.then()</code> and <code>await</code>?</strong></summary>

Both consume Promises, but differ in style, error handling, and composability.

| | `.then()` | `await` |
|--|----------|---------|
| **Style** | Functional/chainable | Imperative, reads like sync |
| **Error handling** | `.catch()` at end of chain | `try/catch` block |
| **Scope** | Each callback is its own closure | All in one function scope |
| **Requirement** | Works anywhere | Must be inside `async` function |
| **Conditional logic** | Awkward with if/else | Natural |
| **Debugging** | Stack traces can be unclear | Better stack traces |

```js
// .then() — great for simple linear chains
fetch('/api/user')
  .then(res => res.json())
  .then(user => processUser(user))
  .catch(handleError);

// await — better for complex logic, conditionals, loops
async function loadUser() {
  try {
    const res  = await fetch('/api/user');
    const user = await res.json();

    if (user.isPremium) {
      const perks = await fetchPerks(user.id); // conditional async
      return { ...user, perks };
    }
    return user;
  } catch (err) {
    handleError(err);
  }
}
```

**They're equivalent under the hood** — `await` compiles down to `.then()`. Choose based on readability: simple chains → `.then()`, complex logic/conditionals/loops → `async/await`.

</details>

---

<details>
<summary><strong>33. What are common mistakes developers make with <code>async/await</code>?</strong></summary>

**1. Sequential awaits when parallel is possible**
```js
// ❌ Slow — waits for each before starting next
const a = await fetchA();
const b = await fetchB();

// ✅ Fast — both fire simultaneously
const [a, b] = await Promise.all([fetchA(), fetchB()]);
```

**2. Missing await — function returns Promise instead of value**
```js
async function getData() {
  return fetch('/api').then(r => r.json()); // forgot await
}
const data = getData(); // Promise, not the data!
```

**3. await inside forEach (doesn't work)**
```js
// ❌ forEach is not async-aware
ids.forEach(async id => { await processId(id); }); // runs concurrently, not sequentially

// ✅ for...of for sequential
for (const id of ids) { await processId(id); }

// ✅ Promise.all for parallel
await Promise.all(ids.map(id => processId(id)));
```

**4. Not catching errors at the top level**
```js
// ❌ Unhandled rejection
async function main() { await riskyOperation(); }
main(); // no .catch()

// ✅
main().catch(console.error);
```

</details>

---

## Advanced Async Patterns

<details>
<summary><strong>34 & 35. How can multiple async operations run in parallel? Sequential vs Parallel?</strong></summary>

```js
// Sequential — total time = sum of all durations
async function sequential() {
  const a = await delay(1000); // wait 1s
  const b = await delay(1000); // wait 1s
  // Total: ~2s
}

// Parallel — total time = max duration
async function parallel() {
  const [a, b] = await Promise.all([delay(1000), delay(1000)]);
  // Total: ~1s
}

// Start all, await individually (parallel but ordered access)
async function startAll() {
  const pA = fetchA(); // fire immediately — no await
  const pB = fetchB(); // fire immediately — no await
  const a = await pA;  // now wait
  const b = await pB;  // already running
}
```

> **Interview Note:** The most impactful performance improvement in async code is often simply parallelizing independent requests with `Promise.all`.

</details>

---

<details>
<summary><strong>36 & 37. What are race conditions and how does <code>AbortController</code> help?</strong></summary>

**Race condition:** Two async operations complete in an unpredictable order, causing stale data to overwrite fresh data.

```js
// ❌ Race condition — user types fast, older request resolves last
searchInput.addEventListener('input', async e => {
  const results = await search(e.target.value);
  displayResults(results); // could show stale results!
});

// ✅ Fix with AbortController
let controller;
searchInput.addEventListener('input', async e => {
  controller?.abort(); // cancel previous request
  controller = new AbortController();

  try {
    const results = await search(e.target.value, { signal: controller.signal });
    displayResults(results);
  } catch (err) {
    if (err.name !== 'AbortError') throw err; // ignore cancellation
  }
});
```

`AbortController` lets you cancel `fetch` requests by passing `signal` — when `abort()` is called, the fetch rejects with an `AbortError`.

</details>

---

<details>
<summary><strong>38. How would you limit concurrent API requests in a frontend application?</strong></summary>

```js
// Process at most `limit` requests concurrently
async function pLimit(tasks, limit) {
  const results = [];
  const executing = new Set();

  for (const task of tasks) {
    const p = task().then(result => {
      executing.delete(p);
      return result;
    });
    executing.add(p);
    results.push(p);

    if (executing.size >= limit) {
      await Promise.race(executing); // wait for one to finish
    }
  }

  return Promise.all(results);
}

// Usage — max 3 concurrent requests
const tasks = urls.map(url => () => fetch(url).then(r => r.json()));
const results = await pLimit(tasks, 3);
```

**Why it matters:** Sending 100 requests simultaneously can overwhelm the server or hit browser connection limits (typically 6 per domain). Concurrency limiting gives you control over load.

</details>

---

## Quick Reference Cheat Sheet

```
Event Loop order per tick:
  1. Synchronous code (call stack)
  2. ALL microtasks (Promises, queueMicrotask)
  3. ONE macrotask (setTimeout, I/O)
  4. Render

Microtasks: Promise.then/catch/finally, queueMicrotask, MutationObserver
Macrotasks: setTimeout, setInterval, setImmediate, I/O, click events

Promise states: pending → fulfilled | rejected (immutable once settled)

Promise internals:
  - Executor runs synchronously
  - .then() callbacks always run as microtasks (never sync)
  - Returning a Promise from .then() flattens the chain (PRP)

Promise combinators:
  all()        → all fulfill or first reject
  allSettled() → wait for all, never rejects
  race()       → first to settle (any outcome)
  any()        → first to fulfill or all rejected

async/await:
  - async fn always returns a Promise
  - await wraps non-Promise values in Promise.resolve()
  - await suspends the function, not the thread
  - Top-level await valid in ES modules

async/await common mistakes:
  - Sequential awaits for independent requests (use Promise.all)
  - await inside forEach (use for...of or Promise.all+map)
  - Missing error handling (always .catch or try/catch)
  - Forgetting await (function returns Promise instead of value)
```

---

*Chapter 8 of JavaScript Interview Prep Series*