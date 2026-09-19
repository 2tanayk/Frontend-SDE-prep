# TypeScript Fundamentals — SDE-2

Practical TypeScript fundamentals needed for a React + TypeScript codebase.

## 1. Why TypeScript?

TypeScript adds static type checking to JavaScript.

```ts
function add(a: number, b: number): number {
  return a + b;
}
```

Many mistakes are caught at compile time before the code reaches the browser.

---

## 2. Basic Types & Type Inference

```ts
let name: string = "Tanay";
let age: number = 25;
let active: boolean = true;
```

Types are often inferred:

```ts
let name = "Tanay"; // string
let age = 25;       // number
```

Arrays:

```ts
const names: string[] = ["Tanay", "John"];
const numbers: number[] = [1, 2, 3];
```

---

## 3. Objects

```ts
type User = {
  id: number;
  name: string;
  email: string;
};

const user: User = {
  id: 1,
  name: "Tanay",
  email: "tanay@example.com"
};
```

---

## 4. `type` vs `interface`

Both can describe object shapes:

```ts
type User = {
  id: number;
  name: string;
};

interface User {
  id: number;
  name: string;
}
```

A useful distinction: interfaces support declaration merging, while types are particularly convenient for unions and type composition.

```ts
type Status = "loading" | "success" | "error";
```

For normal application development, both are commonly used.

---

## 5. Optional & Readonly Properties

```ts
type User = {
  id: number;
  name: string;
  phone?: string;
  readonly email: string;
};
```

- `?` means the property may be absent.
- `readonly` prevents reassignment through TypeScript; it is not runtime immutability.

---

## 6. Function Typing

```ts
function add(a: number, b: number): number {
  return a + b;
}

function greet(name?: string): string {
  return `Hello ${name ?? "Guest"}`;
}
```

Function types:

```ts
type Operation = (a: number, b: number) => number;

const add: Operation = (a, b) => a + b;
```

---

## 7. Union & Intersection Types

Union `|` means **OR**:

```ts
let id: string | number;

id = 10;
id = "abc";
```

Literal unions are very useful:

```ts
type Status = "loading" | "success" | "error";
```

Intersection `&` means **AND** — combine multiple types:

```ts
type Person = {
  name: string;
};

type Employee = {
  employeeId: number;
};

type EmployeePerson = Person & Employee;
```

Mental model:

```
| = OR
& = AND
```

---

## 8. Type Narrowing

TypeScript can narrow a union after checking the value.

```ts
function printId(id: string | number) {
  if (typeof id === "string") {
    console.log(id.toUpperCase());
  } else {
    console.log(id.toFixed());
  }
}
```

Common narrowing techniques include:

- `typeof`
- `instanceof`
- `in`
- discriminated unions

### Discriminated union

```ts
type Result =
  | { status: "loading" }
  | { status: "success"; data: User[] }
  | { status: "error"; message: string };

function handleResult(result: Result) {
  if (result.status === "success") {
    console.log(result.data);
  }

  if (result.status === "error") {
    console.log(result.message);
  }
}
```

Once `status` is checked, TypeScript knows the corresponding object shape.

---

## 9. Generics

Generics allow reusable code while preserving type information.

```ts
function identity<T>(value: T): T {
  return value;
}

const name = identity("Tanay"); // string
const age = identity(25);       // number
```

Very common for API responses:

```ts
type ApiResponse<T> = {
  data: T;
  success: boolean;
};

const response: ApiResponse<User[]> = {
  data: users,
  success: true
};
```

---

## 10. `any` vs `unknown`

### `any`

```ts
let value: any = "hello";

value.foo.bar(); // TypeScript allows it
```

`any` effectively tells TypeScript not to check the value.

**Avoid `any` when possible.**

### `unknown`

Use when the type genuinely isn't known yet:

```ts
let value: unknown = "hello";

value.toUpperCase(); // ❌ must narrow first

if (typeof value === "string") {
  value.toUpperCase(); // ✅
}
```

Mental model:

```
any     → "Trust me. Don't check."
unknown → "I don't know yet. Make me prove it."
```

Prefer `unknown` when dealing with genuinely unknown/untrusted values.

---

## 11. `void` & `never`

### `void`

Usually means a function does not return a useful value:

```ts
function logMessage(message: string): void {
  console.log(message);
}
```

### `never`

Means the function/value cannot successfully return or occur.

```ts
function crash(message: string): never {
  throw new Error(message);
}
```

A particularly useful frontend case is exhaustive checking:

```ts
type Status = "loading" | "success" | "error";

function handleStatus(status: Status) {
  switch (status) {
    case "loading":
      return "Loading";
    case "success":
      return "Done";
    case "error":
      return "Failed";
    default:
      const impossible: never = status;
      return impossible;
  }
}
```

If a new status is added but not handled, the `never` assignment can expose the missing case.

Mental model:

```
void   → returns no useful value
never  → should never successfully return / should be impossible
```

---

## 12. Type Assertions

Sometimes you know something TypeScript cannot infer:

```ts
const element =
  document.getElementById("username") as HTMLInputElement;
```

Important:

> `as` does not perform runtime conversion or validation. It only tells TypeScript how to treat the value.

---

## 13. Utility Types

Given:

```ts
type User = {
  id: number;
  name: string;
  email: string;
};
```

### Partial

Makes every property optional:

```ts
type UpdateUser = Partial<User>;
```

Useful for PATCH/update objects.

### Pick

Select specific properties:

```ts
type UserPreview = Pick<User, "id" | "name">;
```

### Omit

Remove properties:

```ts
type CreateUser = Omit<User, "id">;
```

### Required

Makes optional properties required:

```ts
type RequiredUser = Required<User>;
```

### Readonly

Makes properties readonly at the type level:

```ts
type ReadonlyUser = Readonly<User>;
```

### Record

Defines an object/dictionary with a specified key type and value type:

```ts
type UserMap = Record<string, User>;
```

Mental model:

```
Record<K, V>
    ↓   ↓
  keys values
```

Example:

```ts
type Status = "loading" | "success" | "error";

const messages: Record<Status, string> = {
  loading: "Loading...",
  success: "Done!",
  error: "Something went wrong"
};
```

Here every `Status` key must exist and map to a string.

---

## 14. Typing API Responses

Define the expected response shape:

```ts
type User = {
  id: number;
  name: string;
  email: string;
};
```

Then type the function:

```ts
async function getUser(): Promise<User> {
  const response = await fetch("/api/user");
  return response.json();
}
```

For lists:

```ts
async function getUsers(): Promise<User[]> {
  const response = await fetch("/api/users");
  return response.json();
}
```

This gives the rest of the application useful type information.

### Important limitation

TypeScript types disappear at runtime.

If the backend unexpectedly returns:

```json
{ "id": "hello" }
```

declaring `id: number` does not automatically validate the response.

Runtime validation requires a schema-validation approach/library.

---

## SDE-2 Mental Model

```
Basic types + inference
        ↓
type / interface
        ↓
optional / readonly
        ↓
function typing
        ↓
union | intersection &
        ↓
type narrowing
        ↓
generics
        ↓
any vs unknown
        ↓
void / never
        ↓
type assertions
        ↓
utility types
        ↓
API response typing
```

The goal is practical TypeScript fluency for React/frontend development, not advanced TypeScript type-system internals.
