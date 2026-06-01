# Codex Skills

This repository stores local Codex skills so they can be versioned with Git.

## Skills

- `deep-review`: review pull requests and code changes using perspective-based heuristics.
- `task-pr-orchestrator`: coordinate delegated task PR workers through review, hook, and merge.
- `task-pr-worker`: execute one delegated task through implementation, validation, review, PR, hook, and merge.

## Layout

Each skill directory mirrors the install layout under `~/.codex/skills/<skill-name>/`.

```text
<skill-name>/
  SKILL.md
  agents/
  references/
```

To install or refresh a skill manually, copy the desired directory to `~/.codex/skills/`.
