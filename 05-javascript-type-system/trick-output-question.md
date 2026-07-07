## Chapter 5: Trick Output Questions — JavaScript Type System

> Try to predict the output **before** expanding the answer. Each question reflects a real interview scenario.

---

## typeof & Type Checking

---

**Q1. What is the output?**

```js
console.log(typeof 42);
console.log(typeof 'hello');
console.log(typeof true);
console.log(typeof undefined);
console.log(typeof null);
console.log(typeof {});
console.log(typeof []);
console.log(typeof function(){});
```

<details>
<summary>Show Output & Explanation</summary>

```
number
string
boolean
undefined
object
object
object
function
```

**Explanation:**  
- `typeof null` → `"object"` is a **historical bug** — null is not an object.
- `typeof []` → `"object"` — arrays are objects; use `Array.isArray()` to distinguish.
- `typeof function(){}` → `"function"` — functions are objects but get their own typeof result.

</details>

---

**Q2. What is the output?**

```js
console.log(typeof typeof 42);
```

<details>
<summary>Show Output & Explanation</summary>

```
string
```

**Explanation:**  
Evaluated inside-out. `typeof 42` → `"number"` (a string). Then `typeof "number"` → `"string"`. `typeof` always returns a string — so `typeof` of any `typeof` result is always `"string"`.

</details>

---

**Q3. What is the output?**

```js
console.log(typeof undeclaredVariable);
console.log(undeclaredVariable);
```

<details>
<summary>Show Output & Explanation</summary>

```
undefined
ReferenceError: undeclaredVariable is not defined
```

**Explanation:**  
`typeof` does **not throw** for undeclared variables — it returns `"undefined"`. Accessing an undeclared variable any other way throws a `ReferenceError`. This makes `typeof` the safe way to test existence: `if (typeof myVar !== 'undefined')`.

</details>

---

**Q4. What is the output?**

```js
console.log(Array.isArray([]));
console.log(Array.isArray({}));
console.log(Array.isArray('hello'));
console.log(Array.isArray(new Array(3)));
```

<details>
<summary>Show Output & Explanation</summary>

```
true
false
false
true
```

**Explanation:**  
`Array.isArray()` is the only reliable way to detect arrays. `new Array(3)` creates `[empty × 3]` — still an array. Strings and objects return `false` regardless of their content.

</details>

---

## Equality & Coercion

---

**Q5. What is the output?**

```js
console.log(1 == '1');
console.log(1 === '1');
console.log(null == undefined);
console.log(null === undefined);
console.log(NaN == NaN);
console.log(NaN === NaN);
```

<details>
<summary>Show Output & Explanation</summary>

```
true
false
true
false
false
false
```

**Explanation:**  
- `1 == '1'` → `'1'` coerced to `1` → `true`
- `1 === '1'` → different types → `false`
- `null == undefined` → special rule in the spec → `true`
- `null === undefined` → different types → `false`
- `NaN == NaN` / `NaN === NaN` → `NaN` is never equal to anything, including itself — IEEE 754 rule

</details>

---

**Q6. What is the output?**

```js
console.log(0 == false);
console.log(0 === false);
console.log('' == false);
console.log('' === false);
console.log(null == false);
console.log(undefined == false);
```

<details>
<summary>Show Output & Explanation</summary>

```
true
false
true
false
false
false
```

**Explanation:**  
- `0 == false` → `false` coerced to `0`, then `0 == 0` → `true`
- `'' == false` → `false` → `0`, `''` → `0`, then `0 == 0` → `true`
- `null == false` → **`false`** — `null` only loosely equals `undefined`, nothing else
- `undefined == false` → **`false`** — same rule: `undefined` only == `null`

> **Trap:** Many assume `null` and `undefined` are falsy so they `==` false. They don't — the spec explicitly only allows `null == undefined`.

</details>

---

**Q7. What is the output?**

```js
console.log([] == false);
console.log([] == 0);
console.log([] == '');
console.log([] == ![]);
```

<details>
<summary>Show Output & Explanation</summary>

```
true
true
true
true
```

**Explanation:**  
- `[] == false` → `false` → `0`, `[]` → `''` → `0`, then `0 == 0` → `true`
- `[] == 0` → `[]` → `''` → `0`, then `0 == 0` → `true`
- `[] == ''` → `[]` → `''`, then `'' == ''` → `true`
- `[] == ![]` → `![]` = `false` (arrays are truthy), so `[] == false` → `true`

> **Classic interview trap.** Arrays coerce via `toString()` which gives `''` for empty arrays.

</details>

---

**Q8. What is the output?**

```js
console.log(+[]);
console.log(+{});
console.log(+null);
console.log(+undefined);
console.log(+true);
console.log(+false);
console.log(+'');
console.log(+'42px');
```

<details>
<summary>Show Output & Explanation</summary>

```
0
NaN
0
NaN
1
0
0
NaN
```

**Explanation:**  
The unary `+` operator converts to number:
- `+[]` → `+''` → `0`
- `+{}` → `+'[object Object]'` → `NaN`
- `+null` → `0`
- `+undefined` → `NaN`
- `+true` → `1`, `+false` → `0`
- `+''` → `0`
- `+'42px'` → `NaN` — non-numeric characters prevent conversion

</details>

---

**Q9. What is the output?**

```js
console.log(1 + '2');
console.log('3' - 1);
console.log('3' * '2');
console.log(true + true);
console.log(true + false);
console.log(null + 1);
console.log(undefined + 1);
```

<details>
<summary>Show Output & Explanation</summary>

```
12
2
6
2
1
1
NaN
```

**Explanation:**  
- `1 + '2'` → `+` with a string → concatenation → `'12'`
- `'3' - 1` → `-` only numeric → `'3'` coerced to `3` → `2`
- `'3' * '2'` → both coerced to numbers → `6`
- `true + true` → `1 + 1` = `2`
- `null + 1` → `0 + 1` = `1`
- `undefined + 1` → `NaN + 1` = `NaN`

</details>

---

## Truthy / Falsy

---

**Q10. What is the output?**

```js
const values = [0, '', null, undefined, NaN, false, [], {}, 'false', -1];

values.forEach(v => {
  if (v) console.log(String(v) + ' is truthy');
});
```

<details>
<summary>Show Output & Explanation</summary>

```
[] is truthy
[object Object] is truthy
false is truthy
-1 is truthy
```

**Explanation:**  
The first 6 values are all falsy: `0`, `''`, `null`, `undefined`, `NaN`, `false`.  
- `[]` → truthy (empty array is an object)
- `{}` → truthy (empty object)
- `'false'` → truthy (non-empty string)
- `-1` → truthy (non-zero number)

</details>

---

**Q11. What is the output?**

```js
console.log(Boolean([]));
console.log(Boolean({}));
console.log(Boolean('0'));
console.log(Boolean(0));
console.log([] == false);
console.log(Boolean([]) == false);
```

<details>
<summary>Show Output & Explanation</summary>

```
true
true
true
false
true
false
```

**Explanation:**  
- `Boolean([])` → `true` — arrays are truthy objects
- `Boolean({})` → `true` — objects are truthy
- `Boolean('0')` → `true` — non-empty string
- `Boolean(0)` → `false` — zero is falsy
- `[] == false` → `true` — coercion: `[]` → `''` → `0`, `false` → `0` → equal
- `Boolean([]) == false` → `true == false` → `false` — now comparing primitives

> **Subtle trap:** `[]` is truthy in a boolean context BUT `[] == false` is `true` due to coercion. These are different operations.

</details>

---

## null, undefined & NaN

---

**Q12. What is the output?**

```js
console.log(null + 1);
console.log(null + '1');
console.log(null == 0);
console.log(null > 0);
console.log(null >= 0);
```

<details>
<summary>Show Output & Explanation</summary>

```
1
null1
false
false
true
```

**Explanation:**  
- `null + 1` → `null` → `0` → `1`
- `null + '1'` → string context: `null` → `'null'`, concatenated → `'null1'`
- `null == 0` → `false` — `null` only `==` `undefined`, nothing else
- `null > 0` → `false` — `null` → `0`, then `0 > 0` = `false`
- `null >= 0` → **`true`** — `null` → `0`, then `0 >= 0` = `true`

> **Classic trap.** `null == 0` is `false` but `null >= 0` is `true` — equality and comparison use different algorithms in the spec.

</details>

---

**Q13. What is the output?**

```js
console.log(isNaN('hello'));
console.log(isNaN(undefined));
console.log(isNaN(null));
console.log(isNaN(''));

console.log(Number.isNaN('hello'));
console.log(Number.isNaN(undefined));
console.log(Number.isNaN(NaN));
```

<details>
<summary>Show Output & Explanation</summary>

```
true
true
false
false

false
false
true
```

**Explanation:**  
Global `isNaN()` **coerces** the argument to a number first:
- `'hello'` → `NaN` → `true`
- `undefined` → `NaN` → `true`
- `null` → `0` → `false`
- `''` → `0` → `false`

`Number.isNaN()` does **no coercion** — only returns `true` for the actual `NaN` value:
- `'hello'` → string, not NaN → `false`
- `undefined` → not NaN → `false`
- `NaN` → `true` ✅

</details>

---

**Q14. What is the output?**

```js
console.log(Object.is(NaN, NaN));
console.log(Object.is(+0, -0));
console.log(Object.is(1, 1));
console.log(Object.is(null, null));
console.log(Object.is(undefined, null));
```

<details>
<summary>Show Output & Explanation</summary>

```
true
false
true
true
false
```

**Explanation:**  
`Object.is()` is like `===` with two exceptions:
- `NaN` is equal to itself → `true` (unlike `===`)
- `+0` and `-0` are **not equal** → `false` (unlike `===`)
All other comparisons follow `===` rules.

</details>

---

**Q15. What is the output?**

```js
let a;
const b = null;

console.log(a == b);
console.log(a === b);
console.log(typeof a);
console.log(typeof b);
```

<details>
<summary>Show Output & Explanation</summary>

```
true
false
undefined
object
```

**Explanation:**  
- `a == b` → `undefined == null` → `true` (spec special rule)
- `a === b` → different types (`undefined` vs `null`) → `false`
- `typeof undefined` → `"undefined"`
- `typeof null` → `"object"` (the historical bug)

</details>

---

## Optional Chaining & Nullish Coalescing

---

**Q16. What is the output?**

```js
const user = { profile: { name: 'Alice' } };

console.log(user?.profile?.name);
console.log(user?.address?.city);
console.log(user?.greet?.());
```

<details>
<summary>Show Output & Explanation</summary>

```
Alice
undefined
undefined
```

**Explanation:**  
- `user?.profile?.name` → both exist → `'Alice'`
- `user?.address?.city` → `user.address` is `undefined`, short-circuits → `undefined` (no throw)
- `user?.greet?.()` → `greet` doesn't exist → `undefined` (no `TypeError`)

</details>

---

**Q17. What is the output?**

```js
const config = {
  port: 0,
  name: '',
  debug: false,
  timeout: null
};

console.log(config.port || 3000);
console.log(config.port ?? 3000);

console.log(config.name || 'unnamed');
console.log(config.name ?? 'unnamed');

console.log(config.debug || true);
console.log(config.debug ?? true);

console.log(config.timeout || 5000);
console.log(config.timeout ?? 5000);
```

<details>
<summary>Show Output & Explanation</summary>

```
3000
0

unnamed

true
false

5000
5000
```

**Explanation:**  
`||` returns the right side for ANY falsy value. `??` returns the right side ONLY for `null`/`undefined`.

- `port: 0` → `||` treats `0` as falsy → `3000`; `??` treats `0` as valid → `0`
- `name: ''` → `||` treats `''` as falsy → `'unnamed'`; `??` treats `''` as valid → `''`
- `debug: false` → `||` treats `false` as falsy → `true`; `??` treats `false` as valid → `false`
- `timeout: null` → both `||` and `??` treat `null` as missing → `5000`

> **This is the most important practical difference between `||` and `??`.**

</details>

---

**Q18. What is the output?**

```js
const arr = null;
const obj = undefined;

console.log(arr?.[0]);
console.log(obj?.name);
console.log(arr?.length ?? 'no array');
```

<details>
<summary>Show Output & Explanation</summary>

```
undefined
undefined
no array
```

**Explanation:**  
- `arr?.[0]` → `arr` is `null`, short-circuits → `undefined`
- `obj?.name` → `obj` is `undefined`, short-circuits → `undefined`
- `arr?.length` → `null`, short-circuits → `undefined`; then `undefined ?? 'no array'` → `'no array'`

</details>

---

## Tricky Coercion

---

**Q19. What is the output?**

```js
console.log([] + []);
console.log([] + {});
console.log({} + []);
console.log(+[]);
console.log(+{});
```

<details>
<summary>Show Output & Explanation</summary>

```
""
[object Object]
[object Object]
0
NaN
```

**Explanation:**  
- `[] + []` → `'' + ''` → `''`
- `[] + {}` → `'' + '[object Object]'` → `'[object Object]'`
- `console.log({} + [])` → inside a function call, `{}` is in **expression context** and is parsed as an **object literal**, not an empty block. So `{} + []` → `'[object Object]' + ''` → `'[object Object]'`
- `+[]` → `+''` → `0`
- `+{}` → `+'[object Object]'` → `NaN`

> **Classic JS trick question.** `{}` at the **start of a statement** is parsed as a block (so `{} + []` as a standalone statement = `0`), but inside `console.log(...)` it is in expression context and parsed as an object literal → `'[object Object]'`.

</details>

---

**Q20. What is the output?**

```js
console.log(+'3');
console.log(+true);
console.log(+false);
console.log(+null);
console.log(+undefined);
console.log(+'');
console.log(+' ');
console.log(+'0x10');
```

<details>
<summary>Show Output & Explanation</summary>

```
3
1
0
0
NaN
0
0
16
```

**Explanation:**  
Unary `+` converts to number:
- `+'3'` → `3`
- `+true/false` → `1/0`
- `+null` → `0`
- `+undefined` → `NaN`
- `+''` and `+' '` → `0` (whitespace-only strings → `0`)
- `+'0x10'` → hex literal → `16`

</details>

---

**Q21. What is the output?**

```js
console.log([1, 2, 3].sort());
console.log([10, 9, 2, 1, 100].sort());
console.log([10, 9, 2, 1, 100].sort((a, b) => a - b));
```

<details>
<summary>Show Output & Explanation</summary>

```
[1, 2, 3]
[1, 10, 100, 2, 9]
[1, 2, 9, 10, 100]
```

**Explanation:**  
Array `.sort()` with no comparator converts elements to **strings** and sorts lexicographically. `'100' < '2'` because `'1' < '2'` at the first character. This is a coercion bug that catches many developers off guard.

Always provide a comparator for numeric sorts: `(a, b) => a - b`.

</details>

---

**Q22. What is the output?**

```js
function getDefault(value) {
  return value || 'default';
}

function getSafe(value) {
  return value ?? 'default';
}

console.log(getDefault(0));
console.log(getSafe(0));
console.log(getDefault(false));
console.log(getSafe(false));
console.log(getDefault(null));
console.log(getSafe(null));
```

<details>
<summary>Show Output & Explanation</summary>

```
default
0
default
false
default
default
```

**Explanation:**  
`||` activates on any falsy value — `0` and `false` are falsy, so `getDefault` returns `'default'` for them.  
`??` only activates on `null`/`undefined` — `getSafe(0)` keeps `0`, `getSafe(false)` keeps `false`.  
Both return `'default'` for `null` since both operators treat `null` as missing.

</details>

---

## Mixed / Hard

---

**Q23. What is the output?**

```js
console.log(0.1 + 0.2 === 0.3);
console.log(0.1 + 0.2);
console.log(Math.abs(0.1 + 0.2 - 0.3) < Number.EPSILON);
```

<details>
<summary>Show Output & Explanation</summary>

```
false
0.30000000000000004
true
```

**Explanation:**  
JavaScript uses IEEE 754 double precision. `0.1` and `0.2` cannot be represented exactly in binary, so their sum is `0.30000000000000004` — close but not equal to `0.3`. `===` fails.

`Number.EPSILON` (~2.22e-16) is the smallest difference between two representable doubles. Checking that the difference is less than `Number.EPSILON` is the correct way to compare floats.

> **Real-world impact:** Never compare floating-point currency values with `===`. Store amounts as integers (cents) or use a decimal library.

</details>

---

**Q24. What is the output?**

```js
const a = '5';
const b = 3;

console.log(a + b);
console.log(a - b);
console.log(a * b);
console.log(a / b);
console.log(a > b);
console.log(a == b);
console.log(a === b);
```

<details>
<summary>Show Output & Explanation</summary>

```
53
2
15
1.6666666666666667
true
false
false
```

**Explanation:**  
- `a + b` → `+` with a string → concatenation → `'53'`
- `a - b` → `-` numeric only → `'5'` coerced to `5` → `2`
- `a * b` → `'5'` → `5` → `15`
- `a / b` → `'5'` → `5` → `1.666...`
- `a > b` → `'5'` coerced to `5` for comparison → `5 > 3` → `true`
- `a == b` → `'5'` coerced to `5`, then `5 == 3` → `false`
- `a === b` → different types → `false`

> **Summary:** `+` uniquely favors string concatenation. All other operators (`-`, `*`, `/`, `>`, `<`) coerce to numbers. `==` coerces strings to numbers for number comparisons.

</details>

---

---

## Floating Point & Number Precision

---

**Q25. What is the output?**

```js
console.log(0.1 + 0.2 === 0.3);
console.log(0.1 + 0.2);
console.log(Math.abs(0.1 + 0.2 - 0.3) < Number.EPSILON);
console.log(Number.EPSILON);
```

<details>
<summary>Show Output & Explanation</summary>

```
false
0.30000000000000004
true
2.220446049250313e-16
```

**Explanation:**  
`0.1` and `0.2` cannot be represented exactly in binary (IEEE 754), so the sum is `0.30000000000000004`. Direct `===` fails. `Number.EPSILON` is the smallest difference between two representable doubles (~2.22e-16) — its exact value is `2.220446049250313e-16`.

> **Common Mistake:** Using `===` to compare float results. Use `Number.EPSILON` for equality checks, or `toFixed()` for display.

</details>

---

**Q26. What is the output?**

```js
console.log(parseInt('08'));
console.log(parseInt('08', 10));
console.log(parseInt('0x10'));
console.log(parseInt('10', 2));
console.log(parseInt('3.9'));
console.log(['1', '2', '3'].map(parseInt));
```

<details>
<summary>Show Output & Explanation</summary>

```
8
8
16
2
3
[1, NaN, NaN]
```

**Explanation:**  
- `parseInt('08')` → defaults to radix 10 in modern engines → `8`
- `parseInt('08', 10)` → explicit decimal → `8`
- `parseInt('0x10')` → auto-detected as hex → `16`
- `parseInt('10', 2)` → binary `10` = `2`
- `parseInt('3.9')` → truncates, doesn't round → `3`
- `['1','2','3'].map(parseInt)` → `.map` passes `(value, index, array)`, so the calls are `parseInt('1', 0)`, `parseInt('2', 1)`, `parseInt('3', 2)`. Radix `0` is treated as 10 → `1`. Radix `1` is invalid → `NaN`. `'3'` is not a valid binary digit → `NaN`.

> **Common Mistake:** Passing `parseInt` directly to `.map()` — the array index becomes the radix. Wrap it: `.map(n => parseInt(n, 10))`.

</details>

---

*Chapter 5 — Trick Output Questions | JavaScript Type System*
