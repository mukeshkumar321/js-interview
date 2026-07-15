# JavaScript Interview Questions (3–5 Years Experience)

These questions cover JS fundamentals, closures, async, error handling, modules, ES6+, arrays, performance, design patterns, browser internals, and machine coding discussions commonly asked in frontend interviews.

---

# Core JavaScript Fundamentals

## Data Types & Variables

1. What are the primitive data types in JavaScript?
2. What is the difference between Primitive and Reference types?
3. What is the difference between `null` and `undefined`?
4. What is the difference between `==` and `===`?
5. What is Type Coercion?
6. What is the `typeof` operator and what are its quirks?
7. Why does `typeof null` return `"object"`?
8. What are Truthy and Falsy values?
9. What is `NaN`? Why is `NaN !== NaN`?
10. What is the difference between `var`, `let`, and `const`?
11. What is Hoisting?
12. What is the Temporal Dead Zone (TDZ)?
13. Why is `var` considered problematic?
14. What is the difference between Shallow Copy and Deep Copy? (Overview only—implementation can be covered later.)

## Scope & Closures

15. What is Scope?
16. What is the difference between Lexical Scope and Dynamic Scope?
17. What is the difference between function scope and block scope?
18. What is a Closure?
19. How do Closures work internally?
20. Give a real-world use case of Closures.
21. What is the classic Closure bug with `var` in loops?
22. How do you fix the loop Closure bug?
23. What is a Module Pattern using Closures?
24. What is an IIFE and why use it?
25. Can Closures cause memory leaks?

## `this` Keyword

26. What is `this` in JavaScript?
27. How does `this` behave in a regular function vs an arrow function?
28. What is Default Binding?
29. What is Implicit Binding?
30. What is Explicit Binding (`call`, `apply`, `bind`)?
31. What is the difference between `call()`, `apply()`, and `bind()`?
32. What is `new` Binding?
33. What is the order of precedence for `this` binding?
34. Why do arrow functions not have their own `this`?
35. Can `call()`, `apply()`, or `bind()` change the `this` value of an arrow function?
36. What does `bind()` return?
37. How does `this` behave when an object method is extracted and called separately?
38. How does `this` behave inside `setTimeout`?

## Execution Context

39. What is an Execution Context?
40. What is the Global Execution Context?
41. What is a Function Execution Context?
42. What is the Execution Context Stack (Call Stack)?
43. What happens during the Creation Phase and Execution Phase of an Execution Context?
44. What is the Scope Chain?
45. What is the difference between the Variable Environment and the Lexical Environment?
46. How are Hoisting and the Execution Context related?
47. What is the difference between the Global Execution Context and a Function Execution Context?
48. How is a new Execution Context created during a function call?

## Event Loop

49. What is the Event Loop?
50. What is the Call Stack?
51. What is the Heap?
52. What are Web APIs?
53. What are Macro Tasks and Micro Tasks?
54. What is the Microtask Queue?
55. What is the order of execution between `setTimeout`, `Promise`, and `queueMicrotask()`?
56. Can `setTimeout(fn, 0)` execute immediately? Why?
57. What happens when the Call Stack is blocked?
58. What is `requestAnimationFrame()` and how does it fit into the Event Loop?
59. How do Promises interact with the Event Loop?
60. What is the difference between the Microtask Queue and the Task Queue (Callback Queue)?

---

# Modern JavaScript (ES6+)

## Destructuring & Spread

61. What is Destructuring?
62. What is the difference between `Array` and `Object Destructuring`?
63. What are Default Values in Destructuring?
64. What is the Spread Operator?
65. What is the Rest Parameter?
66. What is the difference between Spread and Rest?
67. How do you deep clone an object?
68. What are the limitations of the Spread Operator for cloning?
69. What are computed property names?
70. What are enhanced object literals?

## Optional Chaining & Nullish Coalescing

71. What is Optional Chaining (`?.`)?
72. What is the Nullish Coalescing operator (`??`)?
73. What is the difference between `??` and `||`?
74. What are Logical Assignment operators (`&&=`, `||=`, `??=`)?

## Iterators & Iterables

75. What is an Iterable?
76. What is an Iterator?
77. What is the Iterator Protocol?
78. What is a `Symbol.iterator`?
79. How does `for...of` work internally?
80. What is the difference between `for...of` and `for...in`?
81. What is the difference between `Map` and a plain `Object`?
82. What is a `Set`?
83. What is a `WeakMap`?
84. What is a `WeakSet`, and when would you use it?

---

# Asynchronous JavaScript

## Callbacks & Promises

85. What is a Callback?
86. What is Callback Hell?
87. What is a Promise?
88. What are the three states of a Promise?
89. What is the difference between `.then()` and `async/await`?
90. What is `Promise.all()`?
91. What is `Promise.allSettled()`?
92. What is `Promise.race()`?
93. What is `Promise.any()`?
94. How do you handle errors in Promises?
95. What is the difference between `Promise.all()`, `Promise.allSettled()`, `Promise.race()`, and `Promise.any()`?

## Async/Await

96. What is `async/await`?
97. What does an async function return?
98. What happens when you `await` a non-`Promise`?
99. How do you handle errors with `async/await`?
100. What is the difference between sequential and parallel `await`?
101. When would you prefer `Promise.all()` over sequential awaits?
102. What is the problem with using `await` inside `forEach()`?
103. Why doesn't `Array.prototype.forEach()` work well with `async/await`?
104. Can you use `await` outside an async function?

---

# Functions

## Function Fundamentals

105. What is the difference between a Function Declaration and a Function Expression?
106. What is an Arrow Function?
107. What are the differences between Arrow Functions and Regular Functions?
108. Why can't Arrow Functions be used as constructors?
109. What is a Generator Function?
110. What is `yield`?
111. What are the use cases of Generator Functions?
112. What is a Pure Function?
113. What is a Side Effect?
114. What is a Higher-Order Function (HOF)?
115. What is Currying?

---

# Prototypes & Inheritance

## Prototypal Inheritance

116. What is a Prototype in JavaScript?
117. What is the Prototype Chain?
118. How does JavaScript perform property lookup through the Prototype Chain?
119. What is `Object.create()`?
120. What is the difference between `__proto__` and `prototype`?
121. What is the difference between a function's `prototype` property and an object's `[[Prototype]]`?
122. What happens when you access a property that doesn't exist on an object?
123. What is `hasOwnProperty()`?
124. What is the difference between Prototypal and Classical Inheritance?
125. How do you create inheritance in JavaScript without using ES6 classes?

## Classes

126. What are ES6 Classes?
127. Are ES6 Classes just syntactic sugar?
128. What is a Constructor?
129. What is `super()`?
130. What is the difference between instance methods and static methods?
131. What is method overriding?
132. How does `instanceof` work?
133. What is the difference between `Object.assign()` and the spread operator for objects?
134. When would you choose Prototypes over ES6 Classes?

---

# Array & Object Methods

## Array Methods

135. What is the difference between `map()`, `filter()`, and `reduce()`?
136. What does `reduce()` return when the array is empty and no initial value is provided?
137. What is the difference between `forEach()` and `map()`?
138. What is `flat()` and `flatMap()`?
139. What is the difference between `find()` and `filter()`?
140. What is the difference between `some()` and `every()`?
141. What does `Array.from()` do?
142. What is the difference between `splice()` and `slice()`?
143. What is the difference between `push()`/`pop()` and `shift()`/`unshift()`?
144. How do you remove duplicates from an array?
145. What is the difference between `Array.isArray()` and `instanceof Array`?
146. How do you flatten a nested array?

## Object Methods & Immutability

147. What do `Object.keys()`, `Object.values()`, and `Object.entries()` return?
148. What is `Object.freeze()`?
149. What is the difference between `Object.freeze()` and `const`?
150. Is `Object.freeze()` deep or shallow?
151. What is `Object.seal()`?
152. What is `Object.defineProperty()`?
153. What is the difference between `enumerable`, `configurable`, and `writable` property descriptors?
154. What is the difference between `Object.assign()` and the object spread operator?
155. How do you merge two objects without mutating the originals?

---

# Error Handling

156. What is the difference between `throw` and `return`?
157. What happens if you `throw` inside a `try` block?
158. What does `finally` always do?
159. Does `finally` run even if there is a `return` inside `try`?
160. What are the built-in `Error` types in JavaScript?
161. How do you create a Custom Error class?
162. How do you distinguish between `Error` types using `instanceof`?
163. What is the difference between `throw new Error()` and `console.error()`?
164. How do you handle errors in `async/await` vs `Promises`?
165. What is an Unhandled Promise Rejection?
166. How do you catch Unhandled Promise Rejections globally?
167. When should you throw an error instead of returning a value?

---

# Modules

168. What is the difference between ES Modules and CommonJS?
169. What is a named export vs a default export?
170. Can you have multiple default exports in one file?
171. What is a re-export and when would you use it?
172. What is a dynamic `import()`?
173. When would you use dynamic import over static import?
174. What is tree shaking and how do ES Modules enable it?
175. Why can't CommonJS be tree-shaken as effectively?
176. What is a circular dependency and how do ES Modules handle it?
177. What is the difference between `import` and `require`?

---

# Memory Management

178. How does JavaScript Garbage Collection work?
179. What is the Mark-and-Sweep Garbage Collection algorithm?
180. What is a Memory Leak?
181. What are the common causes of Memory Leaks in JavaScript?
182. How would you detect a Memory Leak?
183. Why can Closures cause Memory Leaks?
184. Why can detached DOM nodes cause Memory Leaks?
185. How does `WeakMap` help prevent Memory Leaks?
186. What is the difference between `Map` and `WeakMap`?
187. What is the difference between `Set` and `WeakSet`?
188. What are some best practices to prevent Memory Leaks in JavaScript applications?

---

# Browser & DOM

## DOM Manipulation

189. What is the DOM?
190. What is the difference between `innerHTML`, `innerText`, and `textContent`?
191. Why is `innerHTML` dangerous?
192. What is XSS?
193. What is Event Bubbling?
194. What is Event Capturing?
195. What is Event Delegation?
196. Explain `stopPropagation()` vs `preventDefault()`.
197. What is the difference between `event.target` and `event.currentTarget`?
198. What is the difference between `addEventListener()` and inline event handlers?
199. What is a Shadow DOM?
200. What is a Web Component?
201. When should you use Event Delegation?

## Web APIs

202. What is the difference between `localStorage`, `sessionStorage`, and `Cookies`?
203. What is `IndexedDB`?
204. What is a Service Worker?
205. What is a Web Worker?
206. What is the difference between a Service Worker and a Web Worker?
207. What is the Fetch API?
208. What is CORS?
209. What are HTTP methods, and when are they used?
210. What is a WebSocket?
211. What is the difference between Fetch API and XMLHttpRequest (XHR)?
212. What is the same-origin policy?
213. What is the difference between authentication and authorization?

---

# Interview Readiness Checklist

If you can confidently answer:

- ✅ 80+ Questions → Good for Mid-Level JS Interviews
- ✅ 140+ Questions → Strong for 4–5 Years Experience
- ✅ 200+ Questions → Senior Frontend Interview Ready
- ✅ 213 Questions → JS Internals & Architecture Level Understanding
