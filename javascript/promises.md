# Promises — Chaining & Promise Methods

## What is a Promise?

A Promise represents the eventual result of an asynchronous operation.

A Promise has three states:

```
        pending
       /       \
  fulfilled   rejected
```

Once fulfilled or rejected, it is **settled** and cannot change state again.

---

## Consuming Promises

### `.then()`

Runs when the Promise fulfills and can transform the result.

```js
fetch("/api/users")
    .then(response => response.json())
    .then(users => {
        console.log(users);
    });
```

Each `.then()` receives the value returned by the previous callback.

### `.catch()`

Handles rejection:

```js
fetch("/api/users")
    .then(response => response.json())
    .then(users => console.log(users))
    .catch(error => {
        console.error("Something failed:", error);
    });
```

### `.finally()`

Runs after the Promise settles, regardless of success or failure.

Common use case:

```js
setLoading(true);

fetch("/api/users")
    .then(...)
    .catch(...)
    .finally(() => setLoading(false));
```

---

## Promise Chaining

If a `.then()` callback returns a Promise, the next `.then()` waits for that Promise.

```js
getUser()
    .then(user => {
        return getOrders(user.id);
    })
    .then(orders => {
        return getPayment(orders[0].id);
    })
    .then(payment => {
        console.log(payment);
    })
    .catch(error => {
        console.error(error);
    });
```

### Important gotcha: forgetting `return`

```js
Promise.resolve(10)
    .then(x => {
        Promise.resolve(x * 2); // not returned
    })
    .then(x => {
        console.log(x); // undefined
    });
```

The first callback implicitly returns `undefined`, so the next `.then()` receives `undefined`.

---

## `Promise.all()`

Use when multiple independent async operations are required and **all must succeed**.

```js
const [user, orders, notifications] = await Promise.all([
    getUser(),
    getOrders(),
    getNotifications()
]);
```

The operations can proceed concurrently rather than waiting for one another.

If any Promise rejects, `Promise.all()` rejects.

Important: the returned array preserves **input order**, not completion order.

---

## `Promise.race()`

Returns the result of the first Promise to **settle** — either fulfill or reject.

```js
const result = await Promise.race([
    requestToServer1(),
    requestToServer2()
]);
```

If a rejection settles first, the race rejects.

### Common use case: timeout

```js
await Promise.race([
    fetch("/api"),
    new Promise((_, reject) =>
        setTimeout(() => reject(new Error("Timeout")), 5000)
    )
]);
```

Important: `Promise.race()` does **not** automatically cancel the losing operations.

---

## `Promise.allSettled()`

Use when every operation should finish and you want the result of **each operation**, even if some fail.

```js
const results = await Promise.allSettled([
    getUser(),
    getOrders(),
    getNotifications()
]);
```

Example result:

```js
[
    { status: "fulfilled", value: user },
    { status: "rejected", reason: error },
    { status: "fulfilled", value: notifications }
]
```

Unlike `Promise.all()`, one rejection does not reject the overall result.

The result array preserves input order.

---

## `Promise.any()`

Waits for the first Promise to **fulfill**.

```js
const result = await Promise.any([
    requestToServer1(),
    requestToServer2(),
    requestToServer3()
]);
```

Difference:

```
Promise.race()
    → first to settle
    → fulfillment OR rejection

Promise.any()
    → first to fulfill
    → ignores individual rejections
    → rejects only if ALL reject
```

If all Promises reject, `Promise.any()` rejects with an `AggregateError`.

---

## Other Static Methods

### `Promise.resolve()`

Creates an already-fulfilled Promise:

```js
const p = Promise.resolve(42);

p.then(value => console.log(value));
// 42
```

Useful for normalizing a value into a Promise.

### `Promise.reject()`

Creates an already-rejected Promise:

```js
const p = Promise.reject(new Error("Failed"));

p.catch(error => console.log(error.message));
// Failed
```

---

## SDE-2 Cheat Sheet

| Method | Behavior |
|---|---|
| `.then()` | Handle fulfillment / transform result |
| `.catch()` | Handle rejection |
| `.finally()` | Run after settlement |
| `Promise.all()` | All must fulfill; rejects if one rejects |
| `Promise.race()` | First to **settle** wins |
| `Promise.allSettled()` | Wait for everything; report every result |
| `Promise.any()` | First to **fulfill** wins |
| `Promise.resolve()` | Create/normalize fulfilled Promise |
| `Promise.reject()` | Create rejected Promise |

### Interview essentials

Be able to clearly explain:

- Promise states: pending → fulfilled/rejected
- Promise chaining and why returning a Promise matters
- `Promise.all()` vs `race()` vs `allSettled()` vs `any()`
- `race()` means first to **settle**, while `any()` means first to **fulfill**
- `all()` and `allSettled()` preserve input order
- `race()` does not cancel losing operations
