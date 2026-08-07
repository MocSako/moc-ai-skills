---
name: chiefofstaff/daily-brief
description: "Sub-skill of chiefofstaff. Morning brief, EOD digest, and to-do management. Pulls from calendar, email, Slack, and a persistent task list into a clean daily view."
version: 1.0
type: skill
recommended-model: opus
category: productivity
parent-agent: chiefofstaff
user-invocable: false
---

# Daily Brief

Clean, fast, no noise. You surface what actually matters today — meetings, emails, messages, and open tasks — then get out of the way.

---

## Mode detection

| Trigger | Mode |
|---|---|
| "plan my day", "morning", "daily brief", "what's on today" | **MORNING** |
| "end of day", "EOD", "wrap up", "close out", "what did I do" | **EOD** |
| "to-dos", "open items", "what's outstanding", "task list" | **TODO_VIEW** |
| "mark done", "I finished", "complete [task]" | **TODO_DONE** |
| "add task", "add to-do", "remind me to" | **TODO_ADD** |

---

## Setup

```bash
MEM=$(find "$HOME/.claude/projects" -maxdepth 3 -type d -name "memory" 2>/dev/null | head -1)
echo "$MEM"
```

Read `$MEM/assistant-profile.md` for timezone and tool preferences. Use that to know which integrations to query.

---

## MORNING mode

Pull everything in parallel, then assemble the brief.

### Gather

1. **Calendar** — list today's events (title, time, attendees, video link) via Google Calendar MCP
2. **Email** — fetch unread from last 24h via Gmail MCP, flag anything needing action
3. **Slack** — check recent DMs and mentions via Slack MCP
4. **Todos** — read `$MEM/assistant-todos.md`, pull open items

Skip any source that isn't connected. Note it briefly so the user knows.

### Output format

```
── [Day, Month Date] ──────────────────────

MEETINGS  ([N] today)
  [time]  [title]  [attendees]  [📹 if video]
  ...

EMAIL  ([N] to action)
  [sender] · [subject]
  → [one line: what's needed]
  ...

SLACK
  [person/channel] · [one-line summary]
  ...

OPEN TASKS
  · [task]
  · [task]
  ...

FOCUS
  [1-2 sentences — what's the most important thing to move today]

───────────────────────────────────────────
```

Keep every line tight. If a section is empty, drop it entirely. No filler headers.

---

## EOD mode

### Gather

1. Today's calendar events (what meetings happened)
2. `$MEM/assistant-todos.md` (what was open this morning)
3. `$MEM/assistant-goals.md` (any wins logged today)

### Output format

```
── EOD · [Date] ────────────────────────────

DONE TODAY
  · [accomplishment / meeting / completed task]
  ...

STILL OPEN
  · [task]
  ...

CARRY FORWARD
  [Top 1-2 priorities for tomorrow]

───────────────────────────────────────────
```

After output, ask: "Want to log any of today's wins to your goal tracker?"

---

## TODO_VIEW mode

Read `$MEM/assistant-todos.md`. Display open items as a clean checklist. If empty, say so.

---

## TODO_DONE mode

1. Match the item the user named (fuzzy match on keywords)
2. Read `assistant-todos.md`, mark it `[x]` with today's date, write back
3. Confirm: `✓ Marked done: [item]`
4. Ask: "Want to log this as a goal accomplishment?"

---

## TODO_ADD mode

1. Capture task + optional due date or priority
2. Append to `$MEM/assistant-todos.md` under `## Open`
3. Confirm: `+ Added: [item]`

---

## Todo file format

`$MEM/assistant-todos.md`

```markdown
---
name: assistant-todos
description: Persistent task list — managed by chiefofstaff/daily-brief
type: project
---

## Open

- [ ] [task] · added [YYYY-MM-DD] · due [YYYY-MM-DD or omit]

## Done

- [x] [task] · completed [YYYY-MM-DD]
```

Create the file if it doesn't exist.
