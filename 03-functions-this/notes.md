# Chapter 3: Functions & `this`

> **Target:** Frontend Developer (4+ years) | Interview-focused, practical explanations

---

## Table of Contents

- [Functions Fundamentals](#functions-fundamentals)
- [The `this` Keyword](#the-this-keyword)
- [call, apply & bind](#call-apply--bind)

---

## Functions Fundamentals

<details>
<summary><strong>1. What are First-Class Functions in JavaScript?</strong></summary>

In JavaScript, functions are **first-class citizens** — they can be:
- Assigned to variables
- Passed as arguments to other functions
- Returned from functions
- Stored in data structures

```js
// Assigned to variable
const greet = function (name) { return `Hello, ${name}`; };

// Passed as argument
[1, 2, 3].map(n => n * 2);

// Returned from function
function multiplier(x) {
  return (y) => x * y; // function returned
}

// Stored in object
const utils = {
  double: (n) => n * 2,
  triple: (n) => n * 3
};
```

**Why it matters:** First-class functions are the foundation for higher-order functions, closures, callbacks, and every functional programming pattern in JavaScript.

</details>

---

<details>
<summary><strong>2. What is the difference between Function Declaration, Function Expression, and Arrow Function?</strong></summary>

| | Declaration | Expression | Arrow |
|--|-------------|------------|-------|
| **Hoisted** | Yes (full body) | No (`undefined` if `var`) | No |
| **`this`** | Dynamic | Dynamic | Lexical (inherits) |
| **`arguments`** | Yes | Yes | No |
| **`new`** | Yes | Yes | No (throws) |
| **Named** | Always | Optional | No |

```js
// Declaration — hoisted, callable before definition
function add(a, b) { return a + b; }

// Expression — not hoisted
const add = function (a, b) { return a + b; };

// Arrow — no own `this`, no `arguments`, can't use `new`
const add = (a, b) => a + b;
```

**When to use which:**
- **Declaration** — top-level utilities, when you need hoisting or recursion by name
- **Expression** — when assigning conditionally or passing as a value
- **Arrow** — callbacks, array methods, anything that should inherit `this` from outer context

> **Interview Note:** The key arrow function distinctions are `this` binding and no `arguments` object. Interviewers often ask "when would you NOT use an arrow function?" — answer: object methods, constructors, `prototype` methods.

</details>

---

<details>
<summary><strong>3. What are Higher-Order Functions and where are they used?</strong></summary>

A **Higher-Order Function (HOF)** is a function that either:
1. Takes one or more functions as arguments, **or**
2. Returns a function

```js
// Takes a function as argument
[1, 2, 3].map(x => x * 2);       // [2, 4, 6]
[1, 2, 3].filter(x => x > 1);    // [2, 3]
[1, 2, 3].reduce((acc, x) => acc + x, 0); // 6

// Returns a function
function withLogging(fn) {
  return function (...args) {
    console.log('calling with', args);
    const result = fn(...args);
    console.log('result:', result);
    return result;
  };
}

const loggedAdd = withLogging((a, b) => a + b);
loggedAdd(2, 3); // logs input and output
```

**Real-world uses:** middleware in Express, React HOCs, debounce/throttle, memoization, decorators.

> **Interview Note:** HOFs enable **function composition** and **separation of concerns** — knowing this shows functional programming awareness.

</details>

---

<details>
<summary><strong>4. What is a Callback Function?</strong></summary>

A **callback** is a function passed to another function to be invoked later — either synchronously or asynchronously.

```js
// Synchronous callback
[1, 2, 3].forEach(function (n) {
  console.log(n); // called immediately, inline
});

// Asynchronous callback
setTimeout(function () {
  console.log('executed later');
}, 1000);

// Node-style error-first callback
fs.readFile('file.txt', function (err, data) {
  if (err) throw err;
  console.log(data);
});
```

Callbacks are the original async pattern in JavaScript — later superseded by Promises and `async/await`, but callbacks are still used in event listeners, array methods, and many APIs.

</details>

---

<details>
<summary><strong>5. What is the difference between Synchronous and Asynchronous Callbacks?</strong></summary>

| | Synchronous | Asynchronous |
|--|-------------|--------------|
| **Execution** | Immediately, inline | Later, after current call stack clears |
| **Blocks** | Yes | No |
| **Examples** | `forEach`, `map`, `sort` | `setTimeout`, `fetch`, event listeners |

```js
// Synchronous — blocks execution
console.log('before');
[1, 2, 3].forEach(n => console.log(n)); // runs immediately
console.log('after');
// before → 1 → 2 → 3 → after

// Asynchronous — non-blocking
console.log('before');
setTimeout(() => console.log('async'), 0);
console.log('after');
// before → after → async
```

> **Common Mistake:** Assuming `setTimeout(fn, 0)` runs immediately. It still goes through the event loop and fires after the current synchronous code finishes.

</details>

---

<details>
<summary><strong>6. What is the difference between Pure and Impure Functions?</strong></summary>

A **pure function**:
1. Given the same inputs, always returns the same output
2. Has no side effects (no mutation, no I/O, no external state)

```js
// Pure
function add(a, b) { return a + b; }
const double = x => x * 2;

// Impure — depends on external state
let tax = 0.1;
function total(price) { return price + price * tax; } // changes with `tax`

// Impure — side effect
function addToCart(item) {
  cart.push(item); // mutates external array
}
```

**Why it matters:**
- Pure functions are **predictable, testable, and cacheable** (memoizable)
- React's rendering model assumes components behave like pure functions

> **Interview Note:** React's `useMemo` and `useCallback` only make sense for pure functions. Interviewers test this when asking about rendering optimization.

</details>

---

<details>
<summary><strong>7. What are Default Parameters and when should they be used?</strong></summary>

Default parameters assign a fallback value when an argument is `undefined` (not passed, or explicitly `undefined`).

```js
function greet(name = 'Guest', greeting = 'Hello') {
  return `${greeting}, ${name}!`;
}

greet();                // "Hello, Guest!"
greet('Alice');         // "Hello, Alice!"
greet('Bob', 'Hi');     // "Hi, Bob!"
greet(undefined, 'Hey'); // "Hey, Guest!" — undefined triggers default
greet(null, 'Hey');     // "Hey, null!" — null does NOT trigger default
```

**Important:** Defaults are evaluated at **call time**, not at definition time — so you can use expressions or even other parameters:
```js
function foo(a, b = a * 2) {
  return b;
}
foo(5); // 10
```

> **Common Mistake:** Thinking `null` triggers the default — only `undefined` does.

</details>

---

<details>
<summary><strong>8. What are Rest Parameters and how do they differ from the <code>arguments</code> object?</strong></summary>

| | `...rest` | `arguments` |
|--|-----------|-------------|
| **Type** | Real Array | Array-like object |
| **Arrow functions** | Yes | No |
| **All args** | Only uncaptured ones | All args |
| **Array methods** | Directly | Needs `Array.from()` |

```js
// Rest — collects remaining args into a real array
function sum(first, ...rest) {
  return rest.reduce((acc, n) => acc + n, first);
}
sum(1, 2, 3, 4); // 10

// arguments — old way, array-like
function oldSum() {
  return Array.from(arguments).reduce((a, b) => a + b, 0);
}

// arguments doesn't exist in arrow functions
const arrow = () => {
  console.log(arguments); // ❌ ReferenceError (or outer arguments in non-strict)
};
```

> **Interview Note:** Always prefer rest parameters in modern code. `arguments` is legacy and unavailable in arrow functions.

</details>

---

<details>
<summary><strong>9. What is Currying and what problems does it solve?</strong></summary>

**Currying** transforms a function that takes multiple arguments into a sequence of functions, each taking one argument.

```js
// Normal
const add = (a, b) => a + b;
add(2, 3); // 5

// Curried
const curriedAdd = a => b => a + b;
curriedAdd(2)(3); // 5

const add2 = curriedAdd(2); // partially applied
add2(3); // 5
add2(10); // 12
```

**Problems it solves:**
1. **Reusability** — create specialized functions from general ones
2. **Partial application** — pre-fill some arguments
3. **Function composition** — compose curried functions cleanly

```js
// Practical: curried validator
const isGreaterThan = min => value => value > min;
const isAdult = isGreaterThan(18);
const isPositive = isGreaterThan(0);

[22, 15, 30, 5].filter(isAdult);   // [22, 30]
[-1, 0, 5, 3].filter(isPositive);  // [5, 3]
```

> **Interview Note:** Interviewers may ask you to implement a generic `curry()` function. Key insight: check `fn.length` against received args count, and keep accumulating until satisfied.

</details>

---

<details>
<summary><strong>10. What is the difference between Currying and Partial Application?</strong></summary>

| | Currying | Partial Application |
|--|----------|---------------------|
| **Args per call** | Exactly 1 | One or more |
| **Calls to resolve** | `n` calls for `n` args | Fewer than original |
| **Returns** | Always a unary function | Function with remaining args |

```js
// Currying — strictly one arg at a time
const curry = a => b => c => a + b + c;
curry(1)(2)(3); // 6

// Partial Application — fix some args, provide rest later
function partial(fn, ...presetArgs) {
  return function (...laterArgs) {
    return fn(...presetArgs, ...laterArgs);
  };
}

const add = (a, b, c) => a + b + c;
const add10 = partial(add, 10);
add10(5, 3); // 18
```

**In practice:** `bind()` is the built-in partial application tool:
```js
function multiply(a, b) { return a * b; }
const double = multiply.bind(null, 2);
double(5); // 10
```

</details>

---

## The `this` Keyword

<details>
<summary><strong>11. What is the <code>this</code> keyword in JavaScript?</strong></summary>

`this` refers to the **execution context** — the object that is currently executing the function. Unlike variables in lexical scope, `this` is determined **at call time**, not at write time (except for arrow functions).

```js
const obj = {
  name: 'Alice',
  greet() {
    console.log(this.name); // `this` = obj at call time
  }
};

obj.greet(); // 'Alice'
```

The value of `this` depends entirely on **how the function is called**, not where it's defined.

> **Interview Note:** The most important sentence about `this`: *"Arrow functions don't have their own `this` — they inherit it from the enclosing lexical scope."* Everything else follows from this.

</details>

---

<details>
<summary><strong>12. How is the value of <code>this</code> determined at runtime?</strong></summary>

JavaScript uses **four binding rules** in order of priority:

1. **`new` binding** — `this` = newly created object
2. **Explicit binding** (`call`/`apply`/`bind`) — `this` = provided object
3. **Implicit binding** (method call) — `this` = object before the dot
4. **Default binding** — `this` = `window` (non-strict) or `undefined` (strict)

```js
function show() { console.log(this); }

// Default binding
show(); // window / undefined (strict)

// Implicit binding
const obj = { show };
obj.show(); // obj

// Explicit binding
show.call({ x: 1 }); // { x: 1 }

// new binding
function Person(name) { this.name = name; }
const p = new Person('Alice'); // p = { name: 'Alice' }
```

</details>

---

<details>
<summary><strong>13. What is the value of <code>this</code> in different contexts?</strong></summary>

**Global Scope:**
```js
console.log(this); // window (browser) | {} (Node module) | global (Node REPL)
```

**Regular Function:**
```js
function foo() { console.log(this); }
foo();        // window (non-strict) | undefined (strict)
```

**Object Method:**
```js
const obj = {
  name: 'Obj',
  greet() { console.log(this.name); }
};
obj.greet(); // 'Obj' — this = obj
```

**Constructor Function:**
```js
function Car(model) { this.model = model; }
const c = new Car('Tesla');
c.model; // 'Tesla' — this = newly created object
```

**Arrow Function:**
```js
const obj = {
  name: 'Obj',
  greet: () => console.log(this.name) // `this` = outer scope (window/undefined)
};
obj.greet(); // undefined — arrow has no own `this`
```

**Inside class:**
```js
class Person {
  constructor(name) { this.name = name; }
  greet() { console.log(this.name); } // this = instance
}
```

</details>

---

<details>
<summary><strong>14. Why do Arrow Functions not have their own <code>this</code>?</strong></summary>

Arrow functions were designed to solve a common problem: `this` losing its context inside callbacks. They achieve this by **not having their own `this`** — instead they inherit `this` from the enclosing lexical scope at definition time.

**The problem arrow functions solve:**
```js
function Timer() {
  this.seconds = 0;

  // ❌ Regular function — `this` is window/undefined inside setInterval
  setInterval(function () {
    this.seconds++; // `this` is NOT the Timer instance here
  }, 1000);
}

function Timer() {
  this.seconds = 0;

  // ✅ Arrow function — `this` is lexically bound to Timer instance
  setInterval(() => {
    this.seconds++; // `this` = Timer instance ✓
  }, 1000);
}
```

**Why NOT to use arrow functions as methods:**
```js
const obj = {
  name: 'Obj',
  greet: () => console.log(this.name) // `this` = window, not obj
};
obj.greet(); // undefined
```

> **Interview Note:** Arrow functions capture `this` from where they are **written** (lexical), not where they are **called** (dynamic). This makes them perfect for callbacks but wrong for object methods.

</details>

---

<details>
<summary><strong>15. What happens to <code>this</code> when an object method is passed as a callback?</strong></summary>

When a method is **detached from its object** and passed as a callback, it loses its implicit binding — `this` falls back to the default binding (`window` or `undefined`).

```js
const obj = {
  name: 'Alice',
  greet() { console.log(this.name); }
};

obj.greet(); // 'Alice' — implicit binding, this = obj

const fn = obj.greet;
fn(); // undefined — default binding, this = window/undefined

setTimeout(obj.greet, 100); // undefined — detached, this = window
```

**Three fixes:**
```js
// 1. bind
setTimeout(obj.greet.bind(obj), 100);

// 2. Arrow wrapper
setTimeout(() => obj.greet(), 100);

// 3. Arrow method (at definition time)
const obj = {
  name: 'Alice',
  greet: () => console.log(this.name) // ⚠️ wrong for methods — loses obj context
};
```

> **Interview Note:** This is the most common `this` bug in production — passing a method as an event handler or callback without binding it. Always use `.bind()` or an arrow wrapper.

</details>

---

<details>
<summary><strong>16. What are the four binding rules of <code>this</code>?</strong></summary>

In priority order (highest → lowest):

**1. `new` binding** — `this` = new object
```js
function Person(n) { this.name = n; }
const p = new Person('Alice'); // this = {}  (new instance)
```

**2. Explicit binding** — `this` = specified object
```js
function greet() { console.log(this.name); }
greet.call({ name: 'Bob' }); // this = { name: 'Bob' }
```

**3. Implicit binding** — `this` = object before the dot
```js
const obj = { name: 'Carol', greet() { console.log(this.name); } };
obj.greet(); // this = obj
```

**4. Default binding** — `this` = global or `undefined`
```js
function greet() { console.log(this); }
greet(); // window (non-strict) | undefined (strict)
```

**Arrow functions** — none of the above applies; always lexical.

> **Interview Note:** Knowing this priority is what lets you predict `this` in any situation. When `new` and explicit binding conflict, `new` wins.

</details>

---

## call, apply & bind

<details>
<summary><strong>17. What is the difference between <code>call()</code>, <code>apply()</code>, and <code>bind()</code>?</strong></summary>

All three explicitly set `this`. The difference is **when** and **how** arguments are passed.

| Method | Calls Immediately | Arguments |
|--------|-------------------|-----------|
| `call` | Yes | Comma-separated: `fn.call(ctx, a, b)` |
| `apply` | Yes | Array: `fn.apply(ctx, [a, b])` |
| `bind` | No (returns new fn) | Comma-separated, can be partial |

```js
function introduce(greeting, punctuation) {
  console.log(`${greeting}, I'm ${this.name}${punctuation}`);
}

const person = { name: 'Alice' };

introduce.call(person, 'Hello', '!');   // Hello, I'm Alice!
introduce.apply(person, ['Hi', '.']);   // Hi, I'm Alice.

const greetAlice = introduce.bind(person, 'Hey');
greetAlice('?'); // Hey, I'm Alice?  — called later
```

**Memory trick:** `call` = **C**omma, `apply` = **A**rray.

</details>

---

<details>
<summary><strong>18. When would you use <code>bind()</code> and why is it important?</strong></summary>

Use `bind()` when you need to **fix `this`** for a function that will be called later — especially callbacks and event handlers.

```js
class Button {
  constructor(label) {
    this.label = label;
    // Without bind, `this` inside handleClick would be the DOM element
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    console.log(`${this.label} clicked`);
  }

  mount(el) {
    el.addEventListener('click', this.handleClick);
  }
}
```

**Partial application with bind:**
```js
function log(level, message) {
  console.log(`[${level}] ${message}`);
}

const warn  = log.bind(null, 'WARN');
const error = log.bind(null, 'ERROR');

warn('Low memory');   // [WARN] Low memory
error('Crash!');      // [ERROR] Crash!
```

> **Interview Note:** In modern React with hooks, class component binding is less common — but the *concept* of `bind` for partial application is still widely used.

</details>

---

<details>
<summary><strong>19. What is Function Borrowing and how do <code>call()</code> and <code>apply()</code> help implement it?</strong></summary>

**Function borrowing** means using a method defined on one object on a completely different object — without copying or redefining it.

```js
const dog = {
  name: 'Rex',
  describe() {
    return `I am ${this.name}, a ${this.type}`;
  }
};

const cat = { name: 'Whiskers', type: 'cat' };

dog.describe.call(cat); // "I am Whiskers, a cat"
```

**Classic real-world use — borrowing `Array` methods for array-like objects:**
```js
function foo() {
  // `arguments` is array-like but has no array methods
  const args = Array.prototype.slice.call(arguments);
  // Now args is a real array
  return args.map(x => x * 2);
}

foo(1, 2, 3); // [2, 4, 6]
```

> **Modern equivalent:** `Array.from(arguments)` or rest parameters `...args` — but the `call` pattern appears heavily in legacy code and is a common interview topic.

</details>

---

<details>
<summary><strong>20. Why are Arrow Functions generally not recommended as object methods?</strong></summary>

Arrow functions inherit `this` from their **lexical scope at definition**, not from the object they're placed in. As an object method, the enclosing scope is typically global — so `this` ends up as `window` or `undefined`.

```js
const obj = {
  name: 'Alice',

  // ❌ Arrow — this = window/undefined, not obj
  greetArrow: () => {
    console.log(this.name); // undefined
  },

  // ✅ Regular method — this = obj (implicit binding)
  greetRegular() {
    console.log(this.name); // 'Alice'
  }
};

obj.greetArrow();   // undefined
obj.greetRegular(); // 'Alice'
```

**When arrow functions ARE useful inside methods:**
```js
const obj = {
  name: 'Alice',
  greet() {
    // Arrow here inherits `this` from greet — which IS obj
    [1, 2, 3].forEach(n => console.log(this.name, n));
  }
};
obj.greet(); // Alice 1, Alice 2, Alice 3
```

> **Rule:** Use regular functions for object methods. Use arrow functions for callbacks *inside* those methods.

</details>

---

<details>
<summary><strong>21. What are common <code>this</code>-related bugs in JavaScript applications?</strong></summary>

**Bug 1: Method passed as callback loses `this`**
```js
class Modal {
  open() { console.log(this); }
  attach(btn) {
    btn.addEventListener('click', this.open); // ❌ this = btn element
    btn.addEventListener('click', this.open.bind(this)); // ✅
  }
}
```

**Bug 2: Arrow function as object method**
```js
const obj = {
  val: 42,
  get: () => this.val // ❌ this = window
};
```

**Bug 3: `this` in nested regular function**
```js
const obj = {
  name: 'Obj',
  greet() {
    function inner() {
      console.log(this.name); // ❌ this = undefined (strict) or window
    }
    inner();
  }
};
// Fix: arrow function or const self = this
```

**Bug 4: Destructuring a method**
```js
const { greet } = obj;
greet(); // ❌ this = undefined — lost implicit binding
```

</details>

---

<details>
<summary><strong>22. What are common real-world use cases of <code>call()</code>, <code>apply()</code>, and <code>bind()</code>?</strong></summary>

**`call` — invoke with specific context:**
```js
// Super constructor call (before class syntax)
function Animal(name) { this.name = name; }
function Dog(name, breed) {
  Animal.call(this, name); // borrow Animal's constructor
  this.breed = breed;
}
```

**`apply` — spread array as arguments:**
```js
const nums = [3, 1, 4, 1, 5, 9];

Math.max.apply(null, nums); // 9
// Modern equivalent:
Math.max(...nums); // 9

// Merging arrays (legacy)
Array.prototype.push.apply(arr1, arr2);
```

**`bind` — event handlers, partial application, React class components:**
```js
// Partial application
const multiply = (a, b) => a * b;
const triple = multiply.bind(null, 3);
triple(7); // 21

// React class component
this.handleSubmit = this.handleSubmit.bind(this);
```

> **Interview Note:** `apply` with `Math.max` is a classic question. The modern answer is spread (`...`), but knowing the `apply` version demonstrates deep JS knowledge.

</details>

---

## Quick Reference Cheat Sheet

```
this value determination (in order of priority):
  1. new binding      → newly created object
  2. call/apply/bind  → explicitly provided object
  3. obj.method()     → obj (implicit binding)
  4. function()       → window (non-strict) | undefined (strict)
  Arrow functions     → always lexical, cannot be changed
```

| Function Type | Has own `this` | Has `arguments` | Can use `new` |
|---------------|---------------|-----------------|----------------|
| Declaration   | Yes (dynamic) | Yes | Yes |
| Expression    | Yes (dynamic) | Yes | Yes |
| Arrow         | No (lexical)  | No  | No  |

| Method | Executes immediately | Argument format |
|--------|---------------------|-----------------|
| `call` | Yes | `(ctx, a, b, c)` |
| `apply`| Yes | `(ctx, [a, b, c])` |
| `bind` | No (returns fn) | `(ctx, a, b)` partial |

---

*Chapter 3 of JavaScript Interview Prep Series*
