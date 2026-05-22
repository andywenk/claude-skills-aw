---
name: ai-news
description: Produces a curated digest of the last 7 days of AI news — research, industry moves, policy, and tools — as an interactive expandable widget with German summaries. Use this skill whenever the user types `/ai-news` (with or without modifiers like a different time window, topic filter, or item count). Also use it whenever the user asks for "AI news", "what happened in AI this week", "AI digest", "KI-News", "was gibt's Neues in KI", or any similar request for a roundup of recent AI developments. Calibrated for an experienced reader who already knows the basics — skip beginner explainers and skip pure hype.
---

# AI News Digest (`/ai-news`)

A skill that produces a high-signal, German-language digest of recent AI developments, rendered as an interactive expandable widget.

## When to trigger

- The user types `/ai-news` (with or without arguments)
- The user asks for a roundup of recent AI news, an "AI digest", "KI-News", "was ist neu in KI", or similar
- The user mentions wanting to catch up on AI/ML developments from the last days/week

## Arguments (optional)

The user may pass modifiers after `/ai-news`:
- A time window: `/ai-news 3d`, `/ai-news 14d`, `/ai-news last 24h`
- A topic filter: `/ai-news alignment`, `/ai-news EU policy`, `/ai-news open source`
- An item count: `/ai-news 20`

Default if no arguments: **last 7 days, 10-15 items, all four categories**.

## What the digest covers

Four categories, weighted roughly equally unless one is genuinely quiet:

1. **Research** — papers, technical reports, alignment/safety work
2. **Industry** — model releases, major deals, acquisitions, hiring/leadership moves (only if substantive)
3. **Policy** — EU AI Act, German regulation (KI-Bundesverband, BSI), international AI governance
4. **Tools** — agent frameworks, dev tools, infra, open-source releases worth knowing about

## Sources

The list of sources to prioritize, grouped by tier, lives in **`references/sources.md`**. Read that file at the start of every run.

Edit `references/sources.md` to add, remove, or reorder sources — no changes to this SKILL.md are needed when curating the source list.

Higher tiers are preferred during deduplication: if two outlets cover the same story, the higher tier wins. The "Explicitly avoid" section at the bottom of `sources.md` lists source types to ignore.

## Glossary

A wachsendes glossary of technical terms and abbreviations lives in **`GLOSSAR.md`** (same folder as this SKILL.md). The file:

- Provides definitions used to populate clickable popovers in the widget.
- Grows automatically: each run, new technical terms that appear in the digest are added to the file.
- Is user-editable: definitions can be refined, entries deleted, synonyms added. The skill never overwrites existing entries.

Format details and term-detection rules are in Steps 1, 5, and 7 of the workflow.

## Focus

The user's focus topics live in **`FOCUS.md`** (same folder as this SKILL.md). The file lets the user define keywords and short descriptions of topics they care about more than others.

Focus affects sorting (matches first) and adds a ★-Fokus badge to matched items. It is **soft** — non-matched items still appear in the digest.

Format details are in `FOCUS.md` itself. Parsing rules are in Step 1 of the workflow.

## What to filter OUT (hard rules)

- **Hype/clickbait** — "AI will end the world by Tuesday" style. If the headline is the whole content, skip it.
- **Pure product launches without substance** — "X startup launches Y AI feature" with no architectural detail, no benchmark, no real news beyond the launch itself.
- **Beginner content** — "What is an LLM", "How to use ChatGPT", explainer pieces aimed at people who don't know the field.
- **Duplicate coverage** — if 5 outlets covered the same story, pick the best single source (usually the primary, e.g. the lab blog over the press recap) and drop the rest.

When in doubt: would Simon Willison or Zvi consider this worth a paragraph? If no, skip.

## Workflow

### Step 1: Load sources, glossary, focus, and parse arguments

Read three files at the start of every run:

1. **`references/sources.md`** — current list of prioritized sources.
2. **`GLOSSAR.md`** — known terms and their definitions. Parse out each `### TERM` heading and the paragraph below it. Note: terms may have parenthetical synonyms — `### KI-VO (KI-Verordnung, EU AI Act)` means three matching strings.
3. **`FOCUS.md`** — the user's focus topics. Parse each line under "Aktive Fokus-Themen" that starts with `-`. Format: `- KEYWORD1, KEYWORD2: description`. Collect all keywords (the description is context for you, not for matching).

**First-run bootstrap:** If `GLOSSAR.md` or `FOCUS.md` does not yet exist (fresh install from the repo), copy the corresponding `.example` file (`GLOSSAR.md.example`, `FOCUS.md.example`) to the live filename and proceed. Tell the user once that the file was created so they know where to customize it.

Then determine the time window, topic filter (if any), and target item count from the user's message. Apply defaults where not specified.

### Step 2: Search across categories

Run web searches across the four categories. Use multiple targeted queries rather than one broad one — for example:

- `Anthropic OpenAI DeepMind announcement <last week>`
- `AI safety alignment research <last week>`
- `EU AI Act <current month>`
- `AI agent framework release <last week>`
- `arXiv AI paper discussed <last week>`

Use the actual current date (not "2025" or stale years) when forming time-bounded queries. Be specific — short 3-6 word queries tend to work best.

For German/EU policy, also search in German: `KI-Verordnung`, `BSI KI`, `EU AI Act Umsetzung`.

### Step 3: Filter, deduplicate, and apply focus weighting

Apply the filter rules above. When the same story shows up across outlets, keep the primary source. Aim for the target item count after filtering — if the week was quiet, return fewer high-quality items rather than padding with weak ones.

**Focus weighting (from FOCUS.md):**
- For each remaining item, check if any focus keyword appears in the headline, summary, or "Warum es wichtig ist" field (case-insensitive substring match).
- Items with at least one focus match get a `focus: true` flag and a `★ Fokus` badge in the widget.
- Sort order: focus-matched items first (by importance, then recency); non-matched items follow (same internal sort). Focus weighting is **soft** — non-matched items still appear in the digest.
- If 80%+ of items are focus-matched, the focus list may be too broad; if 0% are matched, it may be too narrow. Quietly note this to the user at the end of the response, not in the widget.

### Step 4: Write the digest

For each item, produce:

- **Headline** (short, in source language — English or German depending on origin)
- **One-paragraph summary in German** (2-4 sentences, factual, no fluff)
- **"Warum es wichtig ist"** — one short sentence on why this matters, calibrated for a technical reader who works in AI/insurance compliance and follows the field
- **Source link** (primary source preferred)
- **Category tag** (Research / Industry / Policy / Tools)
- **Date** (when the story broke, not when it was indexed)
- **Focus flag** (set in Step 3)

Tone for German summaries: precise, no marketing language, German-native phrasing (not translated-from-English).

### Step 5: Identify and define glossary terms

After writing the digest, scan all summaries and "Warum es wichtig ist" fields for technical terms and abbreviations. A term qualifies if it's a:

- Abbreviation in ALL CAPS (BSI, MCP, GPAI, HRAIS, DSGVO, etc.) — 2 to 6 letters, possibly followed by digits (Gemini 3.5, Opus 4.7).
- Proper noun for a product, project, framework, law, or system (Antigravity, Project Glasswing, KI-VO, Colossus 1, Agent SDK, MCP-Tunnels).
- Domain-specific compound term (Hochrisiko-KI-System, Marktüberwachungsbehörde).

Skip generic words ("Modell", "Unternehmen", "Forschung") and very common terms ("KI", "AI", "LLM" — unless the user explicitly added them to GLOSSAR.md).

For each detected term, check whether it already exists in GLOSSAR.md (match against the heading text and any parenthetical synonyms). If not, write a new entry:

```
### TERM
1-3 sentence German definition. Factual, no marketing. Mention the context where the term appeared if it helps.
```

Insert new entries in alphabetical order, preserving existing entries verbatim. Never overwrite or rewrite existing definitions, even if you'd phrase them differently.

After updating, the full set of glossary terms (existing + newly added) is what will be made clickable in the widget. Build a lookup map: `{ term_lowercase: definition }`, including each synonym from parenthetical headings as its own key pointing to the same definition.

### Step 6: Render as widget

Use the `visualize:show_widget` tool to render an interactive expandable digest. Call `visualize:read_me` with `["interactive"]` first to get the styling tokens.

Widget structure:
- Header with date range (e.g. "AI-Digest · 15.–22. Mai 2026") and item count.
- Category filter chips row (Alle / Research / Industry / Policy / Tools — clicking filters the list).
- Each item is a collapsed card showing headline + category tag + ★-Fokus badge if applicable + date. Focus-matched items appear first.
- Expanding reveals the German summary, "Warum es wichtig ist", and source link.
- **Clickable terms**: In the summary and "Warum es wichtig ist" text, wrap every occurrence of a glossary term in an inline button. The first occurrence per item is enough — don't wrap repeated mentions in the same item. Clicking shows the term's definition as a small inline popover directly below the clicked term (not a chat message). Style: dotted underline, slightly emphasized text color (`var(--color-text-info)`), cursor pointer. Popover: small card with the term name as heading and the definition as body, plus a close (×) button.
- The whole thing stays calm and dense — this is a reading tool, not a dashboard.

Implementation note for the widget: pass the glossary lookup map as a JSON object in a `<script>` block. Use a single delegated click handler on the items container that checks `data-term` attributes. Popover is a single element that gets moved to the clicked term's parent and shown there. Popovers are dismissible by clicking × or clicking outside.

Keep the widget self-contained: no external API calls from inside the widget (it can't make them anyway).

### Step 7: Save the updated glossary

If new terms were added in Step 5, write the full updated GLOSSAR.md to disk. Preserve the file's header section (everything above the `## Einträge` line) verbatim. Only the entries section is modified.

## Output checklist

Before showing the widget, verify:

- [ ] 10-15 items (or whatever the user requested), unless the week was genuinely quiet
- [ ] All four categories represented (unless one was empty)
- [ ] No duplicate stories
- [ ] No beginner content, no pure hype, no contentless product launches
- [ ] Every item has a "Warum es wichtig ist" line
- [ ] Every item has a working source link
- [ ] Summaries are in German, headlines in source language
- [ ] Focus matches sorted to the top with ★-Fokus badge
- [ ] All detected technical terms either exist in GLOSSAR.md or have been added
- [ ] Glossary terms in summaries are wrapped as clickable popovers (first occurrence per item)
- [ ] GLOSSAR.md file saved if new entries were added

## Example invocations

- `/ai-news` → last 7 days, 10-15 items, all categories, default widget
- `/ai-news 3d` → last 3 days
- `/ai-news alignment` → last 7 days, filtered to alignment/safety topics
- `/ai-news 20 policy` → last 7 days, 20 items, policy-focused
