---
name: scraperapi-seo-audit
description: >
  Run an SEO audit of a specific website or page using live SERP and page data from the ScraperAPI
  MCP server. Use this skill whenever the user names their own site or URL and wants to check SEO
  health, find out why a page isn't ranking, diagnose a traffic drop, check indexation, review
  on-page signals (titles, meta, headings, schema), compare their site against competitors in
  search, find content gaps, or get prioritized SEO fixes. Trigger on: "audit the SEO of
  example.com", "why isn't my page showing up on Google", "my traffic dropped", "what's wrong with
  my site's SEO", "SEO check for [URL]". It needs a site to audit: for keyword-only questions with
  no site (what Google shows for a keyword, which keywords to target, AI Overview presence), use
  scraperapi-serp-intelligence instead. Requires the ScraperAPI MCP server and an API key.

  Note: Transmits user-supplied queries, URLs, and content to ScraperAPI.
metadata:
  openclaw:
    requires:
      env:
        - SCRAPERAPI_API_KEY
    emoji: "📈"
    homepage: https://docs.scraperapi.com/
---

# ScraperAPI SEO Audit

Live SEO analysis powered by ScraperAPI's real-time SERP and scraping tools. It needs the ScraperAPI MCP server connected with the user's API key; if `google_search` and `scrape` aren't among your tools, tell the user to connect the server first. The approach is outside-in: start with what Google is actually showing, then scrape pages to explain why.

**You, Claude, make the tool calls yourself.** Call the MCP tools, read the results, and report findings. The user asked for an audit, so code or instructions for running the calls themselves would hand the work back to them.

**Report only what the tools returned.** Cite the tool call and output behind each finding, so the user can check it. When something can't be measured with these tools, say so and name the right external tool: an SEO finding the user can't trace is one they can't act on with confidence.

---

## MCP Tools Available

These tools come from the ScraperAPI MCP server. Their full names carry a prefix that depends on how the server is connected (plugin, claude.ai connector, or a local config), so match them by the tool name below.

| Tool | What it returns |
|------|----------------|
| `google_search` | Live structured SERP as parsed JSON: `organic_results` (position, title, link, snippet), `related_questions` (People Also Ask), `knowledge_graph`, `videos`, `pagination`. Other SERP features appear only when the response includes them. Geo-targeting via `countryCode` and `gl`; pagination via `start`. |
| `scrape` | Full page content. Use `outputFormat: "html"` for schema markup, meta tags and canonical inspection, `outputFormat: "markdown"` for content and word count. Rendering is off unless you set `render: true`. |
| `google_news` | Brand and topic mentions with dates, for E-E-A-T and freshness signals. Filter by date with `timePeriod` (`1M`, `1Y`) or `dateRangeStart`/`dateRangeEnd`. |

### Render guidance for `scrape`
Start without flags: static HTML (WordPress, most marketing sites, news articles) works that way and costs 1 credit. If the returned HTML has an empty `<head>` or missing body content, the page builds itself with JavaScript (React/Next.js apps, SPAs): retry with `render: true` (10 credits). Add `premium: true` as well (25 credits together) only if the rendered request is still blocked. Note any escalation in the report.

---

## Core Workflow

For every SEO audit request:

1. **Probe for missing parameters** — Confirm URL, target keywords, geography, and scope before starting (see Parameter Probing below).
2. **Select mode** — Single-Page Audit (Mode A) or Site-Wide Audit (Mode B).
3. **Stage 1** — Indexation & SERP Snapshot.
4. **Stage 2** — Keyword Visibility and SERP Features.
5. **Stage 3** — On-Page Analysis.
6. **Stage 4** — Competitor Gap Analysis.
7. **Stage 5** — Brand & Freshness Signals.
8. **Calculate health grade** and write report using `references/report-template.md`.

## Data Collection Rules

- **Parallelize** — Within each stage, issue all tool calls in a single response turn; calls made one at a time make the audit several times slower.
- **Call budget** — Mode A: up to 12 tool calls. Mode B: up to 30. Page 2–3 searches (Stage 2) count toward the budget. If you need more, tell the user and ask how to prioritize.
- **Credit budget** — From the [public credit costs](https://docs.scraperapi.com/getting-started/quick-start/credits-and-requests-costs): each `google_search` or `google_news` call costs 25 credits; a page costs 1 on an ordinary site, 10 on a bot-protected one or with `render`, 25 with `render` + `premium`. Mode A is roughly 125–300 credits and Mode B 200–450, mostly from searches; each page-2/3 search adds 25. Tell the user the estimate before starting.
- **Geo-target every search** — Pass the target market's `countryCode` and `gl` to every `google_search` and `google_news` call, brand queries included. Rankings differ by country, so a result without the right country tells the user little.
- **Handle render failures** — If a page returns empty or near-empty content, retry once with `render: true`, then with `premium: true` added (see [Render guidance](#render-guidance-for-scrape)).
- **Cite every finding** — Include the exact query or URL. Users must be able to trace every claim back to a tool call.
- **Date-stamp every report** — Include "Audited: [date]" so users know how fresh the data is.

---

## Parameter Probing

Check the user's request for these four parameters. Infer what you can from context first, then ask everything still unclear in one message, so the user answers once.

| Parameter | Default if not specified | When to ask |
|-----------|--------------------------|-------------|
| **Site URL** | — | Always ask if not provided |
| **Target keywords** | Infer from homepage content after scraping | Confirm inferred keywords before proceeding |
| **Target market / geography** | United States | Ask unless the request or the site makes it clear (a `.co.uk` domain, prices in euros, "we only sell in Canada"). A UK site's rankings on google.com tell you little. |
| **Audit scope** | Mode B (site-wide) for bare domains; Mode A (single-page) for full URL paths | Ask only if genuinely ambiguous |

---

## Audit Modes

### 1. Single-Page Audit (Mode A)

**When to use**: User gave a specific URL, asked why a particular page isn't ranking, or has a narrow question about one page's SEO performance. ~8–12 tool calls.

**Data gathering**:
```
# Stage 1: Indexation check (run in parallel)
google_search  query="site:[domain]"     countryCode="[cc]"  gl="[cc]"
google_search  query='"[Brand Name]"'    countryCode="[cc]"  gl="[cc]"

# Stage 2: Keyword visibility — one per target keyword (run in parallel)
google_search  query="[target keyword 1]"  countryCode="[cc]"  gl="[cc]"
google_search  query="[target keyword 2]"  countryCode="[cc]"  gl="[cc]"

# Stage 3: On-page analysis — target page + homepage (run in parallel)
scrape  url="[target-page-url]"  outputFormat="html"
scrape  url="[target-page-url]"  outputFormat="markdown"
scrape  url="[homepage-url]"     outputFormat="html"

# Stage 4: Top 2 competitors surfaced in Stage 2 (run in parallel)
scrape  url="[competitor-1-url]"  outputFormat="html"
scrape  url="[competitor-2-url]"  outputFormat="html"

# Stage 5: Brand freshness signals
google_news  query='"[Brand Name]"'  countryCode="[cc]"  timePeriod="1Y"
```

**Analysis**: Run all five Audit Stages below. Focus depth on the target page and its direct competitors.

**Output format**: Use `references/report-template.md`. Adapt section depth to single-page scope. Include health grade.

---

### 2. Site-Wide Audit (Mode B)

**When to use**: User gave a bare domain or said "audit my site." Samples homepage plus up to 7 key pages inferred from homepage navigation. ~20–30 tool calls.

**Data gathering**:
```
# Stage 1: Indexation check (run in parallel)
google_search  query="site:[domain]"     countryCode="[cc]"  gl="[cc]"
google_search  query='"[Brand Name]"'    countryCode="[cc]"  gl="[cc]"
google_search  query="[Brand Name]"      countryCode="[cc]"  gl="[cc]"

# Stage 2: Keyword visibility (run in parallel, one per keyword)
google_search  query="[target keyword 1]"  countryCode="[cc]"  gl="[cc]"
google_search  query="[target keyword 2]"  countryCode="[cc]"  gl="[cc]"
google_search  query="[target keyword 3]"  countryCode="[cc]"  gl="[cc]"

# Stage 3: On-page analysis — homepage + up to 7 key pages (run in parallel)
# Infer key pages from homepage navigation links
scrape  url="[homepage-url]"   outputFormat="html"
scrape  url="[page-2-url]"     outputFormat="html"
scrape  url="[page-3-url]"     outputFormat="html"
# ... (up to 8 pages total)
# Follow with markdown for content depth assessment on highest-priority pages:
scrape  url="[priority-page-url]"  outputFormat="markdown"

# Stage 4: Top 2–3 competitors per keyword (run in parallel)
scrape  url="[competitor-1-url]"  outputFormat="html"
scrape  url="[competitor-2-url]"  outputFormat="html"
scrape  url="[competitor-3-url]"  outputFormat="html"

# Stage 5: Brand freshness signals
google_news  query='"[Brand Name]"'  countryCode="[cc]"  timePeriod="1Y"
```

**Analysis**: Run all five Audit Stages below at full depth across all sampled pages.

**Output format**: Use `references/report-template.md`. Full section depth for all five stages. Include health grade.

---

## Audit Stages

These five stages apply to both modes. Run them in order; parallelize all calls within each stage.

### Stage 1 — Indexation & SERP Snapshot
**What it answers**: "Does Google know this site exists, and what does it see?"

Extract from `site:` results: approximate indexed page count, sitelinks presence, any unexpected URLs in the index (staging domains, UTM-parameterised URLs, duplicate content signals). From brand search: Knowledge Panel presence, branded SERP features.

Note: `site:` counts are Google's displayed estimate — approximate only. Precise coverage data requires Google Search Console. Always caveat this in findings.

Read `references/serp-playbook.md` for exact query patterns and extraction fields.

### Stage 2 — Keyword Visibility and SERP Features
**What it answers**: "Where does the site actually appear, and what is Google showing above it?"

For each target keyword, extract from `organic_results`: the site's position, the ranking URL, and the title and snippet as shown in the SERP. From `related_questions`, list the top 3–4 People Also Ask questions.

Report other SERP features (featured snippet, AI Overview, shopping or local results) only when the response actually contains them. When it does, put them first for that keyword, with the domains they cite: a site ranking #1 below an AI Overview is in a different position from one ranking #1 on a clean SERP. When the response has no field for a feature, write "not in SERP data" rather than "none", because the parsed results may not capture every feature Google shows.

If the site ranks outside position 10, search pages 2–3 (`start: 10`, `start: 20`) before classifying it as "not ranking". Each extra page is another 25-credit call that counts toward the call budget, so do it only for keywords where the site isn't in the top 10. Extract the top 3 organic competitors per keyword (excluding Wikipedia/Reddit unless they consistently outrank the target) — these feed Stage 4.

Read `references/serp-playbook.md` for extraction fields and geo-targeting parameters.

### Stage 3 — On-Page Analysis
**What it answers**: "What signals is Google reading from these pages?"

Scrape each target page with `outputFormat: "html"` for precise signal extraction, then `outputFormat: "markdown"` for content depth. For each page extract and evaluate: title tag (present, keyword-included, 50–60 chars, unique across site), meta description (present, compelling, 150–160 chars), H1 (exactly one, includes keyword, matches intent), H2/H3 hierarchy (logical, covers subtopics competitors rank for), schema markup types (`<script type="application/ld+json">`), internal links, image alt text, approximate word count.

Read `references/page-analysis.md` for the full extraction checklist.

### Stage 4 — Competitor Gap Analysis
**What it answers**: "What are the top-ranking pages doing that this site isn't?"

Take the top 2–3 competitors from Stage 2 per keyword. Scrape them with the same checklist from Stage 3. Compare: schema types in use, H2 topic coverage, content depth (word count), SERP features owned (featured snippet, PAA). Flag patterns appearing across all top-ranking pages — these are the highest-signal gaps, because they reflect what Google has repeatedly rewarded for this intent.

### Stage 5 — Brand & Freshness Signals
**What it answers**: "Does Google have reason to trust this site?"

Run `google_news` for the brand name (`timePeriod: "1Y"`, or `dateRangeStart`/`dateRangeEnd` for the last 3 months). Note mention count and source quality. Check homepage for author bios, about page, expertise signals, and last-modified dates visible in page content.

Note: backlink authority cannot be measured with these tools. If the audit finds strong on-page SEO with poor rankings, flag backlinks as a likely limiting factor with Ahrefs/Semrush as the recommended tool.

---

## Health Grade

Calculate after completing all five stages. Grade is based on finding severity across Stages 1–4.

| Grade | Criteria |
|-------|----------|
| **A** | 0 critical findings, ≤ 2 high-priority issues |
| **B** | 0 critical, 3–5 high-priority issues |
| **C** | 1–2 critical findings, or 6–10 high-priority |
| **D** | 3–5 critical findings, or > 10 high-priority |
| **F** | > 5 critical findings, or indexation blocked |

**Critical findings** (each counts toward grade): `noindex` on a key page, title tags missing across the site, no schema on content pages, blocked by robots.txt.
**High-priority findings** (each counts toward grade): title length violations, duplicate titles, missing meta descriptions, no H1, zero internal linking, image alt text missing at scale.

Include the grade prominently in the executive summary.

---

## Audit Mode Selection Guide

| User says... | Mode |
|---|---|
| "Audit my site" / "Check my SEO" / bare domain provided | Site-Wide (Mode B) |
| "Why isn't [specific page] ranking" / full URL path provided | Single-Page (Mode A) |
| "My traffic dropped" / "What's wrong with my SEO" | Site-Wide (Mode B) |
| "How do I rank for [keyword]" | Single-Page (Mode A) — target the most relevant page |
| "Why am I not showing up on Google" | Site-Wide (Mode B) |
| "SEO check on my homepage" / "review my /pricing page" | Single-Page (Mode A) |

---

## Output Quality Standards

1. **Cite every finding.** Include the exact query or URL that surfaced each issue. "Title tag is missing" must name the page URL and the tool call that confirmed it.
2. **Give every ranking its SERP context.** State the SERP features the response shows for that keyword, and which domains they cite. Where the data has no field for a feature, say "not in SERP data".
3. **Geo-target every SERP call.** A ranking result without the correct country context is misleading. Note the target market in the report header.
4. **Separate measurement from inference.** "The SERP shows..." for tool output, "This suggests..." for interpretation, so the user knows which claims they can check.
5. **Make recommendations specific.** "Change the title tag on /pricing from X to Y" — not "improve title tags." Every recommendation names the page and the exact change.
6. **Lead with executive summary and health grade.** Many users only read the top. The first section must state the grade, the 3 most important issues, and the single highest-impact action.
7. **Out-of-scope factors belong in Out-of-Scope Notes.** Flag with the right external tool: Core Web Vitals (field data) → PageSpeed Insights / Search Console; Crawl errors / coverage → Search Console > Coverage; Backlink profile → Ahrefs, Semrush, or Moz; Click-through rates → Search Console > Performance; Manual penalties → Search Console > Manual Actions.
8. **Stay within the call budget.** Mode A: 12 calls. Mode B: 30. Page 2–3 searches count. If more are needed, tell the user and ask how to prioritize.
