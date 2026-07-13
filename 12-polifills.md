# Polyfills — Self Evaluation

> **How to use:** Read the question and requirements. Try to write the implementation yourself first. Only then click **"Show Implementation"** to reveal the code.

---

## Array Method Polyfills

---

### 1. `Array.prototype.map()`

Implement a custom `myMap` on `Array.prototype`.

**Requirements:**
- Accepts a callback `fn(element, index, array)`
- Returns a **new array** with each element transformed by `fn`
- Does **not** mutate the original array
- Skips holes in sparse arrays (like native `map`)
- Throws `TypeError` if `fn` is not a function

```js
// Expected usage:
[1, 2, 3].myMap(x => x * 2);       // [2, 4, 6]
[1, 2, 3].myMap((x, i) => i + x);  // [1, 3, 5]
```

<details>
<summary>Show Implementation</summary>

```js
Array.prototype.myMap = function (fn, thisArg) {
  if (typeof fn !== 'function') {
    throw new TypeError(fn + ' is not a function');
  }

  const result = [];
  for (let i = 0; i < this.length; i++) {
    if (Object.prototype.hasOwnProperty.call(this, i)) { // skip holes
      result[i] = fn.call(thisArg, this[i], i, this);
    }
  }
  return result;
};
```

**Key points:**
- `this` refers to the array inside the method
- `fn.call(thisArg, ...)` supports the optional `thisArg` second argument (like native `map`)
- `hasOwnProperty` check skips sparse holes

</details>

---

### 2. `Array.prototype.filter()`

Implement a custom `myFilter` on `Array.prototype`.

**Requirements:**
- Accepts a callback `fn(element, index, array)` returning a boolean
- Returns a **new array** containing only elements for which `fn` returns truthy
- Does **not** mutate the original array
- Preserves order
- Skips holes in sparse arrays

```js
// Expected usage:
[1, 2, 3, 4].myFilter(x => x % 2 === 0); // [2, 4]
[1, 2, 3].myFilter(x => x > 5);          // []
```

<details>
<summary>Show Implementation</summary>

```js
Array.prototype.myFilter = function (fn, thisArg) {
  if (typeof fn !== 'function') {
    throw new TypeError(fn + ' is not a function');
  }

  const result = [];
  for (let i = 0; i < this.length; i++) {
    if (Object.prototype.hasOwnProperty.call(this, i)) {
      if (fn.call(thisArg, this[i], i, this)) {
        result.push(this[i]);
      }
    }
  }
  return result;
};
```

**Key points:**
- Push only when callback returns truthy
- Result is always a dense (non-sparse) array
- Does not use `push` with index — that would create holes

</details>

---

### 3. `Array.prototype.reduce()`

Implement a custom `myReduce` on `Array.prototype`.

**Requirements:**
- Accepts `fn(accumulator, currentValue, index, array)` and optional `initialValue`
- If no `initialValue`: uses first element as accumulator, starts from index 1
- If array is empty and no `initialValue`: throws `TypeError`
- Does **not** mutate the original array

```js
// Expected usage:
[1, 2, 3, 4].myReduce((acc, val) => acc + val, 0);  // 10
[1, 2, 3, 4].myReduce((acc, val) => acc + val);      // 10
[].myReduce((acc, val) => acc + val);                // TypeError
```

<details>
<summary>Show Implementation</summary>

```js
Array.prototype.myReduce = function (fn, initialValue) {
  if (typeof fn !== 'function') {
    throw new TypeError(fn + ' is not a function');
  }

  const hasInitial = arguments.length >= 2;
  let acc = hasInitial ? initialValue : this[0];
  let startIndex = hasInitial ? 0 : 1;

  if (this.length === 0 && !hasInitial) {
    throw new TypeError('Reduce of empty array with no initial value');
  }

  for (let i = startIndex; i < this.length; i++) {
    if (Object.prototype.hasOwnProperty.call(this, i)) {
      acc = fn(acc, this[i], i, this);
    }
  }
  return acc;
};
```

**Key points:**
- `arguments.length >= 2` distinguishes `reduce(fn)` from `reduce(fn, undefined)` — both are valid but the second has an explicit initial value
- When no initial value: first element becomes accumulator, loop starts at index 1
- Empty array without initial value throws `TypeError`

</details>

---

### 4. `Array.prototype.forEach()`

Implement a custom `myForEach` on `Array.prototype`.

**Requirements:**
- Accepts a callback `fn(element, index, array)`
- Executes `fn` for each element — purely for side effects
- Always returns `undefined`
- Skips holes in sparse arrays
- Does **not** mutate the original array (unless the callback does so explicitly)

```js
// Expected usage:
[1, 2, 3].myForEach(x => console.log(x)); // 1, 2, 3 (returns undefined)
```

<details>
<summary>Show Implementation</summary>

```js
Array.prototype.myForEach = function (fn, thisArg) {
  if (typeof fn !== 'function') {
    throw new TypeError(fn + ' is not a function');
  }

  for (let i = 0; i < this.length; i++) {
    if (Object.prototype.hasOwnProperty.call(this, i)) {
      fn.call(thisArg, this[i], i, this);
    }
  }
  // implicitly returns undefined
};
```

**Key points:**
- No return value — the function returns `undefined`
- The callback's return value is completely ignored
- Unlike `map`, `forEach` cannot be used to produce a new array

</details>

---

### 5. `Array.prototype.find()`

Implement a custom `myFind` on `Array.prototype`.

**Requirements:**
- Accepts a callback `fn(element, index, array)` returning a boolean
- Returns the **first element** for which `fn` returns truthy
- Returns `undefined` if no element matches
- Stops iterating as soon as a match is found (short-circuits)
- Does **not** mutate the original array

```js
// Expected usage:
[1, 2, 3, 4].myFind(x => x > 2);           // 3
[1, 2, 3].myFind(x => x > 10);             // undefined
[{ id: 1 }, { id: 2 }].myFind(o => o.id === 2); // { id: 2 }
```

<details>
<summary>Show Implementation</summary>

```js
Array.prototype.myFind = function (fn, thisArg) {
  if (typeof fn !== 'function') {
    throw new TypeError(fn + ' is not a function');
  }

  for (let i = 0; i < this.length; i++) {
    if (fn.call(thisArg, this[i], i, this)) {
      return this[i]; // short-circuit on first match
    }
  }
  return undefined;
};
```

**Key points:**
- Returns the **element itself**, not the index (use `findIndex` for the index)
- Short-circuits — stops at the first truthy result
- Returns `undefined` (not `-1`) when not found

</details>

---

### 6. `Array.prototype.flat()`

Implement a custom `myFlat` on `Array.prototype`.

**Requirements:**
- Accepts an optional `depth` argument (default: `1`)
- Returns a **new array** with nested arrays flattened up to `depth` levels
- `depth = Infinity` flattens completely
- Does **not** mutate the original array

```js
// Expected usage:
[1, [2, [3, [4]]]].myFlat();      // [1, 2, [3, [4]]]
[1, [2, [3, [4]]]].myFlat(2);     // [1, 2, 3, [4]]
[1, [2, [3, [4]]]].myFlat(Infinity); // [1, 2, 3, 4]
```

<details>
<summary>Show Implementation</summary>

```js
Array.prototype.myFlat = function (depth = 1) {
  const result = [];

  function flatten(arr, currentDepth) {
    for (const item of arr) {
      if (Array.isArray(item) && currentDepth > 0) {
        flatten(item, currentDepth - 1);
      } else {
        result.push(item);
      }
    }
  }

  flatten(this, depth);
  return result;
};
```

**Key points:**
- Recursive approach — decrements depth on each level
- When `depth === 0`, treat nested arrays as plain values
- `Infinity` works naturally since `Infinity - 1 === Infinity`

</details>

---

## Function Method Polyfills

---

### 7. `Function.prototype.call()`

Implement a custom `myCall` on `Function.prototype`.

**Requirements:**
- Sets `this` inside the function to the provided `context`
- Accepts additional arguments spread individually
- Returns the function's return value
- If `context` is `null` or `undefined`, uses the global object

```js
// Expected usage:
function greet(greeting) {
  return `${greeting}, ${this.name}!`;
}
greet.myCall({ name: 'Alice' }, 'Hello'); // 'Hello, Alice!'
```

<details>
<summary>Show Implementation</summary>

```js
Function.prototype.myCall = function (context, ...args) {
  // Use globalThis if context is null/undefined
  context = context ?? globalThis;

  // Create a unique key to avoid overwriting existing properties
  const fnKey = Symbol('fn');
  context[fnKey] = this; // `this` here is the function being called

  const result = context[fnKey](...args);
  delete context[fnKey];

  return result;
};
```

**Key points:**
- Temporarily attach the function as a property on the context object — calling it as a method sets `this` correctly
- Use a `Symbol` key to avoid collisions with existing properties
- Delete the temporary property after the call
- `...args` collects all additional arguments

</details>

---

### 8. `Function.prototype.apply()`

Implement a custom `myApply` on `Function.prototype`.

**Requirements:**
- Same as `call()` but accepts arguments as an **array** (or array-like), not spread
- If `argsArray` is `null` or `undefined`, call with no arguments
- Returns the function's return value

```js
// Expected usage:
function sum(a, b, c) { return a + b + c; }
sum.myApply(null, [1, 2, 3]); // 6

Math.max.myApply(null, [3, 1, 4, 1, 5]); // 5
```

<details>
<summary>Show Implementation</summary>

```js
Function.prototype.myApply = function (context, argsArray) {
  context = context ?? globalThis;

  const fnKey = Symbol('fn');
  context[fnKey] = this;

  const result = argsArray
    ? context[fnKey](...argsArray)
    : context[fnKey]();

  delete context[fnKey];
  return result;
};
```

**Key points:**
- Same approach as `myCall` — attach function as a method on context
- Spread `argsArray` on invocation (`...argsArray`)
- Handle the case where `argsArray` is `null` or `undefined` (call with no args)
- `apply` is just `call` with array-spread arguments

</details>

---

### 9. `Function.prototype.bind()`

Implement a custom `myBind` on `Function.prototype`.

**Requirements:**
- Returns a **new function** with `this` permanently bound to `context`
- Supports **partial application** — pre-fill arguments passed to `myBind`
- The bound function can receive additional arguments when called
- The bound function should work correctly when used as a constructor (`new`)

```js
// Expected usage:
function greet(greeting, punct) {
  return `${greeting}, ${this.name}${punct}`;
}
const greetAlice = greet.myBind({ name: 'Alice' }, 'Hello');
greetAlice('!'); // 'Hello, Alice!'
greetAlice('?'); // 'Hello, Alice?'
```

<details>
<summary>Show Implementation</summary>

```js
Function.prototype.myBind = function (context, ...presetArgs) {
  const originalFn = this;

  return function (...callArgs) {
    // If called with `new`, ignore the bound context
    if (this instanceof boundFn) {
      return new originalFn(...presetArgs, ...callArgs);
    }
    return originalFn.apply(context, [...presetArgs, ...callArgs]);
  };

  // Named so we can check `this instanceof boundFn`
  var boundFn = arguments.callee; // for simpler versions, skip `new` support
};
```

**Cleaner version (commonly accepted in interviews):**

```js
Function.prototype.myBind = function (context, ...presetArgs) {
  const originalFn = this;

  function boundFn(...callArgs) {
    const ctx = this instanceof boundFn ? this : context;
    return originalFn.apply(ctx, [...presetArgs, ...callArgs]);
  }

  // Preserve prototype chain for `new` usage
  boundFn.prototype = Object.create(originalFn.prototype);
  return boundFn;
};
```

**Key points:**
- Returns a new function — does not call the original immediately
- Merges preset args (`presetArgs`) with call-time args (`callArgs`)
- `this instanceof boundFn` detects `new` usage and bypasses the bound context
- `Object.create(originalFn.prototype)` preserves the prototype chain for constructor use

</details>

---

## Promise Polyfills

---

### 10. `Promise.all()`

Implement a custom `myPromiseAll`.

**Requirements:**
- Accepts an **iterable** of Promises
- Resolves with an **array of results** in the same order as input when all resolve
- **Rejects immediately** if any Promise rejects (fail-fast)
- Non-Promise values are treated as already-resolved values
- Empty iterable resolves with `[]`

```js
// Expected usage:
myPromiseAll([Promise.resolve(1), Promise.resolve(2), 3])
  .then(console.log); // [1, 2, 3]

myPromiseAll([Promise.resolve(1), Promise.reject('err')])
  .catch(console.log); // 'err'
```

<details>
<summary>Show Implementation</summary>

```js
function myPromiseAll(iterable) {
  return new Promise((resolve, reject) => {
    const promises = Array.from(iterable);
    const results = [];
    let remaining = promises.length;

    if (remaining === 0) {
      resolve([]);
      return;
    }

    promises.forEach((p, i) => {
      Promise.resolve(p).then(value => {
        results[i] = value;       // preserve order
        remaining--;
        if (remaining === 0) resolve(results);
      }).catch(reject);           // fail-fast on first rejection
    });
  });
}
```

**Key points:**
- `results[i] = value` preserves the input order (not completion order)
- `remaining` counter tracks how many are still pending
- `Promise.resolve(p)` wraps non-Promise values safely
- `.catch(reject)` is called on the outer `reject` — first rejection wins

</details>

---

### 11. `Promise.allSettled()`

Implement a custom `myPromiseAllSettled`.

**Requirements:**
- Accepts an iterable of Promises
- **Never rejects** — always resolves with an array of result objects
- Each result object has `{ status: 'fulfilled', value }` or `{ status: 'rejected', reason }`
- Waits for **all** Promises to settle regardless of individual failures

```js
// Expected usage:
myPromiseAllSettled([Promise.resolve(1), Promise.reject('err'), Promise.resolve(3)])
  .then(console.log);
// [
//   { status: 'fulfilled', value: 1 },
//   { status: 'rejected', reason: 'err' },
//   { status: 'fulfilled', value: 3 }
// ]
```

<details>
<summary>Show Implementation</summary>

```js
function myPromiseAllSettled(iterable) {
  const promises = Array.from(iterable);

  return Promise.all(
    promises.map(p =>
      Promise.resolve(p)
        .then(value => ({ status: 'fulfilled', value }))
        .catch(reason => ({ status: 'rejected', reason }))
    )
  );
}
```

**Key points:**
- The elegant trick: wrap each Promise in a new Promise that **always resolves** — to a result descriptor object
- `.then` wraps the value as `{ status: 'fulfilled', value }`
- `.catch` wraps the reason as `{ status: 'rejected', reason }` — converting rejection to resolution
- Then use `Promise.all` on these never-rejecting Promises

</details>

---

### 12. `Promise.race()`

Implement a custom `myPromiseRace`.

**Requirements:**
- Accepts an iterable of Promises
- Resolves or rejects with the **first Promise to settle** (either way)
- All other Promises are ignored after the first settles
- If iterable is empty, the returned Promise **never settles**

```js
// Expected usage:
const fast = new Promise(r => setTimeout(() => r('fast'), 50));
const slow = new Promise(r => setTimeout(() => r('slow'), 200));

myPromiseRace([fast, slow]).then(console.log); // 'fast'
```

<details>
<summary>Show Implementation</summary>

```js
function myPromiseRace(iterable) {
  return new Promise((resolve, reject) => {
    for (const p of iterable) {
      Promise.resolve(p).then(resolve).catch(reject);
    }
    // Empty iterable: the promise never settles (by spec)
  });
}
```

**Key points:**
- Only the **first** `resolve` or `reject` call on a Promise takes effect — subsequent calls are silently ignored (Promise state is immutable once settled)
- So attaching `resolve`/`reject` to all Promises and letting the fastest win is safe
- `Promise.resolve(p)` handles non-Promise values

</details>

---

### 13. `Promise.any()`

Implement a custom `myPromiseAny`.

**Requirements:**
- Accepts an iterable of Promises
- Resolves with the value of the **first fulfilled** Promise
- Only rejects if **all** Promises reject — with an `AggregateError` containing all rejection reasons
- Ignores individual rejections unless all fail

```js
// Expected usage:
myPromiseAny([Promise.reject('e1'), Promise.resolve('ok'), Promise.reject('e2')])
  .then(console.log); // 'ok'

myPromiseAny([Promise.reject('e1'), Promise.reject('e2')])
  .catch(e => console.log(e.errors)); // ['e1', 'e2']
```

<details>
<summary>Show Implementation</summary>

```js
function myPromiseAny(iterable) {
  const promises = Array.from(iterable);

  return new Promise((resolve, reject) => {
    if (promises.length === 0) {
      reject(new AggregateError([], 'All promises were rejected'));
      return;
    }

    const errors = [];
    let rejectedCount = 0;

    promises.forEach((p, i) => {
      Promise.resolve(p)
        .then(resolve) // first fulfillment wins
        .catch(reason => {
          errors[i] = reason; // preserve order of errors
          rejectedCount++;
          if (rejectedCount === promises.length) {
            reject(new AggregateError(errors, 'All promises were rejected'));
          }
        });
    });
  });
}
```

**Key points:**
- Inverse of `Promise.all`: resolves on first success, rejects only if all fail
- Collect errors in order using index (`errors[i]`) — same pattern as `Promise.all` results
- `AggregateError` is the specified error type — it has an `errors` array property

</details>

---

## Utility Polyfills

---

### 14. `debounce()`

Implement a `debounce` utility function.

**Requirements:**
- Returns a new function that delays invoking `fn` until after `delay` ms of silence
- Each call resets the timer
- The debounced function should accept any arguments and pass them to `fn`
- **Bonus:** add a `cancel()` method to cancel any pending invocation

```js
// Expected usage:
const search = debounce(query => fetchResults(query), 300);
input.addEventListener('input', e => search(e.target.value));
// Fires once, 300ms after the user stops typing
```

<details>
<summary>Show Implementation</summary>

```js
function debounce(fn, delay) {
  let timer = null;

  function debounced(...args) {
    clearTimeout(timer);
    timer = setTimeout(() => {
      fn.apply(this, args);
      timer = null;
    }, delay);
  }

  debounced.cancel = function () {
    clearTimeout(timer);
    timer = null;
  };

  return debounced;
}
```

**Key points:**
- Each call clears the previous timer and sets a new one — only the **last call** in a burst fires
- `fn.apply(this, args)` preserves the calling context and arguments
- `.cancel()` allows cleanup (e.g., on component unmount)
- Use case: search inputs, form autosave, window resize handler

</details>

---

### 15. `throttle()`

Implement a `throttle` utility function.

**Requirements:**
- Returns a new function that invokes `fn` **at most once per `interval`** ms
- The first call fires immediately (leading edge)
- Subsequent calls within the interval are ignored
- **Bonus:** add a `cancel()` method

```js
// Expected usage:
const onScroll = throttle(() => updateProgressBar(), 100);
window.addEventListener('scroll', onScroll);
// Fires at most once every 100ms no matter how fast the user scrolls
```

<details>
<summary>Show Implementation</summary>

```js
function throttle(fn, interval) {
  let lastTime = 0;
  let timer = null;

  function throttled(...args) {
    const now = Date.now();
    const remaining = interval - (now - lastTime);

    if (remaining <= 0) {
      // Enough time has passed — fire immediately
      lastTime = now;
      fn.apply(this, args);
    }
    // else: within throttle window — call is dropped
  }

  throttled.cancel = function () {
    clearTimeout(timer);
    timer = null;
    lastTime = 0;
  };

  return throttled;
}
```

**Key points:**
- Tracks `lastTime` — the timestamp of the last actual invocation
- If the gap since the last call is ≥ `interval`, invoke immediately
- Calls within the interval are **silently dropped** (unlike debounce which reschedules)
- Use case: scroll, mousemove, game loop, resize — when you want consistent rate-limiting

</details>

---

### 16. `memoize()`

Implement a `memoize` utility function.

**Requirements:**
- Returns a new function that caches results of previous calls
- Uses arguments as the cache key
- On repeated calls with the same arguments, returns the cached result without re-executing `fn`
- Works for functions with **primitive arguments** (string, number, boolean)

```js
// Expected usage:
const expensiveCalc = memoize(n => {
  console.log('computing...');
  return n * n;
});

expensiveCalc(5); // 'computing...' → 25
expensiveCalc(5); // (no log) → 25  (from cache)
expensiveCalc(6); // 'computing...' → 36
```

<details>
<summary>Show Implementation</summary>

```js
function memoize(fn) {
  const cache = new Map();

  return function (...args) {
    const key = JSON.stringify(args);

    if (cache.has(key)) {
      return cache.get(key);
    }

    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}
```

**Key points:**
- `JSON.stringify(args)` creates a string cache key — works for primitives and simple objects
- `Map` is used (vs plain object) because keys can be any string including `__proto__`
- Limitation: `JSON.stringify` doesn't distinguish `undefined` from missing, and can't handle functions or circular objects as arguments
- Trade-off: unbounded cache growth — in production, add LRU eviction or a size limit

</details>

---

### 17. `curry()`

Implement a `curry` utility function.

**Requirements:**
- Takes a function `fn` and returns a **curried version**
- The curried function can be called with **any number of arguments at a time**
- When enough arguments have been collected (≥ `fn.length`), invoke `fn`
- Supports partial application at each step

```js
// Expected usage:
const add = curry((a, b, c) => a + b + c);
add(1)(2)(3);    // 6
add(1, 2)(3);    // 6
add(1)(2, 3);    // 6
add(1, 2, 3);    // 6
```

<details>
<summary>Show Implementation</summary>

```js
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      // Have enough arguments — invoke the original function
      return fn.apply(this, args);
    }

    // Not enough yet — return a function that collects more
    return function (...moreArgs) {
      return curried.apply(this, [...args, ...moreArgs]);
    };
  };
}
```

**Key points:**
- `fn.length` gives the number of declared parameters — the arity of `fn`
- Each partial application accumulates args using spread
- The inner function calls `curried` recursively — collecting args until enough are gathered
- Limitation: doesn't work with rest parameters (`...args`) since `fn.length` counts only non-rest params

</details>

---

### 18. `once()`

Implement a `once` utility function.

**Requirements:**
- Returns a new function that can only be **invoked once**
- On the first call: executes `fn` and caches the result
- On subsequent calls: returns the **cached result** without re-executing `fn`
- All arguments are passed to `fn` on the first call

```js
// Expected usage:
const init = once(() => {
  console.log('initialized!');
  return 42;
});

init(); // 'initialized!' → 42
init(); // (no log) → 42
init(); // (no log) → 42
```

<details>
<summary>Show Implementation</summary>

```js
function once(fn) {
  let called = false;
  let cachedResult;

  return function (...args) {
    if (!called) {
      called = true;
      cachedResult = fn.apply(this, args);
    }
    return cachedResult;
  };
}
```

**Key points:**
- `called` flag prevents re-execution — set to `true` on first call
- `cachedResult` stores the return value for all subsequent calls
- Even if `fn` returns `undefined`, subsequent calls return `undefined` (not re-execute)
- Use case: SDK initialization, singleton setup, expensive one-time computation

</details>

---

### 19. `compose()`

Implement a `compose` utility function.

**Requirements:**
- Takes multiple functions as arguments
- Returns a new function that applies them **right to left** (mathematical composition)
- Each function receives the result of the previous one
- The **rightmost** function is called first with the initial argument

```js
// Expected usage:
const double = x => x * 2;
const addOne = x => x + 1;
const square = x => x * x;

const transform = compose(double, addOne, square);
transform(3); // double(addOne(square(3))) = double(addOne(9)) = double(10) = 20
```

<details>
<summary>Show Implementation</summary>

```js
function compose(...fns) {
  return function (x) {
    return fns.reduceRight((acc, fn) => fn(acc), x);
  };
}
```

**Key points:**
- `reduceRight` processes functions from right to left — matching mathematical composition `f∘g∘h`
- The initial value `x` is the starting accumulator
- Each function must accept a single argument and return a single value
- `compose(f, g, h)(x)` is equivalent to `f(g(h(x)))`

</details>

---

### 20. `pipe()`

Implement a `pipe` utility function.

**Requirements:**
- Same as `compose()` but applies functions **left to right** (more intuitive for data pipelines)
- The **leftmost** function is called first with the initial argument

```js
// Expected usage:
const double = x => x * 2;
const addOne = x => x + 1;
const square = x => x * x;

const transform = pipe(square, addOne, double);
transform(3); // double(addOne(square(3))) = double(addOne(9)) = double(10) = 20
```

<details>
<summary>Show Implementation</summary>

```js
function pipe(...fns) {
  return function (x) {
    return fns.reduce((acc, fn) => fn(acc), x);
  };
}
```

**Key points:**
- `reduce` (left-to-right) vs `reduceRight` (right-to-left) is the only difference from `compose`
- `pipe(f, g, h)(x)` is equivalent to `h(g(f(x)))` — reads in natural execution order
- More intuitive for data transformation pipelines (like Unix pipes: `cat file | grep | sort`)
- Preferred in functional programming libraries (Ramda's `pipe`, RxJS operators)

</details>

---

### 21. `deepClone()`

Implement a `deepClone` utility function.

**Requirements:**
- Returns a **complete deep copy** of the input — no shared references with the original
- Handles: plain objects, arrays, nested structures, primitives
- Handles **circular references** without infinite recursion
- Does not need to handle: functions, Dates, Maps, Sets (unless you want bonus points)

```js
// Expected usage:
const obj = { a: 1, b: { c: 2 }, d: [3, 4] };
const clone = deepClone(obj);
clone.b.c = 99;
console.log(obj.b.c); // 2 — original unchanged
```

<details>
<summary>Show Implementation</summary>

```js
function deepClone(value, seen = new WeakMap()) {
  // Primitives — return as-is
  if (value === null || typeof value !== 'object') return value;

  // Circular reference check
  if (seen.has(value)) return seen.get(value);

  // Arrays
  if (Array.isArray(value)) {
    const clone = [];
    seen.set(value, clone);
    for (const item of value) {
      clone.push(deepClone(item, seen));
    }
    return clone;
  }

  // Plain objects
  const clone = {};
  seen.set(value, clone); // register before recursing (circular ref safety)
  for (const key of Object.keys(value)) {
    clone[key] = deepClone(value[key], seen);
  }
  return clone;
}

// Modern one-liner (if environment supports it):
// const deepClone = value => structuredClone(value);
```

**Key points:**
- `WeakMap` tracks already-cloned objects — if we see the same reference again, return the clone (not the original) to break the cycle
- Register the clone in `seen` **before** recursing into its properties — otherwise circular refs would still infinite-loop
- `structuredClone()` is the native built-in (available in modern browsers and Node 17+) — mention it in interviews

</details>

---

### 22. `EventEmitter`

Implement a simple `EventEmitter` class.

**Requirements:**
- `on(event, listener)` — subscribe to an event
- `off(event, listener)` — unsubscribe a specific listener
- `emit(event, ...args)` — trigger all listeners for an event, passing args
- `once(event, listener)` — subscribe for a single emission, auto-removes after firing

```js
// Expected usage:
const emitter = new EventEmitter();
emitter.on('data', val => console.log('received:', val));
emitter.emit('data', 42); // 'received: 42'
emitter.emit('data', 99); // 'received: 99'

const handler = val => console.log('once:', val);
emitter.once('end', handler);
emitter.emit('end', 'done'); // 'once: done'
emitter.emit('end', 'again'); // (nothing)
```

<details>
<summary>Show Implementation</summary>

```js
class EventEmitter {
  constructor() {
    this.events = {}; // { eventName: [listener1, listener2, ...] }
  }

  on(event, listener) {
    if (!this.events[event]) this.events[event] = [];
    this.events[event].push(listener);
    return this; // allow chaining
  }

  off(event, listener) {
    if (!this.events[event]) return this;
    this.events[event] = this.events[event].filter(l => l !== listener);
    return this;
  }

  emit(event, ...args) {
    if (!this.events[event]) return false;
    // Slice to avoid issues if a listener calls off() during emit
    this.events[event].slice().forEach(listener => listener(...args));
    return true;
  }

  once(event, listener) {
    const wrapper = (...args) => {
      listener(...args);
      this.off(event, wrapper); // auto-remove after first call
    };
    wrapper._original = listener; // store reference for off() compatibility
    return this.on(event, wrapper);
  }
}
```

**Key points:**
- `this.events` is a plain object mapping event names to listener arrays
- `.slice()` in `emit` creates a snapshot — safe if listeners modify the array during emission
- `once` wraps the listener in a function that calls `off` after the first invocation
- Store `_original` on the wrapper if you want `off(event, originalFn)` to work with `once` listeners

</details>

---

## Quick Reference

| Polyfill | Key Concept |
|----------|------------|
| `map` | New array, fn per element, preserve holes |
| `filter` | New array, push only truthy results |
| `reduce` | Accumulator, handle no-initial-value edge case |
| `forEach` | Side effects only, always returns `undefined` |
| `find` | First match, short-circuit, returns element or `undefined` |
| `flat` | Recursive, depth-limited, `Infinity` for full flatten |
| `call` | Attach fn as temp property on context, then call |
| `apply` | Same as `call` but args as array |
| `bind` | Return new function, merge preset + call args, handle `new` |
| `Promise.all` | All resolve → array; first reject → reject |
| `Promise.allSettled` | Never rejects; wraps each in always-resolving Promise |
| `Promise.race` | First to settle (resolve or reject) wins |
| `Promise.any` | First to resolve wins; all reject → AggregateError |
| `debounce` | Clear + reset timer on each call; fires after silence |
| `throttle` | Track lastTime; drop calls within interval |
| `memoize` | Cache by JSON key; return cached if hit |
| `curry` | Collect args until arity met, then call |
| `once` | `called` flag; cache result; ignore subsequent calls |
| `compose` | `reduceRight` — right to left |
| `pipe` | `reduce` — left to right |
| `deepClone` | Recursive + WeakMap for circular ref safety |
| `EventEmitter` | Map of event → listeners; `once` wraps with auto-off |

---

*JavaScript Interview Prep — Polyfills*
