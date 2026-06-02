# claude-writing-tools

Make Claude write like a human and run a content pipeline.

- **commands/** — `/humanize` (strip the AI tell), `/tighten` (cut 30–40% losslessly), `/hook` (10 scroll-stopping hooks). `cp commands/*.md ~/.claude/commands/`.
- **prompts/writing-and-content.md** — humanise rewrites + voice/hook prompts.
- **brand/** — a content-pipeline setup: brand-voice prompts feeding a self-hosted Postiz publisher across platforms.

## Use in Claude chat (claude.ai)

These tools are also packaged as **Agent Skills** under `skills/`, so they work in
**claude.ai**, not just Claude Code:

1. Grab a ready-to-upload zip from `skills/dist/` (one per skill).
2. In claude.ai → **Settings → Capabilities → Skills**, upload the zip (Pro/Max/Team/
   Enterprise). Or add it to a **Project** so it's available in that project's chats.
3. Each skill auto-triggers from its description — e.g. ask *"make this sound human"*
   and the `humanize` skill kicks in; no slash command needed.

In **Claude Code**, copy a skill folder into `~/.claude/skills/` (global) or the
project's `.claude/skills/`. The slash commands in `commands/` remain the quick path there.


---

_Curated/built from techniques shared by creators on Instagram (May 2026); marketing hype and inflated stats stripped out. Prompt text is rewritten, not copied; source handles are credited in the pack files._
