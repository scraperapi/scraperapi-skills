---
name: scraperapi-research-agent
description: >
  Sets up and runs a Python research script in the user's project: it plans Google searches, scrapes the top sources
  through ScraperAPI, uploads them to the Anthropic Files API, and has Claude write a cited
  markdown report with a numbered source list. Use only when the user wants this script run, or
  wants a standalone, repeatable research pipeline outside the chat. Trigger on: "run the
  scraperapi research agent on X", "use the research agent script to investigate Y", "give me a
  script that researches a question and writes a cited report". Requires Python 3.10+, `pip install
  -U requests anthropic`, a ScraperAPI key and an Anthropic API key. For research done directly in the
  conversation, use the ScraperAPI MCP tools (scraperapi-mcp); for a whole market or category, use
  scraperapi-market-research; for one person or company, use scraperapi-lead-enrichment.

  Note: Transmits user-supplied queries, URLs, and content to ScraperAPI; uploads scraped content
  to the Anthropic Files API.
metadata:
  openclaw:
    requires:
      env:
        - SCRAPERAPI_API_KEY
        - ANTHROPIC_API_KEY
    emoji: "🔬"
    homepage: https://docs.scraperapi.com/
---

# ScraperAPI Research Agent

End-to-end autonomous research: ScraperAPI finds and fetches sources → Anthropic Files API
ingests them as cited documents → Claude synthesizes a report.

**Run it:** save the script from [references/research-agent-script.md](references/research-agent-script.md)
as `research_agent.py` in the user's project, then:

```bash
# Install dependencies (a recent anthropic SDK: the script uses the Files API)
pip install -U requests anthropic

# Replace YOUR_API_KEY and YOUR_ANTHROPIC_API_KEY at the top of research_agent.py

# Run
python research_agent.py \
  --question "What are the best practices for rate limiting in web APIs?" \
  --max-sources 5 \
  --output report.md
```

---

## Planning Checklist

Before starting a research run, establish:

- [ ] **Question clarity** — Is the question specific enough to produce useful search queries? Vague questions like "tell me about AI" produce noise. Better: "What are the tradeoffs between RAG and fine-tuning for domain-specific LLMs?"
- [ ] **Source count** — How many sources are needed? 3–5 is usually sufficient for a factual summary; 8–10 for a comparative analysis. More sources = more ScraperAPI credits.
- [ ] **Recency** — The script doesn't filter search results by date. If the answer depends on recent events, put the year or "latest" in the question so the planned searches carry it.
- [ ] **Credit budget** — Each Google search costs 25 credits and the script runs 1–3. Each scraped page costs 1 credit on a normal site and 10 on a bot-protected one. 5 sources on normal sites ≈ 30–85 credits; see [Credit cost estimate](#credit-cost-estimate).
- [ ] **Stop condition** — Define when to stop. The default stop is `--max-sources` (5). Do not loop indefinitely.

---

## Research Loop

```
1. PLAN
   ↓ Claude decomposes the question into 2–3 targeted search queries

2. DISCOVER
   ↓ ScraperAPI google/search structured endpoint → list of (url, title, snippet)

3. DEDUPLICATE
   ↓ Filter to top N unique URLs (default: 5), skipping PDFs and low-quality domains

4. FETCH
   ↓ ScraperAPI scrape each URL as markdown (output_format=markdown)
   ↓ Skip pages returning < 200 characters (blocked, error pages)

5. UPLOAD
   ↓ Upload each scraped page to Anthropic Files API as a text/plain artifact
   ↓ Store file_id for each source

6. SYNTHESIZE
   ↓ Claude (claude-opus-4-8, adaptive thinking) reads all document artifacts
   ↓ Returns structured report with inline citations [1], [2]...

7. CLEAN UP
   ↓ Delete uploaded file artifacts from Anthropic
   ↓ Write or print the final report

STOP when: max_sources reached, or all queries exhausted (whichever comes first).
```

---

## Stop Conditions

The agent stops when any of the following is true:

1. **`--max-sources` reached** (default: 5): that many pages scraped successfully.
2. **Candidates exhausted**: the script tries at most `2 × --max-sources` URLs from the searches, then stops even if fewer pages succeeded.

Together these bound a run: at most 3 searches and `2 × --max-sources` page fetches, so `--max-sources` is the credit cap.

---

## Key Parameters

| Flag | Default | Description |
|------|---------|-------------|
| `--question` | (required) | Research question |
| `--max-sources` | 5 | Max pages to scrape (credit budget) |
| `--output` | stdout | Write report to file |
| `--country` | `us` | ScraperAPI country code for geo-targeted results |
| `--model` | `claude-opus-4-8` | Anthropic model for synthesis |

---

## Output Format

See [assets/report_template.md](assets/report_template.md) for the report structure.

The report is a markdown document with:
- **Title** derived from the research question
- **Summary** — 2–3 sentence executive summary
- **Findings** — structured sections with inline `[N]` citations
- **Sources** — numbered bibliography with URLs and titles

---

## Credit Cost Estimate

ScraperAPI credits per run, from the [public credit costs](https://docs.scraperapi.com/getting-started/quick-start/credits-and-requests-costs):

| Sources | Searches (1–3 × 25) | Page fetches, normal sites (1 each) | ScraperAPI total | Anthropic tokens |
|---------|--------------------|-------------------------------------|------------------|------------------|
| 3 | 25–75 | 3–6 | ~28–81 | ~15K in / ~2K out |
| 5 | 25–75 | 5–10 | ~30–85 | ~25K in / ~3K out |
| 10 | 25–75 | 10–20 | ~35–95 | ~50K in / ~5K out |

Page fetches range up to `2 × --max-sources` because the script tries extra candidates when a page fails. Pages behind bot protection cost 10 credits each, so a run on protected sites can reach 75 + 20 × `--max-sources`. Only successful (200) and not-found (404) responses are charged.

---

## ScraperAPI Endpoints Used

- **Google Search** — `GET https://api.scraperapi.com/structured/google/search` — finds source URLs
- **Scrape** — `GET https://api.scraperapi.com/?output_format=markdown` — fetches page content

See [ScraperAPI docs](https://docs.scraperapi.com/) for rate limits and credit costs.

## Anthropic APIs Used

- **Files API** — uploads scraped pages as document artifacts
- **Messages API** — Claude synthesizes the report with citations

Requires an Anthropic API key with access to `claude-opus-4-8` and the Files API.
