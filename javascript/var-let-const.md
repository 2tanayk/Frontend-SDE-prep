# var vs let vs const

## Overview

JavaScript provides three variable declaration keywords: `var`, `let`, and `const`.

The key differences are around **scope**, **hoisting**, **Temporal Dead Zone (TDZ)**, reassignment, and redeclaration.

## Quick Comparison

| | `var` | `let` | `const` |
|---|---|---|---|
| Scope | Function | Block | Block |
| Reassignment | Yes | Yes | No |
| Redeclaration in same scope | Yes | No | No |
| Hoisted | Yes | Yes | Yes |
| Accessible before declaration | `undefined` | ReferenceError (TDZ) | ReferenceError (TDZ) |

## 1. Scoping

### `var` — function scoped

`var` does not respect block scope. It is scoped to the nearest function.

```javascript
function test() {
    if (true) {
        var x = 10;
    }

    console.log(x); // 10
}
```

The `if` block does not create a separate scope for `var`.

### `let` and `const` — block scoped

A block is code enclosed by `{}`. `let` and `const` are scoped to that block.

```javascript
if (true) {
    let x = 10;
    const y = 20;
}

console.log(x); // ReferenceError
console.log(y); // ReferenceError
```

## 2. The `var` Loop Gotcha

```javascript
for (var i = 0; i < 3; i++) {
    setTimeout(() => console.log(i), 100);
}
```

Output:

```text
3
3
3
```

There is one function-scoped `i`. By the time the callbacks execute, the loop has completed and `i` is `3`.

With `let`:

```javascript
for (let i = 0; i < 3; i++) {
    setTimeout(() => console.log(i), 100);
}
```

Output:

```text
0
1
2
```

`let` provides a separate binding for each loop iteration.

## 3. Hoisting

### `var`

The declaration is hoisted and initialized to `undefined` before execution.

```javascript
console.log(x); // undefined
var x = 10;
```

Conceptually:

```javascript
var x;
console.log(x);
x = 10;
```

Important: **the declaration is hoisted, not the assignment/initialization.**

### `let` and `const`

`let` and `const` are also hoisted, but their bindings are not initialized before their declaration is evaluated.

```javascript
console.log(x); // ReferenceError
let x = 10;
```

So saying "`let` and `const` are not hoisted" is technically inaccurate. They are hoisted but remain inaccessible during the Temporal Dead Zone.

## 4. Temporal Dead Zone (TDZ)

The TDZ is the period between entering a variable's scope and the point where its `let`/`const` declaration is initialized.

```javascript
{
    // TDZ starts
    console.log(x); // ReferenceError

    let x = 10;
    // TDZ ends
}
```

Mental model:

```text
Scope begins
    |
    |  TDZ — x exists as a binding but cannot be accessed
    |
let x = 10
    |
    v
TDZ ends — x can now be accessed
```

## 5. `const` Does Not Mean Immutable

`const` prevents reassignment of the binding. It does not make an object immutable.

```javascript
const user = {
    name: "Tanay"
};

user.name = "John"; // Allowed

user = {}; // TypeError
```

The variable cannot be made to point to a different object, but the existing object can still be mutated.

## 6. Redeclaration

`var` permits redeclaration in the same scope:

```javascript
var x = 10;
var x = 20;
```

`let` and `const` do not:

```javascript
let x = 10;
let x = 20; // SyntaxError
```

```javascript
const x = 10;
const x = 20; // SyntaxError
```

## 7. Shadowing

A variable in an inner block can shadow an outer variable.

```javascript
var x = 10;

{
    let x = 20;
    console.log(x); // 20
}

console.log(x); // 10
```

The inner `let x` is a different binding.

With `var`, a block does not create a separate scope:

```javascript
var x = 10;

{
    var x = 20;
}

console.log(x); // 20
```

## Practical Rule

Prefer:

```text
const → default choice
let   → when reassignment is required
var   → generally avoid in modern JavaScript
```

## SDE-2 Mental Model

```text
                 var             let              const
                 |               |                 |
Scope        function         block             block
                 |               |                 |
Hoisted?        yes             yes               yes
                 |               |                 |
Before decl.  undefined       TDZ error         TDZ error
                 |               |                 |
Reassign       yes             yes                no
                 |               |                 |
Redeclare      yes             no                 no
```

### Key Takeaway

**Hoisting does not mean safe accessibility.** `let` and `const` are hoisted, but their bindings remain in the Temporal Dead Zone until initialization.
