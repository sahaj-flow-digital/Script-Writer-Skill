# Where to run this skill: Claude Code vs. claude.ai Projects

There are two supported ways to use these skills. They work differently under the hood — pick based on whether the person is a Claude Code user or a claude.ai (web/app) user.

## Side-by-side

| | **Claude Code** (this repo, symlinked) | **claude.ai Project** + GitHub connector |
|---|---|---|
| **Where it works** | Any Claude Code session on that machine | Only inside that one Project |
| **Auto-triggers by description** (no need to name the skill) | Yes — native Skills behavior | No — Claude only checks GitHub because the Project's custom instructions tell it to, every conversation |
| **Always the latest version?** | Yes, but on a delay — pulls once per session start (`SessionStart` hook) | Yes, and live — reads the file straight from GitHub at the start of each chat, no lag at all |
| **Setup effort** | One-time, per person: SSH key, clone, symlink, hook (see `SETUP.md`) | One-time, per person: create a Project, connect the GitHub connector, paste in custom instructions (below) |
| **Requires terminal / git / SSH** | Yes | No |
| **Editing the skill files** | Edit locally, commit, push (see "Keeping this skill current" in each `SKILL.md`) | Ask Claude to update the file via the GitHub connector — same "show diff, confirm before writing" rule applies |
| **Best for** | People comfortable in a terminal, or who want it working the same way across every local session automatically | People who live in claude.ai and don't want any local setup |

**Bottom line:** Claude Code is the "set it up once, forget about it" option, with a small lag (one pull per session start). The Project route has zero lag — it's reading GitHub live — but only works in that one Project, and needs the fetch instruction re-stated to Claude each time via the Project's custom instructions (which is a one-time setup too, it just lives on claude.ai's side instead of your machine).

---

## Option A: Claude Code

Full instructions are in [`SETUP.md`](./SETUP.md). Short version: generate a dedicated SSH key, clone this repo to `~/repos/Script-Writer-Skill`, symlink each skill folder into `~/.claude/skills/`, and add the `SessionStart` hook so it auto-pulls on every session start.

## Option B: claude.ai Project + GitHub connector

1. **Connect the GitHub connector** to your claude.ai account, if it isn't already (Settings → Connectors), and make sure it has access to `sahaj-flow-digital/Script-Writer-Skill`.
2. **Create a Project** (or open the one you want this in) — e.g. "Flow Digital Scripts."
3. **Open the Project's settings** and paste this into the custom instructions field:

   ```
   Before writing or editing a Flow Digital video script, use the GitHub connector to fetch
   the current version of the relevant file(s) from sahaj-flow-digital/Script-Writer-Skill
   on the main branch:

   - Long-form "hero" video (~5-10 min): flow-video-scripts/SKILL.md, plus the named
     presenter's file in flow-video-scripts/presenters/ (jacob.md or sahaj.md).
   - Short-form clip (60-90s, LinkedIn/Instagram): flow-shortform-video-scripts/SKILL.md.

   Follow those files' instructions exactly — arc, tone, markers, and output format. Don't
   rely on a cached or remembered version; always fetch fresh at the start of the
   conversation, since these files change as the team refines voice and rules.

   If asked to save an edit back to the repo, show the user a plain-language summary of what
   changed and get explicit confirmation before writing to GitHub — this is a shared repo
   everyone else reads from, so nothing gets pushed without a yes.
   ```

4. Every new chat inside that Project now pulls the live skill files automatically at the start of the conversation — no local setup, no re-uploading anything when the repo changes.

**Limitation to know about:** this only applies inside that Project. A plain claude.ai chat outside any Project won't know about these instructions, and won't auto-fetch anything.
