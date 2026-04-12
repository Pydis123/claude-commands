# /cleanclaudemd

Audits your project's `CLAUDE.md` file and shows you exactly where you're spending tokens — and what you can safely trim.

## Why this matters

Your `CLAUDE.md` is loaded into **every single message** in a Claude Code session. A 2,000-word file costs roughly 2,600 tokens per message. Over a 100-message session, that's 260,000 tokens just from instructions.

This command finds what can be trimmed, condensed, or moved to external files that Claude only reads when needed.

## How to use

```
/cleanclaudemd
```

No arguments needed. Run it in any project that has a `CLAUDE.md`.

## What you get

The command produces a report with three parts:

1. **Token Map** — Every file loaded into your context on each message (your `CLAUDE.md`, any `@`-includes, parent directory files, global `~/.claude/CLAUDE.md`), with word counts and estimated token costs.

2. **Findings** — Issues found, by severity:
   - **WARNING** — Dead file references, obsolete sections that no longer match the codebase
   - **SUGGESTION** — Duplicated content, verbose text, sections that could be lazy-loaded

3. **Suggested Actions** — Concrete proposed changes: remove, condense, extract to external file, or replace `@`-include with a plain reference. Each shows exactly what would change.

The command **never modifies your files** without your explicit approval. You review the report first, then choose which changes to apply.

## Key concept: token cost levels

| Level | How it works | When tokens are spent |
|-------|-------------|----------------------|
| **Inline** | Text written directly in CLAUDE.md | Every message |
| **@-include** | `@FILE.md` reference in CLAUDE.md | Every message (auto-expanded) |
| **Reference** | "See `FILE.md` for details" | Only when Claude decides to read it |

The main optimization is moving content from inline/@-include to reference where it's safe to do so.

## Installation

See the [main README](../README.md#installation) for installation instructions.
