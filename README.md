# agent-skills

A collection of skills for AI coding agents — GitHub Copilot CLI, Claude Code, and other agent-based tools.

## Install all skills

```bash
npx skills add b4r7x/agent-skills
```

## Install a specific skill

```bash
npx skills add b4r7x/agent-skills@improve-prompt
```

## Skills

### [`improve-prompt`](./improve-prompt/)

Takes a rough, unpolished prompt idea and transforms it into a precise, structured AI coding prompt — with automatic project context research.

**Triggers:** "improve this prompt", "refine my prompt", "ulepszony prompt", "dopracuj prompt", rough prompt ideas

**What it does:**
1. Interprets your intent (task type, language/framework, expected output)
2. Researches the current project (stack, file structure, conventions, git history)
3. Outputs a structured AI coding prompt with: Role, Context, Task, Constraints, Output format, Acceptance criteria, and specific file references

## Contributing

Each skill lives in its own directory at the root of this repo. See [skills.sh](https://skills.sh/) for the ecosystem.
