---
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, Skill, AskUserQuestion
description: End session — project-aware: optional security audit, WORKLOG, GSD-batch, memory persist, git check
---

# End Session (project-aware)

Persist what's worth keeping, run optional safety checks based on what the project provides, then instruct user to `/clear`.

**Project-aware steps activate based on what the project provides** — falls back to memory-only flow for projects without these conventions:

| Step | Condition |
|---|---|
| 1. Security audit | `security-audit` skill is available globally |
| 2. WORKLOG update | `WORKLOG.md` exists in project root |
| 3. GSD task batch | `tools/gsd.php` exists in project root, and there are spillover candidates |
| 4. Memory persist | Always (core behavior) |
| 5. Git status check | Always (universal safety net against tab-close-loss) |
| 6. /clear instruction | Always |

**Never destructive.** When uncertain about memory cleanup: keep. Memory files are not git-tracked; deletion is permanent.

**Fail-fast:** Step 1 stops on MEDIUM+ findings. Step 5 stops if user chooses "abort". Other steps run in sequence without confirmation.

---

## Step 1: Security audit (gate, conditional)

**Run only if** the `security-audit` skill is available globally. Check by inspecting the available-skills list in this session — if `security-audit` appears, proceed. Otherwise skip silently.

Invoke the `security-audit` skill via `Skill` tool with args `diff --fail-on medium`.

After audit completes, parse the last output line matching `SECURITY_AUDIT_EXIT: PASS|FAIL`:

- **FAIL** → STOP endsession immediately. Show:
  > **Security audit found MEDIUM+ findings.** See `./security-audit-report.md`. Fix these, then run `/endsession` again.

  Do not run Steps 2–6, do not give the `/clear` instruction.

- **PASS** → continue. If `./security-audit-report.md` contains LOW/INFO findings (grep for `### 🟢 [LOW-` and `### 🔵 [INFO-`), buffer titles for Step 3.

If the project's CLAUDE.md specifies how to batch INFO findings (e.g. one task per audit pass), follow that. Default: one GSD task per LOW finding, one batched task for all INFO findings.

## Step 2: WORKLOG update (conditional)

**Run only if** `WORKLOG.md` exists in project root. Otherwise skip.

Read the project's CLAUDE.md for any session-end conventions (entry format, max entries, language used). Read the top of `WORKLOG.md` to match existing style exactly.

Add a new entry covering: date, who (check `git config user.name`), what was done (1–3 bullets), what remains (these are flagged in Step 3).

If CLAUDE.md specifies a max-entry limit, trim older entries (history lives in git).

**Skip** if the session made no codebase changes (pure read-only Q&A with no output file or commit).

## Step 3: GSD task batch (conditional)

**Run only if** `tools/gsd.php` exists in project root. Otherwise skip.

Collect candidates from two sources:

1. **Audit LOW/INFO** buffered from Step 1 (if any)
2. **WORKLOG spillover**: grep the new WORKLOG entry for project-language patterns indicating remaining/future work. Default patterns: `kvarstår`, `återstår`, `future`, `TODO`, `remains`, `pending`. Check CLAUDE.md for project-specific patterns.

**No candidates** → skip.

**≥1 candidates** → AskUserQuestion (multiSelect). Show each with proposed title, story slug, priority. Defaults:

| Source | Default story | Default priority |
|---|---|---|
| Audit LOW | `sakerhet` (or project equivalent) | `low` |
| Audit INFO (batched) | `sakerhet` | `low` |
| WORKLOG spillover | `tech-debt` | `normal` |

Read CLAUDE.md for project-specific story slugs if conventions differ.

For each selected candidate:
```
php tools/gsd.php create --title="..." --story=... --type=task --priority=... --description="..."
```

Description = original WORKLOG line or audit finding title.

Report created GSD IDs in the Step 6 summary.

## Step 4: Memory persist (always)

Review this session for information worth persisting. Standard types: **user**, **feedback**, **project**, **reference**.

**Do NOT save:** code patterns, file paths, git history, debugging solutions, anything in CLAUDE.md, anything already in an existing memory entry. Respect CLAUDE.md memory policies if defined.

Check existing memories first — update rather than duplicate.

**Cleanup:** Read all memory files and `MEMORY.md`. Remove ONLY entries that are:
- **Obviously duplicated** — same info exists elsewhere
- **Clearly outdated** — references things that no longer exist

Keep everything you're unsure about. Update `MEMORY.md` index if entries added or removed.

## Step 5: Git status check (gate, always)

Run `git status --porcelain`. Empty → continue directly to Step 6.

Dirty → show `git status --short` + AskUserQuestion (single-select):

- **Commit now** → AskUserQuestion for commit message (default suggestion: `chore: endsession YYYY-MM-DD`). Run:
  ```
  git add -A && git commit -m "<message>"
  ```
  Continue to Step 6 with commit SHA.

- **Stash** → run:
  ```
  git stash push -m "endsession-stash $(date +%Y-%m-%d-%H%M)" --include-untracked
  ```
  Continue to Step 6 with stash name.

- **Abort endsession** → STOP without `/clear` instruction. User handles manually.

**Why this is the last step:** WORKLOG additions from Step 2 + any pre-existing dirty changes from the session should both be caught here. Tab-close (especially Warp on Windows after `/clear`) loses uncommitted changes silently.

## Step 6: Summary + /clear

One line per step that ran:

1. **Audit** — PASS (X LOW/INFO buffered) / FAIL (already stopped) / skipped
2. **WORKLOG** — new entry added / skipped
3. **GSD tasks** — IDs created or "none"
4. **Memory** — N saved / M cleaned / "none"
5. **Git** — clean / committed (SHA) / stashed (stash@{N})

If in a feature-branch worktree (cwd under `.claude/worktrees/`) and the feature feels complete, write the handoff block (per the project's CLAUDE.md if it specifies one).

End with:
> Run `/clear` to clear the session.

---

## Execution

Steps 1–6 in order. No confirmation between steps except for explicit interactive prompts (Step 3 multiSelect, Step 5 dirty-tree prompt).

If Step 1 FAILs or Step 5 user chooses "abort": stop without `/clear` instruction.

**For projects with no conventions detected** (no `WORKLOG.md`, no `tools/gsd.php`, no `security-audit` skill): the flow degenerates to memory persist + git-status-check + /clear. This is a strict superset of the original endsession behavior.
