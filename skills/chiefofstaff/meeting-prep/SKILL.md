---
name: chiefofstaff/meeting-prep
description: "Sub-skill of chiefofstaff. Pre-meeting briefs and post-meeting recaps. Pulls context from calendar, Slack, email, and memory to build a focused brief for any meeting type."
version: 1.0
type: skill
recommended-model: opus
category: productivity
parent-agent: chiefofstaff
user-invocable: false
---

# Meeting Prep

Two modes: **PREP** before a meeting, **RECAP** after. Clean, structured, no padding.

---

## Setup

```bash
MEM=$(find "$HOME/.claude/projects" -maxdepth 3 -type d -name "memory" 2>/dev/null | head -1)
echo "$MEM"
```

Read `$MEM/assistant-profile.md` for key people and tool preferences.

---

## PREP mode

### Step 1 — Identify the meeting

Parse meeting name/person from the request. If ambiguous, check Google Calendar for upcoming events today and ask which one.

### Step 2 — Classify

| Type | Examples |
|---|---|
| `one_on_one` | 1:1 with manager, mentor, report, peer |
| `standup` | daily standup, team sync |
| `review` | sprint review, design review, code review, demo |
| `interview` | job interview, candidate screen |
| `client` | client call, external partner |
| `skip_level` | meeting with someone 2+ levels above |
| `general` | anything else |

### Step 3 — Gather (parallel)

- **Calendar**: title, attendees, time, existing agenda/description
- **Slack**: recent DMs and threads with the attendees
- **Email**: recent email threads with attendees
- **Goals/todos**: open items that might come up (`assistant-goals.md`, `assistant-todos.md`)

Skip any source that's unavailable.

### Step 4 — Build the brief

**one_on_one:**
```
── 1:1 with [name] · [time] ────────────────

BRING UP
  · [accomplishment or update worth sharing]
  · [open question or ask]
  · [anything from Slack/email threads with this person]

ANTICIPATE
  · [topics they might raise based on recent context]

ASK
  · [specific question you want answered]

───────────────────────────────────────────
```

**standup:**
```
── Standup · [time] ────────────────────────

YESTERDAY  [what you worked on]
TODAY      [what you're working on]
BLOCKERS   [anything blocking you, or "None"]

───────────────────────────────────────────
```

**review / demo:**
```
── [Meeting title] · [time] ────────────────

WHAT TO SHOW
  · [completed work relevant to this review]

CONTEXT FOR THE ROOM
  · [anything attendees need to know going in]

QUESTIONS TO EXPECT
  · [anticipated questions based on the work]

───────────────────────────────────────────
```

**interview:**
```
── Interview · [time] ──────────────────────

ABOUT THEM
  · [attendee names, roles if available]

TALKING POINTS
  · [relevant experience or projects to mention]

QUESTIONS TO ASK
  · [thoughtful questions about the role/team]

───────────────────────────────────────────
```

**skip_level / general:**
```
── [Meeting title] · [time] ────────────────

WHO'S IN THE ROOM
  · [attendees + their relationship to you]

AGENDA
  [From calendar description, or inferred]

TALKING POINTS
  · [what's worth raising]

───────────────────────────────────────────
```

---

## RECAP mode

### Step 1 — Get notes

Ask: "Paste your notes or describe what happened."

### Step 2 — Output

```
── Recap · [meeting title] · [date] ────────

KEY TAKEAWAYS
  · [point]

YOUR ACTION ITEMS
  · [ ] [task] · due [date if mentioned]

DECISIONS MADE
  · [point]

───────────────────────────────────────────
```

After output:
- "Want me to add these action items to your task list?"
- "Want to log any wins to your goal tracker?"

---

## General rules

- Never invent context — flag gaps rather than fill them with assumptions
- Keep briefs scannable, not exhaustive
- For standups, 3 lines max — it's a standup, not an essay
