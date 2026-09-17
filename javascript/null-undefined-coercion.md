# null vs undefined vs undeclared, Equality & Type Coercion

## Overview

These concepts are closely related because JavaScript frequently converts values between types, especially with `==`, conditionals, and operators.

## 1. `undefined`

A variable exists, but currently has no assigned value.

```js
let x;
console.log(x); // undefined
```

JavaScript also produces `undefined` for things such as:

```js
function test() {}
test(); // undefined

const user = {};
user.name; // undefined
```

**Mental model:** the value exists, but no value is currently available.

---

## 2. `null`

`null` is an explicitly assigned value meaning "there is intentionally no value."

```js
let user = null;
```

Compare:

```js
let a;
let b = null;

console.log(a); // undefined
console.log(b); // null
```

**Mental model:** `undefined` usually represents missing/unassigned; `null` explicitly represents no value.

---

## 3. Undeclared

An undeclared identifier does not exist.

```js
console.log(foo);
// ReferenceError: foo is not defined
```

This is different from:

```js
let foo;
console.log(foo); // undefined
```

Here `foo` is declared and its value is `undefined`.

### `typeof` gotcha

```js
typeof foo; // "undefined"
```

`typeof` can safely return `"undefined"` for an undeclared identifier, whereas directly accessing it throws `ReferenceError`.

---

## 4. `==` vs `===`

### `===` — strict equality

No type coercion is performed for the equality comparison. Both type and value must match.

```js
5 === 5           // true
5 === "5"         // false
true === 1        // false
null === undefined // false
```

### `==` — loose equality

JavaScript may coerce operands before comparing them according to its abstract equality rules.

```js
5 == "5"           // true
true == 1           // true
false == 0          // true
null == undefined  // true
```

Common examples:

```js
0 == false        // true
"" == false      // true
"0" == false     // true
```

### `null` and `undefined`

There is a special loose-equality relationship:

```js
null == undefined   // true
null === undefined  // false
```

But:

```js
null == 0       // false
null == false   // false
null == ""      // false
```

### Practical rule

Prefer `===` in production code. If you intentionally need to handle both `null` and `undefined`, `x == null` is a valid deliberate pattern, but it should be used knowingly.

---

## 5. Truthy and Falsy

When JavaScript expects a boolean, it can convert a value to a boolean.

```js
if (value) {
    // value is truthy
}
```

### Falsy values

The falsy values to memorize are:

```text
false
0
-0
0n
""
null
undefined
NaN
```

Everything else is truthy.

Especially important:

```js
"0"       // truthy
"false"   // truthy
[]         // truthy
{}         // truthy
```

You can explicitly check boolean conversion:

```js
Boolean(0)         // false
Boolean("")        // false
Boolean(null)      // false
Boolean(undefined) // false
Boolean(NaN)       // false

Boolean("0")       // true
Boolean([])        // true
Boolean({})        // true
```

**Mental model:** JavaScript defines a small set of falsy values; everything else is truthy.

---

## 6. Type Coercion

Type coercion means converting a value from one type to another.

### Explicit coercion

The developer explicitly asks for conversion:

```js
Number("123")  // 123
String(123)     // "123"
Boolean(1)      // true
```

This is easier to reason about because the conversion is visible.

### Implicit coercion

JavaScript performs conversion automatically because an operation requires it.

```js
"5" + 2 // "52"
"5" - 2 // 3
"5" * 2 // 10
"5" / 2 // 2.5
```

`+` is particularly important because it can perform string concatenation when a string is involved.

The other arithmetic operators convert values to numbers.

---

## 7. Where Implicit Coercion Bites

### The `+` trap

```js
"10" + 5 // "105"
"10" - 5 // 5
```

Prefer explicit conversion when numeric input is expected:

```js
const result = Number(input) + 5;
```

### Truthiness trap

```js
const page = 0;

if (page) {
    // does not run because 0 is falsy
}
```

Truthiness is useful, but remember that legitimate values such as `0`, `""`, and `false` can be falsy.

---

## 8. `||` vs `??`

This is an important practical consequence of truthiness.

```js
const count = 0;

count || 10 // 10
count ?? 10 // 0
```

`||` uses truthiness, so it falls back for any falsy value.

`??` only falls back when the value is `null` or `undefined`.

```js
0 ?? 10          // 0
"" ?? "default" // ""
false ?? true    // false
null ?? 10       // 10
undefined ?? 10  // 10
```

Use `??` when you mean:

> "Use the fallback only when the value is missing (`null`/`undefined`)."

---

## 9. Why `==` Can Get Weird

Loose equality can trigger multiple coercions.

```js
[] == false // true
```

Conceptually:

```text
[] → "" → 0
false → 0

0 == 0 → true
```

Another example:

```js
[1] == 1 // true
```

Conceptually:

```text
[1] → "1" → 1
```

These rules are why `==` can be difficult to reason about in real code.

---

## SDE-2 Mental Model

```text
UNDECLARED
    ↓
Identifier doesn't exist
    ↓
ReferenceError

undefined
    ↓
Value exists, but is missing/unassigned

null
    ↓
Value exists and explicitly means "no value"

TRUTHINESS
    ↓
Boolean(value)
    ↓
Only a small set is falsy

==
    ↓
May coerce types before comparison

===
    ↓
No equality coercion
    ↓
Type + value must match
```

## Key Takeaways

- `undefined` is a value; an undeclared identifier is not.
- `null` explicitly represents the absence of a value.
- Prefer `===` over `==` unless loose equality is intentional.
- Memorize the falsy values; objects and non-empty strings are truthy.
- Prefer explicit conversion when the type matters.
- `+` can concatenate strings, while `-`, `*`, and `/` perform numeric coercion.
- `||` falls back on any falsy value; `??` falls back only for `null`/`undefined`.
