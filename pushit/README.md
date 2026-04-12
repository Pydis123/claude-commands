# /pushit

Quality-gate and push your work to GitHub. Commits uncommitted changes, runs your project's checks, and pushes — stopping at the first failure.

## How to use

```
/pushit
```

No arguments needed. Run it in any project with a git remote.

## What it does

The command runs five steps in order, stopping at the first failure:

### Step 1 — Preflight

Checks that a git remote exists, there's no in-progress rebase or merge conflict, and there's actually something to push. Shows your current branch and status.

### Step 2 — Commit uncommitted changes

If there are uncommitted changes, stages and commits them with descriptive messages. Warns (but does not block) if it detects sensitive files like `.env` or hardcoded API keys. If everything is already committed, this step is skipped.

### Step 3 — Pre-push verification

Runs your project's quality checks in two layers:

**Layer A — Project-specific checks:** Follows whatever rules your `CLAUDE.md` defines — tests, linting, typechecking, smoke tests. If your project has no `CLAUDE.md` or no pre-push rules, nothing happens.

**Layer B — Security audit:** If you have the `/security-audit` command installed, runs it in diff mode and blocks on MEDIUM or higher findings. If not installed, shows a one-time tip about where to get it.

### Step 4 — Diff summary

Shows the commits that will be pushed, so you can see exactly what's going out.

### Step 5 — Push

Runs `git push`. If the remote has new commits, explains the situation and suggests `git pull --rebase` — but never runs it automatically.

## Recommended workflow

```
/endsession     ← persist context, update project files
/pushit         ← commit, verify, push
/clear          ← start fresh
```

Or standalone, any time during a session:

```
/pushit
```

## Optional: Security audit

For pre-push security scanning, install the `/security-audit` command from this same repository. When installed, `/pushit` runs it automatically in diff mode before pushing.

## Installation

See the [main README](../README.md#installation) for installation instructions.
