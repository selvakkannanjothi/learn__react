# JavaScript Function Definitions vs. Function Calls (React Components)

## JavaScript

```js
function F1() {
  // Function declaration: defines F1.
}
```

`F1` is defined here and can be called later with `F1()`.

```js
F2(() => {
  // ...
});
```

This calls an already-defined function named `F2` and passes it an arrow function as an argument. The arrow function is a value that `F2` may call; this syntax does not define `F2`.

To define a function with arrow syntax:

```js
const F2 = () => {
  // Function expression assigned to F2.
};
```

**Remember:** `F2(() => {})` is a function call; `const F2 = () => {}` is a function definition.

## React

A function component is a JavaScript function that returns React UI. It can be written either way:

```jsx
function F1() {
  return <h2>Hello</h2>;
}

const F2 = () => {
  return <h2>Hello</h2>;
};
```

Render components with JSX:

```jsx
<F1 />
<F2 />
```

For example, `F2(() => { ... })` does not define the component referenced by `<F2 />`; it attempts to call `F2` and pass it a callback. Define `F2` as a function or arrow function component instead.

When rendering data with `map`, put JavaScript expressions inside JSX curly braces and provide a stable `key` for each element:

```jsx
function F2() {
  return my_data.map(item => (
    <pre key={item.name}>{JSON.stringify(item)}</pre>
  ));
}
```

# Hoisting & Temporal Dead Zone (Why `F1` Works but `F2` Fails)

Mistake: `<F2 />` was rendered *above* `const F2 = () => {...}`, while `<F1 />` worked even though `function F1()` was also defined below.

```jsx
root.render(<><F1 /><F2 /></>); // F1 works, F2 throws

function F1() { return <h2>F1</h2>; }  // hoisted with its body
const F2 = () => <h2>F2</h2>;           // not usable until this line runs
```

- **Function declaration** (`function F1() {}`) is **hoisted with its body** → usable before the line where it's written.
- **`const` / `let`** names are reserved but stay in the **Temporal Dead Zone (TDZ)** until the assignment executes. Accessing them earlier throws `ReferenceError: Cannot access 'F2' before initialization`.
- `var` is hoisted too, but as `undefined` → calling it early gives `TypeError: F2 is not a function`.
- This is a **JavaScript** rule, not React. JSX `<F2 />` compiles to code that reads the `F2` value, so it needs `F2` to exist at that moment.

**Fix:** define `const F2 = ...` above `root.render(...)`, or use `function F2() {...}`.

# JSX Curly Braces

JSX treats plain text as text. Wrap JavaScript expressions in `{}`.

```jsx
<pre>JSON.stringify(item)</pre>    // ❌ shows the literal text "JSON.stringify(item)"
<pre>{JSON.stringify(item)}</pre>  // ✅ runs the expression
```

# React List `key`

A `key` is an identifier React uses to match each rendered list element with the same item across renders.

```jsx
{my_data.map(item => (
  <pre key={item.name}>{JSON.stringify(item)}</pre>
))}
```

**Why it matters:** when items are added, removed, or reordered, keys let React update the correct elements instead of mixing them up. Missing keys trigger a console warning; unstable or duplicate keys can cause incorrect UI updates (e.g., input values or state sticking to the wrong row).

- Must be **unique among siblings** (not across the whole app).
- Must be **stable** — the same item keeps the same key every render. Prefer a real ID (e.g., database ID).
- Put the `key` on the **top-level element returned by `map`**.
- Avoid **array indexes** when the list can change order or size; never use **random values** (`Math.random()`), since they change every render.
- `key` is used by React itself — it is **not** available inside the component as `props.key`. Pass it separately (e.g., `id={item.id}`) if needed.

# ✍️ Quick Revision Notes

### Functions vs Components
- `function F1() {}` → **defines** a function.
- `F2(() => {})` → **calls** `F2`, passing a callback.
- `const F2 = () => {}` → **defines** an arrow function.
- React component = function that returns UI. Capitalized name.
- Define it, then render with `<F2 />`.

### Hoisting / TDZ
- Function declaration → hoisted → usable before its definition.
- `const`/`let` → TDZ until initialized → define **before** use.
- JavaScript rule; not React.

### JSX
- JS inside JSX → wrap in `{}`.

### `key`
- Helps React track list items across renders.
- Unique among siblings + stable.
- Use item ID: `<li key={item.id}>…</li>`.
- No indexes for changing lists; never random keys.
- Put on the top-level element returned by `map`.
- Not available as `props.key`.

**Memory line:** *Declaration hoists; `const` waits. Define, then render. Braces for JS. Stable keys for lists.*