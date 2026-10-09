---
name: jev-claude-codex
description: Use when setting up or using Jev AI (TypeSafe decision model) together with Claude Code and/or Codex — model routing by task complexity, skill/file selection, tool-call guardrails, "is it really done" checks, Claude+Codex cross-review. Triggers: jev, typesafe, роутинг моделей, выбор модели по сложности, jev + claude, jev + codex.
---

# Jev AI + Claude Code + Codex

Source: 12 YouTube notes in github.com/wrx74/all_in_one, `notes/youtube/ai/2026-10-09-*`.

## What Jev is (and is not)

- Jev = TypeSafe's **decision model**. It does NOT generate text or code.
- Input: context + a typed schema of options. Output: a choice / yes-no / score + **probability**.
- Fast (hundreds of ms; one test: Jev 413 ms vs Haiku 1486 ms vs Sonnet 4545 ms), very cheap (input tokens only).
- "Zero hallucinations" means only *schema-valid* output. Jev can be confidently wrong in the correct format.
- Code is written by Claude / Codex. Jev only decides **where, what and whether**.

## Access (pick one)

| Option | When |
|---|---|
| TypeSafe direct (`console.typesafe.ai`, docs `api.typesafe.ai/docs`) | lowest latency, official SDK (pip / JS) and skill work as-is |
| OpenRouter (`openrouter.ai/typesafe`, model "Jev Latest") | no waitlist, one key for Jev + Claude + GPT |
| Vercel AI Gateway | if already on Vercel |

Key goes to an env var (`TYPESAFE_API_KEY` / `OPENROUTER_API_KEY`) or a git-ignored `.env` as `NAME=value`. Never into code or chat.

Official Claude Code skill: `docs.typesafe.ai/agent-skill` (marketplace + plugin install). For OpenRouter, ask Claude to adapt the skill (example: github.com/philippacsany/agent-skills) and make it non-proactive (only when Jev is explicitly needed).

## Where Jev pays off

Ranked by how consistently the videos confirm it:

1. **Model routing by complexity**: Jev classifies the task → cheap model for simple, frontier model for hard. Ref impl: github.com/gargpratyush/jev-router.
2. **Skill selection**: Jev picks which skill to load from short descriptions instead of the LLM reading the whole catalog (one case: 137 skills ≈ 19k tokens of system prompt). Ref: github.com/safzanpirani/pi-jev-skill-picker.
3. **File shortlisting**: Quicksilver (github.com/UditAkhourii/quicksilver): 37 files in ~2 s, ~67k tokens saved.
4. **Guardrails on tool calls**: Toolgate (github.com/RiskAverseTech/toolgate) blocks destructive / out-of-scope / exfiltration actions (works with TypeSafe, OpenRouter, Vercel keys). Downside: also blocks legit deletes.
5. **Rule compliance**: Abide (github.com/coldteadotai/abide) compiles AGENTS.md + CLAUDE.md into a yes/no rubric and checks every edit. Works with Claude Code and Codex.
6. **Done-check**: Jev Belay (github.com/valentynkit/jev-belay) won't let Claude stop without evidence the task is done (threshold ~75%).
7. **Mid-run messages**: Jev Steer or Queue (github.com/Larkspur-Wang/Jev_steer_or_queue) chooses steer / queue / interrupt, acting only at ≥90% confidence. Claude Code + Codex.
8. **When to compact**: Compact Adviser (github.com/kunchenguid/compact-adviser).

## Routing setup (core pattern)

Hook: `UserPromptSubmit` in Claude Code. Every prompt goes to Jev first.

1. Define **explicit, non-overlapping** categories with examples:
   - `trivial`: rename, format, one-line fix, factual lookup → cheapest model (Haiku / mini GPT) or answer directly via a tool call
   - `standard`: a feature in known files, tests, refactor of one module → Sonnet / Codex
   - `hard`: architecture, cross-module bug, security, money paths → Opus / GPT-6 Astra
   - `risky`: destructive or irreversible → frontier model + human confirmation
2. Jev returns category + probability.
3. **Confidence threshold**: below ~0.8 → escalate to the stronger model. Never route down on low confidence.
4. Log every decision (prompt hash, category, p, chosen model) to tune the criteria.
5. Mark Jev-handled replies visibly (e.g., a different color) so you know when the main model was skipped.

> "Jev does not rescue a lazy schema." Vague or overlapping categories give fast, cheap, wrong routing. Spend the effort on the criteria and a small eval set (20–50 labeled prompts) before trusting it.

## Claude Code + Codex together

- VS Code: Claude (Agent SDK) and Codex (official OpenAI extension) both run inside Copilot Chat. Switch via the session target at the bottom of the chat.
- Working split from the videos: **Claude/Opus builds, Codex reviews.** Run 1–3 Codex review subagents before every PR. In the demo Codex caught flaws in Opus's solutions.
- Jev sits in front of both: routes the task (Claude vs Codex vs cheap model), and Abide / Steer-or-Queue / Belay work for either agent.
- Shared rules: keep `AGENTS.md` (Codex) and `CLAUDE.md` (Claude) in sync, or have CLAUDE.md reference AGENTS.md, so Abide checks one rule set.

## Install order (minimal)

1. Get a key (TypeSafe or OpenRouter) → env var. Smoke-test with the curl example from the provider page.
2. Install the Jev skill (official or OpenRouter-adapted). Remove duplicates, `/reload-skills`.
3. Add the routing hook (`UserPromptSubmit`) with the categories above.
4. Add guardrails one at a time: Belay → Abide → Toolgate. Verify each before adding the next.
5. Codex extension in VS Code + an "always Codex-review before PR" rule in CLAUDE.md.

## Don't

- Don't use Jev to generate code or answers that need reasoning text.
- Don't trust its "0% hallucination" claim. Validate on your own labeled data.
- Don't enable every plugin at once: hooks stack latency and conflict.
- Third-party repos above are community code. Read them before installing; Toolgate-style hooks see every command.
