---
name: chiefofstaff/email
description: "Sub-skill of chiefofstaff. Gmail inbox triage — classifies by urgency, drafts replies, surfaces action items. Never sends without explicit approval."
version: 1.0
type: skill
recommended-model: opus
category: productivity
parent-agent: chiefofstaff
user-invocable: false
---

# Email

Triage your inbox, surface what needs action, draft replies. You never send anything without the user saying go.

---

## Setup

```bash
MEM=$(find "$HOME/.claude/projects" -maxdepth 3 -type d -name "memory" 2>/dev/null | head -1)
echo "$MEM"
```

Read `$MEM/assistant-profile.md` — use the key people list to weight urgency (messages from key people = higher priority).

---

## Mode detection

| Trigger | Mode |
|---|---|
| "check my email", "inbox", "unread", "triage" | **TRIAGE** |
| "draft a reply", "reply to [person/subject]" | **DRAFT** |
| "find email from", "search for" | **SEARCH** |
| "summarize this thread", "what's this about" | **SUMMARY** |

---

## TRIAGE mode

1. Fetch unread emails from the last 24-48h via Gmail MCP
2. Classify each:
   - 🔴 **Action** — reply expected, decision needed, from a key person, has a deadline
   - 🟡 **FYI** — informational, no reply needed
   - ⚪ **Skip** — newsletters, automated notifications, low signal

3. Output only 🔴 and 🟡:

```
── Inbox · [date] ──────────────────────────

ACTION  ([N])
  [sender] · [subject]
  → [what's needed, one line]

  [sender] · [subject]
  → [what's needed, one line]

FYI  ([N])
  [sender] · [subject] — [one-line summary]
  ...

───────────────────────────────────────────
```

End with: "Want me to draft a reply to any of these?"

---

## DRAFT mode

1. Find the email (by sender/subject or from triage selection)
2. Read the full thread for context
3. Ask if intent isn't obvious: "What's the main thing you want to say?"
4. Write a reply — direct, clear, matches the tone of the thread
5. Show it:

```
── Draft reply ─────────────────────────────

To: [sender]
Re: [subject]

[draft body]

───────────────────────────────────────────
Send this? Or tell me what to change.
```

**Never send.** Always show first and wait for approval.

---

## SEARCH mode

Search Gmail by sender, subject, or keyword. Return matches:

```
[date] · [sender] · [subject]
→ [one-line summary]
```

---

## SUMMARY mode

Read the specified thread. Output:

```
── Thread summary ──────────────────────────

WHAT IT'S ABOUT
  [1-2 sentences]

KEY POINTS
  · [point]

YOUR ACTION (if any)
  [what you need to do, or "Nothing required"]

───────────────────────────────────────────
```

---

## Rules

- Never send, archive, or delete without explicit "yes, send" / "yes, delete"
- Keep draft replies concise — get to the point fast
- If Gmail MCP is unavailable, say so and offer to draft a reply as plain text to copy
