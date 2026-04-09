---
name: code-audit
description: Comprehensive codebase quality audit using parallel agents. Checks DRY, SRP, anti-slop, naming, file organization, type safety, error handling, patterns, dead code, architecture, and reusability. Produces findings report + fix plan for multi-agent execution. Use when the user wants to audit code quality, review architecture, check for smells, run a quality check, or says "audit", "code audit", "quality check", "review codebase".
metadata:
  author: b4r7x
  version: "1.0.0"
  argument-hint: <number-of-agents> [scope]
---

# Code Audit

Comprehensive codebase quality audit using parallel explore agents. Always checks everything. Agent count and scope are user-controlled.

## Arguments

Parse the user's input for:
- **Agent count** (required) -- how many parallel explore agents to launch. The user provides this number.
- **Scope** (optional) -- one of:
  - `full` (default) -- entire codebase
  - `changed` -- only files changed since last commit (`git diff --name-only HEAD`)
  - `staged` -- only staged files (`git diff --cached --name-only`)
  - `branch` -- files changed on current branch vs main (`git diff --name-only main...HEAD`)
  - `<path>` -- specific directory or glob pattern (e.g. `src/engine/`)

If the user provides just a number, use `full` scope. If they provide a path, use that as scope.

Examples:
- `/code-audit 10` -- 10 agents, full codebase
- `/code-audit 5 changed` -- 5 agents, only changed files
- `/code-audit 15 src/engine/` -- 15 agents, only engine directory
- `/code-audit 3 branch` -- 3 agents, files changed on current branch

## Workflow

Execute all phases in strict order. Do not skip phases.

### Phase 1 -- Understand the Project

Before launching agents, gather project context:

1. Read `CLAUDE.md`, `AGENTS.md`, `.cursorrules`, or equivalent project instruction files
2. Read `package.json` (or equivalent manifest) for tech stack
3. Run `find . -maxdepth 2 -type d | grep -v node_modules | grep -v .git | head -30` for structure overview
4. Check `git log --oneline -5` for recent activity

Extract:
- **Tech stack** (language, framework, test runner, linter)
- **Project conventions** (from CLAUDE.md -- e.g. "zero classes", "ESM imports", "zero memoization", "colocated tests")
- **Architecture pattern** (monorepo, feature-based, layer-based, etc.)

This context is passed to every agent in Phase 2.

### Phase 2 -- Resolve Scope

Based on the scope argument, produce a concrete file list:

```
full     -> all source files (exclude node_modules, dist, .git, lockfiles, generated)
changed  -> git diff --name-only HEAD
staged   -> git diff --cached --name-only
branch   -> git diff --name-only main...HEAD (or master)
<path>   -> glob expand the path
```

Count the files. This determines how to distribute work across agents.

### Phase 3 -- Distribute and Launch Explore Agents

Split the work across N agents (the user's requested count). Each agent gets:

1. **A subset of files or a focus area** -- distribute evenly by file count or by logical grouping (directory, feature area)
2. **The full project context** from Phase 1 (tech stack, conventions, architecture)
3. **The complete audit checklist** (below)

**Agent distribution strategy:**
- If N <= 5: assign each agent a broad area (e.g. "core/", "engine/", "components/", "hooks+stores", "utils+ui")
- If N <= 15: assign by feature/module (e.g. "orchestrator", "implementers", "planners", "config", "pickers", etc.)
- If N > 15: assign by individual file groups (3-8 files per agent) plus dedicated cross-cutting agents

**Always reserve at least 2 agents for cross-cutting analysis** (regardless of N):
- 1 agent for **DRY analysis** -- search for duplicate patterns across the ENTIRE scope, not just its assigned files
- 1 agent for **Architecture/SRP analysis** -- check module boundaries, import graph, file placement, naming consistency

**Agent prompt template:**

Each explore agent receives this prompt structure:
```
You are auditing code quality in [project name] at [project path].

Project context:
- Tech stack: [from Phase 1]
- Conventions: [from Phase 1]
- Architecture: [from Phase 1]

Your assigned files: [file list]

Read each file and evaluate against the FULL audit checklist below.
Report findings with specific file:line references and severity (critical/high/medium/low).
Group findings by category. Be thorough but only flag real issues -- no false positives.

[Full audit checklist -- see below]
```

Launch ALL agents in a single message (parallel). Use `subagent_type: "Explore"` and `model: "opus"`.

### Phase 4 -- Synthesize Findings

After ALL agents complete:

1. Collect all findings from all agents
2. Deduplicate (different agents may flag the same issue)
3. Sort by severity: critical > high > medium > low
4. Group by category (DRY, SRP, anti-slop, naming, etc.)
5. For each finding, include:
   - **Category** (DRY, SRP, naming, etc.)
   - **Severity** (critical/high/medium/low)
   - **Location** (file:line)
   - **Description** (what's wrong)
   - **Suggested fix** (brief, actionable)

**Do NOT bloat the main context with raw agent output.** Synthesize into a concise findings report.

Present the findings to the user. Ask:
- "Do you want me to create an execution plan for fixing these?" (default: yes)
- If the user has specific comments or disagreements, incorporate them

### Phase 5 -- Generate Fix Plan

If the user approves, create a structured fix plan:

1. **Group fixes into independent batches** -- fixes that touch non-overlapping files can run in parallel
2. **Order batches by dependency** -- moves/renames before DRY extractions before component refactoring
3. **For each batch, define agent instructions** -- what files to modify, what specifically to change
4. **Include verification step** -- run tests + typecheck + lint after each batch

Write the plan to the plan file (if in plan mode) or present inline.

**Plan structure:**
```markdown
# Code Audit Fix Plan

## Context
[Why this audit was run, scope, findings summary]

## Phase N: [Batch Name] (X parallel agents)

**Agent 1 -- [focus]:**
- [file:line] -- [specific change]
- [file:line] -- [specific change]

**Agent 2 -- [focus]:**
- ...

### Verification
npm test && npm run typecheck && npm run lint
```

### Phase 6 -- Execute (if approved)

If the user approves the plan, execute it:

1. Launch agents for each phase/batch in parallel (using the plan's agent instructions)
2. After each batch, verify: tests + typecheck + lint
3. If verification fails, fix before proceeding
4. Report results after each batch
5. Final comprehensive verification after all batches complete

---

## Audit Checklist

Every agent checks ALL of these against its assigned files. Cross-cutting agents check items marked with [cross-cutting] across the full scope.

### 1. DRY -- Don't Repeat Yourself [cross-cutting]

- [ ] Duplicate or near-duplicate functions across files
- [ ] Copy-pasted logic between similar modules
- [ ] Repeated type definitions or interfaces
- [ ] Similar error handling patterns that could be unified
- [ ] Duplicate string constants or magic values
- [ ] Same validation/guard pattern repeated 3+ times
- [ ] Re-export chains that add unnecessary indirection (consumers should import from source)

### 2. SRP -- Single Responsibility [cross-cutting]

- [ ] Files doing too many things (>200 lines is a smell, not a rule)
- [ ] Functions with "and" in their description
- [ ] Modules mixing concerns (e.g. I/O + validation + transformation in one file)
- [ ] God components that wire too many things together
- [ ] Grab-bag utility files with unrelated functions

### 3. Anti-Slop -- AI-Generated Patterns

- [ ] Comments restating what code does ("// Create X" before create call)
- [ ] Section dividers ("// --- section ---")
- [ ] JSDoc on internal/obvious functions
- [ ] Over-descriptive variable names
- [ ] Unnecessary type annotations where TypeScript infers
- [ ] "robust", "seamless", "leverage", "ensure" in comments (AI voice)
- [ ] Defensive checks on values that can't be null/undefined
- [ ] Try-catch blocks catching impossible errors
- [ ] Fallback values that never trigger

### 4. Naming & Conventions

- [ ] File name doesn't match primary export (e.g. `use-terminal-size.ts` exporting `useResponsiveLayout`)
- [ ] Stuttered paths (e.g. `commands/commands.ts`)
- [ ] Inconsistent naming across similar modules
- [ ] Names that mislead about content (e.g. "types/" containing runtime logic)
- [ ] Project convention violations (from CLAUDE.md)

### 5. File Organization [cross-cutting]

- [ ] Files in wrong directory based on content/consumers
- [ ] Domain logic in `utils/` that belongs in `core/` or feature module
- [ ] Pure logic in UI directories
- [ ] Test importing across architectural boundaries
- [ ] Re-exports that should be direct imports

### 6. Type Safety

- [ ] `as` type assertions that bypass validation
- [ ] `any` types that could be narrowed
- [ ] Missing discriminated union exhaustiveness checks
- [ ] Type aliases that add no semantic value
- [ ] Branded types without constructors

### 7. Error Handling

- [ ] Inconsistent error patterns (some throw, some return, some log)
- [ ] Raw `throw new Error()` where structured errors exist
- [ ] Empty catch blocks without justification
- [ ] Error messages that don't help debugging
- [ ] Missing error handling at system boundaries

### 8. Dead Code & Redundancy

- [ ] Unused exports (exported but never imported)
- [ ] Dead re-exports (re-exported but nobody imports from that path)
- [ ] Unused imports
- [ ] Commented-out code
- [ ] Variables assigned but never read
- [ ] Redundant overrides that duplicate base behavior

### 9. Patterns & Best Practices

- [ ] React: hook rules violations, stale closures, unnecessary refs
- [ ] React: convention violations (useMemo/useCallback/memo when project bans them)
- [ ] React: forwardRef/useImperativeHandle when project bans them
- [ ] React: Context for state when project uses external stores
- [ ] Class usage when project is zero-class (Error subclasses are OK)
- [ ] Mutable state where immutable patterns are expected
- [ ] Callback naming inconsistency across similar interfaces

### 10. Architecture [cross-cutting]

- [ ] Circular dependencies
- [ ] Wrong-direction imports (e.g. core importing from UI)
- [ ] Tight coupling between modules that should be independent
- [ ] Missing abstraction boundaries
- [ ] Factory patterns that could be simplified
- [ ] Configuration objects being built with fake/dummy data just to satisfy types

### 11. Reusability [cross-cutting]

- [ ] Patterns appearing 3+ times that should be extracted
- [ ] Similar helper functions in different modules doing the same thing
- [ ] Shared business logic duplicated between test and source
- [ ] Constants defined in multiple places instead of one source of truth

### 12. Performance (flag only clear issues)

- [ ] Synchronous I/O in hot paths
- [ ] Unnecessary re-computation on every render (when framework provides alternatives)
- [ ] N+1 patterns in data fetching
- [ ] Missing early returns causing unnecessary work

---

## Severity Guide

| Severity | Definition | Example |
|----------|-----------|---------|
| Critical | Bugs, security issues, boundary violations, wrong behavior | Core module importing from UI layer |
| High | Real code smells that will cause maintenance pain | 5x copy-pasted logic, files in wrong directory |
| Medium | Improvements that make code cleaner | Unnecessary comments, naming mismatches, redundant re-exports |
| Low | Nitpicks and style preferences | Could rename for clarity, minor ternary simplification |
