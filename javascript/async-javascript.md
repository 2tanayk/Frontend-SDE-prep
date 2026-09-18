# Async JavaScript — Event Loop, Callbacks & Promises

## 1. JavaScript Is Single-Threaded

JavaScript's main execution model is **single-threaded**:

> One JavaScript thread executes one piece of JavaScript at a time using a single call stack.

However, the **runtime environment** (browser or Node.js) is not necessarily single-threaded. It can perform work such as network I/O, file I/O, timers, rendering, and other background operations outside the JavaScript call stack.

This allows:

```js
fetch("/api");
console.log("Hello");
```

The network operation can be handled by the runtime while JavaScript continues executing.

### SDE-2 answer

> JavaScript's main execution model is single-threaded: one call stack executes JavaScript at a time. The runtime can perform asynchronous work using background capabilities/threads, with the event loop coordinating callbacks/continuations back onto the JavaScript thread.

---

# 2. Event Loop

The **event loop** coordinates synchronous JavaScript execution with asynchronous callbacks by monitoring the call stack and queues and executing queued work when the stack is available.

A simplified execution model is:

```
Current Task
    ↓
Execute synchronous JS
    ↓
Call Stack becomes empty
    ↓
Drain ALL Microtasks
    ↓
Execute next Task
    ↓
Drain ALL Microtasks
    ↓
Execute next Task
    ↓
...
```

### Important rule

> **After a task finishes, the event loop drains the microtask queue before moving to the next task.**

This does **not** mean that tasks are always waiting for some existing microtask. It means that whenever the current task completes, any queued microtasks are processed before the next task is selected.

---

# 3. Task/Macrotask Queue vs Microtask Queue

The important distinction is that an asynchronous operation does **not necessarily put its callback into a queue immediately**.

Instead:

```
Async operation starts
       ↓
JavaScript continues executing
       ↓
Operation becomes ready
       ↓
Its callback/continuation is queued
       ↓
Event loop eventually executes it
```

## Task / Macrotask Queue

Common examples:

- `setTimeout` callback
- `setInterval` callback
- DOM event callbacks such as `click` and `keydown`
- Many browser event callbacks
- Certain Node.js I/O callbacks

Example:

```js
setTimeout(() => {
    console.log("timer");
}, 1000);
```

The timer is **started first**. Once it expires, its callback becomes eligible to enter the task queue.

It does not mean the callback runs exactly after 1000 ms.

> A timer delay is a **minimum delay before the callback becomes eligible**, not a guarantee of execution time.

## Microtask Queue

Common examples:

- Promise `.then()`
- Promise `.catch()`
- Promise `.finally()`
- `await` continuations
- `queueMicrotask()`
- Browser `MutationObserver` callbacks

Example:

```js
Promise.resolve().then(() => {
    console.log("promise");
});
```

The `.then()` callback is scheduled as a microtask.

### Cheat sheet

```
setTimeout / setInterval  → Task
DOM events                → Task

Promise.then/catch/finally → Microtask
await continuation         → Microtask
queueMicrotask             → Microtask
```

---

# 4. Timers Do Not Execute Immediately

Consider:

```js
console.log("A");

setTimeout(() => console.log("B"), 0);

console.log("C");
```

The timer callback does not jump into the call stack immediately.

Roughly:

```
A
↓
start timer
↓
C
↓
current task finishes
↓
timer callback becomes eligible as a Task
↓
B
```

Output:

```
A
C
B
```

Even `setTimeout(..., 0)` means the callback is eligible only after the timer delay and after the current JavaScript work has yielded.

---

# 5. Microtasks Run Before the Next Task

```js
console.log("A");

setTimeout(() => console.log("B"), 0);

Promise.resolve().then(() => console.log("C"));

console.log("D");
```

Execution:

```
A
D
    ↓
current task finishes
    ↓
drain microtasks
    ↓
C
    ↓
take next task
    ↓
B
```

Output:

```
A
D
C
B
```

### Why?

The Promise's `.then()` callback is a microtask, while the timer callback is a task.

The event loop drains microtasks before taking the next task.

---

# 6. Async Operations and Queues

A useful distinction:

> **The async operation itself is not necessarily sitting in the task/microtask queue.**

For example:

```js
fetch("/api").then(() => console.log("done"));
```

Conceptually:

```
fetch()
  ↓
Browser/runtime handles network operation
  ↓
JavaScript continues
  ↓
Response arrives
  ↓
fetch Promise settles
  ↓
.then() continuation → Microtask Queue
```

So do not memorize:

> "Network = microtask."

The **network operation** and the **Promise continuation** are separate things.

---

# 7. Long Network Request vs Short Timer

Consider:

```js
console.log("A");

fetch("/huge-api")
    .then(() => console.log("network done"));

setTimeout(() => console.log("timer"), 1);

console.log("B");
```

If the network request takes 5 seconds:

```
Immediately:
A
B

1 ms:
timer becomes eligible
    ↓
Task Queue
    ↓
timer callback runs
    ↓
timer

5 seconds:
network response arrives
    ↓
Promise settles
    ↓
.then() continuation → Microtask Queue
    ↓
network done
```

Likely output:

```
A
B
timer
network done
```

The long network request does not block the JavaScript thread because the runtime handles the network operation outside the JS call stack.

### Contrast with long-running JavaScript

```js
setTimeout(() => console.log("timer"), 1);

while (true) {
    // long-running JavaScript
}
```

The timer may become eligible after 1 ms, but its callback cannot execute because the JavaScript call stack is still occupied.

> **Asynchronous I/O can happen outside the JS stack; long-running JavaScript cannot be bypassed by the event loop.**

---

# 8. Callbacks and Callback Hell

A callback is a function passed to another function to be executed later.

Example:

```js
getUser(userId, user => {
    getOrders(user.id, orders => {
        getOrder(orders[0].id, order => {
            getPayment(order.id, payment => {
                console.log(payment);
            });
        });
    });
});
```

The problem is not callbacks themselves. The problem is **deeply nested dependent callbacks**.

This creates:

- difficult-to-read code
- difficult error handling
- difficult maintenance
- deeply nested control flow

This pattern is commonly called **callback hell**.

Promises provide a flatter mechanism for representing and composing asynchronous operations.

---

# 9. What Is a Promise?

A **Promise represents the eventual result of an asynchronous operation**.

It has three states:

```
        pending
       /       \
  fulfilled   rejected
```

- **pending** — operation still in progress
- **fulfilled** — operation succeeded
- **rejected** — operation failed

Once fulfilled or rejected, the Promise is **settled** and cannot change to another state.

---

# 10. `.then()`

Runs when the Promise fulfills and can transform the result.

```js
fetch("/api/users")
    .then(response => response.json())
    .then(users => {
        console.log(users);
    });
```

Each `.then()` receives the value returned by the previous callback.

Promise reactions such as `.then()` are scheduled as **microtasks** when the relevant Promise settles.

---

# 11. `.catch()`

Handles rejection:

```js
fetch("/api/users")
    .then(response => response.json())
    .then(users => console.log(users))
    .catch(error => {
        console.error("Something failed:", error);
    });
```

A rejection can propagate down a Promise chain until a suitable `.catch()` handles it.

---

# 12. `.finally()`

Runs after the Promise settles, regardless of whether it fulfilled or rejected.

```js
setLoading(true);

fetch("/api/users")
    .then(...)
    .catch(...)
    .finally(() => setLoading(false));
```

A common use case is cleanup such as stopping a loading indicator.

---

# 13. Promise Chaining

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

Conceptually:

```
getUser()
   ↓
user
   ↓
getOrders(user.id)
   ↓
orders
   ↓
getPayment(...)
   ↓
payment
```

### Important gotcha: forgetting `return`

```js
Promise.resolve(10)
    .then(x => {
        Promise.resolve(x * 2); // NOT returned
    })
    .then(x => {
        console.log(x); // undefined
    });
```

The first callback returns `undefined`, so the next `.then()` receives `undefined`.

---

# 14. `Promise.all()`

Use when multiple independent async operations are required and **all need to succeed**.

```js
const [user, orders, notifications] = await Promise.all([
    getUser(),
    getOrders(),
    getNotifications()
]);
```

The operations can start independently rather than waiting sequentially.

If any Promise rejects, `Promise.all()` rejects.

Important:

> The result array preserves **input order**, not completion order.

```js
const results = await Promise.all([
    slowRequest(),
    fastRequest(),
    mediumRequest()
]);
```

`results[0]` is still the result of `slowRequest()`.

---

# 15. `Promise.race()`

Returns the result of the first Promise to **settle**.

Settled means:

```
fulfilled OR rejected
```

```js
const result = await Promise.race([
    requestToServer1(),
    requestToServer2()
]);
```

If a rejection settles first, the race rejects.

### Timeout example

```js
await Promise.race([
    fetch("/api"),
    new Promise((_, reject) =>
        setTimeout(() => reject(new Error("Timeout")), 5000)
    )
]);
```

Important:

> `Promise.race()` does **not** automatically cancel the losing operations.

---

# 16. `Promise.allSettled()`

Use when every operation should finish and you want the result of **each operation**, including failures.

```js
const results = await Promise.allSettled([
    getUser(),
    getOrders(),
    getNotifications()
]);
```

Example:

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

# 17. `Promise.any()`

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

# 18. `Promise.resolve()`

Creates an already-fulfilled Promise:

```js
const p = Promise.resolve(42);

p.then(value => console.log(value));
// 42
```

Useful for normalizing a value into a Promise.

---

# 19. `Promise.reject()`

Creates an already-rejected Promise:

```js
const p = Promise.reject(new Error("Failed"));

p.catch(error => console.log(error.message));
// Failed
```

---

# 20. `async/await` Connection

An `async` function always returns a Promise.

```js
async function getUser() {
    return user;
}
```

Even though `user` is returned directly, calling `getUser()` produces a Promise.

`await` pauses the **async function's continuation**, not the JavaScript thread.

```js
async function load() {
    const user = await getUser();
    console.log(user);
}

console.log("A");
load();
console.log("B");
```

The JavaScript thread is not blocked while `getUser()` is pending. Once the awaited Promise settles, the async function's continuation is scheduled through Promise/microtask machinery.

### Error handling

A rejected Promise can be handled with `try/catch` around `await`:

```js
async function load() {
    try {
        const user = await getUser();
        console.log(user);
    } catch (error) {
        console.error(error);
    }
}
```

### Practical mental model

> `async/await` is cleaner syntax built on Promise-based asynchronous control flow. It does not make JavaScript multithreaded or block the JS thread while awaiting.

---

# 21. `setTimeout` vs `setInterval`

## `setTimeout`

Schedules a callback once after a minimum delay.

```js
setTimeout(() => {
    console.log("runs once");
}, 1000);
```

The callback becomes eligible after the delay but still waits for the event loop.

## `setInterval`

Schedules timer callbacks repeatedly.

```js
setInterval(() => {
    console.log("runs repeatedly");
}, 1000);
```

The callback does not execute concurrently with another piece of JavaScript on the same JS thread. If the thread is busy, execution is delayed.

For practical purposes, timer callbacks are **tasks/macrotasks**.

---

# 22. Promise Method Cheat Sheet

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

---

# 23. SDE-2 Mental Model

```
JavaScript
    ↓
Single main JS execution thread
    ↓
Call Stack
    ↓
synchronous code executes

Async operation starts
    ↓
Browser / Node runtime handles the operation
    ↓
when ready, callback/continuation is queued

Task Queue
    → timers
    → DOM events
    → many other event callbacks

Microtask Queue
    → Promise reactions
    → async/await continuations
    → queueMicrotask

Event Loop
    → when current task finishes
    → drain ALL microtasks
    → execute next task
    → drain ALL microtasks
    → repeat
```

### Interview definition

> **The event loop is the mechanism that coordinates synchronous JavaScript execution with asynchronous callbacks by monitoring the call stack and queues. After the current task finishes, microtasks are drained before the event loop proceeds to the next task.**

### High-value interview distinctions

- JavaScript's main execution is single-threaded, but the runtime can perform background asynchronous work.
- Starting an async operation does **not** mean its callback is immediately placed into a queue.
- `setTimeout` callbacks are tasks once their timer becomes eligible.
- Promise reactions and `await` continuations are microtasks.
- Microtasks are drained before the next task.
- A long network request does not block JS; long-running JavaScript does.
- `Promise.all()` requires all to fulfill.
- `Promise.race()` is first to **settle**.
- `Promise.any()` is first to **fulfill**.
- `Promise.allSettled()` waits for everything.
- `Promise.all()` and `allSettled()` preserve input order.
- `Promise.race()` does not cancel losing operations.
- `setTimeout(0)` does not mean immediate execution.


# Fetch API & AbortController

## Fetch API

`fetch()` is the browser's built-in Promise-based API for making HTTP requests.

```js
const response = await fetch("/api/users");
const users = await response.json();
```

Important data flow:

```
fetch()
  ↓
Promise<Response>
  ↓
Response object
  ↓
response.json()
  ↓
actual data
```

### Request configuration

```js
const response = await fetch("/api/users", {
    method: "POST",
    headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${token}`
    },
    body: JSON.stringify({
        name: "Tanay"
    })
});
```

Common options:
- `method`
- `headers`
- `body`
- `credentials`
- `signal`

### Response object

Common properties:

```js
response.status
response.ok
response.headers
```

`response.ok` is true for HTTP status codes 200–299.

### Critical error-handling distinction

```
Network failure / abort
    ↓
fetch Promise rejects

HTTP 400 / 401 / 403 / 404 / 500
    ↓
fetch Promise normally fulfills
    ↓
response.ok === false
    ↓
application handles the HTTP error explicitly
```

Therefore:

```js
const response = await fetch("/api/users");

if (!response.ok) {
    throw new Error(`HTTP error: ${response.status}`);
}

const data = await response.json();
```

### Response body methods

Common methods:

```js
response.json()
response.text()
response.blob()
response.arrayBuffer()
```

The response body is generally consumed once. If it needs to be consumed twice, use `response.clone()`.

### POST example

```js
const response = await fetch("/api/users", {
    method: "POST",
    headers: {
        "Content-Type": "application/json"
    },
    body: JSON.stringify({
        name: "Tanay",
        email: "tanay@example.com"
    })
});

if (!response.ok) {
    throw new Error(`Request failed: ${response.status}`);
}

const createdUser = await response.json();
```

---

## AbortController

`AbortController` provides a standard cancellation mechanism for APIs that support an `AbortSignal`. `fetch` supports it.

```js
const controller = new AbortController();

fetch("/api/users", {
    signal: controller.signal
});

// Cancel the request
controller.abort();
```

For fetch, aborting causes the Promise to reject, typically with an `AbortError`.

### Why use it?

Suppose a user rapidly changes a search:

```
"jav"        → Request 1
"java"       → Request 2
"javascript" → Request 3
```

If Request 1 is now obsolete, cancel it:

```js
const controller = new AbortController();

fetch("/api/search?q=java", {
    signal: controller.signal
});

controller.abort();
```

This is useful for cancelling obsolete requests and controlling request lifecycles.

### AbortController + timeout

Manual timeout:

```js
const controller = new AbortController();

const timeoutId = setTimeout(() => {
    controller.abort();
}, 5000);

try {
    const response = await fetch("/api/users", {
        signal: controller.signal
    });

    if (!response.ok) {
        throw new Error(`HTTP ${response.status}`);
    }

    const data = await response.json();
} catch (error) {
    if (error.name === "AbortError") {
        console.log("Request was cancelled");
    } else {
        console.error("Request failed", error);
    }
} finally {
    clearTimeout(timeoutId);
}
```

Modern browsers also support:

```js
const response = await fetch("/api/users", {
    signal: AbortSignal.timeout(5000)
});
```

### Reusable cancellation pattern

```js
async function getUsers(signal) {
    const response = await fetch("/api/users", { signal });

    if (!response.ok) {
        throw new Error(`HTTP ${response.status}`);
    }

    return response.json();
}

const controller = new AbortController();

try {
    const users = await getUsers(controller.signal);
} catch (error) {
    if (error.name === "AbortError") {
        console.log("Request cancelled");
    } else {
        console.error(error);
    }
}
```

The caller controls the request lifecycle while the function simply accepts a signal.

### SDE-2 mental model

```
fetch(url, options)
    ↓
Promise<Response>
    ↓
Response metadata
    → status
    → ok
    → headers
    ↓
response.json()/text()/blob()
    ↓
actual body

AbortController
    ↓
AbortSignal
    ↓
fetch({ signal })
    ↓
controller.abort()
    ↓
fetch rejects with AbortError
```

### Interview definition

> Fetch is a Promise-based Web API for making HTTP requests. It resolves to a Response object when a response is received; HTTP error statuses do not inherently reject the Promise, so applications typically check response.ok or response.status. AbortController provides cancellation by passing its AbortSignal to fetch and calling abort() when the request should be cancelled.

### High-value points

- `fetch()` returns a `Promise<Response>`, not parsed JSON.
- `response.json()` is also asynchronous and returns a Promise.
- HTTP 4xx/5xx responses normally do not reject fetch by themselves.
- Network failures and aborts reject the fetch Promise.
- Response bodies are generally consumed once.
- `AbortController` creates a signal that can be passed to fetch.
- `controller.abort()` cancels an in-flight fetch.
- `AbortSignal.timeout()` provides a convenient timeout signal.
