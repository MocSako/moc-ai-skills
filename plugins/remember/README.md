# remember

Continuous memory for Claude Code. Saves session state automatically so the next session starts with context already loaded — no re-explaining, no copy-pasting notes.

**Source:** [claude.com/plugins/remember](https://claude.com/plugins/remember)  
**By:** Digital-Process-Tools

## Install

```
/plugin remember
/reload-plugins
```

## What it does

- Hooks into Claude Code's lifecycle — saves sessions automatically
- Compresses history into layered daily summaries
- Loads context back at the start of the next session
- Run `/remember:doctor` to check if it's working correctly

## Why I use it

Claude Code starts every session blank by default. This plugin gives it continuity — it remembers what you worked on, what conventions you follow, what already broke. Essential if you're working on ongoing projects across multiple sessions.
