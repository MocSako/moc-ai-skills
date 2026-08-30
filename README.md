# moc-ai-skills

Agent skills I use daily across Cursor, Claude Code, and Codex. Mix of curated picks from builders I follow and ones I built from my own workflow.

I put this together on my free time because I needed it — something that actually runs like a personal assistant, not just another productivity template. It's been helping me stay on top of school, job searching, and everything in between. Sharing it in case it helps you too.

---

## Chiefofstaff

The main thing I built. A full personal chief of staff that lives inside your agent. Tell it what you need and it routes automatically.

| Sub-skill | What it does | Say this |
|---|---|---|
| `setup` | First-time onboarding — name, role, timezone, tools, goals | `"set me up"` |
| `daily-brief` | Morning kickoff — calendar, email, Slack, open tasks | `"plan my day"` |
| `daily-brief` | End-of-day wrap — what got done, what carries forward | `"wrap up"` |
| `meeting-prep` | Pre-meeting brief built from calendar, Slack, and email | `"prep for my [meeting]"` |
| `meeting-prep` | Post-meeting recap with action items and decisions | `"recap my meeting"` |
| `email` | Inbox triage — urgency classification, draft replies, never sends without you | `"check my email"` |
| `goal-tracker` | Log wins, track progress, weekly summaries | `"log: [what you did]"` |
| `network-tracker` | Contact log (CSV) — who you met, how, and when to follow up | `"I met [name] at [place]"` |
| `calendar-assistant` | Detect conflicts, protect focus time, flag heavy days | `"scan my calendar"` |
| `weekly-review` | Full week recap + top 3 priorities for next week | `"weekly review"` |

### What a morning brief looks like

```
── Tuesday, August 5 ───────────────────────────

MEETINGS  (2 today)
  9:00am   Coffee chat · Sarah @ Google
  2:00pm   Technical interview · Stripe · 📹

EMAIL  (3 to action)
  Stripe Recruiting · "Next steps — your interview today"
  → Confirm video link and reply
  Prof. Williams · "Project deadline moved up"
  → Respond and update your task list

OPEN TASKS
  · Finish cover letter for Figma role
  · Follow up with Alex from last week's meetup

FOCUS
  Interview at 2pm is the priority. Block your morning.

───────────────────────────────────────────────
```

### Install (Claude Code)

```bash
/plugin chiefofstaff
/reload-plugins
```

Then say `"set me up"` and it walks you through onboarding in about 2 minutes.

### Install (Cursor / Codex / other agents)

Paste this into your agent:

> Fetch and install the chiefofstaff skill from `https://github.com/MocSako/moc-ai-skills` into this agent. Use the correct skills directory for this runtime. After installing, run first-time setup.

---

## Plugins

Curated plugins I use alongside my skills. Each has its own README with install instructions.

| Plugin | What it does | Source |
|---|---|---|
| `remember` | Continuous memory — saves sessions automatically so Claude starts the next one with context already loaded | [claude.com/plugins/remember](https://claude.com/plugins/remember) |

---

## Skills

Curated skills I use alongside chiefofstaff. Mostly from builders I follow — listed here so they're in one place.

| Skill | What it does | Source |
|---|---|---|
| `grill-me` | Stress-tests a plan or design by drilling you with questions, round by round, until nothing is left silently assumed | [Matt Pocock](https://github.com/mattpocock/skills) |
| `grill-with-docs` | Same as grill-me, but also sharpens your domain language and writes ADRs and a glossary as decisions get made | [Matt Pocock](https://github.com/mattpocock/skills) |
| `deslop` | Scans the diff against main and strips AI-generated slop — unnecessary comments, defensive checks, `any` casts, deeply nested code | [Cursor Team Kit](https://github.com/getcursor/cursor) |
| `no-mistakes` | Local git proxy that runs a full validation pipeline (review, tests, lint, docs) before anything hits GitHub, then opens the PR automatically | [kunchenguid](https://github.com/kunchenguid/no-mistakes) |
| `teach` | Turns any directory into a stateful learning workspace — builds interactive HTML lessons, tracks progress across sessions, and adapts to your zone of proximal development | [Matt Pocock](https://github.com/mattpocock/skills) |

> `grill-me` and `grill-with-docs` depend on the `grilling` and `domain-modeling` sub-skills included in this repo.

---

## Credits

Built on the shoulders of people who share their work openly:

- [Matt Pocock](https://github.com/mattpocock) — skill-building patterns that shaped how I think about agent workflows
- [claude-code-templates](https://aitmpl.com) — great reference for commands and skill structure
- [Digital-Process-Tools](https://github.com/Digital-Process-Tools/claude-remember) — the remember plugin for continuous Claude Code memory

