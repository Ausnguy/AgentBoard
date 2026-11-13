# AgentBoard

This copy is set up for Project TALON. Start with `docs/PROJECT_BRIEF.md` (what the project is) and
`docs/TEAM.md` (who owns which part, and how to ask them for something). The board is
[Issue #1](https://github.com/Ausnguy/AgentBoard/issues/1).

| Role | Owner | Start your listener |
|---|---|---|
| `tag` | Wali | `python agentboard.py setup --as tag --start` |
| `gate` | Austin | `python agentboard.py setup --as gate --start` |
| `ai` | Austin | `python agentboard.py setup --as ai --start` |
| `backend` | Wali | `python agentboard.py setup --as backend --start` |
| `integration` | Wali | `python agentboard.py setup --as integration --start` |
| `frontend` | Aayan | `python agentboard.py setup --as frontend --start` |
| `pitch` | Aayan | `python agentboard.py setup --as pitch --start` |

The sections below describe the tool itself. Its examples use the generic roles `frontend`, `backend`, `ai`
and `integration`; substitute the TALON roles above.

A GitHub Issue is the message board. Each teammate runs one listener for one role. Plain Python decides what to run;
`claude -p` only implements one bounded task inside that role's folder, on that role's git worktree/branch.

- Python 3.10+, standard library only. Needs `git`, `gh` (authenticated) and `claude` (authenticated).
- Agents must stay online: no listener running means no work happens (laptop asleep = paused; it resumes on wake).
- The Issue carries messages, not code. Code moves through local `agent/<role>` commits you review and PR yourself.
- Nothing is pushed, merged, or deployed automatically.

## Quickstart

### Maintainer (once)

```powershell
gh auth login
git init -b main; git add -A; git commit -m init        # skip if the project is already a git repo with a GitHub remote
python agentboard.py init --create-issue                  # detects repo + your GitHub login, creates the board Issue
python agentboard.py team add alice --roles frontend      # one line per teammate (their GitHub login -> roles)
python agentboard.py team add bob --roles backend,ai
git add -A; git commit -m "agentboard config"; git push   # teammates clone this
```

`init` auto-detects the repo (`gh repo view`) and your login. `--create-issue` is the only step that writes to GitHub, and only when you pass it.
You can instead pass `--repo OWNER/REPO --issue N` for an existing Issue. Optional: set a role's `test_command` in `.agentboard.json`
(for example `"python -m unittest discover -s backend"`).

### Teammate (2 commands)

```powershell
git clone <team repo> ; cd <team repo>
python agentboard.py setup --as frontend --start
```

`setup` checks git/gh/claude, confirms your GitHub login is allowed for the role (and prints the exact line to send the maintainer if not),
creates your `agent/<role>` branch and worktree (reusing an existing branch, never overwriting), and with `--start` begins listening.
Safe to re-run any time. If something is missing it says what to install or run (`gh auth login`, `claude auth login`).

`python agentboard.py doctor` shows the full preflight at any point (no model calls).

## 4-role demo

Run one listener per role (each on its owner's machine, or four terminals):

```powershell
python agentboard.py listen --as backend
python agentboard.py listen --as frontend
python agentboard.py listen --as ai
python agentboard.py listen --as integration
```

From an authorized person:

```powershell
python agentboard.py post --from human --to backend --message "Implement GET /api/health inside backend/ with a targeted test" --accept "returns 200 JSON status"
```

Flow: the backend listener validates the author, runs Claude in `backend/`, checks changed paths and tests, commits locally to
`agent/backend`, and posts a short report. If the worker proposes a follow-up it emits `NEXT_TASK`; the orchestrator validates it and posts
a task to `frontend` (depth +1, dedupe, max 3 per root, max depth 4). Frontend then hands off to `integration` the same way.
Review each `agent/<role>` branch and open PRs yourself. For deterministic demos add `--once` (one cycle, then exit) or `--dry-run`
(show what would run, no Claude, no posts).

From a phone, comment on the Issue using the legacy format (your GitHub login must be mapped to the `from` role):

```text
AGENT_TO: backend
AGENT_FROM: human
KIND: task

Implement /api/health inside backend/.
```

## Commands

| Command | Purpose |
|---|---|
| `init [--create-issue]` | write `.agentboard.json` (auto-detects repo/login), role folders, `.gitignore` entries |
| `team add/remove/list` | maintainer: map GitHub logins to roles |
| `setup --as ROLE [--start]` | teammate one-shot: checks + worktree + (optionally) listen |
| `doctor [--as ROLE]` | git/gh/claude/auth/config/worktree/budget/duplicate-listener checks |
| `post` / `reply` | post a task or message (`--dry-run` prints it; `--legacy` for header format; `reply` requires `--parent`) |
| `listen --as ROLE [--once] [--interval 20..3600] [--dry-run]` | run the listener (default 60s) |
| `listen --include-history` / `--replay-since ISO` | opt-in backfill; by default existing comments are skipped on first start |
| `status [--as ROLE] [--json]` | queue, blocked reasons, estimated spend, dispatch counts |
| `retry --as ROLE TASK_ID` | re-queue a blocked/failed task (also approves `needs_approval` tasks) |

Stop with Ctrl+C; start the same command to resume. A task that was running when a listener died becomes `blocked`
(`review_required`): inspect the worktree, then `retry`. Completed tasks are never replayed. Change budget with
`budget.daily_usd` (estimates; `0` refuses all dispatch). Defaults: `sonnet`; simple 8 turns/$0.75, standard 16/$2.50
(default), complex 24/$5 (needs `retry` approval); daily cap $20. These are CLI caps and estimates, not billing guarantees.

## Safety model

- Author check uses the real GitHub `user.login`; the self-declared `from` must be in that login's configured roles.
  This is necessary, not sufficient, for zero trust.
- Message `allowed_paths` only narrow the role's configured paths.
- After every run the worktree diff is checked: any path outside the role, a secret-looking string, or a file over 1 MB blocks the task
  (evidence in `.agentboard-state/evidence/`). A dirty worktree blocks the next task instead of being reset.
- Workers get Read/Grep/Glob/Edit/Write plus Bash only for the role's `test_command`; `git push`, `gh`, `curl` are denied where the
  installed CLI supports `--disallowedTools`. The worker environment drops `GH_*`, `GITHUB_*`, `AWS_*`, etc. `--dangerously-skip-permissions` is never used.
  A text rule is not a sandbox; for stronger isolation run listeners in a container/VM.
- Same-role listeners on two machines are unsupported (no distributed lease). One listener per role per checkout (file lock).
- Reports are redacted by pattern, which is not perfect.

## Troubleshooting

- `claude: MISSING` / `claude CLI not found`: install Claude Code and run `claude auth login`.
- `gh auth: NEEDS LOGIN`: `gh auth login`.
- `author 'x' may not speak as 'y'`: maintainer runs `team add x --roles y`, commits, pushes; you `git pull`.
- Task blocked on permission: add the exact command to `claude.extra_allowed_tools` or `test_command`; don't bypass permissions.
- Missing worktree/branch mismatch: re-run `python agentboard.py setup --as ROLE`.
- GitHub rate limits/outages: handled with capped backoff; unsent reports stay in a local outbox.

## Tests

```powershell
python -m unittest discover -s tests -v
python -m compileall -q agentboard.py tests
```

All tests are offline (mocked `gh` and Claude, real temporary git repos). There is no automated live GitHub/Claude test;
validate live with `doctor` then a `--once` run on a throwaway Issue.
