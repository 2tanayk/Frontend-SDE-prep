# Frontend-SDE-prep

JavaScript, TypeScript and React interview revision at practical SDE-2 level.

## JavaScript

### Fundamentals
- [var vs let vs const](javascript/var-let-const.md) — scoping, hoisting, Temporal Dead Zone
- [null vs undefined vs undeclared, equality & type coercion](javascript/null-undefined-coercion.md) — `==` vs `===`, truthy/falsy, implicit vs explicit coercion, `||` vs `??`
- [`typeof` vs `instanceof`](javascript/typeof-vs-instanceof.md) — runtime type checks, prototype-chain checks, arrays, primitives, cross-realm gotcha
- [Functions, `this`, Closures & IIFE](javascript/functions-this-closures-iife.md) — function types, `this` binding, lexical `this`, closures, `var` loop gotcha, block vs function scope, IIFE
- [JavaScript Modules — ESM, Globals & CommonJS](javascript/modules-esm-commonjs.md) — import/export, named vs default, module scope, `window`, classic scripts, React usage, CommonJS vs ESM
- [npm, package.json, package-lock.json & Version Ranges](javascript/npm-package-management.md) — npm dependencies, `node_modules`, lockfiles, exact versions, `~` vs `^`, bundling and `dist`
- [Arrays & Objects](javascript/arrays-objects.md) — array methods, object methods, destructuring, spread/rest, optional chaining and nullish coalescing
- [Async JavaScript — Event Loop, Callbacks & Promises](javascript/async-javascript.md) — call stack, task/microtask queues, callbacks, Promise chaining and Promise methods
- [Browser Storage — localStorage, sessionStorage & Cookies](javascript/storage.md) — persistence, tab/session scope, cookies, HTTP requests and security attributes

## TypeScript

### Fundamentals
- [TypeScript Fundamentals](typescript/typescript-fundamentals.md) — types, inference, type/interface, functions, unions/intersections, narrowing, generics, any/unknown/never, assertions, utility types and API typing

## React

### Fundamentals
- [React Core Concepts](react/core-concepts.md) — JSX, Virtual DOM, reconciliation, keys, components, props, state, re-renders and controlled vs uncontrolled components

### Hooks
- [React Hooks](react/hooks.md) — useState, useEffect, useRef, useMemo, useCallback, useContext, useReducer, Context + Reducer, custom hooks and Rules of Hooks

### State Management
- [React State Management](react/state-management.md) — prop drilling, Context, Context + useReducer, provider value identity, re-render considerations, Redux store/actions/reducers/dispatch/selectors and tradeoffs

### Performance
- [React Performance](react/performance.md) — unnecessary re-renders, React.memo, useMemo vs useCallback, lazy loading, code splitting, keys and performance best practices
