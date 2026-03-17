# Hook Patterns from Top React Libraries (2025–2026)

## Pattern Summary

| Library | Function stability | State pattern | Consumer callbacks |
|---|---|---|---|
| **react-hook-form** | `useRef` — functions created once in `createFormControl` | `Proxy` on `formState` — property tracking | Stored in ref internally |
| **TanStack Query** | `useCallback` only for `useSyncExternalStore` subscribe | `trackResult` — property-access tracking | Accepts raw, ignores stability |
| **ahooks** | `useMemoizedFn` — wrapper in `useRef`, absolutely stable | `useCreation` instead of `useMemo` | `useLatest` internally |
| **SWR** | `useCallback` + `useLatest` on options | `useMemo` on returned object | Through `configRef` (`useLatest`) |
| **Mantine** | No memoization — plain function recreated each render | Plain `useState` | Called directly, no ref |

## Key Insight

No top library forces consumers to `useCallback` on callbacks passed as options. They store them internally in a ref (`useLatest` / `useMemoizedFn`) or simply accept instability.

## Mantine `useUncontrolled` (8M+ downloads/week)

The simplest correct implementation of controlled/uncontrolled state:

```ts
function useUncontrolled<T>({ value, defaultValue, finalValue, onChange }) {
  const [uncontrolled, setUncontrolled] = useState(defaultValue ?? finalValue);
  const controlled = value !== undefined;

  const handleChange = (next: T) => {
    if (!controlled) setUncontrolled(next);
    onChange?.(next);
  };

  return [controlled ? value! : uncontrolled, handleChange, controlled];
}
```

- No refs, no `useCallback`, no `useMemo`
- `handleChange` is a new reference every render — and that's fine
- Stability is the consumer's responsibility (e.g. `useMemo` on context value)

## TkDodo on useCallback Cascades (July 2025)

From real Sentry code — three nested `useCallback` were useless because one dep (`attachments.filter`) created a new array every render, breaking the entire chain:

```ts
const screenshots = attachments.filter(/* new array EVERY render */);
const paginateItems = useCallback(() => { use(screenshots) }, [screenshots]); // useless
const paginateHotkeys = useMemo(() => [...], [paginateItems]); // useless
const onKeyDown = useCallback(() => { run(paginateHotkeys) }, [paginateHotkeys]); // useless
```

> "It breaks way too often. It's not worth it." — TkDodo

## useEffectEvent (React 19)

Official React solution for "stable function + fresh values" — but **only inside effects**:

```ts
const onConnected = useEffectEvent(() => {
  showNotification('Connected!', theme); // always fresh
});

useEffect(() => {
  connection.on('connected', onConnected);
  // ...
}, [roomId]); // theme NOT in deps
```

Limitations:
- Cannot be passed to other components or hooks
- Cannot be used as event handler props
- Solves the problem only within `useEffect` scope

## The React API Gap (as of 2026)

There is **no official React API** that gives both:
- Stable function reference (same `===` across renders)
- Always-fresh values (no stale closure)

...outside of `useEffect`. Libraries work around this with `ref.current = value` during render (conscious trade-off: theoretical tearing in Concurrent Mode vs practical SSR breakage).

For `"use client"` components, `useLayoutEffect` eliminates the SSR problem, making ref-based patterns safe.
