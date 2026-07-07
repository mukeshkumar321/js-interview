## Chapter 8: Event Loop & Async JavaScript — Trick Output Questions

> Self-evaluate first. Predict the output, then reveal the answer.

---

### Q1. Microtask vs macrotask order

```js
console.log('1');

setTimeout(() => console.log('2'), 0);

Promise.resolve().then(() => console.log('3'));

console.log('4');
```

<details>
<summary>Show Output & Explanation</summary>

```
1
4
3
2
```

Synchronous code runs first (`1`, `4`). Then the **microtask queue** is drained — `Promise.then` callbacks run before any macrotasks (`3`). Only then does the **macrotask** (`setTimeout`) run (`2`). `setTimeout(fn, 0)` does NOT run immediately.

</details>

---

### Q2. Multiple microtasks vs one macrotask

```js
setTimeout(() => console.log('timeout'), 0);

Promise.resolve()
  .then(() => console.log('p1'))
  .then(() => console.log('p2'))
  .then(() => console.log('p3'));

console.log('sync');
```

<details>
<summary>Show Output & Explanation</summary>

```
sync
p1
p2
p3
timeout
```

Synchronous runs first (`sync`). All **microtasks are drained completely** before any macrotask runs. Each `.then` queues the next `.then` as a new microtask — so `p1`, `p2`, `p3` all run before `timeout`.

</details>

---

### Q3. `async/await` is sugar for Promises

```js
async function foo() {
  console.log('A');
  await Promise.resolve();
  console.log('B');
}

console.log('1');
foo();
console.log('2');
```

<details>
<summary>Show Output & Explanation</summary>

```
1
A
2
B
```

`foo()` runs synchronously until the first `await`. `'A'` logs synchronously. The `await` suspends `foo` and returns control — `'2'` logs synchronously. The continuation after `await` is a microtask, so `'B'` logs after the current synchronous block finishes.

</details>

---

### Q4. `await` on a non-Promise

```js
async function test() {
  const val = await 42;
  console.log(val);
}

test();
console.log('after');
```

<details>
<summary>Show Output & Explanation</summary>

```
after
42
```

`await` on a non-Promise value wraps it in `Promise.resolve(42)`. The function still suspends at `await`, yielding control. `'after'` logs first (synchronous), then `42` logs as a microtask.

</details>

---

### Q5. `Promise.all` fails fast

```js
const p1 = Promise.resolve('one');
const p2 = Promise.reject('error');
const p3 = Promise.resolve('three');

Promise.all([p1, p2, p3])
  .then(values => console.log('resolved:', values))
  .catch(err => console.log('rejected:', err));
```

<details>
<summary>Show Output & Explanation</summary>

```
rejected: error
```

`Promise.all` **fails fast** — if any Promise rejects, the entire `Promise.all` rejects immediately with that rejection reason. The other Promises continue executing but their results are ignored. Use `Promise.allSettled` if you need all results regardless of failures.

</details>

---

### Q6. `Promise.allSettled` never rejects

```js
Promise.allSettled([
  Promise.resolve('ok'),
  Promise.reject('fail'),
  Promise.resolve('also ok'),
]).then(results => {
  results.forEach(r => console.log(r.status, r.value ?? r.reason));
});
```

<details>
<summary>Show Output & Explanation</summary>

```
fulfilled ok
rejected fail
fulfilled also ok
```

`Promise.allSettled` **never rejects** — it always resolves with an array of result objects, each with `status: 'fulfilled'` (and `value`) or `status: 'rejected'` (and `reason`). Essential when you need all results even if some fail.

</details>

---

### Q7. `Promise.race` resolves with the fastest

```js
const slow = new Promise(resolve => setTimeout(() => resolve('slow'), 200));
const fast = new Promise(resolve => setTimeout(() => resolve('fast'), 50));
const fail = new Promise((_, reject) => setTimeout(() => reject('fail'), 100));

Promise.race([slow, fast, fail])
  .then(v => console.log('resolved:', v))
  .catch(e => console.log('rejected:', e));
```

<details>
<summary>Show Output & Explanation</summary>

```
resolved: fast
```

`Promise.race` resolves/rejects with the **first Promise to settle**, regardless of outcome. `fast` resolves after 50ms — faster than both `slow` (200ms) and `fail` (100ms). The others are ignored after the first settles.

</details>

---

### Q8. Error in `.then` is caught by `.catch`

```js
Promise.resolve('ok')
  .then(val => {
    throw new Error('something broke');
    return val;
  })
  .then(val => console.log('then:', val))
  .catch(err => console.log('catch:', err.message));
```

<details>
<summary>Show Output & Explanation</summary>

```
catch: something broke
```

Throwing inside a `.then` callback automatically **rejects the returned Promise**. The next `.then` is skipped, and `.catch` handles the error. Promise chains automatically propagate errors through the chain until a `.catch` handles them.

</details>

---

### Q9. `async/await` error handling

```js
async function fetchData() {
  try {
    const result = await Promise.reject(new Error('network error'));
    console.log('success:', result);
  } catch (e) {
    console.log('caught:', e.message);
  }
}

fetchData();
```

<details>
<summary>Show Output & Explanation</summary>

```
caught: network error
```

`await` on a rejected Promise throws the rejection reason as an exception. `try/catch` around `await` catches this — it's equivalent to `.catch()` in the Promise chain. This is the main benefit of `async/await` for error handling ergonomics.

</details>

---

### Q10. Forgetting `await` — silent bug

```js
async function getValue() {
  return 42;
}

async function main() {
  const result = getValue(); // missing await!
  console.log(result);
  console.log(typeof result);
}

main();
```

<details>
<summary>Show Output & Explanation</summary>

```
Promise { 42 }
object
```

Without `await`, `getValue()` returns a **Promise object**, not `42`. `typeof` a Promise is `'object'`. This is a common silent bug — the code doesn't throw, but operates on a Promise instead of the resolved value. Always `await` async functions when you need their return value.

</details>

---

### Q11. `Promise` constructor executor runs synchronously

```js
console.log('before');

const p = new Promise(resolve => {
  console.log('inside executor');
  resolve('done');
});

p.then(v => console.log('resolved:', v));

console.log('after');
```

<details>
<summary>Show Output & Explanation</summary>

```
before
inside executor
after
resolved: done
```

The **executor function** passed to `new Promise()` runs **synchronously**. `.then` callbacks are microtasks and run after the current synchronous block. Many developers assume the executor is async — it's not.

</details>

---

### Q12. Chaining with returned Promises

```js
Promise.resolve(1)
  .then(v => {
    console.log(v);
    return Promise.resolve(2);
  })
  .then(v => {
    console.log(v);
    return 3;
  })
  .then(v => console.log(v));
```

<details>
<summary>Show Output & Explanation</summary>

```
1
2
3
```

Returning a Promise from `.then` **flattens** it — the next `.then` receives the resolved value of that inner Promise, not the Promise itself. Returning a plain value wraps it in `Promise.resolve`. This is how Promise chains avoid nesting.

</details>

---

### Q13. `setTimeout` order with different delays

```js
setTimeout(() => console.log('A'), 100);
setTimeout(() => console.log('B'), 0);
setTimeout(() => console.log('C'), 50);
```

<details>
<summary>Show Output & Explanation</summary>

```
B
C
A
```

`setTimeout` fires approximately after the specified delay. `B` (0ms) fires first, then `C` (50ms), then `A` (100ms). Note: `setTimeout(fn, 0)` is never truly immediate — browser minimum is ~4ms and there's always task queue overhead.

</details>

---

### Q14. `async` function always returns a Promise

```js
async function greet() {
  return 'hello';
}

const result = greet();
console.log(result instanceof Promise);
result.then(v => console.log(v));
```

<details>
<summary>Show Output & Explanation</summary>

```
true
hello
```

An `async` function **always returns a Promise**, even if you return a plain value. The returned value is automatically wrapped in `Promise.resolve(...)`. This means every `async` function call produces a thenable.

</details>

---

### Q15. Parallel vs sequential `await`

```js
async function sequential() {
  const a = await new Promise(r => setTimeout(() => r(1), 100));
  const b = await new Promise(r => setTimeout(() => r(2), 100));
  return a + b;
}

async function parallel() {
  const [a, b] = await Promise.all([
    new Promise(r => setTimeout(() => r(1), 100)),
    new Promise(r => setTimeout(() => r(2), 100)),
  ]);
  return a + b;
}

// Both return 3, but sequential takes ~200ms, parallel takes ~100ms
sequential().then(console.log);
parallel().then(console.log);
```

<details>
<summary>Show Output & Explanation</summary>

```
3
3
```

Both return `3`, but `sequential` takes ~200ms (waits for each) while `parallel` takes ~100ms (both Promises run concurrently). When Promises don't depend on each other, use `Promise.all` for parallel execution. The order `3, 3` may vary slightly due to timing.

</details>

---

### Q16. Unhandled Promise rejection

```js
async function fail() {
  throw new Error('oops');
}

fail();
console.log('after fail()');
```

<details>
<summary>Show Output & Explanation</summary>

```
after fail()
UnhandledPromiseRejection: Error: oops
```

`fail()` returns a rejected Promise, but nothing handles it. `'after fail()'` logs first (synchronous). The unhandled rejection causes a warning/error — in Node.js this terminates the process by default. Always `await` or `.catch()` async functions.

</details>

---

### Q17. `.finally` receives no argument

```js
Promise.resolve('value')
  .then(v => {
    console.log('then:', v);
    return 'modified';
  })
  .finally(v => {
    console.log('finally arg:', v);
    return 'from finally';
  })
  .then(v => {
    console.log('after finally:', v);
  });
```

<details>
<summary>Show Output & Explanation</summary>

```
then: value
finally arg: undefined
after finally: modified
```

`.finally` receives **no argument** — it's for cleanup regardless of outcome. It also **passes through** the original resolved value (`'modified'`), not the value returned from `finally`. The `return 'from finally'` is ignored unless it's a rejection.

</details>

---

### Q18. `Promise.any` resolves with the first fulfilled

```js
Promise.any([
  Promise.reject('error1'),
  Promise.resolve('success'),
  Promise.reject('error2'),
])
  .then(v => console.log('resolved:', v))
  .catch(e => console.log('AggregateError:', e.errors));
```

<details>
<summary>Show Output & Explanation</summary>

```
resolved: success
```

`Promise.any` resolves with the **first fulfilled Promise**, ignoring rejections. It only rejects if **all** Promises reject — in that case with an `AggregateError` containing all rejection reasons. The inverse of `Promise.all` (which fails on any rejection).

</details>

---

### Q19. `queueMicrotask` vs `setTimeout`

```js
queueMicrotask(() => console.log('microtask'));
setTimeout(() => console.log('timeout'), 0);
console.log('sync');
```

<details>
<summary>Show Output & Explanation</summary>

```
sync
microtask
timeout
```

`queueMicrotask` explicitly adds to the **microtask queue** — same queue as Promise `.then` callbacks. Microtasks run before macrotasks. Order: synchronous (`sync`) → microtask queue (`microtask`) → macrotask queue (`timeout`).

</details>

---

### Q20. Async function with `for...of` loop

```js
async function processItems(items) {
  for (const item of items) {
    await new Promise(r => setTimeout(r, 10));
    console.log(item);
  }
}

processItems([1, 2, 3]);
console.log('called');
```

<details>
<summary>Show Output & Explanation</summary>

```
called
1
2
3
```

`processItems` returns a Promise immediately when it hits the first `await` — `'called'` logs synchronously. The `for...of` with `await` runs items **sequentially**, each waiting for the previous. For parallel processing, use `Promise.all(items.map(...))`.

</details>

---

### Q21. Rejected Promise inside `try` without `await`

```js
async function main() {
  try {
    Promise.reject(new Error('missed'));
  } catch (e) {
    console.log('caught:', e.message);
  }
  console.log('end');
}

main();
```

<details>
<summary>Show Output & Explanation</summary>

```
end
UnhandledPromiseRejection: Error: missed
```

`Promise.reject(...)` creates a rejected Promise **but without `await`, it's never awaited**. `try/catch` only catches synchronous throws — not unhandled Promise rejections. `'end'` logs normally, then the unhandled rejection surfaces. Add `await` before `Promise.reject(...)` to catch it.

</details>

---

### Q22. Event loop order with nested `setTimeout`

```js
setTimeout(() => {
  console.log('outer');
  setTimeout(() => console.log('inner'), 0);
  Promise.resolve().then(() => console.log('micro'));
}, 0);
```

<details>
<summary>Show Output & Explanation</summary>

```
outer
micro
inner
```

When the outer `setTimeout` fires, `'outer'` logs. Within that macrotask, a microtask (`Promise.then`) and another macrotask (`setTimeout`) are queued. The microtask queue is drained first (`'micro'`), then the next macrotask runs (`'inner'`).

</details>

---

### Q23. `AbortController` cancels fetch

```js
const controller = new AbortController();
const { signal } = controller;

fetch('/api/data', { signal })
  .then(r => r.json())
  .then(data => console.log('data:', data))
  .catch(err => {
    if (err.name === 'AbortError') {
      console.log('request was aborted');
    } else {
      console.log('other error');
    }
  });

controller.abort();
```

<details>
<summary>Show Output & Explanation</summary>

```
request was aborted
```

`controller.abort()` cancels the fetch. The `fetch` Promise rejects with a `DOMException` where `name === 'AbortError'`. This is the standard pattern for cancellable requests — useful in `useEffect` cleanup in React to avoid state updates on unmounted components.

</details>

---

### Q24. `async/await` vs Promise chain error propagation

```js
async function outer() {
  try {
    await inner();
  } catch (e) {
    console.log('outer caught:', e.message);
  }
}

async function inner() {
  throw new Error('inner error');
}

outer();
```

<details>
<summary>Show Output & Explanation</summary>

```
outer caught: inner error
```

A throw inside an `async` function rejects the returned Promise. `await inner()` in `outer` catches that rejection and rethrows it as a regular exception — which `try/catch` catches. `async/await` seamlessly bridges Promise rejections and synchronous exceptions.

</details>

---

---

**Q25. What is the output?**

```js
async function* asyncRange(start, end) {
  for (let i = start; i <= end; i++) {
    await new Promise(r => setTimeout(r, 0));
    yield i;
  }
}

(async () => {
  for await (const num of asyncRange(1, 3)) {
    console.log(num);
  }
  console.log('done');
})();
```

<details>
<summary>Show Output & Explanation</summary>

```
1
2
3
done
```

**Explanation:**  
`async function*` is an async generator — it can both `await` and `yield`. `for await...of` iterates async iterables, awaiting each value. The `await new Promise(r => setTimeout(r, 0))` simulates async work per iteration. The loop waits for each yielded value before continuing. This pattern is key for processing streams of data lazily.

> **Common Mistake:** Using `for...of` (without `await`) on an async iterable — it would iterate over Promise objects, not the resolved values.

</details>

---

**Q26. What is the output?**

```js
console.log('1: sync start');

requestIdleCallback(() => {
  console.log('3: idle callback');
});

setTimeout(() => {
  console.log('2: timeout');
}, 0);

console.log('1: sync end');
```

<details>
<summary>Show Output & Explanation</summary>

```
1: sync start
1: sync end
2: timeout
3: idle callback
```

**Explanation:**  
`requestIdleCallback` runs during browser idle time — after all pending tasks (including setTimeout callbacks) have been processed, when the browser has spare time before the next frame. `setTimeout(fn, 0)` fires as a macrotask but still before idle time. Sync code always runs first.

> **Common Mistake:** Assuming `requestIdleCallback` fires immediately after the current task like `setTimeout(fn, 0)`. It only runs when the browser is genuinely idle, which could be much later or throttled if the tab is busy.

</details>

---

**Q27. What is the output?**

```js
// main.js
const worker = new Worker('worker.js');

worker.postMessage({ type: 'ADD', a: 5, b: 3 });

worker.onmessage = (e) => {
  console.log('Result:', e.data.result);
};

console.log('Message sent');

// worker.js
self.onmessage = (e) => {
  const { type, a, b } = e.data;
  if (type === 'ADD') {
    self.postMessage({ result: a + b });
  }
};
```

<details>
<summary>Show Output & Explanation</summary>

```
Message sent
Result: 8
```

**Explanation:**  
Web Workers run in a separate thread — `postMessage` is async. Sync code (`'Message sent'`) runs before the worker responds. The worker receives the message, computes `5 + 3 = 8`, and sends back `{ result: 8 }` via `self.postMessage`. The main thread's `onmessage` then fires with `e.data.result = 8`. Workers don't share memory — all data is copied (structured clone).

> **Common Mistake:** Expecting the worker response to be available synchronously after `postMessage`. Worker communication is always async — you must use `onmessage`/`addEventListener` to receive results.

</details>

---

*Chapter 8 of JavaScript Interview Prep Series*
