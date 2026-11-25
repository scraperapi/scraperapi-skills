# SERP & News Monitoring — Detailed Guide

A workflow for monitoring Google Search (SERP) and Google News over time with the `google_search` and `google_news` MCP tools, and reporting what changed between runs. Useful for SEO monitoring, media tracking, and competitive intelligence. For a one-off analysis of a SERP (intent, content gaps, who ranks and why), use the `scraperapi-serp-intelligence` skill instead.

## Overview

This workflow enables:

- Fetching Google Search results (organic + Top Stories)
- Fetching Google News headlines
- Normalizing and canonicalizing URLs
- Deduplicating results
- Producing structured JSON output
- Generating change reports (new / removed / rank shifts)

Use the structured Google tools rather than `scrape`-ing google.com: they cost the same (25 credits per call) and return parsed JSON with explicit positions, so runs can be compared field by field instead of re-parsing a page each time.

## Inputs to Ask For (only if missing)

- **Query** (required)
- **Mode**: `serp` or `news` (default: `serp`)
- **Locale**: `hl` (default: `en`), `gl` (default: `GB`)
- **Result count** for SERP (default: 10)
- **Recency filter**: none | last day | last week | last month

## Calls to Make

### SERP

Call `google_search` with `query`, `num`, `hl`, `gl`, and `countryCode` set to the market, so every run sees the same localized results. For recency, set `timePeriod`: `"1D"` (last day), `"1W"` (last week), or `"1M"` (last month).

Map `organic_results[]` to the SERP JSON below (`position` → `rank`). The documented response has no Top Stories block; if the user specifically needs Top Stories, `scrape` `https://www.google.com/search?q=<QUERY>&hl=<HL>&gl=<GL>` with `outputFormat: "markdown"` for that part only.

### Google News

Call `google_news` with `query`, `hl`, `gl`, `countryCode`, and a `timePeriod` — undated news searches return stale coverage.

If a call fails, see the escalation and error table in [../references/scraping.md](../references/scraping.md).

## Extraction Requirements

### SERP JSON

Extract and return:
- `query`, `fetched_at`, `locale {hl, gl}`, `num`
- `organic_results`: array of
  - `rank` (1..N)
  - `title`
  - `link` (canonicalized; remove Google redirect wrappers)
  - `displayed_link` (if visible)
  - `snippet` (if visible)
- `top_stories` (optional): array of
  - `title`
  - `source` (if visible)
  - `time` (if visible)
  - `link` (canonicalized)

### News JSON

Extract and return:
- `query`, `fetched_at`, `locale {hl, gl}`
- `headlines`: array of
  - `title`
  - `source` (publisher)
  - `time` (relative time if present)
  - `link` (canonicalized)
  - `snippet` (if visible)

## Normalization Rules

- If a result link is a Google redirect (e.g., `/url?q=<real_url>`), extract and use the real URL.
- Canonicalize URLs by stripping common tracking params (`utm_*`).
- Dedupe by canonical URL; if missing/unstable, fallback to hash(title + source).

## Output Format (always)

Return a single JSON object:

```json
{
  "mode": "serp | news",
  "query": "...",
  "locale": { "hl": "...", "gl": "..." },
  "fetched_at": "...ISO timestamp...",
  "results": { "..." },
  "notes": [ "..." ]
}
```

If the user asks for insights, add a short prose summary after the JSON.

## Optional: Change Report

If the user provides a previous snapshot JSON, compute:

- `new_items` — results present now but not in the previous snapshot
- `removed_items` — results in the previous snapshot but not now
- `moved_items` (SERP only) — results whose rank changed

Then summarize the changes.

## Best Practices

- **Keep the parameters identical between runs** (`query`, `num`, `hl`, `gl`, `countryCode`) — otherwise the change report shows differences caused by the settings, not the SERP.
- **Use recency filters** to keep results fresh for monitoring use cases.
- **Store snapshots** as JSON to enable change detection across runs.
- **Dedupe before comparing** — URL canonicalization prevents false positives in change reports.
