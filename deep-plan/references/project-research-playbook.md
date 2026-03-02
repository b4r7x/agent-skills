# Project Research Playbook

Systematic approach to gathering project context before generating a refined prompt.

## Research Priority Order

Run these checks from highest to lowest signal — stop when you have enough context.

### 1. Project Identity (always run)

```bash
# Project overview and conventions
cat README.md 2>/dev/null | head -80
cat CLAUDE.md 2>/dev/null | head -60
cat AGENTS.md 2>/dev/null | head -60
cat .github/copilot-instructions.md 2>/dev/null | head -40
cat .cursorrules 2>/dev/null | head -40
ls .cursor/rules/ 2>/dev/null | head -10
```

Extracts: project purpose, tech stack, coding conventions, commands, agent-specific rules.

### 2. Stack Detection (always run)

```bash
# Node.js / JS / TS
cat package.json 2>/dev/null | python3 -c "import sys,json; d=json.load(sys.stdin); print('name:', d.get('name')); print('deps:', list(d.get('dependencies',{}).keys())[:10]); print('devDeps:', list(d.get('devDependencies',{}).keys())[:8])"

# Python
cat pyproject.toml 2>/dev/null | head -40
cat requirements.txt 2>/dev/null | head -20

# Go
cat go.mod 2>/dev/null | head -20

# Rust
cat Cargo.toml 2>/dev/null | head -20
```

Extracts: framework, key libraries, language version.

### 3. Directory Structure (always run)

```bash
# Top-level structure
ls -la

# Source directory (2 levels)
find src -maxdepth 2 -type d 2>/dev/null || find . -maxdepth 2 -name "*.ts" -o -name "*.py" -o -name "*.go" | head -30
```

Extracts: monorepo vs single package, key source directories.

### 4. Recent Activity (run if task context is unclear)

```bash
git log --oneline -10 2>/dev/null
git diff --name-only HEAD~3 HEAD 2>/dev/null | head -20
```

Extracts: recent changes, areas of active development, relevant modified files.

### 5. TypeScript Config (run for TS projects)

```bash
cat tsconfig.json 2>/dev/null | python3 -c "import sys,json; d=json.load(sys.stdin); print(json.dumps(d.get('compilerOptions',{}), indent=2))" 2>/dev/null | head -30
```

Extracts: strict mode flags, module resolution, path aliases.

### 6. File-Specific Research (run when task mentions specific functionality)

```bash
# Find relevant files by keyword
grep -r "keyword" src/ --include="*.ts" -l 2>/dev/null | head -10

# Read specific file
cat src/path/to/relevant/file.ts | head -80
```

Use glob to find files by pattern, grep to find files by content.

## What to Extract Per Research Step

| Research | Key Extractions |
|---|---|
| README/CLAUDE.md | Framework, commands, conventions, do-not-touch areas |
| package.json | Framework (React/Next/NestJS/etc.), test runner, build tool |
| Directory structure | Module boundaries, naming patterns, co-location conventions |
| Git log | What's being actively changed, which files are "hot" |
| tsconfig | `strict`, `noUncheckedIndexedAccess`, path aliases |
| Source files | Existing patterns to follow, function signatures to preserve |

## Stack-Specific Signals

### TypeScript / Node.js
- `"type": "module"` in package.json → ESM, use `.js` in imports
- `strict: true` in tsconfig → mention in constraints
- Presence of `zod` → use for validation
- Presence of `prisma` → use its patterns for DB access

### React / Next.js
- Next.js version (13+ = App Router likely)
- `tailwindcss` → use Tailwind, avoid inline styles
- `shadcn` components → follow shadcn patterns

### Python
- `fastapi` → async first, use dependency injection
- `sqlalchemy` → check if async or sync version
- `pydantic` → use for validation/schemas

### Monorepo signals
- `pnpm-workspace.yaml` / `nx.json` / `turbo.json` → identify which package is relevant
- `packages/` or `apps/` dirs → be specific about which package to modify

## Minimum Viable Research

If time-constrained, run just these three:

```bash
cat README.md 2>/dev/null | head -50
cat package.json 2>/dev/null | grep -E '"(name|dependencies|devDependencies)"' -A 10 | head -30
find . -maxdepth 2 -type d | grep -v node_modules | grep -v .git | head -20
```
