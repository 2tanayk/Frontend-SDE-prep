# Arrays & Objects

## Array methods

### `map()` — transform every element

Use `map` when you want one array transformed into another array.

```js
const numbers = [1, 2, 3, 4];

const doubled = numbers.map(n => n * 2);
// [2, 4, 6, 8]
```

Think:

> **map = transform each element**

The result normally has the same length as the original array.

---

### `filter()` — keep matching elements

Use `filter` when you want a subset of the original array.

```js
const numbers = [1, 2, 3, 4, 5];

const even = numbers.filter(n => n % 2 === 0);
// [2, 4]
```

Think:

> **filter = which elements should I keep?**

The resulting array can have a different length.

---

### `reduce()` — accumulate into one result

`reduce` iterates over an array while maintaining an accumulator.

```js
const numbers = [1, 2, 3, 4];

const sum = numbers.reduce(
    (acc, n) => acc + n,
    0
);
// 10
```

Conceptually:

```text
acc = 0
0 + 1 → 1
1 + 2 → 3
3 + 3 → 6
6 + 4 → 10
```

The result does not have to be a number. You can build an object, array, map, etc.

Think:

> **reduce = iterate while maintaining an accumulator**

---

### `find()` — first matching element

```js
const users = [
    { id: 1, name: "Tanay" },
    { id: 2, name: "Rahul" }
];

const user = users.find(user => user.id === 2);
// { id: 2, name: "Rahul" }
```

Returns the first matching element, or `undefined` if nothing matches.

> **find = give me the first matching element**

---

### `findIndex()` — index of first match

```js
const numbers = [10, 20, 30, 40];

numbers.findIndex(n => n === 30);
// 2
```

Returns `-1` if nothing matches.

```text
find       → element / undefined
findIndex  → index / -1
```

---

### `some()` — at least one?

```js
[1, 3, 5, 8].some(n => n % 2 === 0);
// true
```

Returns a boolean.

> **some = does at least one satisfy the condition?**

---

### `every()` — all?

```js
[2, 4, 6, 8].every(n => n % 2 === 0);
// true
```

Returns a boolean.

> **every = do all satisfy the condition?**

Easy distinction:

```text
some   → at least ONE?
every  → ALL?
```

---

### `flat()` — flatten nested arrays

```js
const arr = [1, [2, 3], [4, 5]];

arr.flat();
// [1, 2, 3, 4, 5]
```

By default, `flat()` flattens one level.

```js
[1, [2, [3, 4]]].flat();
// [1, 2, [3, 4]]
```

Specify depth when needed:

```js
[1, [2, [3, 4]]].flat(2);
// [1, 2, 3, 4]
```

---

### `flatMap()` — map + flatten one level

Without `flatMap`:

```js
const result = [1, 2, 3]
    .map(n => [n, n * 2])
    .flat();

// [1, 2, 2, 4, 3, 6]
```

With `flatMap`:

```js
const result = [1, 2, 3]
    .flatMap(n => [n, n * 2]);

// [1, 2, 2, 4, 3, 6]
```

Think:

> **flatMap = map + flat(1)**

---

## Object methods

### `Object.keys()`

Returns an array of the object's property names.

```js
const user = {
    name: "Tanay",
    age: 25,
    role: "USER"
};

Object.keys(user);
// ["name", "age", "role"]
```

> **keys = property names**

---

### `Object.values()`

Returns an array of the object's property values.

```js
Object.values(user);
// ["Tanay", 25, "USER"]
```

Useful with array methods:

```js
const prices = {
    apple: 100,
    banana: 50,
    mango: 150
};

const total = Object.values(prices)
    .reduce((sum, price) => sum + price, 0);

// 300
```

> **values = property values**

---

### `Object.entries()`

Returns an array of `[key, value]` pairs.

```js
Object.entries(user);
// [
//   ["name", "Tanay"],
//   ["age", 25],
//   ["role", "USER"]
// ]
```

Common iteration pattern:

```js
Object.entries(user).forEach(([key, value]) => {
    console.log(key, value);
});
```

> **entries = key-value pairs**

---

### `Object.assign()`

Copies properties from source objects into a target object.

```js
const user = { name: "Tanay" };
const details = { age: 25, role: "USER" };

const result = Object.assign({}, user, details);
// { name: "Tanay", age: 25, role: "USER" }
```

Later sources overwrite conflicting properties:

```js
Object.assign(
    {},
    { age: 25 },
    { age: 30 }
);
// { age: 30 }
```

Important: `Object.assign` writes into the target object.

---

### Object spread

Modern code commonly uses object spread for simple copying and merging:

```js
const copy = { ...user };

const updatedUser = {
    ...user,
    age: 26
};
```

Later properties override earlier ones.

```js
const result = {
    ...a,
    ...b
};
```

If both contain the same property, `b` wins.

Unlike `Object.assign`, spread creates a new object rather than mutating an explicitly supplied target.

Both object spread and `Object.assign` are **shallow** copies.

---

## Destructuring

Destructuring extracts values from arrays or objects directly into variables.

### Array destructuring

```js
const numbers = [10, 20, 30];

const [first, second, third] = numbers;
// first = 10
// second = 20
// third = 30
```

Positions matter.

You can skip elements:

```js
const [first, , third] = [10, 20, 30];
// first = 10
// third = 30
```

### Array default values

```js
const [a, b = 20] = [10];
// a = 10
// b = 20
```

Defaults apply when the value is `undefined`, not `null`.

---

### Object destructuring

Property names matter:

```js
const user = {
    name: "Tanay",
    age: 25,
    role: "USER"
};

const { name, age, role } = user;
```

Equivalent conceptually to reading:

```text
name → user.name
age  → user.age
role → user.role
```

### Renaming

```js
const { name: userName, age: userAge } = user;
```

This means:

```text
user.name → variable userName
user.age  → variable userAge
```

### Defaults + renaming

```js
const {
    name: userName,
    age: userAge = 25
} = user;
```

The default is used only when `user.age` is `undefined`.

### Function parameter destructuring

```js
function printUser({ name, age }) {
    console.log(name);
    console.log(age);
}

printUser({
    name: "Tanay",
    age: 25
});
```

This is common in application code.

---

## Spread vs rest — `...`

The syntax is the same, but the job is different.

> **Spread = unpack**
>
> **Rest = collect**

### Spread

Unpacks the contents of an array/object into another expression.

```js
const numbers = [1, 2, 3];
const copy = [...numbers];

const a = [1, 2];
const b = [3, 4];
const result = [...a, ...b];
// [1, 2, 3, 4]
```

For objects:

```js
const updatedUser = {
    ...user,
    age: 26
};
```

### Rest

Collects the remaining values.

Array destructuring:

```js
const [first, ...rest] = [10, 20, 30, 40];

// first = 10
// rest = [20, 30, 40]
```

Object destructuring:

```js
const { name, ...otherDetails } = user;

// otherDetails = { age: 25, role: "USER" }
```

Function parameters:

```js
function sum(...numbers) {
    return numbers.reduce((total, n) => total + n, 0);
}

sum(10, 20, 30);
// 60
```

Rest must be the last parameter.

### Mental model

```text
... → unpacking → SPREAD
... → collecting remaining things → REST
```

---

## Optional chaining — `?.`

Optional chaining safely accesses a property or calls a method when something might be `null` or `undefined`.

Without it:

```js
const user = null;

user.address.city;
// TypeError
```

With it:

```js
user?.address?.city;
// undefined
```

If the chain hits `null` or `undefined`, evaluation stops and returns `undefined`.

It can also be used for optional method calls:

```js
user?.getName?.();
```

Think:

> **?. = safely access/call something**

---

## Nullish coalescing — `??`

Provides a fallback only when the left side is `null` or `undefined`.

```js
const name = null;

const result = name ?? "Unknown";
// "Unknown"
```

The key difference from `||`:

```js
0 ?? 10;
// 0

0 || 10;
// 10
```

`||` uses the fallback for any falsy value, while `??` only does so for `null` or `undefined`.

Examples:

```js
false ?? true;       // false
0 ?? 100;            // 0
"" ?? "default";     // ""
null ?? "default";   // "default"
undefined ?? "default"; // "default"
```

A very common combination:

```js
const city = user?.address?.city ?? "Unknown";
```

Read it as:

> Safely get `user.address.city`; if the result is `null` or `undefined`, use `"Unknown"`.

---

## SDE-2 cheat sheet

```text
map       → transform every element
filter    → keep matching elements
reduce    → accumulate into one result
find      → first matching element / undefined
findIndex → first matching index / -1
some      → at least one?
every     → all?
flat      → flatten nested arrays
flatMap   → map + flatten one level

Object.keys()    → array of keys
Object.values()  → array of values
Object.entries() → array of [key, value] pairs
Object.assign()  → copy/merge into target
{ ...obj }       → shallow-copy object

Array destructuring  → position matters
Object destructuring → property name matters
... spread           → unpack
... rest             → collect remaining

?. → safely access/call through null/undefined
?? → fallback only for null/undefined
|| → fallback for any falsy value
```
