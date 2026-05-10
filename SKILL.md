---
name: sync-claude-md
description: Sync your global CLAUDE.md across machines via a private GitHub repo, Dropbox, or GitHub Gist. Run this on each machine to keep instructions consistent.
---

# Sync Global CLAUDE.md

Sync `~/.claude/CLAUDE.md` across machines so all machines share the same global instructions. The user runs `/sync-claude-md` on each machine to sync.

## Step 1: detect existing setup

Before doing anything, check whether `~/.claude/CLAUDE.md` is already a symlink. If it is, resolve the target — that tells you which sync mode is already in use, and you should keep using it. **Do not silently switch modes**, or you will orphan the user's existing sync.

```bash
target=$(readlink ~/.claude/CLAUDE.md 2>/dev/null)
```

- If `$target` is empty → no symlink yet, this is a fresh setup → go to Step 4 (first-time setup).
- If `$target` resolves into a directory that's inside a git repo (`git -C "$(dirname "$target")" rev-parse --show-toplevel` succeeds) → **private-repo mode**, go to Step 2.
- Otherwise (Dropbox path, iCloud path, etc.) → **Dropbox / cloud-folder mode**, go to Step 3.

A `.claude-md-gist-id` file at `~/.claude/.claude-md-gist-id` indicates **Gist mode** (no symlink — the file is overwritten in place). Treat its presence as the trigger to use Step 3b.

## Step 2: private GitHub repo sync

This is the path for users whose `~/.claude/CLAUDE.md` is symlinked into a git repo (e.g. `~/Documents/claude-md/CLAUDE.md`).

```bash
repo_dir=$(dirname "$(readlink -f ~/.claude/CLAUDE.md)")
cd "$repo_dir"
git pull --rebase --autostash
# If there are local edits, stage + commit + push.
if ! git diff --quiet HEAD -- CLAUDE.md; then
  git add CLAUDE.md
  git commit -m "Update CLAUDE.md"
  git push
fi
```

Tell the user the repo URL (`git remote get-url origin`) so they can clone it on other machines. Setup on a second machine is two commands:

```bash
git clone <url> <local-path>
ln -s <local-path>/CLAUDE.md ~/.claude/CLAUDE.md
```

## Step 3: Dropbox / cloud-folder sync

Existing Dropbox setup, or the user explicitly chose Dropbox during first-time setup.

- **Before replacing the local file**, diff `~/.claude/CLAUDE.md` against `~/Dropbox/Claude/CLAUDE.md`. If the local file has content not present in the shared file, show the diff to the user and merge the unique local content into the shared file. Do not silently discard instructions that only exist on this machine.
- Replace `~/.claude/CLAUDE.md` with a symlink to `~/Dropbox/Claude/CLAUDE.md` if not already a symlink.
  - macOS/Linux: `rm ~/.claude/CLAUDE.md && ln -s ~/Dropbox/Claude/CLAUDE.md ~/.claude/CLAUDE.md`
  - Windows: print the symlink command (see Windows notes below).
- Print: "Syncing via Dropbox. Edit ~/Dropbox/Claude/CLAUDE.md and it syncs automatically."

### Step 3b: Gist sync

When `~/.claude/.claude-md-gist-id` exists, the user is on Gist mode.

- Read the gist ID from `~/.claude/.claude-md-gist-id`.
- **Before overwriting**, diff local against the gist. If local has content not in the gist, show the diff and push first.
- Sync:
  - Pull: `gh gist view <id> -f CLAUDE.md > ~/.claude/CLAUDE.md` (or `curl -sL https://gist.githubusercontent.com/{user}/{id}/raw/CLAUDE.md -o ~/.claude/CLAUDE.md`)
  - Push: `gh gist edit <id> -f CLAUDE.md ~/.claude/CLAUDE.md`

## Step 4: first-time setup

Use AskUserQuestion to ask which sync method the user wants. Recommend private GitHub repo when `gh` is authenticated and the user wants version history; Dropbox when they value zero-effort sync; Gist when they don't have Dropbox and don't want a full repo.

| Option | Pros | Cons |
|--------|------|------|
| **Private GitHub repo** (recommended when `gh` auth is set) | Version history, diffable, ssh-friendly to non-Dropbox machines like a Linux GPU box | Manual `git pull` / `git push` step (the skill handles it on subsequent runs) |
| **Dropbox** | Auto-syncs in the background, zero ongoing effort | Requires Dropbox installed on every target machine; no version history |
| **GitHub Gist** | Works on any machine with `gh`; no symlink | No version history beyond gist's own; gist URLs are long |

### 4a. Private GitHub repo setup

1. Ask the user: repo name (default `claude-md`) and clone path (default `~/Documents/claude-md`).
2. `gh repo create <name> --private --description "Synced global ~/.claude/CLAUDE.md across machines"` — do NOT pass `--source` because that requires an existing repo on disk.
3. Init clone:
   ```bash
   mkdir -p <clone-path> && cd <clone-path>
   git init -b main
   cp ~/.claude/CLAUDE.md ./CLAUDE.md   # if file exists; otherwise write the starter template (below)
   git add CLAUDE.md
   git commit -m "Initial sync of global CLAUDE.md"
   git remote add origin git@github.com:<user>/<repo>.git    # or https URL — use whatever matches gh auth's protocol
   git push -u origin main
   ```
4. Swap the symlink:
   ```bash
   rm ~/.claude/CLAUDE.md
   ln -s <clone-path>/CLAUDE.md ~/.claude/CLAUDE.md
   ```
5. Print: "Syncing via private GitHub repo: <repo-url>. Edit ~/.claude/CLAUDE.md (it's symlinked into the repo) and re-run /sync-claude-md to push."
6. If the user previously synced via Dropbox, ask whether to delete `~/Dropbox/Claude/CLAUDE.md` (default: keep as backup).

### 4b. Dropbox setup

Same as the existing Dropbox flow described in Step 3.

### 4c. Gist setup

Same as the existing Gist flow described in Step 3b. Save the gist ID to `~/.claude/.claude-md-gist-id`.

## Step 5: verify

Show the first 5 lines of the resulting `~/.claude/CLAUDE.md` and the resolved symlink target (or the gist URL if Gist mode).

## Starter template

If no CLAUDE.md exists anywhere, create one with:

```markdown
# Global Instructions

## Writing Style
- Prefer clear, concise language.

## Git Commits
- Write descriptive commit messages.

## README Maintenance
- Update README.md when making significant changes to a project.
```

## Windows notes

Git bash cannot create symlinks. For Dropbox/private-repo modes that need a symlink, **print a command for the user to run in PowerShell or Command Prompt** (not git bash):

```
del C:\Users\USERNAME\.claude\CLAUDE.md
powershell -Command "cmd /c mklink 'C:\Users\USERNAME\.claude\CLAUDE.md' '<TARGET-PATH>'"
```

Replace USERNAME and TARGET-PATH. This only needs to be done once.

Symlinks require either Administrator privileges or Developer Mode:
Settings -> System -> For developers (or Advanced on some Windows 11 builds).

For Gist mode no symlink is needed; it works on Windows out of the box.

## Important

- Never make the gist public. Always create secret gists (`gh gist create` defaults to secret).
- Never make the GitHub repo public — `--private` is required.
- On Windows, `~/.claude` is `%USERPROFILE%\.claude`.
- Be concise in output. Don't over-explain.
- **Never silently switch sync modes.** Detect what's already in use and stick with it; only ask the user when there's no existing setup.
