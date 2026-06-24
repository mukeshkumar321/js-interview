# Chapter 11: Performance Optimization

> **Target:** Frontend Developer (4+ years) | Interview-focused, practical explanations

---

## Table of Contents

- [Performance Fundamentals](#performance-fundamentals)
- [Browser Rendering Performance](#browser-rendering-performance)
- [Event & Rendering Optimization](#event--rendering-optimization)
- [Bundle & Network Optimization](#bundle--network-optimization)
- [Memory & React Performance](#memory--react-performance)

---

## Performance Fundamentals

<details>
<summary><strong>1. How would you identify performance bottlenecks in a web application?</strong></summary>

**Step 1 — Measure first, optimize second.** Never guess.

```
1. Open Chrome DevTools → Performance tab
2. Record while reproducing the slow scenario
3. Look for:
   - Long tasks (>50ms) — blocks main thread
   - Layout/paint (purple/green blocks) — expensive DOM work
   - JavaScript execution (yellow) — heavy computation
   - Network waterfalls — slow/blocking resources
```

**Key questions to ask:**
- Is it slow on load or during interaction?
- Is it a network issue or compute issue?
- Does it reproduce on slow CPU throttling?
- What's the Lighthouse score? Which metric fails?

**Profile → identify hotspot → fix → measure again.** Repeat.

</details>

---

<details>
<summary><strong>2. What tools do you use to measure and debug performance issues?</strong></summary>

| Tool | What it measures |
|------|-----------------|
| **Chrome DevTools Performance** | Runtime profiling, call stack, paint/layout |
| **Lighthouse** | Load performance score (LCP, FID, CLS) |
| **WebPageTest** | Real network conditions, waterfall |
| **Chrome DevTools Network** | Request timing, payload sizes, caching |
| **Chrome DevTools Memory** | Heap snapshots, memory leaks |
| **React DevTools Profiler** | Component render times, unnecessary re-renders |
| `performance.now()` | High-resolution timing in code |
| `performance.mark/measure` | Custom timing for specific flows |

```js
// Manual timing
performance.mark('start');
expensiveOperation();
performance.mark('end');
performance.measure('op-time', 'start', 'end');
const [entry] = performance.getEntriesByName('op-time');
console.log(`${entry.duration.toFixed(2)}ms`);
```

</details>

---

<details>
<summary><strong>3. What is the difference between Runtime Performance and Load Performance?</strong></summary>

| | Load Performance | Runtime Performance |
|--|-----------------|---------------------|
| **When** | Page loading | After load, during interaction |
| **Metrics** | LCP, FCP, TTI, TTFB | FPS, interaction latency, memory |
| **Tools** | Lighthouse, WebPageTest | Chrome DevTools Performance tab |
| **Fix** | Bundle size, caching, CDN | Algorithm, debounce, virtualization |

**Load:** How fast does the page become usable?  
**Runtime:** How smooth is it to use after it loads?

A page can load fast but feel janky (poor runtime performance) — e.g., unoptimized scroll handlers causing dropped frames.

</details>

---

<details>
<summary><strong>4. What are Core Web Vitals and why are they important?</strong></summary>

Core Web Vitals are Google's metrics for real-world user experience — they directly affect SEO rankings.

| Metric | Measures | Good threshold |
|--------|----------|----------------|
| **LCP** (Largest Contentful Paint) | Loading — time until main content visible | < 2.5s |
| **FID** (First Input Delay) / **INP** (Interaction to Next Paint) | Interactivity — time from input to response | < 100ms / < 200ms |
| **CLS** (Cumulative Layout Shift) | Visual stability — how much content shifts | < 0.1 |

```
LCP — optimize: hero image, server response time, render-blocking resources
INP — optimize: long tasks, heavy JS, event handler performance
CLS — optimize: always set image dimensions, avoid inserting content above fold
```

> **Interview Note:** FID was replaced by INP as a Core Web Vital in 2024. INP measures all interactions throughout the page lifetime, not just the first one.

</details>

---

<details>
<summary><strong>5. What metrics would you monitor to evaluate frontend performance?</strong></summary>

```
Load metrics:
  TTFB  — Time to First Byte (server response speed)
  FCP   — First Contentful Paint (when any content appears)
  LCP   — Largest Contentful Paint (main content visible)
  TTI   — Time to Interactive (JS loaded, handlers attached)
  TBT   — Total Blocking Time (sum of long task blocking time)

Runtime metrics:
  FPS        — frames per second (60fps = smooth)
  INP        — Interaction to Next Paint
  CLS        — layout shift score
  JS heap    — memory usage

Custom metrics:
  Time to first product visible
  Time to interactive checkout
  API response p50/p95/p99
```

</details>

---

## Browser Rendering Performance

<details>
<summary><strong>6. What is the Critical Rendering Path?</strong></summary>

The sequence of steps the browser takes before painting the first pixel:

```
HTML → DOM
CSS  → CSSOM
         ↓
    DOM + CSSOM → Render Tree (only visible nodes)
                        ↓
                    Layout (positions, sizes)
                        ↓
                    Paint (pixels)
                        ↓
                  Composite (GPU layers)
```

**Optimizing the CRP:**
- Minimize render-blocking resources (CSS in `<head>`, JS with `defer`/`async`)
- Inline critical CSS (above-the-fold styles)
- Preload key resources: `<link rel="preload" href="hero.jpg" as="image">`
- Reduce CSS size — browser must parse full CSSOM before rendering
- Serve HTML from cache / CDN — reduces TTFB

</details>

---

<details>
<summary><strong>7 & 8. What is the difference between Reflow and Repaint? What causes unnecessary ones?</strong></summary>

**Reflow (Layout)** — recalculates positions and sizes of elements. Expensive — affects the whole page.  
**Repaint** — redraws pixels without changing layout. Cheaper — only the affected area.

```
Reflow  → always triggers repaint
Repaint → does NOT trigger reflow
```

**Properties that trigger reflow:**
`width`, `height`, `margin`, `padding`, `border`, `font-size`, `position`, `display`, `top/left`, `offsetWidth/offsetHeight`

**Properties that only trigger repaint:**
`color`, `background-color`, `opacity`, `visibility`, `box-shadow`

**Properties that trigger neither (GPU composited):**
`transform`, `opacity` (with `will-change`)

```js
// ❌ Causes layout thrashing — alternating read/write
for (const el of elements) {
  const h = el.offsetHeight;     // read → forces layout
  el.style.height = h + 10 + 'px'; // write → invalidates layout
}

// ✅ Batch reads, then writes
const heights = elements.map(el => el.offsetHeight); // all reads
elements.forEach((el, i) => el.style.height = heights[i] + 10 + 'px'); // all writes
```

</details>

---

<details>
<summary><strong>9 & 10. How does frequent DOM manipulation impact performance? Techniques to optimize?</strong></summary>

Every DOM change can trigger reflow + repaint. Multiple changes in quick succession cascade into expensive layout recalculations.

```js
// ❌ Appending inside loop — N reflows
for (let i = 0; i < 1000; i++) {
  list.innerHTML += `<li>Item ${i}</li>`; // parses + reflows each time!
}

// ✅ Build string, set once
const html = Array.from({ length: 1000 }, (_, i) => `<li>Item ${i}</li>`).join('');
list.innerHTML = html; // one reflow

// ✅ DocumentFragment — detached DOM
const frag = document.createDocumentFragment();
for (let i = 0; i < 1000; i++) {
  const li = document.createElement('li');
  li.textContent = `Item ${i}`;
  frag.appendChild(li); // no reflow — frag is off-screen
}
list.appendChild(frag); // one reflow

// ✅ Use CSS classes instead of inline styles
el.classList.add('active'); // one reflow
// vs setting 5 style properties = potential 5 reflows

// ✅ Use transform/opacity for animations (GPU layer)
el.style.transform = 'translateX(100px)'; // no reflow
el.style.opacity = '0.5'; // no reflow
```

</details>

---

## Event & Rendering Optimization

<details>
<summary><strong>11 & 12. What is Debouncing vs Throttling? When to use which?</strong></summary>

Both limit how often a function executes — for different reasons.

**Debounce** — fires **after** a quiet period. Good when you only care about the final state.

```js
function debounce(fn, delay) {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  };
}

// Search input — wait for user to stop typing
const search = debounce(query => fetchResults(query), 300);
input.addEventListener('input', e => search(e.target.value));
// Types "react" → fires once, 300ms after last keystroke
```

**Throttle** — fires **at most once per interval**. Good when you want consistent updates during a continuous event.

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

// Scroll handler — update progress bar, but not 60x per second
const onScroll = throttle(() => updateProgressBar(), 100);
window.addEventListener('scroll', onScroll);
```

| | Debounce | Throttle |
|--|---------|---------|
| **Fires** | After quiet period | At fixed intervals |
| **Use for** | Search, resize complete, form autosave | Scroll, mousemove, game loop |

</details>

---

<details>
<summary><strong>14 & 15. How would you render large lists? What is Virtualization?</strong></summary>

Rendering 10,000 DOM nodes simultaneously destroys performance — too many layout/paint operations, huge memory usage.

**Virtualization (Windowing)** — render only the items currently **visible in the viewport**. As the user scrolls, render the next items and remove the ones that scrolled out.

```
Viewport shows items 10–30 (20 items)
DOM only contains those 20 items
Items 0–9 and 31+ are not in DOM
```

```jsx
// React — react-window (popular library)
import { FixedSizeList } from 'react-window';

function Row({ index, style }) {
  return <div style={style}>Row {index}</div>;
}

<FixedSizeList
  height={500}
  itemCount={10000}
  itemSize={35}
  width={300}
>
  {Row}
</FixedSizeList>

// Also: react-virtuoso (variable size), TanStack Virtual
```

**When to use virtualization:**
- Lists with 200+ items
- Tables with many rows
- Infinite scroll feeds
- Log viewers

</details>

---

## Bundle & Network Optimization

<details>
<summary><strong>16 & 17. What is Lazy Loading and Code Splitting?</strong></summary>

**Lazy Loading** — defer loading of non-critical resources until they're needed.

```html
<!-- Images — native lazy loading -->
<img src="product.jpg" loading="lazy" alt="Product">

<!-- Scripts — load only when needed -->
<script src="analytics.js" defer></script>
```

```jsx
// React — component lazy loading
const Dashboard = React.lazy(() => import('./Dashboard'));

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <Dashboard />
    </Suspense>
  );
}
```

**Code Splitting** — break the JS bundle into smaller chunks loaded on demand. Webpack/Vite do this automatically with dynamic imports.

```js
// Route-based splitting (React Router)
const Home    = lazy(() => import('./pages/Home'));
const Profile = lazy(() => import('./pages/Profile'));
const Admin   = lazy(() => import('./pages/Admin'));
// Each page = separate chunk, loaded only when navigated to
```

**Impact:** Instead of 500KB bundle, users download 100KB initial + lazy-load additional chunks. Dramatically improves TTI.

</details>

---

<details>
<summary><strong>18. What are Tree Shaking and Dead Code Elimination?</strong></summary>

**Tree Shaking** — bundler statically analyzes `import`/`export` statements and removes code that's never imported.

```js
// utils.js
export const used   = () => 'I am used';
export const unused = () => 'I am never imported'; // ← removed by tree shaking

// app.js
import { used } from './utils'; // only `used` is included in bundle
```

**Requirements for tree shaking:**
- ES Module syntax (`import`/`export`) — not CommonJS (`require`)
- `sideEffects: false` in `package.json` (tells bundler the module has no side effects)
- Production mode (disabled in development for faster builds)

```js
// ❌ CommonJS — not tree-shakeable
const { cloneDeep } = require('lodash'); // pulls in entire lodash!

// ✅ ES Module import — tree-shakeable
import { cloneDeep } from 'lodash-es'; // only cloneDeep included
```

</details>

---

<details>
<summary><strong>19 & 20. How does image optimization improve performance? Common bundle size reduction techniques?</strong></summary>

**Image optimization:**
```html
<!-- Modern formats -->
<picture>
  <source srcset="image.avif" type="image/avif">
  <source srcset="image.webp" type="image/webp">
  <img src="image.jpg" alt="..." loading="lazy" width="800" height="600">
</picture>

<!-- Always specify dimensions → prevents CLS -->
<!-- loading="lazy" → defer off-screen images -->
<!-- Use srcset for responsive images -->
```

Format sizes for same quality: `AVIF < WebP < JPEG < PNG`

**Bundle size reduction:**
```
1. Code splitting + lazy loading
2. Tree shaking (ES modules + production build)
3. Compression (Gzip/Brotli on server — 60-80% reduction)
4. Minification (removes whitespace, shortens names)
5. Replace heavy libs with lighter alternatives
   lodash → native ES methods
   moment.js → date-fns or dayjs
   axios → native fetch
6. Analyze bundle: webpack-bundle-analyzer / vite-plugin-visualizer
7. Dynamic imports for large third-party libs (chart.js, monaco)
8. Externalize rarely-changed deps → cache longer
```

</details>

---

## Memory & React Performance

<details>
<summary><strong>21 & 22. What are memory leaks? How do listeners, timers, and closures cause them?</strong></summary>

A **memory leak** is when memory is allocated but never freed — typically because a reference to the data is held longer than needed.

**Event listener leak:**
```js
// ❌ Registered but never removed
function setup() {
  const data = new Array(10000).fill('x'); // 10K items
  window.addEventListener('resize', () => {
    console.log(data.length); // closure keeps `data` alive!
  });
  // setup() returns — but data is kept alive by the listener
}

// ✅ Remove when no longer needed
const handler = () => { ... };
window.addEventListener('resize', handler);
return () => window.removeEventListener('resize', handler); // cleanup
```

**Timer leak:**
```js
// ❌ setInterval never cleared
function startPolling() {
  setInterval(() => fetchData(), 5000); // runs forever
}

// ✅ Store ID and clear
const id = setInterval(() => fetchData(), 5000);
// On component unmount:
clearInterval(id);
```

**React useEffect leak:**
```js
useEffect(() => {
  const id = setInterval(() => setCount(c => c + 1), 1000);
  return () => clearInterval(id); // cleanup function = no leak
}, []);
```

</details>

---

<details>
<summary><strong>23 & 24. Most important React performance optimizations? When to use memo, useMemo, useCallback?</strong></summary>

**React re-renders when:** state changes, parent re-renders, context changes.

**React.memo** — memoize a component. Only re-renders if props change.
```jsx
const ExpensiveList = React.memo(({ items }) => {
  return items.map(item => <Item key={item.id} {...item} />);
});
// Without memo: re-renders every time parent renders
// With memo: only re-renders when `items` reference changes
```

**useMemo** — memoize an expensive computed value.
```jsx
const sorted = useMemo(() =>
  [...items].sort((a, b) => a.price - b.price),
  [items] // only recompute when items changes
);
```

**useCallback** — memoize a function reference (prevent child re-renders when passing callbacks).
```jsx
const handleDelete = useCallback((id) => {
  setItems(prev => prev.filter(i => i.id !== id));
}, []); // stable reference — won't cause memo'd child to re-render
```

**When NOT to optimize:**
```
Don't add memo/useMemo/useCallback to every component.
Profile first — premature optimization is the root of all evil.
memo adds overhead too (comparing props). Only add when:
  - Component renders frequently
  - Component is expensive to render
  - Props rarely change
```

</details>

---

<details>
<summary><strong>25. How would you profile and debug React performance issues?</strong></summary>

**Step 1 — React DevTools Profiler:**
```
Install React DevTools extension →
Open DevTools → Profiler tab →
Record → interact with slow feature →
Stop → examine:
  - Which components rendered
  - How long each render took
  - Why they rendered (prop/state/context change)
```

**Step 2 — Identify why a component re-renders:**
```jsx
// Add to component temporarily
import { useRef, useEffect } from 'react';
function useWhyRerender(name, props) {
  const prev = useRef(props);
  useEffect(() => {
    const changed = Object.entries(props).filter(
      ([k, v]) => prev.current[k] !== v
    );
    if (changed.length) console.log(`${name} re-rendered:`, changed);
    prev.current = props;
  });
}
```

**Step 3 — Common culprits and fixes:**

| Culprit | Fix |
|---------|-----|
| New object/array in render | `useMemo` |
| New callback in render | `useCallback` |
| Child always re-renders | `React.memo` |
| Context causes all consumers to re-render | Split context, use selectors (Zustand/Redux) |
| List without keys | Add stable `key` props |
| Large bundle blocks hydration | Code split + lazy load |

</details>

---

## Quick Reference Cheat Sheet

```
Reflow triggers: width, height, margin, padding, offsetWidth, clientHeight
Repaint triggers: color, background, visibility (no layout change)
Compositor only: transform, opacity → use for animations

Debounce  → wait for silence → search inputs, autosave
Throttle  → fixed rate → scroll, resize, mousemove

React optimization hierarchy:
  1. Profile first (React Profiler)
  2. Fix unnecessary re-renders (React.memo)
  3. Stabilize references (useCallback, useMemo)
  4. Virtualize large lists (react-window)
  5. Code split heavy routes (React.lazy + Suspense)

Core Web Vitals targets:
  LCP < 2.5s   (loading)
  INP < 200ms  (interactivity)
  CLS < 0.1    (visual stability)

Memory leak checklist:
  □ removeEventListener in cleanup
  □ clearInterval/clearTimeout in cleanup
  □ AbortController for fetch in useEffect
  □ Unsubscribe from observables/stores
```

---

*Chapter 11 of JavaScript Interview Prep Series*
