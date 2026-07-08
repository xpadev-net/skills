# Lessons

## 2026-07-08 - Remote-first orchestration state

- Tags: task-pr-orchestrator, git, ledger, workflow
- Symptom: Skill wording initially emphasized inspecting and salvaging stale local branch history after it accumulated, which could encourage broad backup-branch incorporation.
- Root cause: The orchestration guardrail focused on recovery behavior instead of preventing local-only ledger/plan history from drifting in the first place.
- Fix: Reworded `task-pr-orchestrator` to prefer frequent small pull/rebase/push cycles for ledger and plan updates, with backup branches used only as secondary evidence sources.
- Prevention: Orchestrator skill now treats the remote branch as the source of truth for orchestration state and instructs agents to restart from the remote base when a local branch has drifted too far.
