# Investigation Before Editing

Use this checklist before implementation when the delegated task is a bug fix, correctness repair, optimization, migration, or non-trivial refactor. Keep it bounded, but leave enough evidence that the implementation follows from the diagnosis rather than from guesswork.

## Required Shape

1. Read the current task, relevant plans, reports, ADRs, known-bug notes, and prior failure logs before reading broad source areas.
2. Establish the baseline: current failing command, known pre-existing failures, performance/size numbers, corpus pass rate, or other acceptance metric. If a full baseline is too expensive, record the focused substitute and why it is enough.
3. Locate code anchors for the suspected behavior: writer and reader paths, guards, flags, call sites, tests, generated artifacts, and any runtime boundary involved.
4. Turn repeated symptoms into a shared invariant or structural cause when possible. Prefer "write representation and read representation disagree" over a list of unrelated failing cases.
5. Actively refute the leading diagnosis. Look for existing guards, alternate call paths, environment/runtime causes, and tests that would disprove it.
6. Separate stopgaps from root fixes. If a narrow patch would add another special case or persistent mixed mode, record why it is temporary or choose the structural fix.
7. Define the rollout and validation path before editing: feature flag, shadow path, unchanged default behavior, targeted tests, corpus checks, type gates, parity checks, or heap/performance measurements as appropriate.

## Evidence To Carry Into Implementation

- Symptom and impact in one or two sentences.
- Current baseline and known unrelated failures.
- Code anchors with file paths and line references when available.
- Root-cause hypothesis and the invariant it protects.
- Counter-evidence checked, including why VM/SDK/framework/runtime blame is or is not plausible.
- Chosen fix direction, rejected alternatives, and the reason stopgaps are insufficient if relevant.
- Validation plan with the cheapest focused gate first and broader gates after.

## When To Stop And Ask

Stop and report to the orchestrator when the investigation shows that the delegated task crosses ownership boundaries, needs a product or rollout decision, requires a risky default flip, or depends on missing credentials/environment data. Include the evidence above and a concrete decomposition or decision request.
