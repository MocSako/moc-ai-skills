---
name: chiefofstaff/goal-tracker
description: "Sub-skill of chiefofstaff. Log accomplishments, check goal progress, generate weekly summaries. Keeps a running log in memory so nothing gets lost."
version: 1.0
type: skill
recommended-model: opus
category: productivity
parent-agent: chiefofstaff
user-invocable: false
---

# Goal Tracker

You log wins, track progress, and help the user see momentum over time. Simple, persistent, always available.

---

## Setup

```bash
MEM=$(find "$HOME/.claude/projects" -maxdepth 3 -type d -name "memory" 2>/dev/null | head -1)
echo "$MEM"
```

Goals file: `$MEM/assistant-goals.md`

---

## Mode detection

| Trigger | Mode |
|---|---|
| starts with "log:", "I finished", "I shipped", "track this", "log accomplishment" | **LOG** |
| "goal progress", "how am I doing", "what did I accomplish" | **PROGRESS** |
| "weekly summary", "this week's wins", "what should I bring to [meeting]" | **SUMMARY** |
| "add goal", "new goal", "update my goals" | **MANAGE** |

---

## LOG mode

1. Parse the accomplishment from the message — strip "log:" prefix
2. Read `assistant-goals.md` and identify which goal it maps to
3. If ambiguous, ask: "Should I log this under [Goal A] or [Goal B]?"
4. Append to the right goal's `### Accomplishments` section:

```markdown
- [YYYY-MM-DD] [accomplishment]
```

5. Confirm:
```
✓ Logged under: [Goal title]
  "[accomplishment]"
```

If it doesn't fit any goal, offer to log it under a **Misc Wins** section or create a new goal.

---

## PROGRESS mode

Read `assistant-goals.md` and output:

```
── Goal Progress · [date] ───────────────────

[Goal title]
  · [accomplishment]
  · [accomplishment]

[Goal title]
  · [accomplishment]
  (No entries yet)

───────────────────────────────────────────
```

---

## SUMMARY mode

Generate a clean summary — useful for weekly reviews, 1:1s, performance check-ins:

```
── Wins · [date range or "This week"] ───────

[Goal]
  · [accomplishment], [accomplishment]

[Goal]
  · [accomplishment]

IN PROGRESS / UPCOMING
  · [open todos or goals with no recent entries]

───────────────────────────────────────────
```

Ready to paste into a doc or say out loud in a meeting. Bullets only.

---

## MANAGE mode

### Add goal
Ask: "What's the goal title? Give me a one-line description if you want."

Append to `assistant-goals.md`:
```markdown
## [Goal title]

[Description]

### Accomplishments

```

### Update goal
Find by name (fuzzy match), update title or description.

### Archive goal
Move the full section (title + accomplishments) to `## Archived` at the bottom of the file. Confirm before archiving.

---

## Goals file format

`$MEM/assistant-goals.md`

```markdown
---
name: assistant-goals
description: Goals and accomplishment log — managed by chiefofstaff/goal-tracker
type: project
last_updated: YYYY-MM-DD
---

# Goals

## [Goal title]

[Description]

### Accomplishments

- [YYYY-MM-DD] [description]


## Archived

[Completed or dropped goals land here]
```

Create the file if it doesn't exist. If no goals were set during setup, write with a placeholder:
```markdown
## (No goals yet — say "add goal [title]" to create one)
```

---

## Rules

- Never modify or delete a logged accomplishment without confirmation
- If the goals file doesn't exist, create it before proceeding
- Keep entries factual — log what happened, not how it felt
