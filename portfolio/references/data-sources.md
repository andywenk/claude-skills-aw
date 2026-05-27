# Data sources for `/portfolio`

This file lists the web sources the skill should consult for each kind of data point. Edit freely — add, remove, or reorder. The skill reads this file at the start of every run.

Tiers below are preference order, not hard rules. If a higher-tier source fails or doesn't have the data, fall back to the next.

---

## By data point

### ETF master data (TER, AUM, replication, domicile, distribution policy)

1. **justETF** — https://www.justetf.com/en/etf-profile.html?isin={ISIN}
2. **extraETF** — https://extraetf.com/de/etf-profile/{ISIN}
3. **Provider website** (see `etf-providers.md` for URL patterns)

### ETF holdings (top 10, sector breakdown, regional breakdown)

1. **Provider website** — most up to date and authoritative (monthly factsheets)
2. **justETF** — has a holdings tab but may lag the provider by a few weeks

### ETF performance and tracking difference

1. **trackingdifferences.com** — https://www.trackingdifferences.com/ETF/ISIN/{ISIN}
2. **justETF** — performance chart on the ETF profile page
3. **Provider factsheet** — has 1Y/3Y/5Y figures

### Single-stock prices and fundamentals

1. **Yahoo Finance** — https://finance.yahoo.com/quote/{TICKER}
2. **finanzen.net** — https://www.finanzen.net/aktien/{name}-aktie (DE-focused, EUR-denominated)
3. **Google Finance** — https://www.google.com/finance/quote/{TICKER}:{EXCHANGE}

### Ad-hoc events (last 30 days)

- Web search by ISIN/ticker + name, restricted to the last 30 days
- Examples: dividend announcements, index reconstitution, ETF closure, M&A
- For German news prefer: handelsblatt.com, finanzen.net, justETF news section

### ESG methodology

1. **Provider website** — KIID / factsheet describes ESG screen and exclusions
2. **justETF** — ESG tab shows screened categories
3. **MSCI / Morningstar ESG profile pages** — if linked from the above

### EUR / FX rates

- **European Central Bank reference rates** — https://www.ecb.europa.eu/stats/policy_and_exchange_rates/euro_reference_exchange_rates/
- Use the most recent published daily rate. Same rate applied consistently within a single skill run.

---

## Reliability rules

- **Prefer primary sources.** Provider website beats aggregator; ECB beats Yahoo for FX.
- **Disclose disagreements.** If two sources give different TER/AUM figures, report both and name the sources rather than silently picking one.
- **No fabrication.** If a data point cannot be found, say so. Never invent a plausible value.
- **No live ticks.** End-of-day prices are sufficient. Don't pretend to have intraday data.

---

## Detail appendix — query patterns

These are the exact URL or search shapes to use when fetching. Update when a provider changes its URL scheme.

| Need | URL template | Notes |
|---|---|---|
| justETF profile | `https://www.justetf.com/en/etf-profile.html?isin={ISIN}` | German variant: `/de/etf-profile.html`. Profile page has TER, AUM, replication, domicile, distribution, top-10. |
| trackingdifferences | `https://www.trackingdifferences.com/ETF/ISIN/{ISIN}` | Shows 1Y/3Y/5Y tracking diff vs index. May 404 for smaller ETFs. |
| extraETF profile | `https://extraetf.com/de/etf-profile/{ISIN}` | Useful fallback when justETF is incomplete. |
| Yahoo quote | `https://finance.yahoo.com/quote/{TICKER}` | Use exchange suffix for non-US: `SAP.DE`, `7203.T`, `ASML.AS`. |
| Yahoo historical | `https://finance.yahoo.com/quote/{TICKER}/history` | For period returns. |
| ECB FX | `https://www.ecb.europa.eu/stats/policy_and_exchange_rates/euro_reference_exchange_rates/html/{ccy}.xml` | XML; the most recent `<Obs>` is today's reference rate. |
| Web search for events | `"{ISIN}" OR "{name}"` + last 30 days | Use the WebSearch tool with a date-bounded query. |

---

## Notes for future edits

- If a provider changes URL scheme, update the table above and `etf-providers.md`.
- If you add a new ETF data aggregator (e.g. a regional one), add it as a secondary fallback under the relevant data point.
- The skill never depends on a single source — every category should have at least one fallback.
