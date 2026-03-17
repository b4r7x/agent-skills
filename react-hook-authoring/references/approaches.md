# Approach A vs Approach B — When to Use Which

## Approach A — Simple (Mantine-style)

Default choice. No memoization in the hook. Stability is handled at the consumer level.

```ts
"use client";
import { useState } from "react";

export function useControllableState<T>({
  value,
  defaultValue,
  onChange,
}: {
  value?: T;
  defaultValue: T;
  onChange?: (value: T) => void;
}) {
  const [internal, setInternal] = useState(defaultValue);
  const controlled = value !== undefined;
  const current = controlled ? value! : internal;

  const setValue = (next: T | ((prev: T) => T)) => {
    const resolved = typeof next === "function"
      ? (next as (prev: T) => T)(current)
      : next;
    if (!controlled) setInternal(resolved);
    onChange?.(resolved);
  };

  return [current, setValue, controlled] as const;
}
```

**Properties:**
- `setValue` is a new reference every render
- No `useCallback`, no `useRef`, no `useLayoutEffect`
- `current` in closure is always fresh (no stale closure risk) because `setValue` is recreated each render
- Updater pattern `setValue((prev) => ...)` uses `current` from closure, not React scheduler's `prev` — equivalent when `setValue` is not called twice in the same handler

**Use when:**
- Building any hook that returns a setter function
- Consumer stabilizes via `useMemo` on context value (most common)
- No measured performance problem exists

**Consumer pattern:**
```ts
const [value, setValue] = useControllableState({ ... });

// setValue in useMemo deps is fine — useMemo recomputes when setValue changes,
// which happens when the component re-renders anyway
const contextValue = useMemo(
  () => ({ value, onChange: setValue }),
  [value, setValue],
);
```

## Approach B — Stable setValue (when profiler demands it)

Use only when Approach A causes measured performance issues AND `setValue` must be referentially stable (e.g. deeply nested context consumers re-rendering unnecessarily).

```ts
"use client";
import { useState, useCallback, useRef, useLayoutEffect } from "react";

export function useControllableState<T>({
  value,
  defaultValue,
  onChange,
}: {
  value?: T;
  defaultValue: T;
  onChange?: (value: T) => void;
}) {
  const [internal, setInternal] = useState(defaultValue);
  const isControlled = value !== undefined;
  const current = isControlled ? value! : internal;

  const ref = useRef({ current, isControlled, onChange });
  useLayoutEffect(() => {
    ref.current = { current, isControlled, onChange };
  });

  const setValue = useCallback((next: T) => {
    if (!Object.is(next, ref.current.current)) {
      if (!ref.current.isControlled) setInternal(next);
      ref.current.onChange?.(next);
    }
  }, []); // stable forever

  return [current, setValue, isControlled] as const;
}
```

**Properties:**
- `setValue` has stable reference (empty deps)
- `useLayoutEffect` updates ref after commit — safe in Concurrent Mode
- Requires `"use client"` (useLayoutEffect causes SSR warnings without it)
- Consumer's `onChange` does not need to be memoized — stored in ref

**Use when:**
- Profiler shows unnecessary re-renders from context consumers
- `setValue` is in `useMemo` deps of a context serving many consumers
- Approach A was tried first and proved insufficient

## Decision

```
Is there a measured performance problem?
  ├─ No → Approach A
  └─ Yes
      └─ Is setValue in context useMemo deps with many consumers?
          ├─ No → Approach A (problem is elsewhere)
          └─ Yes → Approach B
```

## Why NOT ref-during-render

```ts
// ❌ React docs: "Do not write ref.current during rendering"
const ref = useRef(value);
ref.current = value; // during render — tearing risk in Concurrent Mode
```

Libraries (ahooks, Radix, react-hook-form) do this because they must work without `"use client"` and cannot use `useLayoutEffect` (SSR warnings). With `"use client"`, use `useLayoutEffect` instead — it's safe and React-compliant.
