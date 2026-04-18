# React for WordPress Plugin Development — Complete Study Guide

> **Purpose:** Comprehensive reference for building blocks, block editor extensions, and wp-admin React screens.  
> **Level:** Mid-level developer with JavaScript background, learning the WordPress/React ecosystem.

---

## Table of Contents

1. [Modern JavaScript](#1-modern-javascript)
2. [React with @wordpress/element](#2-react-with-wordpresselement)
3. [React Hooks (Core)](#3-react-hooks-core)
4. [Composition and Architecture](#4-composition-and-architecture)
5. [Gutenberg Blocks — Registration and Metadata](#5-gutenberg-blocks--registration-and-metadata)
6. [Block Editor UI (@wordpress/block-editor)](#6-block-editor-ui-wordpressblock-editor)
7. [Editor Chrome and Data](#7-editor-chrome-and-data)
8. [@wordpress/components — Common UI](#8-wordpresscomponents--common-ui)
9. [Internationalization (@wordpress/i18n)](#9-internationalization-wordpressi18n)
10. [REST API and Settings (@wordpress/api-fetch)](#10-rest-api-and-settings-wordpressapi-fetch)
11. [Icons and Media](#11-icons-and-media)
12. [Styling for Plugins](#12-styling-for-plugins)
13. [Build Tooling (@wordpress/scripts)](#13-build-tooling-wordpressscripts)
14. [Scripts Outside React](#14-scripts-outside-react)
15. [Security and Content Safety](#15-security-and-content-safety)
16. [Optional / Advanced](#16-optional--advanced)
17. [Suggested Study Order](#17-suggested-study-order)

---

## 1. Modern JavaScript

WordPress plugin development requires solid Modern JavaScript (ES6+) foundations. These features appear throughout React and @wordpress/* code.

---

### ES Modules — `import` / `export`

**Definition:** A standardised module system that lets you split code across files and explicitly declare what each file exposes and consumes.

**Key concepts:**
- **Named export/import** — export multiple values by name from one file.
- **Default export/import** — export one primary value per file.
- **Side-effect import** — import a file purely for its side effects (e.g., registering a block).

**Usage examples:**

```js
// named exports
export const blockName = 'myplugin/hero';
export function save() { return null; }

// named imports
import { blockName, save } from './block';

// default export
export default function Edit({ attributes }) { /* ... */ }

// default import (any name works)
import Edit from './Edit';

// side-effect import (registers the block on load)
import './blocks/hero';
```

**In WordPress context:** `@wordpress/scripts` (Webpack) resolves `@wordpress/*` packages as Webpack externals — they are provided by WordPress at runtime, not bundled.

---

### Destructuring — Objects, Arrays, Rest, Function Params

**Definition:** Syntax to unpack values from objects or arrays into distinct variables in a single statement.

**Usage examples:**

```js
// Object destructuring
const { title, content, className } = attributes;

// With rename
const { title: blockTitle } = attributes;

// Default values
const { color = 'blue' } = attributes;

// Array destructuring
const [first, second, ...rest] = items;

// In function parameters (very common in React components)
function Edit({ attributes, setAttributes, className }) {
  // attributes, setAttributes, className are ready to use
}

// Nested destructuring
const { typography: { fontSize } } = supports;
```

**Why it matters:** Every WordPress block component receives `{ attributes, setAttributes, clientId, isSelected }` as props — you'll destructure this constantly.

---

### Spread — Objects, Arrays, Prop Spreading

**Definition:** The `...` operator expands an iterable or object into individual elements. Used heavily for immutable state updates.

**Usage examples:**

```js
// Spread to copy + update object (immutable update pattern)
setAttributes({ ...attributes, title: 'New Title' });

// Merge objects
const merged = { ...defaults, ...userSettings };

// Spread array
const newItems = [...existingItems, newItem];

// Prop spreading in JSX — passes all props through
function Wrapper(props) {
  return <div {...props} />;
}

// Practical block example — update one attribute without mutating
function onChangeTitle(newTitle) {
  setAttributes({ ...attributes, title: newTitle });
}
```

---

### Arrow Functions

**Definition:** A compact function syntax (`=>`) that does not have its own `this` binding. Essential in hooks-heavy React code where callbacks are common.

**Usage examples:**

```js
// Basic arrow function
const double = n => n * 2;

// With body block
const handleClick = (event) => {
  event.preventDefault();
  doSomething();
};

// As inline callback (common in JSX)
<Button onClick={() => setAttributes({ isOpen: true })}>Open</Button>

// Returning an object (wrap in parens to avoid ambiguity with block body)
const makeItem = (id) => ({ id, selected: false });
```

---

### Template Literals

**Definition:** Backtick strings that support multi-line content and embedded expressions using `${}`.

```js
// Interpolation
const message = `Block "${title}" saved successfully.`;

// Multi-line (useful for building class names)
const className = `
  my-block
  ${isActive ? 'my-block--active' : ''}
  ${align ? `my-block--align-${align}` : ''}
`.trim();

// REST API URL building
const endpoint = `/wp/v2/posts/${postId}`;
```

---

### Optional Chaining — `?.`

**Definition:** Allows safe access to deeply nested properties. If any part of the chain is `null` or `undefined`, the expression short-circuits to `undefined` instead of throwing.

```js
// Without optional chaining (risky)
const size = attributes.typography.fontSize; // throws if typography is undefined

// With optional chaining
const size = attributes?.typography?.fontSize; // returns undefined safely

// On method calls
const label = post?.getTitle?.();

// On array indexing
const first = posts?.[0]?.title;
```

**WordPress context:** API responses are often partial — use `?.` when reading deeply nested REST API payloads.

---

### Nullish Coalescing — `??` vs `||`

**Definition:** `??` returns the right-hand value only when the left is `null` or `undefined` (not `0`, `false`, or `''`). This is different from `||` which treats any falsy value as absent.

```js
// || treats 0 and '' as falsy (wrong for many attributes)
const columns = attributes.columns || 3; // if columns is 0, returns 3 — WRONG

// ?? only triggers on null/undefined — correct for attribute defaults
const columns = attributes.columns ?? 3; // if columns is 0, returns 0 — CORRECT

// Common pattern for block attributes with 0 or false as valid values
const showTitle = attributes.showTitle ?? true;
const opacity   = attributes.opacity   ?? 100;
```

---

### Array Methods — `map`, `filter`, `reduce`, `find`, `some`

**Definition:** Built-in Array methods that return new arrays/values without mutating the original — essential for React's immutable state model.

```js
const posts = [
  { id: 1, title: 'Hello', published: true },
  { id: 2, title: 'World', published: false },
];

// map — transform each item
const titles = posts.map(post => post.title);
// ['Hello', 'World']

// filter — keep matching items
const published = posts.filter(post => post.published);
// [{ id: 1, title: 'Hello', published: true }]

// find — first matching item
const target = posts.find(post => post.id === 2);

// some — does at least one match?
const hasUnpublished = posts.some(post => !post.published); // true

// reduce — aggregate into a single value
const idMap = posts.reduce((acc, post) => {
  acc[post.id] = post;
  return acc;
}, {});

// Immutable update — update one item in an array
const updated = posts.map(post =>
  post.id === 1 ? { ...post, title: 'Updated' } : post
);
```

---

### Promises and `async`/`await`

**Definition:** Promises represent eventual results of asynchronous operations. `async`/`await` is syntactic sugar that makes promise-based code read like synchronous code.

```js
// Promise chain style
apiFetch({ path: '/wp/v2/posts' })
  .then(posts => console.log(posts))
  .catch(error => console.error(error));

// async/await style (preferred)
async function loadPosts() {
  try {
    const posts = await apiFetch({ path: '/wp/v2/posts' });
    return posts;
  } catch (error) {
    console.error('Failed to load posts:', error);
  }
}

// Promise.all — run multiple requests in parallel
const [posts, categories] = await Promise.all([
  apiFetch({ path: '/wp/v2/posts' }),
  apiFetch({ path: '/wp/v2/categories' }),
]);
```

---

### JSON

**Definition:** JavaScript Object Notation — the data format for WordPress REST API request and response bodies.

```js
// Parsing a response string
const data = JSON.parse('{"id":1,"title":"Hello"}');
// { id: 1, title: 'Hello' }

// Serialising for a POST body
const body = JSON.stringify({ title: 'New Post', status: 'draft' });

// Pretty printing for debugging
console.log(JSON.stringify(data, null, 2));

// apiFetch handles JSON automatically — you rarely call these manually
// but they're needed when storing JSON in block attributes
const stored = JSON.stringify(complexObject);
setAttributes({ config: stored });
const parsed = JSON.parse(attributes.config);
```

---

## 2. React with `@wordpress/element`

WordPress ships React as part of core and exposes it through `@wordpress/element`. You import from `@wordpress/element` in plugin code — not directly from `react`.

```js
// Correct for WordPress plugins
import { useState, useEffect, Fragment } from '@wordpress/element';

// NOT this (would bundle a second copy of React)
import React, { useState } from 'react';
```

---

### JSX

**Definition:** JSX is a syntax extension that lets you write HTML-like markup inside JavaScript. Babel/webpack compiles it to `React.createElement()` calls.

**Key rules:**
- `class` → `className`
- `for` → `htmlFor`
- Self-closing tags need `/>`
- Only one root element (or use `<>` Fragment)
- JavaScript expressions go inside `{}`

```jsx
import { __ } from '@wordpress/i18n';

function MyBlock({ attributes }) {
  const { title, isActive } = attributes;

  return (
    <div className={`my-block ${isActive ? 'is-active' : ''}`}>
      <h2>{title || __('Default Title', 'my-plugin')}</h2>
      <img src={attributes.imgUrl} alt={attributes.imgAlt} />
    </div>
  );
}
```

---

### Fragments — `<>` and `<Fragment>`

**Definition:** A Fragment groups elements without adding a DOM node. Use `<Fragment key={id}>` when you need a `key` prop on the fragment (e.g., in lists).

```jsx
import { Fragment } from '@wordpress/element';

// Short syntax — when no props needed
function Controls() {
  return (
    <>
      <label>Title</label>
      <input type="text" />
    </>
  );
}

// Long form — when key is required
function ItemList({ items }) {
  return (
    <ul>
      {items.map(item => (
        <Fragment key={item.id}>
          <li>{item.title}</li>
          <li className="divider" />
        </Fragment>
      ))}
    </ul>
  );
}
```

---

### Conditional UI

**Definition:** Render different elements or nothing based on state/prop values.

```jsx
function Edit({ attributes, isSelected }) {
  const { showCaption, caption } = attributes;

  // && — render only if truthy (watch out: 0 renders as "0" in JSX)
  return (
    <div>
      {isSelected && <Toolbar />}

      {/* Ternary — two alternatives */}
      {showCaption
        ? <figcaption>{caption}</figcaption>
        : <figcaption className="placeholder">Add a caption…</figcaption>
      }

      {/* Early return for entire component */}
      {/* (done outside JSX, before the return statement) */}
    </div>
  );
}

// Early return pattern
function MaybeRender({ isReady }) {
  if (!isReady) return null;
  return <div>Ready!</div>;
}
```

---

### Lists and Keys

**Definition:** Render arrays of elements using `.map()`. React requires a stable `key` prop on each list item so it can efficiently reconcile DOM changes.

```jsx
function PostList({ posts }) {
  return (
    <ul>
      {posts.map(post => (
        // key must be unique among siblings and stable (use IDs, not array index)
        <li key={post.id}>
          <a href={post.link}>{post.title.rendered}</a>
        </li>
      ))}
    </ul>
  );
}
```

**Bad practice — avoid using index as key when list can reorder:**
```jsx
// AVOID: index as key causes bugs if list items reorder
{items.map((item, index) => <li key={index}>{item}</li>)}

// CORRECT: use stable unique ID
{items.map(item => <li key={item.id}>{item.label}</li>)}
```

---

### Function Components

**Definition:** A function that accepts props and returns JSX. The standard way to write React components (class components are legacy in modern WordPress code).

```jsx
// Basic component
function BlockTitle({ title, level = 2 }) {
  const Tag = `h${level}`;
  return <Tag className="block-title">{title}</Tag>;
}

// Usage
<BlockTitle title="My Section" level={3} />
```

---

### Controlled Inputs

**Definition:** An input whose `value` is driven by React state, with every change handled through an `onChange` handler. This keeps the UI and data in sync.

```jsx
import { useState } from '@wordpress/element';

function NameInput() {
  const [name, setName] = useState('');

  return (
    <input
      type="text"
      value={name}                          // value from state
      onChange={e => setName(e.target.value)} // update state on every keystroke
    />
  );
}

// In a block — controlled via attributes
function Edit({ attributes, setAttributes }) {
  return (
    <input
      type="text"
      value={attributes.title}
      onChange={e => setAttributes({ title: e.target.value })}
    />
  );
}
```

---

### Mounting Admin UIs

**Definition:** For wp-admin screens that are not blocks, you render a React app onto a DOM node that WordPress provides in the page HTML.

```php
// PHP — provide the mount point
echo '<div id="my-plugin-app"></div>';
```

```js
// JS — mount React onto the DOM node
import { createRoot } from '@wordpress/element';
import App from './App';

const container = document.getElementById('my-plugin-app');
if (container) {
  const root = createRoot(container);
  root.render(<App />);
}
```

---

## 3. React Hooks (Core)

Hooks let function components use React features like state and side effects. All hooks follow the rules: call only at the **top level** of a component or custom hook, never inside loops, conditions, or nested functions.

---

### `useState`

**Definition:** Adds local state to a function component. Returns `[currentValue, setter]`.

```jsx
import { useState } from '@wordpress/element';

function Counter() {
  const [count, setCount] = useState(0);      // initial value = 0

  // Functional update — use when new state depends on old state
  const increment = () => setCount(prev => prev + 1);

  return <button onClick={increment}>Count: {count}</button>;
}

// Object state — always spread to avoid losing other fields
function Form() {
  const [form, setForm] = useState({ name: '', email: '' });

  const updateName = (e) =>
    setForm(prev => ({ ...prev, name: e.target.value }));

  return <input value={form.name} onChange={updateName} />;
}
```

---

### `useEffect`

**Definition:** Runs side effects after render — data fetching, subscriptions, DOM mutations, timers. The dependency array controls when it re-runs.

```jsx
import { useState, useEffect } from '@wordpress/element';
import apiFetch from '@wordpress/api-fetch';

function PostLoader({ postId }) {
  const [post, setPost] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    let cancelled = false; // cleanup flag for async race conditions

    setLoading(true);
    apiFetch({ path: `/wp/v2/posts/${postId}` })
      .then(data => {
        if (!cancelled) {
          setPost(data);
          setLoading(false);
        }
      });

    // Cleanup function — runs before next effect or on unmount
    return () => { cancelled = true; };

  }, [postId]); // re-run whenever postId changes

  if (loading) return <p>Loading…</p>;
  return <h2>{post?.title?.rendered}</h2>;
}
```

**Dependency array rules:**
- `[]` — run once after mount, clean up on unmount
- `[a, b]` — run after mount and whenever `a` or `b` change
- No array — run after every render (rarely correct)

---

### `useLayoutEffect`

**Definition:** Like `useEffect` but fires synchronously after DOM mutations, before the browser paints. Use only when you need to read layout (e.g., element dimensions) and immediately update state to avoid a visible flash.

```jsx
import { useRef, useLayoutEffect, useState } from '@wordpress/element';

function MeasuredBox({ children }) {
  const ref = useRef(null);
  const [width, setWidth] = useState(0);

  useLayoutEffect(() => {
    // Runs after DOM update, before paint
    setWidth(ref.current.getBoundingClientRect().width);
  }, []);

  return <div ref={ref}>{children} — width: {width}px</div>;
}
```

**Rule of thumb:** Start with `useEffect`. Switch to `useLayoutEffect` only if you see a visual flicker.

---

### `useRef`

**Definition:** Returns a mutable ref object whose `.current` property persists across renders without triggering re-renders. Used for: DOM node access, storing mutable values (timers, previous values), avoiding stale closures.

```jsx
import { useRef, useEffect } from '@wordpress/element';

// 1. DOM node access — auto-focus an input
function AutoFocusInput() {
  const inputRef = useRef(null);

  useEffect(() => {
    inputRef.current?.focus();
  }, []);

  return <input ref={inputRef} />;
}

// 2. Mutable value — store a timer ID without triggering re-render
function Debounced({ onSearch }) {
  const timerRef = useRef(null);

  const handleChange = (value) => {
    clearTimeout(timerRef.current);
    timerRef.current = setTimeout(() => onSearch(value), 300);
  };

  return <input onChange={e => handleChange(e.target.value)} />;
}
```

---

### `useMemo`

**Definition:** Memoizes the result of an expensive computation. The cached value is reused until listed dependencies change.

```jsx
import { useMemo } from '@wordpress/element';

function FilteredList({ items, query }) {
  // Only re-filters when items or query changes
  const filtered = useMemo(
    () => items.filter(item =>
      item.title.toLowerCase().includes(query.toLowerCase())
    ),
    [items, query]
  );

  return <ul>{filtered.map(item => <li key={item.id}>{item.title}</li>)}</ul>;
}
```

**When to use:** Only when the computation is measurably expensive or when you need referential stability for an object passed to a memoised child component.

---

### `useCallback`

**Definition:** Returns a memoised function that only changes if listed dependencies change. Used to give stable function references to child components or effect dependency arrays.

```jsx
import { useState, useCallback } from '@wordpress/element';

function Parent() {
  const [count, setCount] = useState(0);

  // Without useCallback, a new function is created every render
  // causing Child to re-render unnecessarily
  const handleClick = useCallback(() => {
    setCount(c => c + 1);
  }, []); // no deps — function never needs to change

  return <Child onClick={handleClick} />;
}
```

---

### `useId`

**Definition:** Generates a stable, unique ID that is consistent between server and client renders. Use for accessibility: connecting `<label htmlFor>` with `<input id>`.

```jsx
import { useId } from '@wordpress/element';

function LabelledInput({ label }) {
  const id = useId();

  return (
    <>
      <label htmlFor={id}>{label}</label>
      <input id={id} type="text" />
    </>
  );
}
```

---

### `useReducer`

**Definition:** An alternative to `useState` for complex state with multiple sub-values or when next state depends on multiple fields together. Uses a reducer function: `(state, action) => newState`.

```jsx
import { useReducer } from '@wordpress/element';

const initialState = { loading: false, data: null, error: null };

function reducer(state, action) {
  switch (action.type) {
    case 'FETCH_START':  return { ...state, loading: true, error: null };
    case 'FETCH_SUCCESS': return { loading: false, data: action.payload, error: null };
    case 'FETCH_ERROR':  return { loading: false, data: null, error: action.error };
    default: return state;
  }
}

function DataFetcher() {
  const [state, dispatch] = useReducer(reducer, initialState);

  async function load() {
    dispatch({ type: 'FETCH_START' });
    try {
      const data = await apiFetch({ path: '/wp/v2/posts' });
      dispatch({ type: 'FETCH_SUCCESS', payload: data });
    } catch (error) {
      dispatch({ type: 'FETCH_ERROR', error });
    }
  }

  return (
    <div>
      {state.loading && <p>Loading…</p>}
      {state.error && <p>Error: {state.error.message}</p>}
      {state.data && <p>{state.data.length} posts loaded.</p>}
      <button onClick={load}>Load</button>
    </div>
  );
}
```

---

### Stale Closures

**Definition:** A stale closure occurs when a function "closes over" a value from a previous render, reading outdated state/props. This is a common bug with `useEffect` and `useCallback`.

```jsx
// BUG: stale closure — count is captured at 0 and never updates
useEffect(() => {
  const timer = setInterval(() => {
    console.log(count); // always logs 0!
    setCount(count + 1);
  }, 1000);
  return () => clearInterval(timer);
}, []); // count missing from deps

// FIX 1: Add count to deps (but resets timer each change)
useEffect(() => { /* ... */ }, [count]);

// FIX 2: Use functional update — no need to read count
useEffect(() => {
  const timer = setInterval(() => {
    setCount(prev => prev + 1); // reads fresh value internally
  }, 1000);
  return () => clearInterval(timer);
}, []); // correct: effect doesn't read count
```

---

## 4. Composition and Architecture

---

### Splitting Components

**Definition:** Breaking a large component into smaller, single-responsibility pieces. Each component should do one thing and do it well.

```jsx
// Before — monolithic component
function BlockEdit({ attributes, setAttributes }) {
  return (
    <div>
      <InspectorControls>
        <PanelBody title="Settings">
          {/* lots of controls... */}
        </PanelBody>
      </InspectorControls>
      <div className="hero">
        {/* complex markup... */}
      </div>
    </div>
  );
}

// After — split into focused components
function BlockEdit({ attributes, setAttributes }) {
  return (
    <>
      <BlockSidebar attributes={attributes} setAttributes={setAttributes} />
      <HeroPreview attributes={attributes} />
    </>
  );
}

function BlockSidebar({ attributes, setAttributes }) { /* inspector controls */ }
function HeroPreview({ attributes }) { /* the actual block UI */ }
```

---

### Lifting State

**Definition:** When two sibling components need to share state, move that state up to their closest common ancestor and pass it down as props.

```jsx
function Parent() {
  const [selectedId, setSelectedId] = useState(null);

  return (
    <>
      <ItemList
        onSelect={setSelectedId}
        selectedId={selectedId}
      />
      <ItemDetail id={selectedId} />
    </>
  );
}
```

---

### Custom Hooks

**Definition:** A function whose name starts with `use` that calls other hooks. Encapsulates reusable stateful logic.

```jsx
// Custom hook — reusable post-fetching logic
function usePost(postId) {
  const [post, setPost] = useState(null);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    if (!postId) return;
    setIsLoading(true);
    apiFetch({ path: `/wp/v2/posts/${postId}` })
      .then(data => { setPost(data); setIsLoading(false); })
      .catch(err => { setError(err); setIsLoading(false); });
  }, [postId]);

  return { post, isLoading, error };
}

// Usage in any component
function Edit({ attributes }) {
  const { post, isLoading } = usePost(attributes.postId);
  if (isLoading) return <Spinner />;
  return <div>{post?.title?.rendered}</div>;
}
```

---

### `useContext` + `createContext`

**Definition:** Context provides a way to share values across the component tree without passing props through every level (prop drilling).

```jsx
import { createContext, useContext, useState } from '@wordpress/element';

// 1. Create the context
const SettingsContext = createContext({ theme: 'light', lang: 'en' });

// 2. Provide it high up in the tree
function PluginRoot() {
  const [settings, setSettings] = useState({ theme: 'light', lang: 'en' });

  return (
    <SettingsContext.Provider value={{ settings, setSettings }}>
      <App />
    </SettingsContext.Provider>
  );
}

// 3. Consume it anywhere in the tree
function ThemeToggle() {
  const { settings, setSettings } = useContext(SettingsContext);

  return (
    <button onClick={() =>
      setSettings(s => ({ ...s, theme: s.theme === 'light' ? 'dark' : 'light' }))
    }>
      Current theme: {settings.theme}
    </button>
  );
}
```

---

### Portals

**Definition:** Render a component's output into a different DOM node than the parent — useful for modals, tooltips, and toast notifications that must escape overflow/z-index constraints.

```jsx
import { createPortal } from '@wordpress/element';

function Modal({ children, onClose }) {
  return createPortal(
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal-content" onClick={e => e.stopPropagation()}>
        {children}
        <button onClick={onClose}>Close</button>
      </div>
    </div>,
    document.body // render directly into <body>
  );
}
```

---

### Error Boundaries

**Definition:** Class components that catch JavaScript errors in their child tree, preventing the whole page from crashing. React does not yet support function component error boundaries.

```jsx
class BlockErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    console.error('Block error:', error, info);
  }

  render() {
    if (this.state.hasError) {
      return <p>This block encountered an error.</p>;
    }
    return this.props.children;
  }
}

// Usage — wrap risky components
<BlockErrorBoundary>
  <ComplexBlock />
</BlockErrorBoundary>
```

---

## 5. Gutenberg Blocks — Registration and Metadata

---

### `registerBlockType` + `block.json`

**Definition:** The modern way to register a Gutenberg block. `block.json` declares metadata; `registerBlockType()` ties JS behaviour to it.

**File structure:**

```
my-block/
  block.json   ← metadata
  index.js     ← JS registration
  edit.js      ← editor component
  save.js      ← saved HTML
```

**block.json:**

```json
{
  "$schema": "https://schemas.wp.org/trunk/block.json",
  "apiVersion": 3,
  "name": "myplugin/hero",
  "version": "1.0.0",
  "title": "Hero Block",
  "category": "text",
  "icon": "cover-image",
  "description": "A full-width hero section.",
  "keywords": ["hero", "banner", "header"],
  "textdomain": "my-plugin",
  "editorScript": "file:./index.js",
  "editorStyle": "file:./editor.css",
  "style": "file:./style.css",
  "attributes": {
    "title": { "type": "string", "default": "" },
    "align": { "type": "string", "default": "wide" }
  },
  "supports": {
    "html": false,
    "align": ["wide", "full"]
  }
}
```

**index.js:**

```js
import { registerBlockType } from '@wordpress/blocks';
import metadata from './block.json';
import Edit from './edit';
import save from './save';

registerBlockType(metadata.name, {
  ...metadata,
  edit: Edit,
  save,
});
```

---

### Attributes

**Definition:** Typed data slots for a block. WordPress serialises these to the post content and deserialises them on load.

**Attribute types:** `string`, `number`, `boolean`, `array`, `object`, `null`

**Sources** — where the value comes from:

```json
{
  "attributes": {
    "title": {
      "type": "string",
      "default": "Hello"
    },
    "url": {
      "type": "string",
      "source": "attribute",
      "selector": "a",
      "attribute": "href"
    },
    "content": {
      "type": "string",
      "source": "html",
      "selector": "p"
    },
    "items": {
      "type": "array",
      "source": "query",
      "selector": "ul li",
      "query": {
        "text": { "type": "string", "source": "text" }
      },
      "default": []
    }
  }
}
```

**In the editor:**

```jsx
function Edit({ attributes, setAttributes }) {
  // Read
  const { title, url } = attributes;

  // Write — always spread, always immutable
  const updateTitle = (newTitle) => setAttributes({ title: newTitle });

  return <TextControl value={title} onChange={updateTitle} />;
}
```

---

### `supports`

**Definition:** A set of built-in WordPress features you can opt a block into without writing custom controls.

```json
{
  "supports": {
    "html": false,
    "anchor": true,
    "align": ["wide", "full"],
    "color": {
      "background": true,
      "text": true,
      "gradients": true
    },
    "spacing": {
      "margin": true,
      "padding": true
    },
    "typography": {
      "fontSize": true,
      "lineHeight": true
    }
  }
}
```

WordPress automatically adds the corresponding inspector panels and wires the values into `useBlockProps()`.

---

### Block Styles

**Definition:** Alternative visual appearances for a block, selectable in the editor sidebar. Registered with a CSS class suffix.

```js
// PHP registration
register_block_style('core/button', [
  'name'  => 'outline',
  'label' => 'Outline',
]);

// JS registration
import { registerBlockStyle } from '@wordpress/blocks';

registerBlockStyle('core/quote', {
  name: 'large',
  label: 'Large',
});
```

When the "Large" style is selected, WordPress adds `is-style-large` to the block's wrapper class.

---

### Block Variations

**Definition:** Predefined configurations of a block, displayed as separate block inserter options.

```js
import { registerBlockVariation } from '@wordpress/blocks';

registerBlockVariation('core/columns', {
  name: 'two-columns-equal',
  title: 'Two Columns (Equal)',
  icon: 'columns',
  attributes: {},
  innerBlocks: [
    ['core/column'],
    ['core/column'],
  ],
  isDefault: false,
});
```

---

### Dynamic Blocks

**Definition:** Blocks whose front-end HTML is rendered by PHP at request time, not saved in the database. The editor uses a React component for preview; the front end uses a PHP `render_callback`.

**block.json:**

```json
{
  "render": "file:./render.php"
}
```

**render.php:**

```php
<?php
// $attributes array, $content string, $block WP_Block object are available
$title = esc_html($attributes['title'] ?? '');
?>
<div <?php echo get_block_wrapper_attributes(); ?>>
  <h2><?php echo $title; ?></h2>
  <?php echo $content; ?>
</div>
```

**edit.js** — show a live preview using `ServerSideRender`:

```jsx
import ServerSideRender from '@wordpress/server-side-render';
import { useBlockProps } from '@wordpress/block-editor';

export default function Edit({ attributes }) {
  return (
    <div {...useBlockProps()}>
      <ServerSideRender
        block="myplugin/my-dynamic-block"
        attributes={attributes}
      />
    </div>
  );
}
```

---

### `deprecated`

**Definition:** When you change a block's `save` output, existing posts have the old HTML. `deprecated` provides migration paths so old content still loads without a block validation error.

```js
registerBlockType('myplugin/hero', {
  // ... current definition
  save({ attributes }) {
    return <div className="hero-v2">{attributes.title}</div>;
  },
  deprecated: [
    {
      // v1 — old save function
      save({ attributes }) {
        return <div className="hero">{attributes.title}</div>;
      },
      // optional: migrate attributes from v1 format to v2
      migrate(attributes) {
        return { ...attributes };
      },
    },
  ],
});
```

---

## 6. Block Editor UI (`@wordpress/block-editor`)

---

### `useBlockProps`

**Definition:** Returns an object of props (className, data attributes, event handlers) that must be spread onto the block's root element in both `edit` and `save` functions. Required for the editor to properly identify and manage the block.

```jsx
import { useBlockProps } from '@wordpress/block-editor';

// edit.js
export default function Edit({ attributes }) {
  const blockProps = useBlockProps({
    className: 'my-custom-class',
    style: { backgroundColor: attributes.bgColor },
  });

  return (
    <div {...blockProps}>
      {/* block content */}
    </div>
  );
}

// save.js
export function save({ attributes }) {
  const blockProps = useBlockProps.save({
    className: 'my-custom-class',
  });

  return <div {...blockProps}>{/* static HTML */}</div>;
}
```

---

### `InnerBlocks` and `useInnerBlocksProps`

**Definition:** Allows a block to contain other blocks inside it — creating "container" patterns like columns, cards, or accordions.

```jsx
import { useBlockProps, useInnerBlocksProps, InnerBlocks } from '@wordpress/block-editor';

const ALLOWED_BLOCKS = ['core/paragraph', 'core/image', 'core/heading'];
const TEMPLATE = [
  ['core/heading', { level: 2, placeholder: 'Section title' }],
  ['core/paragraph', { placeholder: 'Write something…' }],
];

export default function Edit() {
  const blockProps = useBlockProps({ className: 'my-container' });
  const innerBlocksProps = useInnerBlocksProps(blockProps, {
    allowedBlocks: ALLOWED_BLOCKS,
    template: TEMPLATE,
    templateLock: false, // or 'all', 'insert'
  });

  return <div {...innerBlocksProps} />;
}

// save.js
export function save() {
  const blockProps = useBlockProps.save();
  const innerBlocksProps = useInnerBlocksProps.save(blockProps);
  return <div {...innerBlocksProps} />;
}
```

---

### `InspectorControls` and `BlockControls`

**Definition:**
- `InspectorControls` — renders controls into the right-hand Settings sidebar panel.
- `BlockControls` — renders controls into the floating toolbar above the selected block.

```jsx
import {
  useBlockProps,
  InspectorControls,
  BlockControls,
} from '@wordpress/block-editor';
import {
  PanelBody,
  PanelRow,
  ToggleControl,
  ToolbarGroup,
  ToolbarButton,
} from '@wordpress/components';

export default function Edit({ attributes, setAttributes }) {
  return (
    <>
      {/* Sidebar controls */}
      <InspectorControls>
        <PanelBody title="Display Settings" initialOpen={true}>
          <PanelRow>
            <ToggleControl
              label="Show title"
              checked={attributes.showTitle}
              onChange={val => setAttributes({ showTitle: val })}
            />
          </PanelRow>
        </PanelBody>
      </InspectorControls>

      {/* Toolbar controls */}
      <BlockControls>
        <ToolbarGroup>
          <ToolbarButton
            icon="edit"
            label="Edit content"
            onClick={() => setAttributes({ isEditing: true })}
          />
        </ToolbarGroup>
      </BlockControls>

      {/* The actual block */}
      <div {...useBlockProps()}>
        {attributes.showTitle && <h2>{attributes.title}</h2>}
      </div>
    </>
  );
}
```

---

### `RichText`

**Definition:** An editable, formatted text area inside the block. Supports inline formatting (bold, italic, links) and fires `onChange` with the resulting HTML string.

```jsx
import { RichText, useBlockProps } from '@wordpress/block-editor';

export default function Edit({ attributes, setAttributes }) {
  return (
    <div {...useBlockProps()}>
      <RichText
        tagName="h2"
        value={attributes.title}
        onChange={val => setAttributes({ title: val })}
        placeholder="Enter a title…"
        allowedFormats={['core/bold', 'core/italic']}
      />
    </div>
  );
}

// save.js — use RichText.Content to output the stored HTML
export function save({ attributes }) {
  return (
    <div {...useBlockProps.save()}>
      <RichText.Content tagName="h2" value={attributes.title} />
    </div>
  );
}
```

---

### `MediaUpload` / `MediaUploadCheck`

**Definition:** Opens the WordPress media library and returns the selected media item. `MediaUploadCheck` guards the UI — hides the upload button if the user lacks `upload_files` capability.

```jsx
import { MediaUpload, MediaUploadCheck, useBlockProps } from '@wordpress/block-editor';
import { Button } from '@wordpress/components';

export default function Edit({ attributes, setAttributes }) {
  const { mediaId, mediaUrl } = attributes;

  return (
    <div {...useBlockProps()}>
      <MediaUploadCheck>
        <MediaUpload
          onSelect={media => setAttributes({ mediaId: media.id, mediaUrl: media.url })}
          allowedTypes={['image']}
          value={mediaId}
          render={({ open }) => (
            <Button variant="secondary" onClick={open}>
              {mediaUrl ? 'Replace image' : 'Select image'}
            </Button>
          )}
        />
      </MediaUploadCheck>
      {mediaUrl && <img src={mediaUrl} alt="" />}
    </div>
  );
}
```

---

## 7. Editor Chrome and Data

### `useSelect` and `useDispatch`

**Definition:**
- `useSelect(selector)` — subscribes to a registered data store and returns derived state. Re-renders automatically when relevant store data changes.
- `useDispatch(storeName)` — returns action creators for a given store.

```jsx
import { useSelect, useDispatch } from '@wordpress/data';
import { store as editorStore } from '@wordpress/editor';
import { store as blockEditorStore } from '@wordpress/block-editor';

function PostStatusDisplay() {
  // Read from store
  const { postTitle, isSaving } = useSelect(select => ({
    postTitle: select(editorStore).getEditedPostAttribute('title'),
    isSaving:  select(editorStore).isSavingPost(),
  }), []);

  // Write to store
  const { editPost } = useDispatch(editorStore);

  return (
    <div>
      <p>Title: {postTitle} {isSaving && '(saving…)'}</p>
      <button onClick={() => editPost({ title: 'New Title' })}>
        Change title
      </button>
    </div>
  );
}
```

---

### Core Stores — Quick Reference

| Store constant | Package | Common selectors |
|---|---|---|
| `core` | `@wordpress/core-data` | `getEntityRecord`, `getEntityRecords` |
| `core/editor` | `@wordpress/editor` | `getCurrentPost`, `getEditedPostAttribute`, `isSavingPost` |
| `core/block-editor` | `@wordpress/block-editor` | `getBlocks`, `getSelectedBlock`, `getEditorSettings` |
| `core/edit-post` | `@wordpress/edit-post` | `isEditorPanelOpened`, `getPreference` |

---

### `core-data` — Entities

**Definition:** `@wordpress/core-data` provides a caching layer over the REST API. Use it to read and write posts, terms, users, and settings without rolling your own fetch logic.

```jsx
import { useSelect, useDispatch } from '@wordpress/data';
import { store as coreStore } from '@wordpress/core-data';

function PostSelector({ postId }) {
  const post = useSelect(
    select => select(coreStore).getEntityRecord('postType', 'post', postId),
    [postId]
  );

  const { saveEntityRecord } = useDispatch(coreStore);

  const updateTitle = async () => {
    await saveEntityRecord('postType', 'post', {
      id: postId,
      title: 'Updated from component',
    });
  };

  if (!post) return <Spinner />;
  return (
    <div>
      <p>{post.title.rendered}</p>
      <button onClick={updateTitle}>Update title</button>
    </div>
  );
}
```

---

## 8. `@wordpress/components` — Common UI

These pre-built components match the Gutenberg design system. Always prefer them over custom HTML in admin and editor UIs.

---

### Text and Choice Controls

```jsx
import {
  TextControl,
  TextareaControl,
  SelectControl,
  RadioControl,
  CheckboxControl,
  ToggleControl,
} from '@wordpress/components';

// Text input
<TextControl
  label="API Key"
  value={apiKey}
  onChange={setApiKey}
  type="password"
  help="Your plugin API key from Settings."
/>

// Dropdown select
<SelectControl
  label="Post Status"
  value={status}
  options={[
    { label: 'Draft',     value: 'draft' },
    { label: 'Published', value: 'publish' },
  ]}
  onChange={setStatus}
/>

// Toggle switch
<ToggleControl
  label="Enable feature"
  checked={isEnabled}
  onChange={setIsEnabled}
/>
```

---

### Numeric Controls

```jsx
import { RangeControl, UnitControl } from '@wordpress/components';

<RangeControl
  label="Columns"
  value={columns}
  onChange={val => setAttributes({ columns: val })}
  min={1}
  max={6}
/>

<UnitControl
  label="Gap"
  value={gap}
  onChange={setGap}
  units={[
    { value: 'px', label: 'px' },
    { value: 'rem', label: 'rem' },
  ]}
/>
```

---

### Color Controls

```jsx
import { ColorPalette, PanelColorSettings } from '@wordpress/block-editor';

// In InspectorControls
<PanelColorSettings
  title="Colour Settings"
  colorSettings={[
    {
      label: 'Background',
      value: attributes.bgColor,
      onChange: val => setAttributes({ bgColor: val }),
    },
    {
      label: 'Text',
      value: attributes.textColor,
      onChange: val => setAttributes({ textColor: val }),
    },
  ]}
/>
```

---

### Overlays — `Modal` and `Popover`

```jsx
import { useState } from '@wordpress/element';
import { Modal, Button } from '@wordpress/components';

function ConfirmDialog({ onConfirm }) {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <>
      <Button variant="primary" onClick={() => setIsOpen(true)}>
        Delete block
      </Button>
      {isOpen && (
        <Modal
          title="Confirm deletion"
          onRequestClose={() => setIsOpen(false)}
        >
          <p>Are you sure? This cannot be undone.</p>
          <Button
            variant="primary"
            isDestructive
            onClick={() => { onConfirm(); setIsOpen(false); }}
          >
            Delete
          </Button>
        </Modal>
      )}
    </>
  );
}
```

---

## 9. Internationalization (`@wordpress/i18n`)

**Definition:** The `@wordpress/i18n` package wraps GNU gettext functions. Every user-facing string in your plugin should pass through one of these functions so translators can localise it.

```jsx
import { __, _x, _n, sprintf } from '@wordpress/i18n';

// Basic translation
const label = __('Save settings', 'my-plugin');

// With context — disambiguate identical strings with different meanings
const post  = _x('Post', 'noun: a blog post',       'my-plugin');
const postV = _x('Post', 'verb: to publish content', 'my-plugin');

// Plural forms
const msg = _n(
  '%d item found',   // singular
  '%d items found',  // plural
  count,             // number to test
  'my-plugin'
);

// sprintf — insert values into translated strings
const greeting = sprintf(
  /* translators: %s is the user's display name */
  __('Welcome back, %s!', 'my-plugin'),
  userName
);
```

**Generating the POT file** (command to run once):

```bash
wp i18n make-pot . languages/my-plugin.pot --domain=my-plugin
```

---

## 10. REST API and Settings (`@wordpress/api-fetch`)

**Definition:** `@wordpress/api-fetch` is a thin wrapper around `fetch` pre-configured with WordPress authentication (cookie + nonce), base URL, and middleware. Use it for all REST API calls from admin JS.

---

### Basic Usage — GET, POST, DELETE

```js
import apiFetch from '@wordpress/api-fetch';

// GET
const posts = await apiFetch({ path: '/wp/v2/posts?per_page=5' });

// POST — create
const newPost = await apiFetch({
  path: '/wp/v2/posts',
  method: 'POST',
  data: { title: 'Hello World', status: 'draft' },
});

// PUT — update
const updated = await apiFetch({
  path: `/wp/v2/posts/${postId}`,
  method: 'PUT',
  data: { title: 'Updated Title' },
});

// DELETE
await apiFetch({
  path: `/wp/v2/posts/${postId}`,
  method: 'DELETE',
});
```

---

### Custom REST Routes

**PHP:**

```php
add_action('rest_api_init', function () {
  register_rest_route('myplugin/v1', '/settings', [
    'methods'             => 'GET',
    'callback'            => 'myplugin_get_settings',
    'permission_callback' => function () {
      return current_user_can('manage_options');
    },
  ]);

  register_rest_route('myplugin/v1', '/settings', [
    'methods'             => 'POST',
    'callback'            => 'myplugin_save_settings',
    'permission_callback' => function () {
      return current_user_can('manage_options');
    },
    'args' => [
      'api_key' => [
        'type'              => 'string',
        'sanitize_callback' => 'sanitize_text_field',
        'required'          => true,
      ],
    ],
  ]);
});
```

**JS — calling the custom route:**

```js
const settings = await apiFetch({ path: '/myplugin/v1/settings' });

await apiFetch({
  path: '/myplugin/v1/settings',
  method: 'POST',
  data: { api_key: '1234' },
});
```

---

### Plugin Options via `show_in_rest`

**PHP:**

```php
register_setting('myplugin_options', 'myplugin_api_key', [
  'type'         => 'string',
  'default'      => '',
  'show_in_rest' => true,
]);
```

**JS — read/write through the Settings endpoint:**

```js
// Read
const { myplugin_api_key } = await apiFetch({ path: '/wp/v2/settings' });

// Write
await apiFetch({
  path: '/wp/v2/settings',
  method: 'POST',
  data: { myplugin_api_key: newKey },
});
```

---

### Error Handling

```js
try {
  const result = await apiFetch({ path: '/myplugin/v1/settings', method: 'POST', data });
  setStatus('Saved!');
} catch (error) {
  // WordPress REST errors have: error.code, error.message, error.data.status
  console.error(error.code, error.message);
  setError(error.message);
}
```

---

## 11. Icons and Media

### `@wordpress/icons`

**Definition:** A curated set of SVG icons as React components, consistent with the Gutenberg UI.

```jsx
import { Icon, edit, trash, upload } from '@wordpress/icons';
import { Button } from '@wordpress/components';

// Inline icon
<Icon icon={edit} size={20} />

// Icon in a button
<Button icon={trash} label="Delete" isDestructive />

// Icon in registerBlockType
registerBlockType(metadata.name, {
  ...metadata,
  icon: upload, // pass the icon object directly
  edit: Edit,
  save,
});
```

---

### Responsive Preview

The editor has three preview modes — desktop, tablet, and mobile. If your block output changes significantly at different widths, test all three. You can read the current mode from the store:

```jsx
import { useSelect } from '@wordpress/data';
import { store as editPostStore } from '@wordpress/edit-post';

function Edit() {
  const deviceType = useSelect(
    select => select(editPostStore).__experimentalGetPreviewDeviceType(),
    []
  );

  return (
    <div className={`my-block my-block--preview-${deviceType.toLowerCase()}`}>
      {/* ... */}
    </div>
  );
}
```

---

## 12. Styling for Plugins

### Scoped CSS — Avoiding Clashes

Use a unique prefix (your plugin slug) for all class names:

```scss
// editor.scss — editor-only
.myplugin-hero {
  border: 2px dashed #ccc; // editor outline, not shown on front

  &__title {
    font-size: 1.5rem;
  }
}

// style.scss — front end + editor
.myplugin-hero {
  position: relative;
  overflow: hidden;

  &__title {
    color: var(--myplugin-title-color, #111);
  }
}
```

---

### Editor vs Front-end Styles

| File | When loaded | Purpose |
|---|---|---|
| `editor.scss` / `editor.css` | Block editor only | Editing chrome, fake placeholders |
| `style.scss` / `style.css` | Editor + front end | Actual block appearance |
| `view.js` + its CSS | Front end only | Interactive behaviour (e.g. slider JS) |

**Enqueuing (PHP):**

```php
// Modern approach — block.json handles this automatically
// "editorStyle": "file:./editor.css"
// "style":       "file:./style.css"
```

---

### CSS Custom Properties from PHP Attributes

```php
// render.php — write inline CSS from attribute values
$style = sprintf(
  '--hero-bg-color: %s; --hero-text-color: %s;',
  esc_attr($attributes['bgColor']  ?? '#fff'),
  esc_attr($attributes['textColor'] ?? '#000')
);
?>
<div <?php echo get_block_wrapper_attributes(['style' => $style]); ?>>
```

```scss
// style.scss — consume the custom property
.myplugin-hero {
  background-color: var(--hero-bg-color);
  color:            var(--hero-text-color);
}
```

---

### RTL Support

```scss
// Option 1: logical properties (modern, preferred)
.my-block {
  margin-inline-start: 1rem; // left in LTR, right in RTL
  padding-inline-end:  1rem;
}

// Option 2: separate rtl.css (generated by build)
// @wordpress/scripts can auto-generate this with postcss-rtl
```

---

## 13. Build Tooling (`@wordpress/scripts`)

### Setup

```bash
npm install --save-dev @wordpress/scripts
```

**package.json scripts:**

```json
{
  "scripts": {
    "start":  "wp-scripts start",
    "build":  "wp-scripts build",
    "lint:js": "wp-scripts lint-js",
    "lint:css": "wp-scripts lint-style"
  }
}
```

---

### Entry Points

By default `@wordpress/scripts` uses `src/index.js` as the entry point. For multiple blocks or an admin app, configure them explicitly:

```js
// webpack.config.js
const defaultConfig = require('@wordpress/scripts/config/webpack.config');

module.exports = {
  ...defaultConfig,
  entry: {
    'blocks/hero/index':  './src/blocks/hero/index.js',
    'blocks/cta/index':   './src/blocks/cta/index.js',
    'admin/settings':     './src/admin/settings.js',
  },
};
```

---

### `.asset.php` and Enqueueing

After `npm run build`, Webpack generates `build/index.asset.php` containing dependency handles and a version hash. Use this in PHP when enqueueing:

```php
$asset = include plugin_dir_path(__FILE__) . 'build/blocks/hero/index.asset.php';

wp_enqueue_script(
  'myplugin-hero-editor',
  plugin_dir_url(__FILE__) . 'build/blocks/hero/index.js',
  $asset['dependencies'],  // e.g. ['wp-blocks', 'wp-element', 'wp-block-editor']
  $asset['version'],
  true
);
```

---

### Webpack Externals

`@wordpress/*` packages are marked as Webpack externals — they resolve to global variables provided by WordPress (`wp.blocks`, `wp.element`, etc.) rather than being bundled. This prevents shipping a duplicate copy of React or Gutenberg libraries.

```js
// Your import:
import { registerBlockType } from '@wordpress/blocks';

// What Webpack resolves it to at runtime:
// window.wp.blocks.registerBlockType
```

---

## 14. Scripts Outside React

### `view.js` — Front-end Vanilla JS

Block front-end behaviour (sliders, accordions, tabs) often doesn't need React. Keep these scripts lightweight:

```js
// src/blocks/accordion/view.js — no JSX, no React
document.addEventListener('DOMContentLoaded', () => {
  const toggles = document.querySelectorAll('.myplugin-accordion__toggle');

  toggles.forEach(toggle => {
    toggle.addEventListener('click', () => {
      const panel = toggle.nextElementSibling;
      const isOpen = toggle.getAttribute('aria-expanded') === 'true';

      toggle.setAttribute('aria-expanded', String(!isOpen));
      panel.hidden = isOpen;
    });
  });
});
```

**Register in block.json:**

```json
{
  "viewScript": "file:./view.js"
}
```

---

### Event Delegation

For dynamically added elements, attach listeners to a stable parent:

```js
// Instead of attaching to each button (might not exist yet)
document.querySelector('.myplugin-list').addEventListener('click', (e) => {
  if (e.target.matches('.myplugin-list__item-btn')) {
    handleItemClick(e.target.dataset.id);
  }
});
```

---

### WordPress Interactivity API

An optional, modern WP API (WordPress 6.5+) for building declarative, reactive front-end behaviour without shipping a full React bundle:

```js
// view.js using Interactivity API
import { store, getContext } from '@wordpress/interactivity';

store('myplugin/accordion', {
  actions: {
    toggle() {
      const context = getContext();
      context.isOpen = !context.isOpen;
    },
  },
});
```

```html
<!-- render.php -->
<div data-wp-interactive="myplugin/accordion"
     data-wp-context='{"isOpen": false}'>
  <button data-wp-on--click="actions.toggle">Toggle</button>
  <div data-wp-bind--hidden="!context.isOpen">Content here</div>
</div>
```

---

## 15. Security and Content Safety

### Capabilities — `current_user_can`

Always check capabilities before performing sensitive operations in REST callbacks:

```php
register_rest_route('myplugin/v1', '/delete-data', [
  'methods'             => 'DELETE',
  'callback'            => 'myplugin_delete_data',
  'permission_callback' => function () {
    return current_user_can('manage_options'); // admins only
  },
]);
```

---

### Nonces

A nonce is a single-use token that verifies a request originated from your admin page:

```php
// PHP — output nonce for JS to use
wp_localize_script('myplugin-admin', 'mypluginData', [
  'nonce' => wp_create_nonce('wp_rest'),
]);
```

`@wordpress/api-fetch` automatically includes the REST nonce via the `X-WP-Nonce` header. For non-REST (admin-ajax) requests, verify manually:

```php
check_ajax_referer('myplugin_action', 'nonce');
```

---

### Sanitization vs Validation

| Concept | Purpose | PHP functions |
|---|---|---|
| **Sanitization** | Strip/transform unsafe input | `sanitize_text_field`, `sanitize_email`, `wp_kses_post` |
| **Validation** | Reject input that doesn't meet rules | `is_email()`, `absint()`, REST `args` schema |
| **Escaping** | Make output safe for its context | `esc_html`, `esc_attr`, `esc_url`, `wp_json_encode` |

```php
// Sanitize on save
update_option('myplugin_api_key', sanitize_text_field($_POST['api_key']));

// Escape on output
echo '<a href="' . esc_url($link) . '">' . esc_html($label) . '</a>';
```

---

### React and Escaping

React automatically escapes text content, so plain string values in JSX are safe:

```jsx
// Safe — React escapes this
<p>{userInput}</p>

// UNSAFE — bypasses React's escaping — only use with trusted/sanitized HTML
<p dangerouslySetInnerHTML={{ __html: trustedHtml }} />

// In blocks — use RawHTML only for known-safe or server-sanitized content
import { RawHTML } from '@wordpress/element';
<RawHTML>{sanitizedContent}</RawHTML>
```

---

## 16. Optional / Advanced

### SlotFill — Extending the Editor

**Definition:** A pattern to inject UI into named slots in the editor from a plugin, without modifying core code.

```jsx
import { Fill, Slot } from '@wordpress/components';
import { PluginSidebar, PluginSidebarMoreMenuItem } from '@wordpress/edit-post';

function MyPluginSidebar() {
  return (
    <>
      <PluginSidebarMoreMenuItem target="myplugin-sidebar">
        My Plugin
      </PluginSidebarMoreMenuItem>
      <PluginSidebar name="myplugin-sidebar" title="My Plugin">
        <p>Custom sidebar content here.</p>
      </PluginSidebar>
    </>
  );
}

// Register the plugin
import { registerPlugin } from '@wordpress/plugins';
registerPlugin('myplugin', { render: MyPluginSidebar });
```

---

### Custom Stores

**Definition:** Register a custom Redux-like store for complex cross-component state that doesn't belong in a single component's `useState`.

```js
import { createReduxStore, register } from '@wordpress/data';

const store = createReduxStore('myplugin/store', {
  reducer(state = { count: 0 }, action) {
    switch (action.type) {
      case 'INCREMENT': return { ...state, count: state.count + 1 };
      default: return state;
    }
  },
  actions: {
    increment: () => ({ type: 'INCREMENT' }),
  },
  selectors: {
    getCount: (state) => state.count,
  },
});

register(store);

// In components
const count = useSelect(select => select('myplugin/store').getCount(), []);
const { increment } = useDispatch('myplugin/store');
```

---

### Performance — Avoiding Unnecessary Re-renders

```jsx
import { memo } from '@wordpress/element';

// memo — skip re-render if props haven't changed
const ExpensiveList = memo(function ExpensiveList({ items }) {
  return <ul>{items.map(item => <li key={item.id}>{item.title}</li>)}</ul>;
});

// useSelect selector must be stable — don't create objects inline
// BAD — creates a new object every render, always triggers re-render
const data = useSelect(select => ({
  posts: select('core').getEntityRecords('postType', 'post'),
}), []);

// GOOD — destructure to primitive / stable references
const posts = useSelect(
  select => select('core').getEntityRecords('postType', 'post'),
  []
);
```

---

## 17. Suggested Study Order

Work through these checkpoints in order. Each builds on the previous.

| Step | Focus | Key outcome |
|---|---|---|
| 1 | ES modules + `async`/`await` + array methods | Comfortable with modern JS patterns |
| 2 | JSX + `useState` + controlled inputs | Can build a simple React form |
| 3 | `useEffect` + `apiFetch` | Can load REST data in a component |
| 4 | `registerBlockType` + `block.json` + `useBlockProps` + one `InspectorControls` | First working custom block |
| 5 | `RichText` or `MediaUpload` | Block accepts rich content |
| 6 | `useSelect` / `useDispatch` | Reads and writes editor data |
| 7 | Context + Portals | Cross-cutting UI (toasts, modals) |
| 8 | Build pipeline | Correct enqueue with `.asset.php`, no duplicate React |

---

## Quick Reference Cheatsheet

### Attribute Update Pattern

```js
// Always spread — never mutate attributes directly
setAttributes({ fieldName: newValue });
setAttributes({ ...attributes, nested: { ...attributes.nested, key: val } });
```

### Hook Dependency Rules

```js
useEffect(() => { /* effect */ }, []);         // once on mount
useEffect(() => { /* effect */ }, [id]);       // on mount + when id changes
useEffect(() => { /* effect */ });             // on every render — rarely correct
```

### `apiFetch` Quick Patterns

```js
// GET
const data = await apiFetch({ path: '/wp/v2/posts' });

// POST with body
await apiFetch({ path: '/myplugin/v1/settings', method: 'POST', data: { key: val } });

// Error handling
try { await apiFetch(/* ... */); } catch (e) { console.error(e.code, e.message); }
```

### Common `@wordpress/components` Controls

```jsx
<TextControl   label="…" value={v} onChange={fn} />
<SelectControl label="…" value={v} options={[{label, value}]} onChange={fn} />
<ToggleControl label="…" checked={b} onChange={fn} />
<RangeControl  label="…" value={n} min={1} max={10} onChange={fn} />
```

---

*End of study guide. Revisit each section after shipping a feature that uses it. Add your own notes and links to [official WordPress block editor handbook](https://developer.wordpress.org/block-editor/).*
