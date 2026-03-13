---
name: anti-slop
description: Audits source code files for AI-generated slop patterns — unnecessary comments, over-engineering, defensive over-coding, AI voice markers, dead code, type workarounds, and verbose patterns. Outputs a structured report with line references and severity. Use when the user wants to audit code for AI slop, check code quality, find unnecessary comments, detect over-engineering, or asks "audit this", "check for slop", "anti-slop", "review for AI patterns".
metadata:
  author: b4r7x
  version: "1.0.0"
  argument-hint: <file-paths-or-globs>
---

# Anti-Slop

Audits source code for AI-generated slop patterns and outputs a structured report.

## Workflow

### Step 1 — Resolve files

Parse the argument as file paths or glob patterns. Expand globs to a concrete file list.

```bash
# If argument is a glob like "src/**/*.ts", expand it
# If argument is a file path, use it directly
# If multiple paths/globs, expand all
```

**Supported extensions:** `.ts`, `.tsx`, `.js`, `.jsx`, `.py`, `.go`, `.rs`, `.java`, `.rb`, `.php`, `.swift`, `.kt`

Skip files larger than 1000 lines — flag them as "skipped (too large)" in the report. For those, audit only the first 500 lines.

If no argument is provided, prompt the user:
> "Which files should I audit? Pass file paths or glob patterns (e.g. `src/**/*.ts`)."

### Step 2 — Read the pattern catalog

Read `references/slop-code-patterns.md` for the full detection rules and severity guide.

### Step 3 — Audit each file

For each file:

1. Read the file contents
2. Scan for patterns across all 7 categories:
   - **Cat 1: Unnecessary comments** — restating code, obvious JSDoc, vague TODOs, section dividers, "removed" markers
   - **Cat 2: Over-engineering** — premature abstractions, unnecessary factories, config for hardcoded values, forwarding wrappers, one-off interfaces
   - **Cat 3: Defensive over-coding** — null checks on non-nullable types, try-catch on infallible ops, redundant else-after-return, verbose error messages, fallbacks hiding bugs
   - **Cat 4: AI voice** — "enhanced/optimized/improved" naming, "ensure/robust/graceful" comments, control-flow narration, apologetic comments
   - **Cat 5: Dead code** — unused imports, assigned-but-unread variables, unreachable code, backwards-compat re-exports
   - **Cat 6: Type workarounds** — `as any`, unexplained `@ts-ignore`, unnecessary assertions
   - **Cat 7: Verbose patterns** — ternary replaceable by `??`, boolean verbosity, unnecessary async/await, pointless spread

3. For each issue found, record:
   - **Line number(s)**
   - **Category** (1-7)
   - **Pattern** (e.g., "1a: restating code")
   - **Severity** (high / medium / low)
   - **The offending code** (1-3 lines)
   - **Suggested fix** (brief — what to do, not the full rewrite)

### Step 4 — Cross-file checks

After individual file audits, check for cross-file patterns:

- **Functions called from only one file** — may be premature abstractions (Cat 2a)
- **Exported symbols not imported elsewhere** — may be dead code (Cat 5d)
- **Duplicated utility functions** across files — AI often creates the same helper in multiple places

Only run these if auditing 3+ files. Use `grep` to search for function/export usage across the codebase.

### Step 5 — Output the report

Format the report as follows:

```
## Anti-Slop Audit Report

**Files audited:** N
**Issues found:** N (H high, M medium, L low)

---

### `path/to/file.ts`

| Line | Severity | Category | Pattern | Issue |
|------|----------|----------|---------|-------|
| 12 | medium | Unnecessary comment | 1a: restating code | `// set the name` above `this.name = name` |
| 45-52 | medium | Over-engineering | 2a: premature abstraction | `formatDate()` called once, body is one line |
| 89 | high | Type workaround | 6a: `as any` | `response.data as any` — fix the response type |

**Suggested fixes:**
- L12: Delete the comment
- L45-52: Inline `formatDate()` at L78
- L89: Type `response.data` as `ApiResponse` based on the fetch call at L85

---

### `path/to/other-file.ts`
...

---

### Cross-File Issues

- `formatSlug()` defined in `utils.ts:34` is only imported by `page.ts:2` — consider inlining
- `UserDTO` exported from `types.ts:12` is not imported anywhere

---

### Summary by Category

| Category | Count |
|----------|-------|
| 1. Unnecessary comments | N |
| 2. Over-engineering | N |
| 3. Defensive over-coding | N |
| 4. AI voice | N |
| 5. Dead code | N |
| 6. Type workarounds | N |
| 7. Verbose patterns | N |
```

If zero issues are found, output:
```
## Anti-Slop Audit Report

**Files audited:** N
**Issues found:** 0

Clean. No slop detected.
```
