# ai-news

A Claude skill that produces a curated weekly digest of AI news — research, industry moves, policy, and tools — as an interactive expandable widget. German-language summaries, soft focus weighting for topics you care about, and an auto-growing glossary with clickable inline definitions.

## Trigger

- `/ai-news` — default: last 7 days, 10–15 items, all categories
- `/ai-news 3d` — different time window (`3d`, `14d`, `last 24h`)
- `/ai-news 20 policy` — different item count and/or topic focus
- Natural language: "what happened in AI this week", "KI-News", "AI digest"

## Files

| File | Purpose | In Git? |
|---|---|---|
| `SKILL.md` | The skill itself for Claude Code in the terminal. References separate FOCUS/GLOSSAR/sources files. | yes |
| `project-instructions.md` | Single-file version for claude.ai Projects. All content (workflow, sources, focus, glossary) inline. Use this when you can't run Claude Code. | yes |
| `references/sources.md` | Prioritized source list, grouped by tier. Edit to curate. | yes |
| `FOCUS.md` | Your focus topics. Matches get sorted to the top with a ★-Fokus badge. | no (gitignored) |
| `FOCUS.md.example` | Template — copied to `FOCUS.md` on first run if missing. | yes |
| `GLOSSAR.md` | Auto-growing glossary. Terms in digest summaries become clickable. | no (gitignored) |
| `GLOSSAR.md.example` | Seed template — copied to `GLOSSAR.md` on first run if missing. | yes |

## Two ways to use this skill

**A) Claude Code (terminal, Mac/Linux/Windows):** symlink this folder into `~/.claude/skills/` (see root README). Then `/ai-news` works in any `claude` session. Output is text-only — no interactive widget.

**B) claude.ai Projects (web, iPad, mobile):** copy the full content of `project-instructions.md` into a new Project's instructions. Then `/ai-news` works in that Project on any device. Full interactive widget with clickable glossary terms.

## First run

The skill auto-creates `FOCUS.md` and `GLOSSAR.md` from the `.example` templates the first time it runs. Both files are then yours to edit. They're gitignored so your private content stays out of the repo.

## Output

An interactive HTML widget in chat with:
- Date range header (e.g. "AI-Digest · 15.–22. Mai 2026") and item count
- Category filter chips (Alle / Research / Industry / Policy / Tools)
- Collapsed cards sorted with focus matches first, ★-Fokus badge on matched items
- Click a card to expand: German summary, "Warum es wichtig ist", source link
- Click any dotted-underlined term to see its definition as an inline popover

## Customization

- **Sources**: edit `references/sources.md` — six tiers, plus an "explicitly avoid" list at the bottom
- **Focus topics**: edit `FOCUS.md` — `- KEYWORD1, KEYWORD2: short description`
- **Glossary**: edit `GLOSSAR.md` — `### TERM` heading + 1–3 sentence definition. Skill never overwrites existing entries.

## Language

Headlines stay in source language (English or German depending on origin). Summaries and "Warum es wichtig ist" lines are in German.
