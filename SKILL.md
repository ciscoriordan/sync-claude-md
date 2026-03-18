---
name: sync-claude-md
description: Sync your global CLAUDE.md across machines via Dropbox or GitHub Gist. Run this on each machine to keep instructions consistent.
---

# Sync Global CLAUDE.md

Set up syncing of `~/.claude/CLAUDE.md` across machines so all machines share the same global instructions.

## Steps

1. **Check for Dropbox.** Look for `~/Dropbox` or `~/Library/CloudStorage/Dropbox` (macOS) or `%USERPROFILE%\Dropbox` (Windows). Also check if the Dropbox desktop app folder exists.

2. **If Dropbox is available:**
   - Create `~/Dropbox/Claude/CLAUDE.md` if it doesn't exist (copy from `~/.claude/CLAUDE.md` if that exists, otherwise create a starter template).
   - **Before replacing the local file**, diff `~/.claude/CLAUDE.md` against `~/Dropbox/Claude/CLAUDE.md`. If the local file has content not present in the shared file, show the diff to the user and merge the unique local content into the shared file. Do not silently discard instructions that only exist on this machine.
   - Replace `~/.claude/CLAUDE.md` with a symlink to `~/Dropbox/Claude/CLAUDE.md`.
   - On Windows use `mklink` instead of `ln -s`.
   - Print: "Syncing via Dropbox. Edit ~/Dropbox/Claude/CLAUDE.md and it syncs automatically."

3. **If Dropbox is NOT available, use GitHub Gist:**
   - Check if `gh` CLI is installed and authenticated (`gh auth status`).
   - **First-time setup (no gist exists yet):**
     - Check if `~/.claude/.claude-md-gist-id` exists. If yes, use that gist ID.
     - Otherwise ask the user if they have an existing gist ID to pull from. If yes, pull it and save the ID.
     - If no existing gist, create one: `gh gist create --filename "CLAUDE.md" --desc "Shared global CLAUDE.md" ~/.claude/CLAUDE.md` and save the gist ID.
   - **Store the gist ID** in `~/.claude/.claude-md-gist-id` for future use.
   - **Before overwriting the local file**, diff `~/.claude/CLAUDE.md` against the gist content. If the local file has content not present in the gist, show the diff to the user and offer to merge the unique local content into the gist before pulling. Do not silently discard instructions that only exist on this machine.
   - **Pull:** `curl -sL https://gist.githubusercontent.com/{user}/{gist_id}/raw/CLAUDE.md -o ~/.claude/CLAUDE.md`
   - **Push:** `gh gist edit {gist_id} -f CLAUDE.md ~/.claude/CLAUDE.md`
   - Ask the user whether they want to **pull** (overwrite local with gist) or **push** (update gist from local). Default to pull on new machines, push if local file is newer or user just edited it.
   - Print the gist URL and instructions for other machines.

4. **Verify** by showing the first 5 lines of the resulting `~/.claude/CLAUDE.md`.

## Starter template

If no CLAUDE.md exists anywhere, create one with these sections:

```markdown
# Global Instructions

## Writing Style
- Prefer clear, concise language.

## Git Commits
- Write descriptive commit messages.

## README Maintenance
- Update README.md when making significant changes to a project.
```

## Important
- Never make the gist public. Always create secret gists.
- On Windows, `~/.claude` is `%USERPROFILE%\.claude`.
- Be concise in output. Don't over-explain.
