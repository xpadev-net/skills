---
name: task-pr-orchestrator
description: Coordinate a backlog of repository tasks by splitting safe work into separate background threads/worktrees, tracking task status in a task ledger, closing completed threads, handling worker-requested decomposition, and ensuring each worker reaches PR creation, independent review, review-hook success, and merge. Use when Codex is asked to orchestrate many coding tasks without directly editing implementation code.
---

# Task PR Orchestrator

## Core Role

Act as the parent-thread orchestrator. Do not implement product code in the parent thread. Delegate implementation to small worker threads, each using its own worktree and branch.

Allowed parent mutations are limited to the task ledger or other files explicitly permitted by the user after the current request. Never stage, commit, push, or merge from the parent thread unless the user explicitly asks for parent-owned git work.

## Operating Loop

1. Read the task ledger and identify statuses such as unstarted, in progress, split in progress, stopped, blocked, and complete.
2. Check open PRs with `gh pr list/view` and read active worker threads.
3. For completed worker reports, verify the PR is merged, record PR number, merge commit, and review-hook exit 0 in the ledger, then archive/close the worker thread.
4. For blocked worker reports, record the blocker only when it is concrete and cannot be resolved by ordinary worker iteration.
5. For still-running workers, leave them alone unless a stale push, missing PR, or ambiguous blocker needs a short follow-up.
6. Start additional unstarted tasks only when their files, domains, or dependencies do not conflict with active workers.
7. Keep tasks small. Split broad tasks by route group, provider, UI area, validation surface, or user workflow whenever that reduces review risk or avoids conflicts.
8. If a worker reports that a task is too large or should be decomposed, close/stop that worker thread, update the ledger with the reason, then create new worker threads for the smaller split tasks.

Do not sit in a manual polling loop. For long-running worker activity, create or update a recurring automation/heartbeat when the platform supports it, and let that scheduled callback perform the next orchestration check. Prefer automation over repeated sleeps, busy waiting, or frequent manual status checks.

## Delegation Rules

When creating a worker thread, include:

- Repository/workspace target and requirement to use a separate worktree.
- Exact task ID/title and bounded scope.
- Branch name.
- Files or directories the worker may touch.
- Files or domains the worker must not touch.
- Requirement to load repo instructions and any available orchestration harness.
- Requirement to use the repository shell wrapper when one is specified by local instructions.
- Requirement to avoid reverting others' changes.
- Requirement to avoid shared/high-conflict files unless essential.
- Required validation commands.
- Requirement for independent review before PR.
- Requirement for worker self-review to use `$deep-review` when that skill is available.
- Requirement for workers to use subagents actively for research, implementation support, and review when available.
- Requirement for the worker to message or report back to the orchestrator thread when the task is merged, blocked, stopped, or needs decomposition.
- Requirement for the worker to stop and request orchestrator decomposition instead of continuing when scope becomes too large, crosses ownership boundaries, or creates a broad diff.
- Requirement to create a PR, run the review hook until exit 0, push fixes as normal commits, and merge after hook success.
- Final report requirements: PR URL, merge commit, validation evidence, review evidence, and review-hook exit 0.

Prefer a new worker over expanding a running worker when the next task is independent. Prefer waiting when the next task touches the same files or depends on a running task.

## Split Requests

Treat a worker split request as an orchestration event:

1. Read the worker's proposed decomposition and current diff status.
2. Decide whether to keep any completed narrow work, abandon the partial work, or ask the worker to produce a smaller PR first.
3. Stop the oversized worker thread once the path is clear.
4. Update the ledger to show the original task as split/stopped and list the replacement slices.
5. Create new worker threads, one per slice, with separate branches and non-overlapping ownership.
6. Tell replacement workers not to reuse broad partial changes unless explicitly instructed.

## Ledger Updates

Update the ledger whenever orchestration state changes:

- Mark newly delegated work as `in progress` with thread ID and branch.
- Mark split work as `split in progress` and list completed and remaining slices.
- Mark stopped work with the user-approved reason.
- Mark complete only after current evidence proves merge completion and required hook success.

Do not claim completion from memory. Verify against current PR state, worker final report, or both.

## Thread Hygiene

Close/archive worker threads promptly after they are merged, stopped by user decision, or replaced by a new split. If archiving fails because the thread is inactive or unloaded, record the attempted closure in the parent status and avoid sending more work to that thread.

If an old worker cannot be resumed, start a replacement thread and update the ledger with both the old stopped thread and the new active thread.

## Automation

Use scheduled automation actively for ongoing orchestration:

- Create a heartbeat or recurring check when workers are expected to run for more than a short interactive turn.
- Reuse or update an existing automation instead of creating duplicates.
- Keep automation prompts self-contained: inspect PRs, read worker summaries, update the ledger when allowed, archive completed workers, and report only blockers or material changes.
- Delete or pause the automation when all delegated tasks are merged, stopped, or no longer worth checking.
- Avoid ad hoc polling in the main thread. If nothing needs immediate action, end the turn and rely on the automation to wake the orchestrator later.

## User Reporting

Keep routine updates concise. Report immediately only for:

- A merged PR recorded in the ledger.
- A concrete blocker needing user input.
- A task intentionally stopped or split.
- A new worker thread started.

For heartbeat-style checks, stay quiet when all workers are simply progressing.
