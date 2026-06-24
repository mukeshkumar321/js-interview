# Chapter 6: ES6+ Features

> **Target:** Frontend Developer (4+ years) | Interview-focused, practical explanations

---

## Table of Contents

- [Variables & Scope](#variables--scope)
- [Destructuring & Parameters](#destructuring--parameters)
- [Arrow Functions & Modules](#arrow-functions--modules)
- [Modern JavaScript Features](#modern-javascript-features)

---

## Variables & Scope

<details>
<summary><strong>1. What is the difference between <code>var</code>, <code>let</code>, and <code>const</code>?</strong></summary>

| | `var` | `let` | `const` |
|--|-------|-------|---------|
| **Scope** | Function | Block | Block |
| **Hoisted** | Yes (`undefined`) | Yes (TDZ) | Yes (TDZ) |
| **Re-declarable** | Yes | No | No |
| **Re-assignable** | Yes | Yes | No |
| **Global object prop** | Yes | No | No |
| **Introduced** | ES1 | ES6 | ES6 |

```js
// var — function scoped, leaks out of blocks
for (var i = 0; i < 3; i++) {}
console.log(i); // 3 — leaks out

// let — block scoped
for (let j = 0; j < 3; j++) {}
console.log(j); // ReferenceError

// const — block scoped, binding immutable
const obj = { x: 1 };
obj.x = 99;  // ✅ mutation allowed — reference is const, not the object
obj = {};    // ❌ TypeError — can't reassign the binding
```

> **Interview Note:** `const` doesn't make values immutable — it makes the **binding** immutable. You can still mutate arrays and objects assigned to `const`. For deep immutability use `Object.freeze()`.

</details>

---

<details>
<summary><strong>2. When should you use <code>const</code> vs <code>let</code> in production code?</strong></summary>

**Rule:** Default to `const`. Use `let` only when you know the value will be reassigned. Never use `var`.

```js
// ✅ const for everything that doesn't get reassigned
const MAX_RETRIES = 3;
const user = { name: 'Alice' };  // reference won't change
const double = x => x * 2;

// ✅ let only when reassignment is needed
let count = 0;
count++;

let result;
if (condition) result = 'A';
else result = 'B';

// ❌ avoid var entirely
var x = 10; // function-scoped, hoisting surprises
```

**Why `const` by default:**
- Signals intent — "this reference won't change"
- Prevents accidental reassignment
- Easier to reason about in closures
- Most linters enforce it (`prefer-const` ESLint rule)

</details>

---

## Destructuring & Parameters

<details>
<summary><strong>3. What is Object Destructuring and when would you use it?</strong></summary>

Object destructuring extracts properties from an object into variables in a single statement.

```js
const user = { name: 'Alice', age: 30, role: 'admin' };

// Basic destructuring
const { name, age } = user;

// Rename while destructuring
const { name: userName, role: userRole } = user;

// Default values
const { name, city = 'Unknown' } = user;
console.log(city); // 'Unknown' — not in user

// Nested destructuring
const { address: { city, zip } } = { address: { city: 'NY', zip: '10001' } };

// In function parameters (most common real-world use)
function greet({ name, role = 'user' }) {
  return `Hello ${name}, you are a ${role}`;
}
greet(user);

// Rest in destructuring
const { name, ...rest } = user; // rest = { age: 30, role: 'admin' }
```

> **Interview Note:** Destructuring in function parameters is the most practical use — it self-documents which properties are used and provides defaults inline. Common in React component props.

</details>

---

<details>
<summary><strong>4. What is Array Destructuring and what are common use cases?</strong></summary>

Array destructuring extracts values by position.

```js
const [first, second, third] = [1, 2, 3];

// Skip elements
const [a, , b] = [1, 2, 3]; // a=1, b=3

// Default values
const [x = 0, y = 0] = [5]; // x=5, y=0

// Rest
const [head, ...tail] = [1, 2, 3, 4]; // head=1, tail=[2,3,4]

// Swap variables (no temp variable needed)
let m = 1, n = 2;
[m, n] = [n, m]; // m=2, n=1

// From function returns
const [data, error] = await fetchData(); // common async pattern

// useState in React
const [count, setCount] = useState(0);
```

**Common use case — multiple return values:**
```js
function getMinMax(arr) {
  return [Math.min(...arr), Math.max(...arr)];
}
const [min, max] = getMinMax([3, 1, 4, 1, 5]);
```

</details>

---

<details>
<summary><strong>5. What are Default Parameters and why were they introduced?</strong></summary>

Default parameters assign a fallback value when an argument is `undefined`. They replaced the error-prone `||` pattern used before ES6.

```js
// Pre-ES6 (fragile — fails for 0, '', false)
function connect(host, port) {
  host = host || 'localhost'; // ❌ fails if host = ''
  port = port || 3000;        // ❌ fails if port = 0
}

// ES6 default parameters
function connect(host = 'localhost', port = 3000) {
  console.log(host, port);
}

connect();              // 'localhost', 3000
connect('example.com'); // 'example.com', 3000
connect('', 0);         // '', 0 — empty/zero are valid, don't trigger defaults
connect(undefined, 80); // 'localhost', 80 — undefined triggers default
connect(null, 80);      // null, 80 — null does NOT trigger default
```

**Expressions as defaults:**
```js
function foo(a, b = a * 2) { return b; }
foo(5); // 10

function bar(arr = []) { arr.push(1); return arr; }
bar(); // [1] — new array each call (expression evaluated fresh)
```

> **Common Mistake:** Using `||` for defaults in pre-ES6 code breaks for falsy values like `0` or `''`. Default parameters fix this cleanly.

</details>

---

<details>
<summary><strong>6. What are Rest Parameters and how do they work?</strong></summary>

Rest parameters collect all remaining function arguments into a real array.

```js
function sum(...numbers) {
  return numbers.reduce((total, n) => total + n, 0);
}
sum(1, 2, 3, 4); // 10

// With named params before rest
function log(level, ...messages) {
  messages.forEach(msg => console.log(`[${level}] ${msg}`));
}
log('ERROR', 'Not found', 'Check URL', 'Retry'); // 3 messages

// Only the last parameter can be rest
function invalid(...a, b) {} // ❌ SyntaxError
```

**vs `arguments` object:**

| | `...rest` | `arguments` |
|--|-----------|-------------|
| Type | Real Array | Array-like |
| Arrow functions | ✅ | ❌ |
| Only remaining args | ✅ | All args |
| Array methods | Directly | Need `Array.from()` |

```js
// arguments — old, array-like
function old() { return Array.from(arguments).join(', '); }

// rest — modern, real array
const modern = (...args) => args.join(', ');
```

</details>

---

<details>
<summary><strong>7. What is the Spread Operator and how is it different from Rest Parameters?</strong></summary>

Same syntax (`...`) but opposite purpose:
- **Rest** — collects multiple values **into** an array (function definition)
- **Spread** — expands an iterable **out** into individual values (function call / literal)

```js
// Rest — in function definition (collects)
function sum(...nums) { return nums.reduce((a, b) => a + b, 0); }

// Spread — in function call (expands)
const nums = [1, 2, 3];
sum(...nums);           // same as sum(1, 2, 3)
Math.max(...nums);      // 3

// Spread in array literals (copy / merge)
const a = [1, 2];
const b = [3, 4];
const merged = [...a, ...b]; // [1, 2, 3, 4]
const copy = [...a];         // shallow copy

// Spread in object literals (copy / merge)
const defaults = { color: 'blue', size: 'M' };
const custom   = { size: 'L', weight: 'heavy' };
const merged2  = { ...defaults, ...custom }; // { color: 'blue', size: 'L', weight: 'heavy' }
// Later keys override earlier ones
```

> **Interview Note:** Spread creates **shallow copies** — nested objects are still shared by reference. `[...arr]` ≠ deep copy.

</details>

---

## Arrow Functions & Modules

<details>
<summary><strong>8. What are Arrow Functions and what problems do they solve?</strong></summary>

Arrow functions are a concise syntax for functions that also fix `this` binding in callbacks.

```js
// Traditional
const double = function(x) { return x * 2; };

// Arrow — concise
const double = x => x * 2;

// Multi-line
const process = (x, y) => {
  const result = x + y;
  return result * 2;
};

// Returning object literal (needs parentheses)
const makeUser = name => ({ name, active: true });
```

**Problem they solve — `this` in callbacks:**
```js
// ❌ Old problem
function Timer() {
  this.count = 0;
  setInterval(function() {
    this.count++; // `this` = window, not Timer
  }, 1000);
}

// ✅ Arrow solution
function Timer() {
  this.count = 0;
  setInterval(() => {
    this.count++; // `this` = Timer instance (lexically inherited)
  }, 1000);
}
```

</details>

---

<details>
<summary><strong>9. What are the limitations of Arrow Functions?</strong></summary>

Arrow functions are NOT a universal replacement for regular functions:

**1. No own `this` — can't be used as object methods**
```js
const obj = {
  name: 'Obj',
  greet: () => console.log(this.name) // ❌ this = window/undefined
};
```

**2. Can't be used as constructors**
```js
const Person = (name) => { this.name = name; };
new Person('Alice'); // ❌ TypeError: Person is not a constructor
```

**3. No `arguments` object**
```js
const fn = () => console.log(arguments); // ❌ ReferenceError (or outer scope's arguments)
```

**4. Can't be used as generator functions**
```js
const gen = *() => {}; // ❌ SyntaxError
```

**5. No `prototype` property**
```js
const fn = () => {};
fn.prototype; // undefined — can't extend
```

> **Interview Note:** The most important limitation to know is no own `this`. Arrow functions are ideal for callbacks and array methods but wrong for object methods, constructors, and prototype methods.

</details>

---

<details>
<summary><strong>10. What are ES Modules and why are they preferred over CommonJS?</strong></summary>

**CommonJS (Node.js, pre-ES6):**
```js
// Export
module.exports = { add, subtract };
const utils = require('./utils');
```

**ES Modules (ES6+):**
```js
// Export
export const add = (a, b) => a + b;
export default function App() {}

// Import
import { add } from './utils';
import App from './App';
```

| | CommonJS | ES Modules |
|--|----------|------------|
| **Loading** | Synchronous | Asynchronous |
| **Analysis** | Runtime | Static (parse time) |
| **Tree shaking** | ❌ Hard | ✅ Easy |
| **`this` at top level** | `module.exports` | `undefined` |
| **Circular deps** | Supported (with caveats) | Supported |
| **Browser native** | ❌ | ✅ |

**Why ES Modules win:**
- **Static analysis** — bundlers can determine imports/exports at parse time
- **Tree shaking** — dead code is eliminated (smaller bundles)
- **Browser native** — no bundler needed for simple use cases
- **Strict mode** — modules are always strict

</details>

---

<details>
<summary><strong>11. What is the difference between Named Exports and Default Exports?</strong></summary>

**Named exports** — multiple per file, imported with the exact name (or aliased).  
**Default export** — one per file, imported with any name.

```js
// named-exports.js
export const PI = 3.14;
export function add(a, b) { return a + b; }
export class User {}

// Import named — must match name (or alias)
import { PI, add, User } from './named-exports';
import { add as sum } from './named-exports'; // aliased

// default-export.js
export default function calculate() {}

// Import default — any name works
import calculate from './default-export';
import calc from './default-export';      // also fine
import myFn from './default-export';      // also fine

// Mixed
export default class App {}
export const VERSION = '1.0';

import App, { VERSION } from './app';
```

**When to use which:**
- **Default** — for the primary thing a module exports (component, class, function)
- **Named** — for utilities, constants, multiple exports from one module

> **Interview Note:** Avoid mixing many defaults with named exports — it creates confusion. React component files typically use a default export for the component and named exports for types/utilities.

</details>

---

<details>
<summary><strong>12. What is Dynamic Import and when would you use it?</strong></summary>

Dynamic `import()` loads a module **on demand** at runtime rather than at the top of the file. It returns a Promise.

```js
// Static import — always loaded, even if not needed
import heavyLib from './heavy-lib';

// Dynamic import — loaded only when needed
async function loadFeature() {
  const { heavyFunction } = await import('./heavy-lib');
  heavyFunction();
}

// Conditional loading
if (userIsAdmin) {
  const { AdminPanel } = await import('./AdminPanel');
}

// React lazy loading
const Dashboard = React.lazy(() => import('./Dashboard'));
```

**Real-world use cases:**
1. **Route-based code splitting** — load page components only when navigated to
2. **Feature flags** — load a feature only if it's enabled
3. **Heavy libraries** — load chart library only when a chart is rendered
4. **Polyfills** — load only if the browser needs them

```js
// Webpack/Vite chunk splitting via dynamic import
const { Chart } = await import(
  /* webpackChunkName: "chart" */ 'chart.js'
);
```

</details>

---

## Modern JavaScript Features

<details>
<summary><strong>13. What is the difference between <code>for...of</code> and <code>for...in</code> loops?</strong></summary>

| | `for...of` | `for...in` |
|--|-----------|------------|
| **Iterates over** | Values | Keys/property names |
| **Works with** | Any iterable (array, string, Map, Set) | Objects (enumerable properties) |
| **Prototype chain** | No | Yes — includes inherited |
| **Arrays** | ✅ Use this | ⚠️ Avoid — iterates indices as strings |
| **Objects** | ❌ Plain objects not iterable | ✅ |

```js
const arr = [10, 20, 30];

// for...of — values
for (const val of arr) {
  console.log(val); // 10, 20, 30
}

// for...in — keys (indices as strings for arrays)
for (const key in arr) {
  console.log(key); // '0', '1', '2' — strings!
}

// for...in on objects (its intended use)
const obj = { a: 1, b: 2 };
for (const key in obj) {
  console.log(key, obj[key]); // 'a' 1, 'b' 2
}

// for...of with Map, Set, String
for (const char of 'hello') console.log(char); // h, e, l, l, o
for (const [key, val] of new Map([['a', 1]])) console.log(key, val);
```

> **Common Mistake:** Using `for...in` on arrays — it iterates indices as strings and can include prototype properties. Always use `for...of` or `forEach` for arrays.

</details>

---

<details>
<summary><strong>14. What are Template Literals and what advantages do they provide?</strong></summary>

Template literals use backticks and enable embedded expressions, multi-line strings, and tagged templates.

```js
const name = 'Alice';
const age = 30;

// String interpolation — no more concatenation
const msg = `Hello, ${name}! You are ${age} years old.`;

// Expressions inside ${}
const result = `${2 + 2} is four`;
const upper  = `${name.toUpperCase()} is here`;

// Multi-line strings (preserves newlines)
const html = `
  <div>
    <h1>${name}</h1>
    <p>Age: ${age}</p>
  </div>
`;

// Tagged templates — process template with a function
function highlight(strings, ...values) {
  return strings.reduce((result, str, i) =>
    `${result}${str}${values[i] ? `<b>${values[i]}</b>` : ''}`, '');
}
const tagged = highlight`Hello ${name}, age ${age}`;
// "Hello <b>Alice</b>, age <b>30</b>"
```

**Real-world tagged template use cases:**
- `css\`...\`` in styled-components
- `html\`...\`` in lit-element
- `gql\`...\`` in Apollo GraphQL
- `sql\`...\`` for safe SQL queries (prevents injection)

</details>

---

<details>
<summary><strong>15. What are Symbols and why were they introduced?</strong></summary>

`Symbol` is a primitive type that creates a **guaranteed unique value**. Every `Symbol()` call produces a new, distinct value.

```js
const id1 = Symbol('id');
const id2 = Symbol('id');
id1 === id2; // false — always unique

// Use as object keys to avoid name collisions
const ID = Symbol('id');
const user = {
  name: 'Alice',
  [ID]: 123 // Symbol key — hidden from for...in and Object.keys()
};

user[ID];         // 123
Object.keys(user); // ['name'] — Symbol keys hidden
for (const k in user) console.log(k); // 'name' only
```

**Well-known Symbols — customize built-in behavior:**
```js
class Range {
  constructor(start, end) { this.start = start; this.end = end; }

  [Symbol.iterator]() {
    let current = this.start;
    const end = this.end;
    return {
      next() {
        return current <= end
          ? { value: current++, done: false }
          : { done: true };
      }
    };
  }
}

for (const n of new Range(1, 5)) console.log(n); // 1, 2, 3, 4, 5
```

**Other well-known symbols:** `Symbol.toPrimitive`, `Symbol.hasInstance`, `Symbol.toStringTag`

> **Interview Note:** Symbols are primarily used for: (1) unique property keys to avoid collisions in libraries/mixins, (2) customizing language behavior via well-known symbols.

</details>

---

<details>
<summary><strong>16. What is BigInt and when should it be used?</strong></summary>

`BigInt` is a numeric type for integers **beyond `Number.MAX_SAFE_INTEGER`** (`2^53 - 1 = 9007199254740991`).

```js
// Regular numbers lose precision beyond MAX_SAFE_INTEGER
console.log(9007199254740991 + 1); // 9007199254740992
console.log(9007199254740991 + 2); // 9007199254740992 — wrong!

// BigInt handles it correctly
console.log(9007199254740991n + 1n); // 9007199254740992n
console.log(9007199254740991n + 2n); // 9007199254740993n ✅

// Creating BigInt
const big = 9999999999999999999n; // literal
const big2 = BigInt('9999999999999999999'); // from string

// Can't mix with regular numbers
1n + 1;  // ❌ TypeError
1n + 1n; // ✅ 2n
Number(1n); // 1 — explicit conversion ok
```

**When to use:**
- Financial calculations requiring exact large integers
- Cryptography (large prime numbers)
- Database IDs that exceed `MAX_SAFE_INTEGER` (e.g., Twitter snowflake IDs)
- Working with 64-bit integers from other systems

> **Interview Note:** You **cannot** mix `BigInt` and `Number` in arithmetic — explicit conversion is required. `BigInt` also doesn't work with `Math` methods.

</details>

---

## Quick Reference Cheat Sheet

**Destructuring patterns:**
```js
const { a, b: renamed, c = 'default', ...rest } = obj;
const [first, , third, ...tail] = arr;
function fn({ x, y = 0 } = {}) {}   // with default empty object
```

**Spread vs Rest:**
```
...rest  → in function params → collects into array
...spread → in calls/literals → expands out of array/object
```

**`for...of` vs `for...in`:**
```
for...of  → VALUES  → arrays, strings, Map, Set, iterables
for...in  → KEYS    → object property names (avoid for arrays)
```

**Export patterns:**
```js
export default Foo        // one per file, import with any name
export const bar = 1      // named, must import with { bar }
export { foo, bar as baz } // rename on export
import Foo, { bar } from './mod' // mixed import
```

---

*Chapter 6 of JavaScript Interview Prep Series*
