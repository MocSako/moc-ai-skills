---
name: chiefofstaff/weekly-review
description: "Sub-skill of chiefofstaff. Structured weekly review and next-week planning. Triggers on 'weekly review', 'week in review', 'how was my week', 'plan next week', 'weekly wrap', 'reflect on my week', 'week recap'."
version: 1.0
type: skill
recommended-model: opus
category: productivity
parent-agent: chiefofstaff
user-invocable: false
---

# Weekly Review

Two modes: **REVIEW** (what happened this week) and **PLAN** (what matters next week). Run both together on Friday or Sunday for a full reset.

---

## Setup

```bash
MEM=$(find "$HOME/.claude/projects" -maxdepth 3 -type d -name "memory" 2>/dev/null | head -1)
echo "$MEM"
```

Read `$MEM/assistant-profile.md` for context on role, goals, and preferences.

---

## Mode detection

| Trigger | Mode |
|---|---|
| "weekly review", "week in review", "how was my week", "week recap", "reflect on my week" | **REVIEW** |
| "plan next week", "what should I focus on next week", "next week priorities" | **PLAN** |
| "weekly wrap", no clear mode, or both | **REVIEW then PLAN** |

Default to running both back-to-back — that's the most useful pattern.

---

## REVIEW mode

### Gather (run in parallel)

1. **Goals/wins** — read `$MEM/assistant-goals.md`, pull accomplishments logged this week (entries from last 7 days)
2. **Todos** — read `$MEM/assistant-todos.md`, pull tasks completed this week and still-open items
3. **Calendar** — via Google Calendar MCP, fetch events from the past 7 days (meetings, calls, key events)
4. **Network** — read `$MEM/assistant-network.csv`, check if any contacts were added or last_contact updated this week

Skip any source that's unavailable; note it briefly.

### Output

```
── Week in Review · [Mon Date] – [Sun Date] ────

WINS THIS WEEK
  · [accomplishment from goal-tracker]
  · [accomplishment]

COMPLETED TASKS
  · [task marked done]
  · [task marked done]

KEY MEETINGS / MOMENTS
  · [meeting or event worth noting]
  · [meeting or event]

STILL OPEN
  · [task]
  · [task]

PATTERNS
  [1–2 honest sentences: What went well? What kept slipping? Any recurring blockers?]

────────────────────────────────────────────────
```

If a section has no entries, drop it entirely — don't show empty headers.

After output, ask: "Want to plan next week now?"

---

## PLAN mode

### Gather

1. **Calendar** — via Google Calendar MCP, fetch next 7 days (what's already locked in)
2. **Open todos** — pull remaining open items from `$MEM/assistant-todos.md`
3. **Goals** — read `$MEM/assistant-goals.md` to see which goals have had no recent activity

### Output

```
── Next Week · [Mon Date] – [Sun Date] ─────────

TOP 3 PRIORITIES
  1. [Most important thing — specific and actionable]
  2. [Second priority]
  3. [Third priority]

ALREADY ON THE CALENDAR
  [Day]  [time]  [event]
  [Day]  [time]  [event]

CARRY-FORWARD TASKS
  · [open task from this week]
  · [open task]

GOALS NEEDING ATTENTION
  · [Goal with no recent wins — worth touching this week]

WATCH OUT FOR
  · [Heavy day, early meeting, known blocker, or deadline]

────────────────────────────────────────────────
```

After output, ask: "Want me to add any of these priorities to your task list?"

---

## Writing priorities

Good priorities are specific and completable:
- "Finish first draft of cover letter for [Company]" not "work on job apps"
- "Send follow-up to [Name] from last week's coffee chat" not "network more"
- "Complete module 3 of [course]" not "study"

If the user's priorities are vague, gently push for specifics: "Want to make that more concrete so you can actually check it off?"

---

## Weekly review file (optional log)

If the user says "save this" or "log my review", write a snapshot to `$MEM/assistant-weekly-reviews.md`:

```markdown
## Week of [Mon Date]

### Wins
- [accomplishment]

### Still Open
- [task]

### Priorities for Next Week
1. [priority]
2. [priority]
3. [priority]

### Reflection
[patterns note]
```

Append — never overwrite past entries.

---

## Rules

- Never invent accomplishments or completions — only use what's in the data
- If goal-tracker and todos are empty, still run the review using calendar only, and note what's missing
- Keep the output scannable — this should take 60 seconds to read, not 5 minutes
