---
name: nextjs-supabase-guard
description: Full-stack validation guard for Next.js + Supabase projects. Orchestrates code-quality, react-guard, nextjs-guard, and Supabase-specific skills.
---

# Next.js + Supabase Guard

Full-stack validation orchestrator. Loads composable sub-guards and Supabase-specific skills.

## Composable Architecture

This skill is built from independent layers that can also be used standalone:

| Layer | Skill | Standalone? |
|---|---|---|
| Universal | `code-quality` | Yes — any project, any language |
| React | `react-guard` | Yes — any React project |
| Next.js | `nextjs-guard` | Yes — any Next.js project |
| Supabase | This skill | No — requires all above |

## When to Use

- During or after implementing any feature in a Next.js + Supabase project
- After writing or modifying components, actions, queries, middleware
- Before committing — as a final quality gate
- When reviewing your own work

## Workflow

### Phase 1: Load Context

Read the git diff to understand what changed:

```bash
git diff --stat
git diff
```

Also read `CLAUDE.md` in the project root — it contains project-specific conventions that override general best practices.

### Phase 2: Load Skills by Domain

Load skills based on what changed. Always start with the universal layer.

**Always load:**
1. `code-quality` — DRY, KISS, YAGNI, SRP principles

**React (if any .tsx/.jsx changed):**
2. `react-guard` — loads all React skills automatically

**Next.js (if routes, layouts, middleware, or server code changed):**
3. `nextjs-guard` — loads Next.js + React skills automatically

**Supabase + Auth (if auth, queries, or DB-related code changed):**
4. `nextjs-supabase-auth`
5. `supabase-postgres-best-practices`
6. `security-review`

**Styling (if className or CSS changed):**
7. `tailwind-patterns`

**Tests (if test files changed or new code lacks tests):**
8. `test-behavior-not-implementation`

**Code simplification (always):**
9. `anti-slop`
10. `code-simplifier`

### Phase 3: Validate Changes

For each changed file, check against the loaded skills. Report issues grouped by severity:

**BLOCKER** — Must fix before merge:
- Security issues (auth bypass, injection, exposed secrets)
- Broken dependency rule (app/ importing from features/ internals, shared code importing features/)
- `'use client'` on a component that doesn't need interactivity
- Business logic in route files (app/)
- Missing Zod validation in Server Actions
- Hardcoded colors instead of Tailwind theme classes
- DRY violations: 3+ occurrences of the same pattern without extraction
- Throwing exceptions from Server Actions (return errors as data)

**WARNING** — Should fix:
- Anti-patterns from loaded React/Next.js skills
- Over-engineering or AI slop (unnecessary abstractions, defensive code)
- Missing `revalidatePath()` before `redirect()` in actions
- Relative imports going up more than one level
- Memoization issues (unnecessary useMemo/useCallback, missing where needed)
- KISS violations: overly complex solutions for simple problems
- YAGNI violations: building features before they're needed

**INFO** — Consider:
- Simplification opportunities from `code-simplifier`
- Performance patterns from `vercel-react-best-practices`
- Testing gaps
- SRP violations: functions/components doing too many things

### Phase 4: Report

Output a structured report:

```
## Guard Report

### Files reviewed
- list of files

### BLOCKER (X)
- [file:line] description — skill reference

### WARNING (X)
- [file:line] description — skill reference

### INFO (X)
- [file:line] description — skill reference

### Verdict
PASS / PASS WITH WARNINGS / BLOCKED
```

## Key Rules from CLAUDE.md to Always Check

These are the most commonly violated project conventions:

1. **Dependency rule**: `app/ -> features/ -> components/, lib/, types/`
2. **Thin route files**: No business logic in `app/` — import and render only
3. **Feature public API**: Cross-feature imports MUST go through `index.ts`
4. **No relative imports** going up more than one level
5. **Server Components by default** — `'use client'` only when needed
6. **Zod validation** in all Server Actions
7. **Return errors as data** from Server Actions, not exceptions
8. **kebab-case** for all files and directories
9. **`@/` path alias** always — never `../../`
10. **No hardcoded hex colors** — use Tailwind theme classes
11. **DRY** — extract patterns repeated 3+ times
12. **KISS** — simplest solution that works
13. **SRP** — one responsibility per function/component
