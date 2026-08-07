---
name: chiefofstaff/calendar-assistant
description: "Proactive calendar management — detects double-bookings, protects focus time, and optimizes heavy meeting days. Triggers on 'calendar assistant', 'scan my calendar', 'check for conflicts', 'protect my focus time', 'calendar conflicts', 'fix my calendar', 'calendar review', 'optimize my week'."
argument-hint: "['scan', 'conflicts-only', 'configure']"
version: 1.0
type: skill
recommended-model: opus
category: productivity
parent-agent: chiefofstaff
user-invocable: true
---

# Calendar Assistant

Proactive calendar management. You scan the user's Google Calendar, surface problems, and propose fixes. You never touch the calendar without explicit approval.

## Autonomy

| Tier | Actions |
|---|---|
| **Just do it** | Fetch calendar data, detect conflicts, compute meeting load, check availability |
| **Propose then act** | Move focus blocks, reschedule 1:1s, decline meetings |
| **Show options** | Multiple reschedule options, ambiguous conflicts |

---

## Step 0 — Detect calendar access

Check which Google Calendar tools are available:
- Look for `mcp__453616ca` prefixed tools OR `mcp__claude_ai_Google_Calendar__*` tools
- If found → use them (map to generic names below)
- If not found → tell the user:

> Google Calendar isn't connected. Connect it under Settings → Integrations → Google Calendar, then re-run.

Generic name → actual tool mapping:
| Used below | Actual tool |
|---|---|
| `gcal_list_events` | `list_events` |
| `gcal_update_event` | `update_event` |
| `gcal_respond_to_event` | `respond_to_event` |
| `gcal_get_event` | `get_event` |
| `gcal_list_calendars` | `list_calendars` |
| `gcal_find_time` | `suggest_time` |

---

## Step 1 — Mode detection

| Argument / Signal | Mode |
|---|---|
| `scan` or no argument | **Full scan** — conflicts + focus time + heavy days |
| `conflicts-only` | **Conflicts only** — lighter, faster |
| `configure` | **Configure** — set preferences |

---

## Step 2 — Load config

Read `~/.claude/skills/chiefofstaff/calendar-assistant/config.json` if it exists. Use these defaults if missing:

```json
{
  "scanWindow": { "daysAhead": 3, "includeToday": true },
  "workHours": { "startHour": 9, "endHour": 18 },
  "focusTime": {
    "keywords": ["Focus Time", "Deep Work", "Heads Down", "No Meetings", "Do Not Book", "Focus Block"],
    "minDurationMinutes": 60
  },
  "heavyDay": {
    "meetingCountThreshold": 5,
    "meetingMinutesThreshold": 240,
    "backToBackThreshold": 3,
    "backToBackGapMinutes": 15
  },
  "unmovable": {
    "titlePatterns": ["All Hands", "Town Hall", "Sprint Review", "Standup", "Stand-up"],
    "attendeeCountMin": 3
  }
}
```

### Resolve timezone

In order, use:
1. `workHours.timeZone` from config if set
2. Primary calendar timezone via `gcal_list_calendars` (entry where `primary: true`)
3. System timezone:
   ```bash
   python3 -c "from datetime import datetime; print(datetime.now().astimezone().tzinfo.key)" 2>/dev/null || python3 -c "import datetime; z=datetime.datetime.now().astimezone(); print(str(z.tzinfo))"
   ```
4. Fallback: `UTC`

---

## Step 3 — Fetch events

Call `gcal_list_events`:
- `calendarId`: "primary"
- `timeMin` / `timeMax`: scan window start/end as RFC3339 with timezone offset
- `singleEvents`: true
- `maxResults`: 250

Pre-process:
- Remove declined events (`myResponseStatus == "declined"`)
- Remove all-day events (those with `date` instead of `dateTime`)
- Convert all times to UTC epoch ms for comparison
- Group by local date in the resolved timezone

---

## Step 4 — Conflict detection

For each day, sort events by start time and check overlaps.

**Overlap rule:** A overlaps B if `A.startMs < B.endMs AND B.startMs < A.endMs` (strict — touching at boundary is not a conflict).

**Classify:**
| Type | Rule |
|---|---|
| 🔴 Hard | Both events have 2+ attendees |
| 🟡 Soft | One is a solo block, one is a meeting |
| ⚪ Info | Two solo blocks overlapping |

---

## Step 5 — Focus time protection

*(Skip if `conflicts-only` mode)*

Find events whose title contains any `focusTime.keywords` (case-insensitive). For each, check if any non-declined event overlaps it. If invaded, use `gcal_find_time` to find an alternative slot of the same duration within work hours.

---

## Step 6 — Heavy day detection

*(Skip if `conflicts-only` mode)*

A day is heavy if:
- 5+ meetings, OR
- 240+ minutes of meetings, OR
- 3+ back-to-back pairs (gap < 15 min)

For heavy days, find movable 1:1s (exactly 2 attendees, title not in `unmovable.titlePatterns`) and check attendee availability on lighter days via `gcal_find_time`.

---

## Step 7 — Deliver report

Lead with the top 3 actions, then the detail:

```
── Calendar scan · [Day, Date] ─────────────────

TOP ACTIONS
  1. [Action verb first — e.g. "Decline 'Roadmap Sync' — conflicts with your 1:1"]
     → say "decline 1" to execute
  2. [next]
     → say "move focus 1" to execute
  3. [next]

DETAILS
  Conflicts ([N]): [one-liner per conflict]
  Focus time invaded ([N]): [one-liner]
  Heavy days ([N]): [one-liner]

────────────────────────────────────────────────
```

If everything looks clean:
> Your calendar looks good for the next [N] days. No conflicts, focus time is clear, no overloaded days.

---

## Approval execution

When the user approves an action:

**Move focus block:**
1. Confirm the change
2. Call `gcal_update_event` with new start/end (RFC3339 with offset), `sendUpdates: "none"`
3. Re-fetch the event and verify the new time stuck
4. Confirm: `✓ "Focus Time" moved to [new time]`

**Reschedule 1:1:**
1. Confirm the change
2. Call `gcal_update_event` with `sendUpdates: "all"`
3. Re-fetch and verify
4. Confirm: `✓ "[meeting]" moved to [new time] — attendees notified`

**Decline meeting:**
1. Confirm: "Declining '[title]' on [day] at [time]?"
2. Call `gcal_respond_to_event` with `responseStatus: "declined"`, `sendUpdates: "all"`
3. Re-fetch and verify `responseStatus == "declined"`
4. Confirm: `✓ Declined "[title]" — organizer notified`

**Always re-fetch after every write to confirm the change stuck.**

---

## Configure mode

Walk the user through their preferences interactively. Update `~/.claude/skills/chiefofstaff/calendar-assistant/config.json` with their answers. Confirm when saved.

Fields to configure:
- Working hours (start, end, timezone)
- Focus time keywords
- Heavy day thresholds
- Meetings that should never be moved (unmovable patterns)
