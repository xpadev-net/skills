# Investigation Quality

Use this when reviewing a bug fix, correctness repair, optimization, migration, or complex refactor. The goal is not to demand process for its own sake; it is to catch changes that are plausible but not actually grounded in the system's current behavior.

## Review Questions

- Did the author read the relevant existing plans, reports, known-bug notes, logs, or design docs before choosing the fix?
- Is there a current baseline for the symptom: failing test, corpus result, performance or size number, parity result, heap measurement, or known pre-existing failure count?
- Are the important code anchors identified, including both write and read paths, guards, flags, call sites, generated/runtime boundaries, and tests?
- Does the diagnosis explain multiple symptoms through a shared invariant or structural cause, instead of patching each observed failure independently?
- Did the author look for counter-evidence: existing guards, alternate execution paths, caller behavior, framework defaults, runtime/environment causes, or tests that would disprove the theory?
- Are stopgap fixes clearly separated from root fixes? Be suspicious of new special cases that preserve or expand mixed representations, duplicate sources of truth, or third read/write paths.
- Is the rollout path proportional to risk: feature flag, shadow mode, unchanged default path, type gate, parity harness, behavior oracle, corpus run, or performance/heap budget?
- Does the validation prove the contract being changed, not merely that a narrow happy path still passes?

## Findings To Raise

Raise an actionable finding when a missing investigation step creates realistic risk in the current PR, such as:

- The fix addresses one symptom but leaves the same broken invariant reachable through another code path.
- A baseline is absent, so the PR can neither prove improvement nor detect regression for the stated goal.
- The change blames an external runtime, SDK, or framework without a minimal reproduction or without ruling out generated/application code.
- A narrow patch adds another representation or special-case path when the safer direction is to unify the representation or source of truth.
- The rollout flips risky behavior by default before shadow, parity, or corpus evidence supports it.

Keep purely process-oriented complaints out of findings. If the investigation is imperfect but the diff is still demonstrably safe, mention residual risk in the summary instead.
