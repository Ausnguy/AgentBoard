---
name: agentboard-ops
description: Operate the AgentBoard listener: diagnose blocked tasks, retry, change budget, recover after a crash. Use when a task is blocked/failed or a listener misbehaves.
---

1. `python agentboard.py status --as ROLE` shows open tasks with reasons.
2. `blocked` reasons: `out_of_scope edits` (see `.agentboard-state/evidence/<task>.patch`), `worktree has uncommitted changes`
   (inspect, commit or discard yourself, never auto-reset), `review_required` (listener died mid-task: check the worktree),
   `needs_approval`, `budget_exhausted`, `max_turns`, `permission denied` (add the command to `claude.extra_allowed_tools` or
   the role's `test_command`; do not use --dangerously-skip-permissions), `dependency_failed`.
3. After fixing the cause: `python agentboard.py retry --as ROLE TASK_ID`. Succeeded tasks are never replayed.
4. Budget: `budget.daily_usd` in `.agentboard.json` (estimates, not a billing guarantee; 0 refuses all dispatch).
5. Reports unposted (GitHub down)? They sit in the local outbox and post on the next cycle after reconciling.
