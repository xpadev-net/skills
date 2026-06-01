---
name: task-pr-worker
description: Execute one delegated repository task in a dedicated worktree from implementation through validation, deep-review self-review, subagent-assisted review, PR creation, review-hook iteration, merge, and orchestrator notification. Use when Codex is acting as a worker thread for a narrowly scoped coding task delegated by an orchestrator.
---

# Task PR Worker

## Core Role

Complete exactly one delegated task in your own worktree and branch. Keep scope narrow. Do not take over orchestration, start unrelated tasks, or edit outside the delegated ownership unless required and explicitly justified.

When available, use subagents proactively for bounded research, implementation checks, and review. Do not let subagents expand the task scope or edit outside the delegated ownership.

## Startup Checklist

1. Read the delegation prompt and repository instructions.
2. Load any required local harness or workflow skill when available.
3. Confirm current worktree, branch, and git status.
4. Create or checkout the requested branch before editing.
5. If the task ledger is unavailable in the worktree, treat the delegation prompt as the authoritative task text.
6. Record a short plan for non-trivial work when the repository workflow expects one.

Use the repository shell wrapper when local instructions require it. Avoid watch mode and interactive commands.

## Implementation Rules

- Edit only the delegated files/directories.
- Keep the change as small as the task allows.
- Do not revert other users' changes.
- Avoid shared/high-conflict files unless essential.
- Do not add secrets, real credentials, production identifiers, or environment-specific private data.
- Preserve existing behavior outside the acceptance criteria.
- If the task is too large, crosses ownership boundaries, or produces a broad diff, stop and ask the orchestrator to split and reassign it. Provide a concrete decomposition, suggested branches, file ownership for each slice, and any partial work status.

If user or orchestrator follow-up narrows scope, obey the newest instruction and remove unnecessary prior changes.

## Validation

Run targeted tests first, then the repository-required checks. When local instructions specify exact commands, use those commands. Common final checks are:

```bash
pnpm lint:fix
pnpm type-check
pnpm test -- --silent
```

If a full check is inappropriate for docs-only or fixture-only work, document the waiver, run meaningful alternatives such as secret scans or targeted lint checks, and explain why the waiver is safe.

## Review

Before PR creation, perform self-review with `$deep-review` when that skill is available. Use it against the final diff and the task acceptance criteria, and fix valid findings.

Also get an independent review pass when tools allow it. Ask the reviewer to inspect only the final diff and task acceptance criteria. Fix valid findings and repeat until no required findings remain.

Do not count your own self-review as the independent review unless no reviewer tool exists; if so, state the limitation.

## PR and Hook Loop

1. Stage and commit coherent changes with normal hooks enabled.
2. Push the branch without force pushing unless explicitly allowed and safe.
3. Create a PR with `gh`.
4. Run `gh-review-hook <PR-number>`.
5. If the hook reports findings, fix them, commit, push, and rerun.
6. If the hook reports the branch is behind, merge the base branch normally; do not rewrite history when the branch already has an open PR.
7. Repeat until `gh-review-hook` exits 0.
8. After hook exit 0, verify no unpushed or unstaged changes remain, then merge the PR.
9. After merge, send or otherwise provide a completion message to the orchestrator thread with the final report details.

If push appears stale, verify the remote ref before retrying. Never merge a PR while local required fixes are unpushed.

## Final Report

Report in the repository's requested language. Include:

- PR URL and number.
- Merge commit.
- Commits or important branch heads if relevant.
- Targeted and full validation results.
- Independent review result.
- `gh-review-hook` exit 0.
- Any waiver or residual risk.

Do not mark the task complete unless the PR is merged or the delegation explicitly allowed a non-merge stopping point.

If you are blocked or request decomposition, report that to the orchestrator immediately instead of continuing to make speculative changes. Include the reason, current validation state, whether any branch/PR exists, and the recommended replacement tasks.
