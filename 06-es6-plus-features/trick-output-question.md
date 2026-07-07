## Chapter 6: Trick Output Questions — ES6+ Features

> Try to predict the output **before** expanding the answer. Each question reflects a real interview scenario.

---

## var, let, const

---

**Q1. What is the output?**

```js
const obj = { x: 1, y: 2 };
obj.x = 99;
obj.z = 3;
console.log(obj);

obj = { x: 1 };
console.log(obj);
```

<details>
<summary>Show Output & Explanation</summary>

```
{ x: 99, y: 2, z: 3 }
TypeError: Assignment to constant variable.
```

**Explanation:**  
`const` makes the **binding** immutable, not the object itself. Mutating properties (`obj.x = 99`, `obj.z = 3`) works fine because the reference doesn't change. Reassigning the variable (`obj = {...}`) changes the binding — that throws a `TypeError`.

</details>

---

**Q2. What is the output?**

```js
const arr = [1, 2, 3];
arr.push(4);
arr[0] = 99;
console.log(arr);

arr = [1, 2, 3];
```

<details>
<summary>Show Output & Explanation</summary>

```
[99, 2, 3, 4]
TypeError: Assignment to constant variable.
```

**Explanation:**  
Same as objects — `const` only protects the reference. Mutating the array's contents (`push`, index assignment) is allowed. Reassigning `arr` to a new array throws.

</details>

---

## Destructuring

---

**Q3. What is the output?**

```js
const { a, b = 10, c: renamed } = { a: 1, c: 99 };
console.log(a);
console.log(b);
console.log(renamed);
console.log(c);
```

<details>
<summary>Show Output & Explanation</summary>

```
1
10
99
ReferenceError: c is not defined
```

**Explanation:**  
- `a` → `1` (own property)
- `b = 10` → not in object → uses default `10`
- `c: renamed` → renames `c` to `renamed` → `99`
- `c` → the original key `c` is NOT created as a variable when renamed → `ReferenceError`

</details>

---

**Q4. What is the output?**

```js
const [a, b = 5, , c] = [1, undefined, 3, 4];
console.log(a);
console.log(b);
console.log(c);
```

<details>
<summary>Show Output & Explanation</summary>

```
1
5
4
```

**Explanation:**  
- `a` → position 0 → `1`
- `b = 5` → position 1 is `undefined` → default `5` applies
- `,` → position 2 is skipped (`3` ignored)
- `c` → position 3 → `4`

</details>

---

**Q5. What is the output?**

```js
const obj = { a: { b: { c: 42 } } };
const { a: { b: { c } } } = obj;

console.log(c);
console.log(a);
console.log(b);
```

<details>
<summary>Show Output & Explanation</summary>

```
42
ReferenceError: a is not defined
ReferenceError: b is not defined
```

**Explanation:**  
Nested destructuring creates only the **final** variable — in this case `c`. The intermediate names `a` and `b` are just path descriptors (like `c: renamed` syntax), not variables. Only `c` is declared.

</details>

---

**Q6. What is the output?**

```js
let a = 1, b = 2;
console.log(a, b);

[a, b] = [b, a];
console.log(a, b);
```

<details>
<summary>Show Output & Explanation</summary>

```
1 2
2 1
```

**Explanation:**  
Array destructuring swap — `[b, a]` creates a temporary array `[2, 1]`, then destructures: `a = 2`, `b = 1`. No temporary variable needed. This is one of the cleanest real-world uses of array destructuring.

</details>

---

**Q7. What is the output?**

```js
function greet({ name = 'Guest', role = 'user' } = {}) {
  console.log(`${name} — ${role}`);
}

greet({ name: 'Alice', role: 'admin' });
greet({ name: 'Bob' });
greet();
greet(null);
```

<details>
<summary>Show Output & Explanation</summary>

```
Alice — admin
Bob — user
Guest — user
TypeError: Cannot read properties of null (reading 'name')
```

**Explanation:**  
- First call → both provided → `'Alice — admin'`
- Second call → `role` missing → default `'user'`
- Third call → no argument → `= {}` default kicks in → both defaults
- `greet(null)` → `null` does NOT trigger the default param (only `undefined` does) → destructuring `null` throws `TypeError`

> **Production pattern:** `= {}` on the parameter makes the entire object optional. But `null` still breaks it.

</details>

---

## Spread & Rest

---

**Q8. What is the output?**

```js
function sum(a, b, ...rest) {
  console.log(a);
  console.log(b);
  console.log(rest);
  return rest.reduce((acc, n) => acc + n, a + b);
}

console.log(sum(1, 2, 3, 4, 5));
```

<details>
<summary>Show Output & Explanation</summary>

```
1
2
[3, 4, 5]
15
```

**Explanation:**  
`a = 1`, `b = 2`, `rest = [3, 4, 5]`. `reduce` starts with `a + b = 3` and adds `3 + 4 + 5 = 12` → total `15`.

</details>

---

**Q9. What is the output?**

```js
const a = [1, 2, 3];
const b = [...a];

b.push(4);
b[0] = 99;

console.log(a);
console.log(b);
```

<details>
<summary>Show Output & Explanation</summary>

```
[1, 2, 3]
[99, 2, 3, 4]
```

**Explanation:**  
`[...a]` creates a **shallow copy**. For a flat array of primitives, this behaves like a deep copy — `b` gets its own independent values. Changes to `b` don't affect `a`.

</details>

---

**Q10. What is the output?**

```js
const a = [{ x: 1 }, { x: 2 }];
const b = [...a];

b[0].x = 99;
b.push({ x: 3 });

console.log(a);
console.log(b);
```

<details>
<summary>Show Output & Explanation</summary>

```
[{ x: 99 }, { x: 2 }]
[{ x: 99 }, { x: 2 }, { x: 3 }]
```

**Explanation:**  
Spread is **shallow**. `b` is a new array, but the objects inside are still the same references. Mutating `b[0].x` mutates `a[0].x` too. `b.push({ x: 3 })` only adds to `b`'s array — doesn't affect `a`.

</details>

---

**Q11. What is the output?**

```js
const defaults = { color: 'blue', size: 'M', weight: 10 };
const custom   = { size: 'L', weight: 20 };
const result   = { ...defaults, ...custom, height: 5 };

console.log(result);
```

<details>
<summary>Show Output & Explanation</summary>

```
{ color: 'blue', size: 'L', weight: 20, height: 5 }
```

**Explanation:**  
Later spread properties **override** earlier ones. `defaults` provides `color`, `size`, `weight`. `custom` overrides `size` and `weight`. `height: 5` is added last. This is the standard object merge/override pattern in modern JS.

</details>

---

**Q12. What is the output?**

```js
const obj = { a: 1, b: 2, c: 3 };
const { a, ...rest } = obj;

console.log(a);
console.log(rest);
console.log(obj);
```

<details>
<summary>Show Output & Explanation</summary>

```
1
{ b: 2, c: 3 }
{ a: 1, b: 2, c: 3 }
```

**Explanation:**  
Rest in destructuring collects all remaining properties into a new object. `a = 1`, `rest = { b: 2, c: 3 }`. The original `obj` is untouched — rest creates a new shallow copy of the remaining keys.

</details>

---

## Arrow Functions

---

**Q13. What is the output?**

```js
const obj = {
  value: 42,
  regular: function() { return this.value; },
  arrow: () => this.value
};

console.log(obj.regular());
console.log(obj.arrow());
```

<details>
<summary>Show Output & Explanation</summary>

```
42
undefined
```

**Explanation:**  
- `regular()` → method call, `this = obj` → `obj.value = 42`
- `arrow()` → no own `this`, inherits from enclosing scope (global/module) → `this.value = undefined`

Arrow functions are wrong for object methods that need `this`.

</details>

---

**Q14. What is the output?**

```js
const makeMultiplier = (x) => (y) => x * y;

const double = makeMultiplier(2);
const triple = makeMultiplier(3);

console.log(double(5));
console.log(triple(5));
console.log(makeMultiplier(4)(5));
```

<details>
<summary>Show Output & Explanation</summary>

```
10
15
20
```

**Explanation:**  
Curried arrow functions. `makeMultiplier(2)` returns `y => 2 * y`, which is `double`. Each call to `makeMultiplier` creates a new closure with its own `x`. `makeMultiplier(4)(5)` → `4 * 5 = 20`.

</details>

---

**Q15. What is the output?**

```js
const fn1 = () => { return { x: 1 }; };
const fn2 = () => ({ x: 1 });
const fn3 = () => { x: 1 };

console.log(fn1());
console.log(fn2());
console.log(fn3());
```

<details>
<summary>Show Output & Explanation</summary>

```
{ x: 1 }
{ x: 1 }
undefined
```

**Explanation:**  
- `fn1` → explicit `return` with block body → `{ x: 1 }`
- `fn2` → object literal wrapped in `()` → concise return → `{ x: 1 }`
- `fn3` → `{ x: 1 }` is parsed as a **block** with a labeled statement `x: 1` (not an object) → no `return` → `undefined`

> **Common bug:** Forgetting to wrap the returned object literal in `()` when using concise arrow functions.

</details>

---

## Template Literals

---

**Q16. What is the output?**

```js
const a = 5;
const b = 10;

console.log(`Sum: ${a + b}`);
console.log(`Product: ${a * b}`);
console.log(`Ternary: ${a > b ? 'a is bigger' : 'b is bigger'}`);
console.log(`Nested: ${`inner ${a}`}`);
```

<details>
<summary>Show Output & Explanation</summary>

```
Sum: 15
Product: 50
Ternary: b is bigger
Nested: inner 5
```

**Explanation:**  
Template literals evaluate any valid JS expression inside `${}` — arithmetic, ternary, even nested template literals. The result is coerced to a string and interpolated.

</details>

---

**Q17. What is the output?**

```js
function tag(strings, ...values) {
  console.log(strings);
  console.log(values);
  return strings.raw[0];
}

const result = tag`Hello\nWorld ${42} !`;
console.log(result);
```

<details>
<summary>Show Output & Explanation</summary>

```
['Hello\nWorld ', ' !']   // (processed — \n is a newline)
[42]
Hello\nWorld              // (raw — \n is literal backslash-n)
```

**Explanation:**  
Tagged templates pass: `strings` (processed string parts array) and spread `values` (interpolated values). `strings.raw` contains the **raw unprocessed** strings — `\n` is literally `\` + `n`, not a newline. `result = strings.raw[0]` → the raw first part.

</details>

---

## for...of vs for...in

---

**Q18. What is the output?**

```js
const arr = [10, 20, 30];
arr.custom = 'hello';

for (const val of arr) {
  console.log(val);
}

console.log('---');

for (const key in arr) {
  console.log(key);
}
```

<details>
<summary>Show Output & Explanation</summary>

```
10
20
30
---
0
1
2
custom
```

**Explanation:**  
- `for...of` → iterates **values** using the iterator protocol → `10, 20, 30` (ignores non-index properties)
- `for...in` → iterates **enumerable property keys** → `'0', '1', '2'` (indices as strings) + `'custom'` (own enumerable property added)

> **Classic trap.** Adding a property to an array pollutes `for...in` but not `for...of`.

</details>

---

**Q19. What is the output?**

```js
const obj = { a: 1, b: 2, c: 3 };

for (const val of obj) {
  console.log(val);
}
```

<details>
<summary>Show Output & Explanation</summary>

```
TypeError: obj is not iterable
```

**Explanation:**  
Plain objects are **not iterable** — they don't implement the `[Symbol.iterator]` protocol. `for...of` only works with iterables: arrays, strings, Maps, Sets, generators, etc.

**Fix:**
```js
for (const [key, val] of Object.entries(obj)) {
  console.log(key, val); // 'a' 1, 'b' 2, 'c' 3
}
```

</details>

---

## Symbols & BigInt

---

**Q20. What is the output?**

```js
const s1 = Symbol('desc');
const s2 = Symbol('desc');

console.log(s1 === s2);
console.log(s1.toString());
console.log(typeof s1);
console.log(s1.description);
```

<details>
<summary>Show Output & Explanation</summary>

```
false
Symbol(desc)
symbol
desc
```

**Explanation:**  
Every `Symbol()` is unique — even with the same description. `s1 === s2` → `false`. `toString()` → `'Symbol(desc)'`. `typeof` → `'symbol'`. `.description` (ES2019) → the string `'desc'` without the `Symbol()` wrapper.

</details>

---

**Q21. What is the output?**

```js
const ID = Symbol('id');
const obj = {
  name: 'Alice',
  [ID]: 123
};

console.log(obj[ID]);
console.log(obj.ID);
console.log(Object.keys(obj));
console.log(JSON.stringify(obj));
```

<details>
<summary>Show Output & Explanation</summary>

```
123
undefined
['name']
{"name":"Alice"}
```

**Explanation:**  
- `obj[ID]` → Symbol key access → `123`
- `obj.ID` → looks for string property `'ID'` → `undefined`
- `Object.keys()` → only returns string enumerable keys → `['name']`
- `JSON.stringify()` → Symbol keys are completely ignored → `{"name":"Alice"}`

> Symbol keys are "hidden" — they don't show up in `Object.keys`, `for...in`, or JSON serialization. Use `Object.getOwnPropertySymbols()` to access them.

</details>

---

**Q22. What is the output?**

```js
console.log(9007199254740991 + 1);
console.log(9007199254740991 + 2);

console.log(9007199254740991n + 1n);
console.log(9007199254740991n + 2n);

console.log(typeof 42n);
console.log(1n + 1);
```

<details>
<summary>Show Output & Explanation</summary>

```
9007199254740992
9007199254740992
9007199254740992n
9007199254740993n
bigint
TypeError: Cannot mix BigInt and other types
```

**Explanation:**  
- Regular `Number` loses precision beyond `MAX_SAFE_INTEGER` — both `+1` and `+2` give the same wrong result.
- `BigInt` handles arbitrary precision — `+1n` and `+2n` give correct distinct results.
- `typeof 42n` → `'bigint'`
- `1n + 1` → mixing `BigInt` and `Number` → `TypeError` — explicit conversion required: `1n + BigInt(1)`

</details>

---

## Mixed / Hard

---

**Q23. What is the output?**

```js
const add = (...args) => args.reduce((a, b) => a + b, 0);

const nums = [1, 2, 3, 4, 5];
console.log(add(...nums));
console.log(add(10, ...nums, 20));

const nested = [[1, 2], [3, 4], [5]];
console.log(add(...nested.flat()));
```

<details>
<summary>Show Output & Explanation</summary>

```
15
45
15
```

**Explanation:**  
- `add(...nums)` → `add(1, 2, 3, 4, 5)` → `15`
- `add(10, ...nums, 20)` → `add(10, 1, 2, 3, 4, 5, 20)` → `45`
- `nested.flat()` → `[1, 2, 3, 4, 5]`, then spread → `add(1, 2, 3, 4, 5)` → `15`

</details>

---

**Q24. What is the output?**

```js
const person = {
  name: 'Alice',
  hobbies: ['reading', 'coding'],
  address: { city: 'NY' }
};

const { name, hobbies: [first, ...otherHobbies], address: { city } } = person;

console.log(name);
console.log(first);
console.log(otherHobbies);
console.log(city);
console.log(hobbies);
```

<details>
<summary>Show Output & Explanation</summary>

```
Alice
reading
['coding']
NY
ReferenceError: hobbies is not defined
```

**Explanation:**  
- `name` → `'Alice'`
- `hobbies: [first, ...otherHobbies]` → renames `hobbies` and simultaneously destructures the array → `first = 'reading'`, `otherHobbies = ['coding']`
- `address: { city }` → nested object destructuring → `city = 'NY'`
- `hobbies` → was used as a path descriptor (renamed), NOT declared as a variable → `ReferenceError`

> **Complex destructuring.** The intermediate names in rename-style destructuring (`hobbies:`, `address:`) are paths, not variable declarations.

</details>

---

## Promise.allSettled and Promise.any

---

**Q25. What is the output?**

```js
const p1 = Promise.resolve('success');
const p2 = Promise.reject('error');
const p3 = Promise.resolve('also success');

Promise.allSettled([p1, p2, p3]).then(results => {
  results.forEach(r => console.log(r.status, r.value ?? r.reason));
});
```

<details>
<summary>Show Output & Explanation</summary>

```
fulfilled success
rejected error
fulfilled also success
```

**Explanation:**  
`Promise.allSettled` waits for all promises regardless of outcome. Each result has `status: 'fulfilled'` with `value`, or `status: 'rejected'` with `reason`. It never short-circuits — you always get one result per promise.

> **Common Mistake:** Using `Promise.all` when you want to run multiple independent operations and handle each result — `Promise.all` rejects immediately on the first failure, losing all other results.

</details>

---

**Q26. What is the output?**

```js
const p1 = Promise.reject('err1');
const p2 = Promise.reject('err2');
const p3 = Promise.resolve('first success');
const p4 = Promise.resolve('second success');

Promise.any([p1, p2, p3, p4])
  .then(v => console.log('resolved:', v))
  .catch(e => console.log('all rejected:', e.message));
```

<details>
<summary>Show Output & Explanation</summary>

```
resolved: first success
```

**Explanation:**  
`Promise.any` resolves with the first fulfilled promise, ignoring rejections. `p1` and `p2` reject, but `p3` fulfills — so `'first success'` wins. If all promises had rejected, it would throw an `AggregateError`.

> **Common Mistake:** Confusing `Promise.any` with `Promise.race` — `race` resolves/rejects with the FIRST settled promise (including rejections), while `any` only resolves on fulfilment and needs ALL to reject before it rejects.

</details>

---

## Object.fromEntries

---

**Q27. What is the output?**

```js
const entries = [['a', 1], ['b', 2], ['c', 3]];
const obj = Object.fromEntries(entries);
console.log(obj);

const original = { x: 10, y: 20, z: 30 };
const doubled = Object.fromEntries(
  Object.entries(original).map(([k, v]) => [k, v * 2])
);
console.log(doubled);
```

<details>
<summary>Show Output & Explanation</summary>

```
{ a: 1, b: 2, c: 3 }
{ x: 20, y: 40, z: 60 }
```

**Explanation:**  
`Object.fromEntries()` converts an iterable of `[key, value]` pairs into an object — it's the inverse of `Object.entries()`. The `entries → map → fromEntries` pattern is the idiomatic way to transform object values without mutation.

> **Common Mistake:** Using `reduce` to rebuild an object after `Object.entries().map()` — `Object.fromEntries` is cleaner and more readable.

</details>

---

**Q28. What is the output?**

```js
const map = new Map([['name', 'Alice'], ['age', 30], ['role', 'dev']]);
const obj = Object.fromEntries(map);
console.log(obj.name);
console.log(typeof obj);
console.log(obj instanceof Map);
```

<details>
<summary>Show Output & Explanation</summary>

```
Alice
object
false
```

**Explanation:**  
`Object.fromEntries` also accepts a `Map` directly (Maps are iterable of `[key, value]` pairs). The result is a plain object, not a Map — `instanceof Map` is `false`.

> **Common Mistake:** Trying to use spread (`{...map}`) to convert a Map to an object — spread on a Map doesn't work as expected because Maps aren't plain iterables in the key-value sense for spread syntax.

</details>

---

## Logical Assignment Operators

---

**Q29. What is the output?**

```js
let a = null;
let b = 0;
let c = 'hello';

a ??= 'default';
b ||= 'fallback';
c &&= c.toUpperCase();

console.log(a);
console.log(b);
console.log(c);
```

<details>
<summary>Show Output & Explanation</summary>

```
default
fallback
HELLO
```

**Explanation:**  
- `a ??= 'default'` → `a` was `null` (nullish) → assigns `'default'`
- `b ||= 'fallback'` → `b` was `0` (falsy) → assigns `'fallback'`
- `c &&= c.toUpperCase()` → `c` was `'hello'` (truthy) → replaces with `'HELLO'`

> **Common Mistake:** Treating `??=` and `||=` as identical. `b = 0; b ??= 'x'` leaves `b` as `0` (0 is not nullish), but `b ||= 'x'` changes it to `'x'` (0 is falsy).

</details>

---

**Q30. What is the output?**

```js
const config = { debug: false, timeout: 0, name: '' };

config.debug ??= true;
config.timeout ??= 5000;
config.name ??= 'default';

console.log(config.debug);
console.log(config.timeout);
console.log(config.name);
```

<details>
<summary>Show Output & Explanation</summary>

```
false
0

```

**Explanation:**  
None of the properties are `null` or `undefined` — they're `false`, `0`, and `''`, which are valid values. `??=` only triggers on `null`/`undefined`, so nothing changes. This demonstrates why `??=` is safer than `||=` for config defaults — it respects intentional falsy values.

> **Common Mistake:** Using `config.debug = config.debug ?? true` (verbose) vs `config.debug ??= true` (concise). They're equivalent, but `??=` is the modern shorthand.

</details>

---

## Proxy and Reflect

---

**Q31. What is the output?**

```js
const handler = {
  get(target, key) {
    return key in target ? target[key] : `Property '${key}' not found`;
  },
  set(target, key, value) {
    if (typeof value !== 'number') throw new TypeError('Only numbers allowed');
    target[key] = value;
    return true;
  }
};

const obj = new Proxy({}, handler);
obj.x = 42;
console.log(obj.x);
console.log(obj.y);

try {
  obj.z = 'hello';
} catch (e) {
  console.log(e.message);
}
```

<details>
<summary>Show Output & Explanation</summary>

```
42
Property 'y' not found
Only numbers allowed
```

**Explanation:**  
`Proxy` wraps an object and intercepts operations via traps. The `get` trap returns a custom message for missing keys instead of `undefined`. The `set` trap validates the value type before storing. `return true` in `set` is required — omitting it causes a `TypeError` in strict mode.

> **Common Mistake:** Forgetting `return true` in the `set` trap. Without it, the proxy throws `TypeError: 'set' on proxy: trap returned falsish`.

</details>

---

*Chapter 6 — Trick Output Questions | ES6+ Features*
