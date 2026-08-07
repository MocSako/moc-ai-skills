---
name: chiefofstaff/agent-team-setup
description: "Interactive setup wizard for Claude Code agent teams. Walks through defining the team goal, choosing roles, drafting tasks, and spawning the team. Triggers on 'set up an agent team', 'create a team', 'spin up agents', 'multi-agent project', 'agent team', 'agent swarm', or any project that benefits from parallel agents."
argument-hint: "[describe your project goal, or 'start' to be guided]"
version: 1.0
type: skill
recommended-model: opus
category: productivity
user-invocable: true
---

# Agent Team Setup

Stand up a Claude Code agent team in one guided session. Goal → roles → tasks → spawn → handoff.

---

## Pre-flight — verify agent teams is enabled

```bash
[ -n "${CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS:-}" ] && echo ENABLED || echo DISABLED
```

If `DISABLED` and no `TeamCreate` tool is available, stop:

> Agent teams aren't enabled. Add this to `~/.claude/settings.json`:
> ```json
> { "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" } }
> ```
> Then restart Claude Code.

If `TeamCreate` is deferred, load it: `ToolSearch select:TeamCreate,TeamDelete,SendMessage`

---

## When to use a team vs. a single agent

Use a **team** when:
- Work splits into independent parallel tracks (frontend + backend, multi-file refactor, multi-source research)
- Different phases need different strengths (research → plan → implement → review)
- You want a builder + critic loop

Use a **single Agent call** when:
- It's a one-shot task or quick lookup
- There's no real parallelism to exploit

If a team isn't the right tool, say so and suggest the `Agent` tool instead.

---

## Step 1 — Capture the goal

Ask, one at a time:

1. What should the team deliver? (1-2 sentences)
2. What does "done" look like? (tests pass, PR opened, report written, etc.)
3. What directory or repo should the team work in?
4. Any hard constraints? (don't push to main, read-only only, specific files only)

Restate the goal in one sentence and confirm before moving on.

---

## Step 2 — Propose roles

Suggest 2-4 teammates based on the goal. Keep it small — more than 4 adds coordination overhead without adding throughput.

For each role:
- **Name** — short, lowercase, hyphenated (`researcher`, `backend-dev`, `reviewer`)
- **Agent type** — match to the work:
  - Read-only search → `Explore`
  - Planning/architecture → `Plan`
  - Implementation, file edits → `general-purpose` or `claude`
  - Code review → `code-reviewer` if available
- **Responsibility** — one line, what they own end-to-end
- **Constraints** — what they must NOT do (no git push, no deletions, etc.)

Rules:
- Never assign implementation to a read-only agent (`Explore`, `Plan`)
- Designate exactly one **team lead** — the main session (you) by default
- If you want a critic loop, name them `reviewer` with clear approve/reject criteria

Show as a table. Confirm before moving on.

---

## Step 3 — Draft the task list

Create an initial backlog of 3-8 tasks. Each task:
- Has an action-verb title ("Map all React imports in src/", "Refactor auth middleware")
- Names a single owner (role name or unassigned)
- Notes dependencies (blocks / blocked-by)
- Has a one-line definition of done

Order so earlier IDs unblock later ones — teammates claim lowest-ID available task first.

Confirm the backlog before creating it.

---

## Step 4 — Create the team

1. Call `TeamCreate` with `team_name` (kebab-case from the goal), agent type, and description
2. Read `~/.claude/teams/<team-name>/config.json` to confirm creation
3. Create `~/.claude/teams/<team-name>/setup-log.md`, append first entry
4. Create each task with owner assigned

---

## Step 5 — Spawn teammates

For each role (not the team lead), call the `Agent` tool with:
- `subagent_type`: the chosen agent type
- `team_name`: the team name
- `name`: the role name
- `description`: one-line responsibility
- `prompt`: self-contained briefing including:
  - Team goal (verbatim)
  - This teammate's role and what they own
  - Tool/behavior constraints
  - Where the task list lives (`~/.claude/tasks/<team-name>/`) — claim lowest-ID unassigned task first
  - Where the roster is (`~/.claude/teams/<team-name>/config.json`)
  - Use `SendMessage` to communicate, `TaskUpdate` to report progress
  - Any "do not" rules

Spawn independent teammates in a single message (parallel Agent calls).

---

## Step 6 — Hand off

Brief the user:

```
── Team ready · [team-name] ────────────────────

You are the team lead. Here's how to drive it:

  SendMessage to '<name>'  — assign work or give instructions
  TaskList                  — see backlog status
  TaskUpdate                — change owner or mark done

Teammates go idle between turns — send a message to wake them.
Messages from teammates auto-deliver as new turns.

When done, say "shut down the team" and I'll clean everything up.

────────────────────────────────────────────────
```

Offer to send the first kickoff message to whoever owns task #1.

---

## Step 7 — Shutdown

When the user signals the work is done:

1. Send `{ type: "shutdown_request", reason: "project complete" }` to each teammate via `SendMessage`
2. Wait for `shutdown_response` with `approve: true` from each
3. Append shutdown entry to setup log
4. Copy setup log to the working directory if the user wants a record
5. Call `TeamDelete` to clean up
6. Confirm: "Team shut down. Setup log saved to [path] if you want a record."

---

## Common team patterns

If the user isn't sure what shape to use:

| Pattern | Roles |
|---|---|
| Research + implement | `researcher` (Explore) + `implementer` (general-purpose) |
| Builder + critic | `builder` (general-purpose) + `reviewer` (general-purpose, read-only charter) |
| Parallel frontend + backend | `frontend-dev` + `backend-dev` (both general-purpose) |
| Plan + execute + verify | `planner` (Plan) + `implementer` (general-purpose) + `verifier` (general-purpose) |

---

## Rules to enforce throughout

- Refer to teammates by **name**, never UUID
- Use `SendMessage` to communicate — plain output is invisible to teammates
- Use `TaskUpdate` for progress — not JSON messages
- Keep teams small (2-4)
- One team lead only
- Clear "do not" rules per teammate, especially for destructive actions
