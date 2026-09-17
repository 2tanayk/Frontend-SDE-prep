# Functions, `this`, Closures & IIFE

## 1. Function declaration vs expression vs arrow

### Function declaration

```js
function add(a, b) {
    return a + b;
}
```

- Function declarations are hoisted with their definition, so they can be called before their declaration.

### Function expression

```js
const add = function(a, b) {
    return a + b;
};
```

- The function is assigned to a variable.
- The variable's initialization rules apply. With `const`, accessing it before initialization is a TDZ error.

### Arrow function

```js
const add = (a, b) => a + b;
```

- Concise function syntax.
- The important SDE-2 distinction is `this`: arrows do **not** have their own `this`.

---

## 2. `this` — regular functions vs arrows

### Regular function: `this` is determined by the call site

```js
const user = {
    name: "Tanay",
    greet: function() {
        console.log(this.name);
    }
};

user.greet(); // Tanay
```

Because `user.greet()` calls the function as a method, `this === user`.

If the method is extracted:

```js
const fn = user.greet;
fn();
```

In strict mode, `this` is `undefined` because it is now a plain function call.

### Arrow function: `this` is lexical

An arrow function does not create its own `this`. It uses the `this` from the surrounding scope where it was created.

```js
const user = {
    name: "Tanay",
    greet: () => {
        console.log(this.name);
    }
};

user.greet();
```

`user.greet()` does **not** make `this === user`, because the arrow ignores the call-site receiver.

### The practical callback example

```js
const user = {
    name: "Tanay",

    greet: function() {
        setTimeout(() => {
            console.log(this.name);
        }, 1000);
    }
};

user.greet(); // Tanay
```

Flow:

```text
user.greet()
    ↓
this inside greet = user
    ↓
arrow callback has no own this
    ↓
arrow captures surrounding this
    ↓
this inside arrow = user
```

If the callback were a regular function instead, it would have its own `this` determined when the callback is called.

### `new`, `call`, `apply`, `bind`

Regular functions can get `this` from different call contexts:

```js
obj.foo();       // this → obj
foo();           // this → undefined in strict mode
foo.call(obj);   // this → obj
new Foo();       // this → newly created object
```

`call`, `apply`, and `bind` can control `this` for regular functions. They cannot replace the lexical `this` of an arrow function.

Arrow functions also cannot be used as constructors with `new`.

### Interview sentence

> Regular functions get `this` dynamically from their call site, while arrow functions do not have their own `this`; they lexically capture `this` from their surrounding scope.

---

## 3. Closures

A closure occurs when a function retains access to variables from its surrounding lexical scope even after the outer function has finished executing.

```js
function outer() {
    let x = 10;

    function inner() {
        console.log(x);
    }

    return inner;
}

const fn = outer();
fn(); // 10
```

Even though `outer()` has returned, `inner` still has access to `x`.

Think of it as:

```text
outer()
 ├── x = 10
 └── inner() ──────→ retains access to x
```

A closure retains access to the **binding/variable**, not simply a frozen copy of its value.

### Practical uses

- Private state / encapsulation
- Callbacks that need access to surrounding variables
- Factories that create functions with their own state

Example:

```js
function createCounter() {
    let count = 0;

    return function() {
        return ++count;
    };
}

const counter = createCounter();

counter(); // 1
counter(); // 2
counter(); // 3
```

`count` cannot be accessed directly from outside, but the returned function retains access to it through the closure.

---

## 4. The `var` closure gotcha in loops

Classic example:

```js
for (var i = 0; i < 3; i++) {
    setTimeout(() => {
        console.log(i);
    }, 1000);
}
```

Output:

```text
3
3
3
```

### Why?

`var` is **function-scoped**, not block-scoped. The loop does not create a separate `i` binding for each iteration.

Conceptually:

```text
ONE shared binding:

i
↓
0 → 1 → 2 → 3

callback 1 ─┐
callback 2 ─┼→ SAME i → 3
callback 3 ─┘
```

Each callback closes over the same `i`. The callbacks run later, after the loop has finished and `i` has become `3`.

### Why `let` works

```js
for (let i = 0; i < 3; i++) {
    setTimeout(() => {
        console.log(i);
    }, 1000);
}
```

Output:

```text
0
1
2
```

`let` is **block-scoped**, and `for` loops have special per-iteration semantics: each iteration gets its own `i` binding.

Conceptually:

```text
iteration 1:
i₁ = 0 → callback 1

iteration 2:
i₂ = 1 → callback 2

iteration 3:
i₃ = 2 → callback 3
```

These are not literally variables named `i₁`, `i₂`, etc.; this is just a mental model for separate bindings.

The callbacks therefore close over different bindings and see `0`, `1`, and `2`.

### Connection to function vs block scope

```js
function test() {
    if (true) {
        var x = 10;
    }

    console.log(x); // 10
}
```

`var` belongs to the surrounding function, so the `if` block does not contain its own `var` scope.

With `let`:

```js
function test() {
    if (true) {
        let x = 10;
    }

    console.log(x); // ReferenceError
}
```

The `if` block owns the `let` binding, so it is inaccessible outside the block.

### Critical mental model

Do **not** think that `let` freezes `i` at a value.

Think:

> Each iteration gets its own `i` binding, and the callback closes over that iteration's binding.

The `var` problem is therefore the combination of:

```text
var → function-scoped → one shared binding
closure → callbacks retain access to that binding
            ↓
all callbacks observe the final value
```

---

## 5. IIFE — Immediately Invoked Function Expression

An IIFE is a function expression that is immediately executed.

```js
(function() {
    const secret = "hidden";
    console.log("runs immediately");
})();
```

Arrow version:

```js
(() => {
    console.log("runs immediately");
})();
```

### Why use it?

Historically, IIFEs were commonly used to create private scope and avoid leaking variables into the global scope, before ES modules and modern block scoping became standard.

```js
(function() {
    const secret = 42;
})();

console.log(secret); // ReferenceError
```

Today, ES modules, `let`/`const`, and normal block scope usually remove the need for IIFEs, but you should recognize the pattern and understand why it existed.

---

## Final mental model

```text
Regular function
→ this comes from HOW it is called

Arrow function
→ no own this; captures surrounding this

Closure
→ function retains access to surrounding lexical bindings

var in loop
→ one shared binding → all callbacks see final value

let in loop
→ separate per-iteration bindings → callbacks see their iteration's value

IIFE
→ function expression that executes immediately; historically useful for private scope
```
