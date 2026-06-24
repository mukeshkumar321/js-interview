# Chapter 2: Closures

> **Target:** Frontend Developer (4+ years) | Interview-focused, practical explanations

---

## Table of Contents

- [Closure Fundamentals](#closure-fundamentals)
- [Practical Applications](#practical-applications)
- [Memory & Performance](#memory--performance)
- [Patterns & Real-World Usage](#patterns--real-world-usage)

---

## Closure Fundamentals

<details>
<summary><strong>1. What is a Closure and why does it exist in JavaScript?</strong></summary>

A **closure** is a function that **remembers the variables from its outer scope** even after that outer function has finished executing.

```js
function outer() {
  let count = 0;

  return function inner() {
    count++;
    console.log(count);
  };
}

const increment = outer();
increment(); // 1
increment(); // 2
increment(); // 3
```

`outer()` has finished, but `inner` still has access to `count`. That's a closure.

**Why it exists:**  
JavaScript is lexically scoped — functions carry their scope with them. When a function is returned or passed around, it takes its birth-scope's environment along. This is a natural consequence of how the engine handles Lexical Environments.

> **Interview Note:** The one-liner definition: *"A closure is a function bundled together with references to its surrounding lexical environment."* Always follow up with a practical example — interviewers want to see you apply it, not just define it.

</details>

---

<details>
<summary><strong>2. How are Closures related to Lexical Scope?</strong></summary>

Closures are **built on top of lexical scope**. Lexical scope determines which variables a function can see at write time. A closure is what happens when that function is used outside its original scope — it holds onto the Lexical Environment where it was created.

```js
function makeGreeter(greeting) {
  return function (name) {   // closes over `greeting`
    return `${greeting}, ${name}!`;
  };
}

const hello = makeGreeter('Hello');
const hi = makeGreeter('Hi');

hello('Alice'); // "Hello, Alice!"
hi('Bob');      // "Hi, Bob!"
```

Each call to `makeGreeter` creates a **new Lexical Environment** with its own `greeting`. The returned function closes over that specific environment.

> **Key distinction:** Lexical scope = the rule. Closure = the mechanism that preserves it across time.

</details>

---

<details>
<summary><strong>3. When is a Closure created?</strong></summary>

A closure is created **every time a function is defined inside another function** — whether or not it's returned or used later. The act of definition creates the closure.

```js
function outer() {
  const x = 10;

  // Closure created here — even if never returned
  function inner() {
    console.log(x);
  }

  inner(); // called directly — still a closure
}
```

**More common scenarios:**
```js
// 1. Returned function
function counter() {
  let n = 0;
  return () => ++n;
}

// 2. Passed as callback
function setup() {
  const msg = 'clicked';
  btn.addEventListener('click', () => console.log(msg)); // closure
}

// 3. Stored in an object
function makeObj() {
  let private = 0;
  return {
    get: () => private,
    set: (v) => { private = v; }
  };
}
```

> **Common Mistake:** Thinking closures only exist when a function is *returned*. Any inner function that references an outer variable is a closure.

</details>

---

<details>
<summary><strong>4. How does JavaScript preserve variables after the outer function has finished execution?</strong></summary>

When a function finishes, its Execution Context is popped off the call stack. Normally, its local variables are eligible for garbage collection.

But if an inner function still holds a **reference to the outer Lexical Environment**, the garbage collector cannot free those variables — the reference keeps them alive in memory.

```js
function outer() {
  let data = 'important'; // normally freed after outer() returns

  return function inner() {
    console.log(data);    // `data` must stay alive — closure holds the reference
  };
}

const fn = outer(); // outer is done, but `data` is NOT freed
fn();               // 'important' — still accessible
```

**Internally:**  
The inner function object has a hidden `[[Environment]]` property that points to the Lexical Environment of `outer`. As long as `fn` exists, that environment (and `data`) stays in memory.

</details>

---

<details>
<summary><strong>5. What is the difference between Scope and Closure?</strong></summary>

| | Scope | Closure |
|--|-------|---------|
| **What it is** | The rule of variable visibility | A function + its captured environment |
| **When** | Determined at write time | Activated when function runs outside its scope |
| **Lifetime** | Exists while block/function runs | Persists as long as the function reference exists |
| **Purpose** | Controls access | Preserves state across calls |

```js
function foo() {
  let x = 10; // x is in scope within foo
  return () => x; // closure captures x beyond foo's lifetime
}
```

Scope answers: *"Can this code access this variable?"*  
Closure answers: *"Can this variable outlive its original scope?"*

</details>

---

## Practical Applications

<details>
<summary><strong>6. How are Closures used to create Private Variables and Data Hiding?</strong></summary>

JavaScript has no `private` keyword for plain functions. Closures simulate privacy by keeping variables inside an outer function's scope — inaccessible from outside.

```js
function createCounter() {
  let count = 0; // private — not accessible directly

  return {
    increment() { count++; },
    decrement() { count--; },
    getCount()  { return count; }
  };
}

const counter = createCounter();
counter.increment();
counter.increment();
counter.getCount(); // 2
counter.count;      // undefined — no direct access
```

The `count` variable is completely hidden. The only way to interact with it is through the returned methods — exactly like a private field.

> **Interview Note:** This pattern predates ES2022 private class fields (`#field`). Interviewers may ask you to implement the same thing both ways — know both.

```js
// ES2022 equivalent
class Counter {
  #count = 0;
  increment() { this.#count++; }
  getCount()  { return this.#count; }
}
```

</details>

---

<details>
<summary><strong>7. How is Memoization implemented using Closures?</strong></summary>

Memoization caches the results of expensive function calls. A closure keeps the cache alive between calls without exposing it globally.

```js
function memoize(fn) {
  const cache = {}; // private cache via closure

  return function (...args) {
    const key = JSON.stringify(args);

    if (key in cache) {
      console.log('cache hit');
      return cache[key];
    }

    cache[key] = fn(...args);
    return cache[key];
  };
}

const expensiveAdd = memoize((a, b) => {
  console.log('computing...');
  return a + b;
});

expensiveAdd(2, 3); // computing... → 5
expensiveAdd(2, 3); // cache hit → 5
expensiveAdd(4, 5); // computing... → 9
```

**Why closure is essential here:** `cache` is created once and persists across all calls to the memoized function, but is completely hidden from the outside world.

> **Interview Note:** Memoization is one of the most common closure interview questions. Be ready to implement it from scratch and explain the `JSON.stringify(args)` key strategy and its limitations (fails for functions, circular refs, etc.).

</details>

---

<details>
<summary><strong>8. How are Closures used in Event Handlers and Callbacks?</strong></summary>

Event handlers frequently close over outer variables to capture state at registration time.

```js
function setupButtons() {
  const buttons = document.querySelectorAll('button');

  buttons.forEach((btn, index) => {
    btn.addEventListener('click', function () {
      console.log(`Button ${index} clicked`); // closes over `index`
    });
  });
}
```

Each callback closes over its own `index` from the `forEach` iteration.

**Real-world React example:**
```js
function TodoItem({ id, onDelete }) {
  // The handler closes over `id`
  const handleClick = () => onDelete(id);
  return <button onClick={handleClick}>Delete</button>;
}
```

**Common bug with `var` (before `let`):**
```js
for (var i = 0; i < 3; i++) {
  document.getElementById(`btn${i}`).addEventListener('click', () => {
    console.log(i); // always 3 — all handlers share the same `i`
  });
}
```

> **Common Mistake:** Registering event handlers in a loop with `var`. The classic fix is `let` or an IIFE to create a new scope per iteration.

</details>

---

<details>
<summary><strong>9. How are Closures used in React Hooks and Components?</strong></summary>

React function components and hooks are **heavily closure-based**.

**useState / useEffect stale closure problem:**
```js
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      console.log(count); // ⚠️ stale closure — captures count = 0
      setCount(count + 1); // always sets to 1
    }, 1000);
    return () => clearInterval(id);
  }, []); // empty deps — closes over the initial `count`
}
```

**Fix — functional update:**
```js
setCount(prev => prev + 1); // doesn't rely on closed-over `count`
```

**Fix — include in deps:**
```js
useEffect(() => { ... }, [count]); // re-runs when count changes
```

**Custom Hook using closure:**
```js
function useCounter(initial = 0) {
  const [count, setCount] = useState(initial);
  const increment = useCallback(() => setCount(c => c + 1), []);
  return { count, increment };
}
```

> **Interview Note:** The stale closure in `useEffect` is a top React interview question for senior developers. Always explain it in terms of closures — each render creates a new closure that captures the values from that render.

</details>

---

<details>
<summary><strong>10. What is a Function Factory and how do Closures enable it?</strong></summary>

A **function factory** is a function that returns customized functions. Closures enable each returned function to carry its own configuration.

```js
function multiplier(factor) {
  return (number) => number * factor; // closes over `factor`
}

const double = multiplier(2);
const triple = multiplier(3);

double(5); // 10
triple(5); // 15
```

Each call to `multiplier` creates a new Lexical Environment with its own `factor`. The returned functions are independent — they don't share state.

**Practical example — validator factory:**
```js
function createRangeValidator(min, max) {
  return (value) => {
    if (value < min) return `Must be at least ${min}`;
    if (value > max) return `Must be at most ${max}`;
    return null;
  };
}

const validateAge   = createRangeValidator(0, 120);
const validateScore = createRangeValidator(0, 100);

validateAge(25);    // null (valid)
validateScore(150); // "Must be at most 100"
```

> **Interview Note:** Function factories demonstrate closures at their most practical. Interviewers often ask you to refactor repetitive code into a factory — recognize that pattern.

</details>

---

## Memory & Performance

<details>
<summary><strong>11. How does Garbage Collection work with Closures?</strong></summary>

JavaScript uses **mark-and-sweep** garbage collection. An object is collected when no references to it remain reachable from the root (global scope, active call stack).

With closures, the outer function's Lexical Environment is kept alive as long as **any closure that references it** is still reachable.

```js
function outer() {
  let bigData = new Array(1000000).fill('x'); // 1MB

  return function inner() {
    console.log(bigData.length);
  };
}

let fn = outer(); // bigData is kept alive — fn holds the reference
fn = null;        // now fn is gone → bigData is eligible for GC
```

Setting `fn = null` removes the last reference to `inner`, which removes the last reference to `bigData` — allowing it to be garbage collected.

> **Key insight:** The GC doesn't collect variables individually — it collects entire Lexical Environments. If your closure only uses one variable from an outer scope of 10 variables, all 10 stay in memory.

</details>

---

<details>
<summary><strong>12. Can Closures cause Memory Leaks?</strong></summary>

Yes — closures cause memory leaks when they **unintentionally keep large objects alive** longer than needed.

**Scenario 1: DOM element retained in closure**
```js
function setup() {
  const element = document.getElementById('huge-table'); // large DOM node

  element.addEventListener('click', function handler() {
    console.log('clicked');
    // `element` is closed over — even after removal from DOM, it stays in memory
  });
}
```

Even if `element` is removed from the DOM, the event handler closure still holds a reference to it — preventing GC.

**Fix:**
```js
element.addEventListener('click', handler);
// When done:
element.removeEventListener('click', handler);
element = null;
```

**Scenario 2: Accidental global via closure**
```js
function leak() {
  const data = new Array(1000000).fill('x');
  window.leakyFn = () => console.log(data.length); // closure attached to global
}
leak();
// `data` never freed — window.leakyFn holds the reference forever
```

> **Interview Note:** In React, the most common leak is registering event listeners or intervals in `useEffect` without returning a cleanup function.

</details>

---

<details>
<summary><strong>13. What are common Closure-related memory issues in frontend applications?</strong></summary>

**1. Forgotten event listeners:**
```js
// ❌ Leak — registered but never removed
window.addEventListener('resize', () => {
  console.log(someHugeObject.data);
});

// ✅ Fix — cleanup in useEffect
useEffect(() => {
  const handler = () => console.log(someHugeObject.data);
  window.addEventListener('resize', handler);
  return () => window.removeEventListener('resize', handler);
}, []);
```

**2. setInterval not cleared:**
```js
// ❌ Leak — interval keeps closure alive forever
setInterval(() => {
  updateUI(largeDataSet);
}, 1000);

// ✅ Fix
const id = setInterval(...);
return () => clearInterval(id);
```

**3. Stale closures in React holding old state:**
```js
// ❌ Stale ref — previous render's `data` held in memory
useEffect(() => {
  socket.on('message', (msg) => processWithData(data));
}, []); // data never updates in handler
```

**4. Closure in cache growing unbounded:**
```js
// ❌ Cache never expires — grows forever
const cache = {};
const memoized = (key) => {
  if (!cache[key]) cache[key] = expensiveCompute(key);
  return cache[key];
};
```

> Add a max-size eviction policy (LRU) for production memoization.

</details>

---

## Patterns & Real-World Usage

<details>
<summary><strong>14. What is the Module Pattern and how does it use Closures?</strong></summary>

The **Module Pattern** uses an IIFE to create a private scope, exposing only what's needed via a returned object. It was the standard way to write modular JS before ES Modules.

```js
const BankAccount = (function () {
  let balance = 0; // private

  function validateAmount(amount) { // private
    return amount > 0;
  }

  return {
    deposit(amount) {
      if (validateAmount(amount)) balance += amount;
    },
    withdraw(amount) {
      if (validateAmount(amount) && balance >= amount) balance -= amount;
    },
    getBalance() {
      return balance;
    }
  };
})();

BankAccount.deposit(100);
BankAccount.getBalance(); // 100
BankAccount.balance;      // undefined — private
```

The IIFE runs once, creating a closure. The returned object's methods all close over the same `balance` variable.

> **Interview Note:** ES Modules (`import`/`export`) have largely replaced this pattern for new code, but it appears in legacy codebases and bundled libraries. Understanding it shows JS fundamentals depth.

</details>

---

<details>
<summary><strong>15. What are the most common real-world use cases of Closures?</strong></summary>

| Use Case | Description |
|----------|-------------|
| **Private state** | Counter, store, config hidden from outside |
| **Memoization / Caching** | Cache persists across calls without globals |
| **Function factories** | Create customized functions (validators, formatters) |
| **Partial application / Currying** | Pre-fill arguments |
| **Event handlers** | Capture loop index or outer state at registration |
| **React hooks** | `useState`, `useCallback`, `useMemo` all rely on closures |
| **Module pattern** | Pre-ES6 encapsulation |
| **Debounce / Throttle** | Timer reference kept in closure |

**Debounce (classic closure example):**
```js
function debounce(fn, delay) {
  let timer; // private — persists across calls via closure

  return function (...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}

const search = debounce((query) => fetchResults(query), 300);
input.addEventListener('input', (e) => search(e.target.value));
```

`timer` is a private variable that persists between calls — exactly what makes debounce work. Without closure, you'd need a global.

> **Interview Note:** Implementing `debounce` or `throttle` from scratch is a top 5 JavaScript interview question. Always explain *why* the closure is necessary — to keep `timer` alive between calls without polluting global scope.

</details>

---

## Quick Reference Cheat Sheet

```
Closure = Function + [[Environment]] reference to outer Lexical Environment

Created:  Every time a function is defined inside another function
Persists: As long as the inner function reference is reachable
Freed:    When no references to the inner function remain
```

| Pattern | Uses Closure For |
|---------|-----------------|
| Counter / Private state | Persisting private variable across calls |
| Memoization | Persisting cache without globals |
| Debounce / Throttle | Persisting timer reference |
| Function factory | Capturing config per factory call |
| Module pattern | Encapsulating private scope |
| React `useEffect` | Capturing render-time values |

---

*Chapter 2 of JavaScript Interview Prep Series*
