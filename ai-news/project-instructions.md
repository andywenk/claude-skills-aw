# Project Instructions: AI-News

Dieses Project enthält einen kompletten AI-News-Skill in den Instructions: Workflow, Quellenliste, Fokus-Themen und Glossar. Trigger: `/ai-news` in jedem neuen Chat innerhalb dieses Projects.

---

## Trigger

- `/ai-news` — Standard: letzte 7 Tage, 10–15 Meldungen, alle Kategorien
- `/ai-news 3d` oder `/ai-news 14d` — anderes Zeitfenster
- `/ai-news 20 policy` — andere Anzahl und/oder Themenfilter
- Natürliche Sprache: "AI-News der Woche", "was ist in KI passiert", "KI-Digest"

---

## Workflow

### Schritt 1 — Parameter parsen

Bestimme aus der Nachricht:
- **Zeitfenster** (Standard: letzte 7 Tage)
- **Themenfilter** (optional, z.B. "policy", "alignment", "EU")
- **Anzahl Meldungen** (Standard: 10–15)

Heutiges Datum: nutze das tatsächliche aktuelle Datum, nicht ein gespeichertes Jahr.

### Schritt 2 — Web-Suchen über alle vier Kategorien

Führe mehrere kurze Suchen durch (3–6 Wörter pro Query), eine pro Kategorie:

1. **Research** — Papers, technische Reports, Alignment/Safety-Forschung
2. **Industry** — Modell-Releases, große Deals, Akquisitionen, Führungswechsel (nur wenn substantiell)
3. **Policy** — EU AI Act, deutsche Regulierung (KI-Bundesverband, BSI), internationale KI-Governance
4. **Tools** — Agent-Frameworks, Dev-Tools, Infra, Open-Source-Releases

Beispielqueries:
- `Anthropic OpenAI DeepMind announcement <last week>`
- `AI safety alignment research <last week>`
- `EU AI Act <current month>`
- `AI agent framework release <last week>`
- `arXiv AI paper discussed <last week>`

Für deutsche/EU-Policy auch auf Deutsch suchen: `KI-Verordnung`, `BSI KI`, `EU AI Act Umsetzung`, `BNetzA KI`.

### Schritt 3 — Filtern, deduplizieren, Fokus-Gewichtung

**Hart filtern** (immer raus):
- **Hype/Clickbait** — wenn die Headline der ganze Inhalt ist, weg.
- **Reine Produktlaunches ohne Substanz** — "X-Startup launcht Y-KI-Feature" ohne Architektur, Benchmark oder echte News.
- **Beginner-Content** — "Was ist ein LLM?", "Wie nutze ich ChatGPT?". Aus.
- **Doppelte Stories** — wenn 5 Outlets dasselbe abdecken, das beste Einzelstück nehmen (meist Primärquelle wie Lab-Blog > Presse-Recap).

**Im Zweifel:** Würde Simon Willison oder Zvi Mowshowitz dem einen Absatz widmen? Wenn nein, überspringen.

**Fokus-Gewichtung** (weich, nicht ausschließend):
- Für jede verbleibende Meldung: prüfe, ob ein Fokus-Keyword (Liste unten) in Headline, Zusammenfassung oder "Warum es wichtig ist" auftaucht (case-insensitive Substring-Match).
- Treffer bekommen ein `★ Fokus`-Badge im Widget und werden in der Sortierung nach oben gezogen.
- Sortierung: Fokus-Treffer zuerst (Wichtigkeit, dann Aktualität); Nicht-Treffer dahinter (gleiche interne Sortierung). Nicht-Treffer bleiben im Digest.
- Wenn >80% der Items Fokus-Treffer sind: Fokus-Liste ist evtl. zu breit. Wenn 0% Treffer: zu eng. In beiden Fällen unauffällig am Ende der Antwort vermerken.

### Schritt 4 — Digest schreiben

Pro Meldung:
- **Headline** (kurz, in Quellsprache — Englisch oder Deutsch)
- **Eine Absatz-Zusammenfassung auf Deutsch** (2–4 Sätze, faktisch, kein Marketing)
- **"Warum es wichtig ist"** — ein kurzer Satz, kalibriert für einen erfahrenen technischen Leser im KI-/Versicherungs-Umfeld
- **Quellenlink** (Primärquelle bevorzugt)
- **Kategorie-Tag** (Research / Industry / Policy / Tools)
- **Datum** (wann die Story brach)
- **Fokus-Flag** (aus Schritt 3)

Tonalität auf Deutsch: präzise, keine Marketing-Sprache, native deutsche Formulierungen (nicht aus dem Englischen übersetzt-klingend).

### Schritt 5 — Glossar-Begriffe identifizieren

Scanne alle Zusammenfassungen und "Warum es wichtig ist"-Felder auf Fachbegriffe und Abkürzungen. Ein Begriff qualifiziert, wenn:

- Abkürzung in GROSSBUCHSTABEN (BSI, MCP, GPAI, HRAIS, DSGVO, etc.) — 2 bis 6 Buchstaben, evtl. mit Zahlen
- Eigenname für Produkt, Projekt, Framework, Gesetz oder System (Antigravity, Project Glasswing, KI-VO, Colossus 1, Agent SDK)
- Domain-spezifischer Compound-Term (Hochrisiko-KI-System, Marktüberwachungsbehörde)

**Überspringe** generische Wörter ("Modell", "Unternehmen", "Forschung") und sehr verbreitete Begriffe ("KI", "AI", "LLM" — außer der User hat sie explizit ins Glossar aufgenommen).

Für jeden gefundenen Begriff: prüfe, ob er im **eingebetteten Glossar weiter unten** schon existiert. Wenn nein, formuliere eine neue Definition (1–3 Sätze, Deutsch, faktisch). Diese neuen Einträge werden am Ende der Antwort, **nach dem Widget**, in einem Markdown-Codeblock ausgegeben, damit der User sie manuell ins Glossar-Sektion dieser Project Instructions kopieren kann.

**Wichtig (iPad-spezifisch):** Da der Skill keine Dateien schreiben kann, ist der Glossar-Update-Schritt manuell. Format des Ausgabe-Blocks:

```
## Neue Glossar-Einträge (bitte oben unter "Glossar" einfügen)

### NEUER_BEGRIFF
Definition.

### NÄCHSTER_BEGRIFF
Definition.
```

Nur **neue** Einträge zeigen, keine bereits vorhandenen.

### Schritt 6 — Widget rendern

Nutze ein Markdown-/HTML-Artifact (über die Artifacts-Funktion in Claude.ai), oder falls Artifacts nicht verfügbar, eine strukturierte Markdown-Antwort mit aufklappbaren `<details>`-Tags.

**Bevorzugt: HTML-Artifact** mit folgender Struktur:
- Header mit Zeitfenster (z.B. "AI-Digest · 15.–22. Mai 2026") und Item-Anzahl
- Kategorie-Filter-Chips (Alle / Research / Industry / Policy / Tools)
- Karten pro Meldung, sortiert mit Fokus-Treffern zuerst, ★-Fokus-Badge auf Treffern
- Karte aufklappen → deutsche Zusammenfassung + "Warum es wichtig ist" + Quellenlink
- Klickbare Glossar-Begriffe: gepunktet unterstrichen (`border-bottom: 1px dotted`), Klick öffnet kleines Popover mit Definition direkt unter dem Begriff
- Visuell ruhig und dicht — Lese-Werkzeug, kein Dashboard

**Fallback: Markdown** mit `<details>`-Tags pro Meldung, ★-Emoji-Badge für Fokus-Treffer, Glossar-Begriffe als normale Links zu Definitionen am Seitenende.

---

## Output-Checkliste

Vor dem Anzeigen prüfen:

- [ ] 10–15 Meldungen (oder vom User angefordert), außer die Woche war wirklich ruhig
- [ ] Alle vier Kategorien vertreten (außer eine war leer)
- [ ] Keine Duplikate
- [ ] Keine Beginner-Inhalte, kein Hype, keine inhaltslosen Produktlaunches
- [ ] Jede Meldung hat eine "Warum es wichtig ist"-Zeile
- [ ] Jede Meldung hat einen funktionierenden Quellenlink
- [ ] Zusammenfassungen auf Deutsch, Headlines in Quellsprache
- [ ] Fokus-Treffer oben sortiert mit ★-Badge
- [ ] Erkannte Glossar-Begriffe im Widget verlinkt/markiert
- [ ] Neue Glossar-Einträge (falls vorhanden) als Markdown-Block nach dem Widget ausgegeben

---

## Quellen (priorisiert)

### Tier 1 — Lab-Blogs (Primärquellen)
- anthropic.com/news
- openai.com/blog
- deepmind.google/discover/blog
- ai.meta.com/blog
- microsoft.com/research/blog
- research.google/blog
- mistral.ai/news

### Tier 2 — Unabhängige Analysten
- simonwillison.net — Simon Willison, fast täglich
- thezvi.substack.com — Zvi Mowshowitz, wöchentliche Roundups
- gwern.net — Gwern, gelegentlich aber tief
- interconnects.ai — Nathan Lambert, Post-Training/RL
- importai.substack.com — Jack Clark, wöchentlicher Newsletter
- aisnakeoil.com — Narayanan & Kapoor, skeptische Sicht

### Tier 3 — Research-Aggregatoren
- arxiv.org/list/cs.AI/recent
- arxiv.org/list/cs.LG/recent
- arxiv.org/list/cs.CL/recent
- huggingface.co/papers
- paperswithcode.com

### Tier 4 — EU / Deutsche Policy
- ki-verband.de — KI-Bundesverband
- bsi.bund.de
- artificialintelligenceact.eu — EU AI Act Tracker
- digital-strategy.ec.europa.eu
- heise.de/thema/Kuenstliche-Intelligenz
- tagesschau.de/thema/kuenstliche_intelligenz
- netzpolitik.org

### Tier 5 — Tech-Presse (sparsam nutzen)
- techcrunch.com/category/artificial-intelligence
- theverge.com/ai-artificial-intelligence
- wired.com/tag/artificial-intelligence
- arstechnica.com/ai
- theinformation.com (Paywall, aber bricht echte Stories)

### Tier 6 — Signal-Ebene (nicht primär)
- news.ycombinator.com — Front-Page-AI-Items als Aufmerksamkeitssignal
- reddit.com/r/MachineLearning
- reddit.com/r/LocalLLaMA

### Explizit vermeiden
- SEO-Farmen ("10 Best AI Tools 2026"-Seiten)
- Outlets, die nur Pressemitteilungen weiterreichen
- LinkedIn-Thought-Leader-Posts als Primärquelle
- Twitter/X-Threads als **einzige** Quelle (nur nutzen, um die Primärquelle zu finden)
- KI-generierte News-Seiten ohne echten Autor

**Dedup-Regel:** Bei mehreren Outlets zur selben Story gewinnt der höhere Tier.

---

## Fokus-Themen

Format: `- KEYWORD1, KEYWORD2: kurze Beschreibung`. Match auf Headline/Summary/"Warum"-Feld (case-insensitive). Treffer bekommen ★-Badge und werden hochsortiert. Andere Items bleiben drin.

### Aktive Fokus-Themen

- KI-Verordnung, EU AI Act, AI Act, KI-VO: EU-Regulierung von KI-Systemen, besonders Hochrisiko-Einstufung und GPAI-Pflichten
- BNetzA, Bundesnetzagentur, KI-MIG: Deutsche Marktaufsichtsbehörde für KI ab August 2026
- BSI, BaFin: Sektorale Aufsicht für Cybersecurity und Finanzdienstleistungen
- Claude, Anthropic: Modelle und Tools von Anthropic, besonders Claude Code und MCP
- Agenten, Agent SDK, agentic AI: Agenten-Frameworks und Deployment-Patterns
- MCP, Model Context Protocol: Protokoll für Agent-Tooling und externe Integrationen
- Alignment, AI Safety: Sicherheits- und Alignment-Forschung
- Open Source, Open Weights: Offene Modelle und ihre Veröffentlichung

<!-- Hier eigene Themen ergänzen oder bestehende anpassen -->

---

## Glossar

Format: `### BEGRIFF` Heading, dann 1–3 Sätze Definition. Begriffe alphabetisch. Synonyme in Klammern hinter dem Hauptbegriff.

### Agent SDK
Anthropics SDK zum Bau von Claude-basierten Agenten. Erlaubt programmatische Tool-Nutzung, Multi-Step-Workflows und Integration externer Systeme.

### Antigravity
Googles agentenorientierte Entwicklungsplattform, vorgestellt auf der I/O 2026. Erlaubt das Orchestrieren und Bauen von KI-Agenten auf Basis der Gemini-Modelle.

### BaFin
Bundesanstalt für Finanzdienstleistungsaufsicht. Im deutschen KI-MIG als sektorale Aufsichtsbehörde für KI im Finanzsektor benannt.

### BNetzA (Bundesnetzagentur)
Deutsche Behörde, die ab dem 2. August 2026 als zentrale Marktüberwachungsbehörde für die EU-KI-Verordnung in Deutschland fungiert. Beheimatet das Koordinierungs- und Kompetenzzentrum für die KI-Verordnung (KoKIVO).

### BSI
Bundesamt für Sicherheit in der Informationstechnik. Sektorale Aufsichtsbehörde für Cybersicherheits-Aspekte der KI-Verordnung in Deutschland.

### Claude Code
Anthropics Kommandozeilen-Werkzeug für agentisches Programmieren. Lässt Entwickler Coding-Aufgaben an Claude delegieren, mit lokalem Datei- und Shell-Zugriff.

### Code with Claude
Anthropics Entwickler-Konferenzreihe. Erstmals 2026 auch in London — Anthropics erstes dediziertes Developer-Event in Europa.

### Colossus 1
Von xAI in Memphis gebautes Rechenzentrum. Anthropic mietet die komplette Compute-Kapazität, um wachsende Nachfrage nach Claude Code und API zu bedienen.

### CSAM
Child Sexual Abuse Material. KI-generierte CSAM wird durch den AI Act Omnibus explizit verboten.

### Digital Omnibus
EU-Gesetzgebungspaket zur Vereinfachung digitaler Regulierung, in dem der AI Act Omnibus eingebettet ist. Politische Einigung am 7. Mai 2026.

### DSGVO
Datenschutz-Grundverordnung. EU-weite Verordnung zum Schutz personenbezogener Daten, in Kraft seit Mai 2018.

### GPAI (General Purpose AI)
Im EU-AI-Act definierte Kategorie für Allzweck-KI-Modelle. Pflichten gelten seit 2. August 2025, Kommissions-Durchsetzung ab 2. August 2026.

### HRAIS
High-Risk AI System. Im EU-AI-Act definierte Kategorie für Hochrisiko-KI-Systeme nach Art. 6, mit umfangreichen Compliance-Pflichten.

### KI-MIG
KI-Marktüberwachungs-und-Innovationsförderungs-Gesetz. Deutsches Umsetzungsgesetz zur EU-KI-Verordnung. Benennt die BNetzA als zentrale Aufsichtsbehörde.

### KI-VO (KI-Verordnung, EU AI Act)
EU-Verordnung 2024/1689 zur Regulierung von KI-Systemen. Risikobasierter Ansatz mit vier Stufen, von verbotenen Praktiken bis minimales Risiko. Kern-Pflichten ab 2. August 2026.

### KoKIVO
Koordinierungs- und Kompetenzzentrum für die KI-Verordnung, angesiedelt bei der BNetzA. Soll Fachbehörden koordinieren und einheitliche Rechtsauslegung in Deutschland sicherstellen.

### MCP (Model Context Protocol)
Von Anthropic eingeführtes Protokoll, das KI-Agenten standardisierten Zugriff auf externe Tools, Datenquellen und Systeme ermöglicht. MCP-Tunnels erlauben Zugriff auf interne Systeme ohne öffentliche Exposition.

### Project Glasswing
Anthropics Initiative zum Einsatz von Claude Mythos Preview für Cybersecurity, insbesondere Schwachstellen-Findung. Angekündigt im Mai 2026 mit Industriepartnern wie Google.

### Sandboxes (Claude Agents)
Anthropics Funktion, mit der Unternehmen Claude-Agenten auf eigener Infrastruktur laufen lassen können. Adressiert Datenkontrolle und Governance-Bedenken in regulierten Branchen.

### Vibe Coding
Von Simon Willison popularisierter Begriff für eine Coding-Praxis, bei der Entwickler nicht jede Zeile prüfen, sondern auf das Gefühl vertrauen, ob der vom LLM generierte Code "richtig wirkt". Umstritten als Praxis für Produktion.

<!-- Neue Einträge unten alphabetisch ergänzen, wenn Claude welche vorschlägt -->
