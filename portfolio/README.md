# portfolio

A Claude skill for structural analysis of a personal stock/ETF portfolio. Three sub-commands, all backed by a single `PORTFOLIO.md` file, with reports written under `portfolio/reports/`.

> ⚠️ **Not investment advice.** The skill produces structural analyses and factual comparisons. It does not generate buy/sell signals, market-timing claims, or tax guidance. You decide what to do with the data.

## Trigger

```
/portfolio review                                  Periodic snapshot vs. last review
/portfolio x-ray                                   Deep structural analysis
/portfolio etf-compare ISIN1 ISIN2 [ISIN3] [ISIN4] Side-by-side comparison
/portfolio help                                    Show sub-command list
```

Optional flag: `--xlsx` (on `x-ray` and `etf-compare`) to produce an Excel export.

## What each sub-command does

| Sub-command | What it answers | Needs PORTFOLIO.md? | State? |
|---|---|---|---|
| `review` | "How's my portfolio doing since last time?" | yes | yes (writes `state/last-review.json`) |
| `x-ray` | "What's actually in there — concentration, overlap, exposure?" | yes | no |
| `etf-compare` | "Which of these 2–4 ETFs?" | no | no |
| `help` | sub-command list | no | no |

## Files

| File | Purpose | In Git? |
|---|---|---|
| `SKILL.md` | The skill for Claude Code in the terminal. References data-sources/etf-providers. | yes |
| `project-instructions.md` | Single-file version for claude.ai Projects. All workflow, sources, and template inline. | yes |
| `CONCEPT.md` | Design document, versioned before the build. | yes |
| `PORTFOLIO.md.example` | Template — copied to `PORTFOLIO.md` on first run. | yes |
| `PORTFOLIO.md` | Your real positions. | no (gitignored) |
| `references/data-sources.md` | Which source for which data point. Tier list + URL patterns. | yes |
| `references/etf-providers.md` | URL patterns per ETF issuer (iShares, Xtrackers, Vanguard, …). | yes |
| `reports/` | Timestamped markdown reports. | no (gitignored) |
| `state/last-review.json` | Snapshot used by `review` to compute deltas. | no (gitignored) |

## Two ways to use this skill

**A) Claude Code (terminal):** symlink `portfolio/` into `~/.claude/skills/` (see root README). Then `/portfolio` works in any `claude` session. Reports are written to disk; widget rendering depends on the host.

**B) claude.ai Projects (web, iPad, mobile):** copy the full content of `project-instructions.md` into a new Project's custom instructions. Paste your `PORTFOLIO.md` content into the chat when running a sub-command. Reports render inline; widget renders with full interactivity.

## First run

The skill auto-creates `PORTFOLIO.md` from `PORTFOLIO.md.example` the first time it runs in Claude Code. Edit it to reflect your real positions, then re-run.

In claude.ai Projects, paste your portfolio into the chat each time (Projects don't have persistent file storage).

## Output

- **Markdown report** under `portfolio/reports/{date}-{sub-command}.md` (Claude Code) or inline in chat (claude.ai). Git-versionable if you opt in by removing `portfolio/reports/` from `.gitignore` in a private fork.
- **Interactive widget** (`x-ray` and `etf-compare`) with SVG charts, collapsible sections, heatmap-style overlap matrix. Best experienced in claude.ai.
- **Excel export** (`--xlsx`) — one sheet per data table, written next to the markdown report.

## Data sources

The skill pulls from justETF, provider websites (iShares, Xtrackers, Vanguard, …), trackingdifferences.com, Yahoo Finance, finanzen.net, and the ECB reference rates. Full list and URL patterns in [`references/data-sources.md`](./references/data-sources.md) and [`references/etf-providers.md`](./references/etf-providers.md).

End-of-day data only. No live ticks. Long-term investing doesn't need them.

## What the skill won't do

- ❌ Buy/sell recommendations
- ❌ Market-timing predictions
- ❌ Tax optimization (Vorabpauschale, Teilfreistellung — Steuerberater-Job)
- ❌ Crypto coverage
- ❌ Broker API integration (you maintain `PORTFOLIO.md` manually)

## Customization

- **Positions**: edit `PORTFOLIO.md` directly. Tables are markdown — git-diff-friendly.
- **Target allocation**: optional block at the top of `PORTFOLIO.md`. Only when defined does `review` compute drift.
- **Data sources**: edit `references/data-sources.md` to add/remove sources or change preference order.
- **Provider URLs**: edit `references/etf-providers.md` when an issuer changes its URL scheme.
