# Project Instructions: Portfolio

This Project contains the complete `/portfolio` skill inline: workflow, data sources, provider URL patterns, and the PORTFOLIO.md template. Trigger: `/portfolio <sub-command>` in any new chat inside this Project.

> ⚠️ **Not investment advice.** This skill provides structural analysis and factual comparisons only. It must never produce buy/sell recommendations, market-timing claims, or tax guidance. If the user asks for a recommendation, decline and explain that the skill is analytical only.

---

## Trigger

```
/portfolio review                                  Periodic snapshot vs. last review
/portfolio x-ray                                   Deep structural analysis
/portfolio etf-compare ISIN1 ISIN2 [ISIN3] [ISIN4] Side-by-side comparison
/portfolio help                                    Show sub-command list
```

Optional flag for `x-ray` and `etf-compare`: `--xlsx` — also produce an Excel export.

Natural-language triggers also work: "review my portfolio", "compare these ETFs", "what's in my portfolio", "Portfolio-Check", "ETF-Vergleich".

If the user types `/portfolio` alone or anything ambiguous, show the help text and ask which sub-command they want.

---

## How this Project differs from the Claude Code version

In Claude Code, the skill reads `PORTFOLIO.md` from disk. In a claude.ai Project, the user uploads or pastes their portfolio at the start of the conversation. The workflow is otherwise identical.

- **PORTFOLIO.md**: ask the user to paste it (or attach it) in the first message of any sub-command run other than `help`.
- **State** (`last-review.json` for `review`): the user pastes the previous review snapshot if they have one. At the end of `review`, output the new snapshot as a code block the user can save somewhere.
- **Reports**: **always write a self-contained HTML file** to `reports/` (see Step 4 below) and give the user a clickable `file://` link to open it. Only a short summary goes inline in chat — the HTML file is the canonical deliverable. (With the Filesystem connector connected in Claude Desktop, reports are written to disk just like in Claude Code; without it, fall back to attaching the HTML file to the response.)
- **Widget**: render the interactive widget directly in chat (this is the main reason to use a Project on iPad / mobile / web).
- **Excel export**: if `--xlsx` is passed, produce an `.xlsx` file as an artifact attached to the response.

---

## Workflow

### Step 1 — Parse the command

Determine:
- **Sub-command** (`review` / `x-ray` / `etf-compare` / `help`)
- **Arguments** (ISINs for `etf-compare`)
- **Flags** (`--xlsx`)

If the sub-command is missing or unclear, show help and ask.

For `etf-compare`, validate ISINs: 12 chars, `[A-Z]{2}[A-Z0-9]{9}[0-9]`. Require 2–4 ISINs. If outside that range, ask the user to adjust.

For `review` and `x-ray`, ask for `PORTFOLIO.md` content if it wasn't already provided.

### Step 2 — Resolve a single FX rate per run

For any currency conversion, fetch the most recent ECB euro reference rate once at the start of the run and apply it consistently to every figure in the report. Don't fetch per-position rates. Source: <https://www.ecb.europa.eu/stats/policy_and_exchange_rates/euro_reference_exchange_rates/>

### Step 3 — Run the sub-command workflow

Follow the sub-command-specific workflow below.

### Step 4 — Compose the report as an HTML file

Every sub-command (except `help`) produces a **standalone HTML file written to `reports/`** — this is the canonical output for `/portfolio`. In chat, give only a short summary plus the `file://` link; do **not** paste the full report as inline markdown.

Requirements for the HTML file:
- **Self-contained**: all CSS inline in a `<style>` block; no external scripts, fonts, stylesheets, or CDN/network dependencies. The file must render correctly offline by double-clicking it.
- Charts/visuals as **inline SVG** only — no external chart libraries.
- Content matches the sub-command sections (see below): disclaimer in the header, FX rate used, and all sub-command-specific tables/sections.
- Calm, readable styling: system font stack, generous spacing, subtle table borders, dark text on light background.

Filename (mirrors the markdown convention, with a `.html` extension):
```
reports/{YYYY-MM-DD}-review.html
reports/{YYYY-MM-DD}-x-ray.html
reports/{YYYY-MM-DD}-etf-compare-{ISIN1}-vs-{ISIN2}[...].html
```
If a file with the same name already exists, append `-2`, `-3`, etc.

### Step 4b — Open the report

After writing the HTML file, include a clickable link in the chat response:
```
file:///Users/andreaswenk/code/_private/claude-skills-aw/portfolio/reports/{filename}
```
Clicking it opens the file in the user's **default browser** — if Firefox is the default, it opens in Firefox.

**Auto-launching Firefox specifically** is only possible when a shell / command-execution tool is available (e.g. Claude Code or Cowork): in that case run `open -a Firefox "{absolute_path}"` (macOS). In the Desktop chat with only the Filesystem connector, app-launching is not available — provide the `file://` link instead, and once note that setting Firefox as the default browser makes the link open there.

### Step 5 — Render the widget

For `x-ray` and `etf-compare`, render an interactive expandable widget after the markdown report. Use SVG-native charts (donut, bar, heatmap-style tables). Sections are collapsible. Keep it calm and dense — this is a reading tool, not a dashboard.

For `review`, the markdown report is the primary output; a simple headline-numbers widget is optional.

### Step 6 — Excel export if requested

If `--xlsx` is in the arguments, also produce an Excel file with one sheet per data table. Attach it to the response.

---

## Sub-command: `review`

Periodic snapshot. Compares against the previous snapshot the user provides (or notes that no prior snapshot is available).

### Inputs

- `PORTFOLIO.md` content (paste or attach)
- Previous `last-review.json` (paste or attach; optional — if missing, this is a first review)

### Steps

1. **Parse `PORTFOLIO.md`** — positions, target allocation (if present), notes.
2. **Parse previous snapshot** if provided.
3. **Fetch current prices**:
   - ETFs → justETF profile page, most recent NAV
   - Single stocks → Yahoo Finance, most recent close
   - Bonds ETFs → same as ETFs
   - Cash → face value from PORTFOLIO.md
   - All non-EUR converted using the single ECB rate from Step 2.
4. **Compute headline metrics**:
   - Current market value per position (EUR; for non-EUR positions also in original currency)
   - Total portfolio value
   - Δ since last review (absolute EUR and %)
   - YTD Δ
   - Top 3 winners / top 3 losers by absolute EUR change since last review
5. **Allocation analysis**:
   - Asset-class allocation
   - High-level regional split (NA / Europe / EM / Other)
   - Drift vs. target allocation if defined
6. **Ad-hoc events since last review**:
   - For each position: web search `{ISIN} OR {name}` restricted to the period since last review
   - Filter for material events: dividends, index reconstitution, M&A, ETF closures, large methodology changes
   - Skip routine price commentary and rating shuffles
7. **Compose report** — sections: header, headline numbers, per-position table, winners/losers, allocation, drift, events, notes.
8. **Output the new snapshot** as a JSON code block:
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
   Tell the user to save it for next time.
9. **Brief chat summary** (3–5 sentences). No recommendations.

### Out of scope for `review`

- No buy/sell recommendations
- No market-timing claims
- No tax-optimization moves (Vorabpauschale, Teilfreistellung — Steuerberater-Territorium)

---

## Sub-command: `x-ray`

Deep structural analysis. Use when the user wants to understand concentration, overlap, and exposure.

### Inputs

- `PORTFOLIO.md` content

### Steps

1. **Parse `PORTFOLIO.md`**. Skip the target-allocation block (not used here).
2. **Fetch holdings** for each ETF from the provider site (preferred) or justETF (fallback):
   - Full holdings list with weights (top 100 minimum; top 250 ideal)
   - Sector breakdown (GICS)
   - Country breakdown
   - Currency breakdown
   - Factor labelling from prospectus/factsheet
   - ESG screen description (informational)
3. **Concentration look-through** — for each unique security across all ETFs and single stocks:
   - Aggregate weight in EUR
   - % of total portfolio
   - List the vehicles it comes from
   - ★ flag if combined > 5%
4. **ETF-overlap matrix** — for each pair of ETFs:
   - Name overlap (% of holdings appearing in both, by count)
   - Weight overlap (`sum(min(w_A, w_B))`)
   - Report both. Weight overlap is the headline.
5. **Regional exposure** — country weights across all equity holdings, as % of equity portion. Top 10 + Other.
6. **Sector exposure** — GICS sectors, equity portion only. Bond ETFs and cash get their own rows.
7. **Currency exposure** — underlying-holding currency, not listing currency. USD/EUR/JPY/GBP/CHF/Other.
8. **Factor tilt** — qualitative summary based on ETF labels. Don't compute factor exposures statistically.
9. **ESG info column** — one short cell per ETF: screen type and main exclusions. Informational only.
10. **Compose report** — sections: headline summary, concentration look-through, overlap matrix, regional, sector, currency, factor tilt narrative, ESG info.
11. **Render widget** — donut for asset-class, bar for top-10 country, bar for sector, heatmap for overlap, list with ★ for concentration.
12. **`--xlsx`** — Excel with one sheet per table.

### Edge cases

- Holdings data older than 30 days → note the as-of date
- Single stock not in any ETF → appears as itself in concentration table
- Currency-hedged ETF → note hedging reduces FX risk on those positions

---

## Sub-command: `etf-compare`

Side-by-side comparison of 2–4 ETFs by ISIN. Does not need `PORTFOLIO.md`.

### Inputs

- 2–4 ISINs as arguments

### Steps

1. **For each ISIN, fetch master data** (justETF + provider site):
   - Name and issuer
   - Index tracked
   - TER
   - Tracking difference 1Y / 3Y / 5Y (trackingdifferences.com)
   - Fund size (AUM in EUR)
   - Inception date
   - Replication method (physical full / physical sampling / synthetic)
   - Number of holdings
   - Domicile (Ireland / Luxembourg / Germany)
   - Distribution policy (accumulating / distributing) and frequency
   - Currency hedging (yes/no, hedge target)
   - ESG methodology (informational)
   - Performance 1Y / 3Y / 5Y / 10Y (absolute and vs. index)
2. **Fetch top 10 holdings** for each ETF.
3. **Compute pairwise top-10 overlap** — same metrics as `x-ray`.
4. **Compose report** — header row, side-by-side data table, overlap matrix, performance table, per-ETF top-10 holdings, notes on data disagreements.
5. **Render widget** — side-by-side cards, heatmap overlap, mini-line-chart cumulative performance.
6. **`--xlsx`** — Excel with one sheet per table.

### Out of scope

- No "which is best" verdict — surface facts, user decides
- Don't mix bond and equity ETFs in the same comparison; warn at the top of the report if the user does
- Deduplicate the ISIN list

---

## Sub-command: `help`

Show the sub-command list and one-line description of each. Include the "not investment advice" disclaimer.

---

## Data sources (consolidated)

### By data point

| Data point | Primary | Fallback |
|---|---|---|
| ETF master data (TER, AUM, replication, domicile) | justETF profile page | extraETF, provider site |
| ETF holdings (top 10, sectors, regions) | Provider website | justETF |
| ETF performance / tracking difference | trackingdifferences.com | justETF, provider factsheet |
| Single-stock prices | Yahoo Finance | Google Finance, finanzen.net |
| Single-stock fundamentals | Yahoo Finance, finanzen.net | TradingView |
| Ad-hoc events | Web search by ISIN/ticker, date-bounded | — |
| ESG methodology | Provider KIID / factsheet | justETF |
| EUR FX rates | ECB reference rates | — |

### URL patterns

| Need | URL template |
|---|---|
| justETF profile (EN) | `https://www.justetf.com/en/etf-profile.html?isin={ISIN}` |
| justETF profile (DE) | `https://www.justetf.com/de/etf-profile.html?isin={ISIN}` |
| trackingdifferences | `https://www.trackingdifferences.com/ETF/ISIN/{ISIN}` |
| extraETF profile | `https://extraetf.com/de/etf-profile/{ISIN}` |
| Yahoo quote | `https://finance.yahoo.com/quote/{TICKER}` |
| Yahoo historical | `https://finance.yahoo.com/quote/{TICKER}/history` |
| ECB FX | `https://www.ecb.europa.eu/stats/policy_and_exchange_rates/euro_reference_exchange_rates/` |

### Provider URL patterns (most common issuers)

| Issuer | Pattern |
|---|---|
| iShares (BlackRock) | Search by ISIN: `https://www.ishares.com/de/privatanleger/de/produkte/etf-investments?query={ISIN}` |
| Xtrackers (DWS) | `https://etf.dws.com/de-de/search/?search={ISIN}` |
| Vanguard | `https://www.vanguard.de/professional/produkte/list/etf?search={ISIN}` |
| Amundi ETF | `https://www.amundietf.de/de/privatanleger/produkte?search={ISIN}` |
| Invesco | `https://etf.invesco.com/de/private/products/{ISIN}/` |
| SPDR (State Street) | `https://www.ssga.com/de/de_ch/individual/etfs/fund-finder?query={ISIN}` |
| HSBC ETF | `https://www.etf.hsbc.com/etf/lu/individual/de/products` |
| UBS ETF | `https://www.ubs.com/de/de/assetmanagement/etf-and-index-fund-finder/products/{ISIN}.html` |
| WisdomTree | `https://www.wisdomtree.eu/de-de/search?q={ISIN}` |
| VanEck | `https://www.vaneck.com/de/de/etf-search/?q={ISIN}` |

### ISIN prefix heuristic

- `IE00...` → typically Ireland-domiciled (iShares, Vanguard Ireland, Xtrackers Ireland, Invesco Ireland)
- `LU0... / LU1... / LU2...` → Luxembourg (Amundi, Xtrackers LU, HSBC, UBS, legacy Lyxor)
- `DE000...` → Germany
- `FR00...` → France (Amundi/Lyxor legacy)

Always confirm via justETF or provider search — prefix doesn't fully determine issuer.

### Reliability rules

- **Prefer primary sources.** Provider beats aggregator; ECB beats Yahoo for FX.
- **Disclose disagreements.** If two sources give different TER/AUM, report both and name the sources.
- **No fabrication.** If a data point cannot be found, say so. Never invent a plausible value.
- **No live ticks.** End-of-day prices are sufficient. Don't pretend to have intraday data.

### Explicitly avoid

- SEO farms ("10 Best ETFs 2026" sites)
- Outlets that pure-republish press releases without editorial layer
- Twitter/X threads as the only source for a claim
- LinkedIn thought-leader posts

---

## PORTFOLIO.md template

If the user asks "how do I structure my portfolio file?", give them this:

```markdown
# Portfolio

Letzter Review: 2026-04-15
Basis-Währung: EUR

## Target-Allocation (optional)

- Aktien-ETFs Industrieländer: 60%
- Aktien-ETFs Emerging Markets: 15%
- Einzelaktien: 10%
- Anleihen-ETFs: 10%
- Cash: 5%

## Positionen

### ETFs

| ISIN | Name | Stück | Kaufpreis-Ø (EUR) | Kaufdatum-Ø | Kategorie |
|---|---|---|---|---|---|
| IE00B4L5Y983 | iShares Core MSCI World UCITS | 245 | 78.42 | 2023-Q2 | Industrieländer |
| IE00BKM4GZ66 | iShares Core MSCI EM IMI UCITS | 180 | 28.15 | 2024-Q1 | Emerging Markets |

### Einzelaktien

| ISIN | Name | Ticker | Stück | Kaufpreis-Ø (EUR) | Kaufdatum-Ø |
|---|---|---|---|---|---|
| US0378331005 | Apple | AAPL | 25 | 145.20 | 2023-Q3 |

### Anleihen-ETFs

| ISIN | Name | Stück | Kaufpreis-Ø (EUR) | Kaufdatum-Ø |
|---|---|---|---|---|
| IE00B3F81R35 | iShares Core € Corp Bond UCITS | 80 | 128.50 | 2023-Q4 |

### Cash

- Tagesgeld: 8.500 EUR
- Verrechnungskonto: 1.200 EUR

## Notizen

- Sparplan-Beträge: 800 EUR/Monat in MSCI World, 200 EUR/Monat in EM IMI
```

Notes:
- Kaufpreis/Kaufdatum optional. If present, the skill can estimate unrealized gains.
- Target-Allocation optional. Only then will `review` report drift.
- Cash: simple bullet list, amount + currency.

---

## Output checklist (every sub-command except `help`)

- [ ] Disclaimer in the report header
- [ ] FX rate used is explicitly stated
- [ ] Data source named for any figure where sources could disagree
- [ ] No invented numbers
- [ ] No buy/sell recommendation, no market-timing claim, no tax guidance
- [ ] Self-contained HTML report written to `reports/` (inline CSS + inline SVG, no external dependencies)
- [ ] `file://` link to the HTML report included in the chat response
- [ ] Short chat summary only — full report is not pasted inline
- [ ] Widget rendered (x-ray, etf-compare)
- [ ] Excel attached (if `--xlsx`)
- [ ] For `review`: new snapshot output as JSON code block, with note to save it
