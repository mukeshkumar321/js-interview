## Table of Contents

- [Scope & Lexical Environment](#scope--lexical-environment)
- [Execution Context & Call Stack](#execution-context--call-stack)
- [Hoisting](#hoisting)

---

## Scope & Lexical Environment

<details>
<summary><strong>1. What is Scope and why is it important in JavaScript?</strong></summary>

Scope defines **where a variable is accessible** in your code. It's the visibility boundary of a variable.

JavaScript uses **lexical scoping** — scope is determined at write time (where you declare the variable), not at runtime.

```js
const x = 10; // global scope

function foo() {
  const y = 20; // function scope
  console.log(x); // ✅ accessible
}

console.log(y); // ❌ ReferenceError
```

**Why it matters in interviews:**  
Scope directly affects closure behavior, memory leaks, and variable lifetime — all common senior-level questions.

</details>

---

<details>
<summary><strong>2. What are the different types of Scope in JavaScript?</strong></summary>

| Scope | Created by | Accessible |
|-------|-----------|------------|
| **Global** | Top-level declarations | Everywhere |
| **Function** | `function` keyword | Inside that function only |
| **Block** | `{}` with `let`/`const` | Inside that block only |
| **Module** | ES Module (`import`/`export`) | Within the module file |

```js
var a = 1;       // global
function f() {
  var b = 2;     // function scope
  if (true) {
    let c = 3;   // block scope
    const d = 4; // block scope
    var e = 5;   // function scope (leaks out of block!)
  }
  console.log(e); // ✅ 5 — var ignores block boundary
  console.log(c); // ❌ ReferenceError
}
```

> **Interview Note:** The `var` leaking out of `if`/`for` blocks is a classic interview trap. Always highlight that `var` is function-scoped, not block-scoped.

</details>

---

<details>
<summary><strong>3. What is Lexical Scope and how does variable lookup work in nested functions?</strong></summary>

Lexical scope means **a function's scope is determined by where it's written in the source code**, not where it's called from.

When JavaScript can't find a variable in the current scope, it walks **up the scope chain** toward the global scope.

```js
const name = 'Global';

function outer() {
  const name = 'Outer';

  function inner() {
    console.log(name); // 'Outer' — looks up to parent scope
  }

  inner();
}

outer();
```

Even if `inner()` is called from somewhere else, it still remembers the scope where it was **defined** — that's lexical scope.

> **Common Mistake:** Confusing lexical scope with dynamic scope. JavaScript always uses lexical scope. `this` is dynamic, but variable lookup is always lexical.

</details>

---

<details>
<summary><strong>4. What is the Scope Chain and how does JavaScript resolve variables?</strong></summary>

The scope chain is the **linked list of environments** JavaScript traverses when looking up a variable.

**Lookup order:** Current scope → Parent scope → ... → Global scope → `ReferenceError`

```js
const a = 1;

function outer() {
  const b = 2;

  function inner() {
    const c = 3;
    console.log(a, b, c); // 1, 2, 3 — walks up chain
  }

  inner();
}
```

**Internally**, each execution context holds a reference to its **outer lexical environment**. This chain is what makes closures possible.

> **Interview Note:** When asked "how do closures work?", mention the scope chain — a closure keeps a reference to its outer scope's environment, not just the values.

</details>

---

<details>
<summary><strong>5. Why are <code>let</code> and <code>const</code> block-scoped while <code>var</code> is function-scoped?</strong></summary>

`var` was JavaScript's original declaration — it was designed before block scoping was considered necessary. ES6 introduced `let` and `const` to fix predictability issues with `var`.

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // 3, 3, 3 — classic bug
}

for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // 0, 1, 2 — each iteration gets its own `i`
}
```

`let`/`const` create a **new binding per block**, while `var` shares one binding across the entire function.

> **Interview Note:** The `for` loop with `var` vs `let` is one of the most asked JavaScript interview questions. The `let` version works because each iteration creates a new block scope with its own `i`.

> **Common Mistake:** Using `var` inside loops with async callbacks. Always use `let` inside loops.

</details>

---

<details>
<summary><strong>6. What is Variable Shadowing and Illegal Shadowing?</strong></summary>

**Variable Shadowing** — a variable in an inner scope has the same name as one in an outer scope, hiding the outer one.

```js
let x = 10;

function foo() {
  let x = 20; // shadows outer x
  console.log(x); // 20
}

console.log(x); // 10
```

**Illegal Shadowing** — you cannot shadow a `let` with a `var` in the same or nested block, because `var` leaks out of the block and would collide.

> **Note:** The two snippets below are separate examples. If you put them in the same script, the `SyntaxError` from the first would prevent any code from running. They illustrate different behaviours — not a single runnable program.

```js
// Example A — illegal: var cannot shadow a let in the same scope
let a = 5;
{
  var a = 10; // ❌ SyntaxError: Identifier 'a' has already been declared
}

// Example B — legal: let can shadow a var
var b = 5;
{
  let b = 10; // ✅ allowed — let is contained to block
}
```

> **Rule:** `var` can shadow `var`; `let`/`const` can shadow anything; `var` **cannot** shadow `let`/`const` in the same or enclosing scope.

</details>

---

<details>
<summary><strong>7. What happens when JavaScript cannot find a variable in the Scope Chain?</strong></summary>

- **Reading** an undeclared variable → `ReferenceError`
- **Writing** to an undeclared variable (non-strict mode) → creates a global variable (silent bug)
- **Writing** to an undeclared variable (strict mode) → `ReferenceError`

```js
// Non-strict mode
function foo() {
  x = 10; // no declaration — creates global `x`
}
foo();
console.log(x); // 10 — pollutes global scope

// Strict mode
'use strict';
function bar() {
  y = 20; // ❌ ReferenceError: y is not defined
}
```

> **Interview Note:** Always mention `'use strict'` as a safeguard. Modern ES modules are strict by default.

</details>

---

<details>
<summary><strong>8. What is the Lexical Environment and how is it related to Scope?</strong></summary>

A **Lexical Environment** is a data structure (internal to the JS engine) that stores:
1. **Environment Record** — the actual variable/function bindings
2. **Reference to outer Lexical Environment** — parent scope link

Every time a function is called or a block is entered, a new Lexical Environment is created.

```
Global LE
  └── outer() LE
        └── inner() LE
```

**Scope** is the conceptual idea; **Lexical Environment** is the concrete implementation of scope in the engine.

> **Interview Note:** When asked about closures at a deep level, explaining Lexical Environment shows strong internals knowledge. A closure = function + reference to its outer Lexical Environment.

</details>

---

## Execution Context & Call Stack

<details>
<summary><strong>9. What is an Execution Context?</strong></summary>

An **Execution Context (EC)** is the environment in which JavaScript code runs. It contains everything needed to execute a piece of code: variable bindings, the scope chain, and the value of `this`.

Three types:
- **Global EC** — created once when the script starts
- **Function EC** — created each time a function is invoked
- **Eval EC** — created inside `eval()` (avoid in production)

```js
// Global EC is created here
const x = 10;

function greet() {
  // Function EC created when greet() is called
  const msg = 'Hello';
  console.log(msg);
}

greet(); // Function EC pushed onto call stack, then popped
```

</details>

---

<details>
<summary><strong>10 & 11. What are the two phases of an Execution Context? What happens during the Creation Phase?</strong></summary>

Every Execution Context has two phases:

### Phase 1: Creation Phase (Memory Allocation)
Before any code runs, the engine scans the function/scope and:
- Creates the `Variable Object` / Environment Record
- Allocates memory for all `var` declarations (initialized to `undefined`)
- Allocates memory for function declarations (stored as full function objects)
- `let`/`const` are registered but **not initialized** (enter TDZ)
- Sets up the scope chain
- Determines value of `this`

### Phase 2: Execution Phase
Code runs line by line:
- Variables are assigned their actual values
- Function calls create new Execution Contexts

```js
console.log(a); // undefined (creation phase set it to undefined)
console.log(b); // ❌ ReferenceError (TDZ — not initialized yet)

var a = 5;
let b = 10;
```

> **Interview Note:** The Creation Phase is the engine's "first pass" — this is what makes hoisting possible. It's not magic; it's a two-pass process.

</details>

---

<details>
<summary><strong>12. What is the difference between Global Execution Context and Function Execution Context?</strong></summary>

| | Global EC | Function EC |
|--|-----------|-------------|
| **Created** | Once, on script load | Each function call |
| **`this`** | `window` (browser) / `global` (Node) | Depends on call site |
| **Variable Object** | Global object (`window`) | Arguments object + local vars |
| **Outer env reference** | `null` | Parent scope |

```js
function multiply(a, b) {
  // Function EC created with:
  // - arguments: { 0: 3, 1: 4 }
  // - this: window (non-strict) or undefined (strict)
  // - outer: Global EC
  return a * b;
}

multiply(3, 4);
```

</details>

---

<details>
<summary><strong>13. What is the Call Stack and how does it work?</strong></summary>

The **Call Stack** is a LIFO (Last In, First Out) data structure that tracks active Execution Contexts.

```
Global EC → pushed on load
  greet() EC → pushed when called
    getName() EC → pushed when called
    getName() EC → popped when returned
  greet() EC → popped when returned
Global EC → popped when script ends
```

```js
function getName() {
  return 'Alice';
}

function greet() {
  const name = getName();
  console.log('Hello, ' + name);
}

greet();
// Call stack at peak: [Global EC | greet EC | getName EC]
```

The browser's DevTools "Sources" tab shows the call stack in real time during debugging.

> **Interview Note:** The call stack has a fixed size (~10,000–15,000 frames depending on the engine). Exceeding it causes a stack overflow.

</details>

---

<details>
<summary><strong>14. What causes "Maximum Call Stack Size Exceeded"?</strong></summary>

Infinite or very deep recursion without a base case exhausts the call stack.

```js
// Stack overflow
function recurse() {
  recurse(); // no base case
}
recurse(); // ❌ RangeError: Maximum call stack size exceeded

// Fixed
function recurse(n) {
  if (n <= 0) return; // base case
  recurse(n - 1);
}
recurse(100); // ✅
```

**Real-world cause:** Circular references in recursive tree/graph traversal, or accidentally calling a function inside itself in event handlers.

**Workaround for very deep recursion:** Use **trampolining** (a technique where each recursive step returns a function instead of calling itself directly, so the call stack never grows) or convert to an iterative approach with an explicit stack.

```js
// Trampoline pattern
function trampoline(fn) {
  return function(...args) {
    let result = fn(...args);
    while (typeof result === 'function') result = result();
    return result;
  };
}
```

</details>

---

## Hoisting

<details>
<summary><strong>15. What is Hoisting?</strong></summary>

**Hoisting** is JavaScript's behavior of processing declarations before executing code. It's a result of the **Creation Phase** of the Execution Context — not actual code movement.

Developers often say "declarations are moved to the top," but what really happens is the engine allocates memory for them in the creation phase, before execution begins.

```js
console.log(foo); // undefined (not ReferenceError!)
foo();             // ✅ works
bar();             // ❌ TypeError: bar is not a function

var foo = 'value';

function foo() {} // function declaration — fully hoisted

var bar = function() {}; // function expression — only `var bar` is hoisted
```

> **Interview Note:** Saying "code is moved to the top" is a simplification. The correct answer is: during the Creation Phase, the engine scans for declarations and allocates memory — `var` gets `undefined`, function declarations get the full function body, and `let`/`const` get placed in the TDZ.

</details>

---

<details>
<summary><strong>16. Is JavaScript actually moving code to the top during hoisting?</strong></summary>

**No.** The source code doesn't change. Hoisting is a mental model for what happens internally during the **Creation Phase**:

1. Engine scans the scope for all declarations
2. Allocates memory for them **before** executing any code
3. `var` → initialized to `undefined`
4. Function declarations → initialized with the full function
5. `let`/`const` → allocated but **not initialized** (TDZ)

```js
// What you write:
console.log(x);
var x = 5;

// What the engine effectively does in memory:
// [Creation Phase]: x = undefined
// [Execution Phase]:
//   console.log(x) → undefined
//   x = 5
```

</details>

---

<details>
<summary><strong>17. How are <code>var</code>, <code>let</code>, and <code>const</code> hoisted differently?</strong></summary>

| Declaration | Hoisted? | Initial Value | Scope |
|-------------|----------|---------------|-------|
| `var` | Yes | `undefined` | Function |
| `let` | Yes* | TDZ (uninitialized) | Block |
| `const` | Yes* | TDZ (uninitialized) | Block |

*`let`/`const` are technically hoisted (the binding is registered), but accessing them before declaration throws a `ReferenceError` due to the Temporal Dead Zone.

```js
console.log(a); // undefined
console.log(b); // ❌ ReferenceError: Cannot access 'b' before initialization
console.log(c); // ❌ ReferenceError

var a = 1;
let b = 2;
const c = 3;
```

> **Common Mistake:** Saying `let`/`const` are "not hoisted." They ARE hoisted — the block knows about them from the start (you can't redeclare them), but they're in the TDZ until the declaration line is reached.

</details>

---

<details>
<summary><strong>18. What is the Temporal Dead Zone (TDZ) and why does it exist?</strong></summary>

The **Temporal Dead Zone** is the period between entering a block scope and the actual declaration line of a `let`/`const` variable.

Accessing the variable in this window throws a `ReferenceError`.

```js
{
  // TDZ for `x` starts here
  console.log(x); // ❌ ReferenceError: Cannot access 'x' before initialization
  let x = 5;      // TDZ ends here
  console.log(x); // ✅ 5
}
```

**Why TDZ exists:**  
`var`'s silent `undefined` caused hard-to-find bugs. TDZ makes the mistake **loud and early** — fail fast instead of silently failing later.

**TDZ with parameters:**
```js
function foo(a = b, b = 2) {} // ❌ ReferenceError — `b` is in TDZ when `a` is evaluated
function bar(a = 1, b = a) {} // ✅
```

> **Interview Note:** TDZ is intentional by design — it enforces "declare before use" discipline. Interviewers love asking why `let`/`const` still throw if they're "hoisted."

</details>

---

<details>
<summary><strong>19. How are Function Declarations, Function Expressions, and Arrow Functions hoisted differently?</strong></summary>

```js
// 1. Function Declaration — fully hoisted (body available before declaration)
greet(); // ✅ "Hello"
function greet() { console.log("Hello"); }

// 2. Function Expression with var — var is hoisted as undefined, not the function
sayHi(); // ❌ TypeError: sayHi is not a function
var sayHi = function() { console.log("Hi"); };

// 3. Function Expression with let/const — TDZ applies
sayBye(); // ❌ ReferenceError
const sayBye = function() { console.log("Bye"); };

// 4. Arrow Function — same as function expression (const/let/var)
sayHey(); // ❌ TypeError or ReferenceError depending on declaration
var sayHey = () => console.log("Hey");
```

| Type | Declaration hoisted | Body hoisted | Callable before declaration |
|------|--------------------|--------------|-----------------------------|
| Function Declaration | ✅ | ✅ | ✅ |
| `var` Function Expression | ✅ (`undefined`) | ❌ | ❌ (TypeError) |
| `let`/`const` Expression | ✅ (TDZ) | ❌ | ❌ (ReferenceError) |
| Arrow Function | Same as above | ❌ | ❌ |

> **Common Mistake:** Treating arrow functions and function declarations the same. Arrow functions are always expressions — never hoisted with their body.

</details>

---

<details>
<summary><strong>20. Why can a Function Declaration be called before its definition?</strong></summary>

Because during the **Creation Phase**, the engine stores the **entire function object** (not just `undefined`) in memory for function declarations.

```js
// This works because of how the creation phase works
sum(2, 3); // ✅ 5

function sum(a, b) {
  return a + b;
}
```

Internally, before execution starts:
```
Environment Record:
  sum → [Function: sum]  ← full function stored
```

This is useful for **mutually recursive functions** and allows you to organize code with the main logic at the top and helper functions below.

> **Interview Note:** This is one case where hoisting is intentionally useful. For everything else, prefer `const` with arrow functions or function expressions to make dependencies explicit.

</details>

---

<details>
<summary><strong>21. What are common hoisting-related bugs in production code?</strong></summary>

**Bug 1: Using var in loops with async callbacks**
```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // 3, 3, 3 — not 0, 1, 2
}
// Fix: use let
```

**Bug 2: Function expression called before declaration**
```js
init(); // ❌ TypeError — `init` is undefined at this point
var init = function() { console.log('ready'); };
```

**Bug 3: `var` leaking out of try/catch blocks**
```js
try {
  var result = riskyOperation();
} catch (e) {
  // result is still accessible outside — may be undefined
}
console.log(result); // undefined — no error, silent bug
```

**Bug 4: Shadowing with var in nested functions**
```js
var x = 1;
function foo() {
  console.log(x); // undefined — NOT 1
  var x = 2;      // this `var` hoists within foo, shadowing outer x
}
```

> **Interview Note:** Bug #4 is a top interview trick question. The inner `var x` is hoisted to the top of `foo`, so `x` is `undefined` at the `console.log` — even though there's an outer `x = 1`.

</details>

---

<details>
<summary><strong>22. How would you avoid hoisting-related issues in modern JavaScript?</strong></summary>

1. **Always use `const` by default, `let` when reassignment is needed — never `var`**
2. **Declare variables at the top of their scope** — makes the mental model match the execution model
3. **Use function expressions or arrow functions** instead of function declarations for module-level utilities
4. **Enable `'use strict'`** — catches accidental global creation
5. **Use ESLint rules** like `no-use-before-define` and `no-var`

```js
// ✅ Modern, predictable code
const MAX = 100;

const calculateTax = (amount) => amount * 0.1;

const main = () => {
  const result = calculateTax(MAX);
  console.log(result);
};

main();
```

> **Interview Note:** If asked "do you use hoisting intentionally?", the correct senior answer is: "I rely on it only for function declarations when needed for mutual recursion or readability, but I avoid `var` entirely and always declare before use."

</details>

---

## Quick Reference Cheat Sheet

| Concept | `var` | `let` | `const` | Function Declaration |
|---------|-------|-------|---------|---------------------|
| Scope | Function | Block | Block | Function |
| Hoisted | Yes (`undefined`) | Yes (TDZ) | Yes (TDZ) | Yes (full body) |
| Re-declarable | Yes | No | No | Yes (same scope)* |
| Re-assignable | Yes | Yes | No | Yes |
| Global object prop | Yes | No | No | Yes |

*Re-declaring a function declaration with the same name works silently in non-strict (sloppy) mode scripts. In **strict mode** (`'use strict'`) and **ES Modules** (which are always strict), re-declaring a function in the same block-level scope is a `SyntaxError`.

---

*Chapter 1 of JavaScript Interview Prep Series*
