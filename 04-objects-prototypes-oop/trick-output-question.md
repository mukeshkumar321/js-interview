# Chapter 4: Trick Output Questions — Objects, Prototypes & OOP

> Try to predict the output **before** expanding the answer. Each question reflects a real interview scenario.

---

## Objects & References

---

**Q1. What is the output?**

```js
const a = { x: 1 };
const b = a;

b.x = 99;

console.log(a.x);
console.log(a === b);
```

<details>
<summary>Show Output & Explanation</summary>

```
99
true
```

**Explanation:**  
Objects are stored by reference. `b = a` makes `b` point to the **same object in memory**, not a copy. Mutating `b.x` mutates the same object `a` references. `a === b` is `true` because they hold the same reference.

</details>

---

**Q2. What is the output?**

```js
const obj1 = { a: 1 };
const obj2 = { a: 1 };

console.log(obj1 == obj2);
console.log(obj1 === obj2);
console.log(obj1.a === obj2.a);
```

<details>
<summary>Show Output & Explanation</summary>

```
false
false
true
```

**Explanation:**  
`obj1` and `obj2` are two different objects in memory — separate references, so both `==` and `===` return `false` even though they have identical content. Primitives compare by value, so `obj1.a === obj2.a` → `1 === 1` → `true`.

</details>

---

**Q3. What is the output?**

```js
const original = { a: 1, b: { c: 2 } };
const copy = { ...original };

copy.a = 99;
copy.b.c = 99;

console.log(original.a);
console.log(original.b.c);
```

<details>
<summary>Show Output & Explanation</summary>

```
1
99
```

**Explanation:**  
Spread `{ ...original }` creates a **shallow copy**. Top-level primitive `a` is copied by value — mutating `copy.a` doesn't affect `original.a`. But `b` is an object — only the reference is copied. Both `original.b` and `copy.b` point to the same nested object, so `copy.b.c = 99` mutates `original.b.c` too.

</details>

---

**Q4. What is the output?**

```js
const obj = Object.freeze({ name: 'Alice', address: { city: 'NY' } });

obj.name = 'Bob';
obj.address.city = 'LA';

console.log(obj.name);
console.log(obj.address.city);
```

<details>
<summary>Show Output & Explanation</summary>

```
Alice
LA
```

**Explanation:**  
`Object.freeze()` is **shallow**. Top-level property `name` cannot be changed — the assignment is silently ignored (or throws in strict mode). But `address` is an object reference — freezing the outer object doesn't freeze `address` itself, so `city` can still be mutated.

</details>

---

**Q5. What is the output?**

```js
const obj = { a: 1 };

Object.defineProperty(obj, 'b', {
  value: 2,
  writable: false,
  enumerable: false
});

obj.b = 99;

console.log(obj.b);
console.log(Object.keys(obj));
console.log('b' in obj);
```

<details>
<summary>Show Output & Explanation</summary>

```
2
['a']
true
```

**Explanation:**  
- `writable: false` → assignment `obj.b = 99` is silently ignored (throws in strict mode). `obj.b` stays `2`.
- `enumerable: false` → `Object.keys()` only returns enumerable own properties — `'b'` is excluded.
- `'b' in obj` → `in` checks existence (including non-enumerable), not enumerability → `true`.

</details>

---

**Q6. What is the output?**

```js
const obj = Object.seal({ name: 'Alice', age: 30 });

obj.name = 'Bob';
obj.city = 'NY';
delete obj.age;

console.log(obj.name);
console.log(obj.city);
console.log(obj.age);
```

<details>
<summary>Show Output & Explanation</summary>

```
Bob
undefined
30
```

**Explanation:**  
`Object.seal()` allows **modifying** existing properties but prevents adding or deleting.
- `obj.name = 'Bob'` → allowed ✅
- `obj.city = 'NY'` → adding blocked, `city` is never set → `undefined`
- `delete obj.age` → deletion blocked → `age` remains `30`

</details>

---

## Prototype Chain

---

**Q7. What is the output?**

```js
function Animal(name) {
  this.name = name;
}
Animal.prototype.speak = function () {
  return `${this.name} speaks`;
};

const dog = new Animal('Rex');

console.log(dog.hasOwnProperty('name'));
console.log(dog.hasOwnProperty('speak'));
console.log('speak' in dog);
```

<details>
<summary>Show Output & Explanation</summary>

```
true
false
true
```

**Explanation:**  
- `name` is set directly on `dog` via `this.name = name` in the constructor → own property → `true`
- `speak` is on `Animal.prototype`, not on `dog` itself → not own → `false`
- `in` operator checks the **entire prototype chain**, including inherited properties → `true`

</details>

---

**Q8. What is the output?**

```js
function Foo() {}
Foo.prototype.x = 1;

const a = new Foo();
const b = new Foo();

a.x = 2;

console.log(a.x);
console.log(b.x);

delete a.x;

console.log(a.x);
```

<details>
<summary>Show Output & Explanation</summary>

```
2
1
1
```

**Explanation:**  
- `a.x = 2` creates an **own property** on `a` that shadows `Foo.prototype.x`. `b` doesn't have an own `x`, so it reads from the prototype → `1`.
- `delete a.x` removes the own property. Now `a` no longer has own `x`, so the prototype chain lookup finds `Foo.prototype.x = 1` → `1`.

> **Key insight:** You can shadow prototype properties with own properties, and deleting the own property reveals the prototype value again.

</details>

---

**Q9. What is the output?**

```js
function Person(name) {
  this.name = name;
}

const alice = new Person('Alice');

console.log(alice instanceof Person);
console.log(alice instanceof Object);
console.log(typeof alice);
```

<details>
<summary>Show Output & Explanation</summary>

```
true
true
object
```

**Explanation:**  
- `instanceof Person` → `Person.prototype` is in `alice`'s chain → `true`
- `instanceof Object` → `Object.prototype` is at the end of every prototype chain → `true`
- `typeof` on any object (including instances) returns `'object'`

</details>

---

**Q10. What is the output?**

```js
const obj = Object.create(null);
obj.name = 'Alice';

console.log(obj.name);
console.log(obj.hasOwnProperty);
console.log(obj.toString);
```

<details>
<summary>Show Output & Explanation</summary>

```
Alice
undefined
undefined
```

**Explanation:**  
`Object.create(null)` creates an object with **no prototype** — the chain ends immediately at `null`. There is no `Object.prototype` in the chain, so inherited methods like `hasOwnProperty` and `toString` don't exist on this object.

> **Use case:** Pure hash maps / dictionaries where prototype pollution is a concern.

</details>

---

**Q11. What is the output?**

```js
function Animal(name) { this.name = name; }
Animal.prototype.type = 'animal';

function Dog(name) {
  Animal.call(this, name);
}
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

const rex = new Dog('Rex');

console.log(rex.name);
console.log(rex.type);
console.log(rex.constructor === Dog);
console.log(rex instanceof Animal);
```

<details>
<summary>Show Output & Explanation</summary>

```
Rex
animal
true
true
```

**Explanation:**  
- `rex.name` → set via `Animal.call(this, name)` → own property → `'Rex'`
- `rex.type` → not on `rex`, not on `Dog.prototype`, found on `Animal.prototype` → `'animal'`
- `rex.constructor === Dog` → we manually restored `Dog.prototype.constructor = Dog` (otherwise it would point to `Animal`) → `true`
- `rex instanceof Animal` → `Animal.prototype` is in the chain → `true`

</details>

---

## Classes & Inheritance

---

**Q12. What is the output?**

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
  speak() {
    return `${this.name} barks`;
  }
}

const d = new Dog('Rex');
console.log(d.speak());
console.log(d instanceof Dog);
console.log(d instanceof Animal);
```

<details>
<summary>Show Output & Explanation</summary>

```
Rex barks
true
true
```

**Explanation:**  
`Dog` overrides `speak()`. The own prototype method is found first in the chain — `Animal`'s `speak` is never reached. Both `instanceof` checks return `true` because both prototypes are in `d`'s chain.

</details>

---

**Q13. What is the output?**

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
  speak() {
    return super.speak() + ' and barks';
  }
}

const d = new Dog('Rex');
console.log(d.speak());
```

<details>
<summary>Show Output & Explanation</summary>

```
Rex makes a sound and barks
```

**Explanation:**  
`super.speak()` calls `Animal.prototype.speak` with `this = d`. So `this.name` is `'Rex'` from the `Dog` instance. The result is concatenated with `' and barks'`.

</details>

---

**Q14. What is the output?**

```js
class Person {
  constructor(name) {
    this.name = name;
  }
}

class Employee extends Person {
  constructor(name, company) {
    this.company = company;
    super(name);
  }
}

const e = new Employee('Alice', 'Google');
console.log(e.name);
```

<details>
<summary>Show Output & Explanation</summary>

```
ReferenceError: Must call super constructor in derived class before accessing 'this'
```

**Explanation:**  
In a derived class (one that uses `extends`), `this` is not available until `super()` is called. Accessing `this.company` before `super(name)` throws a `ReferenceError`.

**Fix:**
```js
constructor(name, company) {
  super(name);         // ✅ super first
  this.company = company;
}
```

</details>

---

**Q15. What is the output?**

```js
class Counter {
  #count = 0;

  increment() { this.#count++; }
  get value()  { return this.#count; }
}

const c = new Counter();
c.increment();
c.increment();

console.log(c.value);
console.log(c.#count);
```

<details>
<summary>Show Output & Explanation</summary>

```
2
SyntaxError: Private field '#count' must be declared in an enclosing class
```

**Explanation:**  
Private class fields (prefixed with `#`) are **truly private** — not just a convention. `c.value` works because `value` is a public getter. Accessing `c.#count` from outside the class is a `SyntaxError` caught at parse time.

</details>

---

**Q16. What is the output?**

```js
class Animal {
  static count = 0;

  constructor(name) {
    this.name = name;
    Animal.count++;
  }

  static getCount() {
    return Animal.count;
  }
}

const a1 = new Animal('Dog');
const a2 = new Animal('Cat');

console.log(Animal.getCount());
console.log(a1.count);
console.log(a1.getCount);
```

<details>
<summary>Show Output & Explanation</summary>

```
2
undefined
undefined
```

**Explanation:**  
- `Animal.count` is a **static property** — it belongs to the class itself, not instances.
- `Animal.getCount()` → `2` (two instances created)
- `a1.count` → `undefined` — instances don't have `count` on them
- `a1.getCount` → `undefined` — static methods are not on the prototype, only on the class constructor itself

</details>

---

**Q17. What is the output?**

```js
class Shape {
  area() { return 0; }
}

class Circle extends Shape {
  constructor(r) {
    super();
    this.r = r;
  }
  area() { return Math.PI * this.r ** 2; }
}

class Square extends Shape {
  constructor(s) {
    super();
    this.s = s;
  }
  area() { return this.s ** 2; }
}

const shapes = [new Circle(1), new Square(2), new Shape()];
console.log(shapes.map(s => s.area().toFixed(2)));
```

<details>
<summary>Show Output & Explanation</summary>

```
['3.14', '4.00', '0.00']
```

**Explanation:**  
This demonstrates **polymorphism** — same `area()` call, different behavior per class. `Circle` computes `π * 1² ≈ 3.14`, `Square` computes `2² = 4`, `Shape` returns `0`.

</details>

---

## Mixed / Hard

---

**Q18. What is the output?**

```js
function Foo() {
  this.x = 1;
}

Foo.prototype.x = 2;
Foo.prototype.y = 3;

const obj = new Foo();

console.log(obj.x);
console.log(obj.y);

delete obj.x;

console.log(obj.x);
```

<details>
<summary>Show Output & Explanation</summary>

```
1
3
2
```

**Explanation:**  
- `obj.x` → own property set in constructor → `1` (shadows prototype's `x = 2`)
- `obj.y` → not own, found on `Foo.prototype` → `3`
- After `delete obj.x` → own property removed → prototype lookup finds `Foo.prototype.x = 2` → `2`

</details>

---

**Q19. What is the output?**

```js
const obj = {
  val: 10,
  double() { return this.val * 2; }
};

const proxy = Object.create(obj);
proxy.val = 5;

console.log(proxy.double());
console.log(obj.double());
```

<details>
<summary>Show Output & Explanation</summary>

```
10
20
```

**Explanation:**  
`proxy.double()` — `double` is inherited from `obj`, but `this = proxy` at call time. `proxy.val = 5`, so `5 * 2 = 10`.  
`obj.double()` — called on `obj` directly, `this = obj`, `obj.val = 10`, so `10 * 2 = 20`.

> Methods on the prototype use `this` based on **who calls them**, not where they're defined.

</details>

---

**Q20. What is the output?**

```js
class A {
  greet() { return 'A'; }
}

class B extends A {
  greet() { return 'B'; }
}

class C extends B {
  greet() { return super.greet() + 'C'; }
}

const c = new C();
console.log(c.greet());
```

<details>
<summary>Show Output & Explanation</summary>

```
BC
```

**Explanation:**  
`super.greet()` inside `C` calls `B.prototype.greet()` (the immediate parent), which returns `'B'`. Then `'B' + 'C'` = `'BC'`. `A.prototype.greet` is not reached because `B` overrides it.

</details>

---

**Q21. What is the output?**

```js
function Person(name) {
  this.name = name;
  if (typeof Person.count === 'undefined') {
    Person.count = 0;
  }
  Person.count++;
}

new Person('Alice');
new Person('Bob');
new Person('Carol');

console.log(Person.count);
console.log(new Person('Dave').count);
```

<details>
<summary>Show Output & Explanation</summary>

```
3
undefined
```

**Explanation:**  
`Person.count` is a property on the **constructor function** itself, not on instances. The first `console.log(Person.count)` runs after 3 `new Person(...)` calls (Alice, Bob, Carol), so `Person.count = 3`. The `new Person('Dave')` only runs inside the second `console.log` expression — after the first log has already printed. `new Person('Dave').count` looks for `count` on the instance — it's not there (own property check) and not on `Person.prototype` (only on `Person` the function) → `undefined`.

</details>

---

**Q22. What is the output?**

```js
class Vehicle {
  constructor(type) {
    this.type = type;
  }

  describe() {
    return `I am a ${this.type}`;
  }
}

class Car extends Vehicle {
  constructor(brand) {
    super('car');
    this.brand = brand;
  }

  describe() {
    return `${super.describe()} made by ${this.brand}`;
  }
}

const tesla = new Car('Tesla');
console.log(tesla.describe());
console.log(tesla.type);
console.log(tesla instanceof Vehicle);
```

<details>
<summary>Show Output & Explanation</summary>

```
I am a car made by Tesla
car
true
```

**Explanation:**  
`super('car')` calls `Vehicle`'s constructor, setting `this.type = 'car'`. `super.describe()` calls `Vehicle.prototype.describe()` with `this = tesla` → `'I am a car'`. Concatenation gives the full string. `tesla.type = 'car'` (set by parent constructor). `instanceof Vehicle` → `true`.

</details>

---

**Q23. What is the output?**

```js
const animal = {
  type: 'animal',
  describe() {
    return `I am a ${this.type}`;
  }
};

const dog = Object.create(animal);
dog.type = 'dog';

const puppy = Object.create(dog);

console.log(puppy.describe());
console.log(puppy.type);
console.log(Object.getPrototypeOf(puppy) === dog);
```

<details>
<summary>Show Output & Explanation</summary>

```
I am a dog
dog
true
```

**Explanation:**  
Prototype chain: `puppy → dog → animal → Object.prototype → null`  
- `puppy.describe()` → not on `puppy`, not on `dog`, found on `animal`. Called with `this = puppy`. `puppy` has no own `type`, so walks up chain to `dog.type = 'dog'` → `'I am a dog'`
- `puppy.type` → not own, found on `dog` → `'dog'`
- `Object.getPrototypeOf(puppy) === dog` → `true`

</details>

---

**Q24. What is the output?**

```js
class EventEmitter {
  #listeners = {};

  on(event, fn) {
    if (!this.#listeners[event]) this.#listeners[event] = [];
    this.#listeners[event].push(fn);
    return this;
  }

  emit(event, data) {
    (this.#listeners[event] || []).forEach(fn => fn(data));
    return this;
  }
}

const emitter = new EventEmitter();

emitter
  .on('data', x => console.log('A:', x))
  .on('data', x => console.log('B:', x * 2))
  .emit('data', 5);
```

<details>
<summary>Show Output & Explanation</summary>

```
A: 5
B: 10
```

**Explanation:**  
Method chaining works because `on()` and `emit()` both `return this`. Two listeners are registered for `'data'`. When `emit('data', 5)` fires, both are called in registration order. Private field `#listeners` is inaccessible from outside the class.

> This combines: **private fields**, **method chaining**, **encapsulation**, and **the Observer pattern** — all common interview topics in one question.

</details>

---

*Chapter 4 — Trick Output Questions | Objects, Prototypes & OOP*
