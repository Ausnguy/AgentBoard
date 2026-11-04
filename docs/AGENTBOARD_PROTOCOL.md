# AgentBoard protocol

## Messages
New messages: first line `AGENTBOARD_V1`, then one JSON object (optionally in a ```json fence). Unknown keys are rejected.
Required: `v`=1, `kind` (task|report|blocked|request), `task_id` (8-64 chars `[A-Za-z0-9_-]`), `from`, `to` (a configured role or `human`).
Tasks also need `objective`. Limits: body 6000 chars, 8 acceptance items, 4 dependencies, handoff depth <= 4, 3 handoffs per root.
Optional: `parent_id`, `root_id`, `title`, `acceptance`, `allowed_paths`, `depends_on`, `priority` (high|normal|low),
`requires_approval`, `tier` (simple|standard|complex), `status` and `summary` (reports).

Legacy (still accepted): header lines `AGENT_TO:`, `AGENT_FROM:`, optional `KIND:`, blank line, then the objective. `task_id` = `legacy-<comment id>`.

## Authorization
The GitHub comment author (`user.login`) is looked up in `users`; the claimed `from` must be in that login's role list.
`allowed_paths` can only narrow the role's configured paths. A `report`/`blocked`/`request` comment never starts Claude.
This is necessary but not sufficient for zero trust: anyone who can write to the Issue can post text (it is ignored unless authorized),
and anyone holding an allowed login can speak as its roles. Edited comments are ignored after first sight (dedupe by comment id).

## Task states
`queued -> running -> succeeded | blocked | failed_retryable | failed_permanent`
(`seen/validated` are recorded per comment in the `comments` table.)
- `failed_retryable`: transient network error, auto-retried at most 2 times with capped backoff.
- `blocked`: out-of-scope edits, failed tests, budget/turns exhausted, auth/permission, dirty worktree, interrupted run, needs approval.
- A task found `running` at listener start becomes `blocked` (`review_required`); it is never silently repeated.

## Reports and handoffs
Report = `AGENTBOARD_V1 {...}` line plus TASK/STATUS/CHANGED/CHECKS/NEXT/EST_COST_USD lines, <= 600 chars (<= 1000 on failure).
A worker may emit one `NEXT_TASK: {json}` line. Python validates target role, depth, per-root cap and
`(parent, target, objective hash)` dedupe, then posts the task itself. Reports are posted before handoffs so dependencies resolve.

## Local state
`.agentboard-state/<role>.db` (SQLite), `<role>.lock` (one listener per role), `slot-N.lock` (local concurrency), `evidence/`.
Posting is via an outbox: an ambiguous POST failure is reconciled against the Issue by `task_id` before any re-post.
