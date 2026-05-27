# ETF providers — URL patterns

The skill uses this file to locate provider-side factsheets, holdings lists, and KIIDs (the most authoritative source for ETF data).

Edit when a provider changes its URL scheme. Add new providers as needed.

---

## Quick list (issuers covered)

| Issuer | Brand | Primary domain |
|---|---|---|
| BlackRock | iShares | ishares.com |
| DWS | Xtrackers | etf.dws.com |
| Vanguard | Vanguard | vanguard.de / vanguard.com |
| Amundi | Amundi ETF / Lyxor | amundietf.de / amundietf.com |
| Invesco | Invesco | etf.invesco.com |
| State Street | SPDR | ssga.com / spdrs.com |
| HSBC | HSBC ETF | etf.hsbc.com |
| UBS | UBS ETF | ubs.com/etf |
| WisdomTree | WisdomTree | wisdomtree.eu |
| VanEck | VanEck | vaneck.com |
| Franklin Templeton | Franklin | franklintempleton.de |
| L&G | L&G ETF | lgim.com |

---

## URL pattern detail

For each issuer, the canonical "ETF detail page" follows a predictable pattern. The skill should construct the URL from the ISIN or use a provider site search if the slug isn't known.

### iShares (BlackRock)

- Detail page: `https://www.ishares.com/de/privatanleger/de/produkte/{PRODUCT_ID}/`
- The `PRODUCT_ID` is a numeric ID; the easiest path is a site search by ISIN:
  - Search URL: `https://www.ishares.com/de/privatanleger/de/produkte/etf-investments?siteEntryPassthrough=true&query={ISIN}`
- Factsheet PDF: linked on the detail page under "Dokumente" → "Factsheet"
- Holdings (CSV): linked on the detail page under "Portfolio" → "Bestände herunterladen"

### Xtrackers (DWS)

- Detail page: `https://etf.dws.com/de-de/{ISIN}-{slug}/`
- If slug unknown: `https://etf.dws.com/de-de/search/?search={ISIN}` and follow the first hit
- Factsheet and holdings linked under "Dokumente" / "Bestände"

### Vanguard

- DE detail page: `https://www.vanguard.de/professional/produkte/produkt/etf/{TICKER}/{slug}`
- Global detail page: `https://www.vanguard.com/`
- Search by ISIN: `https://www.vanguard.de/professional/produkte/list/etf?search={ISIN}`

### Amundi ETF (incl. former Lyxor)

- DE detail page: `https://www.amundietf.de/de/privatanleger/produkte/{slug}-{ISIN}`
- Search: `https://www.amundietf.de/de/privatanleger/produkte?search={ISIN}`
- Lyxor ETFs were migrated into the Amundi range in 2023–2024; their ISINs typically resolve to amundietf.de now.

### Invesco

- Detail page: `https://etf.invesco.com/de/private/products/{ISIN}/`
- Holdings: "Investments" tab on the detail page

### SPDR (State Street)

- Detail page: `https://www.ssga.com/de/de_ch/individual/etfs/funds/{slug}-{TICKER}`
- Search: `https://www.ssga.com/de/de_ch/individual/etfs/fund-finder?query={ISIN}`

### HSBC ETF

- Detail page: `https://www.etf.hsbc.com/etf/lu/individual/de/{slug}-{ISIN}`
- Search: `https://www.etf.hsbc.com/etf/lu/individual/de/products`

### UBS ETF

- Detail page: `https://www.ubs.com/de/de/assetmanagement/etf-and-index-fund-finder/products/{ISIN}.html`
- Search: `https://www.ubs.com/de/de/assetmanagement/etf-and-index-fund-finder.html`

### WisdomTree

- Detail page: `https://www.wisdomtree.eu/de-de/etfs/{category}/{ticker}`
- Search: `https://www.wisdomtree.eu/de-de/search?q={ISIN}`

### VanEck

- Detail page: `https://www.vaneck.com/de/de/{slug}-ucits-etf/`
- Search: `https://www.vaneck.com/de/de/etf-search/?q={ISIN}`

---

## ISIN to issuer heuristic

If you don't know the issuer from the ISIN, look at the prefix:

- `IE00...` — typically Ireland-domiciled, common for iShares, Vanguard Ireland, Xtrackers Ireland, Invesco Ireland
- `LU0... / LU1... / LU2...` — Luxembourg, common for Amundi, Xtrackers Luxembourg, HSBC, UBS, Lyxor (legacy)
- `DE000...` — Germany, less common for UCITS ETFs but used by some Xtrackers legacy products
- `FR00...` — France, Amundi/Lyxor legacy

The domicile prefix doesn't fully determine the issuer — always confirm via justETF or the provider search.

---

## When the provider site is paywalled, geo-blocked, or fails

Fall back order:

1. justETF (`https://www.justetf.com/en/etf-profile.html?isin={ISIN}`)
2. extraETF (`https://extraetf.com/de/etf-profile/{ISIN}`)
3. The KIID PDF — every UCITS ETF has one, often indexed by a web search for `KIID {ISIN}`

Report which source was used if it wasn't the primary one — keeps the report transparent.
