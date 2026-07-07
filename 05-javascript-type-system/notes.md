# Chapter 5: JavaScript Type System

> **Target:** Frontend Developer (4+ years) | Interview-focused, practical explanations

---

## Table of Contents

- [Primitive vs Reference Types](#primitive-vs-reference-types)
- [Type Coercion](#type-coercion)
- [Type Checking & Modern Features](#type-checking--modern-features)

---

## Primitive vs Reference Types

<details>
<summary><strong>1. What are the different data types in JavaScript?</strong></summary>

JavaScript has **8 data types** — 7 primitives and 1 reference type.

**Primitives (immutable, stored by value):**
| Type | Example | `typeof` |
|------|---------|----------|
| `Number` | `42`, `3.14`, `NaN`, `Infinity` | `"number"` |
| `String` | `'hello'` | `"string"` |
| `Boolean` | `true`, `false` | `"boolean"` |
| `undefined` | `undefined` | `"undefined"` |
| `null` | `null` | `"object"` ⚠️ (bug) |
| `Symbol` | `Symbol('id')` | `"symbol"` |
| `BigInt` | `9007199254740991n` | `"bigint"` |

**Reference type:**
| Type | Example | `typeof` |
|------|---------|----------|
| `Object` | `{}`, `[]`, functions, `Date` | `"object"` or `"function"` |

> **Interview Note:** `typeof null === "object"` is a well-known JavaScript bug from the original implementation — `null` is not an object. Also, `typeof function(){}` returns `"function"`, not `"object"`, even though functions are objects.

</details>

---

<details>
<summary><strong>2. What is the difference between Primitive and Reference Types?</strong></summary>

| | Primitives | Reference Types |
|--|-----------|-----------------|
| **Stored** | By value (stack) | By reference (heap) |
| **Copied** | Value is duplicated | Reference (pointer) is duplicated |
| **Compared** | By value | By reference |
| **Mutable** | No — immutable | Yes — mutable |
| **Examples** | `number`, `string`, `boolean`, `null`, `undefined`, `Symbol`, `BigInt` | Objects, Arrays, Functions |

```js
// Primitive — copy by value
let a = 10;
let b = a;
b = 20;
console.log(a); // 10 — unchanged

// Reference — copy by reference
let obj1 = { x: 1 };
let obj2 = obj1;
obj2.x = 99;
console.log(obj1.x); // 99 — mutated
```

**Primitives are immutable:**
```js
let str = 'hello';
str[0] = 'H'; // silently fails
console.log(str); // 'hello' — strings are immutable
```

</details>

---

<details>
<summary><strong>3. How are Primitive Values stored in memory?</strong></summary>

Primitives are stored directly on the **call stack** as fixed-size values. When you assign a primitive to a variable, the actual value is stored — not a reference.

```js
let x = 42;   // stack: x → 42
let y = x;    // stack: y → 42 (independent copy)
y = 100;
console.log(x); // 42 — x is unaffected
```

**String interning:** JavaScript engines often reuse (intern) identical string literals to save memory:
```js
const a = 'hello';
const b = 'hello';
// Engine may store only one 'hello' in memory,
// both a and b point to the same interned string
```

**Wrapper objects:** Primitives temporarily get wrapped in objects when you access methods:
```js
'hello'.toUpperCase(); // JS creates a temporary String object, calls method, discards it
(42).toFixed(2);       // Same — temporary Number wrapper
```

</details>

---

<details>
<summary><strong>4. How are Objects and Arrays stored in memory?</strong></summary>

Objects and arrays are stored in the **heap** (dynamic memory). The variable holds a **reference (pointer)** to the heap location — not the data itself.

```
Stack             Heap
------            ----
obj → [ref] ---→  { name: 'Alice', age: 30 }
arr → [ref] ---→  [1, 2, 3, 4]
```

```js
const arr = [1, 2, 3];
// arr variable on stack holds memory address of [1, 2, 3] in heap

const arr2 = arr;  // copies the address, not the array
arr2.push(4);

console.log(arr);  // [1, 2, 3, 4] — same heap location
```

**Why `const` objects are still mutable:**
```js
const obj = { x: 1 };
obj.x = 99; // ✅ allowed — we're modifying heap data, not the reference
obj = {};   // ❌ TypeError — can't reassign the reference itself
```

</details>

---

<details>
<summary><strong>5. What is the difference between Pass by Value and Pass by Reference?</strong></summary>

JavaScript is **always pass by value** — but for objects, the value being passed is a **reference (memory address)**.

```js
// Primitives — true pass by value
function double(n) {
  n = n * 2; // local copy
}
let x = 5;
double(x);
console.log(x); // 5 — unchanged

// Objects — pass by value OF the reference
function addProp(obj) {
  obj.y = 99; // mutates the original object via the shared reference
}
const o = { x: 1 };
addProp(o);
console.log(o.y); // 99 — mutated

// Reassigning the parameter doesn't affect original
function replace(obj) {
  obj = { completely: 'new' }; // new reference, original untouched
}
const o2 = { x: 1 };
replace(o2);
console.log(o2); // { x: 1 } — unchanged
```

> **Interview Note:** The correct phrasing is *"JavaScript passes the reference by value"* — not "pass by reference." Reassigning the parameter inside a function never affects the caller's variable.

</details>

---

<details>
<summary><strong>6. What happens when you assign one object to another variable?</strong></summary>

Both variables end up pointing to the **same object in memory**. There is no copy.

```js
const user = { name: 'Alice', scores: [90, 85] };
const admin = user; // both point to same object

admin.name = 'Admin';
admin.scores.push(100);

console.log(user.name);   // 'Admin' — mutated via admin
console.log(user.scores); // [90, 85, 100] — same array

console.log(user === admin); // true — same reference
```

To get an independent copy:
```js
// Shallow copy
const copy = { ...user };
// Deep copy
const deepCopy = structuredClone(user);
```

</details>

---

<details>
<summary><strong>7. Why are Objects compared by reference instead of by value?</strong></summary>

Because objects can be arbitrarily large — comparing by value would mean recursively comparing every property, which is expensive and ambiguous (how deep do you go?). Comparing by reference is O(1).

```js
const a = { x: 1 };
const b = { x: 1 };
const c = a;

console.log(a === b); // false — different objects in memory
console.log(a === c); // true  — same reference

// To compare by value, you need a custom utility
JSON.stringify(a) === JSON.stringify(b); // true — but fragile (key order, circular refs)
```

**Deep equality in practice:**
```js
// Use a library for reliable deep equality
import { isEqual } from 'lodash';
isEqual(a, b); // true
```

> **Interview Note:** This reference comparison behavior is why React's `shouldComponentUpdate`, `useMemo`, and `useCallback` exist — they avoid expensive re-renders when references haven't changed.

</details>

---

## Type Coercion

<details>
<summary><strong>8. What is Type Coercion? Explain Implicit vs Explicit Coercion.</strong></summary>

**Type Coercion** is JavaScript automatically (or explicitly) converting a value from one type to another.

**Implicit coercion** — JavaScript converts automatically:
```js
'5' + 3       // '53'  — 3 coerced to string (+ prefers string)
'5' - 3       // 2     — '5' coerced to number (- is numeric only)
true + 1      // 2     — true → 1
false + 1     // 1     — false → 0
null + 1      // 1     — null → 0
undefined + 1 // NaN   — undefined → NaN
[] + {}       // '[object Object]'
{} + []       // 0     (tricky — as a standalone statement, {} is parsed as a block, then +[] = 0)
              // BUT: console.log({} + []) → '[object Object]' (in expression context, {} is an object literal)
```

**Explicit coercion** — you convert intentionally:
```js
Number('42')    // 42
Number(true)    // 1
Number(null)    // 0
Number('')      // 0
Number('abc')   // NaN

String(42)      // '42'
String(null)    // 'null'
String(undefined) // 'undefined'

Boolean(0)      // false
Boolean('')     // false
Boolean(null)   // false
Boolean([])     // true  — empty array is truthy!
Boolean({})     // true  — empty object is truthy!
```

> **Interview Note:** The `+` operator is the most coercion-prone. It concatenates if either operand is a string. `-`, `*`, `/` always convert to numbers.

</details>

---

<details>
<summary><strong>9. What are Truthy and Falsy values in JavaScript?</strong></summary>

**Falsy values** — exactly 8 values that coerce to `false` in a boolean context:

```js
false
0
-0
0n          // BigInt zero
''          // empty string
null
undefined
NaN
```

**Everything else is truthy**, including:
```js
[]          // empty array ✅ truthy
{}          // empty object ✅ truthy
'0'         // non-empty string ✅ truthy
'false'     // non-empty string ✅ truthy
-1          // non-zero number ✅ truthy
Infinity    // ✅ truthy
```

```js
if ([]) console.log('truthy'); // prints — arrays are always truthy

// Common pattern
const name = input || 'Default'; // uses 'Default' if input is falsy
```

> **Common Mistake:** Assuming `[]` and `{}` are falsy because they're "empty." They are objects — objects are always truthy.

</details>

---

<details>
<summary><strong>10. What is the difference between <code>==</code> and <code>===</code>?</strong></summary>

`===` (strict equality) — compares **value AND type**, no coercion.  
`==` (loose equality) — compares after performing **type coercion**.

```js
// === — no surprises
1 === 1       // true
1 === '1'     // false
null === undefined // false

// == — coercion rules apply
1 == '1'      // true  — '1' coerced to 1
0 == false    // true  — false coerced to 0
0 == ''       // true  — '' coerced to 0
null == undefined // true  — special rule
null == 0     // false — null only == undefined
NaN == NaN    // false — NaN is never equal to anything
```

**Abstract Equality (`==`) rules:**
1. Same type → compare like `===`
2. `null == undefined` → `true`
3. `number == string` → convert string to number
4. `boolean == anything` → convert boolean to number first
5. `object == primitive` → call `valueOf()`/`toString()` on object

> **Interview Note:** Always use `===` in production. The only valid use of `==` is `value == null` which catches both `null` and `undefined` in one check — many style guides permit this specific pattern.

</details>

---

<details>
<summary><strong>11. What is the difference between <code>null</code> and <code>undefined</code>?</strong></summary>

| | `null` | `undefined` |
|--|--------|-------------|
| **Meaning** | Intentional absence of value | Variable declared but not assigned |
| **Set by** | Developer explicitly | JavaScript engine |
| **typeof** | `"object"` (bug) | `"undefined"` |
| **In JSON** | Preserved | Stripped out |
| **Default param** | Does NOT trigger default | Triggers default |

```js
let a;          // undefined — JS sets this
let b = null;   // null — developer sets this intentionally

console.log(typeof a); // 'undefined'
console.log(typeof b); // 'object' — famous bug

// Both are loosely equal
null == undefined;  // true
null === undefined; // false

// null does NOT trigger default params
function foo(x = 'default') { return x; }
foo(undefined); // 'default'
foo(null);      // null — null is a real value here
```

> **Interview Note:** Use `null` to intentionally clear a value (e.g., `ref.current = null`). `undefined` typically signals "not yet set" or "missing." Never explicitly assign `undefined` — use `null` for intentional emptiness.

</details>

---

<details>
<summary><strong>12. Why does <code>typeof null</code> return <code>"object"</code>?</strong></summary>

This is a **bug from JavaScript's original implementation (1995)** that was never fixed to preserve backward compatibility.

In the original engine, values were stored with a type tag in the first bits of a 32-bit word. The tag for objects was `000`. `null` was represented as a null pointer (`0x00`) — which also had `000` bits — so it was misidentified as an object.

```js
typeof null        // "object" — bug, not a feature
typeof undefined   // "undefined"
typeof {}          // "object"
typeof []          // "object"
typeof function(){} // "function"
```

**How to correctly check for null:**
```js
value === null       // ✅ only safe way
typeof value === 'object' && value !== null // ✅ object but not null
```

> **Interview Note:** This is one of the most famous JavaScript quirks. A proposal to fix it (`typeof null === "null"`) was rejected because it would break too much existing code.

</details>

---

---

## Floating Point Precision

### Why 0.1 + 0.2 !== 0.3

JavaScript uses **IEEE 754 double-precision** floating point. Most decimal fractions (0.1, 0.2, 0.3) cannot be represented exactly in binary — they're stored as approximations.

```js
console.log(0.1 + 0.2);         // 0.30000000000000004
console.log(0.1 + 0.2 === 0.3); // false
```

### The Fix: Number.EPSILON

`Number.EPSILON` is the smallest difference between two representable doubles (~2.22e-16). Use it for float comparisons:

```js
function approximately(a, b) {
  return Math.abs(a - b) < Number.EPSILON;
}

approximately(0.1 + 0.2, 0.3); // true
```

### Practical Fixes

```js
// For display
(0.1 + 0.2).toFixed(2);        // "0.30"

// For money — use integers (cents, not dollars)
const price = 199; // $1.99 stored as 199 cents
```

> **Interview tip:** "Never use floating point for money calculations. Store amounts as integers (cents) or use a library like decimal.js."

---

<details>
<summary><strong>13. What is <code>NaN</code> and why is <code>NaN !== NaN</code>?</strong></summary>

`NaN` stands for **Not a Number** — it's the result of an invalid numeric operation. Despite its name, `typeof NaN === "number"`.

```js
0 / 0           // NaN
parseInt('abc') // NaN
Math.sqrt(-1)   // NaN
undefined + 1   // NaN
```

**Why `NaN !== NaN`:**  
This follows the **IEEE 754 floating-point standard** — `NaN` represents an undefined or unrepresentable value. Two undefined results are not necessarily the same undefined result, so they can't be equal. This is by design, not a bug.

```js
NaN === NaN  // false
NaN == NaN   // false

// How to check for NaN
Number.isNaN(NaN)      // true  ✅ reliable
Number.isNaN('abc')    // false ✅ (not a number, but also not NaN)
isNaN('abc')           // true  ⚠️ coerces first — unreliable
Number.isNaN(undefined)// false ✅

// Also works
Object.is(NaN, NaN)    // true
```

</details>

---

<details>
<summary><strong>14. What is the difference between <code>isNaN()</code> and <code>Number.isNaN()</code>?</strong></summary>

| | `isNaN()` | `Number.isNaN()` |
|--|-----------|-----------------|
| **Coerces first?** | Yes | No |
| **Reliable?** | No | Yes |
| **Added** | ES1 (legacy) | ES6 |

```js
isNaN(NaN)         // true  ✅
isNaN('hello')     // true  ⚠️ — 'hello' coerced to NaN first, then checked
isNaN(undefined)   // true  ⚠️ — undefined → NaN → true
isNaN('123')       // false — '123' coerced to 123, not NaN
isNaN(null)        // false — null coerced to 0, not NaN

Number.isNaN(NaN)       // true  ✅
Number.isNaN('hello')   // false ✅ — no coercion, 'hello' is not NaN type
Number.isNaN(undefined) // false ✅ — no coercion
Number.isNaN(123)       // false ✅
```

> **Interview Note:** Always use `Number.isNaN()`. The global `isNaN()` is a footgun — it coerces the argument to a number first, giving false positives for strings and `undefined`.

</details>

---

<details>
<summary><strong>15. What is <code>Object.is()</code> and how is it different from <code>==</code> and <code>===</code>?</strong></summary>

`Object.is()` is like `===` but handles two special edge cases differently:

| Comparison | `==` | `===` | `Object.is()` |
|-----------|------|-------|--------------|
| `NaN vs NaN` | `false` | `false` | **`true`** |
| `+0 vs -0` | `true` | `true` | **`false`** |
| `null vs undefined` | `true` | `false` | `false` |
| `1 vs '1'` | `true` | `false` | `false` |

```js
NaN === NaN       // false
Object.is(NaN, NaN) // true ✅

+0 === -0         // true
Object.is(+0, -0) // false ✅ — distinguishes signed zeros

Object.is(1, 1)   // true
Object.is(null, null) // true
```

**When to use `Object.is()`:**  
In utility functions that need precise equality (e.g., React's reconciler uses it internally to detect state changes — `Object.is(prevState, nextState)`).

</details>

---

## Type Checking & Modern Features

<details>
<summary><strong>16. How does the <code>typeof</code> operator work and what are its limitations?</strong></summary>

`typeof` returns a string indicating the type of a value. It's evaluated at runtime and works on undeclared variables without throwing.

```js
typeof 42            // "number"
typeof 'hello'       // "string"
typeof true          // "boolean"
typeof undefined     // "undefined"
typeof Symbol()      // "symbol"
typeof 42n           // "bigint"
typeof function(){}  // "function"

// Limitations:
typeof null          // "object" — bug
typeof []            // "object" — can't distinguish array from object
typeof {}            // "object"
typeof new Date()    // "object"

// Safe to use on undeclared variables
typeof undeclaredVar // "undefined" — no ReferenceError
```

**For reliable type checking:**
```js
Array.isArray([])                        // true — for arrays
Object.prototype.toString.call(null)     // "[object Null]"
Object.prototype.toString.call([])       // "[object Array]"
Object.prototype.toString.call(new Date()) // "[object Date]"
```

> **Interview Note:** `Object.prototype.toString.call(value)` is the most reliable way to get the exact type tag of any value in JavaScript.

</details>

---

<details>
<summary><strong>17. What is the difference between <code>typeof</code>, <code>instanceof</code>, and <code>Array.isArray()</code>?</strong></summary>

| Method | Best for | Limitation |
|--------|----------|------------|
| `typeof` | Primitives | Can't distinguish null/array/object |
| `instanceof` | Class instances | Fails across iframes/realms |
| `Array.isArray()` | Arrays specifically | Only for arrays |

```js
const arr = [1, 2, 3];
const obj = { a: 1 };
const fn  = function(){};

typeof arr        // "object" — not helpful
typeof obj        // "object"
typeof fn         // "function"

arr instanceof Array   // true
obj instanceof Object  // true
arr instanceof Object  // true — arrays are objects

Array.isArray(arr)     // true  ✅
Array.isArray(obj)     // false ✅

// Cross-realm issue
// An array from an iframe:
arr instanceof Array   // false — different Array constructor
Array.isArray(arr)     // true  — works regardless of realm
```

> **Interview Note:** `Array.isArray()` is the **only reliable way** to check for arrays. Use it over `instanceof Array` in any code that might run with iframes or across module boundaries.

</details>

---

<details>
<summary><strong>18. What are Optional Chaining (<code>?.</code>) and Nullish Coalescing (<code>??</code>)?</strong></summary>

**Optional Chaining (`?.`)** — safely access nested properties without throwing if an intermediate value is `null` or `undefined`.

```js
const user = { address: { city: 'NY' } };

// Without ?.
user && user.address && user.address.city; // 'NY'
user && user.phone && user.phone.number;   // undefined

// With ?.
user?.address?.city;   // 'NY'
user?.phone?.number;   // undefined — no throw
user?.greet?.();       // undefined — method doesn't exist, no throw

// Also works with arrays
const first = arr?.[0]; // undefined if arr is null/undefined
```

**Nullish Coalescing (`??`)** — returns the right side only if the left is `null` or `undefined` (not other falsy values).

```js
null ?? 'default'      // 'default'
undefined ?? 'default' // 'default'
0 ?? 'default'         // 0     — 0 is not null/undefined
'' ?? 'default'        // ''    — '' is not null/undefined
false ?? 'default'     // false — false is not null/undefined
```

**Combining both:**
```js
const port = config?.server?.port ?? 3000;
```

</details>

---

<details>
<summary><strong>19. What is the difference between <code>||</code> and <code>??</code>?</strong></summary>

| Operator | Returns right side when left is... | Triggers on falsy `0`, `''`, `false`? |
|----------|------------------------------------|---------------------------------------|
| `\|\|` | Any **falsy** value | Yes |
| `??` | **`null` or `undefined` only** | No |

```js
// || uses falsy check
0     || 'default' // 'default' ⚠️ — 0 is falsy
''    || 'default' // 'default' ⚠️ — '' is falsy
false || 'default' // 'default' ⚠️ — false is falsy
null  || 'default' // 'default' ✅
undefined || 'default' // 'default' ✅

// ?? uses nullish check
0     ?? 'default' // 0         ✅ — 0 is a valid value
''    ?? 'default' // ''        ✅ — '' is a valid value
false ?? 'default' // false     ✅ — false is a valid value
null  ?? 'default' // 'default' ✅
undefined ?? 'default' // 'default' ✅
```

**Real-world bug with `||`:**
```js
function setVolume(vol) {
  const volume = vol || 50; // ❌ if vol = 0, volume becomes 50!
  const volume = vol ?? 50; // ✅ if vol = 0, volume stays 0
}
```

> **Interview Note:** `??` was introduced specifically to fix the `||` problem with valid falsy values like `0`, `''`, and `false`. Use `??` for default values, `||` for boolean logic.

</details>

---

<details>
<summary><strong>20. What are common bugs caused by automatic type coercion?</strong></summary>

**Bug 1: `+` operator concatenates instead of adding**
```js
function add(a, b) { return a + b; }
add('5', 3); // '53' — not 8
// Fix: Number(a) + Number(b)
```

**Bug 2: Falsy value swallowed by `||`**
```js
const count = userCount || 10; // ❌ if userCount = 0, uses 10
const count = userCount ?? 10; // ✅
```

**Bug 3: Loose equality surprises**
```js
0 == ''       // true
0 == '0'      // true
'' == '0'     // false — all three can't be equal if transitivity held
null == false // false — null only == undefined
```

**Bug 4: `parseInt` parsing stops at non-numeric**
```js
parseInt('10px') // 10 — not NaN
parseInt('px10') // NaN
```

### parseInt Gotchas

```js
// Always pass a radix
parseInt('08');        // 8 (modern) — always specify radix to be safe
parseInt('08', 10);    // 8 (explicit decimal)
parseInt('0x10');      // 16 (auto hex detection)
parseInt('10', 2);     // 2 (binary)
parseInt('3.9');       // 3 (truncates, doesn't round)

// Classic trap — never pass parseInt directly to .map()
['1', '2', '3'].map(parseInt);
// Calls: parseInt('1', 0), parseInt('2', 1), parseInt('3', 2)
// Returns: [1, NaN, NaN]

// Fix:
['1', '2', '3'].map(Number);  // [1, 2, 3]
['1', '2', '3'].map(n => parseInt(n, 10)); // [1, 2, 3]
```

**Bug 5: Array/object coercion in arithmetic**
```js
[] + []   // '' — both coerce to ''
[] + {}   // '[object Object]'
{} + []   // 0  — {} as block, +[] = 0
+[]       // 0
+{}       // NaN
```

**Bug 6: `sort()` default lexicographic sort**
```js
[10, 9, 2, 1, 100].sort() // [1, 10, 100, 2, 9] — sorted as strings!
[10, 9, 2, 1, 100].sort((a, b) => a - b) // [1, 2, 9, 10, 100] ✅
```

> **Interview Note:** Knowing these coercion bugs by heart signals deep JS understanding. Always mention `===`, `Number.isNaN()`, and `??` as modern mitigations.

</details>

---

## Quick Reference Cheat Sheet

**Falsy values (exactly 8):**
```
false | 0 | -0 | 0n | '' | null | undefined | NaN
```

**`typeof` results:**
```
"number" | "string" | "boolean" | "undefined" | "symbol" | "bigint" | "object" | "function"
typeof null → "object" (bug)
typeof [] → "object" (use Array.isArray)
```

**Equality quick reference:**
```
null == undefined  → true    null === undefined → false
NaN == NaN         → false   Object.is(NaN,NaN) → true
+0 === -0          → true    Object.is(+0,-0)   → false
[] == false        → true    [] === false        → false
```

**Default value operators:**
```
value || default  → right side if value is ANY falsy
value ?? default  → right side ONLY if value is null/undefined
value?.prop       → undefined (not throw) if value is null/undefined
```

---

*Chapter 5 of JavaScript Interview Prep Series*
