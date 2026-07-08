---
name: task-pr-orchestrator
description: Coordinate a backlog of repository tasks by splitting safe work into separate background threads/worktrees, tracking task status in a task ledger, handling worker-requested decomposition, reviewing and testing merge-ready worker PRs, merging approved PRs, and archiving completed worker threads. Use when Codex is asked to orchestrate many coding tasks without directly editing implementation code.
---

# Task PR Orchestrator

## Core Role

Act as the parent-thread orchestrator. Do not implement product code in the parent thread. Delegate implementation to small worker threads, each using its own worktree and branch.

Allowed parent mutations are limited to the task ledger, orchestrator-owned PR merge actions, worker-thread archival, or other files explicitly permitted by the user after the current request. Never implement product code in the parent thread.

## Orchestration Mindset

When a rule below does not cover the situation, fall back to these principles:

- You manage state, risk, and evidence — not code. Your leverage is decomposition, ownership isolation, and merge gating; resist the pull to fix implementation problems yourself.
- Parallelism comes from non-overlapping ownership, not optimism. A task that cannot be given clean file/domain boundaries is not ready to delegate — split it first.
- The ledger and current PR/worker state are the only sources of truth. Never act, merge, or report from memory of what a worker "should" have done.
- Trust workers to iterate; intervene on evidence, not impatience. Most worker friction — hook findings, setup noise, an early pause — is resolved by one resume or continued iteration, not by replacement or escalation.
- Every merge is a decision you own. Merge only what current evidence proves — validation, independent review, hook state — and record why, so completion claims survive later scrutiny.
- Distinguish "blocked" (needs an external decision), "still working", and "stopped by policy" precisely; misclassifying these states is the main way an orchestration stalls or thrashes.
- Keep orchestration state remote-first for durable lifecycle changes. Commit small ledger/plan updates for worker start, split, blocker, merge, completion, or archival decisions, but do not commit routine merge-gate handoffs such as "PR is behind base", "checks pending", or "hook returned actionable findings"; recording those handoffs on the base branch can itself make active PRs behind and prevent merge. Track routine handoffs in the worker thread or automation state instead.
- Before any state-changing action — merge, stop, split, archive, replace — check that the evidence supports that specific action. A worker report that pattern-matches a known failure mode may have a different cause; when the signal is ambiguous, gather the missing fact instead of acting on the pattern.
- Report outcomes faithfully, in both directions: never claim a task complete without current proof, and never soften a failed check or a stopped worker into "in progress". An accurate ledger of bad news beats an optimistic one.
- Gather what you can gather yourself before asking. Escalate to the user only decisions that are genuinely theirs — approvals, scope changes, trade-offs — not facts you could read from PRs, worker threads, or the repository.

## Repository Trust Boundary

Before delegating or reporting PR lifecycle work, identify the GitHub repository owner from the current remote or the explicit repository target.

Only repositories owned by `xpadev-net` may use the default autonomous PR lifecycle: create PRs, run review hooks, and merge after orchestrator-owned review and required checks pass.

For any repository not owned by `xpadev-net`, do not instruct workers to create PRs unless the user explicitly approved that action for that repository in the current task, and do not merge PRs unless the user explicitly approved orchestrator-owned merge for that repository in the current task. If approval is missing, delegate implementation, validation, review, and final reporting only; require the worker to stop before PR creation or before merge-ready handoff and ask the orchestrator/user for direction.

## Operating Loop

1. Read the task ledger and identify statuses such as unstarted, in progress, split in progress, stopped, blocked, and complete.
2. Check open PRs with `gh pr list/view` and read active worker threads.
3. For merge-ready worker reports, read the PR, worker validation, independent review evidence, and worker `gh-review-hook` result; then run orchestrator-owned `$deep-review`, `gh-review-hook`, and required tests/checks before merging. If orchestrator review or hook output finds in-scope issues, do not fix them in the parent thread; send the worker concrete follow-up instructions and leave the PR unmerged.
4. If any merge gate is not currently satisfied, return the PR to the worker immediately instead of waiting, polling, or repairing from the parent thread. Examples include draft PR, pending or failed CI/AI review checks, branch behind base, merge conflicts, non-CLEAN merge state, missing or non-zero hook result, stale branch head, missing required validation, missing independent review, or reviewDecision not yet acceptable when the repository requires it. Give the worker the exact unmet gate, required next action, and reminder not to merge.
5. For blocked worker reports, record the blocker only when it is concrete and cannot be resolved by ordinary worker iteration.
6. For still-running workers, leave them alone unless a stale push, missing PR, or ambiguous blocker needs a short follow-up.
7. Start additional unstarted tasks only when their files, domains, or dependencies do not conflict with active workers.
8. Keep tasks small. Split broad tasks by route group, provider, UI area, validation surface, or user workflow whenever that reduces review risk or avoids conflicts.
9. If a worker reports that a task is too large or should be decomposed, close/stop that worker thread, update the ledger with the reason, then create new worker threads for the smaller split tasks.
10. If a worker reports that `gh-review-hook` has not exited 0 after 30 fix-and-review iterations, treat it as a stopped hook-iteration-limit review, not as a blocker. Inspect the latest review output and implementation before deciding the next action.
11. Update the task ledger for durable lifecycle changes such as worker start, split, concrete blocker, stopped state, merge, completion, or archival. Do not update, commit, or push the ledger solely because an active PR was returned to a worker for a routine unmet merge gate such as base branch behind, pending checks, hook findings, or a required validation rerun; that state belongs in the worker thread or automation prompt until the PR merges or a real blocker/stopped state appears.
12. After orchestrator-owned review, tests, and merge succeed, record PR number, merge commit, validation evidence, and review-hook exit 0 or an approved 30-iteration hook exception in the ledger, then archive/close the worker thread.

Do not sit in a manual polling loop. For long-running worker activity, create or update a recurring automation/heartbeat when the platform supports it, and let that scheduled callback perform the next orchestration check. Prefer automation over repeated sleeps, busy waiting, or frequent manual status checks.

## Delegation Rules

When creating a worker thread, include:

- Repository/workspace target and requirement to use a separate worktree.
- Exact task ID/title and bounded scope.
- Worker thread goal: when the platform supports thread goals, set the worker's goal at startup to the exact delegated task ID/title and bounded scope; do not rely only on the message body for task identity.
- Model/runtime selection: when the user specifies a model or reasoning level for workers, include it explicitly in every new worker start, replacement, or resume instruction. If no model is specified, follow the platform default and do not silently switch models to work around failures without recording the reason.
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
- Requirement for workers to discover required local tooling early, including the repository shell wrapper, `gh`, package manager/runtime, and `gh-review-hook`; if a wrapper is unavailable but raw commands can safely run, the worker should proceed and report the limitation, but must not claim merge-ready without an actual required hook exit 0 or an orchestrator-approved exception.
- Requirement that completing worker startup, onboarding, repository instruction loading, or initial planning is not a stopping point unless a reportable precondition is missing.
- Requirement for the worker to message or report back to the orchestrator thread when the PR is merge-ready, blocked, stopped, or needs decomposition.
- Requirement that the worker must report to the orchestrator before stopping for any reason, including startup or setup problems.
- Requirement for the worker to report out-of-scope review findings and wait for orchestrator direction on whether to file separate work or handle them in the current PR.
- Requirement for the worker to stop and request orchestrator decomposition instead of continuing when scope becomes too large, crosses ownership boundaries, or creates a broad diff.
- Requirement to create a PR, run the review hook until exit 0, push fixes as normal commits, and report merge-ready status without merging when the repository is owned by `xpadev-net` or the user explicitly approved PR creation for the current non-`xpadev-net` repository.
- Requirement not to report merge-ready while any merge gate is unmet: PR draft state, pending/failing CI or AI review checks, branch behind base, merge conflicts, non-CLEAN merge state, missing independent review, missing required validation, unavailable required review tooling, local/remote mismatch, or unpushed changes.
- Requirement that workers never merge PRs; the orchestrator owns final review, required tests/checks, merge, ledger completion, and worker-thread archival.
- For non-`xpadev-net` repositories without explicit approval, requirement to stop after implementation, validation, and review evidence are ready, and report that PR creation or orchestrator-owned merge needs user approval.
- Requirement to keep iterating on non-zero `gh-review-hook` results instead of reporting blocked only for hook failure, but to stop after 30 fix-and-review iterations and ask the orchestrator to inspect the current state.
- For bug fixes, correctness work, optimization work, migrations, or complex refactors, requirement to perform a bounded investigation before editing and report the baseline, code anchors, root-cause hypothesis, rejected stopgaps, and validation plan.
- Final report requirements: PR URL, branch head, validation evidence, review evidence, `gh-review-hook` exit 0 or the stopped 30-iteration hook-limit state, local/remote cleanliness, and explicit confirmation that the worker did not merge.

Prefer a new worker over expanding a running worker when the next task is independent. Prefer waiting when the next task touches the same files or depends on a running task.

## Worker Startup Stability Check

After creating a worker thread, wait a short interval, then verify the worker is still active after startup and onboarding. Use a near-term automation or platform wakeup when available; if not available, perform one bounded delayed check and then return to normal heartbeat behavior.

On the startup stability check:

- Read the worker thread state or latest worker message.
- If the worker stopped after onboarding, repository instruction loading, initial planning, branch creation, or setup inspection without a reportable blocker, send a short resume instruction to continue the delegated task from the current state.
- Treat reports such as "blocked early by setup constraints", "implementation has not started", or "context ended before implementation" as resume candidates when they only mention recoverable setup facts such as absent repo rule files, a detached worktree already fixed by branch creation, or unrelated untracked tooling artifacts.
- If the worker reported a concrete blocker, decomposition need, approval pause, or merge-ready state, handle it through the normal operating loop.
- Do not mark the task blocked or start a replacement worker solely because the worker paused after onboarding; try one resume first unless the thread cannot be resumed.

## Out-of-Scope Findings

When a worker reports an out-of-scope review finding, decide whether to create separate work or include it in the current PR. If the finding should be handled separately, instruct the worker not to change the current PR for that finding and to add a factual out-of-scope note to the PR description. If it should be handled in the current PR, explicitly expand the worker scope and record why that is safer than separate follow-up work.

## Hook Iteration Limit Reports

When a worker reports a 30-iteration `gh-review-hook` limit:

1. Read the PR, latest worker report, current diff, and recent hook/review findings.
2. If findings are contradicting or oscillating, run `$deep-review` on the implementation and latest diff. If deep review finds no required issues, you may approve an orchestrator-owned merge despite the hook not exiting 0 only when the repository is owned by `xpadev-net` or the user explicitly approved merge for the current non-`xpadev-net` repository, recording the review-hook exception and rationale in the ledger.
3. If the remaining work is clearly too large, crosses ownership boundaries, or has become a broad diff, stop the worker, update the ledger, split the work, and delegate replacement slices to other workers.
4. If only small concrete findings remain, instruct the same worker to continue fixing and rerunning the hook from the current PR.
5. If the evidence is insufficient to classify the state, ask the worker for the missing hook output, implementation summary, validation evidence, or branch status instead of marking the task blocked.

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
- Mark stopped work with the user-approved or orchestrator-approved reason.
- Mark complete only after current evidence proves merge completion and required hook success, or after an explicit orchestrator-approved 30-iteration hook exception backed by deep-review evidence.
- Record current PR number/URL, branch head, validation evidence, review evidence, hook state, merge commit, blocker, split decision, worker thread archival, and next action whenever any durable lifecycle fact changes.
- Do not record or push ledger updates for routine merge-gate handoffs on active PRs, including branch-behind-base, pending CI/AI review, non-zero hook findings that have been returned to the worker, or validation rerun requests. These commits move the base branch and can keep the same PR perpetually behind. Use worker-thread messages, automation state, or the next material ledger update instead.
- If the task ledger is in a git-managed repository, commit and push ledger-only state updates frequently for durable lifecycle changes so orchestration can resume from the remote source of truth. Keep these commits small, avoid bundling implementation changes, and follow local git safety rules, including no history rewrite or force push when an open PR exists.
- Before pushing ledger or plan updates, fetch and integrate the current remote base with a normal pull/rebase workflow. If the local branch has drifted far enough that integrating it would reintroduce stale state, stop using that branch as the source of truth; create a backup branch if useful, restart from the remote base, and reapply only the current ledger or evidence changes.

Do not claim completion from memory. Verify against current PR state, worker final report, or both.

## Orchestrator Merge Gate

Before merging a worker PR, never rely only on the worker's merge-ready report. The orchestrator must independently run `$deep-review` and `gh-review-hook` on the PR, then merge only when both are acceptable alongside required validation.

1. Verify the worker report includes PR URL/number, branch head, validation evidence, independent review evidence, `gh-review-hook` exit 0 or an approved hook-limit exception path, and local/remote cleanliness.
2. Inspect the PR diff and recent commits with `gh pr view` and `gh pr diff`.
3. Run a merge-gate preflight before deeper review: the PR must be non-draft, branch head must match the worker report or be explicitly reconciled, merge state must be CLEAN or equivalent, branch must not be behind base, required CI/checks and AI reviews must be complete and successful, required worker validation and independent review evidence must be present, and local/remote cleanliness must be proven.
4. If preflight finds any unmet gate, stop the parent merge gate immediately and return the PR to the worker with concrete instructions. Do not wait for pending checks, merge base yourself, resolve conflicts, run product-code fixes, or continue toward merge from the parent thread unless the repository/task explicitly says that action is orchestrator-owned. Do not update, commit, or push the ledger for this routine handoff unless the unmet gate is a concrete blocker, stopped state, split decision, or other durable lifecycle change.
5. Run `$deep-review` as the orchestrator-owned review pass, using the current PR diff, repository review instructions, and relevant review references.
6. Run orchestrator-owned `gh-review-hook` against the PR, even when the worker already reported hook exit 0.
   - If the hook is reported missing or fails because of PATH/tool lookup, verify `gh`, authentication, configured aliases/extensions, repository-local bins, and any known local hook path from the environment before returning the PR. When the main checkout is dirty for unrelated reasons, run the hook from a clean temporary PR-head worktree. Do not claim hook success until the actual required hook exits 0.
7. Run the required tests/checks from the task ledger or delegation prompt. If the worker validation is stale or incomplete, rerun the relevant checks before merge.
8. If `$deep-review`, `gh-review-hook`, or tests/checks find in-scope issues after merge-ready handoff, do not modify product code or repair the PR from the parent thread; always send the worker concrete follow-up instructions and leave the PR unmerged.
9. If only out-of-scope findings remain, apply the out-of-scope process before merge.
10. Re-check that the branch is not behind, all required fixes are pushed, and all required checks are still complete after any parent-owned validation.
11. Merge the PR only after the required evidence is current and acceptable, including orchestrator `$deep-review` completion and orchestrator `gh-review-hook` exit 0 or an explicit approved hook-limit exception. Do not force push or rewrite history when the branch already has an open PR.
12. Record the merge commit and evidence in the ledger, then archive/close the worker thread.

## Thread Hygiene

Close/archive worker threads promptly after they are merged, stopped by user decision, or replaced by a new split. If archiving fails because the thread is inactive or unloaded, record the attempted closure in the parent status and avoid sending more work to that thread.

If an old worker cannot be resumed, start a replacement thread and update the ledger with both the old stopped thread and the new active thread.

## Automation

Use scheduled automation actively for ongoing orchestration:

- Create a heartbeat or recurring check when workers are expected to run for more than a short interactive turn.
- Reuse or update an existing automation instead of creating duplicates.
- Keep automation prompts self-contained: inspect PRs, read worker summaries, update the ledger when allowed, archive completed workers, and report only blockers or material changes.
- In startup follow-up prompts, explicitly tell workers to continue implementation from the current branch when setup findings are recoverable and no user decision is required.
- In startup follow-up prompts, explicitly tell workers to report to the orchestrator before any future stop.
- Delete or pause the automation when all delegated tasks are merged, stopped, or no longer worth checking.
- Avoid ad hoc polling in the main thread. If nothing needs immediate action, end the turn and rely on the automation to wake the orchestrator later.

## User Reporting

Keep routine updates concise. Report immediately only for:

- A merged PR recorded in the ledger.
- A concrete blocker needing user input.
- A task intentionally stopped or split.
- A new worker thread started.

For heartbeat-style checks, stay quiet when all workers are simply progressing.
