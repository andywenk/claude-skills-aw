# Portfolio Skill — Konzept

Dieses Dokument beschreibt den geplanten `/portfolio`-Skill. Es ist die Design-Grundlage für die spätere Implementierung (SKILL.md, project-instructions.md, etc.) und wird vor dem Build committed, damit Design-Entscheidungen versioniert sind.

> ⚠️ **Wichtig:** Dieser Skill liefert strukturelle Analysen und faktische Vergleiche. Er gibt **keine Anlageberatung, keine Handlungsempfehlungen, keine Kauf-/Verkaufs-Signale**. Claude ist kein Finanzberater. Der Skill bereitet Daten so auf, dass du selbst entscheiden kannst.

---

## Ziel

Drei Aktien-/ETF-Analyse-Workflows, gebündelt in einem Skill mit Sub-Commands:

```
/portfolio review        → Periodische Bestandsaufnahme
/portfolio x-ray         → Tiefen-Analyse (Konzentration, Overlap, Allokation)
/portfolio etf-compare   → Vergleich von 2-4 ETFs Seite an Seite
/portfolio help          → Übersicht der Sub-Commands
```

Möglich zukünftig (Phase 2):
```
/portfolio rebalance     → Drift gegen Target-Allocation
/portfolio dividend      → Cashflow-Übersicht
```

---

## Trigger und Sub-Commands

### `/portfolio review` — *"Wie geht's meinem Portfolio gerade?"*

**Use case:** Periodische Bestandsaufnahme, z.B. monatlich.

**Input:** `PORTFOLIO.md` (siehe unten).

**Output:**
- Aktueller Marktwert je Position und gesamt
- Performance seit letztem Review (vom Skill-State protokolliert) und seit Jahresanfang
- Top-Gewinner / Top-Verlierer seit letztem Review
- Asset-Klassen-Allokation (Aktien-ETFs / Einzelaktien / Anleihen / Cash / sonstiges)
- Regionale Verteilung auf High-Level (Nordamerika / Europa / Emerging Markets / Sonstige)
- Größere Ad-hoc-Events seit letztem Review (Dividenden, Index-Änderungen, Übernahme-News, ETF-Auflösungen) — via Web-Suche pro Position
- Drift gegen Target-Allocation (wenn definiert in PORTFOLIO.md)

### `/portfolio x-ray` — *"Was steckt eigentlich drin?"*

**Use case:** Tiefen-Analyse, einmalig oder bei größeren Portfolio-Änderungen.

**Input:** `PORTFOLIO.md`.

**Output:**
- **Konzentrations-Look-Through**: Welche Einzelaktien hast du durch alle ETFs hinweg? Beispiel: Apple in MSCI World, S&P 500, Nasdaq 100 — Gesamt-Apple-Exposure als % des Portfolios
- **ETF-Overlap-Matrix**: Paar-für-Paar Überlappung in % der Holdings
- **Regional-Exposure** detailliert: pro Land (USA, Japan, Deutschland, …) als % des Aktien-Anteils
- **Sektor-Exposure**: nach GICS-Sektoren (Tech, Healthcare, Financials, …)
- **Währungs-Exposure**: USD/EUR/JPY/GBP/Sonstige — wichtig für Wechselkursrisiken
- **Faktor-Tilt**: Value/Growth/Size (Large/Mid/Small Cap), Quality, Momentum — auf Basis öffentlicher ETF-Daten
- **Klumpenrisiken**: Top-10-Einzelpositionen durchgereicht, mit %-Anteil am Portfolio
- **ESG-Info-Spalte** (kein Score, keine Bewertung): Pro Position ESG-Methodologie des ETF / SRI-Variante / Ausschlüsse — nur informativ

### `/portfolio etf-compare ISIN1 ISIN2 [ISIN3] [ISIN4]` — *"Welcher ETF von diesen?"*

**Use case:** Konkrete Entscheidung zwischen ähnlichen ETFs.

**Input:** 2-4 ISINs als Argumente. Kein PORTFOLIO.md nötig.

**Output:**
- TER (Total Expense Ratio) und tatsächliche Tracking-Difference letzte 1/3/5 Jahre
- Fondsgröße (Fondsvolumen) und Auflagedatum
- Replikationsmethode (physisch voll / physisch Sampling / synthetisch)
- Index und Anzahl Holdings
- Top-10-Holdings-Overlap zwischen den ETFs (paarweise)
- Domizil (Irland / Luxemburg / Deutschland) — relevant für Quellensteuer
- Ausschüttungsart (thesaurierend / ausschüttend) und Ausschüttungsfrequenz
- Currency-Hedging (ja/nein, Hedging-Zielwährung)
- ESG-Methodologie / SRI-Variante / Ausschlüsse — als Info-Spalte
- Performance-Historie (1/3/5/10 Jahre, absolute Rendite und vs. Index)

### `/portfolio help`

Listet die Sub-Commands mit kurzem Hinweis. Auch bei `/portfolio` ohne Argument.

---

## Datenquellen

Mix aus Web-Quellen, gewählt nach passendster Quelle pro Frage. Reihenfolge der Bevorzugung:

| Datenpunkt | Primärquelle | Fallback |
|---|---|---|
| ETF-Stammdaten (TER, AUM, Replikation, Domizil) | justETF.com | extraETF.com, Anbieter-Website |
| ETF-Holdings (Top 10, Sektoren, Regionen) | Anbieter-Website (iShares, Xtrackers, Vanguard, Amundi, Lyxor) | justETF |
| ETF-Performance / Tracking-Difference | trackingdifferences.com, justETF | Anbieter |
| Einzelaktien-Kurse | Yahoo Finance | Google Finance |
| Aktien-Fundamentaldaten | Yahoo Finance, finanzen.net | TradingView |
| Ad-hoc-News pro Position | Web-Suche nach ISIN/Ticker + letzte 30 Tage | — |
| ESG-Methodologie | Anbieter-Website (im KIID/Factsheet beschrieben) | justETF |

**Wichtig:** Keine Live-Tick-Daten. Alle Kurse sind Tagesschluss-Kurse oder leicht verzögert. Für Long-Term-Investing reicht das.

**Zuverlässigkeits-Notiz:** Web-gescrapte Daten sind nicht perfekt. Bei Diskrepanzen zwischen Quellen sagt der Skill das offen und nennt die unterschiedlichen Werte, statt einen scheinbar präzisen Wert zu erfinden.

---

## PORTFOLIO.md — Format

Eine vom Nutzer gepflegte Markdown-Datei. Beispiel:

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
| IE00BJ0KDQ92 | Xtrackers MSCI World ESG UCITS | 120 | 45.30 | 2024-Q3 | Industrieländer ESG |

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

- Speku-Positionen (max 5% gesamt): keine aktuell
- Geplante Sparplan-Beträge: 800 EUR/Monat in MSCI World, 200 EUR/Monat in EM IMI
```

**Bewusste Designentscheidungen:**
- **Markdown-Tabellen** statt CSV — bleibt lesbar und Git-diff-freundlich
- **Kaufpreis und Datum** optional — wenn vorhanden, kann der Skill Realized-/Unrealized-Gains schätzen
- **Target-Allocation optional** — nur dann gibt der Skill Drift-Hinweise
- **Notizen-Sektion** — für freie Anmerkungen, die der Skill bei `review` aufgreift
- **Basis-Währung EUR** für die zusammengefasste Sicht — Einzelpositionen werden zusätzlich in Original-Währung angezeigt (z.B. AAPL in USD und EUR)

---

## Output-Formate

### Markdown-Report (Standard für alle Sub-Commands)

Wird **immer** erzeugt, in `portfolio/reports/` mit Zeitstempel:

```
portfolio/reports/2026-05-27-review.md
portfolio/reports/2026-05-27-x-ray.md
portfolio/reports/2026-05-27-etf-compare-IE00B4L5Y983-vs-IE00BKM4GZ66.md
```

Vorteil: Git-versionierbar. Du kannst sehen, wie sich dein Portfolio über Zeit verändert hat, indem du `git log portfolio/reports/` machst.

**Reports-Verzeichnis ist gitignored** (`portfolio/reports/`) — enthält private Daten. Wer Reports versionieren will, entfernt das aus `.gitignore` und committet in ein privates Repo.

### Interaktives Widget (Standard für `x-ray` und `etf-compare`)

Inline im Chat:
- **x-ray**: aufklappbare Sektionen (Konzentration / Overlap / Regional / Sektor / Währung / Faktor), Donut-Charts für Allokation, Heatmap-Tabelle für Overlap, ★-Markierung bei Klumpenrisiken (>5% einer Einzelaktie)
- **etf-compare**: Side-by-side-Tabelle, Heatmap für Holdings-Overlap, Mini-Charts für Performance-Historie

### Excel-Export (optional, via `--xlsx`)

```
/portfolio x-ray --xlsx
/portfolio etf-compare IE00... IE00... --xlsx
```

Output: Excel-Datei mit allen Tabellen, eine pro Sheet. Für Archivierung, Weiterverarbeitung, Teilen mit Steuerberater.

---

## Skill-State

Der Skill braucht Zustand für `review`-Vergleiche ("Performance seit letztem Review"). Lebt in:

```
portfolio/state/last-review.json
```

Inhalt:
```json
{
  "last_review_date": "2026-04-15",
  "snapshot": {
    "IE00B4L5Y983": { "shares": 245, "price_eur": 89.30, "value_eur": 21878.50 },
    ...
  },
  "total_value_eur": 47230.00
}
```

State-Verzeichnis ist gitignored — enthält private Werte.

`review` liest den letzten State, vergleicht gegen aktuelle Werte, schreibt neuen State.

---

## Repository-Struktur (geplant)

```
portfolio/
├── README.md                       (Skill-Übersicht, Setup)
├── SKILL.md                        (für Claude Code)
├── project-instructions.md         (für claude.ai Projects)
├── CONCEPT.md                      (dieses Dokument)
├── PORTFOLIO.md.example            (Template, committed)
├── PORTFOLIO.md                    (deine echten Daten, gitignored)
├── references/
│   ├── data-sources.md             (welche Quelle für welchen Datenpunkt)
│   └── etf-providers.md            (URL-Patterns der Anbieter-Websites)
├── reports/                        (gitignored, alle erzeugten Reports)
│   └── .gitkeep
└── state/                          (gitignored, Skill-State)
    └── .gitkeep
```

`.gitignore` Ergänzung:
```
portfolio/PORTFOLIO.md
portfolio/reports/*
!portfolio/reports/.gitkeep
portfolio/state/*
!portfolio/state/.gitkeep
```

---

## Abgrenzungen

**Was der Skill nicht macht:**
- ❌ **Keine Handlungsempfehlungen** ("Kauf X, verkauf Y")
- ❌ **Keine Markt-Timing-Aussagen** ("Jetzt einsteigen", "Korrektur droht")
- ❌ **Keine steuerlichen Optimierungsvorschläge** (Steuerberater-Job)
- ❌ **Keine Crypto-Coverage** (zu viele Edge-Cases bei Steuer/Verwahrung — separater Skill, wenn überhaupt)
- ❌ **Keine Echtzeit-Kurse** (Tagesschluss reicht für Long-Term)
- ❌ **Keine Broker-Anbindung** (manuelle Pflege der PORTFOLIO.md)

**Was der Skill macht:**
- ✅ Strukturelle Analyse: Konzentration, Overlap, Exposure
- ✅ Faktische Vergleiche: TER, Replikation, Performance-Historie
- ✅ Drift-Berechnung gegen selbst-definierte Target-Allocation
- ✅ Event-Aggregation: Was ist seit letztem Review passiert
- ✅ Excel-Export für Weiterverarbeitung

---

## Entschiedene Designfragen

Diese Punkte waren vor der Implementierung offen und wurden bewusst entschieden:

1. **Wechselkurs-Konvertierung** ✅ entschieden: Werte werden **sowohl in EUR als auch in Original-Währung** ausgegeben. Tagesschluss-Kurs, einheitlich pro Report-Lauf. Beispiel: `AAPL: 25 × $182.30 = $4.557,50 (≈ 4.221 EUR @ 1.0800 EUR/USD)`.

2. **Steuerliche Aspekte** ✅ entschieden: **Bleiben raus**. Vorabpauschale, Teilfreistellung, Quellensteuer — Steuerberater-Territorium. Skill bleibt rein analytisch.

3. **Report-Granularität** ✅ entschieden: **Immer Markdown-Report** bei jedem Lauf, in `portfolio/reports/` mit Zeitstempel. Verzeichnis ist gitignored.

4. **Charts im Widget** ✅ entschieden: **SVG-native**, wie bei ai-news. Konsistente Optik, keine zusätzliche Abhängigkeit.

5. **Multi-Portfolio-Support** ✅ entschieden: **Nur ein Portfolio**. Eine PORTFOLIO.md, kein Multi-Strategy-Support.

---

## Implementierungs-Reihenfolge (Vorschlag)

Wenn wir das Konzept abnicken, würde ich in dieser Reihenfolge bauen:

1. `PORTFOLIO.md.example` mit Beispieldaten, `.gitignore`-Updates
2. `references/data-sources.md` und `references/etf-providers.md`
3. `SKILL.md` mit Sub-Command-Routing
4. `etf-compare` als erstes — am isoliertesten, kein State nötig, einfach zu testen
5. `x-ray` — kein State, aber komplexere Datenaggregation
6. `review` — komplettes Setup inkl. State-Management
7. `project-instructions.md` (einheitliche Single-File-Version für claude.ai Projects)
8. `README.md` für die Portfolio-Skill-Doku

---

## Status

Alle Designfragen sind entschieden. Konzept ist freigegeben für Implementierung.

Implementierungs-Reihenfolge (siehe oben):
1. `PORTFOLIO.md.example` + `.gitignore`-Updates
2. `references/data-sources.md`, `references/etf-providers.md`
3. `SKILL.md` mit Sub-Command-Routing
4. `etf-compare` (isoliert, kein State)
5. `x-ray` (komplexe Aggregation)
6. `review` (mit State-Management)
7. `project-instructions.md` (claude.ai-Version)
8. `README.md`