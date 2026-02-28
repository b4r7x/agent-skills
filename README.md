# agent-skills

Skills for GitHub Copilot CLI, Claude Code, and other agent-based tools.

## Install all skills

```bash
npx skills add b4r7x/agent-skills
```

## Install a specific skill

```bash
npx skills add b4r7x/agent-skills@improve-prompt
```

## Skills

### [`human-commit`](./human-commit/)

Reads your staged (or unstaged) git diff and generates 3 commit message options that sound like a developer wrote them — not AI.

**Triggers:** "commit message", "what should I write for commit", "human-like commit", "wiadomość do commita"

**What it does:**
1. Reads `git diff --cached` (falls back to `git diff HEAD` if nothing staged)
2. Identifies what changed and why
3. Outputs 3 short, lowercase, imperative-mood options — no `feat:` prefixes, no filler

---

### [`humanize-readme`](./humanize-readme/)

Rewrites a `README.md` to strip out AI-generated patterns — buzzwords, generic openers, fake enthusiasm — and replace them with direct, honest prose.

**Triggers:** "humanize readme", "remove AI slop", "fix the README", "make it sound human"

**What it does:**
1. Reads the README and audits it against a list of banned phrases and structural patterns
2. Rewrites each section in plain, specific language — keeps code blocks and commands untouched
3. Outputs the full rewritten README ready to copy

---

### [`improve-prompt`](./improve-prompt/)

Takes a rough prompt idea and turns it into a structured AI coding prompt, after researching your actual project context first.

**Triggers:** "improve this prompt", "refine my prompt", "ulepszony prompt", "dopracuj prompt", rough prompt ideas

**What it does:**
1. Interprets your intent (task type, language/framework, expected output)
2. Researches the current project (stack, file structure, conventions, git history)
3. Outputs a structured prompt with: Role, Context, Task, Constraints, Output format, Acceptance criteria, and specific file references

## Contributing

Each skill is a directory at the root of this repo. See [skills.sh](https://skills.sh/) for the ecosystem.
