# Where to run this skill: Claude Code vs. claude.ai

**Strong recommendation: use Claude Code.** It's the only option confirmed to actually do both halves of this workflow — auto-pulling the latest skill files, and pushing your edits back to the repo. The claude.ai chat route has real limitations (below) that make it more manual than it first looks. Only use claude.ai if someone genuinely won't set up a terminal.

## What we confirmed testing this (2026-09-04)

We initially assumed claude.ai had a live, write-capable GitHub connector you could enable in Settings → Connectors, the same way Claude Code has real git access. That turned out not to be the case on the accounts we tested:

- **No GitHub connector appeared in Settings → Connectors.** The only GitHub-related feature was **"Add from GitHub,"** a button under the **+** icon in a chat — this is a **read-only content picker**. It attaches a snapshot of files/repo content as context; it does not give Claude an ongoing, authenticated connection it can write through.
- Without a real connector, a claude.ai session asked to fetch files from this repo falls back to plain unauthenticated `curl`/web fetch against `raw.githubusercontent.com` (only possible because this repo is public). That explains why reading "worked" in earlier testing — it looked like a live pull, but it wasn't a connector, wasn't authenticated, and carried no write credentials at all.
- **Consequence: claude.ai cannot push changes back to this repo** on its own in this setup. When it suggests an edit (e.g. a Jacob voice correction) and you say yes, it has nothing to write with — the change stays unconfirmed and unwritten, full stop. Someone has to copy the diff and commit it manually via GitHub's web UI, or via Claude Code.
- If your org's plan supports **adding a custom remote MCP connector** (Settings → Connectors → Add custom connector), GitHub does publish an official hosted MCP server that supports real read/write — that would close this gap without a terminal. We have not verified whether that option is available/works end-to-end on our plan; treat it as untested until someone confirms it.

## Side-by-side (as actually confirmed, not as originally designed)

| | **Claude Code** (this repo, symlinked) | **claude.ai chat/Project** |
|---|---|---|
| **Where it works** | Any Claude Code session on that machine | Only inside a Project with the right custom instructions |
| **Auto-triggers by description** (no need to name the skill) | Yes — native Skills behavior | No — Claude only checks GitHub because custom instructions tell it to |
| **Reads the latest version?** | Yes — real `git pull` once per session start | Yes, but only as a public, unauthenticated file fetch (not a connector) — fine for reading, proves nothing about writing |
| **Can push edits back to the repo?** | **Yes** — real git, works today | **No**, confirmed — no write-capable connector found. An edit Claude proposes just sits unwritten; a human has to commit it manually |
| **Setup effort** | One-time, per person: SSH key, clone, symlink, hook (see `SETUP.md`) | One-time: create a Project, paste in custom instructions (below) — but doesn't get you write access regardless |
| **Requires terminal / git / SSH** | Yes | No |
| **Best for** | Anyone who can run a few terminal commands once — this is the workflow the rest of this repo assumes | Only as a read-only drafting tool, with manual copy-paste back to GitHub for anything that needs saving |

---

## Option A: Claude Code (recommended)

Full instructions are in [`SETUP.md`](./SETUP.md). Short version: generate a dedicated SSH key, clone this repo to `~/repos/Script-Writer-Skill`, symlink each skill folder into `~/.claude/skills/`, and add the `SessionStart` hook so it auto-pulls on every session start. This is the only setup where "suggest a push" in a session can actually become a real commit.

## Option B: claude.ai Project (read-only drafting, manual save-back)

Use this only if someone won't set up a terminal, and accept that saving any edit back to the repo is a manual step for them, not something Claude does for them.

1. **Create a Project** (or open the one you want this in).
2. **Open the Project's settings** and paste this into the custom instructions field:

   ```
   Before writing or editing a Flow Digital video script, fetch the current version of the
   relevant file(s) from sahaj-flow-digital/Script-Writer-Skill on the main branch, using
   "Add from GitHub" or the GitHub connector if one is available in this session — check
   what's actually available and tell the user which method you used. If neither is
   available, fall back to a plain fetch of the raw file and say so explicitly rather than
   presenting it as a live authenticated read.

   - Long-form "hero" video (~5-10 min): flow-video-scripts/SKILL.md, plus the named
     presenter's file in flow-video-scripts/presenters/ (jacob.md or sahaj.md).
   - Short-form clip (60-90s, LinkedIn/Instagram): flow-shortform-video-scripts/SKILL.md.

   Follow those files' instructions exactly — arc, tone, markers, and output format.

   You do not have write access to this repo from claude.ai. If a change to a skill file
   comes out of the conversation, show the exact new file content (or a clear diff) and tell
   the user they need to save it themselves — either by pasting it into GitHub's web editor,
   or by having someone with Claude Code apply and push it. Never claim you pushed or
   committed anything.
   ```

3. Every new chat inside that Project pulls the skill files at the start of the conversation. Re-attach via "Add from GitHub" if the custom instructions alone don't trigger a fetch — it's a manual button, not guaranteed to fire automatically the way a connector's tool-call would.

**Limitations to know about:**
- Only applies inside that one Project — a plain claude.ai chat elsewhere won't know about any of this.
- Read access is a snapshot/unauthenticated fetch, not a guaranteed live connector read (works fine in practice since the repo is public, just don't describe it to people as "live" in the connector sense).
- **No write-back.** Any edit has to be applied manually by a human, either by pasting into GitHub directly or by handing it to someone on Claude Code.
