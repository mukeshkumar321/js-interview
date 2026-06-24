# 08 Event Loop & Async JavaScript

## JavaScript Concurrency Model

1. How does JavaScript, being single-threaded, handle asynchronous operations?
2. What is the Event Loop and why is it needed?
3. What are the main components of the Event Loop architecture?
   - Call Stack
   - Web APIs / Browser APIs
   - Callback Queue (Task Queue)
   - Microtask Queue
4. What is the difference between the Callback Queue and Microtask Queue?
5. What is the execution order between synchronous code, microtasks, and macrotasks?
6. What are microtasks and macrotasks? Give examples.
7. Why are Promise callbacks executed before `setTimeout()` callbacks?

## Timers & Callbacks

8. What happens internally when `setTimeout()` is executed?
9. Why does `setTimeout(fn, 0)` not execute immediately?
10. What is Callback Hell and why is it a problem?
11. What are common techniques used to avoid Callback Hell?

## Promises

12. What are Promises and what problems do they solve?
13. What are the three states of a Promise?
14. How do `.then()`, `.catch()`, and `.finally()` work?
15. How does Promise Chaining work?
16. What is Promise Error Propagation?
17. What happens when a Promise is resolved with another Promise?
18. What are `Promise.all()`, `Promise.allSettled()`, `Promise.race()`, and `Promise.any()`?
19. When would you use each Promise utility method?

## Async/Await

20. What is `async/await` and what problems does it solve?
21. What happens internally when JavaScript encounters an `await` statement?
22. What is the difference between Promise Chaining and `async/await`?
23. How is error handling done with `async/await`?
24. What are common mistakes developers make with `async/await`?

## Advanced Async Patterns

25. How can multiple asynchronous operations be executed in parallel?
26. What is the difference between sequential and parallel execution?
27. Why is `Promise.all()` generally faster than awaiting requests one by one?
28. What are race conditions and how can they be avoided?
29. How does `AbortController` help in managing asynchronous operations?
30. How would you limit concurrent API requests in a frontend application?
