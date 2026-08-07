---
name: chiefofstaff/skill-builder
description: "Interactive skill creator — walks you through designing and building a new Claude Code skill or agent, generates the SKILL.md with proper structure and best practices. Triggers on 'create a skill', 'build a skill', 'new skill', 'scaffold a skill', 'make a skill', 'skill builder'."
argument-hint: "[describe your skill idea, or 'start' to be guided]"
version: 1.0
type: skill
recommended-model: opus
category: productivity
user-invocable: true
---

# Skill Builder

Build new Claude Code skills — interactively, following best practices, generating clean SKILL.md files ready to use or share.

---

## Step 0 — What do you want to do?

Ask:

> **What would you like to do?**
>
> **A) Create a new skill** — one specific thing, done well
> **B) Create a new agent** — orchestrates multiple skills into a workflow
> **C) Update an existing skill** — improve or fix something you already built
>
> **Skill vs agent?**
> | Build a **skill** when... | Build an **agent** when... |
> |---|---|
> | It does one specific thing | It orchestrates multiple steps |
> | User knows exactly when to invoke it | User describes a high-level goal |
> | It does the work itself | It routes to other skills |

Route accordingly. For **update**, ask which skill and what to change, then skip to Step 3.

---

## Step 1 — Discovery (ask one at a time)

**What does it do?**
> Describe what the skill should do in a sentence or two.

Silently classify the pattern (don't tell the user):
- **Document/Asset creation** — output is a file, report, doc
- **Workflow automation** — multi-step sequential process
- **Multi-service coordination** — multiple tools or APIs
- **Data & analysis** — queries, metrics, charts
- **Domain intelligence** — validation, specialized knowledge

**What should it be called?**
> Suggest a kebab-case name based on their description. Names should be lowercase with hyphens (e.g. `budget-tracker`, `pr-reviewer`).

**Starting point:**
> Have you already done this task successfully in a conversation?
> - Yes → paste or describe what worked, I'll extract it into a skill
> - No → I'll design the workflow from scratch

**Which tools does it need?**
> Does it use any integrations? (e.g. Google Calendar, Gmail, Slack, GitHub, a web search, file system)

---

## Step 2 — Generate the spec

From the user's answers, infer everything possible:

| Field | How to determine it |
|---|---|
| **Description** | What it does + when to use it + 3-4 trigger phrases. Keep under 300 chars. |
| **Workflow steps** | Break down the task into numbered steps with clear inputs/outputs |
| **Error handling** | For each step that can fail, define what happens |
| **Prerequisites** | Anything the user must have configured first |

Show the spec:

```
── Skill spec ──────────────────────────────────

Name:         [kebab-case-name]
Type:         [Skill / Agent]
Description:  [trigger-rich description]
Version:      1.0
Dependencies: [list or "None"]

Workflow:
  1. [Step — expected outcome]
  2. [Step — expected outcome]
  ...

Error handling:
  - [Step X] fails → [what happens]

Files:
  ~/.claude/skills/[name]/SKILL.md

────────────────────────────────────────────────
```

Ask: "Does this look right? Any changes before I generate?"

Iterate until approved.

---

## Step 3 — Generate the SKILL.md

Write to `~/.claude/skills/[name]/SKILL.md`:

```markdown
---
name: [name]
description: "[keyword-rich description with trigger phrases]"
argument-hint: "[what to pass as an argument]"
version: 1.0
type: [skill / agent]
recommended-model: [sonnet / opus]
category: [productivity / engineering / research / creative / generic]
---

# [Skill Title]

[One-line description of what it does.]

---

## [Step 1: First step]

[Specific, actionable instructions]

[Expected output: what success looks like]

## [Step 2: Next step]

...

## Error Handling

### [Step X] fails

**Cause:** [Why it might fail]
**Solution:** [What to do]
```

If the skill is an agent, also include:
- `type: agent` and `recommended-model: opus` in frontmatter
- Autonomy tiers table (AUTONOMOUS / CONFIRM-THEN-ACT / PRESENT-AND-WAIT)
- Sub-skills routing table

Tell the user what was written:

```
── Created ─────────────────────────────────────

  ~/.claude/skills/[name]/SKILL.md

  Try it: /[name]

────────────────────────────────────────────────
```

---

## Step 4 — Validate

Run a quick sanity check on the generated file:

- [ ] Frontmatter has `---` delimiters and valid YAML
- [ ] `name` is kebab-case, matches folder name
- [ ] `description` exists and is under 1024 chars
- [ ] `version` is in X.Y format
- [ ] No hardcoded absolute paths (use `$HOME` or relative)
- [ ] Steps are numbered and have clear expected outputs
- [ ] Error handling exists for any step that touches external services

Show a pass/fail for each check. Fix any failures automatically.

---

## Step 5 — What's next?

> Your skill is ready. Here's what you can do:
>
> - **Use it now** → type `/[name]`
> - **Add it to a plugin** → move the folder to your plugin's `skills/` directory
> - **Share it** → push to GitHub and link in a README

Offer to also add the skill to the `chiefofstaff` routing table if it fits the assistant theme.
