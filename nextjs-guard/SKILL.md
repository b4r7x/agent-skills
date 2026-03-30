---
name: nextjs-guard
description: Next.js code validation guard. Loads Next.js-specific skills and validates routing, RSC boundaries, and data patterns.
---

# Next.js Guard

Validates Next.js App Router code against best practices. Requires React knowledge (loads react-guard).

## When to Use

- After writing or modifying Next.js routes, layouts, middleware, or server code
- Before committing Next.js code changes
- When reviewing Next.js PRs

## Workflow

### Phase 1: Load Skills

Load ALL of the following skills using the Skill tool:

1. `react-guard` — React validation (loads its own React skills)
2. `next-best-practices` — file conventions, RSC, async patterns, data patterns
3. `nextjs` — comprehensive Next.js reference
4. `routing-middleware` — Vercel routing middleware patterns
5. `tailwind-patterns` — Tailwind CSS v4 (if className/CSS changed)

### Phase 2: Validate

For each changed file, check:

**BLOCKER:**
- `'use client'` on a component that doesn't need interactivity
- Business logic in route files (`app/`) — should be in `features/`
- Async client components (invalid in RSC)
- Non-serializable props passed from Server to Client components
- Missing Zod validation in Server Actions

**WARNING:**
- Data waterfalls (sequential fetches that could be parallel)
- Missing `revalidatePath()` before `redirect()` in Server Actions
- Throwing exceptions from Server Actions (return errors as data)
- Heavy components not using `next/dynamic` for lazy loading
- Barrel file imports (import directly instead)

**INFO:**
- Missing metadata/OG configuration
- Images not using `next/image`
- Scripts not using `next/script`

### Phase 3: Report

Output findings grouped by severity with file:line references.
