# JavaScript Modules — ES Modules, Globals & CommonJS

## 1. Why modules?

Modules split code across files while keeping variables isolated and making dependencies explicit.

```js
// utils.js
export function formatDate(date) {
    return date.toISOString();
}
```

```js
// app.js
import { formatDate } from "./utils.js";
```

Without modules, classic scripts can rely on globals and manually ordered `<script>` tags. This becomes fragile as an application grows.

---

## 2. ES Modules (ESM)

A browser file is treated as an ES module with:

```html
<script type="module" src="./app.js"></script>
```

Modules support `import` and `export`.

### Named exports

A module can have multiple named exports:

```js
// math.js
export function add(a, b) {
    return a + b;
}

export function subtract(a, b) {
    return a - b;
}
```

Import them by name:

```js
import { add, subtract } from "./math.js";
```

A named export can be renamed:

```js
import { add as sum } from "./math.js";
```

### Default export

A module can have one default export:

```js
// UserCard.jsx
export default function UserCard() {}
```

Import without braces:

```js
import UserCard from "./UserCard.jsx";
```

The importer can choose its local name:

```js
import Card from "./UserCard.jsx";
```

### Mental model

```text
Named:
export { foo }  →  import { foo }

Default:
export default foo  →  import anything
```

---

## 3. Modules and scope

An ES module has its own module scope. Top-level variables are not automatically placed on the browser's global `window` object.

```js
// module.js
var x = 10;
let y = 20;
const z = 30;

console.log(x);        // 10
console.log(window.x); // undefined
```

This is true even for `var`.

### Important: modules can still use `window`

```js
window.alert("Hello");
window.setTimeout(() => {}, 1000);
console.log(window.innerWidth);
console.log(window.localStorage);
```

So:

```text
ES module
├── Can access window?              YES
├── Can use window APIs/methods?   YES
└── Own variables become window properties? NO
```

Module scope and browser global APIs are separate concepts.

---

## 4. Classic scripts and `window`

A normal script is a classic script:

```html
<script src="app.js"></script>
```

In a browser, top-level `var` in a classic global script becomes a property of `window`:

```html
<script>
    var a = 10;
    let b = 20;
    const c = 30;
</script>
```

Conceptually:

```text
var a   → window.a
let b   → global lexical binding, NOT window.b
const c → global lexical binding, NOT window.c
```

Therefore:

```js
window.a; // 10
window.b; // undefined
window.c; // undefined
```

Do not remember "all global variables become window properties." The precise rule is that top-level `var` in a classic browser script becomes a `window` property; top-level `let`/`const` do not.

---

## 5. Why avoid globals in larger applications?

Globals work for small pages, but they create implicit dependencies and possible name collisions.

Example:

```html
<script src="utils.js"></script>
<script src="api.js"></script>
<script src="app.js"></script>
```

If `app.js` uses `formatDate()` from `utils.js`, the dependency is not declared in `app.js`; it depends on `utils.js` having already executed.

If the order is reversed, `app.js` may try to use `formatDate` before it exists.

Globals can also collide:

```js
// file A
window.user = "Tanay";

// file B
window.user = { id: 123 };
```

The second assignment overwrites the first.

Modules make dependencies explicit:

```js
import { formatDate } from "./utils.js";
```

The benefit is not that classic scripts are broken; modules simply scale better through explicit dependencies and isolated scope.

---

## 6. What should a plain web application use?

Even without React or another framework, a modern browser application should generally use ES modules:

```html
<script type="module" src="./js/app.js"></script>
```

Then use `import`/`export` between files.

For a tiny one- or two-file page, classic scripts can still be perfectly workable. Globals should be used deliberately rather than as the default architecture for a growing application.

---

## 7. Why React uses modules

React applications are usually organized as a module dependency graph:

```text
App.jsx
├── UserCard.jsx
│   └── utils.js
├── Navbar.jsx
└── userService.js
```

Example:

```js
import UserCard from "./UserCard.jsx";
import { formatDate } from "./utils.js";
```

Modern React build tools such as Vite process this dependency graph and bundle/transform the application for the browser.

Imports and exports are not React-specific; React applications use JavaScript's module system.

---

## 8. CommonJS vs ESM

CommonJS is another JavaScript module system, historically associated with Node.js.

### CommonJS

```js
const express = require("express");

module.exports = myFunction;
```

Import:

```js
const myFunction = require("./myFunction");
```

### ES Modules

```js
import express from "express";

export default myFunction;
```

Import:

```js
import myFunction from "./myFunction.js";
```

| | CommonJS | ESM |
|---|---|---|
| Import | `require()` | `import` |
| Export | `module.exports` / `exports` | `export` / `export default` |
| Browser native support | No, not as CommonJS | Yes |
| Static module syntax | No | Yes |
| Modern React | Generally not the application syntax | Yes |

Modern Node.js also supports ESM, so "CommonJS = Node" and "ESM = browser" are not absolute rules.

### Static ESM syntax

```js
import { add } from "./math.js";
```

ESM dependencies are statically analyzable, which enables tooling and optimizations such as tree shaking.

---

## 9. Final mental model

```text
Classic <script>
    ↓
normal script execution
    ↓
top-level var in a global script → window property

<script type="module">
    ↓
ES module
    ↓
own module scope
    ↓
explicit import/export
    ↓
variables do not automatically become window properties
```

And:

```text
ESM:
import / export

CommonJS:
require / module.exports
```

### Interview takeaway

> ES modules provide isolated module scope and explicit dependencies through `import` and `export`. Named exports are imported by name using braces, while a module can have one default export. In browsers, `type="module"` enables ESM, and module variables do not automatically become `window` properties. CommonJS uses `require` and `module.exports` and is a different module system that is still used in Node.js.
