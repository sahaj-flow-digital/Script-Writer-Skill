# Setup — Script Writer Skill

This repo holds two Claude Code skills as sibling folders. Claude Code only auto-discovers a skill when its `SKILL.md` sits one level under `~/.claude/skills/`, so the repo is cloned to its own location and each skill folder is **symlinked** into `~/.claude/skills/`. One `git pull` updates both skills; every Claude Code session pulls automatically on startup.

**No prior GitHub or command-line experience needed** — every step below is a command you paste in. If something doesn't match what you see on screen, stop and ask Sahaj rather than guessing; it's much easier to fix before something's pushed than after.

## Prerequisites (check these first)

1. **A terminal app is open.** On a Mac: press `Cmd + Space`, type `Terminal`, hit Enter. You'll paste every command below into this window and press Enter after each one.
2. **You have a GitHub account.** If not, make one free at [github.com/signup](https://github.com/signup) first — use whatever email you'd like, personal or work.
3. **Git is installed.** Paste this in:
   ```bash
   git --version
   ```
   If you see something like `git version 2.x.x`, you're set. If instead you get `command not found`, macOS will usually offer to install it for you the first time you run a git command — follow that prompt, then re-run the command above.
4. **Claude Code is already installed and you can open it.** If you're reading this file, you probably already have it — if not, ask Sahaj.
5. **Ask Sahaj (or whoever owns the repo) to add you as a collaborator** on `github.com/sahaj-flow-digital/Script-Writer-Skill` — Settings → Collaborators on the repo, or they can just add your GitHub username. Without this, step 1 below will generate a key that GitHub accepts, but you still won't be able to push changes.

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

Paste this whole block in — it appends the config for you, so you don't need to open or edit any file by hand:

```bash
cat >> ~/.ssh/config <<'EOF'

Host github-scriptwriter
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_scriptwriter
  IdentitiesOnly yes
EOF
chmod 600 ~/.ssh/config
```

Test it:

```bash
ssh -T github-scriptwriter
```

You should see `Hi <your GitHub username>! You've successfully authenticated, but GitHub does not provide shell access.` — that "success" wording is expected, it's not actually an error.

**If instead you see `Permission denied (publickey)`** — the key hasn't been added on GitHub yet, or was added under the wrong account. Go back to step 1, re-copy the key with `cat ~/.ssh/id_ed25519_scriptwriter.pub`, and double check it's pasted at github.com/settings/ssh/new on the account Sahaj added as a collaborator.

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

**If `git clone` fails with `Permission denied (publickey)` or `Repository not found`**, it almost always means Sahaj hasn't added you as a collaborator yet (prerequisite #5) — check with him before troubleshooting further.

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

## 5. Verify it worked

Open Claude Code (a fresh session) and ask it something like:

> "Draft a hero video script about [any topic]."

You don't need to invoke anything by name — Claude Code reads each skill's description automatically and picks the right one based on what you say (mentioning "script," "hero video," "Reel," etc. is enough). If it responds by asking who the presenter is, or starts producing a script following the beat structure, it's working. If it just answers generically with no mention of the beat structure, presenter voice, or the arc, the skill likely isn't loading — see Troubleshooting below.

## 6. How updates flow between people

- **Nothing auto-commits or auto-pushes.** When Claude edits a `SKILL.md` or a `presenters/*.md` file during a session (e.g. adjusting someone's voice based on feedback), it will show you a diff, summarize the change in plain language, and ask before committing/pushing — see "Keeping this skill current" at the bottom of each `SKILL.md`.
- **Uncommitted work is never lost on session end.** Edits sit on disk regardless of git state; a commit is just a snapshot. The only thing that can go stale is your local copy relative to the repo, and step 4's `--ff-only` pull fails safe rather than overwriting anything.
- **Once pushed, everyone picks it up automatically** — no one needs to manually `git pull`; the SessionStart hook does it the next time they open Claude Code.
- If two people edit the same file without pushing in between, you'll hit a normal git merge conflict on `git pull` — resolve it like any other conflict (or just ask Claude to help).

## Troubleshooting

- **The skill doesn't show up / Claude doesn't seem to know it.** Check the symlinks exist: `ls -la ~/.claude/skills` should list `flow-video-scripts` and `flow-shortform-video-scripts` pointing at `~/repos/Script-Writer-Skill/...`. If they're missing, re-run the `ln -s` commands in step 3. Then fully quit and reopen Claude Code (not just start a new chat).
- **`Permission denied (publickey)` on clone, pull, or push.** You're not authenticated, or not added as a collaborator yet — see the note under step 3, and re-check step 1/2.
- **`ssh-keygen` says the file already exists.** You (or a previous setup attempt) already made this key. Either reuse it — skip straight to copying the existing `~/.ssh/id_ed25519_scriptwriter.pub` — or add `-f ~/.ssh/id_ed25519_scriptwriter2` (or similar) to make a second one, adjusting the filename in later steps to match.
- **You edited a skill file and it's not showing up for anyone else.** Confirm it was actually committed and pushed (not just saved) — `git -C ~/repos/Script-Writer-Skill status` should say "nothing to commit, working tree clean" and "up to date with 'origin/main'" once it's pushed.
- **Still stuck?** Ask Sahaj — screenshot the exact error and which step you were on.

<!-- Add new setup/onboarding notes below as the workflow evolves. -->
