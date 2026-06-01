---
name: deep-review
description: Review pull requests and code changes using reusable, perspective-based review heuristics. Use when Codex is asked to review a PR, inspect review readiness, find likely review comments, prepare a code review, or check changes touching APIs, user interfaces, automation, event-driven systems, external integrations, external commands, generated artifacts, data stores, background jobs, realtime connections, auth, validation, or tests.
---

# Deep Review

Use this skill to perform a review-oriented pass over code changes. Prioritize actionable bugs, regressions, missing tests, and broken contracts over style commentary. For review tasks, use subagents even when the user did not explicitly request them.

## Workflow

1. Identify the changed files, PR intent, base/head range, and any local review instructions. If the repository has `.github/REVIEW.md`, `AGENTS.md`, or a user-provided review rubric, read it before choosing scope or output format.
2. Honor local review contracts first for language, severity buckets, ignored paths, line-number rules, and whether external publishing commands such as `gh api` are forbidden. Do not switch to machine-readable output unless the user explicitly asks for it.
3. Classify the change area: API/backend, user interface, automation, external commands/artifacts, event-driven or realtime workflows, external integration, data store/background jobs, auth/security, tests, or cross-cutting refactor.
4. Read only the relevant sections of `references/review-checklist.md`.
5. Always use subagents by review perspective before finalizing the review. Do this even when the user only asks for a generic "review" and does not mention subagents. Assign focused, non-overlapping review passes so gaps are less likely:
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
7. Merge subagent findings with your own review, de-duplicate overlaps, verify each reported issue against the actual diff, and discard speculative comments that cannot be grounded in changed code.
8. Report findings first, ordered by severity, with file and line references. Keep summary secondary.

## Subagent Instructions

When spawning subagents, give each one the PR diff or changed-file scope plus one review perspective. Include any local review contract that affects scope or classification, such as ignored generated/mock/docs files or Actionable/Nitpick buckets. Ask for findings only, with exact file/line references and a short explanation of the broken contract. Do not ask subagents to edit files.

Use this division by default:

- **Type/boundary reviewer**: runtime validation, schema contracts, casts/assertions, external/data-store/serialized-data/environment boundaries, optional/nullish behavior.
- **Security reviewer**: auth, roles, scopes, ownership checks, public/private surface separation, namespace isolation, fallback identities, cross-origin/session/cookie behavior.
- **Failure/lifecycle reviewer**: status/error semantics, realtime connection start ordering, retry behavior, cleanup, permits, timers, subscriptions, abort/disconnect paths.
- **Interface reviewer**: labels, titles, empty/loading/error states, user action failures, accessibility, effects/listeners, unmount/dispose safety, duplicated UI constants.
- **Tests/concurrency reviewer**: missing regression tests, weak assertions, bulk updates, conditional updates, job/status transitions, retry races, migration safety.
- **Automation/artifact reviewer**: automation noise filtering, revision/time windows, API pagination, external command timeout/cancellation, temp file cleanup, generated artifact safety, timestamp/sample alignment.

If the PR is small, spawn only the relevant perspective agents, but still use at least one subagent. If subagents are unavailable in the current runtime, perform the same perspective passes yourself and explicitly state in the final review that subagents could not be used.

## Local Review Contracts

- Prefer repository-local review instructions over the skill defaults when they specify language, severity names, ignored file patterns, line-number rules, or command restrictions.
- If a local rubric defines buckets such as Actionable, Outside Diff, and Nitpick, classify issues by fix necessity and diff eligibility before writing the final response. Keep low-risk style comments out of inline/actionable findings when the rubric says they belong in summary-only nitpicks.
- When the local workflow gives a diff command or fallback range, use that range and document any fallback used. Apply local skip rules for generated files, mocks, docs, protobuf/swagger output, or other non-reviewable artifacts before assigning subagents.
- Do not publish review comments or call external APIs unless the user explicitly asks for posting. If local instructions say not to call `gh api`, treat that as a hard constraint.

## Core Review Heuristics

- Treat external data, stored serialized data, environment variables, cookies, query params, request bodies, and upstream responses as untrusted until runtime-validated.
- Challenge `as` casts at boundaries. Prefer schema validation or type-narrowing patterns that preserve observable behavior.
- Check auth, role, scope, ownership, and namespace guards on every route and mutation path, including fallback or anonymous paths.
- Verify failures remain meaningful to callers: status codes, user-visible state, retry behavior, realtime event semantics, and monitoring signals must match the contract.
- Look for cleanup holes around `finally`, stream aborts, timers, subscriptions, permits, counters, jobs, and early returns.
- Require tests for the bug or contract being protected, not just the happy path. Treat missing tests as actionable only when they protect a realistic regression or changed behavior; otherwise classify them as lower-severity coverage suggestions when the local rubric supports that distinction.

## Area-Specific Focus

- **API/backend**: request/response schemas, auth guards, owner checks, status codes, pagination/count cost, and conditional data-store updates under concurrency.
- **User interface**: failed user actions, empty/loading/error states, page titles, labels, accessibility, shared UI utilities, effect/listener cleanup, and unmount/dispose safety.
- **External integrations**: upstream response validation, missing optional fields, safe fallback behavior, namespace/scope checks, and upstream error mapping.
- **Automation**: correct revision and time-window selection, machine-generated noise filtering, delayed/async result waits, transient request retry bounds, idempotency of retried mutations, and internal artifact leakage.
- **External commands/artifacts/media**: context-aware process spawning, timeout and process-group cleanup, temp file uniqueness and cleanup, download header sanitization, partial-read/delete races, and timestamp/sample alignment.
- **Event-driven or realtime workflows**: workspace/channel scoping, invoker authorization, permission-cache fail-closed behavior, state-cache misses, grace timers, reconnect/requeue behavior, and user-visible failure reporting.
- **Data stores/background jobs/batch**: bulk update scope, conditional expressions, job/status semantics, retry races, migration safety, and producer/consumer state-machine agreement.
- **Go/backend services**: context propagation, nil-pointer risks, transaction boundaries, deferred cleanup, error wrapping/classification, gomock expectation drift, pagination/filter consistency, and domain-layer boundary violations.
- **Realtime connections/streaming**: perform auth and limit rejection before stream start, preserve protocol-level failures, release permits on disconnect/abort/error, and avoid shared fallback buckets.

## Output Expectations

- Lead with concrete findings. Use severity implicitly through ordering.
- Include exact file/line references whenever possible.
- Explain the broken contract and the realistic failure mode.
- State missing tests only when they protect a specific risk.
- If no issues are found, say so and mention residual risk or test gaps.

## Reference

Use `references/review-checklist.md` for the full checklist and examples of recurring review concerns. Load it when the change touches a non-trivial area or when you need a structured checklist for a review.
