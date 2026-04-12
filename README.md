# claude-commands

A collection of global commands for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) that work across all your projects.

## Available Commands

| Command | Description |
|---------|-------------|
| [`/cleanclaudemd`](#cleanclaudemd) | Audit and optimize your CLAUDE.md to reduce token cost without losing functionality |
| [`/endsession`](#endsession) | End session gracefully — persist context, prune memory, prepare for /clear |

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed and working
- A project with a `CLAUDE.md` file

## Installation

**Option A: Clone and symlink (recommended)**

This lets you pull updates with `git pull`.

macOS / Linux:
```bash
git clone https://github.com/Pydis123/claude-commands.git ~/claude-commands

# Symlink each command you want
ln -s ~/claude-commands/cleanclaudemd.md ~/.claude/commands/cleanclaudemd.md
```

Windows (PowerShell as Administrator):
```powershell
git clone https://github.com/Pydis123/claude-commands.git $HOME\claude-commands

# Symlink each command you want
New-Item -ItemType SymbolicLink `
  -Path "$HOME\.claude\commands\cleanclaudemd.md" `
  -Target "$HOME\claude-commands\cleanclaudemd.md"
```

> **Note:** On Windows, creating symlinks requires either Administrator privileges or [Developer Mode](https://learn.microsoft.com/en-us/windows/apps/get-started/enable-your-device-for-development) enabled.

**Option B: Copy individual files**

macOS / Linux:
```bash
curl -o ~/.claude/commands/cleanclaudemd.md \
  https://raw.githubusercontent.com/Pydis123/claude-commands/main/cleanclaudemd.md
```

Windows (PowerShell):
```powershell
Invoke-WebRequest `
  -Uri "https://raw.githubusercontent.com/Pydis123/claude-commands/main/cleanclaudemd.md" `
  -OutFile "$HOME\.claude\commands\cleanclaudemd.md"
```

> **Note:** With Option B you get a copy, not a link. To update, re-run the command.

After installation, the command will appear when you type `/clean` in Claude Code.

## Commands

### /cleanclaudemd

Audits your project's `CLAUDE.md` file and produces a report with concrete optimization suggestions. The goal is to reduce the token cost of your CLAUDE.md without losing any functional value.

**CLAUDE.md files are loaded into every single message** in a Claude Code session. A 2,000-word CLAUDE.md costs roughly 2,600 tokens per message. Over a 100-message session, that's 260,000 cache-read tokens just from instructions.

This command helps you find what can be trimmed, condensed, or moved to external files that are only read when needed.

#### How to use

```
/cleanclaudemd
```

No arguments. Run it in any project that has a CLAUDE.md.

#### What the report contains

**Part 1 — Token Map**

Lists every file loaded into your context on each message: your project's CLAUDE.md, any `@`-includes, parent directory CLAUDE.md files, and the global `~/.claude/CLAUDE.md`. Shows word count and estimated token cost for each.

**Part 2 — Findings**

Issues found, categorized by severity:

- **WARNING** — Dead file references, obsolete sections that no longer match the codebase
- **SUGGESTION** — Duplicated content, verbose text, `@`-includes that could be plain references, sections that could be lazy-loaded

**Part 3 — Suggested Actions**

Concrete proposed changes grouped by type: remove, condense, extract to external file, or replace `@`-include with a plain reference. Each suggestion shows exactly what would change.

The command **never modifies your files without explicit approval**. You review the report first, then choose which changes (if any) to apply.

#### How it thinks

The command follows six principles when analyzing your CLAUDE.md:

1. **Function over cost.** Never suggest removing anything with functional value.
2. **Every line must carry its weight.** If a line doesn't prevent mistakes, save tool calls, or trigger correct behavior — question it.
3. **Maximize lazy loading.** Move detail from inline to "read when needed" references where safe.
4. **Conservative by default.** When uncertain, keep. Better 50 extra tokens than a broken behavior.
5. **@-includes are expensive.** They expand on every message. Flag and suggest alternatives.
6. **Critical rules stay inline.** Safety rules that prevent destructive mistakes must never be moved to external files.

#### Token cost levels

The core insight behind this command:

| Level | Mechanism | Token cost |
|-------|-----------|------------|
| Inline | Text in CLAUDE.md | Every message |
| @-include | `@FILE.md` in CLAUDE.md | Every message (expanded by the system) |
| Reference | "See `FILE.md` for details" | Only when Claude actually reads it |

The main optimization is moving information from level 1/2 to level 3 where it's safe to do so, keeping a short inline summary as a trigger.

### /endsession

Automates session cleanup at the end of a Claude Code session. It persists what's worth keeping, prunes what's stale, and tells you when it's safe to run `/clear`.

The command never destructively removes anything you haven't explicitly reviewed. When uncertain about a memory entry — keep it. Memory files are not git-tracked; deletion is permanent.

#### How to use

```
/endsession
```

No arguments. Run it at the end of any session, in any project.

#### The 4-step flow

**Step 1 — Adaptive project file update**

Reads the project's `CLAUDE.md` (if it exists) and scans for session-end instructions: files to update on close (such as a worklog, decisions log, or roadmap), `.md` file policies, or documentation routines. If such instructions exist, it follows them — but only updates files that actually need updating and never creates files that don't already exist. Projects without a `CLAUDE.md`, or whose `CLAUDE.md` has no session-end policy, skip this step entirely.

**Step 2 — Save new information to memory**

Reviews the session's conversation for information worth persisting across sessions. Saves to auto memory in the standard categories: user preferences, feedback and corrections, ongoing project context (with relative dates converted to absolute), and pointers to external systems. Updates existing memory entries rather than duplicating them. Respects project-level memory policies (e.g. if `CLAUDE.md` says "project knowledge belongs in WORKLOG.md, not memory").

**Step 3 — Conservative memory cleanup**

Reads all memory files and removes only entries that are obviously duplicated (the same information was just written to a project file) or clearly outdated (references tools, files, or people that demonstrably no longer exist). Everything uncertain is kept. The MEMORY.md index is updated to reflect any changes.

**Step 4 — Summary**

Reports what happened: which project files were updated, what was saved or updated in memory, what was cleaned and why, and whether there are uncommitted file changes from the session wrap-up. Ends with the instruction to run `/clear`.

#### Design principles

- **Never destructive.** When uncertain, keep. Memory deletion is permanent.
- **Adaptive to CLAUDE.md.** The command reads and respects your project's conventions — no hardcoded assumptions about file names or structure.
- **Portable.** Works in any project, with or without a CLAUDE.md. Projects without a memory or documentation policy get a tip on how to add one.
- **No confirmation prompts.** Executes all four steps, then presents the summary. You review the result, not each individual action.

#### Why `/clear` is manual

`/endsession` cannot run `/clear` automatically — Claude Code does not allow slash commands to trigger other slash commands. After reviewing the summary, you run `/clear` yourself. This is intentional: the summary gives you a moment to confirm everything looks right before clearing the session.

#### Typical workflow

```
/pusha          # push your commits (optional)
/endsession     # persist context, clean memory
/clear          # start fresh
```

Or simply:

```
/endsession
/clear
```

#### Tip: .md file policy

If your project has no `CLAUDE.md`, or its `CLAUDE.md` has no session-end instructions, `/endsession` will skip step 1 entirely and remind you with:

> **Tip:** You can improve cross-session persistence by adding a .md file policy to your CLAUDE.md (e.g. a WORKLOG.md for work status or DECISIONS.md for architecture decisions).

## Uninstalling

Remove the symlink (or copied file):

```bash
rm ~/.claude/commands/cleanclaudemd.md
```

If you cloned the repo, remove it too:

```bash
rm -rf ~/claude-commands
```

## Disclaimer

This software is provided "as is", without warranty of any kind. Use it at your own risk and discretion. The authors are not responsible for any changes made to your files, any token costs incurred, or any other consequences of using these commands. Always review suggested changes before accepting them.

These commands instruct Claude Code to analyze and potentially modify your project files. You are solely responsible for reviewing and approving any changes before they are applied.

## Contributing

Found a bug or have an idea for a new command? Open an issue or submit a pull request.

## License

[MIT](LICENSE)
