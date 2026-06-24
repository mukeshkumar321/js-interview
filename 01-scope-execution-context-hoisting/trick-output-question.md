# Chapter 1: Trick Output Questions — Scope, Execution Context & Hoisting

> Try to predict the output **before** expanding the answer. Each question reflects a real interview scenario.

---

## Hoisting

---

**Q1. What is the output?**

```js
console.log(a);
var a = 10;
console.log(a);
```

<details>
<summary>Show Output & Explanation</summary>

```
undefined
10
```

**Explanation:**  
`var a` is hoisted to the top of its scope and initialized to `undefined` during the Creation Phase. The assignment `a = 10` only happens at execution time.

Engine sees it as:
```js
var a; // hoisted, initialized to undefined
console.log(a); // undefined
a = 10;
console.log(a); // 10
```

</details>

---

**Q2. What is the output?**

```js
console.log(a);
let a = 10;
```

<details>
<summary>Show Output & Explanation</summary>

```
ReferenceError: Cannot access 'a' before initialization
```

**Explanation:**  
`let` is hoisted but placed in the **Temporal Dead Zone (TDZ)**. Accessing it before the declaration line throws a `ReferenceError` — not `undefined` like `var`.

> Common mistake: saying `let` is "not hoisted." It IS hoisted — the block knows about it — but it's uninitialized until the line is reached.

</details>

---

**Q3. What is the output?**

```js
foo();
bar();

function foo() {
  console.log('foo');
}

var bar = function () {
  console.log('bar');
};
```

<details>
<summary>Show Output & Explanation</summary>

```
foo
TypeError: bar is not a function
```

**Explanation:**  
- `foo` is a **function declaration** — fully hoisted with its body. Calling it before the definition works fine.
- `bar` is a **function expression** assigned to a `var`. Only `var bar` is hoisted (as `undefined`). Calling `undefined()` throws a `TypeError`.

</details>

---

**Q4. What is the output?**

```js
var x = 1;

function foo() {
  console.log(x);
  var x = 2;
  console.log(x);
}

foo();
```

<details>
<summary>Show Output & Explanation</summary>

```
undefined
2
```

**Explanation:**  
Inside `foo`, `var x` is hoisted to the **top of the function** scope — not to the global scope. So at the first `console.log`, the local `x` exists but is `undefined`. The outer `x = 1` is **shadowed** before the assignment even runs.

Engine sees `foo` as:
```js
function foo() {
  var x;         // hoisted — shadows outer x
  console.log(x); // undefined
  x = 2;
  console.log(x); // 2
}
```

> **Classic interview trap.** Many developers expect `1` for the first log.

</details>

---

**Q5. What is the output?**

```js
console.log(typeof foo);
console.log(typeof bar);

var foo = 10;
function bar() {}
```

<details>
<summary>Show Output & Explanation</summary>

```
undefined
function
```

**Explanation:**  
- `var foo` is hoisted as `undefined`, so `typeof foo` is `"undefined"`.
- `bar` is a function declaration — fully hoisted with its body, so `typeof bar` is `"function"`.

> `typeof` on a `var`-declared-but-uninitialized variable returns `"undefined"`, not a ReferenceError — making it safe to use before declaration (unlike `let`/`const`).

</details>

---

**Q6. What is the output?**

```js
var a = 1;
var a = 2;
console.log(a);

let b = 1;
let b = 2; // ?
```

<details>
<summary>Show Output & Explanation</summary>

```
2
SyntaxError: Identifier 'b' has already been declared
```

**Explanation:**  
- `var` allows re-declaration in the same scope — the second `var a` simply overwrites the first.
- `let` does **not** allow re-declaration in the same scope. This is a `SyntaxError` thrown at parse time, before any code runs.

</details>

---

## Scope & Scope Chain

---

**Q7. What is the output?**

```js
var x = 'global';

function outer() {
  var x = 'outer';

  function inner() {
    console.log(x);
  }

  inner();
}

outer();
```

<details>
<summary>Show Output & Explanation</summary>

```
outer
```

**Explanation:**  
`inner` doesn't have its own `x`, so it walks up the scope chain and finds `x = 'outer'` in the `outer` function's scope. The global `x = 'global'` is never reached.

This is **lexical scoping** in action — the scope is determined by where `inner` is **written**, not where it's called from.

</details>

---

**Q8. What is the output?**

```js
var x = 10;

function foo() {
  console.log(x);
}

function bar() {
  var x = 20;
  foo();
}

bar();
```

<details>
<summary>Show Output & Explanation</summary>

```
10
```

**Explanation:**  
JavaScript uses **lexical scope** (static scope), not dynamic scope. `foo` was **defined** in the global scope, so it looks up `x` in the global scope — finding `x = 10`. The fact that `foo` was *called from* `bar` (where `x = 20`) is irrelevant.

> This is the key difference between lexical scope and dynamic scope. JS always uses lexical.

</details>

---

**Q9. What is the output?**

```js
for (var i = 0; i < 3; i++) {
  setTimeout(function () {
    console.log(i);
  }, 0);
}
```

<details>
<summary>Show Output & Explanation</summary>

```
3
3
3
```

**Explanation:**  
`var i` is function-scoped (or global here) — all three callbacks share the **same `i`**. By the time the callbacks run (after the loop), `i` has been incremented to `3`.

**Fix with `let`:**
```js
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0); // 0, 1, 2
}
```
Each `let` iteration creates a new block scope with its own `i`.

**Fix with IIFE (pre-ES6):**
```js
for (var i = 0; i < 3; i++) {
  (function (j) {
    setTimeout(() => console.log(j), 0);
  })(i);
}
```

</details>

---

**Q10. What is the output?**

```js
let x = 1;

{
  let x = 2;
  {
    let x = 3;
    console.log(x);
  }
  console.log(x);
}

console.log(x);
```

<details>
<summary>Show Output & Explanation</summary>

```
3
2
1
```

**Explanation:**  
Each `{}` block creates a new scope. Each `let x` shadows the outer `x` within its block. When each block ends, the inner `x` goes out of scope and the outer `x` is accessible again.

</details>

---

**Q11. What is the output?**

```js
let a = 5;
{
  var a = 10;
}
console.log(a);
```

<details>
<summary>Show Output & Explanation</summary>

```
SyntaxError: Identifier 'a' has already been declared
```

**Explanation:**  
This is **illegal shadowing**. `var a` inside the block tries to declare `a` in the function/global scope (since `var` ignores block boundaries). But `let a` already exists in that same scope. The engine throws a `SyntaxError` at parse time.

> Rule: `var` cannot shadow a `let`/`const` in an enclosing scope.

</details>

---

## Execution Context & Call Stack

---

**Q12. What is the output?**

```js
function first() {
  console.log('first');
  second();
  console.log('first end');
}

function second() {
  console.log('second');
  third();
  console.log('second end');
}

function third() {
  console.log('third');
}

first();
```

<details>
<summary>Show Output & Explanation</summary>

```
first
second
third
second end
first end
```

**Explanation:**  
Call stack at peak: `[Global | first | second | third]`

`third` runs and pops, then `second` resumes and logs `"second end"` before popping, then `first` resumes and logs `"first end"`.

This is the LIFO (Last In, First Out) nature of the call stack.

</details>

---

**Q13. What is the output?**

```js
var name = 'Global';

function greet() {
  var name = 'Local';
  return function () {
    console.log(name);
  };
}

var fn = greet();
fn();
```

<details>
<summary>Show Output & Explanation</summary>

```
Local
```

**Explanation:**  
`greet()` creates a new Execution Context with `name = 'Local'` and returns an inner function. Even after `greet`'s EC is popped off the call stack, the returned function keeps a **closure** over `greet`'s Lexical Environment — so `name` is still `'Local'`.

</details>

---

## Mixed / Hard

---

**Q14. What is the output?**

```js
console.log(foo);
console.log(bar);

var foo = 'hello';

function bar() {
  return 'world';
}
```

<details>
<summary>Show Output & Explanation</summary>

```
undefined
[Function: bar]
```

**Explanation:**  
During the Creation Phase:
- `var foo` → hoisted as `undefined`
- `function bar` → fully hoisted with its body

So `foo` is `undefined` (not yet assigned), and `bar` is already the full function.

</details>

---

**Q15. What is the output?**

```js
function foo() {
  console.log(a);
  console.log(b);
  console.log(c);

  var a = 1;
  let b = 2;
  const c = 3;
}

foo();
```

<details>
<summary>Show Output & Explanation</summary>

```
undefined
ReferenceError: Cannot access 'b' before initialization
```

**Explanation:**  
- `var a` → hoisted to `undefined` inside `foo`
- `let b` → in TDZ, accessing it throws `ReferenceError`
- `const c` → never reached (execution stops at the error)

</details>

---

**Q16. What is the output?**

```js
var x = 'global';

(function () {
  console.log(x);
  var x = 'local';
  console.log(x);
})();
```

<details>
<summary>Show Output & Explanation</summary>

```
undefined
local
```

**Explanation:**  
The IIFE creates a new function scope. Inside it, `var x` is hoisted to the top of the IIFE — shadowing the global `x`. At the first `console.log`, `x` is hoisted but uninitialized (`undefined`). The assignment `x = 'local'` happens next, so the second log is `'local'`.

> Same trap as Q4 — inner `var` always shadows the outer variable from the very start of the function scope.

</details>

---

**Q17. What is the output?**

```js
function test() {
  if (false) {
    var secret = 42;
  }
  console.log(secret);
}

test();
```

<details>
<summary>Show Output & Explanation</summary>

```
undefined
```

**Explanation:**  
`var` is function-scoped, not block-scoped. Even though the `if (false)` block never executes, `var secret` is still **hoisted** to the top of `test()` and initialized as `undefined`. The assignment `secret = 42` never runs, so `secret` remains `undefined`.

> This is one of the most dangerous `var` behaviors — you can reference a variable that's inside an unreachable block.

</details>

---

**Q18. What is the output?**

```js
console.log(a());
console.log(b());
console.log(c());

function a() { return 'A'; }
var b = function () { return 'B'; };
var c = () => 'C';
```

<details>
<summary>Show Output & Explanation</summary>

```
A
TypeError: b is not a function
```

**Explanation:**  
- `function a` → fully hoisted, works fine
- `var b` → hoisted as `undefined`; calling `undefined()` → `TypeError`
- `var c` → never reached (execution stops)

All three are called **before** their definition in the source, but only the function declaration survives intact through hoisting.

</details>

---

**Q19. What is the output?**

```js
let x = 'outer';

function foo() {
  console.log(x);
}

function bar() {
  let x = 'inner';
  foo();
}

bar();
```

<details>
<summary>Show Output & Explanation</summary>

```
outer
```

**Explanation:**  
`foo` was **defined** in the global scope, so its scope chain links to the global scope — not to `bar`'s scope. Even though `foo` is called inside `bar` (where `x = 'inner'` exists), `foo` still sees the global `x = 'outer'`.

> Lexical scope = where the function is **written**, not where it's **called**. This is the single most important rule of JavaScript scope.

</details>

---

**Q20. What is the output?**

```js
var count = 0;

function increment() {
  count++;
}

increment();
increment();
increment();

console.log(count);
```

<details>
<summary>Show Output & Explanation</summary>

```
3
```

**Explanation:**  
`count` is a global variable. Each call to `increment` looks up the scope chain, finds `count` in the global scope, and increments it. After three calls, `count` is `3`.

> Straightforward, but interviewers use this to follow up: "What's the problem with this pattern?" — Answer: it mutates global state, making the code unpredictable at scale. Use closures or modules to encapsulate state.

</details>

---

**Q21. What is the output?**

```js
function outer() {
  var x = 10;

  function inner() {
    var x = 20;
    console.log(x);
  }

  inner();
  console.log(x);
}

outer();
```

<details>
<summary>Show Output & Explanation</summary>

```
20
10
```

**Explanation:**  
`inner` has its own `var x = 20`, which shadows `outer`'s `x`. Inside `inner`, `x` is `20`. After `inner` returns, we're back in `outer`'s scope where `x` is still `10` — the inner variable had no effect on the outer one.

</details>

---

**Q22. What is the output?**

```js
(function () {
  console.log(typeof x);
  console.log(typeof y);

  var x = 10;
})();

console.log(typeof x);
```

<details>
<summary>Show Output & Explanation</summary>

```
undefined
undefined
undefined
```

**Explanation:**  
- Inside the IIFE: `var x` is hoisted to `undefined`. `typeof` on an uninitialized `var` returns `"undefined"`.
- `y` is never declared anywhere. `typeof` on a completely undeclared variable also returns `"undefined"` (no ReferenceError — `typeof` is special).
- Outside the IIFE: `var x` was scoped inside the IIFE, so it doesn't exist globally. `typeof x` returns `"undefined"`.

> `typeof undeclaredVar` is `"undefined"` — the only safe way to check if something exists without risking a ReferenceError.

</details>

---

**Q23. What is the output?**

```js
function foo() {
  return bar();

  var bar = function () {
    return 1;
  };

  function bar() {
    return 2;
  }
}

console.log(foo());
```

<details>
<summary>Show Output & Explanation</summary>

```
2
```

**Explanation:**  
Two things happen in the Creation Phase of `foo`:
1. `var bar` is hoisted as `undefined`
2. `function bar() { return 2; }` is a function declaration — **also hoisted**, and it **overwrites** the `var bar` binding with the full function

So when `return bar()` executes, `bar` is the function declaration (returns `2`). The `var bar = function...` assignment never runs because of the early `return`.

> **Hard question.** Function declarations take precedence over `var` declarations of the same name in the hoisting process.

</details>

---

**Q24. What is the output?**

```js
for (let i = 0; i < 3; i++) {
  setTimeout(function () {
    console.log(i);
  }, i * 100);
}
```

<details>
<summary>Show Output & Explanation</summary>

```
0
1
2
```

**Explanation:**  
`let` in a `for` loop creates a **new binding for each iteration**. Each callback closes over its own `i` — `0`, `1`, and `2` respectively. Even though the callbacks run asynchronously, each one captured a separate copy of `i`.

Compare with `var`: all callbacks would share one `i` and print `3, 3, 3`.

</details>

---

*Chapter 1 — Trick Output Questions | Scope, Execution Context & Hoisting*
