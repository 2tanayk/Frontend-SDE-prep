# React Core Concepts — SDE-2

## 1. JSX

JSX looks like HTML, but it is **not HTML**. It is syntax that is transformed into JavaScript.

Conceptually:

```tsx
<h1>Hello</h1>
```

becomes a React element description through the JSX transformation.

Modern React generally uses the **automatic JSX runtime**, so don't memorize `React.createElement` as the exact modern output. The important idea is:

```
JSX
 ↓
JavaScript transformation
 ↓
React element description
 ↓
React uses it to update the UI
```

JSX can contain JavaScript expressions:

```tsx
const name = "Tanay";

return <h1>Hello {name}</h1>;
```

---

## 2. Virtual DOM

The Virtual DOM is a useful mental model for React's in-memory representation of the UI.

When state or props change, React produces a new element tree and compares it with the previous one.

React does **not** blindly recreate the entire browser DOM.

---

## 3. Reconciliation & Diffing

**Reconciliation** is React's process of determining what changed between renders and what needs to be updated.

**Diffing** refers to comparing the previous and next element trees to determine those changes.

Example:

```
Old:                    New:

div                     div
 ├── h1 "Hello"          ├── h1 "Hello Tanay"
 └── p "Welcome"         └── p "Welcome"
```

React can determine that the `h1` content changed while the surrounding structure can be reused.

### Important distinction

> **Re-render ≠ DOM update.**

A re-render means the component function executes again and produces a new React element tree. Reconciliation then determines what, if anything, needs to change in the actual DOM.

---

## 4. Keys

Keys give React **stable identity for elements in a list**, allowing reconciliation to correctly match items between renders.

```tsx
users.map(user => (
  <User key={user.id} user={user} />
))
```

Good keys are:

- Unique among siblings
- Stable across renders
- Representative of the item's identity

### Why indexes can be problematic

```tsx
users.map((user, index) => (
  <User key={index} user={user} />
))
```

If the list is reordered, inserted into, or deleted from, indexes can refer to different items across renders.

Example:

```
Before: [A, B, C]
After:  [X, A, B, C]
```

Index keys cause the identities to shift:

```
0 → A becomes 0 → X
1 → B becomes 1 → A
2 → C becomes 2 → B
```

This can cause component-local state to become associated with the wrong item.

Use an item's stable ID whenever possible.

Index keys are generally acceptable for genuinely static lists whose ordering and membership never change.

### Avoid random keys

```tsx
key={Math.random()}
```

A new key on every render destroys stable identity and can cause components to be treated as removed and newly mounted.

### Changing a key

Changing a component's key changes its identity. React can treat the old component as unmounted and the new one as mounted, which resets its local state.

### Key is not a prop

```tsx
<User key={user.id} userId={user.id} />
```

`key` is special React metadata and is not available through `props.key`.

---

## 5. Functional vs Class Components

### Functional components

Modern React primarily uses functional components:

```tsx
function User({ name }: { name: string }) {
  return <h1>Hello {name}</h1>;
}
```

### Class components

Class components still exist and should be recognizable:

```tsx
class User extends React.Component {
  render() {
    return <h1>Hello</h1>;
  }
}
```

Know that class components use concepts such as `this.state`, `this.setState()`, and `render()`, but focus primarily on functional components and hooks.

---

## 6. Props

Props are data passed from a parent component to a child.

```tsx
function App() {
  return <User name="Tanay" age={25} />;
}

function User({ name, age }: { name: string; age: number }) {
  return <h1>{name} - {age}</h1>;
}
```

Props are read-only from the child's perspective.

```
Parent
   |
   | props
   ↓
Child
```

---

## 7. Prop Drilling

Prop drilling occurs when data is passed through intermediate components that don't actually need it just to reach a deeply nested component.

Example:

```
App
 ↓
Dashboard
 ↓
UserPanel
 ↓
UserProfile
 ↓
Avatar
```

If only `Avatar` needs `user`, passing it through every layer is prop drilling.

Common solutions include:

- Context
- State management solutions
- Restructuring component boundaries

---

## 8. State with useState

State is data owned by a component and preserved between renders.

```tsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      {count}
    </button>
  );
}
```

Calling the setter schedules a state update and causes the component to render again when the state changes.

### State updates and batching

Multiple updates can be batched.

This:

```tsx
setCount(count + 1);
setCount(count + 1);
setCount(count + 1);
```

uses the same `count` value captured by the current render, so it does not mean "increment three times."

When the next state depends on previous state, use the functional form:

```tsx
setCount(prev => prev + 1);
setCount(prev => prev + 1);
setCount(prev => prev + 1);
```

### State updates don't immediately change the current render's value

```tsx
setCount(count + 1);
console.log(count);
```

The log still sees the `count` from the current render. The new value becomes available in the next render.

### State objects/arrays

Don't mutate state in place.

Bad:

```tsx
user.name = "Rahul";
setUser(user);
```

Prefer a new object/reference:

```tsx
setUser(prev => ({
  ...prev,
  name: "Rahul"
}));
```

For arrays:

```tsx
setUsers(prev => [...prev, newUser]);
```

---

## 9. What Causes a Component to Re-render?

Main cases:

1. Its own state changes.
2. Its parent renders, so the child normally participates in the parent's render/reconciliation.
3. Its props change as part of the parent rendering.
4. A context value it consumes changes.

Important:

> **Re-render does not mean the DOM necessarily changes.**

React reconciles the new result and commits only the necessary DOM changes.

### What does not automatically cause a re-render?

Changing an ordinary JavaScript variable does not.

Changing a ref's `.current` does not.

```tsx
const ref = useRef(0);
ref.current++;
```

This does not trigger a render.

---

## 10. Controlled Components

A controlled input gets its value from React state.

```tsx
function Form() {
  const [name, setName] = useState("");

  return (
    <input
      value={name}
      onChange={e => setName(e.target.value)}
    />
  );
}
```

Flow:

```
User types
   ↓
onChange
   ↓
setName()
   ↓
state changes
   ↓
component renders
   ↓
input value comes from state
```

React is the source of truth.

---

## 11. Uncontrolled Components

An uncontrolled input lets the DOM maintain its current value.

```tsx
function Form() {
  const inputRef = useRef<HTMLInputElement>(null);

  return (
    <>
      <input ref={inputRef} />

      <button onClick={() => {
        console.log(inputRef.current?.value);
      }}>
        Submit
      </button>
    </>
  );
}
```

Mental model:

```
Controlled:
React state → input

Uncontrolled:
DOM → input
```

Uncontrolled inputs can be useful for simple forms, DOM-oriented APIs, and libraries such as React Hook Form.

### Quick comparison

| | Controlled | Uncontrolled |
|---|---|---|
| Source of truth | React state | DOM |
| Value access | State | Ref/DOM |
| Typical pattern | `value` + `onChange` | `ref` |
| Useful for | Interactive/complex forms | Simple or DOM-oriented forms |

---

## SDE-2 Mental Model

```
Props → data from parent
State → data owned by component

State/parent/context update
        ↓
Component render
        ↓
New React element tree
        ↓
Reconciliation / diffing
        ↓
Necessary DOM changes

Keys
→ stable identity during list reconciliation

Controlled input
→ React is source of truth

Uncontrolled input
→ DOM is source of truth
```
