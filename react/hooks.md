# React Hooks — SDE-2

## 1. useState

`useState` stores state that persists between renders.

```tsx
const [count, setCount] = useState(0);
```

Calling the setter schedules a state update and causes a re-render when the state changes.

### Functional updates

When the next state depends on the previous state, use the functional form:

```tsx
setCount(prev => prev + 1);
setCount(prev => prev + 1);
setCount(prev => prev + 1);
```

This avoids relying on the stale value captured by the current render.

### State is a render snapshot

Calling `setCount` does not immediately change the `count` variable in the current render. The new value is available in the next render.

### State updates are batched

React may batch multiple state updates together, so don't assume every setter call causes an immediate separate render.

### Don't mutate state

Create a new object/array reference:

```tsx
setUser(prev => ({ ...prev, name: "Tanay" }));
setUsers(prev => [...prev, newUser]);
```

### Lazy initialization

For expensive initial computation:

```tsx
const [data, setData] = useState(() => expensiveCalculation());
```

The initializer function is used to calculate the initial state rather than running the expensive calculation on every render.

---

## 2. useEffect

`useEffect` is primarily used to **synchronize a component with an external system** such as an API, subscription, timer, browser API, or third-party library.

```tsx
useEffect(() => {
  // synchronize with something external
}, [dependency]);
```

### Dependency behavior

**No dependency array**

```tsx
useEffect(() => {
  // runs after every render
});
```

**Empty dependency array**

```tsx
useEffect(() => {
  // runs after the initial mount
}, []);
```

In development with Strict Mode, React may perform an extra setup/cleanup cycle to expose effect bugs.

**Dependencies**

```tsx
useEffect(() => {
  // initial run + when userId changes
}, [userId]);
```

React compares dependencies using `Object.is`.

### Cleanup

Cleanup runs before the effect runs again because dependencies changed, and when the component unmounts.

```tsx
useEffect(() => {
  const id = setInterval(refresh, 5000);

  return () => clearInterval(id);
}, []);
```

### Common mistakes

- Missing dependencies
- Infinite effect loops caused by updating a dependency from inside the effect
- Using an effect for values that can simply be derived during render
- Forgetting cleanup for subscriptions/timers/listeners
- Stale closures
- Race conditions in async effects

Mental model:

> Effects are for synchronization with things outside React, not a general-purpose place to put component logic.

---

## 3. useRef

`useRef` stores a mutable value that persists across renders **without causing a re-render when it changes**.

```tsx
const ref = useRef(0);

ref.current++;
```

Changing `ref.current` does not trigger rendering.

### DOM access

```tsx
const inputRef = useRef<HTMLInputElement>(null);

<input ref={inputRef} />

inputRef.current?.focus();
```

### Common use cases

- Accessing DOM elements
- Storing a timer ID
- Keeping a previous value
- Holding mutable values that do not belong in the UI

Mental model:

> If the value must update the UI → state.  
> If the value must persist but changing it should not render → ref.

---

## 4. useMemo

`useMemo` memoizes a **computed value**.

```tsx
const filteredUsers = useMemo(() => {
  return users.filter(user => user.active);
}, [users]);
```

React reuses the memoized result while the dependencies remain unchanged.

Use it when:

- A computation is genuinely expensive.
- You need a stable derived value for a reason such as referential equality.

Don't wrap every trivial calculation in `useMemo`. Memoization itself has overhead and adds complexity.

Mental model:

> `useMemo` → memoize a **value**.

---

## 5. useCallback

`useCallback` memoizes a **function reference**.

```tsx
const handleClick = useCallback(() => {
  console.log("Hello");
}, []);
```

With the same dependencies, React can return the same function reference across renders.

### Practical example with React.memo

```tsx
const Child = React.memo(({ onClick }: { onClick: () => void }) => {
  console.log("Child rendered");
  return <button onClick={onClick}>Do something</button>;
});

function Parent() {
  const [count, setCount] = useState(0);

  const handleClick = useCallback(() => {
    console.log("Hello");
  }, []);

  return (
    <>
      <button onClick={() => setCount(prev => prev + 1)}>
        Count: {count}
      </button>

      <Child onClick={handleClick} />
    </>
  );
}
```

When `count` changes:

1. Parent re-renders.
2. `useCallback` returns the same `handleClick` reference because dependencies didn't change.
3. `Child` receives the same function prop.
4. Because `Child` is wrapped in `React.memo`, it can skip its render.

Without `useCallback`, a new function object would normally be created during every Parent render, so `React.memo` would see a changed prop reference.

### Important

`useCallback` alone does **not** prevent the parent from rendering.

The common optimization is:

```
Parent re-render
      ↓
useCallback keeps function reference stable
      ↓
React.memo child sees same prop reference
      ↓
Child can skip rendering
```

If the callback uses a changing value, include it in dependencies:

```tsx
const handleClick = useCallback(() => {
  console.log(userId);
}, [userId]);
```

Incorrect dependencies can produce stale closures.

Also useful when a function is itself a dependency of another hook.

Mental model:

> `useCallback` → memoize a **function reference**.

---

## 6. useContext

`useContext` lets a component consume a value provided by a Context.

```tsx
const UserContext = createContext<User | null>(null);

function UserProfile() {
  const user = useContext(UserContext);
  return <div>{user?.name}</div>;
}
```

A provider supplies the value:

```tsx
<UserContext.Provider value={user}>
  <UserProfile />
</UserContext.Provider>
```

When the context value changes, consumers that read that context can update.

### Context is useful for

- Authentication/user information
- Theme
- Locale
- App-level configuration
- Avoiding excessive prop drilling

Context is not automatically a complete state-management solution. It provides access to shared values; the state itself can live in state, a reducer, or another store.

---

## 7. useReducer

`useReducer` is useful when state has **multiple related fields or complex transitions**.

Instead of directly describing every state update:

```tsx
setLoading(true);
setUsers(data);
setError(null);
```

you model explicit actions:

```tsx
dispatch({ type: "FETCH_START" });
dispatch({ type: "FETCH_SUCCESS", payload: users });
dispatch({ type: "FETCH_ERROR", payload: error });
```

Example:

```tsx
type State = {
  loading: boolean;
  users: User[];
  error: string | null;
};

type Action =
  | { type: "FETCH_START" }
  | { type: "FETCH_SUCCESS"; payload: User[] }
  | { type: "FETCH_ERROR"; payload: string };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case "FETCH_START":
      return { ...state, loading: true, error: null };

    case "FETCH_SUCCESS":
      return { loading: false, users: action.payload, error: null };

    case "FETCH_ERROR":
      return { ...state, loading: false, error: action.payload };

    default:
      return state;
  }
}

function UserList() {
  const [state, dispatch] = useReducer(reducer, {
    loading: false,
    users: [],
    error: null
  });

  // API call can dispatch actions.
  // reducer decides how state changes.

  return null;
}
```

Mental model:

> `useReducer` = **HOW state changes**.

The reducer should be pure: given the same state and action, it should return the same next state. Side effects such as API calls belong outside the reducer.

---

## 8. Context + Reducer

A common pattern is to combine Context and Reducer when multiple components need access to the same state and dispatch function.

```tsx
const UserContext = createContext<{
  state: State;
  dispatch: React.Dispatch<Action>;
} | null>(null);

function UserProvider({ children }: { children: React.ReactNode }) {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <UserContext.Provider value={{ state, dispatch }}>
      {children}
    </UserContext.Provider>
  );
}

function App() {
  return (
    <UserProvider>
      <Header />
      <Dashboard />
      <UserList />
    </UserProvider>
  );
}

function UserList() {
  const context = useContext(UserContext);

  if (!context) throw new Error("UserList must be inside UserProvider");

  const { state, dispatch } = context;

  // read state
  // dispatch actions
}
```

Mental model:

```
useReducer → HOW state changes
Context    → WHO can access it
```

### Important re-render nuance

If the reducer lives inside `UserProvider`, dispatching an action re-renders the **Provider**, not automatically the entire application component tree.

Context consumers that observe a changed context value can update.

Being structurally under a Provider does not mean every descendant component function automatically executes whenever the Provider renders. React can reuse parts of the existing tree during reconciliation.

If the reducer state instead lives directly in `App`, then `App` re-renders when the state changes and its child tree participates in reconciliation.

---

## 9. Custom Hooks

A custom hook is a reusable function whose name starts with `use` and which can call other hooks.

```tsx
function useOnlineStatus() {
  const [online, setOnline] = useState(navigator.onLine);

  useEffect(() => {
    const handleOnline = () => setOnline(true);
    const handleOffline = () => setOnline(false);

    window.addEventListener("online", handleOnline);
    window.addEventListener("offline", handleOffline);

    return () => {
      window.removeEventListener("online", handleOnline);
      window.removeEventListener("offline", handleOffline);
    };
  }, []);

  return online;
}
```

Component:

```tsx
function Header() {
  const online = useOnlineStatus();

  return <span>{online ? "Online" : "Offline"}</span>;
}
```

### Critical point

Custom hooks share **logic**, not state.

If two components both call:

```tsx
const online = useOnlineStatus();
```

each component gets its own hook state.

For genuinely shared state, use something such as Context or an external state store.

---

## 10. Rules of Hooks

Hooks must be called:

- At the top level of a React component.
- At the top level of a custom hook.

Don't call hooks:

- Inside `if` statements
- Inside loops
- Inside nested functions
- After conditional returns that can change hook order

Bad:

```tsx
if (loggedIn) {
  const [user, setUser] = useState(null);
}
```

React relies on a consistent hook call order between renders.

---

## SDE-2 Mental Model

| Hook | Main purpose |
|---|---|
| `useState` | Component state |
| `useEffect` | Synchronize with external systems |
| `useRef` | Persistent mutable value without re-render |
| `useMemo` | Memoize a computed value |
| `useCallback` | Memoize a function reference |
| `useContext` | Consume shared context |
| `useReducer` | Manage complex state transitions |
| Custom hook | Reuse stateful logic |

### Key distinctions

```
useState
→ state changes can trigger rendering

useRef
→ persists, but changing .current does not render

useMemo
→ memoizes a VALUE

useCallback
→ memoizes a FUNCTION REFERENCE

useReducer
→ explicit state transitions

Context
→ makes a value accessible to consumers

Custom Hook
→ reuses hook-based logic
```

### Practical optimization rule

Don't add `useMemo`, `useCallback`, or `React.memo` everywhere.

First make the component correct. Add memoization when there is a real performance or referential-identity reason.
