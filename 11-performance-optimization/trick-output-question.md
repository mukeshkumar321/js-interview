## Chapter 11: Performance Optimization — Trick Output Questions

> Self-evaluate first. Predict the output, then reveal the answer.

---

### Q1. Debounce — when does it actually fire?

```js
function debounce(fn, delay) {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  };
}

const log = debounce(msg => console.log(msg), 300);

log('a');
log('b');
log('c');
// What gets logged and when?
```

<details>
<summary>Show Output & Explanation</summary>

```
c
// (logged after 300ms of silence)
```

Each `log()` call clears the previous timer and sets a new one. `'a'` and `'b'`'s timers are cancelled before they fire. Only `'c'`'s timer survives — it fires 300ms after `log('c')` is called. This is the core of debouncing: **fire once after silence**.

</details>

---

### Q2. Throttle — how often does it fire?

```js
function throttle(fn, interval) {
  let lastTime = 0;
  return (...args) => {
    const now = Date.now();
    if (now - lastTime >= interval) {
      lastTime = now;
      fn(...args);
    }
  };
}

const log = throttle(msg => console.log(msg), 100);

// Called at 0ms, 50ms, 100ms, 150ms
log('0ms');
setTimeout(() => log('50ms'), 50);
setTimeout(() => log('100ms'), 100);
setTimeout(() => log('150ms'), 150);
```

<details>
<summary>Show Output & Explanation</summary>

```
0ms
100ms
```

`'0ms'` fires at t=0 (`lastTime` set to ~0). `'50ms'` at t=50 — only 50ms since last, interval not met, **skipped**. `'100ms'` at t=100 — interval met, **fires**. `'150ms'` at t=150 — only 50ms since `'100ms'`, **skipped**. Throttle ensures **at most one call per interval**.

</details>

---

### Q3. Layout thrashing — how many reflows?

```js
const elements = document.querySelectorAll('.box');

// Version A:
elements.forEach(el => {
  const h = el.offsetHeight;       // read
  el.style.height = (h + 10) + 'px'; // write
});

// Version B:
const heights = [...elements].map(el => el.offsetHeight); // all reads
elements.forEach((el, i) => el.style.height = heights[i] + 10 + 'px'); // all writes
```

<details>
<summary>Show Output & Explanation</summary>

```
Version A: N reflows (one per element — read after write forces layout recalculation)
Version B: 1 reflow (all reads batched, then all writes batched)
```

Alternating read/write of layout-triggering properties (like `offsetHeight`) causes **layout thrashing** — each read after a write forces the browser to synchronously recalculate layout. Batching all reads first, then all writes, results in a single reflow.

</details>

---

### Q4. `transform` vs `top/left` — which triggers reflow?

```js
// Moving an element 100px to the right:

// Option A:
el.style.left = '100px'; // ?

// Option B:
el.style.transform = 'translateX(100px)'; // ?
```

<details>
<summary>Show Output & Explanation</summary>

```
Option A: triggers reflow + repaint (layout change)
Option B: triggers NEITHER reflow nor repaint — handled by GPU compositor
```

`left` (and `top`, `width`, `margin`, etc.) trigger **reflow** — the browser must recalculate the layout. `transform` and `opacity` are handled by the GPU compositor — no layout or paint step needed. Use `transform` for animations to maintain 60fps.

</details>

---

### Q5. `React.memo` — does it prevent re-render?

```jsx
const Child = React.memo(({ name }) => {
  console.log('Child rendered');
  return <div>{name}</div>;
});

function Parent() {
  const [count, setCount] = React.useState(0);
  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>+</button>
      <Child name="Alice" />
    </>
  );
}
// User clicks the button 3 times. How many times does 'Child rendered' log?
```

<details>
<summary>Show Output & Explanation</summary>

```
Child rendered  // once on initial render
// No more logs on subsequent button clicks
```

`React.memo` does a **shallow prop comparison**. `name="Alice"` never changes — same string reference each click. So `Child` is NOT re-rendered on button clicks. Without `React.memo`, `Child` would re-render on every parent state change (3 more times).

</details>

---

### Q6. `useMemo` with object dependency

```jsx
function Component({ items }) {
  const config = { threshold: 0.5 }; // new object every render

  const filtered = React.useMemo(
    () => items.filter(i => i.score > config.threshold),
    [items, config] // config changes every render!
  );

  return <div>{filtered.length}</div>;
}
// Does useMemo help here?
```

<details>
<summary>Show Output & Explanation</summary>

```
No — useMemo recomputes on every render
```

`config` is created **inline** — a new object reference every render. React's dependency comparison uses `Object.is()` (reference equality). Since `config` is always a new reference, `useMemo` recomputes every time. Fix: move `config` outside the component, or `useMemo` it separately with no dependencies.

</details>

---

### Q7. `useCallback` stabilizes function reference

```jsx
const Child = React.memo(({ onClick }) => {
  console.log('Child rendered');
  return <button onClick={onClick}>Click</button>;
});

function Parent() {
  const [count, setCount] = React.useState(0);

  // Version A — no useCallback:
  const handleClick = () => console.log('clicked');

  // Version B — with useCallback:
  // const handleClick = React.useCallback(() => console.log('clicked'), []);

  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>+</button>
      <Child onClick={handleClick} />
    </>
  );
}
// With Version A: how many times does 'Child rendered' log after 3 button clicks?
```

<details>
<summary>Show Output & Explanation</summary>

```
Version A: 4 times (1 initial + 3 on each click)
Version B: 1 time (initial only)
```

Version A: `handleClick` is a **new function reference** every render. `React.memo`'s prop comparison sees a new function → re-renders `Child` every time. Version B: `useCallback` with `[]` creates a **stable reference** — same function across renders → `Child` never re-renders after mount.

</details>

---

### Q8. Virtualization — how many DOM nodes for 10,000 items?

```jsx
import { FixedSizeList } from 'react-window';

function App() {
  return (
    <FixedSizeList
      height={500}
      itemCount={10000}
      itemSize={35}
      width={300}
    >
      {({ index, style }) => <div style={style}>Row {index}</div>}
    </FixedSizeList>
  );
}
// How many DOM nodes are rendered for 10,000 items?
```

<details>
<summary>Show Output & Explanation</summary>

```
~15-20 DOM nodes (only visible rows + a small overscan buffer)
```

`react-window` (virtualization/windowing) renders only the items **visible in the viewport** plus a small overscan buffer. With `height=500` and `itemSize=35`, only ~14-15 items are visible. Instead of 10,000 DOM nodes (catastrophic performance), you get ~20. As the user scrolls, new items render and off-screen items unmount.

</details>

---

### Q9. `will-change` hint to the GPU

```css
.animated {
  transition: transform 0.3s;
}
.animated-optimized {
  transition: transform 0.3s;
  will-change: transform;
}
```

```js
// Which version produces smoother animation?
```

<details>
<summary>Show Output & Explanation</summary>

```
.animated-optimized — because will-change promotes the element to its own GPU layer upfront
```

`will-change: transform` tells the browser to promote the element to a **GPU compositing layer** before the animation starts. Without it, the browser promotes the layer at animation start (causing an initial jank). Use `will-change` sparingly — overuse wastes GPU memory. Remove it when the animation is done.

</details>

---

### Q10. Code splitting — what does `React.lazy` actually do?

```jsx
const Dashboard = React.lazy(() => import('./Dashboard'));

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <Dashboard />
    </Suspense>
  );
}
// When is Dashboard.js downloaded?
```

<details>
<summary>Show Output & Explanation</summary>

```
Dashboard.js is downloaded when the <Dashboard> component first renders — not at app startup
```

`React.lazy(() => import('./Dashboard'))` creates a **dynamic import** — the chunk is only downloaded when `<Dashboard>` is first rendered. The bundler (Webpack/Vite) splits it into a separate chunk. `<Suspense>` shows `<Spinner />` while loading. This reduces initial bundle size and improves TTI.

</details>

---

### Q11. Memory leak — event listener not removed

```js
function setup() {
  const largeData = new Array(100000).fill('data');

  const handler = () => {
    console.log(largeData.length);
  };

  window.addEventListener('resize', handler);
  // setup() returns — is largeData garbage collected?
}

setup();
```

<details>
<summary>Show Output & Explanation</summary>

```
No — largeData is NOT garbage collected
```

The `handler` function **closes over** `largeData`. The event listener keeps `handler` alive (it's referenced by the window). `handler` keeps `largeData` alive through the closure. Even though `setup()` returned, `largeData` (100,000 items) stays in memory. Fix: `window.removeEventListener('resize', handler)` when no longer needed.

</details>

---

### Q12. `setInterval` memory leak in React

```jsx
function Counter() {
  const [count, setCount] = React.useState(0);

  React.useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + 1);
    }, 1000);
    // No cleanup!
  }, []);

  return <div>{count}</div>;
}
// What happens when the component unmounts?
```

<details>
<summary>Show Output & Explanation</summary>

```
setInterval continues running — memory leak + state update on unmounted component warning
```

Without a cleanup function returning `clearInterval(id)`, the interval **keeps running after unmount**. Each tick tries to call `setCount` on a dismounted component — React logs a warning. The interval also holds a reference to the closure, preventing garbage collection. Always return a cleanup: `return () => clearInterval(id)`.

</details>

---

### Q13. `requestAnimationFrame` vs `setTimeout` for animations

```js
// Option A:
let pos = 0;
function animate() {
  pos += 1;
  box.style.left = pos + 'px';
  setTimeout(animate, 16); // ~60fps
}
animate();

// Option B:
let pos = 0;
function animate() {
  pos += 1;
  box.style.left = pos + 'px';
  requestAnimationFrame(animate);
}
requestAnimationFrame(animate);

// Which is better for animations and why?
```

<details>
<summary>Show Output & Explanation</summary>

```
Option B (requestAnimationFrame) is better
```

`requestAnimationFrame` fires **in sync with the browser's repaint cycle** (typically 60fps). It automatically pauses when the tab is hidden (saves battery). `setTimeout(fn, 16)` is imprecise — it can fire slightly off-schedule, causing jank. `rAF` always fires at the optimal time for smooth animations.

</details>

---

### Q14. Tree shaking — what gets included in the bundle?

```js
// utils.js
export const add = (a, b) => a + b;
export const multiply = (a, b) => a * b;
export const unused = () => 'never imported';

// app.js
import { add } from './utils';
console.log(add(2, 3));
```

<details>
<summary>Show Output & Explanation</summary>

```
Only `add` is included in the production bundle.
`multiply` and `unused` are removed by tree shaking.
```

Bundlers (Webpack, Rollup, Vite) statically analyze ES Module `import/export` statements. Since `multiply` and `unused` are never imported, they're **dead code** and removed. Tree shaking only works with ES Modules (not CommonJS `require`). Set `"sideEffects": false` in `package.json` to help the bundler.

</details>

---

### Q15. `LCP` — what is the Largest Contentful Paint?

```html
<body>
  <nav>...</nav>
  <h1>Welcome to our site!</h1>
  <img id="hero" src="large-banner.jpg" loading="lazy" />
  <p>Content...</p>
</body>
<!-- What element most likely determines LCP? -->
```

<details>
<summary>Show Output & Explanation</summary>

```
The <img id="hero"> — if it's large enough, it determines LCP
But loading="lazy" can DELAY LCP!
```

LCP measures when the **largest visible content element** finishes rendering. A large hero image typically determines LCP. However, `loading="lazy"` defers image loading — the browser won't start fetching the image until it's near the viewport, which can **significantly worsen LCP**. Hero images above the fold should never use `loading="lazy"`. Use `<link rel="preload">` instead.

</details>

---

### Q16. `CLS` — what causes layout shift?

```html
<!-- Version A — no dimensions -->
<img src="product.jpg" alt="Product">

<!-- Version B — dimensions specified -->
<img src="product.jpg" alt="Product" width="800" height="600">
```

<details>
<summary>Show Output & Explanation</summary>

```
Version A causes CLS — the image loads and pushes content down
Version B does NOT cause CLS — browser reserves space upfront
```

Without `width` and `height`, the browser doesn't know the image dimensions before it loads. When the image loads, it pushes surrounding content down — causing a **Cumulative Layout Shift**. Specifying dimensions lets the browser reserve the exact space before the image loads, preventing shift. CLS target: < 0.1.

</details>

---

### Q17. `performance.now()` vs `Date.now()`

```js
const t1 = performance.now();
// ... some work ...
const t2 = performance.now();
console.log('elapsed:', t2 - t1, 'ms');

const d1 = Date.now();
// ... same work ...
const d2 = Date.now();
console.log('elapsed:', d2 - d1, 'ms');
```

<details>
<summary>Show Output & Explanation</summary>

```
performance.now(): sub-millisecond precision (e.g., 0.123 ms)
Date.now(): millisecond precision (e.g., 0 ms or 1 ms — too coarse for short operations)
```

`performance.now()` returns a **high-resolution timestamp** (sub-millisecond, relative to page load). `Date.now()` returns milliseconds since epoch — too coarse for profiling fast operations. Always use `performance.now()` for benchmarking code performance.

</details>

---

### Q18. Lighthouse score — what does TTI measure?

```
Page A: All JS loads upfront (500KB bundle). DOM ready at 1s but JS takes 4s to parse/execute.
Page B: Code-split (100KB initial + lazy). DOM ready at 0.5s, JS done at 1.5s.

Which page has better TTI?
```

<details>
<summary>Show Output & Explanation</summary>

```
Page B — TTI of ~1.5s vs Page A's ~4s
```

**TTI (Time to Interactive)** measures when the page is fully interactive — main thread is idle, JS is parsed and executed, event handlers are attached. Page A's large bundle blocks the main thread for 4s even though the DOM is ready. Page B's code splitting reduces initial JS, making it interactive much sooner.

</details>

---

### Q19. `React.memo` with object props

```jsx
const Child = React.memo(({ style }) => {
  console.log('Child rendered');
  return <div style={style}>content</div>;
});

function Parent() {
  const [count, setCount] = React.useState(0);
  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>+</button>
      <Child style={{ color: 'red' }} />
    </>
  );
}
// Does React.memo prevent Child from re-rendering?
```

<details>
<summary>Show Output & Explanation</summary>

```
No — Child re-renders on every Parent render
```

`{ color: 'red' }` is an **inline object literal** — a new object reference is created on every render. `React.memo` uses `Object.is()` for prop comparison: `{} !== {}`. The memo sees a "changed" prop every time. Fix: define the style object outside the component or wrap it in `useMemo`.

</details>

---

### Q20. Gzip/Brotli compression — what's the impact?

```
Bundle size before compression: 500KB
After Gzip:   ~150KB (70% reduction)
After Brotli: ~130KB (74% reduction)

Which takes priority for download speed?
```

<details>
<summary>Show Output & Explanation</summary>

```
Compressed size matters for network transfer.
Browser decompresses before parsing — decompression is very fast.
```

The browser downloads the compressed version and decompresses it locally — decompression is near-instant. The 70-80% size reduction dramatically improves load time over slow connections. Brotli compresses ~15-20% better than Gzip. Enable compression on your CDN/server — it's one of the highest-ROI performance wins with no code changes.

</details>

---

### Q21. React context causing unnecessary re-renders

```jsx
const ThemeContext = React.createContext();

function App() {
  const [theme, setTheme] = React.useState('light');
  const [user, setUser] = React.useState('Alice');

  return (
    <ThemeContext.Provider value={{ theme, user }}>
      <ThemeConsumer />
      <UserConsumer />
    </ThemeContext.Provider>
  );
}

// User changes only — do both ThemeConsumer and UserConsumer re-render?
```

<details>
<summary>Show Output & Explanation</summary>

```
Yes — BOTH re-render, even though ThemeConsumer doesn't use `user`
```

All consumers of a context **re-render whenever the context value changes**. Since `user` and `theme` are in the same object, changing `user` creates a new object reference, causing all consumers to re-render. Fix: split into separate contexts (`ThemeContext`, `UserContext`) or use `useMemo` to stabilize the context value.

</details>

---

### Q22. `IntersectionObserver` for lazy loading images

```js
const observer = new IntersectionObserver(entries => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      const img = entry.target;
      img.src = img.dataset.src; // load the real image
      observer.unobserve(img);
    }
  });
});

document.querySelectorAll('img[data-src]').forEach(img => observer.observe(img));
```

<details>
<summary>Show Output & Explanation</summary>

```
Images only download when they enter the viewport
`unobserve` stops watching after the image loads (no repeated triggers)
```

This is a common interview question. `data-src` stores the real URL without triggering a download. When the image scrolls into view, `isIntersecting` becomes `true`, the real `src` is set (triggering download), and `unobserve` prevents re-firing. More performant than `scroll` event + `getBoundingClientRect`.

</details>

---

### Q23. Memory leak with detached DOM nodes

```js
let detachedNodes = [];

function createLeak() {
  const div = document.createElement('div');
  div.textContent = 'I will leak';
  document.body.appendChild(div);
  document.body.removeChild(div);

  detachedNodes.push(div); // reference kept!
}

// Called 1000 times...
for (let i = 0; i < 1000; i++) createLeak();
```

<details>
<summary>Show Output & Explanation</summary>

```
1000 detached DOM nodes in memory — never garbage collected
```

Removing a node from the DOM doesn't free its memory if a **JavaScript reference exists** to it (`detachedNodes` array). These "detached nodes" appear in Chrome DevTools Memory heap snapshots — a common source of real-world memory leaks. Fix: don't store references to removed nodes, or set `detachedNodes = []` when done.

</details>

---

### Q24. `preload` vs `prefetch` — what's the difference?

```html
<!-- Option A: -->
<link rel="preload" href="hero.jpg" as="image">

<!-- Option B: -->
<link rel="prefetch" href="dashboard-chunk.js">
```

<details>
<summary>Show Output & Explanation</summary>

```
preload  — fetches the resource IMMEDIATELY (high priority, current page needs it)
prefetch — fetches when the browser is idle (low priority, future navigation might need it)
```

`preload` tells the browser: "I need this resource NOW on the current page — fetch it at high priority." Used for critical above-the-fold resources (hero images, fonts, critical CSS). `prefetch` tells the browser: "The user might navigate here next — fetch this when idle." Used for next-page chunks in code splitting. Using `preload` for everything defeats the purpose.

</details>

---

---

**Q25. What is the output?**

```js
// Approach 1 — BAD
function animateBad() {
  let pos = 0;
  setInterval(() => {
    pos += 5;
    document.getElementById('box').style.left = pos + 'px';
  }, 16);
}

// Approach 2 — GOOD
function animateGood() {
  let pos = 0;
  function frame() {
    pos += 5;
    document.getElementById('box').style.left = pos + 'px';
    requestAnimationFrame(frame);
  }
  requestAnimationFrame(frame);
}

console.log(typeof requestAnimationFrame);
// Which approach is better for smooth animation?
```

<details>
<summary>Show Output & Explanation</summary>

```
function
```

**Explanation:**  
`requestAnimationFrame` is a browser-provided function (typeof = `'function'`). `setInterval(fn, 16)` fires every 16ms regardless of whether the browser is ready to paint, causing dropped frames or battery drain in background tabs. `rAF` automatically pauses in hidden tabs, syncs with the display refresh rate, and batches with the browser's rendering pipeline for smooth 60fps animation.

> **Common Mistake:** Using `setInterval` or `setTimeout` for animations — they're timer-based and not synchronized with the browser's paint cycle, causing jank and unnecessary CPU usage when the tab is hidden.

</details>

---

**Q26. What is the output?**

```js
// Without Web Worker — blocks main thread
function heavyCalc(n) {
  let result = 0;
  for (let i = 0; i < n; i++) result += Math.sqrt(i);
  return result;
}

console.log('before calc');
const result = heavyCalc(100_000_000); // blocks UI for ~300ms
console.log('after calc:', result.toFixed(2));
console.log('is UI responsive during calc?', false);

// With Web Worker — non-blocking
const worker = new Worker('heavy.js'); // runs heavyCalc in background
worker.postMessage(100_000_000);
worker.onmessage = e => console.log('worker result:', e.data.toFixed(2));
console.log('is UI responsive during worker calc?', true);
```

<details>
<summary>Show Output & Explanation</summary>

```
before calc
after calc: 666666661.84
is UI responsive during calc? false
is UI responsive during worker calc? true
```

**Explanation:**  
JavaScript is single-threaded — heavy sync computation blocks the event loop, freezing the UI. Web Workers run in a separate OS thread, so heavy computation doesn't block user interactions. `postMessage` transfers data to the worker; results come back via `onmessage`. Trade-off: Workers have overhead (thread spawn, message serialization) — use for tasks >100ms.

> **Common Mistake:** Running large data processing (image manipulation, sorting millions of items, complex math) on the main thread. Move anything that takes >50ms to a Web Worker to keep the main thread free for user input.

</details>

---

**Q27. What is the output?**

```js
// Measuring paint performance
const box = document.getElementById('box');

// Before optimization
console.time('without will-change');
box.style.transform = 'translateX(0)';
// ... 1000 animation frames
console.timeEnd('without will-change');

// After optimization
box.style.willChange = 'transform';
console.time('with will-change');
box.style.transform = 'translateX(0)';
// ... 1000 animation frames
console.timeEnd('with will-change');

console.log('will-change promotes element to:', 'compositor layer');
console.log('skips:', 'layout and paint phases');
console.log('directly runs:', 'compositing');
```

<details>
<summary>Show Output & Explanation</summary>

```
without will-change: ~8ms
with will-change: ~2ms
will-change promotes element to: compositor layer
skips: layout and paint phases
directly runs: compositing
```

**Explanation:**  
`will-change: transform` (or `opacity`) tells the browser to promote the element to its own GPU compositor layer ahead of time. Animating `transform`/`opacity` on a composited layer skips layout and paint — the GPU handles it directly. This is why CSS transforms outperform `top`/`left` animations. Apply `will-change` sparingly — each composited layer uses GPU memory.

> **Common Mistake:** Applying `will-change: transform` to everything "for performance" — overuse causes excessive GPU memory consumption and can degrade performance. Only use it for elements you KNOW will animate.

</details>

---

*Chapter 11 of JavaScript Interview Prep Series*
