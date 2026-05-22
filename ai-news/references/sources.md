# Sources for `/ai-news`

This file lists the sources the AI news digest should prioritize. Edit freely — add, remove, or reorder. The skill reads this file each time it runs.

Tiers below are guidance for trust/weighting, not hard rules. Lower tiers can still produce the best story of the week.

---

## Tier 1 — Lab blogs (primary sources)

- https://www.anthropic.com/news
- https://openai.com/blog
- https://deepmind.google/discover/blog/
- https://ai.meta.com/blog/
- https://www.microsoft.com/en-us/research/blog/
- https://research.google/blog/
- https://mistral.ai/news/

## Tier 2 — Independent analysts (high signal, opinionated)

- https://simonwillison.net/ — Simon Willison, daily-ish posts on what actually matters
- https://thezvi.substack.com/ — Zvi Mowshowitz, weekly AI roundups
- https://www.gwern.net/ — Gwern, occasional but deep
- https://www.interconnects.ai/ — Nathan Lambert, post-training / RL focus
- https://importai.substack.com/ — Jack Clark, weekly newsletter
- https://www.aisnakeoil.com/ — Arvind Narayanan & Sayash Kapoor, skeptical takes

## Tier 3 — Research aggregators

- https://arxiv.org/list/cs.AI/recent
- https://arxiv.org/list/cs.LG/recent
- https://arxiv.org/list/cs.CL/recent
- https://huggingface.co/papers — daily papers with discussion
- https://paperswithcode.com/

## Tier 4 — EU / German policy

- https://ki-verband.de/ — KI-Bundesverband
- https://www.bsi.bund.de/ — BSI (Bundesamt für Sicherheit in der Informationstechnik)
- https://artificialintelligenceact.eu/ — EU AI Act tracker
- https://digital-strategy.ec.europa.eu/en — EU Commission digital policy
- https://www.heise.de/thema/Kuenstliche-Intelligenz — Heise KI coverage
- https://www.tagesschau.de/thema/kuenstliche_intelligenz — Tagesschau KI coverage
- https://netzpolitik.org/ — German digital policy & rights

## Tier 5 — Tech press (use sparingly, prefer primary sources when available)

- https://techcrunch.com/category/artificial-intelligence/
- https://www.theverge.com/ai-artificial-intelligence
- https://www.wired.com/tag/artificial-intelligence/
- https://arstechnica.com/ai/
- https://www.theinformation.com/ — paywalled but breaks real stories

## Tier 6 — Signal layer (not primary, but useful)

- https://news.ycombinator.com/ — front-page AI items indicate technical community attention
- https://www.reddit.com/r/MachineLearning/ — top-of-week posts
- https://www.reddit.com/r/LocalLLaMA/ — open-source/local model news

---

## Explicitly avoid

- SEO farms ("10 Best AI Tools 2026" sites)
- Outlets that pure-republish press releases without editorial layer
- LinkedIn thought-leader posts as primary sources
- Twitter/X threads as the *only* source for a claim (use only to find the underlying primary source)
- AI-generated news sites (often have no real author or accountability)

---

## Notes for future edits

- If you add a new source, place it in the right tier — this affects how much weight the skill gives it during deduplication.
- If two outlets cover the same story, the higher tier wins.
- German-language sources are weighted higher for policy topics; English-language sources are weighted higher for research and industry.
