# Chapter 4: Objects, Prototypes & OOP

> **Target:** Frontend Developer (4+ years) | Interview-focused, practical explanations

---

## Table of Contents

- [Objects Fundamentals](#objects-fundamentals)
- [Prototypes & Prototype Chain](#prototypes--prototype-chain)
- [OOP & Classes](#oop--classes)

**Objects Fundamentals includes:** object creation, dot vs bracket notation, references, shallow/deep copy, cloning, own vs inherited properties, freeze, seal, property descriptors, getters & setters

---

## Objects Fundamentals

<details>
<summary><strong>1. What are the different ways to create an Object in JavaScript?</strong></summary>

```js
// 1. Object literal (most common)
const obj = { name: 'Alice', age: 30 };

// 2. Object.create() — sets explicit prototype
const proto = { greet() { return `Hi, ${this.name}`; } };
const obj2 = Object.create(proto);
obj2.name = 'Bob';
obj2.greet(); // 'Hi, Bob'

// 3. Constructor function
function Person(name) { this.name = name; }
const p = new Person('Carol');

// 4. ES6 Class
class Animal {
  constructor(name) { this.name = name; }
}
const a = new Animal('Dog');

// 5. Object.assign() — creates from existing
const copy = Object.assign({}, obj);

// 6. Factory function — returns a plain object, no `new`
function createUser(name) {
  return { name, greet() { return `Hi, ${this.name}`; } };
}
```

> **Interview Note:** `Object.create(null)` creates an object with **no prototype at all** — useful for pure hash maps with no risk of prototype pollution (`hasOwnProperty` attacks).

</details>

---

<details>
<summary><strong>2. What is the difference between dot notation and bracket notation?</strong></summary>

```js
const obj = { name: 'Alice', 'full-name': 'Alice Smith' };

// Dot notation — clean, used when key is a valid identifier
obj.name;       // 'Alice'

// Bracket notation — required when key is dynamic, has spaces, or starts with a number
obj['full-name'];          // 'Alice Smith' — dot notation would fail
obj['name'];               // 'Alice'

const key = 'name';
obj[key];                  // 'Alice' — dynamic key lookup

obj[Symbol('id')];         // symbols must use bracket notation
```

**Key rule:** Use dot notation by default. Use bracket notation when the key is dynamic, not a valid identifier, or a Symbol.

> **Common Mistake:** Trying to use dot notation with dynamic keys: `obj.key` always reads the literal property named `"key"`, not the variable `key`'s value.

</details>

---

<details>
<summary><strong>3. How are Objects stored in memory and compared in JavaScript?</strong></summary>

Primitives (`number`, `string`, `boolean`, etc.) are stored **by value** on the stack.  
Objects are stored **by reference** — the variable holds a memory address (pointer), not the object itself.

```js
// Primitives — by value
let a = 5;
let b = a;
b = 10;
console.log(a); // 5 — unchanged

// Objects — by reference
const obj1 = { x: 1 };
const obj2 = obj1;   // both point to the same object
obj2.x = 99;
console.log(obj1.x); // 99 — mutated through obj2
```

**Comparison:**
```js
const a = { x: 1 };
const b = { x: 1 };
const c = a;

console.log(a === b); // false — different references
console.log(a === c); // true  — same reference
```

> **Interview Note:** This is why array/object prop changes in React don't trigger re-renders when you mutate in place — the reference doesn't change. You must create a new object/array.

</details>

---

<details>
<summary><strong>4. What is the difference between Shallow Copy and Deep Copy?</strong></summary>

**Shallow Copy** — copies the top-level properties. Nested objects are still shared by reference.  
**Deep Copy** — recursively copies all levels. No shared references.

```js
const original = {
  name: 'Alice',
  address: { city: 'NY' }
};

// Shallow copy — nested object still shared
const shallow = { ...original };
shallow.name = 'Bob';         // ✅ doesn't affect original
shallow.address.city = 'LA';  // ❌ mutates original.address.city too!

console.log(original.name);          // 'Alice'
console.log(original.address.city);  // 'LA' — side effect!

// Deep copy — fully independent
const deep = JSON.parse(JSON.stringify(original));
deep.address.city = 'Chicago';
console.log(original.address.city);  // 'LA' — untouched
```

**Limitations of `JSON.parse/stringify`:** loses `undefined`, functions, `Date` objects, `Symbol`, circular references.

**Modern deep copy:**
```js
const deep = structuredClone(original); // ✅ handles most types, no functions
```

</details>

---

<details>
<summary><strong>5. What are the different ways to clone an Object?</strong></summary>

| Method | Depth | Handles Functions | Handles Dates | Circular |
|--------|-------|-------------------|---------------|----------|
| `Object.assign({}, obj)` | Shallow | Yes | By ref | No |
| `{ ...obj }` | Shallow | Yes | By ref | No |
| `JSON.parse(JSON.stringify)` | Deep | ❌ Lost | ❌ String | ❌ Error |
| `structuredClone()` | Deep | ❌ Omitted (silently) | ✅ | ✅ |
| Custom recursive | Deep | ✅ | ✅ | With care |
| Lodash `_.cloneDeep` | Deep | ✅ | ✅ | ✅ |

```js
const obj = { a: 1, nested: { b: 2 } };

// Shallow
const s1 = Object.assign({}, obj);
const s2 = { ...obj };

// Deep
const d1 = JSON.parse(JSON.stringify(obj));  // no functions/dates
const d2 = structuredClone(obj);             // modern, recommended
```

> **Interview Note:** Know the tradeoffs. For production: `structuredClone` for data objects; Lodash `cloneDeep` when functions/custom classes are involved.

</details>

---

<details>
<summary><strong>6. What is the difference between own properties and inherited properties?</strong></summary>

**Own properties** are defined directly on the object.  
**Inherited properties** come from the object's prototype chain.

```js
function Animal(name) {
  this.name = name; // own property
}
Animal.prototype.speak = function () { // inherited property
  return `${this.name} makes a sound`;
};

const dog = new Animal('Rex');

dog.hasOwnProperty('name');  // true  — own
dog.hasOwnProperty('speak'); // false — inherited

// for...in iterates BOTH own and inherited
for (const key in dog) console.log(key); // name, speak

// Object.keys only own enumerable properties
Object.keys(dog); // ['name']
```

> **Interview Note:** Always use `hasOwnProperty` (or `Object.hasOwn(obj, key)` — the modern version) when you need to guard against prototype chain properties showing up in `for...in` loops.

</details>

---

<details>
<summary><strong>7. What is <code>Object.freeze()</code> and when would you use it?</strong></summary>

`Object.freeze()` makes an object **immutable** — no adding, removing, or modifying properties. It's **shallow**.

```js
const config = Object.freeze({
  API_URL: 'https://api.example.com',
  TIMEOUT: 5000
});

config.API_URL = 'changed'; // silently fails (throws in strict mode)
config.newProp = 'x';       // silently fails
delete config.TIMEOUT;      // silently fails

console.log(config.API_URL); // 'https://api.example.com' — unchanged
```

**Shallow — nested objects are NOT frozen:**
```js
const obj = Object.freeze({ nested: { x: 1 } });
obj.nested.x = 99; // ✅ this WORKS — nested is not frozen
```

**Use cases:** constants, configuration objects, action type definitions in Redux.

> **Interview Note:** For truly immutable nested structures, you need a deep freeze utility or an immutability library like Immer.

</details>

---

<details>
<summary><strong>8. What is <code>Object.seal()</code> and how is it different from <code>Object.freeze()</code>?</strong></summary>

| | `Object.freeze()` | `Object.seal()` |
|--|-------------------|-----------------|
| Add properties | ❌ | ❌ |
| Delete properties | ❌ | ❌ |
| Modify existing values | ❌ | ✅ |
| Prototype changes | ❌ | ❌ |

```js
const obj = Object.seal({ name: 'Alice', age: 30 });

obj.name = 'Bob'; // ✅ modification allowed
obj.city = 'NY';  // ❌ adding not allowed
delete obj.age;   // ❌ deletion not allowed

console.log(obj); // { name: 'Bob', age: 30 }
```

**When to use `seal`:** When you want to prevent shape changes (adding/removing keys) but still allow value updates — e.g., a settings object that should maintain a fixed structure.

</details>

---

<details>
<summary><strong>9. What are Property Descriptors?</strong></summary>

Every object property has a hidden **descriptor** — metadata that controls how the property behaves. You can read or set it with `Object.getOwnPropertyDescriptor` / `Object.defineProperty`.

**Descriptor flags:**

| Flag | Default (normal assignment) | Meaning |
|------|----------------------------|---------|
| `value` | the assigned value | The property's value |
| `writable` | `true` | Can the value be changed? |
| `enumerable` | `true` | Does it show in `for...in` and `Object.keys`? |
| `configurable` | `true` | Can the descriptor itself be changed or the property deleted? |

```js
const obj = { a: 1 };

Object.defineProperty(obj, 'b', {
  value: 2,
  writable: false,    // can't reassign
  enumerable: false,  // hidden from Object.keys / for...in
  configurable: false // can't delete or redefine
});

obj.b = 99;               // silently ignored (throws in strict mode)
console.log(obj.b);       // 2

Object.keys(obj);         // ['a'] — 'b' is not enumerable
'b' in obj;               // true — 'in' checks existence, not enumerability
delete obj.b;             // silently ignored
```

**Inspecting a descriptor:**
```js
Object.getOwnPropertyDescriptor(obj, 'a');
// { value: 1, writable: true, enumerable: true, configurable: true }
```

> **Interview Note:** `Object.freeze()` sets `writable: false` and `configurable: false` on every property. `Object.seal()` sets `configurable: false` but leaves `writable: true`.

</details>

---

<details>
<summary><strong>10. What are Getters and Setters?</strong></summary>

Getters and setters let you define **computed properties** — properties that run a function when accessed or assigned, without the caller knowing it's not a plain value.

**Object literal syntax:**
```js
const person = {
  firstName: 'Alice',
  lastName: 'Smith',

  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  },

  set fullName(value) {
    [this.firstName, this.lastName] = value.split(' ');
  }
};

console.log(person.fullName);       // 'Alice Smith' — calls getter
person.fullName = 'Bob Jones';      // calls setter
console.log(person.firstName);      // 'Bob'
```

**Class syntax:**
```js
class Temperature {
  #celsius;

  constructor(c) { this.#celsius = c; }

  get fahrenheit() { return this.#celsius * 9/5 + 32; }
  set fahrenheit(f) { this.#celsius = (f - 32) * 5/9; }
}

const t = new Temperature(100);
console.log(t.fahrenheit); // 212
t.fahrenheit = 32;
console.log(t.fahrenheit); // 32 — #celsius is now 0
```

**Key points:**
- Accessed like a plain property — no `()` needed
- Useful for computed values, validation on set, and lazy initialization
- A property can have a getter without a setter (read-only) or a setter without a getter (write-only)

> **Common Mistake:** Calling a getter like a method — `person.fullName()` throws `TypeError: person.fullName is not a function`. Access it as a property: `person.fullName`.

</details>

---

## Prototypes & Prototype Chain

<details>
<summary><strong>9. What is a Prototype in JavaScript?</strong></summary>

Every JavaScript object has an internal link (`[[Prototype]]`) to another object called its **prototype**. When you access a property on an object, if it's not found there, JavaScript looks up the prototype chain until it reaches `null`.

```js
const animal = {
  breathe() { return 'breathing'; }
};

const dog = Object.create(animal); // dog's prototype = animal
dog.bark = function () { return 'woof'; };

dog.bark();    // 'woof'    — own method
dog.breathe(); // 'breathing' — found on prototype
dog.toString(); // found on Object.prototype
```

**Prototype chain:**
```
dog → animal → Object.prototype → null
```

Prototypes enable **property and method sharing** without copying — all objects sharing a prototype share the same method in memory.

</details>

---

<details>
<summary><strong>10. What is the Prototype Chain and how does property lookup work?</strong></summary>

When you access `obj.prop`, JavaScript follows this sequence:

1. Check `obj` itself (own properties)
2. Check `obj.__proto__` (its prototype)
3. Check `obj.__proto__.__proto__`
4. Continue until `null` is reached
5. If not found → return `undefined`

```js
function Animal(name) { this.name = name; }
Animal.prototype.speak = function () { return `${this.name} speaks`; };

function Dog(name) { Animal.call(this, name); }
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.bark = function () { return 'woof'; };

const rex = new Dog('Rex');

rex.bark();   // own → Dog.prototype → found: 'woof'
rex.speak();  // own → Dog.prototype → Animal.prototype → found: 'Rex speaks'
rex.toString(); // ... → Object.prototype → found
rex.missing;  // → null → undefined
```

> **Interview Note:** The prototype chain is a **live lookup** — if you add a method to a prototype after creating objects, those objects immediately gain access to it.

</details>

---

<details>
<summary><strong>11. What is the difference between <code>__proto__</code>, <code>prototype</code>, and <code>[[Prototype]]</code>?</strong></summary>

| | What it is | Who has it |
|--|-----------|------------|
| `[[Prototype]]` | Internal spec reference to parent | Every object |
| `__proto__` | Getter/setter to access `[[Prototype]]` | Every object (deprecated) |
| `.prototype` | Object assigned as `[[Prototype]]` of instances | Functions only |

```js
function Dog(name) { this.name = name; }

const rex = new Dog('Rex');

// [[Prototype]] is internal — accessed via:
Object.getPrototypeOf(rex) === Dog.prototype; // true
rex.__proto__ === Dog.prototype;              // true (deprecated)

// .prototype is only on the constructor function
Dog.prototype.bark = function () { return 'woof'; };
rex.bark(); // 'woof' — via [[Prototype]] chain

// Arrow functions and object literals have no .prototype
const fn = () => {};
fn.prototype; // undefined
```

> **Interview Note:** Avoid `__proto__` in production — use `Object.getPrototypeOf()` instead. `__proto__` is deprecated but still widely asked in interviews.

</details>

---

<details>
<summary><strong>12. How does the <code>new</code> keyword work internally?</strong></summary>

When you call `new Foo()`, JavaScript does four things:

1. Creates a new empty object `{}`
2. Sets its `[[Prototype]]` to `Foo.prototype`
3. Calls `Foo` with `this` = the new object
4. Returns the new object (unless the constructor explicitly returns a different object)

```js
function Person(name) {
  this.name = name;
}

// What `new Person('Alice')` does internally:
function simulateNew(Constructor, ...args) {
  const obj = Object.create(Constructor.prototype); // steps 1 & 2
  const result = Constructor.apply(obj, args);       // step 3
  return result instanceof Object ? result : obj;    // step 4 — Note: instanceof Object fails for Object.create(null) objects
}

const alice = simulateNew(Person, 'Alice');
alice.name; // 'Alice'
```

> **Interview Note:** Implementing `new` from scratch is a classic interview question. The key insight: `Object.create(Constructor.prototype)` wires up the prototype chain before the constructor runs.

</details>

---

<details>
<summary><strong>13. What is Prototypal Inheritance?</strong></summary>

Prototypal inheritance means objects inherit directly from other objects — there's no class blueprint involved (classes in JS are syntactic sugar over this).

```js
const vehicle = {
  type: 'vehicle',
  describe() { return `I am a ${this.type} named ${this.name}`; }
};

const car = Object.create(vehicle);
car.type = 'car';

const tesla = Object.create(car);
tesla.name = 'Tesla';

tesla.describe(); // 'I am a car named Tesla'
// Prototype chain: tesla → car → vehicle → Object.prototype → null
```

**With constructor functions:**
```js
function Animal(name) { this.name = name; }
Animal.prototype.eat = function () { return `${this.name} eats`; };

function Dog(name) {
  Animal.call(this, name); // inherit own properties
}
Dog.prototype = Object.create(Animal.prototype); // inherit methods
Dog.prototype.constructor = Dog;                 // fix constructor ref
Dog.prototype.bark = function () { return 'woof'; };

const rex = new Dog('Rex');
rex.eat();  // 'Rex eats'
rex.bark(); // 'woof'
```

</details>

---

<details>
<summary><strong>14. Why does JavaScript use Prototypes instead of Classical Inheritance?</strong></summary>

JavaScript was designed around **object delegation** rather than class-based blueprinting. Objects inherit from other objects directly — more flexible and memory-efficient.

**Classical (class-based) inheritance:**
- Classes are blueprints; instances are copies of those blueprints
- Rigid hierarchies — tight coupling between parent and child

**Prototypal inheritance:**
- Objects inherit from other live objects
- Methods are shared, not copied — more memory efficient
- Flexible — you can change a prototype at runtime and all instances see the change immediately

```js
function Animal() {}
Animal.prototype.speak = function () { return 'generic sound'; };

const dog = new Animal();
dog.speak(); // 'generic sound'

// Adding to prototype AFTER creation — all instances benefit immediately
Animal.prototype.breathe = function () { return 'breathing'; };
dog.breathe(); // 'breathing' — no re-instantiation needed
```

> **Interview Note:** ES6 Classes didn't change the underlying prototype model — `class` is purely syntactic sugar. Under the hood it's still prototype-based.

</details>

---

<details>
<summary><strong>15. What are the advantages of sharing methods through the Prototype?</strong></summary>

When methods are on the prototype (not the instance), **all instances share one copy in memory**.

```js
// ❌ BAD — each instance gets its own copy of greet
function Person(name) {
  this.name = name;
  this.greet = function () { return `Hi, I'm ${this.name}`; }; // copied per instance
}

// ✅ GOOD — all instances share one greet on the prototype
function Person(name) {
  this.name = name;
}
Person.prototype.greet = function () { return `Hi, I'm ${this.name}`; };

const p1 = new Person('Alice');
const p2 = new Person('Bob');

p1.greet === p2.greet; // true — same function reference in memory
```

**Memory impact:** With 10,000 instances, method-on-instance means 10,000 function copies. Method-on-prototype means 1 shared copy.

> **Interview Note:** This is why ES6 classes put methods on the prototype automatically — `class` methods are syntactic sugar for `Constructor.prototype.method = function() {}`.

</details>

---

## OOP & Classes

<details>
<summary><strong>16. What are ES6 Classes and are they true classes or syntactic sugar?</strong></summary>

ES6 Classes are **syntactic sugar over prototypal inheritance** — they don't introduce a new object model. Under the hood, everything still works with prototypes.

```js
class Animal {
  constructor(name) {
    this.name = name; // own property
  }

  speak() {           // added to Animal.prototype
    return `${this.name} makes a sound`;
  }

  static create(name) { // on Animal itself, not prototype
    return new Animal(name);
  }
}

// Equivalent prototype version:
function Animal(name) { this.name = name; }
Animal.prototype.speak = function () { return `${this.name} makes a sound`; };
Animal.create = function (name) { return new Animal(name); };
```

**Proof it's sugar:**
```js
typeof Animal; // 'function' — classes are functions!
Object.getPrototypeOf(Animal.prototype); // Object.prototype
```

**Real differences from constructor functions:**
- Class declarations are in the TDZ — hoisted but cannot be used before their declaration line (unlike function declarations which are fully hoisted)
- Class bodies always run in strict mode
- Methods are non-enumerable by default
- Must be called with `new` (throws otherwise)

</details>

---

<details>
<summary><strong>17. What is the difference between Constructor Functions and ES6 Classes?</strong></summary>

| | Constructor Function | ES6 Class |
|--|---------------------|-----------|
| Hoisted | Yes (as function) | TDZ (hoisted but inaccessible before declaration) |
| Strict mode | Only if opted in | Always |
| Methods enumerable | Yes | No |
| Requires `new` | No (silently breaks) | Yes (throws) |
| `extends`/`super` | Manual, verbose | Built-in |
| Private fields | Convention `_` | `#field` syntax |

```js
// Constructor function
function Car(model) {
  this.model = model;
}
Car.prototype.describe = function () { return this.model; };

// ES6 Class
class Car {
  #mileage = 0; // true private field

  constructor(model) {
    this.model = model;
  }

  describe() { return this.model; }
  addMiles(n) { this.#mileage += n; }
  getMiles()  { return this.#mileage; }
}
```

> **Interview Note:** Classes are preferred in modern code for readability, private fields (`#`), and the safety of always requiring `new`. But knowing they're prototype-based demonstrates deeper understanding.

</details>

---

<details>
<summary><strong>18. What is the purpose of the <code>extends</code> and <code>super</code> keywords?</strong></summary>

`extends` sets up the prototype chain between a child and parent class.  
`super` refers to the parent class — used to call its constructor or methods.

```js
class Animal {
  constructor(name) {
    this.name = name;
  }

  speak() {
    return `${this.name} makes a sound`;
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name);       // MUST call super() before using `this`
    this.breed = breed;
  }

  speak() {
    const base = super.speak(); // call parent method
    return `${base} — specifically: woof`;
  }
}

const rex = new Dog('Rex', 'Labrador');
rex.speak(); // 'Rex makes a sound — specifically: woof'
rex instanceof Dog;    // true
rex instanceof Animal; // true
```

**Rules:**
- If you define a `constructor` in a subclass, `super()` is mandatory before any `this` access
- `super.method()` calls the parent's prototype method

> **Common Mistake:** Forgetting `super()` in a subclass constructor — throws `ReferenceError: Must call super constructor before accessing 'this'`.

</details>

---

<details>
<summary><strong>19. What are the four pillars of OOP and how are they implemented in JavaScript?</strong></summary>

**1. Encapsulation** — bundle data and methods, hide internal details
```js
class BankAccount {
  #balance = 0; // private

  deposit(amount) { this.#balance += amount; }
  getBalance()    { return this.#balance; }
}
```

**2. Abstraction** — expose only what's necessary, hide complexity
```js
class Car {
  start() { this.#initEngine(); this.#warmUp(); return 'ready'; }
  #initEngine() { /* hidden */ }
  #warmUp()     { /* hidden */ }
}
```

**3. Inheritance** — child class reuses parent's behavior
```js
class Animal { speak() { return 'sound'; } }
class Dog extends Animal { bark() { return 'woof'; } }
// Dog inherits speak() from Animal
```

**4. Polymorphism** — same interface, different behavior
```js
class Shape   { area() { return 0; } }
class Circle  extends Shape { area() { return Math.PI * this.r ** 2; } }
class Square  extends Shape { area() { return this.side ** 2; } }

const shapes = [new Circle(5), new Square(4)];
shapes.forEach(s => console.log(s.area())); // each calls its own area()
```

> **Interview Note:** Interviewers often ask you to implement all four with a concrete example. Use a `BankAccount` or `Shape` hierarchy — they cleanly demonstrate all four pillars.

</details>

---

<details>
<summary><strong>20. What is the difference between Composition and Inheritance?</strong></summary>

**Inheritance** — "is-a" relationship. A `Dog` IS an `Animal`.  
**Composition** — "has-a" relationship. A `Robot` HAS a `battery`, HAS a `motor`.

```js
// Inheritance — tight coupling, fragile with deep hierarchies
class FlyingFish extends Fish {
  // Must inherit ALL of Fish, even parts not needed
}

// Composition — mix in only what you need
const canSwim = (state) => ({
  swim: () => `${state.name} is swimming`
});

const canFly = (state) => ({
  fly: () => `${state.name} is flying`
});

function createFlyingFish(name) {
  const state = { name };
  return Object.assign({}, canSwim(state), canFly(state));
}

const nemo = createFlyingFish('Nemo');
nemo.swim(); // 'Nemo is swimming'
nemo.fly();  // 'Nemo is flying'
```

**The problem with deep inheritance:**
```
Vehicle → MotorVehicle → Car → ElectricCar → ElectricSportsCar
// Change at top ripples down — fragile
```

</details>

---

<details>
<summary><strong>21. When would you prefer Composition over Inheritance?</strong></summary>

**Prefer Composition when:**
- The hierarchy would be more than 2 levels deep
- A subclass only needs part of the parent's behavior
- You need to share behavior across unrelated classes
- You want to avoid tight coupling

**Prefer Inheritance when:**
- There's a clear, stable "is-a" relationship
- The hierarchy is shallow (1–2 levels)
- You want to leverage `instanceof` checks
- Framework demands it (e.g., React class components, extending `Error`)

```js
// ✅ Composition — reusable behaviors as mixins
const Serializable = (Base) => class extends Base {
  serialize()   { return JSON.stringify(this); }
  deserialize(s) { return Object.assign(this, JSON.parse(s)); }
};

const Timestamped = (Base) => class extends Base {
  constructor(...args) {
    super(...args);
    this.createdAt = new Date();
  }
};

class User extends Timestamped(Serializable(class {})) {
  constructor(name) { super(); this.name = name; }
}
```

> **Interview Note:** The phrase *"favor composition over inheritance"* comes from the Gang of Four Design Patterns book. In JavaScript specifically, composition is more natural because the language is prototype-based, not class-based.

</details>

---

<details>
<summary><strong>22. How does the <code>instanceof</code> operator work?</strong></summary>

`instanceof` checks if `Constructor.prototype` exists anywhere in the object's **prototype chain**.

```js
class Animal {}
class Dog extends Animal {}

const rex = new Dog();

rex instanceof Dog;    // true — Dog.prototype in chain
rex instanceof Animal; // true — Animal.prototype in chain
rex instanceof Object; // true — Object.prototype in chain

// How it works internally:
// walk rex's [[Prototype]] chain
// rex.__proto__ === Dog.prototype     → true for `instanceof Dog`
// rex.__proto__.__proto__ === Animal.prototype → true for `instanceof Animal`
```

**Pitfall — across iframes/realms:**
```js
// An array from another iframe fails instanceof
const arr = []; // from iframe
arr instanceof Array; // false — different Array constructor
Array.isArray(arr);   // true  — realm-safe check
```

**Custom `Symbol.hasInstance`:**
```js
class Even {
  static [Symbol.hasInstance](num) {
    return num % 2 === 0;
  }
}
2 instanceof Even; // true
3 instanceof Even; // false
```

> **Interview Note:** For type checking, prefer `Array.isArray()`, `typeof`, or `Object.prototype.toString.call()` over `instanceof` — they're more reliable across different execution contexts.

</details>

---

## Quick Reference Cheat Sheet

```
Prototype Chain lookup order:
  obj → obj.__proto__ → ... → Object.prototype → null

new keyword steps:
  1. Create {}
  2. Set [[Prototype]] to Constructor.prototype
  3. Run Constructor with this = {}
  4. Return the object

__proto__    → deprecated accessor to [[Prototype]] (every object)
.prototype   → object assigned to instances' [[Prototype]] (functions only)
[[Prototype]]→ internal link to parent object (every object)
```

| | `freeze` | `seal` | `assign` |
|--|---------|--------|---------|
| Add props | ❌ | ❌ | ✅ |
| Delete props | ❌ | ❌ | N/A |
| Modify props | ❌ | ✅ | ✅ |
| Depth | Shallow | Shallow | Shallow |

---

*Chapter 4 of JavaScript Interview Prep Series*
