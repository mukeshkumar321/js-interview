# Chapter 3: Trick Output Questions — Functions & `this`

> Try to predict the output **before** expanding the answer. Each question reflects a real interview scenario.

---

## Functions & Callbacks

---

**Q1. What is the output?**

```js
function greet(name = 'Guest') {
  console.log(`Hello, ${name}`);
}

greet();
greet(undefined);
greet(null);
greet('');
```

<details>
<summary>Show Output & Explanation</summary>

```
Hello, Guest
Hello, Guest
Hello, null
Hello, 
```

**Explanation:**  
Default parameters only trigger when the argument is **`undefined`** (including not passed). `null`, `''`, `0`, and `false` are all real values — they do NOT trigger the default.

> **Common Mistake:** Assuming any falsy value falls back to the default. Only `undefined` does.

</details>

---

**Q2. What is the output?**

```js
function foo() {
  console.log(arguments[0], arguments[1]);
}

const bar = (...args) => {
  console.log(args[0], args[1]);
};

foo(1, 2);
bar(1, 2);
```

<details>
<summary>Show Output & Explanation</summary>

```
1 2
1 2
```

**Explanation:**  
Both print the same output, but via different mechanisms. `arguments` is an array-like object available in regular functions. `...args` creates a real array. The values are the same here.

The real difference surfaces when you try array methods: `arguments.map(...)` would throw, `args.map(...)` works fine.

</details>

---

**Q3. What is the output?**

```js
const arr = () => {
  console.log(arguments);
};

arr(1, 2, 3);
```

<details>
<summary>Show Output & Explanation</summary>

```
ReferenceError: arguments is not defined
```

**Explanation:**  
Arrow functions do **not** have their own `arguments` object. Accessing `arguments` inside an arrow function either throws a `ReferenceError` (if there's no enclosing regular function) or refers to the outer function's `arguments`.

> **Fix:** Use rest parameters: `const arr = (...args) => { console.log(args); };`

</details>

---

**Q4. What is the output?**

```js
function outer() {
  const arrow = () => {
    console.log(arguments[0]);
  };
  arrow(99);
}

outer(42);
```

<details>
<summary>Show Output & Explanation</summary>

```
42
```

**Explanation:**  
The arrow function has no own `arguments` — it inherits `arguments` from `outer`. So `arguments[0]` refers to `outer`'s first argument, which is `42`. The `99` passed to `arrow` itself is ignored (no `arguments` to capture it).

> **Hard trap.** The arrow inherits `arguments` from the enclosing regular function, not from its own call.

</details>

---

**Q5. What is the output?**

```js
function add(a, b = a) {
  return a + b;
}

console.log(add(5));
console.log(add(5, 3));
```

<details>
<summary>Show Output & Explanation</summary>

```
10
8
```

**Explanation:**  
Default parameters can reference **earlier parameters**. When `b` is not passed, it defaults to `a`. So `add(5)` → `5 + 5 = 10`. When `b` is passed as `3`, the default is ignored → `5 + 3 = 8`.

</details>

---

## `this` Binding

---

**Q6. What is the output?**

```js
const obj = {
  name: 'Alice',
  greet() {
    console.log(this.name);
  }
};

obj.greet();

const fn = obj.greet;
fn();
```

<details>
<summary>Show Output & Explanation</summary>

```
Alice

```

**Explanation:**  
- `obj.greet()` — implicit binding: `this = obj` → `'Alice'`
- `const fn = obj.greet; fn()` — the method is detached. Default binding applies: `this = window` (or `undefined` in strict mode). In browsers, `window.name` defaults to `''` (empty string), not `'Alice'`. In strict mode, `this` is `undefined` and accessing `this.name` throws a `TypeError`.

> **Most common real-world `this` bug.** Always `bind` methods before passing them as callbacks.

</details>

---

**Q7. What is the output?**

```js
const obj = {
  name: 'Obj',
  greet: () => {
    console.log(this.name);
  }
};

obj.greet();
```

<details>
<summary>Show Output & Explanation</summary>

```
undefined
```

**Explanation:**  
Arrow functions have no own `this` — they inherit it from their lexical scope. The object literal `{}` does not create a new `this` scope. So `this` refers to the enclosing scope (global/module), where `this.name` is `undefined`.

> **Rule:** Never use arrow functions as object methods if you need to access the object via `this`.

</details>

---

**Q8. What is the output?**

```js
function Person(name) {
  this.name = name;
}

const alice = new Person('Alice');
const bob   = Person('Bob');

console.log(alice.name);
console.log(bob);
```

<details>
<summary>Show Output & Explanation</summary>

```
Alice
undefined
```

**Explanation:**  
- `new Person('Alice')` — `new` binding: a new object is created, `this.name = 'Alice'` is set on it. `alice` = that object.
- `Person('Bob')` — called **without** `new`. No new object is created. `this` = `window` (non-strict), so `window.name = 'Bob'`. The function returns `undefined` (no explicit return), so `bob = undefined`.

> **Interview Note:** Forgetting `new` with constructor functions is a classic source of global variable pollution.

</details>

---

**Q9. What is the output?**

```js
const obj = {
  value: 10,
  getValue() {
    return this.value;
  }
};

const { getValue } = obj;
console.log(getValue());
```

<details>
<summary>Show Output & Explanation</summary>

```
undefined
```

**Explanation:**  
Destructuring a method detaches it from the object — same as `const getValue = obj.getValue`. Calling it standalone uses default binding: `this = window` (or `undefined` strict). `window.value` is `undefined`.

**Fix:**
```js
const getValue = obj.getValue.bind(obj);
console.log(getValue()); // 10
```

</details>

---

**Q10. What is the output?**

```js
'use strict';

function show() {
  console.log(this);
}

show();
show.call(null);
show.call(undefined);
```

<details>
<summary>Show Output & Explanation</summary>

```
undefined
null
undefined
```

**Explanation:**  
In strict mode:
- `show()` — default binding → `this = undefined` (not `window`)
- `show.call(null)` — explicit binding with `null` → `this = null` in strict mode
- `show.call(undefined)` — explicit binding with `undefined` → `this = undefined`

In **non-strict mode**, `null` and `undefined` passed to `call` would both default to `window`.

</details>

---

**Q11. What is the output?**

```js
const obj = {
  name: 'Outer',
  greet() {
    function inner() {
      console.log(this.name);
    }
    inner();
  }
};

obj.greet();
```

<details>
<summary>Show Output & Explanation</summary>

```
undefined
```

**Explanation:**  
`inner` is a regular function called without any object — default binding applies. `this` inside `inner` is `window` (or `undefined` in strict mode). It does NOT inherit `this` from `greet`.

**Fix with arrow function:**
```js
greet() {
  const inner = () => console.log(this.name); // inherits this from greet
  inner(); // 'Outer'
}
```

</details>

---

**Q12. What is the output?**

```js
const obj = {
  name: 'Obj',
  greet() {
    const inner = () => {
      console.log(this.name);
    };
    inner();
  }
};

obj.greet();
```

<details>
<summary>Show Output & Explanation</summary>

```
Obj
```

**Explanation:**  
The arrow function `inner` has no own `this` — it inherits `this` from the enclosing `greet` method. When `obj.greet()` is called, `this` inside `greet` is `obj`. So `inner` also sees `this = obj` → `'Obj'`.

> This is exactly **why arrow functions exist** — to avoid the `inner` function losing `this`.

</details>

---

## call, apply & bind

---

**Q13. What is the output?**

```js
function greet(greeting, punctuation) {
  console.log(`${greeting}, ${this.name}${punctuation}`);
}

const person = { name: 'Alice' };

greet.call(person, 'Hello', '!');
greet.apply(person, ['Hi', '.']);
```

<details>
<summary>Show Output & Explanation</summary>

```
Hello, Alice!
Hi, Alice.
```

**Explanation:**  
Both `call` and `apply` invoke the function immediately with the specified `this`. The only difference: `call` takes args individually, `apply` takes them as an array.

</details>

---

**Q14. What is the output?**

```js
function greet(greeting) {
  console.log(`${greeting}, ${this.name}`);
}

const person = { name: 'Bob' };

const boundGreet = greet.bind(person, 'Hey');
boundGreet();
boundGreet('Ignored');
```

<details>
<summary>Show Output & Explanation</summary>

```
Hey, Bob
Hey, Bob
```

**Explanation:**  
`bind` returns a **new function** with `this` and the first argument permanently fixed. Calling `boundGreet()` uses `greeting = 'Hey'` from the bind. Passing `'Ignored'` to `boundGreet` has no effect — the `greeting` slot is already filled.

</details>

---

**Q15. What is the output?**

```js
const obj1 = { name: 'Obj1' };
const obj2 = { name: 'Obj2' };

function show() {
  console.log(this.name);
}

const bound = show.bind(obj1);
bound();
bound.call(obj2);
```

<details>
<summary>Show Output & Explanation</summary>

```
Obj1
Obj1
```

**Explanation:**  
`bind` creates a **permanently bound** function. Once `this` is bound to `obj1`, it cannot be overridden — not even by `call` or `apply`. Both calls print `'Obj1'`.

> **Key rule:** `bind` wins over `call`/`apply`. The only thing that can override a bound function's `this` is `new`.

</details>

---

**Q16. What is the output?**

```js
function multiply(a, b) {
  return a * b;
}

const double = multiply.bind(null, 2);
const triple = multiply.bind(null, 3);

console.log(double(5));
console.log(triple(5));
console.log(double(triple(2)));
```

<details>
<summary>Show Output & Explanation</summary>

```
10
15
12
```

**Explanation:**  
`bind` with `null` as `this` context is used purely for **partial application** (pre-filling arguments).

- `double(5)` → `multiply(2, 5)` = `10`
- `triple(5)` → `multiply(3, 5)` = `15`
- `triple(2)` → `6`, then `double(6)` → `multiply(2, 6)` = `12`

</details>

---

**Q17. What is the output?**

```js
const obj = {
  x: 10,
  getX: function () {
    return this.x;
  }
};

const unboundGetX = obj.getX;
const boundGetX   = obj.getX.bind(obj);

console.log(unboundGetX());
console.log(boundGetX());
```

<details>
<summary>Show Output & Explanation</summary>

```
undefined
10
```

**Explanation:**  
- `unboundGetX()` — default binding, `this = window`, `window.x = undefined`
- `boundGetX()` — permanently bound to `obj`, `this.x = 10`

</details>

---

## Mixed / Hard

---

**Q18. What is the output?**

```js
const obj = {
  name: 'Alice',
  friends: ['Bob', 'Carol'],
  printFriends() {
    this.friends.forEach(function (friend) {
      console.log(`${this.name} knows ${friend}`);
    });
  }
};

obj.printFriends();
```

<details>
<summary>Show Output & Explanation</summary>

```
 knows Bob
 knows Carol
```

**Explanation:**  
The callback passed to `forEach` is a regular function — it has its own `this`. Since it's called without an object (default binding), `this = window` in non-strict mode. In browsers, `window.name` defaults to `''` (empty string), so the output is `' knows Bob'` and `' knows Carol'`. In strict mode, `this` is `undefined` and accessing `this.name` would throw a `TypeError`.

**Fix:**
```js
// Option 1: Arrow function (inherits this from printFriends)
this.friends.forEach((friend) => {
  console.log(`${this.name} knows ${friend}`);
});

// Option 2: forEach second argument
this.friends.forEach(function (friend) {
  console.log(`${this.name} knows ${friend}`);
}, this);
```

</details>

---

**Q19. What is the output?**

```js
const obj = {
  name: 'Alice',
  friends: ['Bob', 'Carol'],
  printFriends() {
    this.friends.forEach((friend) => {
      console.log(`${this.name} knows ${friend}`);
    });
  }
};

obj.printFriends();
```

<details>
<summary>Show Output & Explanation</summary>

```
Alice knows Bob
Alice knows Carol
```

**Explanation:**  
The arrow function inherits `this` from `printFriends`. When `obj.printFriends()` is called, `this = obj` inside `printFriends`. The arrow callback captures that same `this`, so `this.name = 'Alice'`.

</details>

---

**Q20. What is the output?**

```js
function Timer() {
  this.count = 0;

  setInterval(function () {
    this.count++;
    console.log(this.count);
  }, 100);
}

const t = new Timer();
```

<details>
<summary>Show Output & Explanation</summary>

```
NaN
NaN
NaN
... (repeating)
```

**Explanation:**  
The `setInterval` callback is a regular function — `this` is `window` (or `undefined` strict), NOT the `Timer` instance. `window.count` is `undefined`. `undefined++` = `NaN`. `NaN++` = `NaN`.

**Fix with arrow function:**
```js
setInterval(() => {
  this.count++; // `this` = Timer instance
  console.log(this.count); // 1, 2, 3...
}, 100);
```

</details>

---

**Q21. What is the output?**

```js
function foo() {
  console.log(this.x);
}

const obj1 = { x: 1, foo };
const obj2 = { x: 2, foo };

obj1.foo();
obj2.foo();

const bar = obj1.foo.bind(obj2);
bar();
```

<details>
<summary>Show Output & Explanation</summary>

```
1
2
2
```

**Explanation:**  
- `obj1.foo()` — implicit binding → `this = obj1` → `x = 1`
- `obj2.foo()` — implicit binding → `this = obj2` → `x = 2`
- `bar()` — bound to `obj2` via `bind` → `this = obj2` → `x = 2`

</details>

---

**Q22. What is the output?**

```js
const obj = {
  val: 1,
  double() {
    return function () {
      return this.val * 2;
    };
  }
};

console.log(obj.double()());
```

<details>
<summary>Show Output & Explanation</summary>

```
NaN
```

**Explanation:**  
`obj.double()` returns an inner regular function. It's called immediately with `()` — no object before the dot, so default binding: `this = window`. `window.val` is `undefined`. `undefined * 2 = NaN`.

**Fix:**
```js
double() {
  return () => this.val * 2; // arrow inherits `this = obj`
}
obj.double()(); // 2
```

</details>

---

**Q23. What is the output?**

```js
function greet() {
  console.log(this.name);
}

const obj = { name: 'Alice' };
const arr = [greet, 'Bob'];

arr[0]();
```

<details>
<summary>Show Output & Explanation</summary>

```
undefined
```

**Explanation:**  
When a function is called as `arr[0]()`, the implicit binding rule applies — `this = arr` (the array). Arrays don't have a `name` property, so `this.name = undefined`.

> **Unusual trap.** Arrays are objects too — accessing a function via bracket notation on an array sets `this` to that array.

</details>

---

**Q24. What is the output?**

```js
function Person(name) {
  this.name = name;
  this.sayName = function () {
    console.log(this.name);
  };
}

const alice = new Person('Alice');
const bob   = new Person('Bob');

alice.sayName();
alice.sayName.call(bob);

const fn = alice.sayName.bind(bob);
fn.call(alice);
```

<details>
<summary>Show Output & Explanation</summary>

```
Alice
Bob
Bob
```

**Explanation:**  
- `alice.sayName()` — implicit binding → `this = alice` → `'Alice'`
- `alice.sayName.call(bob)` — explicit binding overrides implicit → `this = bob` → `'Bob'`
- `fn.call(alice)` — `fn` is bound to `bob` via `bind`. `bind` cannot be overridden by `call` → still `this = bob` → `'Bob'`

> **Key rule:** `bind` creates a permanently bound function. `call` and `apply` cannot override it.

</details>

---

*Chapter 3 — Trick Output Questions | Functions & `this`*
