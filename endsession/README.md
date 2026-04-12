# /endsession

Wraps up your Claude Code session by saving what's worth keeping, cleaning up stale memories, and preparing you to run `/clear`.

## Why this exists

When you end a Claude Code session without saving context, the next session starts from scratch — no memory of decisions made, feedback given, or work in progress. This command automates the cleanup so nothing gets lost.

## How to use

```
/endsession
```

No arguments needed. Run it at the end of any session, in any project.

## What it does

The command runs four steps in order:

### Step 1 — Update project files

If your project has a `CLAUDE.md` with session-end instructions (e.g. "update the worklog"), it follows them. Only updates files that actually need updating. If there are no such instructions, this step is skipped.

### Step 2 — Save new memories

Reviews the session for information worth remembering: your preferences, feedback you gave, project context, and pointers to external tools. Checks existing memories first to avoid duplicates.

### Step 3 — Clean up stale memories

Removes only memories that are obviously duplicated or clearly outdated. When in doubt, it keeps the memory — deletion is permanent.

### Step 4 — Summary

Reports what was updated, saved, and cleaned. Tells you to run `/clear`.

## Typical workflow

```
/endsession     ← persist context, clean memory
/clear          ← start fresh
```

## Good to know

- The command **never deletes anything it's unsure about**. Better one extra memory than a lost insight.
- It adapts to your project. If your `CLAUDE.md` defines a worklog or decisions file, the command maintains them. If not, it skips that step and suggests adding one.
- `/endsession` cannot run `/clear` automatically — Claude Code doesn't allow slash commands to chain. You run `/clear` yourself after reviewing the summary.

## Installation

See the [main README](../README.md#installation) for installation instructions.
