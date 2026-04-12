---
allowed-tools: Read, Grep, Glob, Bash(wc:*), Bash(find:*), Bash(cat:*), Bash(head:*)
description: Audit and optimize your CLAUDE.md to reduce token cost without losing functionality
---

# CLAUDE.md Health Check

You are analyzing this project's CLAUDE.md to find optimization opportunities. Your goal: reduce token cost per message without losing any functional value. Be conservative — when uncertain, keep.

## Principles

1. **Function > cost.** Never suggest removing anything with functional value, regardless of token cost.
2. **Every line must carry its weight.** If a line doesn't prevent mistakes, save tool calls, or trigger correct behavior — question it.
3. **Maximize lazy loading.** Move detail info from inline/@ to "read when needed" references where safe. Keep short summaries as triggers.
4. **Conservative by default.** When uncertain — keep. Better 50 tokens too many than a broken behavior.
5. **@-includes are expensive.** Every @-include is expanded every message. Always flag and suggest alternatives if possible.
6. **Critical rules must be inline.** Things that prevent destructive mistakes must never be moved to external files.

## Token Cost Levels

Understand these three levels — they drive your analysis:

| Level | Mechanism | Token cost |
|-------|-----------|------------|
| **1. Inline** | Text directly in CLAUDE.md | Every message |
| **2. @-include** | `@FILE.md` in CLAUDE.md | Every message (system-expanded, unavoidable) |
| **3. Reference** | "See `FILE.md` for details" | Only when actually read via Read tool |

Core optimization: move information from level 1/2 → level 3 where safe. Keep a short inline summary as a trigger so you know WHEN to read the external file.

## Execution Steps

Follow these steps in order. Do NOT modify any files until step 10.

### Step 1: Find CLAUDE.md

Look for `CLAUDE.md` in the project root (current working directory).

If it does not exist, tell the user: "No CLAUDE.md found in this project. Nothing to analyze." and stop.

Read the entire file.

### Step 2: Map the context chain

Identify ALL files that get loaded into context every message:

1. **Project CLAUDE.md** — the file you just read
2. **@-includes** — scan for lines starting with `@` followed by a filename. Read each included file and note its size. If those files also have @-includes, follow them recursively.
3. **Parent directory CLAUDE.md files** — walk up from the project root checking for CLAUDE.md in each parent directory until you reach `/` or the user's home directory. Read any you find.
4. **Global CLAUDE.md** — check `~/.claude/CLAUDE.md`. Read it if it exists.

For each file found, note: path, word count, estimated tokens (words × 1.3).

### Step 3: Token estimation

Calculate:
- Token cost per file (words × 1.3)
- Total tokens loaded per message (sum of all files from step 2)
- The project's own CLAUDE.md as percentage of total

Use `wc -w` for accurate word counts.

### Step 4: Reference validation

Scan the project's CLAUDE.md for all file references. This includes:
- Backtick-quoted paths like `path/to/file.md`
- References in markdown tables
- Paths after "see", "read", "check", "refer to"
- @-includes

For each reference, verify the file exists on disk using Glob or Bash. Record which references are dead.

### Step 5: Duplicate analysis

For each referenced file that EXISTS and is mentioned with substantive inline content in CLAUDE.md:
- Read the referenced file
- Compare semantically: does CLAUDE.md repeat the same information that's already in the external file?
- A short summary that acts as a trigger ("X handles Y — see file for details") is NOT a duplicate
- A full paragraph restating what's in the external file IS a duplicate

### Step 6: Lazy-load analysis

For each section in CLAUDE.md, ask: "Does this need to be loaded every single message?"

Candidates for lazy loading (move to reference):
- Detailed environment setup instructions (keep only critical gotchas inline)
- Long navigation/documentation tables (consider: does this save tool calls? if yes, keep)
- Detailed process descriptions that only apply to specific tasks
- Historical context or migration details

Must stay inline:
- Critical safety rules (destructive commands, security gates)
- Session-start behavior instructions
- Core coding conventions that apply to every task
- Short trigger summaries for external files

### Step 7: @-include review

For each @-include found in step 2:
- What does it pull in?
- How many tokens does it cost per message?
- Is the included content needed for EVERY task, or only some?
- Could it be replaced with a plain text reference ("See `file.md` when working on X")?
- Is it from a parent directory and potentially not even relevant to this project?

### Step 8: Verbose analysis

Scan CLAUDE.md for:
- Repeated information (same rule stated in different ways)
- Overly verbose explanations where a concise version would work equally well
- Bullet points that could be combined
- Examples or elaborations that don't add value beyond the rule itself

Important: condensing must NOT lose information. If a verbose section exists because the nuance matters, keep it.

### Step 9: Obsolescence check

For sections that reference specific files, directories, tools, or behaviors:
- Verify the referenced items still exist
- Check if described behaviors match the current code state
- Flag sections that appear to describe a past state of the project

This overlaps with step 4 (dead references) but goes deeper — a file might exist but the description of it might be outdated.

### Step 10: Present the report

Format the report as follows:

---

## CLAUDE.md Health Check Report

### Part 1: Token Map

| File | Words | Est. Tokens | Loaded |
|------|-------|-------------|--------|
| (each file from step 2) | (count) | (count) | Every message |

**Total per message: ~X tokens**
**Project CLAUDE.md: ~Y tokens (Z% of total)**

(For files outside the project, add a note: "ℹ️ Files outside this project are shown for awareness but no changes will be suggested for them.")

### Part 2: Findings

Group findings by category. For each finding, show:

**[SEVERITY] Category: Brief description**
Location: line numbers or section name
Details: what the issue is
Principle: which assessment principle applies

Severity levels:
- **WARNING** — Dead references, obsolete sections (factually wrong)
- **SUGGESTION** — Duplicates, verbose text, lazy-load opportunities, @-include alternatives

### Part 3: Suggested Actions

List concrete proposed changes, grouped by type:
1. **Remove** — dead references, obsolete sections
2. **Condense** — verbose text (show before/after)
3. **Extract** — move to external file with inline trigger summary
4. **Replace @-include** — swap @-include for plain reference

For each action, show what would change. For condensations, show the proposed shorter text.

End with: **"Would you like me to implement these changes? I can do all, or you can pick specific ones."**

---

### Step 11: Implement (only after approval)

If the user approves (all or specific items):
- Make the approved changes to CLAUDE.md
- Show a summary of what changed
- Report the new token count vs. the old one

If the user declines, stop. Do not persist or ask again.

## Important Constraints

- NEVER modify files outside the project
- NEVER remove critical safety rules, regardless of token cost
- NEVER create new files — only modify existing CLAUDE.md
- NEVER implement changes without explicit user approval
- When a section's value is ambiguous, classify it as "keep" and note why in the report

## Your Job

Follow the execution steps above in order. Do not skip or reorder them.
