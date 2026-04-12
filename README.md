# claude-commands

A collection of global slash commands for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) that work across all your projects.

## Available Commands

| Command | What it does |
|---------|-------------|
| [`/cleanclaudemd`](cleanclaudemd/) | Audit your CLAUDE.md and find where you're wasting tokens |
| [`/endsession`](endsession/) | Save context and clean up memory before ending a session |
| [`/pushit`](pushit/) | Quality-gate and push to GitHub — commit, verify, push |

Click a command name to see its full documentation.

## What are global commands?

Claude Code lets you create [slash commands](https://docs.anthropic.com/en/docs/claude-code/tutorials#create-slash-commands) — custom prompts you trigger by typing `/` followed by a name. Commands placed in `~/.claude/commands/` are **global**: they work in every project, not just one.

This repository provides ready-made global commands you can install with a single symlink.

## Installation

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed and working
- Git (for Option A)

### Option A: Clone and symlink (recommended)

Cloning lets you update all commands later with `git pull`.

**macOS / Linux:**

```bash
# 1. Clone the repository
git clone https://github.com/Pydis123/claude-commands.git ~/claude-commands

# 2. Make sure the commands directory exists
mkdir -p ~/.claude/commands

# 3. Symlink the commands you want
ln -s ~/claude-commands/cleanclaudemd/cleanclaudemd.md ~/.claude/commands/cleanclaudemd.md
ln -s ~/claude-commands/endsession/endsession.md ~/.claude/commands/endsession.md
ln -s ~/claude-commands/pushit/pushit.md ~/.claude/commands/pushit.md
```

**Windows (PowerShell as Administrator):**

```powershell
# 1. Clone the repository
git clone https://github.com/Pydis123/claude-commands.git $HOME\claude-commands

# 2. Make sure the commands directory exists
New-Item -ItemType Directory -Force -Path "$HOME\.claude\commands"

# 3. Symlink the commands you want
New-Item -ItemType SymbolicLink `
  -Path "$HOME\.claude\commands\cleanclaudemd.md" `
  -Target "$HOME\claude-commands\cleanclaudemd\cleanclaudemd.md"

New-Item -ItemType SymbolicLink `
  -Path "$HOME\.claude\commands\endsession.md" `
  -Target "$HOME\claude-commands\endsession\endsession.md"

New-Item -ItemType SymbolicLink `
  -Path "$HOME\.claude\commands\pushit.md" `
  -Target "$HOME\claude-commands\pushit\pushit.md"
```

> **Note:** On Windows, creating symlinks requires either Administrator privileges or [Developer Mode](https://learn.microsoft.com/en-us/windows/apps/get-started/enable-your-device-for-development) enabled.

You only need to symlink the commands you want — you don't have to install all of them.

### Option B: Download individual files

If you don't want to clone the whole repository, you can download just the commands you need.

**macOS / Linux:**

```bash
mkdir -p ~/.claude/commands

# Download a single command (replace <command> with the name)
curl -o ~/.claude/commands/cleanclaudemd.md \
  https://raw.githubusercontent.com/Pydis123/claude-commands/main/cleanclaudemd/cleanclaudemd.md
```

**Windows (PowerShell):**

```powershell
New-Item -ItemType Directory -Force -Path "$HOME\.claude\commands"

Invoke-WebRequest `
  -Uri "https://raw.githubusercontent.com/Pydis123/claude-commands/main/cleanclaudemd/cleanclaudemd.md" `
  -OutFile "$HOME\.claude\commands\cleanclaudemd.md"
```

> With this option you get a copy, not a link. To update, re-download the file.

### Verify installation

After installing, open Claude Code in any project and type `/`. Your installed commands should appear in the autocomplete list.

## Updating

If you used Option A (clone + symlink):

```bash
cd ~/claude-commands
git pull
```

That's it — your symlinks pick up the changes automatically.

## Uninstalling

Remove the symlinks for any commands you want to uninstall:

**macOS / Linux:**
```bash
rm ~/.claude/commands/cleanclaudemd.md
rm ~/.claude/commands/endsession.md
rm ~/.claude/commands/pushit.md
```

**Windows (PowerShell):**
```powershell
Remove-Item "$HOME\.claude\commands\cleanclaudemd.md"
Remove-Item "$HOME\.claude\commands\endsession.md"
Remove-Item "$HOME\.claude\commands\pushit.md"
```

If you cloned the repository and want to remove it entirely:

```bash
rm -rf ~/claude-commands
```

## Contributing

Found a bug or have an idea for a new command? [Open an issue](https://github.com/Pydis123/claude-commands/issues) or submit a pull request.

## Disclaimer

This software is provided "as is", without warranty of any kind. These commands instruct Claude Code to analyze and potentially modify your project files. You are responsible for reviewing and approving any changes before they are applied.

## License

[MIT](LICENSE)
