# Chapter 2: Trick Output Questions — Closures

> Try to predict the output **before** expanding the answer. Each question reflects a real interview scenario.

---

## Closure Basics

---

**Q1. What is the output?**

```js
function outer() {
  let x = 10;

  function inner() {
    console.log(x);
  }

  x = 20;
  inner();
}

outer();
```

<details>
<summary>Show Output & Explanation</summary>

```
20
```

**Explanation:**  
Closures capture variables **by reference**, not by value. `inner` holds a reference to the variable `x` in `outer`'s Lexical Environment. When `x` is reassigned to `20` before `inner()` is called, the closure sees the updated value.

> **Key rule:** Closures don't snapshot the value at creation time — they reference the live variable.

</details>

---

**Q2. What is the output?**

```js
function makeCounter() {
  let count = 0;

  return {
    increment() { count++; },
    decrement() { count--; },
    value()     { return count; }
  };
}

const c = makeCounter();
c.increment();
c.increment();
c.decrement();
console.log(c.value());
```

<details>
<summary>Show Output & Explanation</summary>

```
1
```

**Explanation:**  
All three methods (`increment`, `decrement`, `value`) close over the **same `count` variable** in `makeCounter`'s scope. Each call modifies or reads the shared variable.

`0 + 1 + 1 - 1 = 1`

</details>

---

**Q3. What is the output?**

```js
function outer() {
  let x = 10;

  return function inner() {
    console.log(x);
  };
}

let fn = outer();
fn(); // call 1
fn(); // call 2
```

<details>
<summary>Show Output & Explanation</summary>

```
10
10
```

**Explanation:**  
`outer()` has already returned, but `fn` still holds a reference to `inner`, which closes over `x = 10`. Each call reads the same `x`. Since nothing modifies `x`, both calls print `10`.

</details>

---

**Q4. What is the output?**

```js
function createFns() {
  const fns = [];

  for (var i = 0; i < 3; i++) {
    fns.push(function () {
      console.log(i);
    });
  }

  return fns;
}

const fns = createFns();
fns[0]();
fns[1]();
fns[2]();
```

<details>
<summary>Show Output & Explanation</summary>

```
3
3
3
```

**Explanation:**  
All three functions close over the **same `i`** — because `var` is function-scoped, there's only one `i` shared across all iterations. By the time any function is called, the loop has finished and `i = 3`.

**Fix with `let`:**
```js
for (let i = 0; i < 3; i++) { ... } // 0, 1, 2
```

**Fix with IIFE:**
```js
fns.push((function (j) {
  return function () { console.log(j); };
})(i));
```

> **Classic closure trap.** This is one of the most frequently asked JS interview questions.

</details>

---

**Q5. What is the output?**

```js
function createFns() {
  const fns = [];

  for (let i = 0; i < 3; i++) {
    fns.push(function () {
      console.log(i);
    });
  }

  return fns;
}

const fns = createFns();
fns[0]();
fns[1]();
fns[2]();
```

<details>
<summary>Show Output & Explanation</summary>

```
0
1
2
```

**Explanation:**  
`let` creates a **new binding per iteration**. Each function closes over its own `i` from that iteration's block scope. They no longer share a single `i`.

</details>

---

**Q6. What is the output?**

```js
function outer() {
  let count = 0;

  function increment() {
    count++;
    console.log(count);
  }

  count = 10;
  return increment;
}

const inc = outer();
inc();
inc();
```

<details>
<summary>Show Output & Explanation</summary>

```
11
12
```

**Explanation:**  
The closure captures `count` by reference. Before `outer` returns, `count` is set to `10`. So the first `inc()` increments from `10` to `11`, and the second from `11` to `12`.

</details>

---

## Shared vs Independent Closures

---

**Q7. What is the output?**

```js
function makeAdder(x) {
  return function (y) {
    return x + y;
  };
}

const add5  = makeAdder(5);
const add10 = makeAdder(10);

console.log(add5(3));
console.log(add10(3));
console.log(add5(10));
```

<details>
<summary>Show Output & Explanation</summary>

```
8
13
15
```

**Explanation:**  
Each call to `makeAdder` creates a **separate Lexical Environment** with its own `x`. `add5` and `add10` are completely independent closures — they don't share state.

`add5(3)  → 5 + 3  = 8`  
`add10(3) → 10 + 3 = 13`  
`add5(10) → 5 + 10 = 15`

</details>

---

**Q8. What is the output?**

```js
function makeCounter() {
  let count = 0;
  return () => ++count;
}

const a = makeCounter();
const b = makeCounter();

console.log(a()); // ?
console.log(a()); // ?
console.log(b()); // ?
console.log(a()); // ?
```

<details>
<summary>Show Output & Explanation</summary>

```
1
2
1
3
```

**Explanation:**  
`a` and `b` are created by **separate calls** to `makeCounter`, so they have **separate Lexical Environments** with independent `count` variables. `b()` starts from `0` regardless of how many times `a()` was called.

</details>

---

**Q9. What is the output?**

```js
const counter = (function () {
  let n = 0;
  return {
    inc: () => ++n,
    dec: () => --n,
    val: () => n
  };
})();

counter.inc();
counter.inc();
counter.inc();
counter.dec();
console.log(counter.val());
```

<details>
<summary>Show Output & Explanation</summary>

```
2
```

**Explanation:**  
The IIFE runs once, creating a single closure with one `n`. All three methods share that same `n`. `3 increments - 1 decrement = 2`.

> This is the **Module Pattern** — a common interview follow-up from a basic closure question.

</details>

---

## Closures & Asynchronous Code

---

**Q10. What is the output?**

```js
for (var i = 1; i <= 3; i++) {
  setTimeout(function () {
    console.log(i);
  }, i * 1000);
}
```

<details>
<summary>Show Output & Explanation</summary>

```
4   (after 1 second)
4   (after 2 seconds)
4   (after 3 seconds)
```

**Explanation:**  
All three `setTimeout` callbacks close over the **same `var i`**. By the time any callback fires, the loop has already completed and `i = 4` (the value that caused the loop condition to fail: `4 <= 3` is `false`).

</details>

---

**Q11. What is the output?**

```js
function createTimer() {
  let seconds = 0;

  setInterval(function () {
    seconds++;
  }, 1000);

  return function () {
    return seconds;
  };
}

const getTime = createTimer();

setTimeout(() => {
  console.log(getTime());
}, 3500);
```

<details>
<summary>Show Output & Explanation</summary>

```
3
```

**Explanation:**  
The `setInterval` callback and `getTime` both close over the same `seconds` variable. After 3500ms, the interval has fired 3 times (at 1s, 2s, 3s), incrementing `seconds` to `3`. `getTime()` reads the live variable.

</details>

---

**Q12. What is the output?**

```js
function foo() {
  let result = [];

  for (let i = 0; i < 3; i++) {
    result.push(() => i);
  }

  return result;
}

const fns = foo();
console.log(fns[0]());
console.log(fns[1]());
console.log(fns[2]());
```

<details>
<summary>Show Output & Explanation</summary>

```
0
1
2
```

**Explanation:**  
`let` in a `for` loop creates a **new `i` binding per iteration**. Each arrow function closes over its own independent `i`. Changing the loop to `var` would make all three return `3`.

</details>

---

## Stale Closures

---

**Q13. What is the output?**

```js
function makeLogger(val) {
  return function () {
    console.log(val);
  };
}

let x = 1;
const log = makeLogger(x);

x = 99;
log();
```

<details>
<summary>Show Output & Explanation</summary>

```
1
```

**Explanation:**  
`x` (the primitive `1`) is **passed by value** into `makeLogger`. The closure captures `val = 1`, not the variable `x` itself. Reassigning `x = 99` later has no effect on what `log` prints.

> **Contrast with Q1:** Closures capture *variables by reference*. But when you *pass a value* as an argument, the parameter is a new binding — changes to the original variable don't affect it.

</details>

---

**Q14. What is the output?**

```js
function makeLogger(obj) {
  return function () {
    console.log(obj.val);
  };
}

let data = { val: 1 };
const log = makeLogger(data);

data.val = 99;
log();
```

<details>
<summary>Show Output & Explanation</summary>

```
99
```

**Explanation:**  
Objects are passed **by reference**. `obj` and `data` point to the same object in memory. Mutating `data.val` mutates the same object that `obj` refers to. When `log()` runs, `obj.val` is `99`.

> **Key distinction:** Primitives passed as arguments create independent bindings. Objects passed as arguments share the reference — mutations are visible through the closure.

</details>

---

**Q15. What is the output?**

```js
function outer() {
  let x = 1;

  function inner() {
    console.log(x);
  }

  x = 2;
  x = 3;

  return inner;
}

outer()();
```

<details>
<summary>Show Output & Explanation</summary>

```
3
```

**Explanation:**  
The closure captures the **variable `x`**, not a snapshot of it. By the time `inner` executes, `x` has been reassigned twice and is `3`. This is a direct consequence of closures capturing by reference.

</details>

---

## Practical Patterns

---

**Q16. What is the output?**

```js
function memoize(fn) {
  const cache = {};
  return function (n) {
    if (n in cache) return cache[n];
    cache[n] = fn(n);
    return cache[n];
  };
}

const square = memoize((n) => {
  console.log('computing', n);
  return n * n;
});

console.log(square(4));
console.log(square(4));
console.log(square(5));
```

<details>
<summary>Show Output & Explanation</summary>

```
computing 4
16
16
computing 5
25
```

**Explanation:**  
The first `square(4)` misses the cache → logs "computing 4" → returns `16`.  
The second `square(4)` hits the cache → no log → returns `16` directly.  
`square(5)` is a new key → logs "computing 5" → returns `25`.

The `cache` object persists between calls because it's captured in the memoized function's closure.

</details>

---

**Q17. What is the output?**

```js
function debounce(fn, delay) {
  let timer;

  return function (...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  };
}

const log = debounce((x) => console.log(x), 100);

log('a');
log('b');
log('c');
```

<details>
<summary>Show Output & Explanation</summary>

```
c
```

**Explanation:**  
Each `log()` call clears the previous timer and sets a new one. Only the last call (`'c'`) isn't cancelled — its `setTimeout` fires after 100ms. The first two are cleared before they execute.

`timer` is kept alive between calls via the closure — that's what makes debounce work.

</details>

---

**Q18. What is the output?**

```js
function outer() {
  let x = 10;

  const inner1 = () => {
    x += 5;
    console.log('inner1:', x);
  };

  const inner2 = () => {
    x *= 2;
    console.log('inner2:', x);
  };

  return [inner1, inner2];
}

const [fn1, fn2] = outer();
fn1();
fn2();
fn1();
```

<details>
<summary>Show Output & Explanation</summary>

```
inner1: 15
inner2: 30
inner1: 35
```

**Explanation:**  
Both `inner1` and `inner2` close over the **same `x`** from `outer`. Mutations by one are visible to the other.

- `fn1()`: `x = 10 + 5 = 15`
- `fn2()`: `x = 15 * 2 = 30`
- `fn1()`: `x = 30 + 5 = 35`

</details>

---

## Hard / Mixed

---

**Q19. What is the output?**

```js
let fns = [];

(function () {
  for (var i = 0; i < 3; i++) {
    fns.push(
      (function (j) {
        return function () { console.log(j); };
      })(i)
    );
  }
})();

fns[0]();
fns[1]();
fns[2]();
```

<details>
<summary>Show Output & Explanation</summary>

```
0
1
2
```

**Explanation:**  
The IIFE wrapping each function creates a **new scope with its own `j`** per iteration. `i` is passed as an argument, creating an independent copy. This is the classic pre-ES6 fix for the `var` in loops problem — before `let` existed.

</details>

---

**Q20. What is the output?**

```js
function secret() {
  let _val = 42;

  return {
    get() { return _val; },
    set(v) { _val = v; }
  };
}

const s = secret();
console.log(s._val);
console.log(s.get());
s.set(100);
console.log(s.get());
```

<details>
<summary>Show Output & Explanation</summary>

```
undefined
42
100
```

**Explanation:**  
- `s._val` → `undefined`. `_val` is a local variable inside `secret`, not a property of the returned object. There is no `_val` property on `s`.
- `s.get()` → `42`. The `get` method closes over `_val`.
- After `s.set(100)`, `_val` is updated to `100`.
- `s.get()` → `100`.

> This demonstrates **true data privacy** via closure — `_val` is inaccessible from outside, unlike a naming convention like `_val` on an object.

</details>

---

**Q21. What is the output?**

```js
function multiplier(x) {
  return function (y) {
    return function (z) {
      return x * y * z;
    };
  };
}

console.log(multiplier(2)(3)(4));
```

<details>
<summary>Show Output & Explanation</summary>

```
24
```

**Explanation:**  
This is **currying** via closures. Each call returns a new function that closes over the previous argument:

- `multiplier(2)` → returns `fn(y)`, which closes over `x = 2`
- `fn(3)` → returns `fn(z)`, which closes over `x = 2, y = 3`
- `fn(4)` → executes `2 * 3 * 4 = 24`

</details>

---

**Q22. What is the output?**

```js
function counter() {
  let count = 0;

  return {
    increment: function () { count++; },
    reset:     function () { count = 0; },
    value:     function () { return count; }
  };
}

const c1 = counter();
const c2 = counter();

c1.increment();
c1.increment();
c2.increment();
c1.reset();

console.log(c1.value());
console.log(c2.value());
```

<details>
<summary>Show Output & Explanation</summary>

```
0
1
```

**Explanation:**  
`c1` and `c2` are created by **separate calls** to `counter()`, so they each have their own `count`. Operations on `c1` don't affect `c2`.

- `c1`: increment → increment → reset → `count = 0`
- `c2`: increment → `count = 1`

</details>

---

**Q23. What is the output?**

```js
function outer() {
  var x = 1;

  function middle() {
    var y = 2;

    function inner() {
      console.log(x + y);
    }

    y = 10;
    return inner;
  }

  x = 5;
  return middle;
}

outer()()();
```

<details>
<summary>Show Output & Explanation</summary>

```
15
```

**Explanation:**  
Walk through the calls:
- `outer()` → returns `middle` (but first sets `x = 5`)
- `outer()()` → calls `middle()` → sets `y = 2`, then reassigns `y = 10`, then returns `inner`
- `outer()()()` → calls `inner()` → reads `x = 5` and `y = 10` → `5 + 10 = 15`

Both `x` and `y` are captured by reference — `inner` always reads their latest values.

</details>

---

**Q24. What is the output?**

```js
const obj = (function () {
  let count = 0;

  return {
    increment() { count++; return this; },
    decrement() { count--; return this; },
    value()     { return count; }
  };
})();

console.log(
  obj.increment().increment().decrement().value()
);
```

<details>
<summary>Show Output & Explanation</summary>

```
1
```

**Explanation:**  
Each method returns `this` (the `obj` object), enabling **method chaining**. The operations apply to the shared `count`:

`0 + 1 + 1 - 1 = 1`

`value()` returns the number `1`, which is what `console.log` prints.

> This combines the **Module Pattern** (IIFE + closure for private state) with the **fluent interface / method chaining** pattern — both common interview topics.

</details>

---

*Chapter 2 — Trick Output Questions | Closures*
