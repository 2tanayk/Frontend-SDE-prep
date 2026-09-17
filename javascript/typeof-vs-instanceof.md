# `typeof` vs `instanceof`

## Overview

They answer different questions:

- **`typeof`** → What broad runtime type is this value?
- **`instanceof`** → Does this object's prototype chain contain this constructor's prototype?

---

## 1. `typeof`

Syntax:

```js
typeof value
```

It returns a string describing the value's type.

```js
typeof 42          // "number"
typeof "hello"     // "string"
typeof true        // "boolean"
typeof undefined    // "undefined"
typeof 123n         // "bigint"
typeof Symbol()     // "symbol"
typeof function() {} // "function"
```

For objects:

```js
typeof {}          // "object"
typeof []          // "object"
typeof new Date()  // "object"
```

### The `null` quirk

```js
typeof null // "object"
```

This is a historical JavaScript quirk. `null` is not actually an object.

Do not explain this as "null is an object"; the behavior is simply part of JavaScript's legacy semantics.

---

## 2. `typeof` and Undeclared Variables

`typeof` can safely check an identifier that was never declared:

```js
typeof someUnknownVariable // "undefined"
```

Direct access would throw:

```js
someUnknownVariable
// ReferenceError
```

This is why code may use:

```js
if (typeof window !== "undefined") {
    // browser-specific code
}
```

---

## 3. The Array Problem

`typeof` cannot distinguish an array from a normal object:

```js
const users = [];

typeof users // "object"
```

Use:

```js
Array.isArray(users) // true
```

`Array.isArray()` is the appropriate array check.

---

## 4. `instanceof`

Syntax:

```js
value instanceof Constructor
```

It checks whether the object's **prototype chain contains `Constructor.prototype`**.

```js
const user = new User();

user instanceof User // true
```

Other examples:

```js
const arr = [];
arr instanceof Array // true

const date = new Date();
date instanceof Date // true
```

---

## 5. How `instanceof` Works

Given:

```js
const user = new User();
user instanceof User
```

Conceptually, JavaScript checks the prototype chain:

```text
user
 ↓
user's [[Prototype]]
 ↓
User.prototype
 ↓
match → true
```

More generally:

```text
object
  ↓
prototype
  ↓
prototype
  ↓
...
  ↓
Constructor.prototype ?
```

If it finds a match → `true`.

If it reaches the end of the prototype chain → `false`.

This explains:

```js
[] instanceof Object // true
```

because the chain is conceptually:

```text
[]
 ↓
Array.prototype
 ↓
Object.prototype
```

---

## 6. `typeof` vs `instanceof`

```js
const arr = [];

typeof arr
// "object"

arr instanceof Array
// true
```

So:

### `typeof`

Answers:

> What broad runtime type is this?

Good for primitive type checks.

### `instanceof`

Answers:

> Does this object's prototype chain contain this constructor's prototype?

Good for class/constructor instance checks.

---

## 7. `instanceof` and Primitive Values

`instanceof` is about objects and prototype chains, not primitive values.

```js
"hello" instanceof String // false
42 instanceof Number      // false
```

But wrapper objects behave differently:

```js
const str = new String("hello");
str instanceof String // true

const num = new Number(42);
num instanceof Number // true
```

In normal code, avoid creating wrapper objects with `new String()`, `new Number()`, etc.

---

## 8. Cross-Realm Gotcha

Objects from another JavaScript realm, such as an iframe, can have different constructors/prototypes.

Therefore an array from another realm can produce:

```js
otherWindowArray instanceof Array // false
```

For array detection, prefer:

```js
Array.isArray(value)
```

---

## 9. Practical Usage

### Check a primitive type

```js
if (typeof value === "string") {
    // ...
}
```

### Check for an array

```js
Array.isArray(value)
```

### Check a custom class instance

```js
if (user instanceof User) {
    // ...
}
```

---

## Quick Comparison

| Value | `typeof` | `instanceof` |
|---|---|---|
| `42` | `"number"` | `false` for `Number` |
| `"hello"` | `"string"` | `false` for `String` |
| `true` | `"boolean"` | Not useful for primitive check |
| `undefined` | `"undefined"` | Not useful |
| `null` | `"object"` | `false` |
| `{}` | `"object"` | `true` for `Object` |
| `[]` | `"object"` | `true` for `Array`, `Object` |
| `new Date()` | `"object"` | `true` for `Date`, `Object` |
| `function(){}` | `"function"` | `true` for `Function`, `Object` |

## SDE-2 Mental Model

```text
typeof
  ↓
Broad type classification
  ↓
Good for primitives
  ↓
Quirk: typeof null === "object"

instanceof
  ↓
Prototype-chain check
  ↓
Good for class/constructor instances
  ↓
Not suitable for primitive type checks
```

## Key Takeaway

> `typeof` performs broad runtime type classification, while `instanceof` checks whether a constructor's prototype appears in an object's prototype chain.

One important interview sentence:

> **`instanceof` does not simply check an object's constructor name; it checks whether `Constructor.prototype` exists somewhere in the object's prototype chain.**
