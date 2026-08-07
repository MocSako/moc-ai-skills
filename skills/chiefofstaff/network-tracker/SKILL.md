---
name: chiefofstaff/network-tracker
description: "Sub-skill of chiefofstaff. Tracks the people you've met and networked with — contact info, how you met, and last-touch date — in a CSV file you can open directly in Excel/Sheets. Log new contacts, look people up, and see who's overdue for a follow-up."
version: 1.0
type: skill
recommended-model: opus
category: productivity
parent-agent: chiefofstaff
user-invocable: false
---

# Network Tracker

A running contact log — who you've met, how you met them, and when you last talked. Stored as a real CSV so the user can open it straight in Excel, Numbers, or Google Sheets whenever they want, not just through chat.

---

## Setup

```bash
MEM=$(find "$HOME/.claude/projects" -maxdepth 3 -type d -name "memory" 2>/dev/null | head -1)
echo "$MEM"
```

Network file: `$MEM/assistant-network.csv`

If it doesn't exist, create it with the header row before doing anything else:

```csv
name,how_met,date_met,company,role,email,phone,linkedin,tags,last_contact,follow_up_date,notes
```

---

## Mode detection

| Trigger | Mode |
|---|---|
| "add contact", "I met", "log this person", "new contact", "add to my network" | **ADD** |
| "show my network", "list contacts", "who have I met", "my contacts" | **LIST** |
| "who do I know at [company]", "find [name]", "look up [name]", "do I know anyone at" | **SEARCH** |
| "update [name]", "talked to [name] again", "change [name]'s email/phone" | **UPDATE** |
| "who do I need to follow up with", "who haven't I talked to", "overdue follow-ups" | **FOLLOW-UP** |
| "open my network sheet", "export contacts", "where's my contact list" | **EXPORT** |

---

## ADD mode

Pull whatever the user gives you in one shot; ask only for what's missing and clearly useful. Don't interrogate — this should feel like jotting a note, not filling out a form.

**Always try to get:**
- Name
- How you met (conference, intro'd by X, cold outreach, alumni event, etc.)
- Date met (default to today if not given)

**Ask only if not offered, and only once, lightweight:**
> Got it — anything else I should save? Company, role, email/phone, LinkedIn, or a quick note on them? Skip anything you don't have.

Never block on missing fields. Append a row even if only name + how_met + date_met are known.

**Append to `assistant-network.csv`:**
- One row per contact.
- `last_contact` = same as `date_met` for a new entry.
- `follow_up_date` = blank unless the user specifies one ("remind me to follow up in 2 weeks" → compute the date).
- `tags` = free-form, semicolon-separated if multiple (e.g. `investor;design;sf`).
- CSV-escape any field containing a comma or quote (wrap in double quotes, double any internal quotes).

Confirm:
```
✓ Added [Name] to your network — met [date] via [how_met].
```

---

## LIST mode

Read `assistant-network.csv` and render as a table, most recently met first:

```
── Your Network · [count] contacts ──────────────

Name           Company        How met                Last contact
─────────────────────────────────────────────────────────────────
[Name]         [Company]      [how_met]               [last_contact]
[Name]         [Company]      [how_met]               [last_contact]

──────────────────────────────────────────────────
```

If empty: "No contacts logged yet — say 'I met [name] at [place]' and I'll start your network sheet."

---

## SEARCH mode

Filter rows by name (fuzzy match), company, tag, or role as implied by the query. Show matching rows in the same table format. If nothing matches, say so plainly — don't guess.

---

## UPDATE mode

Find the contact by name (fuzzy match; if more than one match, ask which). Update only the fields mentioned. If the update is "talked to [name] again" / "caught up with [name]", set `last_contact` to today and leave everything else untouched.

Confirm:
```
✓ Updated [Name] — [field(s) changed]
```

---

## FOLLOW-UP mode

Scan the CSV for:
1. Any row with a `follow_up_date` that is today or earlier
2. Any row with `last_contact` older than 90 days (configurable — ask the user once if they want a different window, and remember their answer in this file's own notes going forward)

```
── Follow-ups ──────────────────────────────

Overdue reminders
  · [Name] — follow-up was due [date]

Gone quiet (90+ days)
  · [Name] — last contact [date]

─────────────────────────────────────────────
```

If none, say so — don't show an empty section.

---

## EXPORT mode

Tell the user exactly where the file lives and that it's a plain CSV:

> Your network sheet is at `[path to assistant-network.csv]` — open it directly in Excel, Numbers, or Google Sheets, or just ask me to add/look up people here and I'll keep it in sync.

---

## Network file format

`$MEM/assistant-network.csv`

```csv
name,how_met,date_met,company,role,email,phone,linkedin,tags,last_contact,follow_up_date,notes
"Jane Doe","Intro'd by Alex at TechCrunch Disrupt",2026-03-14,"Acme Inc","VP Product","jane@acme.com","","linkedin.com/in/janedoe","investor;product",2026-03-14,,"Interested in our roadmap"
```

- Create the file with just the header row if it doesn't exist.
- Never delete a row without explicit confirmation.
- Keep dates in `YYYY-MM-DD`.

---

## Rules

- This is a real spreadsheet, not a hidden log — always be ready to tell the user where the file is.
- Don't merge this with `assistant-profile.md`'s "Key people" section; that's a short list for personalization, this is the full network log. If a name appears in both, that's fine — no need to reconcile them.
- Keep entries factual. Don't infer relationship warmth or interest level unless the user says so.
