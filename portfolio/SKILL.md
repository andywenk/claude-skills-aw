---
name: portfolio
description: Structural analysis and factual comparisons for a personal stock/ETF portfolio. Sub-commands /portfolio review (periodic snapshot vs. last review), /portfolio x-ray (concentration, overlap, exposure breakdown), /portfolio etf-compare ISIN1 ISIN2 [...] (side-by-side comparison of 2–4 ETFs), and /portfolio help. Reads PORTFOLIO.md and fetches current data from web sources. Strictly analytical — no investment advice, no buy/sell signals, no tax guidance.
---

# Portfolio (`/portfolio`)

> ⚠️ **Not investment advice.** This skill provides structural analysis and factual comparisons. It must not produce buy/sell recommendations, market-timing claims, or tax guidance. When summarizing, stick to what the data shows. If the user asks for a recommendation, decline and explain that the skill is analytical only.

A skill that helps the user understand their portfolio: what's in it, how it's allocated, what changed since the last check, and how alternative ETFs compare. Outputs are always reproducible from `PORTFOLIO.md` and the current state of web sources — never from invented data.

## When to trigger

- The user types `/portfolio` (with or without a sub-command and arguments)
- The user asks for a portfolio review, an ETF comparison, an overlap/concentration check, or similar analytical questions about their holdings
- If the trigger is ambiguous (`/portfolio` alone, or natural language like "check my portfolio"), default to showing the help text and ask which sub-command they want

## Sub-commands

```
/portfolio review                                  Periodic snapshot vs. last review
/portfolio x-ray                                   Deep structural analysis
/portfolio etf-compare ISIN1 ISIN2 [ISIN3] [ISIN4] Side-by-side ETF comparison (2–4 ISINs)
/portfolio help                                    Show sub-command list
```

Optional flag for `x-ray` and `etf-compare`:
- `--xlsx` — also produce an Excel export. See "Excel export" below.

## File layout

```
portfolio/
├── SKILL.md                       ← this file
├── PORTFOLIO.md.example           template (in git)
├── PORTFOLIO.md                   live data (gitignored, created on first run)
├── references/
│   ├── data-sources.md            which source for which data point
│   └── etf-providers.md           URL patterns per issuer
├── reports/                       (gitignored) all generated reports
└── state/                         (gitignored) skill state for review comparisons
    └── last-review.json
```

## First-run bootstrap

If `PORTFOLIO.md` does not exist, copy `PORTFOLIO.md.example` to `PORTFOLIO.md` and tell the user once that the file was created so they know where to customize it. Then ask the user to populate it with their real positions before running any sub-command beyond `help`.

## Common workflow steps (shared by all sub-commands)

### Read references first

At the start of every run, read:

1. **`references/data-sources.md`** — preferred sources per data point.
2. **`references/etf-providers.md`** — URL patterns per issuer.

These files are user-editable. If a source is missing or a URL pattern is stale, follow what the file says, not what you remember.

### Resolve the FX rate once per run

For any currency conversion, use a single EUR reference rate per run (ECB published rate, see `data-sources.md`). Don't fetch per-position rates — that introduces drift between figures in the same report.

### Be honest about uncertainty

- If a data point disagrees between sources, **report both values and name the sources**. Don't average and don't silently pick one.
- If a data point cannot be found, **say so**. Never invent a plausible number.
- If a web fetch fails, fall back per the tier order in `data-sources.md` and note which source was used if it wasn't the primary one.

### Always write a markdown report

Every sub-command (except `help`) writes a timestamped report under `portfolio/reports/`:

```
portfolio/reports/{YYYY-MM-DD}-review.md
portfolio/reports/{YYYY-MM-DD}-x-ray.md
portfolio/reports/{YYYY-MM-DD}-etf-compare-{ISIN1}-vs-{ISIN2}[-vs-{ISIN3}][-vs-{ISIN4}].md
```

If a report for the same date and sub-command already exists, append `-2`, `-3`, etc.

The report is the canonical output. The widget (when rendered) is a richer presentation of the same information.

### Render an interactive widget where appropriate

For `x-ray` and `etf-compare`, render an interactive widget via `visualize:show_widget` in addition to the markdown report. Call `visualize:read_me` with `["interactive"]` first for styling tokens. Use SVG-native charts (donut, bar, heatmap) — no external chart libraries.

For `review`, the markdown report is the primary output. A simple widget with the headline numbers is optional but secondary.

If widget rendering isn't available (e.g. terminal-only Claude Code), the markdown report is sufficient — don't apologize for missing visuals.

---

## Sub-command: `review`

> *"How's my portfolio doing?"*

Periodic snapshot. Compares against the previous snapshot in `state/last-review.json` and writes a new snapshot at the end.

### Workflow

1. **Read `PORTFOLIO.md`.** Parse positions, target allocation (if present), and notes. If the file doesn't exist or has no positions, ask the user to populate it.

2. **Read `state/last-review.json`** if it exists. This contains the previous snapshot (date, per-ISIN values, total). If missing, this is a first review — no comparison possible; note that in the report.

3. **Fetch current prices** for each position:
   - ETFs: justETF profile page → most recent NAV
   - Single stocks: Yahoo Finance → most recent close
   - Bonds-ETFs: same as ETFs
   - Cash: take face value from `PORTFOLIO.md`
   - All non-EUR values converted using the same ECB rate (see Common steps).

4. **Compute headline metrics:**
   - Current market value per position (EUR and original currency for non-EUR positions)
   - Total portfolio value
   - Δ since last review (absolute EUR and %)
   - Δ since beginning of current year (YTD)
   - Top 3 winners and top 3 losers by absolute EUR change since last review

5. **Allocation analysis:**
   - Asset-class allocation (equity ETFs / single stocks / bond ETFs / cash / other)
   - High-level regional split (North America / Europe / Emerging Markets / Other) using ETF holdings data
   - If `Target-Allocation` is defined in `PORTFOLIO.md`: drift table (target % vs. actual %, Δ in percentage points)

6. **Ad-hoc events since last review:**
   - For each position, run a web search restricted to the period since last review: `{ISIN} OR {name} dividend OR news OR announcement`
   - Filter for material events: dividend declarations, index reconstitutions, M&A, ETF closures or merges, large index methodology changes
   - Skip routine price commentary and analyst rating shuffles

7. **Compose the report** (`reports/{date}-review.md`) with:
   - Header: review date, basis currency, FX rate used
   - Headline numbers (current total, Δ since last review, YTD)
   - Per-position table: ISIN, name, shares, price, market value EUR, Δ EUR, Δ %
   - Top winners / losers
   - Allocation breakdown
   - Drift table (if target allocation defined)
   - Events list (one bullet per material event, with source link)
   - Notes section (echo back any items from `PORTFOLIO.md` → Notes that mention upcoming changes)

8. **Write the new snapshot** to `state/last-review.json`:
   ```json
   {
     "last_review_date": "{today YYYY-MM-DD}",
     "fx_rate_eur_usd": 1.0800,
     "snapshot": {
       "{ISIN}": { "shares": 245, "price_eur": 89.30, "value_eur": 21878.50 }
     },
     "total_value_eur": 47230.00
   }
   ```

9. **Summarize in chat** in 3–5 sentences: total, Δ since last review, the single most noteworthy event. No recommendations.

### What `review` does NOT do

- Does not say whether to buy or sell anything.
- Does not predict where the market is heading.
- Does not compute tax-optimization moves (Vorabpauschale, Teilfreistellung, etc.).

---

## Sub-command: `x-ray`

> *"What's actually in there?"*

Deep structural analysis. Use when the user wants to understand concentration, overlap, and exposure — typically once a quarter or after meaningful portfolio changes.

### Workflow

1. **Read `PORTFOLIO.md`.** Parse positions. Skip the target-allocation block (not used here).

2. **Fetch holdings for each ETF** from the provider site (preferred) or justETF (fallback). For each ETF, you need:
   - Full holdings list with weights (top 100 minimum; top 250 ideal for overlap calculations)
   - Sector breakdown (GICS)
   - Country breakdown
   - Currency breakdown
   - Factor profile if available (most ETFs label themselves as e.g. "broad market", "value", "growth", "small cap"; pull this from the prospectus / factsheet)
   - ESG screening info (described in KIID / factsheet, even for non-ESG ETFs — "no screen applied" is also a finding)

3. **Compute concentration look-through.** For each unique security held across the portfolio (via ETFs and as a single stock), aggregate:
   - Total weight in EUR terms
   - % of total portfolio
   - Which holding vehicles it comes from (e.g. "Apple: 4.2% via MSCI World, 7.1% via S&P 500, 0.5% via MSCI EM IMI, 100% via direct holding → combined 3.8% of portfolio")
   - Flag if combined weight > 5% (★ concentration marker)

4. **Compute ETF-overlap matrix.** For each pair of ETFs in the portfolio, compute the holdings overlap as a percentage. Two definitions both useful:
   - **Name overlap**: % of holdings (by count) appearing in both
   - **Weight overlap**: sum of `min(weight_A, weight_B)` across shared holdings (Jaccard-style on weights)
   Report both. Use weight overlap for the headline; name overlap as a secondary column.

5. **Compute regional exposure.** Aggregate country weights across all equity holdings (ETFs + single stocks), expressed as % of the equity portion of the portfolio. Top 10 countries explicitly, rest as "Other".

6. **Compute sector exposure.** GICS sectors, same aggregation. Equity portion only; bond ETFs and cash get their own row.

7. **Compute currency exposure.** Underlying currency of each holding (USD/EUR/JPY/GBP/CHF/Other). Important: this is the currency the *holdings* are denominated in, not the listing currency of the ETF.

8. **Factor tilt.** Best-effort qualitative summary based on what the ETFs label themselves as. If the portfolio is e.g. mostly broad-market ETFs plus a single growth ETF, say so. Don't compute factor exposures statistically — that requires returns data outside the skill's scope.

9. **ESG info column.** For each ETF, one short cell: what the screen is (ESG Screened / SRI / Climate Transition / no screen) and what the main exclusions are (controversial weapons, tobacco, fossil fuels). **Informational only** — no scoring, no judgment.

10. **Compose the report** (`reports/{date}-x-ray.md`) with sections:
    - Headline summary (3–5 sentences: top concentration, top overlap pair, currency dominance)
    - Concentration look-through table (sorted by combined weight desc)
    - Overlap matrix (square table)
    - Regional table (top 10 + Other)
    - Sector table
    - Currency table
    - Factor tilt narrative paragraph
    - ESG info table

11. **Render the widget.** SVG-native charts:
    - Donut for asset-class allocation
    - Bar chart for top-10 country exposure
    - Bar chart for sector exposure
    - Heatmap-style table for overlap matrix (cell background scaled to %)
    - List of concentration look-through with ★ on positions > 5%
    Sections are collapsible. Keep the widget calm and dense.

12. **Excel export if `--xlsx`.** Invoke the `anthropic-skills:xlsx` skill, passing the concentration, overlap, regional, sector, currency, and ESG tables as separate sheets. Save to `reports/{date}-x-ray.xlsx`.

### Edge cases

- **ETF holdings data is older than 30 days**: note the as-of date in the report. Holdings drift slowly; a 1-month-old snapshot is fine, but the user should know.
- **A single stock isn't in any ETF**: it appears only as itself in the concentration table.
- **Currency-hedged ETF**: the underlying-currency breakdown reflects holdings, not the hedge. Add a separate note for hedged ETFs explaining that FX risk on those positions is reduced.

---

## Sub-command: `etf-compare`

> *"Which of these ETFs?"*

Side-by-side comparison of 2–4 ETFs. Does not read `PORTFOLIO.md` — operates purely on the ISIN arguments.

### Argument parsing

- Required: 2, 3, or 4 ISINs (12 characters, `[A-Z]{2}[A-Z0-9]{9}[0-9]`).
- If fewer than 2 ISINs are passed, ask the user for more before proceeding.
- If more than 4 ISINs are passed, ask the user to narrow down (4 is the practical limit for a readable side-by-side).

### Workflow

1. **For each ISIN, fetch ETF master data** (justETF profile + provider site):
   - Name and issuer
   - Index tracked
   - TER (total expense ratio)
   - Tracking difference 1Y / 3Y / 5Y (trackingdifferences.com)
   - Fund size (AUM in EUR)
   - Inception date
   - Replication method (physical full / physical sampling / synthetic)
   - Number of holdings
   - Domicile (Ireland / Luxembourg / Germany)
   - Distribution policy (accumulating / distributing) and frequency
   - Currency hedging (yes/no, hedge target currency)
   - ESG methodology (screen type + main exclusions, informational only)
   - Performance 1Y / 3Y / 5Y / 10Y (absolute return and return vs. index where available)

2. **Fetch top 10 holdings for each ETF.** Used for overlap calculation.

3. **Compute pairwise top-10 holdings overlap** for each pair of ETFs in the input set. Same two metrics as `x-ray` (name overlap, weight overlap).

4. **Compose the report** (`reports/{date}-etf-compare-{ISINs}.md`) with:
   - Header row identifying each ETF (name, ISIN, issuer)
   - Side-by-side data table (one column per ETF, one row per data point)
   - Top-10-holdings overlap matrix
   - Performance comparison table (1Y/3Y/5Y/10Y rows, one column per ETF)
   - Per-ETF top-10 holdings list (expandable in widget)
   - Notes section: any data points that disagreed across sources, any data points that couldn't be found

5. **Render the widget.** SVG-native:
   - Side-by-side card grid (one card per ETF) with the headline data points
   - Heatmap-style overlap matrix
   - Mini-line-chart for cumulative performance over 5 years (if data available)

6. **Excel export if `--xlsx`.** Same approach as `x-ray`: invoke `anthropic-skills:xlsx`, one sheet per table.

### What `etf-compare` does NOT do

- Does not say which ETF is "best". The user makes that call — the skill surfaces the facts.
- Does not include ETFs with the same ISIN twice (deduplicate the input list).
- Does not compare bond ETFs and equity ETFs side by side in the same call (they're not really comparable). If the user mixes types, render the comparison but warn at the top of the report.

---

## Sub-command: `help`

Show the sub-command list and a one-line description of each. Mention the disclaimer ("not investment advice"). Used for `/portfolio` with no args and `/portfolio help`.

---

## Output checklist (every sub-command)

Before showing results, verify:

- [ ] Disclaimer present (in the report header, not buried)
- [ ] FX rate used is stated explicitly
- [ ] Data source named for any figure where sources could disagree
- [ ] No invented numbers — every figure traces back to a fetched source or `PORTFOLIO.md`
- [ ] No buy/sell recommendation, no market-timing claim, no tax guidance
- [ ] Report file written under `portfolio/reports/`
- [ ] State file updated (review only)
- [ ] Widget rendered (x-ray, etf-compare; optional for review)

## Example invocations

- `/portfolio` → show help
- `/portfolio help` → show help
- `/portfolio review` → periodic snapshot, compare against last
- `/portfolio x-ray` → deep structural analysis
- `/portfolio x-ray --xlsx` → same plus Excel export
- `/portfolio etf-compare IE00B4L5Y983 IE00BKM4GZ66` → compare two ETFs
- `/portfolio etf-compare IE00B4L5Y983 IE00BKM4GZ66 IE00BJ0KDQ92 --xlsx` → three ETFs plus Excel
