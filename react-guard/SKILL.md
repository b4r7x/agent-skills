---
name: react-guard
description: React code validation guard. Loads React-specific skills and validates patterns, hooks, and component architecture.
---

# React Guard

Validates React code against best practices. Can be used standalone for any React project (no Next.js required).

## When to Use

- After writing or modifying React components (`.tsx`/`.jsx`)
- Before committing React code changes
- When reviewing React PRs

## Workflow

### Phase 1: Load Skills

Load ALL of the following skills using the Skill tool:

1. `code-quality` — universal quality principles
2. `react-senior-guide` — hook router and cross-cutting principles
3. `react-patterns` — React 19 patterns, Server Components, Actions
4. `react-anti-patterns` — 18 anti-patterns by detection difficulty
5. `context-memoization-guide` — memoization decision framework
6. `vercel-react-best-practices` — performance optimization rules

### Phase 2: Validate

For each changed `.tsx`/`.jsx` file, check:

**BLOCKER:**
- Components defined inside other components
- `useEffect` for derived state (compute during render instead)
- Missing `useEffect` cleanup (timers, subscriptions, listeners)
- Hooks called conditionally or after early returns
- State mutation (must create new references)
- `&&` with numbers that could be 0

**WARNING:**
- `useCallback` without `memo()` on child
- Boolean explosion (multiple boolean states instead of union type)
- Props mirroring in state via useEffect
- Missing loading/error/empty states
- Stale closures in setTimeout/setInterval/async
- God components (>200 lines mixing fetch + logic + UI)
- Unnecessary memoization (measure before optimizing)

**INFO:**
- key={index} on dynamic lists
- Manual fetch instead of React Query (or Server Components)
- Granular useState that should be useReducer

### Phase 3: Report

Output findings grouped by severity with file:line references.
