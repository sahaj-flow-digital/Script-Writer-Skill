---
name: flow-video-scripts-jacob
description: >-
  Adapt a Flow Digital hero video script to Jacob's voice and preferred format — a full word-for-word,
  teleprompter-ready script. Use this after (or alongside) flow-video-scripts whenever the presenter is
  Jacob: when the user says "write this for Jacob," "Jacob's version," "make it in Jacob's voice," or hands
  over a draft to put into Jacob's style. This is a voice + output-format layer; it sits on top of the main
  flow-video-scripts spec (tonal arc, tool stance, no-fabrication rules) and does not replace it.
---

# Presenter Layer — Jacob

*Layers on top of `flow-video-scripts`. Load that skill's spec first; this only sets Jacob's voice and format.*

## Output format: word-for-word

Jacob films with a **teleprompter**, so write his scripts **fully written out, word for word** — complete spoken sentences, not bullets. He can and does deviate, but he wants a full read to stay on line.

- Write every beat as finished prose he can read straight through.
- Keep sentences short and breath-sized so they scroll well on a teleprompter.
- He tends to **go off on tangents** — keep the written line tight and on-point, and mark any line that's safe to drop for a cleaner cut with `[CUT-OK]` (per the main spec). This is the guardrail that keeps him "on line."
- Still leave the occasional `[riff here]` where a genuine aside would land, but default to giving him the words.

## Voice

- **Forward-deployed operator POV.** Jacob is the person companies hand messy, undefined problems to — "give me three days, I'll come back with something that works." Write from that in-the-trenches, been-there voice.
- **Client-facing and practical.** He talks to clients all day. He names the real frustration first ("why isn't the sales team getting through their emails?") the way a client would say it, then connects the dots they couldn't.
- **Warm, plain, a little self-deprecating.** Marketing background, calls himself a weak writer and a "nerd about automation." Confident on the substance, easygoing in delivery.
- **Concrete over abstract.** He reaches for real examples ("they were updating a spreadsheet by hand every day") rather than theory.
- **Comfortable being funny**, but keep comedy flagged for approval per the main spec — don't lock jokes into his read.

## Notes

- As Jacob gives feedback on drafts, capture the recurring corrections back into this file so future scripts need less editing. Follow the commit/push workflow in `../SKILL.md` under "Keeping this skill current" — show the diff, summarize the change, and ask before pushing.
- Everything in `flow-video-scripts` still governs: the seven-beat arc, tool-agnostic stance, short-form pull-out markers, and no invented stats/clients/results.
