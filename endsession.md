---
allowed-tools: Read, Write, Edit, Glob, Grep
description: End session gracefully — persist context to memory and project files, prune stale memory, then instruct to /clear
---

# End Session

You are wrapping up this Claude Code session. Your job: persist everything worth keeping, clean up what's stale, and prepare the user to clear the session.

**Core principle: never destructive.** When uncertain whether to keep or remove something — keep it. Memory files are not git-tracked; deletion is permanent.

## Step 1: Project file update (adaptive)

Read the project's CLAUDE.md (if it exists). Scan for instructions about:
- Files that should be maintained at session end or regularly (e.g. worklog, decisions, roadmap)
- .md file policies or documentation strategies
- Any session-end or session-close routines

**If such instructions exist:**
- Follow them, but only update files that actually need updating (already up to date = skip)
- Do NOT create files that don't already exist — if CLAUDE.md references a file that hasn't been created yet, skip it
- Respect the project's conventions for formatting and content

**If no CLAUDE.md exists, or it has no such instructions:** skip this step entirely.

## Step 2: Save new information to memory

Review this session's conversation for information worth persisting across sessions. Save to auto memory using the standard types:

- **user** — role, preferences, expertise, how to tailor responses
- **feedback** — corrections AND confirmations of approach (both matter)
- **project** — ongoing work context not derivable from code/git (convert relative dates to absolute)
- **reference** — pointers to external systems, dashboards, docs

**What NOT to save:**
- Code patterns, architecture, file paths (derivable from the codebase)
- Git history or recent changes (use `git log`)
- Debugging solutions (the fix is in the code)
- Anything already documented in CLAUDE.md
- Anything already covered by an existing memory entry

If CLAUDE.md defines a memory policy (e.g. "project knowledge belongs in WORKLOG.md, not memory"), respect it.

Check existing memories before writing new ones — update rather than duplicate.

## Step 3: Memory cleanup (conservative)

Read all memory files in the project's memory directory and the MEMORY.md index.

Remove ONLY entries that are:
- **Obviously duplicated** — the exact same information exists in another memory file or was just written to a project file in step 1
- **Clearly outdated** — references tools, files, decisions, or people that demonstrably no longer exist or apply

**Keep everything you're unsure about.** One extra memory file costs nearly nothing. A lost insight costs a future session.

Update MEMORY.md index to reflect any changes.

## Step 4: Summary

Report what you did in a concise summary:

1. **Project files** — which files were updated (if any), or "No project file policy found in CLAUDE.md — skipped"
2. **Memories saved** — list new or updated memory entries with one-line descriptions
3. **Memories cleaned** — list removed entries with brief reason, or "No stale memories found"
4. **Uncommitted changes** — if steps 1-3 created file changes, note: "Note: File changes from this session wrap-up are uncommitted."

If the project has no CLAUDE.md or its CLAUDE.md has no .md file policy, add:

> **Tip:** You can improve cross-session persistence by adding a .md file policy to your CLAUDE.md (e.g. a WORKLOG.md for work status or DECISIONS.md for architecture decisions).

End with:

> Run `/clear` to clear the session.

## Execution

Follow steps 1–4 in order. Do not skip or reorder them. Do not ask for confirmation between steps — execute the full flow, then present the summary.

If there is nothing to do (no project file policy, no new memories to save, no stale memories to clean), say so briefly and end with the `/clear` instruction.
