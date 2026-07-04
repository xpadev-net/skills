---
name: deep-review
description: Review pull requests and code changes using reusable, perspective-based review heuristics. Use when Codex is asked to review a PR, inspect review readiness, find likely review comments, prepare a code review, or check changes touching APIs, user interfaces, automation, event-driven systems, external integrations, external commands, generated artifacts, data stores, background jobs, realtime connections, auth, validation, or tests.
---

# Deep Review

Use this skill to perform a review-oriented pass over code changes. Prioritize actionable bugs, regressions, missing tests, and broken contracts over style commentary. For review tasks, use subagents when the current runtime and user instructions allow delegation.

## Workflow

1. Identify the changed files, PR intent, base/head range, original task scope, and any local review instructions. If the repository has `.github/REVIEW.md`, `AGENTS.md`, or a user-provided review rubric, read it before choosing scope or output format.
2. Honor local review contracts first for language, severity buckets, ignored paths, line-number rules, and whether external publishing commands such as `gh api` are forbidden. Do not switch to machine-readable output unless the user explicitly asks for it.
3. Classify the change area and guideline needs from changed paths, extensions, imports, frameworks, and PR intent. Use these guideline files:
   - Always read `references/review-checklist-common.md` for non-trivial reviews.
   - Read only relevant files under `references/domains/` for APIs/backends, UI, Rails/Ruby, Next.js/React/TypeScript, automation, platform PR review workflows, external commands/artifacts, event-driven workflows, integrations, data stores/jobs, media/time-series, Go services, or tests.
   - `references/review-checklist.md` is a compatibility index; prefer the split files above.
4. Before writing findings, investigate plausible impact outside the changed files. Check callers, consumers, data/schema users, tests, duplicated logic, local plans, ADRs, standards docs, and contract boundaries affected by the diff. Keep this bounded to likely risk, but do not review changed lines in isolation when the change may affect existing behavior.
5. Use subagents by review perspective before finalizing the review when delegation is available and allowed. Assign focused, non-overlapping review passes so gaps are less likely:
   - boundary validation and type/schema contracts
   - auth, ownership, namespace, and security
   - failure semantics, API/realtime contracts, and resource lifecycle
   - user-facing UX, accessibility, and UI lifecycle when interface files changed
   - tests, regression coverage, concurrency, retry, and data/job state transitions
   - automation, external commands, generated artifacts, and event-driven workflows when relevant
6. Review locally as the integrator from highest-risk contracts first:
   - boundary data validation
   - auth/ownership preservation
   - failure semantics
   - concurrency/retry/resource lifecycle
   - user-visible UI behavior
   - regression and boundary tests
7. Run a two-pass finding filter:
   - First pass: list plausible issues from the checklist, impact research, and perspective reviews.
   - Second pass: keep only findings that are grounded in the current diff, realistically actionable inside the original PR/task scope, and more than preference or speculative cleanup.
8. Merge subagent findings with your own review, de-duplicate overlaps, verify each reported issue against the actual diff, and discard speculative comments that cannot be grounded in changed code.
9. Report findings first, ordered by severity, with file and line references. Keep summary secondary.

## Scope Control

- Treat the review scope as the intersection of the user request, PR intent, changed behavior, and repository-local review contract.
- Keep a finding actionable only when the current diff caused, exposed, or made materially worse the broken contract and the fix fits the original PR/task scope.
- Classify valid but pre-existing, unrelated, or scope-expanding concerns as out-of-scope follow-up, even when they were noticed during review or requested in review feedback.
- Do not require an out-of-scope concern to be fixed in the current PR. If the user asked to address review comments, explain that the comment exceeds the original scope and propose a separate issue/backlog item instead of expanding the implementation.
- Create an issue only when the user or local workflow asks you to publish issues. Otherwise provide a concise issue draft with title, impact, evidence, and suggested owner/priority when useful.

## Subagent Instructions

When spawning subagents, give each one the PR diff or changed-file scope plus one review perspective. Include any local review contract that affects scope or classification, such as ignored generated/mock/docs files or Actionable/Nitpick buckets. Ask for findings only, with exact file/line references and a short explanation of the broken contract. Do not ask subagents to edit files.

Use this division by default:

- **Type/boundary reviewer**: runtime validation, schema contracts, casts/assertions, external/data-store/serialized-data/environment boundaries, optional/nullish behavior.
- **Security reviewer**: auth, roles, scopes, ownership checks, public/private surface separation, namespace isolation, fallback identities, cross-origin/session/cookie behavior.
- **Failure/lifecycle reviewer**: status/error semantics, realtime connection start ordering, retry behavior, cleanup, permits, timers, subscriptions, abort/disconnect paths.
- **Interface reviewer**: labels, titles, empty/loading/error states, user action failures, accessibility, effects/listeners, unmount/dispose safety, duplicated UI constants.
- **Tests/concurrency reviewer**: missing regression tests, weak assertions, bulk updates, conditional updates, job/status transitions, retry races, migration safety.
- **Automation/artifact reviewer**: automation noise filtering, revision/time windows, API pagination, external command timeout/cancellation, temp file cleanup, generated artifact safety, timestamp/sample alignment.

If the PR is small, spawn only the relevant perspective agents when delegation is available and allowed. If subagents are unavailable or prohibited in the current runtime, perform the same perspective passes yourself and explicitly state in the final review that subagents could not be used.

## Local Review Contracts

- Prefer repository-local review instructions over the skill defaults when they specify language, severity names, ignored file patterns, line-number rules, or command restrictions.
- If a local rubric defines buckets such as Actionable, Outside Diff, and Nitpick, classify issues by fix necessity and diff eligibility before writing the final response. Keep low-risk style comments out of inline/actionable findings when the rubric says they belong in summary-only nitpicks.
- If review feedback asks for work outside the original change scope, treat it like Outside Diff/follow-up unless the user explicitly expands the scope.
- When the local workflow gives a diff command or fallback range, use that range and document any fallback used. Apply local skip rules for generated files, mocks, docs, protobuf/swagger output, or other non-reviewable artifacts before assigning subagents.
- Do not publish review comments or call external APIs unless the user explicitly asks for posting. If local instructions say not to call `gh api`, treat that as a hard constraint.

## Checklist Selection

- For every non-trivial review, load `references/review-checklist-common.md`.
- Load domain files from `references/domains/` only when changed files, imports, framework conventions, or PR intent make them relevant. When unsure between two plausible domains, read both.
- Before using domain checklists, say briefly which domain files are being used and why any obvious files were excluded.
- Keep domain guidance subordinate to repository-local review instructions and the actual diff.

## Core Review Heuristics

- Treat external data, stored serialized data, environment variables, cookies, query params, request bodies, and upstream responses as untrusted until runtime-validated.
- Challenge `as` casts at boundaries. Prefer schema validation or type-narrowing patterns that preserve observable behavior.
- Check auth, role, scope, ownership, and namespace guards on every route and mutation path, including fallback or anonymous paths.
- Verify failures remain meaningful to callers: status codes, user-visible state, retry behavior, realtime event semantics, and monitoring signals must match the contract.
- Look for cleanup holes around `finally`, stream aborts, timers, subscriptions, permits, counters, jobs, and early returns.
- Require tests for the bug or contract being protected, not just the happy path. Treat missing tests as actionable only when they protect a realistic regression or changed behavior; otherwise classify them as lower-severity coverage suggestions when the local rubric supports that distinction.

## Output Expectations

- Lead with concrete findings. Use severity implicitly through ordering.
- Include exact file/line references whenever possible.
- Explain the broken contract and the realistic failure mode.
- Separate out-of-scope follow-ups from actionable findings. For each follow-up, state why it is outside the original scope and whether to draft or create an issue.
- State missing tests only when they protect a specific risk.
- When asked for review readiness or merge safety, separate blockers from notable non-blocking concerns.
- If no issues are found, say so and mention residual risk or test gaps.

## Reference

Use `references/review-checklist-common.md` for shared review concerns and the individual files under `references/domains/` for domain-specific concerns. `references/review-checklist.md` and `references/review-checklist-domains.md` remain as indexes for older references.
