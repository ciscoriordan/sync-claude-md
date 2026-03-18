# sync-claude-md

A [Claude Code skill](https://docs.anthropic.com/en/docs/claude-code/skills) that syncs your global `~/.claude/CLAUDE.md` across machines, so your Claude Code instructions stay consistent everywhere.

## Installation

```bash
npx skills add ciscoriordan/sync-claude-md
```

## Usage

In any Claude Code conversation, run:

```
/sync-claude-md
```

## What it does

The skill detects available sync methods and sets up the best option automatically:

### Dropbox (preferred)

If Dropbox is installed, the skill:

1. Copies `~/.claude/CLAUDE.md` to `~/Dropbox/Claude/CLAUDE.md`
2. Replaces the local file with a symlink to the Dropbox copy
3. Changes sync automatically across all linked machines

### GitHub Gist (fallback)

If Dropbox isn't available but the `gh` CLI is installed and authenticated, the skill:

1. Creates a secret gist containing your CLAUDE.md (or connects to an existing one)
2. Stores the gist ID in `~/.claude/.claude-md-gist-id`
3. Pulls from the gist to sync, pushing first if local has changes

Run `/sync-claude-md` whenever you want to sync. On subsequent machines, provide the same gist ID to pull your shared instructions.

### Safety

Before overwriting anything, the skill diffs local and remote versions. If the local file has content not present in the shared copy, it shows the diff and merges rather than silently discarding instructions.

## First-time setup

If no `~/.claude/CLAUDE.md` exists on any machine, the skill creates a starter template with basic sections for writing style, git commits, and README maintenance.

## Platform support

- **macOS/Linux**: Symlink created automatically by the skill. No manual steps.
- **Windows (Dropbox)**: Git bash can't create symlinks. The skill prints a PowerShell command for you to run once in a regular terminal. Requires Developer Mode (Settings -> System -> For developers) or Administrator privileges.
- **Windows (Gist)**: Works fully automatically, no symlinks needed.
- Dropbox paths checked: `~/Dropbox`, `~/Library/CloudStorage/Dropbox` (macOS), `%USERPROFILE%\Dropbox` (Windows)
