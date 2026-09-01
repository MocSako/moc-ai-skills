# code-simplifier

Simplifies and refines code for clarity, consistency, and maintainability while preserving all functionality.

**Source:** [claude.com/plugins/code-simplifier](https://claude.com/plugins/code-simplifier)  
**By:** Anthropic (Official)

## Install

```
/plugin code-simplifier
/reload-plugins
```

## What it does

- Reviews recently modified code for unnecessary complexity, redundancy, and readability issues
- Applies project-specific coding standards from CLAUDE.md
- Preserves exact functionality — only changes how the code does something, never what it does
- Runs autonomously after code is written or modified
- Avoids over-simplification (no nested ternaries, no dense one-liners over clarity)

## When it triggers

Automatically after edits, or invoke manually via `/simplify`.
