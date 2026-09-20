# React Performance

## 1. What Causes Re-renders?

A component can render again when:

- Its own state changes.
- Its parent renders.
- Its props change as part of the parent render.
- A context value it consumes changes.

**Important:** a re-render does not necessarily mean the DOM changes. React can render, compare the result, and make no DOM update.

### Unstable References

Objects, arrays, and functions created during render get a new reference:

```jsx
const options = { theme: "dark" };
const items = [1, 2, 3];
const handleClick = () => doSomething();
```

Even if their contents are identical, these are new references on every render.

This matters when they are passed to memoized children or used as hook dependencies.

**Inline functions are not inherently bad.** The issue is whether function identity actually matters.

---

## 2. React.memo

`React.memo` memoizes a component's rendered result based on its props.

```jsx
const UserCard = React.memo(function UserCard({ user }) {
  return <div>{user.name}</div>;
});
```

If the parent renders again and the props are unchanged by React's prop comparison, React can skip rendering `UserCard`.

### When it helps

Use it when:

- The child is relatively expensive to render.
- The parent renders frequently.
- The child's props often remain unchanged.

### When it is pointless

It usually provides little value when:

- The component is tiny/cheap.
- The component rarely re-renders.
- Its props change almost every time anyway.

A memoized child also won't help much if the parent keeps creating new object/function props:

```jsx
<UserCard user={{ name: "Tanay" }} />
```

The object is new on every render.

Also remember: `React.memo` does **not** prevent renders caused by the component's own state or consumed context.

---

## 3. React.memo vs useMemo vs useCallback

These solve different problems:

| Tool | Memoizes | Main purpose |
|---|---|---|
| `React.memo` | Component rendering | Skip child re-render when props are unchanged |
| `useMemo` | A computed value | Avoid expensive recalculation / preserve value reference |
| `useCallback` | A function reference | Preserve function identity |

### useMemo

```jsx
const filteredUsers = useMemo(
  () => users.filter(u => u.active),
  [users]
);
```

The calculation is reused until a dependency changes.

Use it for genuinely expensive calculations or when a stable derived value/reference is useful.

### useCallback

```jsx
const handleSelect = useCallback(
  (id) => selectUser(id),
  [selectUser]
);
```

The function reference is reused until a dependency changes.

It is particularly useful when passing a callback to a memoized child or when function identity matters for a hook dependency.

### Classic combination

```jsx
const Child = React.memo(function Child({ data, onSelect }) {
  // ...
});

function Parent({ items }) {
  const data = useMemo(() => buildData(items), [items]);

  const onSelect = useCallback(
    (id) => selectItem(id),
    []
  );

  return <Child data={data} onSelect={onSelect} />;
}
```

Here:

- `React.memo` lets the child bail out.
- `useMemo` stabilizes the object/value passed to it.
- `useCallback` stabilizes the function passed to it.

Don't add all three automatically. Each should solve a real problem.

---

## 4. Lazy Loading & Code Splitting

Large applications should not necessarily send every piece of JavaScript to the browser upfront.

`React.lazy` lets a component be loaded dynamically:

```jsx
const AdminPage = React.lazy(() => import("./AdminPage"));
```

Because `import()` is dynamic, bundlers can create a separate chunk for that code.

Use it with `Suspense`:

```jsx
<Suspense fallback={<div>Loading...</div>}>
  <AdminPage />
</Suspense>
```

### Good candidates

- Route-level pages.
- Large features.
- Features that are rarely used.
- Admin sections or other conditional functionality.

### Best practice

Don't code-split every tiny component. Too many small chunks can add unnecessary loading overhead and complexity.

---

## 5. Keys in Lists

Keys tell React which list item is which across renders.

```jsx
users.map(user => (
  <UserRow key={user.id} user={user} />
));
```

A good key is:

- Stable.
- Unique among siblings.
- Tied to the item's identity.

### Why keys matter

Consider:

```text
Before: A  B  C
After:  X  A  B  C
```

With stable IDs, React knows that X is new and A/B/C are the same existing items.

With index keys, React can associate the wrong existing component instance with the wrong data.

This becomes especially problematic when list items have local state, inputs, focus, animations, or other stateful behavior.

### Avoid

```jsx
key={Math.random()}
```

A random key changes every render, so React treats items as new and can remount them.

Also avoid index keys when the list can be reordered, inserted into, deleted from, or filtered.

Index keys can be acceptable for a truly static list whose order and membership never change.

---

## 6. Performance Best Practices

1. **Don't optimize every re-render.** Rendering is normal React behavior.
2. **Measure before optimizing.** Find the expensive component or calculation first.
3. Keep state as local as reasonably possible; unnecessary state lifting can cause broader re-renders.
4. Be aware that objects, arrays, and functions created during render have new references.
5. Don't eliminate inline functions just because they are inline.
6. Use `React.memo` when an expensive child frequently receives unchanged props.
7. Use `useMemo` for genuinely expensive calculations or when stable value identity is useful.
8. Use `useCallback` when function identity matters, especially with memoized children or hook dependencies.
9. Lazy-load routes and large/infrequently used features.
10. Use stable domain IDs for dynamic lists.
11. Avoid `Math.random()` for keys.
12. Avoid index keys when list identity/order can change.

---

## SDE-2 Mental Model

```text
Unnecessary render
        ↓
Find why
        ↓
Unstable props?
   ├── object/array → useMemo if actually useful
   └── function     → useCallback if identity matters

Expensive child?
        ↓
React.memo

Large/infrequent feature?
        ↓
React.lazy + Suspense
        ↓
Code splitting

Dynamic list?
        ↓
Stable keys
```

The core principle is **not "prevent all re-renders."**

It is:

> Identify unnecessary work, understand why it happens, optimize only where it matters, and verify the improvement.
