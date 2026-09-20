# React State Management — SDE-2

## 1. Prop Drilling

Prop drilling means passing data through intermediate components that do not need it just to reach a deeper component.

For a small hierarchy, props are often the cleanest solution. It becomes painful when many levels or unrelated branches need the same state.

## 2. Context

Context makes a shared value available to descendants without manually passing it through every intermediate component.

```tsx
const UserContext = createContext<User | null>(null);

function UserProfile() {
  const user = useContext(UserContext);
  return <h1>{user?.name}</h1>;
}
```

For a real application, prefer a dedicated Provider component rather than putting the whole implementation inside App:

```tsx
function UserProvider({ children }) {
  const [user, setUser] = useState(null);

  return (
    <UserContext.Provider value={{ user, setUser }}>
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
```

A tiny demo can keep this inside App.

## 3. Context + useReducer

Context answers **WHO can access state**.

useReducer answers **HOW state changes**.

```
useReducer = HOW state changes
Context    = WHO can access it
```

This is a useful lightweight shared-state architecture for small/medium applications.

## 4. Context Provider Value and Re-renders

This is a common gotcha:

```tsx
<UserContext.Provider value={{ user, setUser }}>
```

The object is recreated on every provider render:

```
Render 1 → Object A
Render 2 → Object B

A !== B
```

So the context value has a new identity even when user itself did not change.

It can be stabilized when appropriate:

```tsx
const value = useMemo(
  () => ({ user, setUser }),
  [user]
);
```

Do not blindly memoize every context value. If consumers need the update anyway, memoization does not help.

For unrelated state, splitting contexts can be more useful:

```
UserContext
CartContext
NotificationContext
ThemeContext
```

## 5. When Context Is Not Enough

Context is primarily a **value/dependency distribution mechanism**. It does not automatically provide the broader architecture of an external state manager.

A large application may have:

```
authentication
user preferences
shopping cart
notifications
products
orders
permissions
```

One giant context can become difficult to reason about. You can split contexts or combine Context with useReducer, but an external state manager can provide more structure when coordination becomes complex.

## 6. Redux Conceptually

For SDE-2 interviews, know:

```
Store
Action
Reducer
Dispatch
Selector
```

### Store

Holds application state.

```tsx
{
  user: {...},
  cart: [...],
  orders: [...]
}
```

Modern Redux applications typically use Redux Toolkit to configure the store.

### Action

Describes **what happened**.

```tsx
{
  type: "cart/itemAdded",
  payload: product
}
```

An action describes an event; it does not directly modify state.

### Dispatch

Sends an action:

```tsx
dispatch({
  type: "cart/itemAdded",
  payload: product
});
```

### Reducer

Calculates the next state from current state + action.

```tsx
function cartReducer(state, action) {
  switch (action.type) {
    case "cart/itemAdded":
      return {
        ...state,
        items: [...state.items, action.payload]
      };
    default:
      return state;
  }
}
```

Reducers should be predictable and free of side effects.

### Selector

Reads or derives state needed by a component:

```tsx
const cartItems = useSelector(
  state => state.cart.items
);
```

Selectors can also derive values:

```tsx
const selectCartTotal = state =>
  state.cart.items.reduce(
    (total, item) => total + item.price,
    0
  );
```

## 7. Complete Redux Flow

```
User clicks button
       ↓
Component
       ↓
dispatch(action)
       ↓
Redux store
       ↓
Reducer
       ↓
New state
       ↓
Components whose selected state changed
       ↓
UI update
```

## 8. Context + useReducer vs Redux

| | Context + useReducer | Redux |
|---|---|---|
| State | Usually provider-owned | Central store |
| Distribution | Context | Redux store/provider |
| State transitions | Reducer | Reducer |
| Access | useContext | Selectors |
| Actions | Optional convention | Core concept |
| DevTools | Basic/limited | Strong ecosystem |
| Middleware | Manual/custom | Middleware ecosystem |
| Boilerplate | Usually lower | Historically higher; Redux Toolkit reduces it |
| Typical fit | Small/medium shared state | Complex application-wide state |

Redux is not simply "Context but better."

- **Context** primarily distributes values to descendants.
- **Redux** provides a state-management architecture/ecosystem around centralized state, actions, reducers, selectors, middleware and tooling.

## 9. Does Redux Solve Context's Object-Identity Problem?

With:

```tsx
<UserContext.Provider value={{ user, cart }}>
```

a new object is created on each provider render.

Redux instead uses selectors:

```tsx
const user = useSelector(state => state.user);
```

A component subscribes to its selected result rather than simply subscribing to the whole store.

Example:

```
Redux Store
├── user
├── cart
└── notifications

Component A → state.user
Component B → state.cart
```

If only cart changes, Component B's selected result changes while Component A's selected result can remain the same.

Important nuance: Redux does not magically eliminate all unnecessary renders. For example:

```tsx
useSelector(state => ({
  user: state.user,
  cart: state.cart
}))
```

returns a new object result, so equality behavior matters. Memoized selectors or an appropriate equality strategy can help when necessary.

Interview distinction:

```
Context
→ "Did the context value change?"

Redux + useSelector
→ "Did the state this component selected change?"
```

## 10. When to Use What?

```
Parent → Child
    ↓
  Props

Shared, simple value
    ↓
  Context

Shared state + structured transitions
    ↓
Context + useReducer

Complex application-wide state
    ↓
Redux / another external state manager
```

Do not choose Redux merely because an application is "large." Actual state-sharing and coordination requirements matter.

## SDE-2 Mental Model

```
Props
→ explicit parent → child communication

Context
→ make shared values accessible to descendants

useReducer
→ structure complex state transitions

Context + useReducer
→ lightweight shared-state architecture

Redux
→ structured application-wide state management
```

### Redux vocabulary

```
Store
→ holds state

Action
→ describes what happened

Dispatch
→ sends an action

Reducer
→ calculates next state

Selector
→ reads/derives state for a component
```

### Interview answer: Why not just use Context?

> Context is useful for making shared values available to components, but it doesn't itself provide Redux's broader state-management architecture, such as a centralized store model, action-driven updates, selectors, middleware, and surrounding tooling. For simpler shared state, Context can be sufficient; as state coordination becomes more complex, an external state manager may provide more structure.
