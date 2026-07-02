---
name: task-pr-worker
description: Execute one delegated repository task in a dedicated worktree from implementation through validation, deep-review self-review, subagent-assisted review, PR creation, review-hook iteration, merge, and orchestrator notification. Use when Codex is acting as a worker thread for a narrowly scoped coding task delegated by an orchestrator.
---

# Task PR Worker

## Core Role

Complete exactly one delegated task in your own worktree and branch. Keep scope narrow. Do not take over orchestration, start unrelated tasks, or edit outside the delegated ownership unless required and explicitly justified.

When available, use subagents proactively for bounded research, implementation checks, and review. Do not let subagents expand the task scope or edit outside the delegated ownership.

## Repository Trust Boundary

Before pushing PR lifecycle work, identify the GitHub repository owner from the current remote or the delegation prompt.

Only repositories owned by `xpadev-net` may use the default autonomous PR lifecycle: create PRs, run review hooks, and merge after required checks pass.

For any repository not owned by `xpadev-net`, do not create a PR or merge a PR unless the user or orchestrator explicitly approved that action for the current repository. If approval is missing, complete implementation, validation, and review, then stop before PR creation or merge and report to the orchestrator that user approval is required.

## Startup Checklist

1. Read the delegation prompt and repository instructions.
2. Load any required local harness or workflow skill when available.
3. Confirm current worktree, branch, and git status.
4. Create or checkout the requested branch before editing.
5. If the task ledger is unavailable in the worktree, treat the delegation prompt as the authoritative task text.
6. Record a short plan for non-trivial work when the repository workflow expects one.

Use the repository shell wrapper when local instructions require it. Avoid watch mode and interactive commands.

## Orchestrator Reporting Requirements

Before stopping work for any reason, report to the orchestrator thread when an orchestrator thread or handoff channel is available. Do this even when the next action is simply to wait for direction or approval.

Report immediately, then stop and wait for direction, when:

- A required precondition is missing or unclear, including branch/worktree setup, task ownership, task ledger access, credentials, repository trust approval, validation tooling, or required review tooling.
- A judgment call is needed that could change scope, risk, ownership, PR lifecycle, validation requirements, or acceptance criteria.
- The task is blocked, needs decomposition, exceeds delegated ownership, or cannot continue without user or orchestrator input.
- Work reaches an allowed non-merge stopping point, including approval-required pauses for non-`xpadev-net` repositories.
- The task is complete, the PR is merged, or the delegation explicitly allowed stopping without merge.

Include the reason for stopping, current branch or PR state, local change state, validation and review evidence collected so far, concrete options or recommended next action, and any risks of waiting. If no orchestrator messaging tool is available, put the same report in the final response and state that the orchestrator thread could not be contacted directly.

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

## Out-of-Scope Review Findings

When a reviewer, `$deep-review`, or `gh-review-hook` raises a finding outside the delegated task scope, do not silently expand the PR. Report the finding to the orchestrator and ask whether it should be filed as separate work or handled in the current PR. Include the finding text, why it appears out of scope, the affected files/domains, and any risk of leaving it unfixed.

If the orchestrator instructs you to file or track it separately and not address it in the current PR, leave the implementation unchanged for that finding and add a short note to the PR description stating that the finding is out of scope and will be handled separately. Keep the note factual and reference the orchestrator direction when possible.

## PR and Hook Loop

Apply this loop only when the repository is owned by `xpadev-net` or PR creation and merge were explicitly approved for the current non-`xpadev-net` repository.

1. Stage and commit coherent changes with normal hooks enabled.
2. Push the branch without force pushing unless explicitly allowed and safe.
3. Create a PR with `gh`.
4. Run `gh-review-hook <PR-number>`.
5. If the hook reports in-scope findings, fix them, commit, push, and rerun. For out-of-scope findings, follow the out-of-scope review finding process before deciding whether to modify the PR.
6. If the hook reports the branch is behind, merge the base branch normally; do not rewrite history when the branch already has an open PR.
7. Repeat until `gh-review-hook` exits 0 or the 30-iteration hook limit is reached.
8. After hook exit 0, verify no unpushed or unstaged changes remain, then merge the PR.
9. After merge, send or otherwise provide a completion message to the orchestrator thread with the final report details before stopping.

If push appears stale, verify the remote ref before retrying. Never merge a PR while local required fixes are unpushed.

Do not report blocked only because `gh-review-hook` has not exited 0. Treat hook findings as ordinary review iteration: fix valid findings, commit, push, and rerun.

After 30 fix-and-review iterations without `gh-review-hook` exit 0, stop local iteration and ask the orchestrator to inspect the current state. Report this as a stopped hook-iteration-limit review, not as blocked. Include the latest hook output summary, the recurring or changing findings, the current implementation summary, PR URL/number, branch head, validation evidence, independent review evidence, and whether any local or unpushed changes remain. Wait for orchestrator direction before continuing, decomposing, or merging.

## Final Report

Report in the repository's requested language. Include:

- PR URL and number.
- Merge commit.
- Commits or important branch heads if relevant.
- Targeted and full validation results.
- Independent review result.
- `gh-review-hook` exit 0, or the orchestrator-directed outcome after a 30-iteration hook limit.
- Any out-of-scope findings deferred by orchestrator direction and where they are documented in the PR description.
- Any waiver or residual risk.

Do not mark the task complete unless the PR is merged or the delegation explicitly allowed a non-merge stopping point.

For non-`xpadev-net` repositories where PR creation or merge was not explicitly approved, report the ready branch or diff state, validation evidence, review evidence, and the exact approval needed next.

If you are blocked or request decomposition for a reason other than repeated non-zero `gh-review-hook` results, report that to the orchestrator immediately instead of continuing to make speculative changes. Include the reason, current validation state, whether any branch/PR exists, and the recommended replacement tasks.
