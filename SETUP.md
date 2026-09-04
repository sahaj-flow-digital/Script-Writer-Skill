# Setup — Script Writer Skill

This repo holds two Claude Code skills as sibling folders. Claude Code only auto-discovers a skill when its `SKILL.md` sits one level under `~/.claude/skills/`, so the repo is cloned to its own location and each skill folder is **symlinked** into `~/.claude/skills/`. One `git pull` updates both skills; every Claude Code session pulls automatically on startup.

## Repo layout

```
Script-Writer-Skill/
  flow-video-scripts/            long-form "hero" video skill
    SKILL.md
    presenters/
      jacob.md                   Jacob's voice + output format
      sahaj.md                   Sahaj's voice + output format
  flow-shortform-video-scripts/  60-90s LinkedIn/Instagram clip skill
    SKILL.md
  README.md
  SETUP.md                       this file
```

Add a new presenter by dropping a new file in `flow-video-scripts/presenters/` (copy an existing one as a template) and referencing it from that skill's "Presenter adaptation" section. Add a whole new skill by creating a new top-level folder with its own `SKILL.md`, then symlinking it in step 3 below.

## 1. Generate a dedicated SSH key for this repo

Don't reuse your personal GitHub SSH key — a key just for this repo keeps your commits attributed correctly and means revoking access here never touches your other repos.

```bash
ssh-keygen -t ed25519 -C "script-writer-skill" -f ~/.ssh/id_ed25519_scriptwriter -N ""
cat ~/.ssh/id_ed25519_scriptwriter.pub
```

Copy the printed key and add it at **github.com/settings/ssh/new** on whichever GitHub account has push access to this repo. Give it a recognizable title (e.g. "script-writer-skill — <your name>").

## 2. Point SSH at that key for this repo only

Add this to `~/.ssh/config` (create the file if it doesn't exist):

```
Host github-scriptwriter
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_scriptwriter
  IdentitiesOnly yes
```

Test it:

```bash
ssh -T github-scriptwriter
```

You should see "Hi <your GitHub username>! You've successfully authenticated..."

## 3. Clone the repo and symlink each skill into place

Clone it somewhere outside `~/.claude/skills` — that folder is just for what Claude Code scans, the actual repo lives in `~/repos`:

```bash
mkdir -p ~/repos
git clone git@github-scriptwriter:sahaj-flow-digital/Script-Writer-Skill.git ~/repos/Script-Writer-Skill
mkdir -p ~/.claude/skills
ln -s ~/repos/Script-Writer-Skill/flow-video-scripts ~/.claude/skills/flow-video-scripts
ln -s ~/repos/Script-Writer-Skill/flow-shortform-video-scripts ~/.claude/skills/flow-shortform-video-scripts
```

Set your commit identity for this repo (repo-local, doesn't touch your global git config):

```bash
cd ~/repos/Script-Writer-Skill
git config user.email "you@flow.digital"
git config user.name "Your Name"
```

Restart Claude Code (or open a new session) and both skills should appear in your available skills.

## 4. Auto-update on every session start

Add a `SessionStart` hook so Claude Code pulls the latest commit every time you open it. Edit (or create) `~/.claude/settings.json`:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "git -C ~/repos/Script-Writer-Skill pull --ff-only >/dev/null 2>&1 || true"
          }
        ]
      }
    ]
  }
}
```

If you already have other `SessionStart` hooks configured, add this command object into the existing `hooks` array rather than replacing the file — don't overwrite other tools' hooks.

The `|| true` means a failed pull (no network, a conflict, etc.) never blocks you from starting a session. If you have uncommitted local edits that would be overwritten, `--ff-only` refuses to pull rather than clobber them — worst case you're working off a stale copy until you commit or stash.

## 5. How updates flow between people

- **Nothing auto-commits or auto-pushes.** When Claude edits a `SKILL.md` or a `presenters/*.md` file during a session (e.g. adjusting someone's voice based on feedback), it will show you a diff, summarize the change in plain language, and ask before committing/pushing — see "Keeping this skill current" at the bottom of each `SKILL.md`.
- **Uncommitted work is never lost on session end.** Edits sit on disk regardless of git state; a commit is just a snapshot. The only thing that can go stale is your local copy relative to the repo, and step 4's `--ff-only` pull fails safe rather than overwriting anything.
- **Once pushed, everyone picks it up automatically** — no one needs to manually `git pull`; the SessionStart hook does it the next time they open Claude Code.
- If two people edit the same file without pushing in between, you'll hit a normal git merge conflict on `git pull` — resolve it like any other conflict (or just ask Claude to help).

<!-- Add new setup/onboarding notes below as the workflow evolves. -->
