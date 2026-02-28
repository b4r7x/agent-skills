# AI Coding Prompt Guide

A structured template for generating precise, actionable AI coding prompts.

## Prompt Template

```
**Role:** [Who the AI should be — be specific]
**Context:** [Project stack + relevant files with exact paths]
**Task:** [Precise, single-sentence description of what to do]
**Constraints:** [What NOT to do, style rules, size limits]
**Output format:** [Code only / with explanation / specific file structure]
**Acceptance criteria:** [What must be true after the task completes]
**References:** [Specific file paths: src/auth/login.ts not "the auth file"]
```

## Field Guidelines

### Role
Be specific. Not "a developer" but "a senior TypeScript engineer working on a NestJS REST API".
Match persona to the tech stack found in the project.

Examples:
- `Senior TypeScript engineer building a NestJS REST API with Prisma`
- `React 19 frontend developer working on a TanStack Router SPA`
- `Python backend engineer maintaining a FastAPI service with async SQLAlchemy`
- `Go systems engineer working on a CLI tool`

### Context
Always include:
- Framework + language + key libraries (from package.json / pyproject.toml / go.mod)
- Relevant file paths (discovered during research phase)
- Important architectural patterns (e.g., "uses registry pattern", "ESM-only monorepo")

### Task
One precise action verb sentence. Avoid vague words (improve, fix, update).

Good: `Refactor the authentication middleware in src/middleware/auth.ts to validate JWT tokens using the jose library instead of jsonwebtoken`

Bad: `Fix the auth stuff`

### Constraints
- What NOT to change (public API contracts, interfaces, test file structure)
- Style rules from the project (`strict: true`, `noUncheckedIndexedAccess`, etc.)
- Size limits (`max 50 lines`, `no new dependencies`)
- Specific patterns to follow (e.g., "use existing error handling pattern from src/errors/")

### Output Format
Choose one:
- `Code only, no explanation` — for simple mechanical changes
- `Modified file(s) only with inline comments explaining changes` — for complex refactors
- `Diff format` — when showing what changed matters
- `Step-by-step plan, then implementation` — for architectural changes

### Acceptance Criteria
Testable conditions:
- `All existing tests in __tests__/ continue to pass`
- `No new TypeScript errors`
- `The function signature stays unchanged`
- `Response time under 200ms for the common path`

### References
Always use exact file paths discovered in the research phase:
- `src/auth/middleware.ts` not "the middleware"
- `packages/cli/src/commands/add.ts` not "the add command"
- Reference line numbers when relevant: `src/registry.ts:45-82`

## Anti-Patterns to Avoid in Generated Prompts

| Anti-pattern | Fix |
|---|---|
| Vague file references ("the config file") | Use exact paths found during research |
| Multiple tasks in one prompt | One task per prompt |
| No constraints | Always specify what NOT to do |
| Missing acceptance criteria | Add at least one testable condition |
| Generic role ("a developer") | Match role to actual project stack |
| "Just make it better" | Define specific measurable improvement |

## Prompt Length Guidelines

- **Simple task** (1 file, mechanical change): 5-8 lines total
- **Medium task** (refactor, multi-file): 10-15 lines
- **Complex task** (architecture, new feature): 15-25 lines with references section

Always prefer concrete over comprehensive. A shorter precise prompt outperforms a long vague one.
