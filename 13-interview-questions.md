# JavaScript Interview Questions (3–5 Years Experience)

These questions cover JS fundamentals, closures, async, error handling, modules, ES6+, arrays, performance, design patterns, browser internals, and machine coding discussions commonly asked in frontend interviews.

---

# Core JavaScript Fundamentals

## Data Types & Variables

1. What are the primitive data types in JavaScript?
2. What is the difference between `null` and `undefined`?
3. What is the difference between `==` and `===`?
4. What is Type Coercion?
5. What is the `typeof` operator and what are its quirks?
6. Why does `typeof null` return `"object"`?
7. What is the difference between `var`, `let`, and `const`?
8. What is Hoisting?
9. What is the Temporal Dead Zone (TDZ)?
10. Why is `var` considered problematic?

## Scope & Closures

11. What is Scope?
12. What is the difference between Lexical Scope and Dynamic Scope?
13. What is a Closure?
14. How do Closures work internally?
15. Give a real-world use case of Closures.
16. What is the classic Closure bug with `var` in loops?
17. How do you fix the loop Closure bug?
18. What is a Module Pattern using Closures?
19. What is an IIFE and why use it?
20. What is the difference between function scope and block scope?

## `this` Keyword

21. What is `this` in JavaScript?
22. How does `this` behave in a regular function vs arrow function?
23. What is Implicit Binding?
24. What is Explicit Binding (`call`, `apply`, `bind`)?
25. What is `new` Binding?
26. What is the Default Binding?
27. What is the order of precedence for `this` binding?
28. Why do arrow functions not have their own `this`?
29. What does `bind()` return?
30. How does `this` behave inside `setTimeout`?

## Event Loop

31. What is the Event Loop?
32. What is the Call Stack?
33. What is the Heap?
34. What are Macro Tasks vs Micro Tasks?
35. What is the order of execution: `setTimeout`, `Promise`, `queueMicrotask`?
36. What happens when the Call Stack is blocked?
37. What is `requestAnimationFrame` and how does it fit in the Event Loop?

---

# Prototypes & Inheritance

## Prototypal Inheritance

38. What is a Prototype in JavaScript?
39. What is the Prototype Chain?
40. What is `Object.create()`?
41. What is the difference between `__proto__` and `prototype`?
42. What happens when you access a property not on the object?
43. What is `hasOwnProperty()`?
44. What is the difference between Prototypal and Classical Inheritance?

## Classes

45. What are ES6 Classes?
46. Are ES6 Classes just syntactic sugar?
47. What is a Constructor?
48. What is `super()`?
49. What is the difference between instance methods and static methods?
50. What is method overriding?
51. How does `instanceof` work?
52. What is the difference between `Object.assign()` and spread operator for objects?

---

# Functions

## Function Fundamentals

53. What is the difference between a Function Declaration and a Function Expression?
54. What is an Arrow Function?
55. Why can't Arrow Functions be used as constructors?
56. What is a Generator Function?
57. What is `yield`?
58. What are the use cases of Generator Functions?
59. What is a Pure Function?
60. What is a Side Effect?
61. What is a Higher-Order Function?
62. What is Currying?

## Advanced Function Concepts

63. What is Partial Application?
64. What is Function Composition?
65. What is Memoization?
66. Implement a `memoize()` utility.
67. What is the difference between `call()`, `apply()`, and `bind()`?
68. What is `arguments` object and how does it differ from rest parameters?
69. What is a Thunk?
70. What is Recursion and what are its risks?

---

# Asynchronous JavaScript

## Callbacks & Promises

71. What is a Callback?
72. What is Callback Hell?
73. What is a Promise?
74. What are the three states of a Promise?
75. What is the difference between `.then()` and `async/await`?
76. What is `Promise.all()`?
77. What is `Promise.allSettled()`?
78. What is `Promise.race()`?
79. What is `Promise.any()`?
80. How do you handle errors in Promises?

## Async/Await

81. What is `async/await`?
82. What does `async` return?
83. What happens when you `await` a non-`Promise`?
84. How do you handle errors with `async/await`?
85. What is the difference between sequential and parallel `await`?
86. When would you prefer `Promise.all` over sequential awaits?
87. What is the problem with `await` inside `forEach`?

## Advanced Async

88. What is a Race Condition?
89. How do you prevent Race Conditions in async code?
90. What is `AbortController`?
91. How do you implement a retry mechanism for failed API calls?
92. What is Debouncing?
93. What is Throttling?
94. Implement `debounce()` from scratch.
95. Implement `throttle()` from scratch.
96. What is the difference between Debounce and Throttle?
97. When would you use Debounce vs Throttle?

---

# Error Handling

98. What is the difference between `throw` and `return`?
99. What happens if you `throw` inside a `try` block?
100. What does `finally` always do?
101. Does `finally` run even if there is a `return` inside `try`?
102. What are the built-in `Error` types in JavaScript?
103. How do you create a Custom Error class?
104. How do you distinguish between `Error` types using `instanceof`?
105. How do you handle errors in `async/await` vs `Promises`?
106. What is an Unhandled Promise Rejection?
107. How do you catch Unhandled Promise Rejections globally?

---

# Modules

108. What is the difference between ES Modules and CommonJS?
109. What is a named export vs a default export?
110. Can you have multiple default exports in one file?
111. What is a re-export and when would you use it?
112. What is a dynamic `import()`?
113. When would you use dynamic import over static import?
114. What is tree shaking and how do ES Modules enable it?
115. Why can't CommonJS be tree-shaken as effectively?
116. What is a circular dependency and how do ES Modules handle it?
117. What is the difference between `import` and `require`?

---

# ES6+ Features

## Destructuring & Spread

118. What is Destructuring?
119. What is the difference between `Array` and `Object Destructuring`?
120. What is a Default Value in Destructuring?
121. What is the Spread Operator?
122. What is the Rest Parameter?
123. What is the difference between Spread and Rest?
124. How do you deep clone an object?
125. What are the limitations of spread for cloning?

## Optional Chaining & Nullish Coalescing

126. What is Optional Chaining (`?.`)?
127. What is the Nullish Coalescing operator (`??`)?
128. What is the difference between `??` and `||`?
129. What are Logical Assignment operators (`&&=`, `||=`, `??=`)?

## Iterators & Iterables

130. What is an Iterable?
131. What is an Iterator?
132. What is the Iterator Protocol?
133. What is a `Symbol.iterator`?
134. How does `for...of` work internally?
135. What is the difference between `for...of` and `for...in`?
136. What is a `Map` vs a plain `Object`?
137. What is a `Set`?
138. What is a `WeakMap`?
139. What is a `WeakSet` and when would you use it?

## Symbols, Proxies & Reflect

140. What is a `Symbol`?
141. Why are Symbols useful as object keys?
142. What is a `Proxy`?
143. What are traps in a `Proxy`?
144. What is `Reflect`?
145. How would you use a `Proxy` to validate object properties?
146. What are well-known Symbols?

---

# Array & Object Methods

## Array Methods

147. What is the difference between `map()`, `filter()`, and `reduce()`?
148. What does `reduce()` return when the array is empty and no initial value is provided?
149. What is the difference between `forEach()` and `map()`?
150. What is `flat()` and `flatMap()`?
151. What is the difference between `find()` and `filter()`?
152. What is the difference between `some()` and `every()`?
153. What does `Array.from()` do?
154. What is the difference between `splice()` and `slice()`?
155. What is the difference between `push/pop` and `shift/unshift`?
156. How do you remove duplicates from an array?

## Object Methods & Immutability

157. What does `Object.keys()`, `Object.values()`, `Object.entries()` return?
158. What is `Object.freeze()`?
159. What is the difference between `Object.freeze()` and `const`?
160. Is `Object.freeze()` deep or shallow?
161. What is `Object.seal()`?
162. What is `Object.is()` and how does it differ from `===`?
163. What is `Object.defineProperty()`?
164. What is the difference between `enumerable`, `configurable`, and `writable` property descriptors?

---

# Memory & Performance

## Memory Management

165. How does JavaScript Garbage Collection work?
166. What is Mark and Sweep?
167. What is a Memory Leak?
168. What are common causes of Memory Leaks?
169. How would you detect a Memory Leak?
170. Why can Closures cause Memory Leaks?
171. Why can detached DOM nodes cause Memory Leaks?
172. How does `WeakMap` help prevent Memory Leaks?

## Performance

173. What is the Critical Rendering Path?
174. What is Reflow vs Repaint?
175. What causes Reflow?
176. How do you minimize Layout Thrashing?
177. What is `requestAnimationFrame` and when should you use it?
178. What is `requestIdleCallback`?
179. What is Code Splitting?
180. What is Tree Shaking?
181. What is Lazy Loading?
182. What is the difference between `defer` and `async` script attributes?

---

# Browser & DOM

## DOM Manipulation

183. What is the DOM?
184. What is the difference between `innerHTML` and `textContent`?
185. Why is `innerHTML` dangerous?
186. What is XSS?
187. What is Event Bubbling?
188. What is Event Capturing?
189. What is Event Delegation?
190. Explain `stopPropagation()` vs `preventDefault()`.
191. What is a Shadow DOM?
192. What is a Web Component?

## Web APIs

193. What is `localStorage` vs `sessionStorage` vs `Cookies`?
194. What is `IndexedDB`?
195. What is a Service Worker?
196. What is a Web Worker?
197. What is the difference between Service Worker and Web Worker?
198. What is the Fetch API?
199. What is CORS?
200. What are HTTP methods and when are they used?
201. What is the difference between HTTP/1.1 and HTTP/2?
202. What is WebSocket?

---

# Design Patterns

## Creational Patterns

203. What is the Singleton Pattern?
204. What is the Factory Pattern?
205. What is the Builder Pattern?

## Structural Patterns

206. What is the Module Pattern?
207. What is the Decorator Pattern?
208. What is the Proxy Pattern?

## Behavioral Patterns

209. What is the Observer Pattern?
210. What is the Publish-Subscribe Pattern?
211. What is the difference between Observer and Pub-Sub?
212. What is the Strategy Pattern?
213. What is the Command Pattern?
214. What is the Mediator Pattern?

## Practical

215. How would you implement an Event Emitter from scratch?
216. How would you implement a simple Pub-Sub system?
217. How does the Observer Pattern relate to `addEventListener`?

---

# JavaScript Internals

## Execution Context

218. What is an Execution Context?
219. What is the Global Execution Context?
220. What is a Function Execution Context?
221. What is the Execution Context Stack?
222. What is the creation phase vs execution phase of an Execution Context?
223. What is the Scope Chain?
224. What is Variable Environment vs Lexical Environment?

## Compilation & Runtime

225. Is JavaScript interpreted or compiled?
226. What is JIT Compilation?
227. What is the V8 Engine?
228. What are Hidden Classes in V8?
229. What is Inline Caching?
230. How does V8 optimize hot functions?
231. What is Deoptimization?

---

# Senior-Level Questions (Most Asked)

232. Explain the Event Loop with a code example showing call stack, macro task, and micro task order.
233. What would happen if you called `await` inside a `Promise` constructor?
234. Explain why `0.1 + 0.2 !== 0.3` in JavaScript.
235. What is the difference between `structuredClone()` and `JSON.parse(JSON.stringify())`?
236. How does JavaScript handle concurrency despite being single-threaded?
237. Explain Prototype Chain with a diagram walkthrough.
238. How would you implement private class fields before the `#` syntax was available?
239. What is the output of this code and why? (closure + setTimeout + var)
240. Explain how `async/await` is implemented under the hood using Generators.
241. A page has severe jank. Walk me through your performance debugging process.

---

# Interview Readiness Checklist

If you can confidently answer:

- ✅ 80+ Questions → Good for Mid-Level JS Interviews
- ✅ 140+ Questions → Strong for 4–5 Years Experience
- ✅ 190+ Questions → Senior Frontend Interview Ready
- ✅ 241 Questions → JS Internals & Architecture Level Understanding
