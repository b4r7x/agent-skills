---
name: human-commit
description: Generates human-like git commit messages based on staged or unstaged changes. Reads git diff, analyzes what changed, and outputs 3 natural commit message options that sound like they were written by a developer — not AI. This skill should be used when the user wants a commit message, asks "what should I write for commit", "generate commit message", "human like commit", "wiadomość do commita", or just asks for help committing.
---

# Human Commit

Reads git changes and generates commit messages that sound like a real developer wrote them.

## Workflow

### Step 1 — Read the diff

```bash
git diff --cached 2>/dev/null
```

If output is empty (nothing staged), fall back to:

```bash
git diff HEAD 2>/dev/null
```

Also check what files are involved:

```bash
git status --short 2>/dev/null
```

### Step 2 — Analyze the changes

Identify:
- What files changed and in what area (auth, UI, deps, config, tests, etc.)
- The nature of the change (fix, add, remove, move, update, refactor)
- Whether there's an obvious "why" (bug, cleanup, new feature, maintenance)

### Step 3 — Generate 3 options

Output exactly 3 commit message options. Each on its own line, in a code block.

Rules for human-like messages:
- **Lowercase** subject line
- **Subject ~50 chars** (50/72 rule — up to 72 is fine, aim for brevity)
- **No conventional commit prefixes** (`feat:`, `refactor:`, `chore:`) unless it reads naturally
- **Imperative mood** — "fix X" not "fixed X" or "fixes X"
- **Skip obvious filler** — not "update code to fix the issue with the login"
- **Optional body** — add a blank line + 1-2 lines of context only if the change needs explanation (lines ≤72 chars)

Anti-patterns to avoid:
- ❌ `implement JWT token validation middleware with expiry checking`
- ❌ `refactor: extract authentication logic into separate service module`
- ❌ `fix: resolve issue with user authentication token expiration handling`
- ✅ `fix tokens not expiring`
- ✅ `move auth to its own file`
- ✅ `bump deps`
- ✅ `add dark mode toggle`
- ✅ `fix login redirect on mobile`

## Output Format

```
Option 1: fix tokens not expiring
Option 2: auth middleware wasn't checking expiry
Option 3: fix expired jwt tokens still passing through
```

If one option genuinely benefits from a body, show it like:

```
Option 2: reorganize auth module

pulled jwt logic out, the file was getting too long
```

Pick the best option as Option 1.
