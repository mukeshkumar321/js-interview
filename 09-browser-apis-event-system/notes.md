# Chapter 9: Browser APIs & Event System

> **Target:** Frontend Developer (4+ years) | Interview-focused, practical explanations

---

## Table of Contents

- [DOM & Browser Fundamentals](#dom--browser-fundamentals)
- [Event System](#event-system)
- [Event Listeners](#event-listeners)
- [Modern Browser APIs](#modern-browser-apis)
- [Browser Lifecycle & Performance](#browser-lifecycle--performance)

---

## DOM & Browser Fundamentals

<details>
<summary><strong>1. What are Browser APIs and how do they differ from JavaScript language features?</strong></summary>

**JavaScript** is the language spec (ECMAScript) — it defines syntax, data types, closures, Promises, etc. It has no concept of a browser.

**Browser APIs** are provided by the browser (not the JS spec) — they let JS interact with the web platform:

| Browser API | What it does |
|-------------|-------------|
| DOM API | Manipulate HTML/CSS |
| Fetch API | HTTP requests |
| Web Storage | localStorage, sessionStorage |
| setTimeout/setInterval | Timers |
| Geolocation | User location |
| WebSockets | Real-time connection |
| Intersection Observer | Visibility detection |
| Canvas / WebGL | Graphics rendering |

```js
// JS language feature — works everywhere (browser, Node, Deno)
const arr = [1, 2, 3].map(x => x * 2);

// Browser API — only in browser environments
document.getElementById('app'); // DOM API
fetch('/api/data');              // Fetch API
localStorage.setItem('key','v'); // Web Storage API
```

</details>

---

<details>
<summary><strong>2. What is the DOM and how does the browser create it?</strong></summary>

The **DOM (Document Object Model)** is a tree-structured in-memory representation of an HTML document. Each HTML element becomes a node in the tree.

**Browser's rendering steps:**
1. Parse HTML → build **DOM tree**
2. Parse CSS → build **CSSOM tree**
3. Combine → **Render Tree** (only visible nodes)
4. **Layout** — calculate positions and sizes
5. **Paint** — draw pixels

```html
<div id="app">
  <h1>Hello</h1>
  <p>World</p>
</div>
```
```
Document
  └── html
        ├── head
        └── body
              └── div#app
                    ├── h1 ("Hello")
                    └── p ("World")
```

JS interacts with the DOM through the DOM API — `document.querySelector`, `element.appendChild`, etc. Every DOM manipulation can trigger re-layout or repaint.

</details>

---

<details>
<summary><strong>3. What is the difference between the DOM and the Virtual DOM?</strong></summary>

| | Real DOM | Virtual DOM |
|--|---------|------------|
| **What** | Browser's live tree | In-memory JS object tree |
| **Updates** | Immediate, expensive | Batched, diffed, then applied |
| **Reflow/Repaint** | Every change | Only actual diffs |
| **Used by** | Vanilla JS | React, Vue |

**Virtual DOM workflow (React):**
1. State changes → new Virtual DOM tree
2. Diff old vs new Virtual DOM (reconciliation)
3. Apply only the minimal real DOM changes (commit)

```js
// Real DOM — every line triggers reflow
div.style.width  = '100px'; // reflow
div.style.height = '200px'; // reflow
div.style.color  = 'red';   // repaint

// Better — batch with cssText or class
div.style.cssText = 'width:100px; height:200px; color:red'; // one reflow
div.className = 'active'; // one reflow
```

> **Interview Note:** Virtual DOM's real advantage isn't speed — real DOM updates can be fast. The advantage is the **declarative programming model** — you describe what the UI should look like, and React figures out the minimal changes.

</details>

---

<details>
<summary><strong>4. How do you select DOM elements and when to use different methods?</strong></summary>

```js
// By ID — fastest, returns single element or null
document.getElementById('app');

// CSS selector — flexible, returns first match
document.querySelector('.btn-primary');
document.querySelector('#app .list > li:first-child');

// CSS selector — returns static NodeList
document.querySelectorAll('.items'); // snapshot, not live

// By class — returns live HTMLCollection
document.getElementsByClassName('active');

// By tag — live HTMLCollection
document.getElementsByTagName('div');
```

**Live vs Static:**
```js
const live   = document.getElementsByClassName('item'); // live — updates automatically
const static_ = document.querySelectorAll('.item');     // static snapshot

document.body.innerHTML += '<div class="item"></div>';
live.length;    // automatically updated
static_.length; // still old length
```

**When to use:**
- `getElementById` — fastest for single element by ID
- `querySelector` / `querySelectorAll` — flexible CSS selectors (most common)
- `getElementsBy*` — when you need a live collection

</details>

---

## Event System

<details>
<summary><strong>5 & 6. What is Event Propagation? Bubbling vs Capturing?</strong></summary>

When an event fires on an element, it travels in three phases:

```
Document
  └── html
        └── body
              └── div (capturing goes down ↓, bubbling goes up ↑)
                    └── button ← event target
```

**Phase 1 — Capturing** (top → target): Event travels down from `document` to the target. Rarely used.  
**Phase 2 — Target**: Event fires on the element that was clicked.  
**Phase 3 — Bubbling** (target → top): Event travels back up through ancestors. This is the default.

```js
document.querySelector('div').addEventListener('click', () => {
  console.log('div bubbling'); // fires when button inside is clicked
});

document.querySelector('button').addEventListener('click', () => {
  console.log('button'); // fires first
});

// Click button: 'button' → 'div bubbling'

// Capturing — fires during phase 1 (before target)
document.querySelector('div').addEventListener('click', () => {
  console.log('div capturing'); // fires before 'button'
}, { capture: true }); // or true as 3rd arg
```

</details>

---

<details>
<summary><strong>7 & 8. What is Event Delegation and why is it useful for performance?</strong></summary>

**Event Delegation** — attach a single event listener to a **parent** instead of individual listeners on each child. Uses bubbling to catch events from children.

```js
// ❌ Without delegation — 1000 listeners for 1000 items
document.querySelectorAll('.item').forEach(item => {
  item.addEventListener('click', handleClick);
});

// ✅ With delegation — 1 listener handles all items
document.querySelector('.list').addEventListener('click', e => {
  if (e.target.matches('.item')) {
    handleClick(e.target);
  }
});
```

**Benefits:**
- **Memory** — one listener vs hundreds/thousands
- **Dynamic elements** — new items added to DOM automatically handled (no re-registration)
- **Simpler code** — centralized event logic

**Real-world use:** Tables with hundreds of rows, infinite scroll lists, dynamically generated content.

</details>

---

<details>
<summary><strong>9. What is the difference between <code>event.target</code> and <code>event.currentTarget</code>?</strong></summary>

| | `event.target` | `event.currentTarget` |
|--|---------------|----------------------|
| **What** | Element that triggered the event | Element the listener is attached to |
| **Changes** | No — always the origin | Yes — changes as event bubbles |

```html
<div id="parent">
  <button id="child">Click me</button>
</div>
```

```js
document.getElementById('parent').addEventListener('click', e => {
  console.log(e.target);        // <button id="child"> — what was clicked
  console.log(e.currentTarget); // <div id="parent"> — where listener is
});
```

In event delegation, `e.target` tells you exactly which element triggered the event, while `e.currentTarget` is always the delegating parent.

</details>

---

<details>
<summary><strong>10. What is the difference between <code>preventDefault()</code> and <code>stopPropagation()</code>?</strong></summary>

| Method | What it does |
|--------|-------------|
| `preventDefault()` | Stops the browser's default action for the event |
| `stopPropagation()` | Stops event from bubbling/capturing to other listeners |
| `stopImmediatePropagation()` | Stops event AND prevents other listeners on the same element |

```js
// preventDefault — stop default behavior
link.addEventListener('click', e => {
  e.preventDefault(); // don't navigate to href
  handleNavigation(e.target.href); // custom SPA navigation
});

form.addEventListener('submit', e => {
  e.preventDefault(); // don't refresh page
  handleFormData(e);
});

// stopPropagation — stop event from reaching parent listeners
child.addEventListener('click', e => {
  e.stopPropagation(); // parent's click listener won't fire
});

// Both can be used together
btn.addEventListener('click', e => {
  e.preventDefault();   // no default action
  e.stopPropagation();  // no bubbling
});
```

</details>

---

<details>
<summary><strong>11. In what order are event handlers executed during event propagation?</strong></summary>

```js
// Given: document > div > button
document.addEventListener('click', () => console.log('doc bubble'), false);
document.addEventListener('click', () => console.log('doc capture'), true);

div.addEventListener('click', () => console.log('div bubble'), false);
div.addEventListener('click', () => console.log('div capture'), true);

button.addEventListener('click', () => console.log('button'));

// Click button — order:
// 1. doc capture  (top, capturing phase)
// 2. div capture  (closer to target, capturing)
// 3. button       (target phase — handlers fire in registration order)
// 4. div bubble   (target → root, bubbling)
// 5. doc bubble   (root, bubbling)
```

> **Interview Note:** At the **target element** itself, capture and bubble listeners fire in **registration order**, not phase order. Phase distinction only matters for ancestor elements.

</details>

---

## Event Listeners

<details>
<summary><strong>12 & 13. How does <code>addEventListener()</code> work? What are capture, once, and passive?</strong></summary>

```js
element.addEventListener(type, handler, options);
// options: { capture, once, passive, signal }

// capture — run during capture phase (before target)
el.addEventListener('click', fn, { capture: true });

// once — auto-remove after first invocation
el.addEventListener('click', fn, { once: true });
// Useful for: first-time modals, one-time setup

// passive — promise the handler won't call preventDefault()
// Browser can skip waiting → smoother scroll performance
window.addEventListener('scroll', fn, { passive: true });
window.addEventListener('touchstart', fn, { passive: true });

// signal — remove via AbortController
const controller = new AbortController();
el.addEventListener('click', fn, { signal: controller.signal });
controller.abort(); // removes the listener
```

**Why `passive` matters:** For scroll/touch events, the browser normally waits for your handler to complete before scrolling (in case you call `preventDefault()`). `passive: true` tells the browser to scroll immediately — crucial for smooth 60fps scrolling on mobile.

</details>

---

<details>
<summary><strong>14 & 15. How do you remove a listener? Common memory leak issues?</strong></summary>

```js
// Must pass the SAME function reference to remove
function handler() { console.log('clicked'); }
el.addEventListener('click', handler);
el.removeEventListener('click', handler); // ✅

// ❌ This doesn't work — different function reference each time
el.addEventListener('click', () => console.log('clicked'));
el.removeEventListener('click', () => console.log('clicked')); // no-op!
```

**Common memory leaks:**
```js
// ❌ Leak 1 — listener on global object keeps component alive
class Component {
  init() {
    window.addEventListener('resize', this.handleResize); // never removed!
  }
  destroy() {
    window.removeEventListener('resize', this.handleResize); // ✅ fix
  }
}

// ❌ Leak 2 — DOM element removed from DOM but listener holds reference
const btn = document.getElementById('btn');
btn.addEventListener('click', () => { /* closes over large data */ });
btn.remove(); // element removed but listener (and closure) still in memory
// Fix: removeEventListener before remove, or use { once: true }

// ✅ Modern fix — AbortController
const ctrl = new AbortController();
window.addEventListener('resize', fn, { signal: ctrl.signal });
// Later:
ctrl.abort(); // removes all listeners registered with this signal
```

</details>

---

## Modern Browser APIs

<details>
<summary><strong>16. What is the Intersection Observer API and when would you use it?</strong></summary>

`IntersectionObserver` asynchronously detects when an element enters or leaves the viewport (or another element). Much more performant than listening to `scroll` events.

```js
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      // Element is visible in viewport
      entry.target.classList.add('visible');
      observer.unobserve(entry.target); // stop watching if only needed once
    }
  });
}, {
  threshold: 0.1,    // fire when 10% visible
  rootMargin: '0px 0px -50px 0px' // shrink effective viewport
});

observer.observe(document.querySelector('.lazy-image'));
```

**Use cases:**
- **Lazy loading** images/components — load when near viewport
- **Infinite scroll** — trigger next page when bottom sentinel is visible
- **Scroll-triggered animations** — add class when element enters view
- **Analytics** — track which sections are actually viewed
- **Ads** — measure viewability

</details>

---

<details>
<summary><strong>17. What is the Mutation Observer API and when would you use it?</strong></summary>

`MutationObserver` watches for changes in the DOM tree — attribute changes, child additions/removals, text content changes.

```js
const observer = new MutationObserver(mutations => {
  mutations.forEach(mutation => {
    if (mutation.type === 'childList') {
      console.log('Children changed:', mutation.addedNodes, mutation.removedNodes);
    }
    if (mutation.type === 'attributes') {
      console.log('Attribute changed:', mutation.attributeName);
    }
  });
});

observer.observe(targetElement, {
  childList: true,   // watch child additions/removals
  attributes: true,  // watch attribute changes
  subtree: true,     // watch all descendants
  characterData: true// watch text content
});

observer.disconnect(); // stop observing
```

**Use cases:**
- Third-party script integration (watch when they inject elements)
- Rich text editor tracking content changes
- Detecting when Angular/Vue/React updates the DOM
- Accessibility tools monitoring dynamic content

</details>

---

<details>
<summary><strong>18. What is the Resize Observer API and when would you use it?</strong></summary>

`ResizeObserver` fires when an element's size changes — more precise than `window.resize` because it watches individual elements.

```js
const observer = new ResizeObserver(entries => {
  entries.forEach(entry => {
    const { width, height } = entry.contentRect;
    console.log(`Element is now ${width}px × ${height}px`);

    // Responsive component logic
    if (width < 400) element.classList.add('compact');
    else element.classList.remove('compact');
  });
});

observer.observe(document.querySelector('.chart'));
observer.unobserve(element);
observer.disconnect();
```

**Use cases:**
- **Container queries** (before CSS container queries landed)
- **Charts/graphs** — redraw when container changes size
- **Responsive components** — adapt layout based on own size, not viewport
- **Virtualized lists** — recalculate row heights on resize

</details>

---

## Browser Lifecycle & Performance

<details>
<summary><strong>19. What is the difference between <code>DOMContentLoaded</code> and <code>load</code> events?</strong></summary>

| Event | Fires when | Waits for |
|-------|-----------|-----------|
| `DOMContentLoaded` | HTML parsed, DOM built | Not images/stylesheets |
| `load` | Everything loaded | Images, CSS, scripts, iframes |

```js
// DOMContentLoaded — DOM is ready, safe to query elements
document.addEventListener('DOMContentLoaded', () => {
  document.querySelector('#app').classList.add('ready'); // safe
  initializeApp(); // DOM available, but images may not be loaded
});

// load — all resources fully loaded (images, fonts, etc.)
window.addEventListener('load', () => {
  reportPageLoadTime(); // accurate total load time
  initChartsWithImages(); // safe — images are loaded
});
```

**Best practice:** Use `DOMContentLoaded` for DOM manipulation (faster). Use `load` only when you specifically need images/external resources to be ready.

> **Interview Note:** Scripts with `defer` attribute run after `DOMContentLoaded`. Scripts with `async` run as soon as they download. `defer` is the modern preferred approach for non-critical scripts.

</details>

---

<details>
<summary><strong>20. What are common DOM performance bottlenecks and how can they be avoided?</strong></summary>

**1. Layout thrashing (forced synchronous layout)**
```js
// ❌ Read → Write → Read → Write causes multiple layouts
const h1 = el.offsetHeight; // read — forces layout
el.style.height = h1 + 'px'; // write
const h2 = el.offsetHeight; // read — forces layout AGAIN

// ✅ Batch reads then writes
const h1 = el.offsetHeight; // read
const h2 = el2.offsetHeight; // read
el.style.height = h1 + 'px';  // write
el2.style.height = h2 + 'px'; // write (single reflow)
```

**2. Frequent small DOM updates**
```js
// ❌ Many reflows
items.forEach(item => {
  const li = document.createElement('li');
  li.textContent = item;
  list.appendChild(li); // reflow per item!
});

// ✅ DocumentFragment — one reflow
const frag = document.createDocumentFragment();
items.forEach(item => {
  const li = document.createElement('li');
  li.textContent = item;
  frag.appendChild(li);
});
list.appendChild(frag); // single reflow
```

**3. DOM queries in loops**
```js
// ❌ Queries DOM every iteration
for (let i = 0; i < 1000; i++) {
  document.querySelector('.list').style.color = 'red'; // re-queries each time
}

// ✅ Cache the reference
const list = document.querySelector('.list');
for (let i = 0; i < 1000; i++) {
  list.style.color = 'red';
}
```

</details>

---

## Quick Reference Cheat Sheet

```
Event propagation order:
  Capture (top → target) → Target → Bubble (target → top)

event.target        → what was clicked
event.currentTarget → where the listener is attached

preventDefault()     → stop browser default (form submit, link nav)
stopPropagation()    → stop bubbling/capturing
stopImmediatePropagation() → stop + other listeners on same element

addEventListener options:
  capture: true  → run during capture phase
  once: true     → auto-remove after first fire
  passive: true  → can't call preventDefault → browser scrolls immediately
  signal         → AbortController integration

Observer APIs:
  IntersectionObserver → visibility in viewport (lazy load, infinite scroll)
  MutationObserver     → DOM tree changes (attribute, children)
  ResizeObserver       → element size changes (responsive components)

DOMContentLoaded → HTML parsed, DOM ready
load             → all resources (images, CSS) fully loaded
```

---

*Chapter 9 of JavaScript Interview Prep Series*
