# Review Checklist: Platform PR / MR Review Workflows

Use this when reviewing tools or automations that fetch diffs, generate review comments, moderate findings, post inline comments, approve, request changes, or publish summary reviews on GitHub, GitLab, Bitbucket, or similar platforms.

## Diff And Line Anchoring

- The diff source, base/head refs, and head commit SHA are explicit and match the PR/MR being reviewed.
- Annotated line numbers refer to the new file side of the current diff, not stale local files or outdated review threads.
- Inline comments are only created for lines the platform accepts, usually added/modified lines on the right side of the diff.
- Multi-line comments validate that every line in the range is commentable in the current diff.
- Large, truncated, reconstructed, or API-derived diffs have a fallback path or clear warning when line placement may be unreliable.

## Moderation And Publishing Safety

- Generated comments are reviewed or filtered before posting, especially when the workflow can approve or request changes.
- Posting actions distinguish approve, request changes, comment-only, summary-only, and cancel paths.
- Requesting changes is treated as potentially merge-blocking on platforms where that verdict affects branch protection.
- If all comments are discarded or no actionable issue exists, the workflow does not post noisy placeholder comments unless the user asked for them.
- Publishing failures report which comments were skipped and why, without losing the unposted review content.

## Credentials And Platform Semantics

- Credential checks fail early with actionable errors and do not leak tokens or account details.
- Platform-specific permission scopes are sufficient for posting but not broader than needed.
- GitHub, GitLab, and Bitbucket differences are reflected in behavior; for example, a "request changes" concept may not have the same merge-blocking semantics everywhere.
- Self-hosted or enterprise hosts are detected from repository remotes or explicit configuration without assuming public SaaS defaults.

## Temporary Artifacts And Cleanup

- Temporary diff, metadata, comments, and summary files use unique names and are cleaned up on success and cancellation where possible.
- Interrupted workflows leave recoverable artifacts or clear cleanup guidance instead of silently discarding review state.
- Comment and summary file parsers validate required fields before posting and skip malformed blocks safely.

## Summary / Merge-Readiness Reviews

- Summary mode focuses on merge safety: critical bugs, data loss, security, breaking contracts, critical error handling, and severe performance issues.
- Summary mode excludes style, naming, minor documentation gaps, and broad test coverage suggestions unless they block merge readiness under local policy.
- Verdict labels are clearly defined and map to platform actions only after user confirmation.
