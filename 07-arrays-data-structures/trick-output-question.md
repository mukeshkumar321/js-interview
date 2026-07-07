## Chapter 7: Arrays & Data Structures — Trick Output Questions

> Self-evaluate first. Predict the output, then reveal the answer.

---

### Q1. What does `splice` return?

```js
const arr = [1, 2, 3, 4, 5];
const removed = arr.splice(1, 2);
console.log(removed);
console.log(arr);
```

<details>
<summary>Show Output & Explanation</summary>

```
[2, 3]
[1, 4, 5]
```

`splice` **mutates** the original array and **returns the removed elements**. Many developers confuse it with `slice`, which returns a new array without mutating.

</details>

---

### Q2. `slice` does not mutate

```js
const a = [1, 2, 3, 4, 5];
const b = a.slice(1, 3);
console.log(b);
console.log(a);
```

<details>
<summary>Show Output & Explanation</summary>

```
[2, 3]
[1, 2, 3, 4, 5]
```

`slice(start, end)` returns a **new array** and does not mutate the original. End index is exclusive, so `slice(1, 3)` returns elements at index 1 and 2.

</details>

---

### Q3. `sort` sorts lexicographically by default

```js
const nums = [10, 9, 2, 1, 100];
console.log(nums.sort());
```

<details>
<summary>Show Output & Explanation</summary>

```
[1, 10, 100, 2, 9]
```

`Array.prototype.sort` converts elements to **strings** and sorts lexicographically. `"10" < "2"` because `"1" < "2"`. Always pass a comparator for numeric sorting: `nums.sort((a, b) => a - b)`.

</details>

---

### Q4. `forEach` always returns `undefined`

```js
const arr = [1, 2, 3];
const result1 = arr.map(x => x * 2);
const result2 = arr.forEach(x => x * 2);
console.log(result1);
console.log(result2);
```

<details>
<summary>Show Output & Explanation</summary>

```
[2, 4, 6]
undefined
```

`map` returns a **new array** with transformed values. `forEach` is purely for side effects and always returns `undefined` — any return value from the callback is discarded.

</details>

---

### Q5. Shallow copy of nested array

```js
const original = [[1, 2], [3, 4]];
const copy = [...original];
copy[0].push(99);
console.log(original[0]);
console.log(copy[0]);
```

<details>
<summary>Show Output & Explanation</summary>

```
[1, 2, 99]
[1, 2, 99]
```

Spread `[...arr]` creates a **shallow copy**. The top-level array is new, but nested arrays are still **shared references**. Mutating `copy[0]` also mutates `original[0]`. Use `structuredClone()` for deep copying.

</details>

---

### Q6. `find` vs `filter` return type

```js
const users = [{ id: 1, name: 'Alice' }, { id: 2, name: 'Bob' }];
const found = users.find(u => u.id === 1);
const filtered = users.filter(u => u.id === 1);
console.log(found);
console.log(filtered);
```

<details>
<summary>Show Output & Explanation</summary>

```
{ id: 1, name: 'Alice' }
[{ id: 1, name: 'Alice' }]
```

`find` returns the **first matching element** (or `undefined` if not found). `filter` always returns an **array** (empty if no matches). Common mistake: using `filter` when only one item is needed.

</details>

---

### Q7. `reduce` with no initial value

```js
const arr = [1, 2, 3, 4];
const sum = arr.reduce((acc, val) => acc + val);
console.log(sum);
```

<details>
<summary>Show Output & Explanation</summary>

```
10
```

Without an initial value, `reduce` uses the **first element as the initial accumulator** and starts iteration from index 1. Execution: `1+2=3`, `3+3=6`, `6+4=10`. Always provide an initial value for clarity and safety.

</details>

---

### Q8. `reduce` on empty array without initial value throws

```js
const arr = [];
try {
  const sum = arr.reduce((acc, val) => acc + val);
  console.log(sum);
} catch (e) {
  console.log(e.message);
}
```

<details>
<summary>Show Output & Explanation</summary>

```
Reduce of empty array with no initial value
```

Calling `reduce` on an empty array **without** an initial value throws a `TypeError`. Fix: always provide an initial value — `arr.reduce((acc, val) => acc + val, 0)` returns `0` for an empty array.

</details>

---

### Q9. `Set` and `NaN` equality

```js
const set = new Set([1, '1', true, 1, true, NaN, NaN]);
console.log([...set]);
console.log(set.size);
```

<details>
<summary>Show Output & Explanation</summary>

```
[1, '1', true, NaN]
4
```

`Set` uses **SameValueZero** comparison. `1` and `'1'` are different (different types). Although `NaN !== NaN` in JS, `Set` correctly treats two `NaN` values as duplicates. So both `NaN` entries collapse to one.

</details>

---

### Q10. `Set` preserves insertion order

```js
const set = new Set([3, 1, 4, 1, 5, 9, 2, 6]);
console.log([...set]);
```

<details>
<summary>Show Output & Explanation</summary>

```
[3, 1, 4, 5, 9, 2, 6]
```

`Set` preserves **first-occurrence insertion order** and deduplicates. The duplicate `1` is removed; all other values stay in the order they first appeared.

</details>

---

### Q11. `Map` uses reference equality for object keys

```js
const map = new Map();
const obj = { id: 1 };
map.set(obj, 'Alice');
console.log(map.get({ id: 1 }));
console.log(map.get(obj));
```

<details>
<summary>Show Output & Explanation</summary>

```
undefined
'Alice'
```

`Map` uses **reference equality** for object keys. `{ id: 1 }` creates a new object with a different reference than `obj`, so `map.get({ id: 1 })` returns `undefined`. Only the exact same reference (`obj`) retrieves the value.

</details>

---

### Q12. `WeakMap` keys must be objects

```js
const wm = new WeakMap();
try {
  wm.set('string', 'value');
} catch (e) {
  console.log(e.message);
}
const key = {};
wm.set(key, 42);
console.log(wm.get(key));
```

<details>
<summary>Show Output & Explanation</summary>

```
Invalid value used as weak map key
42
```

`WeakMap` only accepts **objects** as keys — primitives throw a `TypeError`. The weak reference allows the entry to be garbage collected when the object key has no other references, preventing memory leaks.

</details>

---

### Q13. `flat` depth and `flatMap`

```js
const arr = [1, [2, 3], [4, [5, 6]]];
console.log(arr.flat());
console.log(arr.flat(2));
console.log(arr.flatMap(x => Array.isArray(x) ? x : [x]));
```

<details>
<summary>Show Output & Explanation</summary>

```
[1, 2, 3, 4, [5, 6]]
[1, 2, 3, 4, 5, 6]
[1, 2, 3, 4, [5, 6]]
```

`flat()` defaults to depth 1. `flat(2)` flattens two levels. `flatMap` is equivalent to `.map().flat(1)` — maps then flattens exactly one level. Use `flat(Infinity)` for arbitrary depth.

</details>

---

### Q14. `includes` vs `indexOf` with `NaN`

```js
const arr = [1, NaN, 3];
console.log(arr.indexOf(NaN));
console.log(arr.includes(NaN));
```

<details>
<summary>Show Output & Explanation</summary>

```
-1
true
```

`indexOf` uses strict equality (`===`), and `NaN !== NaN`, so it returns `-1`. `includes` uses **SameValueZero** and correctly identifies `NaN`. Always use `includes` when checking for `NaN` in arrays.

</details>

---

### Q15. Mutating array during `forEach`

```js
const arr = [1, 2, 3];
arr.forEach((item, index, array) => {
  if (index === 0) array.push(4);
  console.log(item);
});
```

<details>
<summary>Show Output & Explanation</summary>

```
1
2
3
```

Per the ECMAScript spec, `forEach` captures the array's length **before** iteration begins. Element `4` is pushed during the iteration at index 0, but since the original length was `3`, `forEach` only visits indices 0, 1, and 2. Elements added beyond the initial length are never visited. Avoid mutating the array inside `forEach`.

</details>

---

### Q16. Array destructuring with skip and rest

```js
const [first, , third, ...rest] = [1, 2, 3, 4, 5, 6];
console.log(first);
console.log(third);
console.log(rest);
```

<details>
<summary>Show Output & Explanation</summary>

```
1
3
[4, 5, 6]
```

The double comma `, ,` skips the second element (index 1). `third` captures index 2 (value `3`). `...rest` collects everything from index 3 onward: `[4, 5, 6]`.

</details>

---

### Q17. `Array.from` with map function

```js
const str = 'hello';
console.log(Array.from(str));
console.log([...str]);
console.log(Array.from({ length: 3 }, (_, i) => i * 2));
```

<details>
<summary>Show Output & Explanation</summary>

```
['h', 'e', 'l', 'l', 'o']
['h', 'e', 'l', 'l', 'o']
[0, 2, 4]
```

Both `Array.from` and spread convert iterables to arrays. `Array.from` uniquely accepts an **optional mapping function** as the second argument, useful for generating arrays: `Array.from({ length: 3 }, (_, i) => i * 2)` generates `[0, 2, 4]`.

</details>

---

### Q18. Deduplication with `Set` preserves order

```js
const arr = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3];
const unique = [...new Set(arr)];
console.log(unique);
```

<details>
<summary>Show Output & Explanation</summary>

```
[3, 1, 4, 5, 9, 2, 6]
```

`Set` preserves **first-occurrence order** and deduplicates in O(n) time. Duplicates `1`, `5`, and `3` are removed. Spreading back into an array is the most concise deduplication technique.

</details>

---

### Q19. `every` and `some` on empty arrays

```js
console.log([].every(x => x > 0));
console.log([].some(x => x > 0));
```

<details>
<summary>Show Output & Explanation</summary>

```
true
false
```

`every` on an empty array returns `true` (vacuous truth — no element violates the condition). `some` on an empty array returns `false` (no element satisfies the condition). Both short-circuit on the first determining element.

</details>

---

### Q20. `delete` creates a sparse hole

```js
const arr = [1, 2, 3, 4, 5];
delete arr[2];
console.log(arr);
console.log(arr.length);
console.log(arr[2]);
```

<details>
<summary>Show Output & Explanation</summary>

```
[1, 2, empty × 1, 4, 5]
5
undefined
```

`delete` sets the element to an **empty hole** — it does NOT re-index or change `length`. The array becomes sparse. To properly remove an element and shift others, use `splice(2, 1)`.

</details>

---

### Q21. Chaining array methods

```js
const result = [1, 2, 3, 4, 5]
  .filter(x => x % 2 === 0)
  .map(x => x ** 2);
console.log(result);
```

<details>
<summary>Show Output & Explanation</summary>

```
[4, 16]
```

`filter` returns `[2, 4]`, then `map` squares each: `2²=4`, `4²=16`. Each method creates a new intermediate array. For large datasets, use `reduce` to do both in a single pass and avoid intermediate allocations.

</details>

---

### Q22. `Array.isArray` vs `typeof`

```js
const arr = [1, 2, 3];
console.log(typeof arr);
console.log(Array.isArray(arr));
console.log(arr instanceof Array);
```

<details>
<summary>Show Output & Explanation</summary>

```
'object'
true
true
```

`typeof` an array is `'object'` — useless for array detection. `Array.isArray` is the reliable method — it works even across iframes (where `instanceof Array` can fail due to different `Array` constructors per realm).

</details>

---

### Q23. `reduce` to group by property

```js
const items = [
  { type: 'fruit', name: 'apple' },
  { type: 'veg', name: 'carrot' },
  { type: 'fruit', name: 'banana' },
];

const grouped = items.reduce((acc, item) => {
  (acc[item.type] = acc[item.type] || []).push(item.name);
  return acc;
}, {});

console.log(grouped);
```

<details>
<summary>Show Output & Explanation</summary>

```
{ fruit: ['apple', 'banana'], veg: ['carrot'] }
```

`reduce` builds an accumulator object. For each item, it initializes the type's array if it doesn't exist (`|| []`), then pushes the name. This **groupBy** pattern is a fundamental `reduce` use case seen in many interviews.

</details>

---

### Q24. `Map` vs plain object for computed keys

```js
const map = new Map();
map.set('constructor', 'safe');
map.set('__proto__', 'safe');

const obj = {};
obj['constructor'] = 'unsafe?';
obj['__proto__'] = 'overriding?';

console.log(map.get('constructor'));
console.log(map.size);
console.log(obj['constructor'] === Object);
```

<details>
<summary>Show Output & Explanation</summary>

```
'safe'
2
false
```

`Map` safely stores any string as a key including `'constructor'` and `'__proto__'`. With the plain object, `obj['constructor'] = 'unsafe?'` creates an **own property** named `'constructor'` that **shadows** `Object.prototype.constructor`. So `obj['constructor']` now returns the string `'unsafe?'`, not the `Object` function — making `obj['constructor'] === Object` → `false`. `Map` is safer for dynamic/arbitrary keys.

</details>

---

---

## Array.at() and Modern Methods

---

**Q25. What is the output?**

```js
const arr = [10, 20, 30, 40, 50];

console.log(arr.at(0));
console.log(arr.at(-1));
console.log(arr.at(-2));
console.log(arr[arr.length - 1] === arr.at(-1));
console.log(arr.at(10));
```

<details>
<summary>Show Output & Explanation</summary>

```
10
50
40
true
undefined
```

**Explanation:**  
`Array.at()` supports negative indices — `-1` maps to the last element, `-2` to second-last, etc. It's cleaner than `arr[arr.length - 1]`. Out-of-bounds returns `undefined`, same as bracket notation.

> **Common Mistake:** Using `arr[-1]` expecting the last element — bracket notation with `-1` looks up the property `"-1"` which doesn't exist, returning `undefined` regardless.

</details>

---

**Q26. What is the output?**

```js
const items = [
  { name: 'apple', type: 'fruit' },
  { name: 'banana', type: 'fruit' },
  { name: 'carrot', type: 'vegetable' },
  { name: 'grape', type: 'fruit' },
];

const grouped = Object.groupBy(items, item => item.type);
console.log(Object.keys(grouped));
console.log(grouped.fruit.length);
console.log(grouped.vegetable[0].name);
```

<details>
<summary>Show Output & Explanation</summary>

```
['fruit', 'vegetable']
3
carrot
```

**Explanation:**  
`Object.groupBy()` (ES2024) groups array elements by a key returned from the callback. The result is a null-prototype object where each key holds an array of matching elements. Original order within groups is preserved.

> **Common Mistake:** Using `Array.prototype.reduce` to manually group — `Object.groupBy` is now the idiomatic approach and doesn't require a polyfill in modern environments.

</details>

---

*Chapter 7 of JavaScript Interview Prep Series*
