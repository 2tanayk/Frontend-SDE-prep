## Third-party libraries in a modular vanilla JS app

You do **not** need React or a framework to use third-party libraries modularly.

### CDN + ES modules

A CDN can provide a library as an ES module:

```html
<script type="module" src="./app.js"></script>
```

`app.js`:

```js
import dayjs from "https://cdn.jsdelivr.net/npm/dayjs/+esm";
import { greet } from "./utils.js";

const currentDate = dayjs().format("DD-MM-YYYY");
document.getElementById("date").textContent = `${greet("Tanay")} — ${currentDate}`;
```

`utils.js`:

```js
export function greet(name) {
    return `Hello ${name}`;
}
```

The browser follows the module graph:

```text
index.html
    ↓
app.js
    ├──→ utils.js
    └──→ Day.js CDN
```

Day.js is therefore an explicit module dependency; it does not need to be placed on `window`.

### Global/classic script approach

You can also load a third-party library as a classic script:

```html
<script src="https://some-library.com/library.js"></script>
<script src="./app.js"></script>
```

If it exposes a global, `app.js` can use `SomeLibrary.doSomething()`.

This is useful for tiny pages, simple widgets, or libraries intentionally designed to expose a global. Globals are not inherently bad; use them when global access is intentionally part of the design. Modules are preferred when you want isolated scope and explicit dependencies.

## npm + node_modules + bundling

Instead of a CDN, a larger vanilla JS application can install a library locally:

```bash
npm install dayjs
```

Then:

```js
import dayjs from "dayjs";
```

`node_modules/dayjs` contains the installed package. The browser does not normally load packages directly from `node_modules`. A bundler such as Vite can resolve that package, process the module graph, and produce browser-ready output.

### npm vs bundler

- **npm** → gets and manages dependencies.
- **Bundler** → takes your source + dependencies and prepares optimized browser assets.

Bundling is not what makes the code modular. `import`/`export` already provide the modularity.

## What is `dist`?

`dist` means **distribution** and is a conventional name for the build output:

```text
src/ + node_modules/
        ↓
     Bundler
        ↓
dist/
├── index.html
└── assets/
    ├── app-abc123.js
    └── styles-def456.css
```

`dist` is the complete built/deployable frontend. It is not necessarily one giant JavaScript file; code splitting can produce multiple JS chunks.

---

# npm, package.json, package-lock.json & Version Ranges

## 1. package.json

`package.json` is the project's manifest. It declares what the project depends on and can also contain project metadata and npm scripts.

Example:

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "scripts": {
    "dev": "vite",
    "build": "vite build"
  },
  "dependencies": {
    "dayjs": "^1.11.13"
  },
  "devDependencies": {
    "vite": "^7.0.0"
  }
}
```

Think:

> **package.json = what my project needs.**

---

## 2. package-lock.json

`package-lock.json` records the dependency versions npm actually resolved/installed, along with the dependency tree.

For example, `package.json` might say:

```json
"dayjs": "^1.11.13"
```

while the lockfile records the exact resolved version used by the project.

Think:

> **package-lock.json = the exact dependency resolution npm locked for the project.**

Do not normally edit the lockfile manually. Let npm update it.

If `package.json` is manually changed, the files can temporarily be out of sync. Running `npm install` resolves the new dependency requirements and updates the lockfile and `node_modules` as needed.

---

## 3. node_modules

`node_modules/` contains the actual packages installed locally.

```text
package.json
    ↓
npm install
    ↓
node_modules/
```

The browser does not normally load packages directly from `node_modules`. A build tool such as Vite can read those packages and produce browser-ready output.

---

## 4. Version ranges: exact, ~ and ^

Semantic versions generally follow:

```text
MAJOR.MINOR.PATCH
1.11.13
```

### Exact version

```json
"dayjs": "1.11.13"
```

Requests exactly `1.11.13` rather than allowing a newer matching version through the version range.

### Tilde (~)

```json
"dayjs": "~1.11.13"
```

For a normal non-zero major version, this allows patch updates within the same minor version:

```text
1.11.13  ✅
1.11.14  ✅
1.11.99  ✅
1.12.0   ❌
2.0.0    ❌
```

Effectively:

```text
>= 1.11.13 and < 1.12.0
```

Think:

> **~ = stay within the same minor version.**

### Caret (^)

```json
"dayjs": "^1.11.13"
```

For a normal non-zero major version, this allows compatible updates without changing the major version:

```text
1.11.13  ✅
1.11.14  ✅
1.12.0   ✅
1.99.0   ✅
2.0.0    ❌
```

Effectively:

```text
>= 1.11.13 and < 2.0.0
```

Think:

> **^ = stay within the same major version.**

> Note: npm's semver rules have special behavior for versions beginning with 0, so the simple rules above are primarily for normal versions such as `1.x.y`.

---

## 5. package.json vs package-lock.json

The key distinction:

```text
package.json
→ What dependency/version range does the project want?

package-lock.json
→ What exact dependency tree did npm resolve?

node_modules
→ What is actually installed locally?

dist
→ What the build process produced for deployment?
```

---

## 6. How changing a dependency works

If you manually change:

```json
"dayjs": "^1.11.13"
```

to:

```json
"dayjs": "^1.12.0"
```

the lockfile does **not** magically change at that instant.

The project is temporarily out of sync.

Run:

```bash
npm install
```

and npm will resolve the new requirement, update `package-lock.json`, and update the installed dependency as necessary.

Alternatively, use:

```bash
npm install dayjs@1.12.0
```

which updates the dependency declaration and lockfile for you.

---

## 7. Overall frontend dependency flow

```text
package.json
     │
     │ declares dependencies
     ↓
npm install
     │
     ├──→ package-lock.json  (locks resolved dependency tree)
     │
     └──→ node_modules/      (actual installed packages)
                  │
                  ↓
             Bundler (e.g. Vite)
                  │
                  ↓
                dist/
                  │
                  ↓
               Browser
```

### SDE-2 mental model

- **npm** → dependency management
- **package.json** → project manifest / dependency requirements
- **package-lock.json** → reproducible resolved dependency tree
- **node_modules** → installed packages
- **~** → patch-level updates within the same minor version
- **^** → compatible updates within the same major version (for normal non-zero major versions)
- **Bundler** → processes your source + dependencies into deployable browser assets
- **dist/** → built/deployable frontend output
