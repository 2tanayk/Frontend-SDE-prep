# Browser Storage — localStorage, sessionStorage & Cookies

## 1. localStorage

Persistent client-side storage.

```js
localStorage.setItem("theme", "dark");

localStorage.getItem("theme"); // "dark"
localStorage.removeItem("theme");
localStorage.clear();
```

- Survives page refreshes and browser restarts.
- Scoped to the website's origin.
- Data is **not automatically sent with HTTP requests**.
- Accessible from JavaScript.
- Useful for preferences such as theme, language, or other non-sensitive persistent client-side data.

## 2. sessionStorage

Temporary client-side storage associated with a browser tab/session.

```js
sessionStorage.setItem("currentStep", "3");

sessionStorage.getItem("currentStep"); // "3"
```

Think:

> "Remember this while I'm working in this tab."

- Survives page refreshes.
- Available while that tab's browsing session exists.
- Closing the tab normally ends the session and clears its sessionStorage.
- A different tab has its own sessionStorage.
- Data is **not automatically sent with HTTP requests**.
- Accessible from JavaScript.
- Useful for temporary UI state, multi-step forms, or tab-specific data.

### Simple distinction

**localStorage** → "Remember this for later."

**sessionStorage** → "Remember this while this tab is open."

## 3. Cookies

Small pieces of data stored by the browser that can be automatically sent with matching HTTP requests.

```js
document.cookie = "theme=dark";
```

- Much smaller than Web Storage; ~4 KB per cookie is the conventional limit.
- Can have an expiration time.
- Scoped using attributes such as domain and path.
- Unlike localStorage/sessionStorage, cookies can automatically travel to the server with applicable requests.
- JavaScript can read normal cookies, but an `HttpOnly` cookie cannot be read by JavaScript.

Common security attributes:

- **HttpOnly** → JavaScript cannot access the cookie.
- **Secure** → cookie is sent only over HTTPS.
- **SameSite** → controls when cookies are sent in cross-site contexts and is an important CSRF defense.

## 4. Comparison

| | localStorage | sessionStorage | Cookies |
|---|---|---|---|
| Lifetime | Persistent until cleared | Current tab/session | Configurable |
| Survives refresh | Yes | Yes | Yes |
| Sent automatically with HTTP requests | No | No | Yes, when applicable |
| JavaScript access | Yes | Yes | Yes, unless HttpOnly |
| Typical capacity | Several MB | Several MB | ~4 KB per cookie |
| Main use | Persistent client data | Temporary/tab-specific data | Server-related state, sessions, etc. |

> Storage capacities are browser-dependent; the numbers above are approximate, not universal limits.

## 5. Authentication/security

Do not think of cookies as automatically "safe" and Web Storage as automatically "unsafe."

The important distinction is JavaScript access:

- A token in **localStorage/sessionStorage** can be read by JavaScript, so an XSS vulnerability can potentially expose it.
- An **HttpOnly cookie** cannot be read by JavaScript, which helps protect the credential from JavaScript-based theft.
- However, cookies are automatically sent with requests, so **CSRF** must also be considered. `SameSite` and, where needed, explicit CSRF protections are relevant.

## SDE-2 Mental Model

```
localStorage   = persistent client-side storage
sessionStorage = temporary, tab/session-scoped client-side storage
cookies        = small browser-stored data that can automatically travel
                 with applicable HTTP requests
```
