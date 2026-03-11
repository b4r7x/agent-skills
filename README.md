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

---

### [`deep-plan`](./deep-plan/)

Takes a rough idea and autonomously turns it into an implementation plan. Researches your project, asks clarifying questions, then generates and executes its own prompt to produce a structured plan with todos.

**Triggers:** "plan this", "deep plan", "turn this into a plan", "zaplanuj to", "zrób plan", vague feature requests

**What it does:**
1. Decomposes the rough input — identifies scope, ambiguities, and files to analyze
2. Deep-dives into the project (reads files, maps dependencies, understands patterns)
3. Asks clarifying questions one at a time (multiple-choice, max ~5)
4. Generates an internal prompt for itself, then executes it
5. Produces a structured implementation plan with file changes, todos, and considerations

Best used in plan mode — saves plan.md and populates SQL todos automatically.

---

## React Skills

A suite of 8 React skills extracted from senior-level patterns, anti-patterns, and decision guides. Use individually or start with `react-senior-guide` as an entry point.

### [`react-senior-guide`](./react-senior-guide/) — Start here

Comprehensive React reference that routes to all 7 specialized skills below. Includes cross-cutting principles and an AI code review checklist (15 checks).

**Triggers:** writing or reviewing any React code

---

### [`react-usecallback`](./react-usecallback/)

When to use useCallback — React Compiler impact, the memo() requirement, valid use cases (useEffect deps, custom hooks, context).

**Triggers:** useCallback usage, memoization decisions, "should I wrap this in useCallback?"

---

### [`react-usememo`](./react-usememo/)

When to use useMemo — 4 valid cases (expensive computation, stable ref for memo'd child, useEffect dep, context value), decision heuristic.

**Triggers:** useMemo usage, performance optimization, "should I memoize this?"

---

### [`react-usecontext`](./react-usecontext/)

Context patterns — when to use context, value optimization (useMemo + useCallback + memo), compound components, alternatives table.

**Triggers:** useContext, context re-renders, compound components, prop drilling decisions

---

### [`react-useref`](./react-useref/)

useRef vs useState — 4 valid cases (DOM access, mutable values, stable callbacks, external libs), useEffectEvent replacement, decision table.

**Triggers:** useRef usage, "should this be state or ref?", DOM manipulation, timer IDs

---

### [`react-useeffect`](./react-useeffect/)

When useEffect is needed — decision tree, 6 valid cases, 6 anti-patterns, dependency pitfalls, cleanup checklist.

**Triggers:** useEffect usage, "is this effect necessary?", derived state via effect, missing cleanup

---

### [`react-design-patterns`](./react-design-patterns/)

12 React patterns ranked by 2025 popularity — Custom Hook, Compound Components, Headless, Container/Presentational, Provider, Render Props, Children as Function, Props Getters, Error Boundary, Portal, HOC (legacy), Atomic Design.

**Triggers:** "which pattern should I use?", component architecture decisions, design system patterns

---

### [`react-anti-patterns`](./react-anti-patterns/)

18 anti-patterns organized by detection difficulty — stale closures, state mutation, boolean explosion, component inside component, useEffect abuse, and more. Includes a code review checklist.

**Triggers:** code review, AI-generated React code review, "what's wrong with this code?"

---

## Contributing

Each skill is a directory at the root of this repo. See [skills.sh](https://skills.sh/) for the ecosystem.
