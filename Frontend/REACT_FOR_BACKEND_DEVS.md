# React — A Backend Developer's Practical Reference

React is just a way to build UI as reusable functions that return HTML-like markup (JSX) and re-run automatically when their data changes. Coming from backend, the mental shift is: **UI = a function of state**. Change the state, React re-runs the function, the UI updates. No manual DOM manipulation (which you already learned — good news, you rarely touch `document.querySelector` again).

---

## 1. Components — the basic unit

A component is just a JS function that returns JSX (HTML-looking syntax inside JS).

```jsx
function Greeting() {
  return <h1>Hello, world!</h1>;
}

export default Greeting;
```

- Component names **must start with a capital letter** (`Greeting`, not `greeting`) — React uses this to distinguish components from regular HTML tags.
- JSX looks like HTML but is actually JS — you can embed any JS expression inside `{ }`.

```jsx
function Greeting() {
  const name = "Subham";
  return <h1>Hello, {name}!</h1>;   // {} injects a JS expression
}
```

**Think of a component like a backend function that returns a template** — same idea as a server-side render function (e.g. Jinja/EJS), except it re-runs automatically whenever its inputs change.

---

## 2. Props — passing data INTO a component (like function arguments)

Props are how a parent component passes data down to a child. **They are read-only** — a component never modifies its own props.

```jsx
function UserCard({ name, email }) {
  return (
    <div className="card">
      <h3>{name}</h3>
      <p>{email}</p>
    </div>
  );
}

// Using it — parent passes data down, just like calling a function with args
function App() {
  return <UserCard name="Subham Das" email="subham@example.com" />;
}
```

- `{ name, email }` in the function signature = destructuring the `props` object (same as `props.name`, `props.email`).
- Props flow **one direction only**: parent → child. A child cannot change its parent's data directly — it can only call a function the parent passed down (see events below).

### Passing a list of data (very common — think "rendering rows from a DB query")

```jsx
function UserList({ users }) {
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

**`key` is mandatory when rendering lists** — it's a unique ID (usually your DB primary key) React uses to track which item is which across re-renders, so it can update efficiently instead of re-rendering the whole list. Never use the array index as `key` if the list can reorder/filter/insert.

---

## 3. State — data a component owns and can change (`useState`)

Props come from outside; **state is data the component manages itself**, and changing it triggers a re-render.

```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0); // [currentValue, setterFunction] = useState(initialValue)

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

- `useState(0)` returns an array: `[value, setterFunction]`. The naming convention is always `[thing, setThing]`.
- Calling `setCount(...)` tells React "re-render this component with the new value" — you never mutate state directly (never `count++`, always `setCount(newValue)`).
- **Functional update** — when the new state depends on the old state, pass a function instead of a value, to avoid stale-state bugs (especially inside loops/async code):

```jsx
setCount(prevCount => prevCount + 1);
```

### State with objects (very common for forms — see section 5)

```jsx
const [user, setUser] = useState({ name: '', email: '' });

// Updating one field without losing the others — spread the old object
setUser(prev => ({ ...prev, name: 'New Name' }));
```

You'll do this `{ ...prev, field: newValue }` pattern constantly — it's the equivalent of an immutable `PATCH` update on an object.

---

## 4. Handling Events

React events look like HTML events but are camelCase and take a function, not a string.

```jsx
function Button() {
  function handleClick() {
    console.log('Clicked!');
  }

  return <button onClick={handleClick}>Click me</button>;
}
```

| Pattern | When to use |
|---|---|
| `onClick={handleClick}` | Pass the function reference — React calls it on click. |
| `onClick={() => handleClick(id)}` | Use an arrow function when you need to pass arguments — otherwise it would execute immediately on render instead of on click. |
| `onChange={(e) => setValue(e.target.value)}` | For inputs — fires on every keystroke/change. `e.target.value` is the current input value (same `event.target` concept from plain JS DOM). |
| `onSubmit={(e) => { e.preventDefault(); ... }}` | Form submission — `preventDefault()` stops the default full-page-reload behavior, same as plain JS. |

```jsx
function DeleteButton({ userId, onDelete }) {
  return <button onClick={() => onDelete(userId)}>Delete</button>;
}
```

### Passing functions down as props (child → parent communication)

Since props only flow one way (parent → child), a child triggers changes in the parent by **calling a function the parent passed down**:

```jsx
function Parent() {
  const [count, setCount] = useState(0);
  return <Child onIncrement={() => setCount(count + 1)} />;
}

function Child({ onIncrement }) {
  return <button onClick={onIncrement}>+1</button>;
}
```

This pattern — passing a callback prop — is how React "reverses" the one-way data flow. It's the same idea as a webhook/callback in backend code: the child doesn't own the logic, it just invokes what it was given.

---

## 5. Forms — the part you'll use constantly connecting to your APIs

Unlike plain HTML forms, React usually manages input values through state ("controlled components") instead of letting the DOM hold the value.

```jsx
function LoginForm() {
  const [form, setForm] = useState({ email: '', password: '' });

  function handleChange(e) {
    const { name, value } = e.target;
    setForm(prev => ({ ...prev, [name]: value })); // [name] = computed property key
  }

  async function handleSubmit(e) {
    e.preventDefault();
    const res = await fetch('/api/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(form),
    });
    const data = await res.json();
    console.log(data);
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        name="email"
        type="email"
        value={form.email}
        onChange={handleChange}
      />
      <input
        name="password"
        type="password"
        value={form.password}
        onChange={handleChange}
      />
      <button type="submit">Log in</button>
    </form>
  );
}
```

### Key ideas here:
- **Controlled input** = `value={form.email}` + `onChange={handleChange}` — the input's displayed value always comes from state, and every keystroke updates that state. This is the standard React form pattern.
- One shared `handleChange` using `name` + `[name]: value` (computed key) handles ANY field — you don't need a separate handler per input.
- `e.preventDefault()` stops the native browser form submission (which would reload the page) — you handle the submission yourself with `fetch`, exactly like you'd call your API from plain JS.
- This is your direct bridge to the backend: the `body: JSON.stringify(form)` payload is literally what your Express/Django/etc. route receives as `req.body`.

---

## 6. `useEffect` — running code in response to renders (data fetching, subscriptions)

`useEffect` runs side effects — things outside of just rendering UI, most commonly: fetching data when a component loads.

```jsx
import { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    async function fetchUser() {
      const res = await fetch(`/api/users/${userId}`);
      const data = await res.json();
      setUser(data);
    }
    fetchUser();
  }, [userId]); // dependency array

  if (!user) return <p>Loading...</p>;
  return <h2>{user.name}</h2>;
}
```

### The dependency array `[...]` is the whole concept to understand:

| Dependency array | When the effect runs |
|---|---|
| `useEffect(() => {...})` (no array) | After **every** render — rarely what you want. |
| `useEffect(() => {...}, [])` | Only **once**, right after the first render — like "on component mount." Common for initial data fetch. |
| `useEffect(() => {...}, [userId])` | Runs on first render AND whenever `userId` changes — "re-fetch when this value changes." |

**Mental model:** `useEffect` is your `onMount` / `onUpdate` hook — closest backend analogy is a `useEffect(fn, [])` acting like app startup code, and `useEffect(fn, [dep])` acting like a listener that reacts when a specific piece of state changes (similar to a DB trigger firing when a column updates).

### Cleanup function (for subscriptions, timers, intervals)

```jsx
useEffect(() => {
  const interval = setInterval(() => console.log('tick'), 1000);
  return () => clearInterval(interval); // cleanup — runs before next effect or on unmount
}, []);
```

---

## 7. Other Hooks Worth Knowing (used less often, but you'll hit them)

| Hook | Use case |
|---|---|
| `useRef` | Hold a mutable value that does NOT trigger re-render, or get a direct reference to a DOM element (e.g. focusing an input programmatically). |
| `useContext` | Share data across many components without passing props down manually through every level ("prop drilling"). Common for auth state, theme. |
| `useMemo` | Cache an expensive calculation so it doesn't re-run on every render unless its dependencies change. |
| `useCallback` | Cache a function definition itself (useful when passing callbacks to child components to avoid unnecessary re-renders). |

```jsx
// useRef example — focusing an input
function SearchBox() {
  const inputRef = useRef(null);
  useEffect(() => { inputRef.current.focus(); }, []);
  return <input ref={inputRef} />;
}
```

You won't need `useMemo`/`useCallback` heavily as a beginner — learn `useState`, `useEffect`, and `onChange`/`onClick` handling first; these cover the vast majority of real work.

---

## 8. Fetching Data — the pattern you'll use in almost every app

```jsx
function OrderList() {
  const [orders, setOrders] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetch('/api/orders')
      .then(res => res.json())
      .then(data => setOrders(data))
      .catch(err => setError(err.message))
      .finally(() => setLoading(false));
  }, []);

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error}</p>;

  return (
    <ul>
      {orders.map(order => (
        <li key={order.id}>{order.status}</li>
      ))}
    </ul>
  );
}
```

This **loading / error / data** three-state pattern is the standard shape for any component that fetches from your backend — get comfortable with it, you'll write it constantly.

---

## Suggested Learning Order

1. **Components + JSX** — the basic building block.
2. **Props** — passing data down, rendering lists with `key`.
3. **`useState`** — component-owned data, especially objects for forms.
4. **Event handling** (`onClick`, `onChange`, `onSubmit`) — including passing callback props up to parents.
5. **Controlled forms** — the `value` + `onChange` + shared handler pattern.
6. **`useEffect`** — fetching data on mount, and re-fetching on dependency change.
7. `useRef`, `useContext` — pick up as needed once the above feels natural.

### Practice project idea
Build a small **"Orders Dashboard"**: fetch a list of orders from an API on mount (`useEffect` + `useState` + loading/error states), render them in a list (`props` + `key`), and add a form to create a new order (`useState` for form fields, `onChange`, `onSubmit` with `fetch POST`) that adds the new order to the list on success. This single project touches every concept above and mirrors real full-stack work — connecting a React frontend to your own backend API.
