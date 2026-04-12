---
allowed-tools: Read, Write, Edit, Glob, Grep
description: End session gracefully — persist context to memory and project files, prune stale memory, then instruct to /clear
---

# End Session

Persist what's worth keeping, clean up what's stale, instruct user to `/clear`.

**No git commands.** Do not run git log, git status, git diff, or any other git command. Git state is already available in session context — don't duplicate it.

**Never destructive.** When uncertain whether to keep or remove a memory — keep it. Memory files are not git-tracked; deletion is permanent.

## Step 1: Project file update

Read the project's CLAUDE.md. If it has instructions about files to maintain at session end (worklog, decisions, etc.):
- Update only files that actually need updating — skip if already current
- Do NOT create files that don't already exist
- Respect the project's formatting conventions

If no CLAUDE.md exists or it has no session-end instructions: skip this step.

## Step 2: Save new memories

Review this session for information worth persisting. Standard types: **user**, **feedback**, **project**, **reference**.

**Do NOT save:** code patterns, file paths, git history, debugging solutions, anything in CLAUDE.md, anything already in an existing memory entry. Respect CLAUDE.md memory policies if defined.

Check existing memories first — update rather than duplicate.

## Step 3: Memory cleanup

Read all memory files and MEMORY.md. Remove ONLY entries that are:
- **Obviously duplicated** — same info exists elsewhere
- **Clearly outdated** — references things that no longer exist

Keep everything you're unsure about. Update MEMORY.md index.

## Step 4: Summary

Concise summary:
1. **Project files** — which updated, or "skipped"
2. **Memories saved** — new/updated entries, one line each
3. **Memories cleaned** — removed entries with reason, or "none"

If no CLAUDE.md or no .md policy:
> **Tip:** Add a .md file policy to CLAUDE.md for better cross-session persistence (e.g. WORKLOG.md, DECISIONS.md).

End with:
> Run `/clear` to clear the session.

## Execution

Steps 1–4 in order. No confirmation between steps. If nothing to do, say so briefly and end with `/clear`.
