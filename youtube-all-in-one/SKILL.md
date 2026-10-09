---
name: youtube-all-in-one
description: Use when the user gives YouTube links (or asks to find videos on a topic) and wants them transcribed into the all_in_one repo on GitHub, summarized, and optionally triaged by Jev ("стоит ли смотреть подробнее"). Triggers: транскрибируй ролики, закинь ролики в all in one, обработай youtube, выжимка по роликам.
---

# YouTube → all_in_one → summary (+ optional Jev triage)

Repo: https://github.com/wrx74/all_in_one
Local: `C:\Users\Andrei\Documents\Codex\2026-07-06\rf-2\work\all_in_one`
Python: `%LOCALAPPDATA%\Programs\Python\Python313\python.exe` (NOT the hermes venv)

## 1. Input

- URLs from the user, or a topic → find videos with `yt-dlp "ytsearch8:<query>" --flat-playlist --print "%(title)s | %(channel)s | %(view_count)s | https://youtu.be/%(id)s"` and confirm the list with the user.
- Skip IDs that already have a note: `ls notes/youtube/**/*<id>.md`.

## 2. Transcribe + push

Sequentially, in background (one git push per video, so no parallel runs):

```bash
cd <local repo>; git pull -q
for id in ...; do "$PY" scripts/youtube_to_notes.py "https://www.youtube.com/watch?v=$id" --push 2>&1 | tail -3; done
```

- No captions → retry that ID with `--whisper-fallback` only if the user agrees (slow on CPU).
- Notes land in `notes/youtube/<category>/YYYY-MM-DD-<title>-<id>.md`.
- Check: every ID has `Wrote ...` in the output, and `git status -sb` shows `main...origin/main` with nothing ahead.

## 3. Process each note

The script leaves `## Summary` empty. For each note:

1. Read the `## Author Description` and `## Transcript` sections (skip sponsor blocks and self-promo).
2. Replace the placeholder text under `## Summary` with:
   - **Суть**: 2–3 sentences
   - **Ключевые факты**: 3–7 bullets (tools, commands, repos, numbers)
   - **Практика**: what to actually do, if anything
   - **Ссылки**: repos and docs from the description; no affiliate or course links
3. Write in Russian. Keep tool names, commands and URLs verbatim.
4. One commit for all summaries: `docs: add summaries for <N> YouTube notes`, then push.

## 4. Output to the user

A short digest, nothing more:

```
<N> роликов → all_in_one (commit <sha>)
Общая выжимка: 3–5 bullets across all videos (consensus, contradictions)
По роликам: 1 line each — title — the main takeaway [— Jev: ✅/⏭ p=0.xx]
Не получилось: <ids + reason>
```

## 5. Optional: Jev triage ("стоит ли смотреть подробнее")

Only when the user asks for it, and only if a key is set (`TYPESAFE_API_KEY` or `OPENROUTER_API_KEY`). Never print the key.

- Per video, ask Jev a typed decision: `worth_deeper: yes | no`. Input: the user's stated goal + title + the summary (not the full transcript, to keep it cheap).
- Criteria to pass Jev, so it isn't guessing:
  - **yes**: concrete steps, repos or commands relevant to the goal; new information not covered by other videos
  - **no**: hype or clickbait, sponsor-heavy, duplicates another video, off-topic for the goal
- Use the API format from the current docs (`api.typesafe.ai/docs`, or the OpenRouter model page `openrouter.ai/typesafe`). Do not guess the endpoint or model id.
- Show the result as ✅ (worth it) or ⏭ (skip) with the probability. If p < 0.7, mark it `?` and give your own one-line reason.
- Jev output is schema-valid but can be wrong. It's a hint, not a verdict.
