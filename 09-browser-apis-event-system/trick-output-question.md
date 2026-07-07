## Chapter 9: Browser APIs & Event System — Trick Output Questions

> Self-evaluate first. Predict the output, then reveal the answer.

---

### Q1. Event bubbling order

```html
<div id="outer">
  <div id="inner">
    <button id="btn">Click me</button>
  </div>
</div>
```

```js
document.getElementById('outer').addEventListener('click', () => console.log('outer'));
document.getElementById('inner').addEventListener('click', () => console.log('inner'));
document.getElementById('btn').addEventListener('click', () => console.log('btn'));

// User clicks the button
```

<details>
<summary>Show Output & Explanation</summary>

```
btn
inner
outer
```

Events **bubble** from the target element up through ancestors. The button's listener fires first, then the event propagates up to `inner`, then `outer`. This is the default bubbling phase (third argument defaults to `false`).

</details>

---

### Q2. Capturing vs bubbling order

```html
<div id="parent"><button id="child">Click</button></div>
```

```js
document.getElementById('parent').addEventListener('click', () => console.log('parent bubble'), false);
document.getElementById('parent').addEventListener('click', () => console.log('parent capture'), true);
document.getElementById('child').addEventListener('click', () => console.log('child'));
```

<details>
<summary>Show Output & Explanation</summary>

```
parent capture
child
parent bubble
```

**Capturing** (top → target, `true` flag) fires before the target. **Bubbling** (target → top, `false`) fires after. Order: capture phase down → target → bubble phase up. At the **target element**, listeners fire in registration order regardless of capture/bubble setting.

</details>

---

### Q3. `stopPropagation` stops bubbling

```html
<div id="outer"><button id="btn">Click</button></div>
```

```js
document.getElementById('outer').addEventListener('click', () => console.log('outer clicked'));
document.getElementById('btn').addEventListener('click', e => {
  e.stopPropagation();
  console.log('btn clicked');
});
```

<details>
<summary>Show Output & Explanation</summary>

```
btn clicked
```

`stopPropagation()` prevents the event from bubbling further up the DOM. `outer`'s listener never fires. Note: it does NOT prevent other listeners on the **same element** — use `stopImmediatePropagation()` for that.

</details>

---

### Q4. `preventDefault` does NOT stop bubbling

```html
<div id="outer"><a id="link" href="https://example.com">Click</a></div>
```

```js
document.getElementById('outer').addEventListener('click', () => console.log('outer fires'));
document.getElementById('link').addEventListener('click', e => {
  e.preventDefault();
  console.log('link clicked, default prevented');
});
```

<details>
<summary>Show Output & Explanation</summary>

```
link clicked, default prevented
outer fires
```

`preventDefault()` stops the browser's **default action** (navigation in this case) but does **not** stop event propagation. The event still bubbles up to `outer`. Use `stopPropagation()` separately if you want to stop bubbling.

</details>

---

### Q5. `event.target` vs `event.currentTarget`

```html
<div id="parent"><button id="child">Click</button></div>
```

```js
document.getElementById('parent').addEventListener('click', e => {
  console.log('target:', e.target.id);
  console.log('currentTarget:', e.currentTarget.id);
});

// User clicks the button
```

<details>
<summary>Show Output & Explanation</summary>

```
target: child
currentTarget: parent
```

`e.target` is the element that **triggered** the event (where the click originated). `e.currentTarget` is the element the **listener is attached to**. They differ when events bubble. `currentTarget` is essential in event delegation to know where the listener lives.

</details>

---

### Q6. Event delegation — detecting the right element

```html
<ul id="list">
  <li class="item">A</li>
  <li class="item">B</li>
  <li class="item">C</li>
</ul>
```

```js
document.getElementById('list').addEventListener('click', e => {
  if (e.target.matches('.item')) {
    console.log('clicked:', e.target.textContent);
  }
});

// User clicks list item B
```

<details>
<summary>Show Output & Explanation</summary>

```
clicked: B
```

Event delegation attaches **one listener** to the parent. Clicks on any `.item` child bubble up. `e.target.matches('.item')` filters for the right elements. New items added dynamically are automatically handled — no re-registration needed.

</details>

---

### Q7. `once: true` auto-removes the listener

```js
let count = 0;
document.body.addEventListener('click', () => {
  count++;
  console.log('clicked', count);
}, { once: true });

// User clicks body 3 times
```

<details>
<summary>Show Output & Explanation</summary>

```
clicked 1
```

`{ once: true }` **automatically removes the listener** after the first invocation. Subsequent clicks are not handled. This is equivalent to manually calling `removeEventListener` inside the handler but cleaner.

</details>

---

### Q8. Passive event listener prevents `preventDefault`

```js
document.addEventListener('touchstart', e => {
  try {
    e.preventDefault();
    console.log('prevented');
  } catch (err) {
    console.log('error:', err.message);
  }
}, { passive: true });
```

<details>
<summary>Show Output & Explanation</summary>

```
error: Unable to preventDefault inside passive event listener invocation
```

`{ passive: true }` promises the browser that `preventDefault()` won't be called, allowing the browser to **scroll immediately** without waiting for your handler. Calling `preventDefault()` in a passive listener throws an error (or is silently ignored in some browsers). Use passive for scroll/touch events to improve performance.

</details>

---

### Q9. Removing a listener requires same reference

```js
const handler = () => console.log('fired');

document.body.addEventListener('click', handler);
document.body.removeEventListener('click', () => console.log('fired')); // different reference!

// User clicks body
```

<details>
<summary>Show Output & Explanation</summary>

```
fired
```

`removeEventListener` requires the **exact same function reference**. The anonymous function `() => console.log('fired')` is a new object every time — it doesn't match `handler`. The original listener is NOT removed. Always save a reference when you need to remove it later.

</details>

---

### Q10. `AbortController` removes multiple listeners at once

```js
const controller = new AbortController();
const { signal } = controller;

window.addEventListener('click', () => console.log('click'), { signal });
window.addEventListener('keydown', () => console.log('keydown'), { signal });

controller.abort(); // removes both listeners

// User clicks and presses a key — nothing fires
console.log('listeners removed');
```

<details>
<summary>Show Output & Explanation</summary>

```
listeners removed
```

Passing a `signal` from an `AbortController` to `addEventListener` lets you remove **multiple listeners at once** with a single `controller.abort()` call. Clean, especially in component cleanup where many listeners need removal simultaneously.

</details>

---

### Q11. `DOMContentLoaded` vs `load` timing

```js
document.addEventListener('DOMContentLoaded', () => {
  console.log('DOM ready');
});

window.addEventListener('load', () => {
  console.log('all loaded');
});

console.log('script executing');
```

<details>
<summary>Show Output & Explanation</summary>

```
script executing
DOM ready
all loaded
```

Script code executes synchronously as the parser encounters it. `DOMContentLoaded` fires when HTML is parsed (DOM is ready, images/CSS may not be). `load` fires when **all resources** (images, CSS, iframes) are fully loaded. Always use `DOMContentLoaded` for DOM manipulation — faster.

</details>

---

### Q12. `querySelectorAll` returns a static NodeList

```js
const items = document.querySelectorAll('.item');
console.log(items.length); // initial count

const newItem = document.createElement('div');
newItem.className = 'item';
document.body.appendChild(newItem);

console.log(items.length); // after adding
```

<details>
<summary>Show Output & Explanation</summary>

```
[initial count]
[same initial count]
```

`querySelectorAll` returns a **static NodeList** — a snapshot taken at call time. Adding new matching elements to the DOM does NOT update it. By contrast, `getElementsByClassName` returns a **live HTMLCollection** that updates automatically.

</details>

---

### Q13. Layout thrashing — forced synchronous layout

```js
const el = document.querySelector('.box');

// Each read after a write forces the browser to recalculate layout
el.style.width = '100px'; // write
const w1 = el.offsetWidth;  // read — forces layout
el.style.height = '100px'; // write
const h1 = el.offsetHeight; // read — forces layout

console.log(w1, h1);
```

<details>
<summary>Show Output & Explanation</summary>

```
100 100
```

The code works, but this pattern causes **layout thrashing** — alternating reads and writes force the browser to recalculate layout multiple times. The fix: batch all reads, then all writes. Actual values depend on the element, but the thrashing pattern is the key interview point.

</details>

---

### Q14. `IntersectionObserver` fires when element enters viewport

```js
const observer = new IntersectionObserver(entries => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      console.log('element is visible');
      observer.unobserve(entry.target);
    } else {
      console.log('element not visible');
    }
  });
}, { threshold: 0.5 });

observer.observe(document.querySelector('.lazy'));
```

<details>
<summary>Show Output & Explanation</summary>

```
// Initially (if element is off-screen):
element not visible

// When element is scrolled into view (50%+ visible):
element is visible
```

`threshold: 0.5` fires when 50%+ of the element is visible. `isIntersecting` is `true` when in view. `unobserve` stops watching after the first visibility. This is more performant than `scroll` event listeners for lazy-loading.

</details>

---

### Q15. `MutationObserver` fires after DOM changes

```js
const observer = new MutationObserver(mutations => {
  mutations.forEach(m => console.log('mutation:', m.type, m.addedNodes.length));
});

observer.observe(document.body, { childList: true, subtree: true });

const div = document.createElement('div');
document.body.appendChild(div);
```

<details>
<summary>Show Output & Explanation</summary>

```
mutation: childList 1
```

`MutationObserver` fires asynchronously (as a microtask) after the DOM change. `type: 'childList'` means children were added/removed. `addedNodes.length === 1` — the new `div`. Unlike polling, `MutationObserver` is efficient and fires only when something actually changes.

</details>

---

### Q16. Event listener on removed element

```js
const btn = document.createElement('button');
btn.textContent = 'Click';

let clicked = false;
btn.addEventListener('click', () => {
  clicked = true;
  console.log('clicked!');
});

document.body.appendChild(btn);
document.body.removeChild(btn); // removed from DOM

btn.click(); // programmatically click the detached element
console.log('clicked:', clicked);
```

<details>
<summary>Show Output & Explanation</summary>

```
clicked!
clicked: true
```

Removing an element from the DOM does **not** remove its event listeners or prevent them from firing if the element is still referenced in JS. `btn.click()` fires the listener even on a detached element. This is why detached elements with listeners can cause **memory leaks** — the JS reference keeps them alive.

</details>

---

### Q17. `stopImmediatePropagation` vs `stopPropagation`

```js
const btn = document.querySelector('button');

btn.addEventListener('click', e => {
  console.log('listener 1');
  e.stopImmediatePropagation();
});

btn.addEventListener('click', () => {
  console.log('listener 2'); // will this fire?
});

document.body.addEventListener('click', () => {
  console.log('body'); // will this fire?
});
```

<details>
<summary>Show Output & Explanation</summary>

```
listener 1
```

`stopImmediatePropagation()` stops **both** bubbling AND other listeners on the **same element**. `listener 2` (same element, registered after) is not called. `body` listener is also not called. Compare: `stopPropagation()` would still allow `listener 2` to fire.

</details>

---

### Q18. `document.getElementById` vs `querySelector` performance

```js
// Both find the same element
const byId = document.getElementById('app');
const byQuery = document.querySelector('#app');

console.log(byId === byQuery);
```

<details>
<summary>Show Output & Explanation</summary>

```
true
```

Both return the same DOM element — `===` comparison of object references is `true`. `getElementById` is generally **faster** as it has a direct lookup in an internal hash map. `querySelector` supports full CSS selectors but has slightly more parsing overhead. Use `getElementById` for ID lookups.

</details>

---

### Q19. `ResizeObserver` fires on element — not window

```js
const observer = new ResizeObserver(entries => {
  entries.forEach(entry => {
    console.log('width:', entry.contentRect.width);
  });
});

const box = document.querySelector('.box');
observer.observe(box);
// When the box changes size (e.g. parent resized):
// Logs the new width
```

<details>
<summary>Show Output & Explanation</summary>

```
// On each resize of the .box element:
width: [new width in pixels]
```

`ResizeObserver` watches **individual elements**, not the viewport. It fires when the element's content size changes — regardless of how (CSS, JS, parent resize, flex/grid relayout). Better than `window.resize` for component-level responsive behavior.

</details>

---

### Q20. Inline vs `addEventListener` — multiple handlers

```js
const btn = document.querySelector('button');
btn.onclick = () => console.log('handler 1');
btn.onclick = () => console.log('handler 2');

btn.addEventListener('click', () => console.log('listener 1'));
btn.addEventListener('click', () => console.log('listener 2'));

// User clicks button
```

<details>
<summary>Show Output & Explanation</summary>

```
handler 2
listener 1
listener 2
```

`onclick` is a **property** — assigning it twice overwrites the first. Only `handler 2` fires. `addEventListener` **adds** listeners without replacing — both `listener 1` and `listener 2` fire. `onclick` fires before `addEventListener` handlers at the target phase.

</details>

---

### Q21. Event object is shared across handlers in bubbling

```js
document.getElementById('inner').addEventListener('click', e => {
  e.customData = 'set by inner';
  console.log('inner:', e.customData);
});

document.getElementById('outer').addEventListener('click', e => {
  console.log('outer:', e.customData);
});
```

<details>
<summary>Show Output & Explanation</summary>

```
inner: set by inner
outer: set by inner
```

The **same event object** is passed to all handlers during propagation. Properties added to it in one handler are visible in subsequent handlers in the bubbling chain. This is sometimes used (but rarely recommended) to pass data between event handlers.

</details>

---

### Q22. `DocumentFragment` batches DOM insertions

```js
const list = document.getElementById('list');
const fragment = document.createDocumentFragment();

for (let i = 0; i < 5; i++) {
  const li = document.createElement('li');
  li.textContent = `Item ${i}`;
  fragment.appendChild(li);
}

console.log('fragment children:', fragment.childNodes.length);
list.appendChild(fragment);
console.log('fragment children after append:', fragment.childNodes.length);
```

<details>
<summary>Show Output & Explanation</summary>

```
fragment children: 5
fragment children after append: 0
```

`DocumentFragment` is an off-screen container — appending to it causes no reflow. When appended to a real DOM node, the fragment's children **move** into the DOM (not copied). The fragment becomes empty. This batches 5 potential reflows into 1.

</details>

---

### Q23. Keyboard event properties

```js
document.addEventListener('keydown', e => {
  console.log('key:', e.key);
  console.log('code:', e.code);
  console.log('keyCode:', e.keyCode); // deprecated
});

// User presses 'a'
```

<details>
<summary>Show Output & Explanation</summary>

```
key: a
code: KeyA
keyCode: 65
```

`e.key` returns the **logical key value** (layout-aware, e.g., 'a' or 'A' based on Shift). `e.code` returns the **physical key** (layout-independent, always 'KeyA' regardless of keyboard language). `e.keyCode` is deprecated — prefer `e.key` or `e.code`.

</details>

---

### Q24. Custom events with `CustomEvent`

```js
document.addEventListener('my-event', e => {
  console.log('received:', e.detail.message);
});

const event = new CustomEvent('my-event', {
  detail: { message: 'hello from custom event' },
  bubbles: true,
});

document.dispatchEvent(event);
```

<details>
<summary>Show Output & Explanation</summary>

```
received: hello from custom event
```

`CustomEvent` lets you create and dispatch your own events with arbitrary data in `detail`. `bubbles: true` allows it to propagate up the DOM tree. This is the native browser equivalent of an event emitter — useful for decoupled component communication without a framework.

</details>

---

---

**Q25. What is the output?**

```js
console.log('1');

requestAnimationFrame(() => {
  console.log('3: rAF');
});

setTimeout(() => {
  console.log('2: timeout');
}, 0);

console.log('1 end');
```

<details>
<summary>Show Output & Explanation</summary>

```
1
1 end
2: timeout
3: rAF
```

**Explanation:**  
`requestAnimationFrame` callbacks fire just before the browser paints the next frame (~16ms at 60fps). `setTimeout(fn, 0)` fires as a macrotask and typically runs before the next paint. So the order is: synchronous code → microtasks → macrotasks (setTimeout) → rAF (before paint) → paint. In practice rAF fires at the next vsync, after pending macrotasks.

> **Common Mistake:** Using `setTimeout(fn, 16)` for animation — it's unreliable and causes jank. `requestAnimationFrame` is synchronized with the display refresh rate, giving smooth 60fps animation automatically.

</details>

---

**Q26. What is the output?**

```js
let rafId;
let count = 0;

function animate() {
  count++;
  console.log('frame', count);
  if (count < 3) {
    rafId = requestAnimationFrame(animate);
  }
}

rafId = requestAnimationFrame(animate);
// Assume 3 frames pass
```

<details>
<summary>Show Output & Explanation</summary>

```
frame 1
frame 2
frame 3
```

**Explanation:**  
`requestAnimationFrame` schedules a single callback for the next frame — to animate continuously, you recursively call `rAF` inside the callback. The loop stops when `count >= 3`. `cancelAnimationFrame(rafId)` can stop it at any point.

> **Common Mistake:** Calling `requestAnimationFrame` in a loop or `setInterval` for animation — rAF is self-scheduling via recursion; calling it multiple times creates multiple animation loops.

</details>

---

**Q27. What is the output?**

```js
console.log('start');

fetch('https://api.example.com/data')
  .then(res => {
    console.log('status:', res.ok);
    return res.json();
  })
  .then(data => console.log('data received'))
  .catch(err => console.log('error:', err.message));

console.log('end');
```

<details>
<summary>Show Output & Explanation</summary>

```
start
end
status: true
data received
```

**Explanation:**  
`fetch` is async — it returns a Promise immediately without blocking. Synchronous code (`'end'`) runs first. On success: the first `.then` fires with the Response object (`res.ok` is `true` for 2xx), then `.json()` returns another Promise, then the second `.then` fires with parsed data. Note: `fetch` only rejects on network failure, NOT on HTTP error status codes (404, 500 still resolve with `res.ok = false`).

> **Common Mistake:** Assuming `fetch` rejects on 404/500. It doesn't — always check `res.ok` or `res.status` in the first `.then`. Only network failures (no connection) trigger `.catch`.

</details>

---

**Q28. What is the output?**

```js
console.log('initial:', location.pathname);

history.pushState({ page: 1 }, '', '/page1');
console.log('after push:', location.pathname);

history.pushState({ page: 2 }, '', '/page2');
console.log('after second push:', location.pathname);

history.back(); // triggers popstate asynchronously

window.addEventListener('popstate', (e) => {
  console.log('popstate:', location.pathname, 'state:', e.state?.page);
});
```

<details>
<summary>Show Output & Explanation</summary>

```
initial: /
after push: /page1
after second push: /page2
popstate: /page1 state: 1
```

**Explanation:**  
`pushState` changes the URL and adds a history entry WITHOUT a page reload — core to SPA routing. `replaceState` changes the URL without adding an entry. `history.back()` is asynchronous — the `popstate` event fires when navigation completes. `e.state` contains the state object passed to `pushState`.

> **Common Mistake:** Expecting `pushState` to trigger a `popstate` event — it doesn't. `popstate` only fires on navigation (back/forward button or `history.back()`/`history.go()`), not on `pushState`/`replaceState` calls.

</details>

---

*Chapter 9 of JavaScript Interview Prep Series*
