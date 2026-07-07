# Chapter 7: Arrays & Data Structures

> **Target:** Frontend Developer (4+ years) | Interview-focused, practical explanations

---

## Table of Contents

- [Arrays Fundamentals](#arrays-fundamentals)
- [Array Methods](#array-methods)
- [Modern Array Methods (ES2022+)](#modern-array-methods-es2022)
- [Data Structures](#data-structures)
- [Advanced Concepts](#advanced-concepts)

---

## Arrays Fundamentals

<details>
<summary><strong>1. How are Arrays stored in memory in JavaScript?</strong></summary>

JavaScript arrays are **objects** under the hood — not contiguous blocks of memory like C arrays. The engine optimizes them when they contain uniform types (V8's "fast elements" mode), but they can hold mixed types.

```js
typeof [] // 'object'
[] instanceof Object // true

// Dense array — engine uses contiguous memory (fast)
const dense = [1, 2, 3, 4, 5];

// Sparse array — treated as object with integer keys (slow)
const sparse = [];
sparse[1000] = 'far';
// indices 0–999 are "holes" — not allocated
```

**Engine optimization (V8):**
- **SMI (Small Integer)** arrays — packed int32 values → fastest
- **DOUBLE** arrays — Float64 → fast
- **PACKED** arrays — mixed objects → slower
- **HOLEY** arrays — with gaps → slowest

> **Interview Note:** Adding non-numeric keys to an array or creating sparse arrays can deopt V8 from fast element mode. Always initialize with known size and avoid holes.

</details>

---

<details>
<summary><strong>2. What is the time complexity of common Array operations?</strong></summary>

| Operation | Method | Complexity | Reason |
|-----------|--------|------------|--------|
| Access by index | `arr[i]` | O(1) | Direct memory offset |
| Push / Pop | `push()` / `pop()` | O(1) amortized | Modify end |
| Shift / Unshift | `shift()` / `unshift()` | O(n) | Re-index all elements |
| Insert middle | `splice()` | O(n) | Shift elements |
| Search | `indexOf()` / `find()` | O(n) | Linear scan |
| Sort | `sort()` | O(n log n) | TimSort |
| Slice | `slice()` | O(n) | Copies elements |

```js
const arr = [1, 2, 3, 4, 5];

arr.push(6);    // O(1) — append to end
arr.pop();      // O(1) — remove from end
arr.shift();    // O(n) — remove from front, all indices shift
arr.unshift(0); // O(n) — insert at front, all indices shift
```

> **Interview Note:** This is why linked lists are theoretically faster for frequent front insertions — but in practice, JS array's cache locality often makes them faster than naive linked lists for most use cases.

</details>

---

<details>
<summary><strong>3. Why are <code>push()</code> and <code>pop()</code> generally faster than <code>shift()</code> and <code>unshift()</code>?</strong></summary>

`push`/`pop` operate at the **end** of the array — no re-indexing needed.  
`shift`/`unshift` operate at the **front** — every element's index must be updated.

```js
// O(1) — just place/remove at next slot
arr.push(10);
arr.pop();

// O(n) — must re-index [1,2,3] → after shift → [2,3] (index 1→0, 2→1)
arr.shift();
arr.unshift(0); // [0, old[0], old[1], ...]
```

**Queue vs Stack:**
- **Stack** (LIFO): use `push` + `pop` → O(1)
- **Queue** (FIFO): `push` + `shift` is O(n). For high-frequency queues, use a linked list or circular buffer.

</details>

---

<details>
<summary><strong>4. What is the difference between mutable and immutable Array methods?</strong></summary>

**Mutable methods** — modify the original array in place.  
**Immutable methods** — return a new array, original unchanged.

| Mutable | Immutable |
|---------|-----------|
| `push`, `pop` | `concat` |
| `shift`, `unshift` | `slice` |
| `splice` | `map`, `filter`, `reduce` |
| `sort` | `flat`, `flatMap` |
| `reverse` | `toSorted`, `toReversed` (ES2023) |
| `fill`, `copyWithin` | `toSpliced` (ES2023) |

```js
const arr = [3, 1, 2];

// Mutable — modifies arr
arr.sort(); // arr = [1, 2, 3]

// Immutable alternative (ES2023)
const sorted = [...arr].sort(); // arr unchanged
const sorted2 = arr.toSorted(); // arr unchanged, returns new array
```

> **Interview Note:** In React, always use immutable patterns — mutating arrays in state won't trigger re-renders because the reference doesn't change. Use `[...arr]` or immutable methods.

</details>

---

## Array Methods

<details>
<summary><strong>5. What is the difference between <code>slice()</code> and <code>splice()</code>?</strong></summary>

| | `slice()` | `splice()` |
|--|-----------|------------|
| **Mutates** | No | Yes |
| **Returns** | New sub-array | Removed elements |
| **Use** | Extract / copy | Insert / delete / replace |

```js
const arr = [1, 2, 3, 4, 5];

// slice(start, end) — non-mutating
arr.slice(1, 3);   // [2, 3] — arr unchanged
arr.slice(-2);     // [4, 5] — last 2 elements
arr.slice();       // [1,2,3,4,5] — full shallow copy

// splice(start, deleteCount, ...itemsToInsert) — mutating
arr.splice(1, 2);        // removes [2,3], arr = [1,4,5]
arr.splice(1, 0, 99);    // inserts 99 at index 1, nothing removed
arr.splice(1, 1, 10, 11);// replaces 1 element with 10 and 11
```

> **Memory trick:** s**l**ice → **l**eaves original alone. s**p**lice → **p**okes the original.

</details>

---

<details>
<summary><strong>6. What is the difference between <code>map()</code>, <code>forEach()</code>, and <code>filter()</code>?</strong></summary>

| | `forEach` | `map` | `filter` |
|--|-----------|-------|----------|
| **Returns** | `undefined` | New array (same length) | New array (subset) |
| **Mutates** | No | No | No |
| **Use** | Side effects | Transform each element | Select elements |
| **Break early** | No | No | No |

```js
const nums = [1, 2, 3, 4, 5];

// forEach — for side effects only
nums.forEach(n => console.log(n)); // returns undefined

// map — transform, returns new array
const doubled = nums.map(n => n * 2); // [2, 4, 6, 8, 10]

// filter — select, returns subset
const evens = nums.filter(n => n % 2 === 0); // [2, 4]

// Chaining — readable pipeline
const result = nums
  .filter(n => n > 2)     // [3, 4, 5]
  .map(n => n * 10);       // [30, 40, 50]
```

> **Interview Note:** Never use `map` when you don't need the return value — use `forEach` for side effects. Never use `forEach` when you need a transformed array — use `map`.

</details>

---

<details>
<summary><strong>7. When would you use <code>map()</code> instead of <code>forEach()</code>?</strong></summary>

Use `map` whenever you need a **new array** based on a transformation. Use `forEach` when you only need **side effects** (logging, DOM updates, API calls).

```js
// ✅ map — transform to new array
const userNames = users.map(u => u.name);
const jsxElements = items.map(item => <Item key={item.id} {...item} />);

// ✅ forEach — side effects, no return needed
users.forEach(u => sendEmail(u.email));
nodes.forEach(node => node.classList.add('active'));

// ❌ Anti-pattern — map for side effects (wastes memory creating unused array)
users.map(u => sendEmail(u.email)); // creates array of undefined returns

// ❌ Anti-pattern — forEach when you need result (extra variable needed)
const names = [];
users.forEach(u => names.push(u.name)); // verbose; use map instead
```

</details>

---

<details>
<summary><strong>8. What is the difference between <code>find()</code>, <code>findIndex()</code>, <code>some()</code>, and <code>every()</code>?</strong></summary>

| Method | Returns | Stops at first match? |
|--------|---------|----------------------|
| `find()` | First matching **element** (or `undefined`) | Yes |
| `findIndex()` | Index of first match (or `-1`) | Yes |
| `some()` | `true` if **any** match | Yes |
| `every()` | `true` if **all** match | Yes (on first fail) |

```js
const users = [
  { id: 1, name: 'Alice', active: true },
  { id: 2, name: 'Bob', active: false },
  { id: 3, name: 'Carol', active: true },
];

users.find(u => u.id === 2);        // { id: 2, name: 'Bob', ... }
users.findIndex(u => u.id === 2);   // 1
users.some(u => !u.active);         // true  — Bob is inactive
users.every(u => u.active);         // false — Bob is inactive

// Early exit advantage — all stop as soon as result is known
```

> **Interview Note:** All four short-circuit — they stop iterating as soon as the result is determined. This makes them more efficient than `filter().length > 0` for existence checks.

</details>

---

<details>
<summary><strong>9. What is <code>reduce()</code> and what are its most common use cases?</strong></summary>

`reduce(callback, initialValue)` processes each element and accumulates a **single result**. It's the most powerful array method.

```js
// Signature: reduce((accumulator, current, index, array) => ..., initialValue)
const nums = [1, 2, 3, 4, 5];

// Sum
nums.reduce((acc, n) => acc + n, 0); // 15

// Product
nums.reduce((acc, n) => acc * n, 1); // 120

// Max value
nums.reduce((max, n) => n > max ? n : max, -Infinity); // 5

// Count occurrences
['a','b','a','c','b','a'].reduce((acc, char) => {
  acc[char] = (acc[char] || 0) + 1;
  return acc;
}, {}); // { a: 3, b: 2, c: 1 }

// Flatten one level
[[1,2],[3,4],[5]].reduce((acc, arr) => [...acc, ...arr], []); // [1,2,3,4,5]

// Group by property
users.reduce((groups, user) => {
  const key = user.role;
  (groups[key] ||= []).push(user);
  return groups;
}, {});
```

> **Interview Note:** Always provide an `initialValue`. Without it, `reduce` uses the first element as the initial accumulator — which breaks on empty arrays and can cause type bugs.

</details>

---

<details>
<summary><strong>10. Can you implement common Array operations using <code>reduce()</code>?</strong></summary>

```js
const arr = [1, 2, 3, 4, 5];

// Implement map with reduce
const myMap = (arr, fn) =>
  arr.reduce((acc, item) => [...acc, fn(item)], []);
myMap(arr, x => x * 2); // [2, 4, 6, 8, 10]

// Implement filter with reduce
const myFilter = (arr, fn) =>
  arr.reduce((acc, item) => fn(item) ? [...acc, item] : acc, []);
myFilter(arr, x => x % 2 === 0); // [2, 4]

// Implement flat with reduce
const myFlat = (arr) =>
  arr.reduce((acc, item) =>
    Array.isArray(item) ? [...acc, ...myFlat(item)] : [...acc, item], []);
myFlat([[1, [2]], [3]]); // [1, 2, 3]

// Pipe (compose left to right)
const pipe = (...fns) => x => fns.reduce((v, fn) => fn(v), x);
const process = pipe(x => x * 2, x => x + 1, x => x ** 2);
process(3); // ((3*2)+1)^2 = 49
```

</details>

---

<details>
<summary><strong>11. How would you remove duplicate values from an Array?</strong></summary>

```js
const arr = [1, 2, 2, 3, 3, 3, 4];

// 1. Set — simplest, most readable (ES6)
const unique = [...new Set(arr)]; // [1, 2, 3, 4]

// 2. filter + indexOf
const unique2 = arr.filter((val, i) => arr.indexOf(val) === i);

// 3. reduce
const unique3 = arr.reduce((acc, val) =>
  acc.includes(val) ? acc : [...acc, val], []);

// For objects — by key
const users = [{ id: 1 }, { id: 2 }, { id: 1 }];
const uniqueUsers = [...new Map(users.map(u => [u.id, u])).values()];
// [{ id: 1 }, { id: 2 }]
```

**Performance:** `Set` is O(n), `filter+indexOf` is O(n²). For large arrays, always use `Set`.

> **Interview Note:** `Set` preserves insertion order and handles primitives perfectly. For deduplicating objects, use `Map` with a unique key.

</details>

---

<details>
<summary><strong>12. What are different ways to flatten a nested Array?</strong></summary>

```js
const nested = [1, [2, [3, [4]]], 5];

// flat(depth) — ES2019
nested.flat();     // [1, 2, [3, [4]], 5] — depth 1 (default)
nested.flat(2);    // [1, 2, 3, [4], 5]
nested.flat(Infinity); // [1, 2, 3, 4, 5] — fully flatten

// flatMap — map then flat(1)
[1, 2, 3].flatMap(n => [n, n * 2]); // [1, 2, 2, 4, 3, 6]

// reduce (manual, any depth)
const deepFlat = arr =>
  arr.reduce((acc, val) =>
    Array.isArray(val) ? [...acc, ...deepFlat(val)] : [...acc, val], []);

// Legacy — toString trick (only for numbers/primitives)
nested.toString().split(',').map(Number); // [1, 2, 3, 4, 5]
```

> **Interview Note:** `flat(Infinity)` is the modern, clean answer. Know `flatMap` too — it's flat(1) + map combined, useful for "one-to-many" transforms (e.g., splitting sentences into words).

</details>

---

<details>
<summary><strong>13. How would you sort numbers correctly in JavaScript?</strong></summary>

The default `sort()` converts elements to strings — completely wrong for numbers.

```js
// ❌ Default — lexicographic (string) sort
[10, 9, 2, 1, 100].sort(); // [1, 10, 100, 2, 9]

// ✅ Numeric ascending
[10, 9, 2, 1, 100].sort((a, b) => a - b); // [1, 2, 9, 10, 100]

// ✅ Numeric descending
[10, 9, 2, 1, 100].sort((a, b) => b - a); // [100, 10, 9, 2, 1]

// Sort objects by property
const users = [{ age: 30 }, { age: 25 }, { age: 35 }];
users.sort((a, b) => a.age - b.age); // ascending by age

// Sort strings — use localeCompare
['banana', 'apple', 'cherry'].sort((a, b) => a.localeCompare(b));

// Stable sort — guaranteed since ES2019 (Chrome 70+, Node 11+)
```

> **Interview Note:** `sort` is mutating — it modifies the original array. For immutable sort: `[...arr].sort(...)` or `arr.toSorted(...)` (ES2023).

</details>

---

## Modern Array Methods (ES2022+)

### Array.at() — Clean Negative Indexing

```js
const arr = [10, 20, 30, 40, 50];

arr.at(0);   // 10  (same as arr[0])
arr.at(-1);  // 50  (last element)
arr.at(-2);  // 40  (second to last)
arr.at(10);  // undefined (out of bounds)
```

**Why use it:** `arr[arr.length - 1]` is verbose and error-prone. `arr.at(-1)` is cleaner and works on any array-like (strings, TypedArrays).

> **Common mistake:** `arr[-1]` does NOT give the last element — it looks up a property named `"-1"`, which doesn't exist → `undefined`.

### Object.groupBy() — ES2024

Groups array elements by a key:

```js
const products = [
  { name: 'apple', category: 'fruit' },
  { name: 'carrot', category: 'vegetable' },
  { name: 'banana', category: 'fruit' },
];

const grouped = Object.groupBy(products, p => p.category);
// {
//   fruit: [{ name: 'apple', ... }, { name: 'banana', ... }],
//   vegetable: [{ name: 'carrot', ... }]
// }
```

**Before ES2024** you'd use `reduce`:
```js
products.reduce((acc, p) => {
  (acc[p.category] ??= []).push(p);
  return acc;
}, {});
```

`Object.groupBy` is cleaner and expresses intent directly.

### Immutable Array Methods (ES2023)

These return a NEW array instead of mutating the original:

| Mutating (old) | Immutable (new) |
|---|---|
| `arr.sort()` | `arr.toSorted()` |
| `arr.reverse()` | `arr.toReversed()` |
| `arr.splice()` | `arr.toSpliced()` |
| `arr[i] = x` | `arr.with(i, x)` |

```js
const nums = [3, 1, 2];
const sorted = nums.toSorted();
console.log(sorted); // [1, 2, 3]
console.log(nums);   // [3, 1, 2] — original unchanged
```

---

## Data Structures

<details>
<summary><strong>14. What is a Set and when would you use it?</strong></summary>

A `Set` is a collection of **unique values** of any type. It automatically rejects duplicates.

```js
const set = new Set([1, 2, 2, 3, 3]);
set.size;         // 3
set.has(2);       // true
set.add(4);       // Set {1, 2, 3, 4}
set.delete(2);    // Set {1, 3, 4}

// Iteration — insertion order preserved
for (const val of set) console.log(val);
[...set]; // convert to array

// Set operations (ES2025 native, or manual)
const a = new Set([1, 2, 3]);
const b = new Set([2, 3, 4]);

// Union
new Set([...a, ...b]);            // {1,2,3,4}

// Intersection
new Set([...a].filter(x => b.has(x))); // {2,3}

// Difference
new Set([...a].filter(x => !b.has(x))); // {1}
```

**When to use Set:**
- Remove duplicates from data
- Membership checks (O(1) vs O(n) for arrays)
- Tracking visited nodes in graph traversal

</details>

---

<details>
<summary><strong>15. What is the difference between Set and Array?</strong></summary>

| | Array | Set |
|--|-------|-----|
| **Duplicates** | Allowed | Not allowed |
| **Access** | By index | No index access |
| **Has/includes** | `indexOf` O(n) | `has()` O(1) |
| **Order** | Insertion order | Insertion order |
| **Size** | `.length` | `.size` |
| **Iterable** | Yes | Yes |

```js
// Array — when order and index access matter
const items = ['a', 'b', 'c'];
items[1]; // 'b'

// Set — when uniqueness and fast membership testing matter
const visited = new Set();
visited.add(url);
if (visited.has(url)) return; // O(1) check
```

</details>

---

<details>
<summary><strong>16. What is a Map and when would you use it?</strong></summary>

A `Map` is a key-value store where **any value** (including objects) can be a key, and insertion order is preserved.

```js
const map = new Map();

map.set('name', 'Alice');
map.set(42, 'the answer');
map.set({ id: 1 }, 'user object key');

map.get('name'); // 'Alice'
map.get(42);     // 'the answer'
map.has('name'); // true
map.size;        // 3
map.delete('name');

// Iteration
for (const [key, val] of map) console.log(key, val);
[...map.keys()];
[...map.values()];
[...map.entries()];

// Initialize from array of pairs
const map2 = new Map([['a', 1], ['b', 2]]);
```

**When to use Map:**
- Keys that aren't strings (objects, functions, numbers)
- Frequent add/delete operations (better than object)
- When you need to know the size easily (`.size`)
- Ordered key-value pairs

</details>

---

<details>
<summary><strong>17. What is the difference between Map and Object?</strong></summary>

| | Object | Map |
|--|--------|-----|
| **Key types** | String / Symbol only | Any value |
| **Default keys** | Yes (prototype) | None |
| **Size** | `Object.keys().length` | `.size` |
| **Iteration** | `for...in` (includes prototype) | `for...of` (safe) |
| **Performance** | Slower for frequent add/delete | Optimized for add/delete |
| **JSON** | `JSON.stringify` | Needs manual conversion |
| **Prototype pollution** | Risk | No risk |

```js
// Object — prototype keys can interfere
const obj = {};
'toString' in obj; // true — from prototype!

// Map — clean, no prototype
const map = new Map();
map.has('toString'); // false ✅

// Object key coercion
const obj2 = {};
obj2[{a: 1}] = 'test';
obj2['[object Object]']; // 'test' — object coerced to string!

// Map — true object keys
const map2 = new Map();
const key = { a: 1 };
map2.set(key, 'test');
map2.get(key); // 'test' ✅
```

</details>

---

<details>
<summary><strong>18. What are WeakMap and WeakSet, and when would you use them?</strong></summary>

`WeakMap` and `WeakSet` hold **weak references** to objects — they don't prevent garbage collection.

```js
// WeakMap — keys must be objects, values can be anything
const wm = new WeakMap();
let obj = { id: 1 };
wm.set(obj, 'metadata');
wm.get(obj); // 'metadata'
obj = null;  // obj eligible for GC — WeakMap entry also removed

// WeakSet — only stores objects, weakly
const ws = new WeakSet();
let el = document.querySelector('#btn');
ws.add(el);
ws.has(el); // true
el = null;  // el GC'd — ws entry removed automatically
```

**Key differences from Map/Set:**
- No iteration (`for...of`, `forEach`) — they're not enumerable
- No `.size` property
- Keys/values must be objects

**Use cases:**
- Storing private data per object instance without preventing GC
- Tracking DOM nodes without causing memory leaks
- Caching computed results linked to object lifetime

```js
// Private data pattern with WeakMap
const _private = new WeakMap();

class MyClass {
  constructor() { _private.set(this, { secret: 42 }); }
  getSecret() { return _private.get(this).secret; }
}
```

</details>

---

<details>
<summary><strong>19. When would you choose Array, Object, Map, or Set?</strong></summary>

| Use case | Best choice |
|----------|-------------|
| Ordered list of items | **Array** |
| Unique values / membership test | **Set** |
| Key-value, string keys, JSON needed | **Object** |
| Key-value, any key type, ordered | **Map** |
| Cache tied to object lifetime | **WeakMap** |
| Track objects without leak risk | **WeakSet** |

```js
// Tags on a post — unique strings → Set
const tags = new Set(['js', 'react', 'ts']);

// Config object — string keys → Object (JSON-serializable)
const config = { port: 3000, debug: false };

// Request cache keyed by DOM node → WeakMap
const cache = new WeakMap();

// Ordered events with metadata → Map (preserve insertion order + rich keys)
const eventLog = new Map();
eventLog.set(new Date(), { type: 'click', target: btn });
```

</details>

---

## Advanced Concepts

<details>
<summary><strong>20. What are Array-like Objects and how do you convert them into Arrays?</strong></summary>

Array-like objects have numeric indices and a `length` property but lack array methods.

```js
// Common array-like objects
arguments      // in regular functions
NodeList       // from querySelectorAll
HTMLCollection // from getElementsByClassName
String         // 'hello'[0] = 'h'

function foo() {
  console.log(arguments.length); // exists
  arguments.map(x => x);        // ❌ TypeError — no array methods
}

// Convert to real array
Array.from(arguments);       // ✅ ES6
[...arguments];              // ✅ spread
Array.prototype.slice.call(arguments); // ✅ legacy

// NodeList conversion
const divs = document.querySelectorAll('div');
const arr = [...divs];            // can now use .map(), .filter()
Array.from(divs, el => el.id);   // map while converting
```

</details>

---

<details>
<summary><strong>21. What are common performance pitfalls when working with large Arrays?</strong></summary>

**1. Creating holes (sparse arrays)**
```js
const arr = new Array(1000); // 1000 holes — slow
arr[999] = 1;                // sparse — V8 treats as dict
// Better: new Array(1000).fill(0)
```

**2. Using `shift()`/`unshift()` in hot paths**
```js
// O(n) repeatedly — O(n²) total
while (queue.length) { process(queue.shift()); }
// Fix: reverse loop or use a pointer index
```

**3. Repeated spread in loops**
```js
// O(n²) — creates new array each iteration
let result = [];
bigArray.forEach(item => result = [...result, transform(item)]);
// Fix: use push or map
```

**4. Blocking the main thread with huge arrays**
```js
// Process in chunks to avoid freezing UI
function processChunked(arr, fn, chunkSize = 100) {
  let i = 0;
  function next() {
    arr.slice(i, i + chunkSize).forEach(fn);
    i += chunkSize;
    if (i < arr.length) requestAnimationFrame(next);
  }
  next();
}
```

</details>

---

<details>
<summary><strong>22. What are common real-world use cases of Map and Set in frontend applications?</strong></summary>

**Set use cases:**
```js
// Deduplicate selected items
const selectedIds = new Set();
onSelect(id) { selectedIds.add(id); }
onDeselect(id) { selectedIds.delete(id); }
isSelected(id) { return selectedIds.has(id); } // O(1)

// Track visited routes
const visitedRoutes = new Set();

// Remove duplicate API results
const uniqueResults = [...new Set(apiResults.map(r => r.id))]
  .map(id => apiResults.find(r => r.id === id));
```

**Map use cases:**
```js
// DOM element → metadata
const tooltipData = new Map();
tooltipData.set(buttonEl, { text: 'Submit', delay: 500 });

// Request deduplication
const pendingRequests = new Map();
function fetchOnce(url) {
  if (pendingRequests.has(url)) return pendingRequests.get(url);
  const promise = fetch(url).finally(() => pendingRequests.delete(url));
  pendingRequests.set(url, promise);
  return promise;
}

// Event handler registry (remove listeners later)
const handlers = new Map();
handlers.set(element, handler);
element.removeEventListener('click', handlers.get(element));
```

</details>

---

## Quick Reference Cheat Sheet

```
Array methods — mutable vs immutable:
  Mutable:   sort, reverse, push, pop, shift, unshift, splice, fill
  Immutable: map, filter, reduce, slice, concat, flat, flatMap, toSorted, toReversed

Time complexity:
  push/pop   → O(1)    shift/unshift → O(n)
  indexOf    → O(n)    sort          → O(n log n)
  Set.has()  → O(1)    Map.get/set   → O(1)

Dedup:          [...new Set(arr)]
Flatten:        arr.flat(Infinity)
Sort numbers:   arr.sort((a, b) => a - b)
Group by:       arr.reduce((acc, x) => { (acc[x.key]||=[]).push(x); return acc; }, {})
```

---

*Chapter 7 of JavaScript Interview Prep Series*
