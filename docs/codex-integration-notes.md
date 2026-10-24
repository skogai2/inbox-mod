# Codex integration: findings and remaining work

Working notes from the 2026-10-24 integration session. The target workflow is
one focused task with asynchronous questions, improvement proposals and sanity
checks parked in Cactus. Answers should preserve the reason for a decision and
may later come from another repository's agent. Immediate waits are exceptional.

## Observed in this session

- Worktrunk's skill expects `EnterWorktree`; this host has no such tool.
  `wt switch --create ... --no-cd --format=json` created the checkout through
  an approved external run. Commands must explicitly name its working directory.
- Git metadata and the live Cactus database are outside this session's writable
  access. A clean worktree does not imply that Git operations are writable.
- `cactus feed` opens Store through migrations and fails on the live readonly
  database. This is an access problem as well as a read-path design concern.
- The permission hook converted each escalation to a new run row, then denied
  the host request. Approving a run executed externally without making subsequent
  agent commands writable. Retrying inbox reads created duplicate approvals.
- Answer notifications and frontier injection reached this conversation. That
  proves this configured delivery path, not a generic Codex idle-wake facility.

## Changes and evidence

Permission interception is now opt-in with `CACTUS_PERMISSION_INBOX=1`.
Default hook execution emits no decision and creates no row, leaving the host
permission mechanism in control. Explicit Cactus runs remain available.
Subprocess tests exercise both paths against scratch databases.

The tested follow-up patch makes frontier and session-start change to the payload's cwd before reading rows.
Previously they checked that project's enabled state but queried the launcher
cwd. Tests use two projects with identical qN keys and the same agent, ensuring
only the payload project's answered row is injected and cleared. Missing payload
directories stop the hook rather than silently falling back.

The scope patch was applied after permission-fix commit `7aec005`.
The user subsequently changed the session access settings: a live feed read
and Git commit succeeded. This verifies access under the new settings, not
the updated plugin behavior under the original restricted sandbox. The stale
commit approval q27 was retired after the commit succeeded.

These changes are source-level work. Updating the installed plugin and observing
native permission approval are separate live checks still to perform.

## Next work, in order

1. Reload the installed plugin from reviewed source, preserving its identity.
   Verify one native approval, then one post / answer / read cycle. Do not claim
   success from shell-hook tests alone or edit the installed cache directly.
2. Establish supported access to the live inbox. Consider a read-only Store path
   for feed/list separately from writes: schema compatibility, WAL visibility,
   and `get --agent` acknowledgment writes need explicit treatment. Do not simply
   swallow migration failures or use immutable SQLite reads on a changing inbox.
3. Deliver run results in the frontier. It currently injects the verdict but not
   the captured result; when a row has already executed, an agent must be able to
   distinguish approval from completion before retrying the command.
4. Revisit automatic clearing and persistent verdict delivery. Printing a verdict
   is delivery, not evidence the agent acted. Passed reviews become cleared and
   are excluded by the current frontier status filter. Define acknowledgment
   before changing retention, so history stays intact without endless replays.
5. Make optional permission-inbox mode deduplicate requests if it remains useful.
   Bind any reuse to owner, project, exact command and execution result. Never
   interpret an old approval as a general sandbox grant.
6. Map cross-repository questions only after one local cycle works. Keep evidence
   gathering separate from decision authority; preserve origin, recipient and
   correlation with the original question.

## Questions to park once inbox access works

- Should injected answers remain actionable until the agent explicitly records
  acknowledgment, or should the current print-and-clear behavior remain?
- Is external command execution a desired Cactus capability for Codex, or should
  Cactus hold intent while native permissions always govern tool execution?
- Which repository and agent should be our first bounded cross-repository test?

These are deferred design questions, not approvals to change access or send
messages to other agents. No live inbox state is needed to run the tests.
