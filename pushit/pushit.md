---
allowed-tools: Bash, Read, Edit, Write, Glob, Grep
description: Quality-gate and push to GitHub — commit, verify, push
---

# Push It

Quality-gate and push to GitHub. Run the steps below in order. Stop at the first failure.

All output in English.

## Step 1: Preflight

Run these checks. Stop with a clear message if any fail:

- **Remote exists?** Run `git remote -v`. If no remote is configured → stop: "No remote configured. Add one with `git remote add origin <url>`."
- **Clean git state?** Run `git status`. If in detached HEAD, in-progress rebase, or merge conflict → stop with explanation of what's wrong and how to fix it.
- **Show branch name.** Print: "Branch: <name>"
- **Show status.** Print staged, unstaged, untracked files, and commits ahead of origin.
- **Anything to do?** If no changes AND no commits ahead of origin → print "Nothing to push." → exit.

## Step 2: Commit uncommitted changes

If everything is already committed, skip to Step 3.

If there are staged, unstaged, or untracked changes:

1. **Sensitive file scan.** Before staging, check if any files to be committed look like they contain secrets (.env files, files with API keys, hardcoded passwords, credential files). If found:
   - **Warn, do not block.** Print a warning listing the suspect files/content.
   - Let the user decide whether to include them.
   - Show this warning at most once per session — if the user already acknowledged it, do not warn again.
2. **Stage files.** Respect `.gitignore`. Do not override it.
3. **Create commits.** Write descriptive commit messages following the project's existing commit style. If no style is apparent, use conventional commits (feat/fix/docs prefix). Create multiple commits if the changes are logically separate.

## Step 3: Pre-push verification

Two layers, in order. Stop at the first failure.

### Layer A — Project-specific checks

Follow any pre-push, testing, linting, typechecking, or quality rules defined in the project's CLAUDE.md or other project instructions already loaded in context. Run whatever checks those rules require.

If the project has no CLAUDE.md, or its CLAUDE.md does not mention any pre-push checks → skip this layer.

If any check fails → stop and report what failed.

### Layer B — Security audit

Check if the `/security-audit` command is available (look for `~/.claude/commands/security-audit.md` or a project-level `.claude/commands/security-audit.md`).

**If installed:** Run `/security-audit diff --fail-on medium --lite`. If findings with severity MEDIUM or higher → stop and report.

**If NOT installed:** Show this notice (once per session, then never again):

> **Tip:** You can add pre-push security scanning by installing the `/security-audit` command.
> See: https://github.com/Pydis123/claude-commands

## Step 4: Diff summary

Show what will be pushed:

```
Pushing N commit(s) to origin/<branch>:
<git log origin/<branch>..HEAD --oneline>
```

## Step 5: Push

Run `git push`.

- **If rejected (non-fast-forward):** Explain that the remote branch has new commits that are not in the local branch. Suggest running `git pull --rebase` to integrate them, but do NOT run it automatically. Stop and let the user decide.
- **If successful:** Print the result. Continue to Step 6.

## Step 6: Post-push cleanup hooks

Only run these steps if **all three** are true:
- Push succeeded in Step 5
- The branch pushed is a `feature/`-branch (not `main`/`master`)
- Project has a `.claude/worktrees/` directory (indicates dev-workflow-system is active)

If any condition is false → done, exit.

### 6a — Mark worktree for cleanup

Extract `<topic>` by stripping the `feature/` prefix from the current branch name.
Create a flag file in the project root:

```bash
touch ".claude/worktrees/<topic>.cleanup-pending"
```

This signals to a Claude session running in the project's main directory that the
worktree at `.claude/worktrees/<topic>/` is ready to be removed. Cleanup happens
proactively at the next user message in that session — not here.

### 6b — Close current Warp tab

Detect OS:

- **macOS** (`uname -s` returns `Darwin`):
  ```bash
  osascript -e 'tell application "System Events" to keystroke "w" using command down'
  ```
- **Windows** (`uname -s` returns `MINGW*`, `MSYS*`, or `CYGWIN*`):
  Print: `Done! Press Ctrl+W to close this tab.`
- **Linux** (`uname -s` returns `Linux`):
  Print: `Done! Press Ctrl+Shift+W to close this tab.`

On Mac the tab actually closes automatically. Other OSes get an instruction since
reliable cross-platform automation isn't available in Warp on Windows/Linux yet.
