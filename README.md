# Skills

Claude Code skills.

## jev-claude-codex

How to use Jev AI (the TypeSafe decision model) with Claude Code and Codex. Built from 12 YouTube videos; the transcripts are in [wrx74/all_in_one](https://github.com/wrx74/all_in_one/tree/main/notes/youtube/ai).

What it covers:
- Model routing by task complexity: a `UserPromptSubmit` hook, explicit categories and a confidence threshold
- Picking skills and files with Jev to save tokens
- Guardrails: Toolgate, Abide, Jev Belay, Steer or Queue, Compact Adviser
- Claude Code + Codex together: Claude builds, Codex reviews before the PR
- Access options: TypeSafe direct, OpenRouter, Vercel AI Gateway

### Install

```bash
git clone https://github.com/wrx74/Skills
cp -r Skills/jev-claude-codex ~/.claude/skills/
```

For Codex, point `AGENTS.md` at `jev-claude-codex/SKILL.md` or copy the relevant sections into it.

## youtube-all-in-one

Turns YouTube videos into notes in [wrx74/all_in_one](https://github.com/wrx74/all_in_one), on request.

- Takes links, or a topic (then searches and asks you to confirm the list)
- Transcribes with `scripts/youtube_to_notes.py --push`
- Fills each note's empty `## Summary`: the gist, key facts, what to do, links
- Replies with a short digest across all videos
- Optional: a Jev triage per video, ✅ worth a closer look or ⏭ skip, with the probability (needs `TYPESAFE_API_KEY` or `OPENROUTER_API_KEY`)

Install: `cp -r Skills/youtube-all-in-one ~/.claude/skills/`. For Codex, use `~/.codex/skills/`.
