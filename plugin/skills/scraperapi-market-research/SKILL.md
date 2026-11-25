---
name: scraperapi-market-research
description: >
  Market research powered by live web data. Use this skill whenever the user wants to understand a
  market, category, or customer segment, including consumer sentiment, demand and trend signals,
  price and category structure, or the competitive landscape, even if they don't say "market
  research". Trigger on: "research the [X] market", "what do customers think about [category]",
  "how is [category] priced", "what trends are shaping [industry]", "who are the players in
  [space]". It works at the category level: for a profile of one person or company use
  scraperapi-lead-enrichment; for keyword SERP analysis use scraperapi-serp-intelligence; for the
  current price of specific products use scraperapi-price-monitoring; to run the standalone Python
  research script use scraperapi-research-agent. Requires the ScraperAPI MCP server.

  Note: Transmits user-supplied queries, URLs, and content to ScraperAPI.
metadata:
  openclaw:
    requires:
      env:
        - SCRAPERAPI_API_KEY
    emoji: "📊"
    homepage: https://docs.scraperapi.com/
---

# Market Research

Live market research powered by ScraperAPI's web scraping infrastructure. Pulls structured data from search engines, product platforms, review sites, job boards, and any public URL, then synthesizes it into research-grade analysis.

**You, Claude, make the tool calls yourself.** Call the MCP tools, read the results, and report findings. The user asked for research, so code or instructions for running the calls themselves would hand the work back to them.

**Start with the research question, not the data source.** Before collecting anything, identify what decision the user is trying to inform. The same market looks completely different depending on whether you're trying to enter it, price against it, or understand why customers leave it.

**Gather live data before you analyze.** Training data has a cutoff and markets don't, so a market answer from memory alone is likely to be out of date in ways neither you nor the user can see. Use what you already know to plan the searches, then let the live data drive the findings.

---

## MCP tools

These tools come from the ScraperAPI MCP server. Their full names carry a prefix that depends on how the server is connected (plugin, claude.ai connector, or a local config), so match them by the tool name below.

| Tool | Best used for | Key arguments |
|------|--------------|---------------|
| `google_search` | Discovering sources, mapping who's present, SERP analysis | `query`, `countryCode`, `timePeriod` (`1D`, `1W`, `1M`, `1Y`) |
| `google_news` | Trend signals, recent market moves, funding and launches | `query`, `countryCode` |
| `google_shopping` | Category pricing structure, product listings at scale | `query`, `countryCode` |
| `google_jobs` | Market-level hiring signals, technology bets, expansion | `query`, `countryCode` |
| `scrape` | Any public URL: homepages, pricing pages, review pages | `url`, `render`, `premium`, `countryCode`, `outputFormat` (`markdown` reads well) |
| `amazon_search` | Category structure, bestseller rankings, price distribution | `query`, `tld` |
| `amazon_product` | Product detail, rating, review summary | `asin` (take it from `amazon_search` results) |
| `amazon_offers` | Seller landscape, price variation across sellers | `asin` |
| `walmart_search` | Category structure and pricing on Walmart | `query` |
| `walmart_product` | Product detail and pricing | `productId` (from `walmart_search`) |
| `walmart_review` | Consumer sentiment on physical products | `productId`, `sort` (`recent`, `helpful`) |
| `ebay_search` | Resale market structure and demand signals | `query` |
| `ebay_product` | Secondary market pricing, condition distribution | `productId` |

The crawler tools (`crawler_job_start`, `crawler_job_status`) deliver their results to a webhook, which a chat session can't receive, so they don't fit this workflow. When a module needs many pages from one site, find the URLs with `google_search` (for example `site:example.com/blog`) and `scrape` the best few.

### When to render

`scrape` without flags costs 1 credit on an ordinary site and works for static HTML: news articles, Wikipedia, most company homepages and `/pricing` pages. Add `render: true` (10 credits) when the page builds its content with JavaScript and a plain fetch returns an empty shell, as on G2 grids, Glassdoor or React apps. Add `premium: true` as well (25 credits together) only when a rendered request is still blocked. Try the cheaper call first: most pages don't need either flag.

---

## Workflow

1. **Scope the request.** Check the four parameters in [Scoping](#scoping). If one is missing and would change the research, ask, with all open questions in one message, and wait for the answer. If a parameter has a sensible default, state it and go on.
2. **Pick the modules.** Use the [module guide](#modules) to choose one or more.
3. **Estimate the cost.** Before running more than one module, tell the user the estimated credits (see [Budget and stop conditions](#budget-and-stop-conditions)).
4. **Run each module.** Open its reference file, announce what you're looking for, then make the calls. Issue independent calls in the same turn so they run in parallel.
5. **Analyze with the module's rules**, separating what the data shows from what you infer.
6. **Write the report** from the module's template in `assets/`, including its gap report: the high-priority questions you couldn't answer, each with one concrete follow-up.

---

## Scoping

| Parameter | Default | Ask when |
|-----------|---------|----------|
| **Geography** | United States | The user didn't state it and the category depends on location: physical goods, retail, local services, regulated industries. For categories sold the same way everywhere (most software), state the default ("I'll use US and global English-language sources") and go on. |
| **Modules** | All four | The request points at one angle ("how is this market priced?" means Price & Category only). Don't ask when the request is clearly broad ("research this market"). |
| **Decision** | General market understanding | Knowing the decision (entering the market, pricing against it, understanding churn) would change what to emphasize. |
| **Time horizon** | Last 12–24 months | The category changes fast (AI tools, crypto) or the user signals historical interest. |

Ask everything you need in one message: a user answering three questions at once loses less time than one answering three separate turns. Keep it to one sentence of context and the questions.

> Before I start: which market should I scope this to — US, UK, EU, or global? And is there a decision behind this — market entry, pricing, or a general landscape?

When you go on with defaults, say so in one line so the user can redirect you:

> Scoping this to the US and running all four modules. Tell me if you want a different geography or a narrower focus.

---

## Budget and stop conditions

Credits per call ([current costs](https://docs.scraperapi.com/getting-started/quick-start/credits-and-requests-costs)):

| Call | Credits |
|------|---------|
| `google_search`, `google_news`, `google_shopping`, `google_jobs` | 25 |
| `amazon_*`, `walmart_*`, `ebay_*` | 5 |
| `scrape`, plain | 1 |
| `scrape` with `render` | 10 |
| `scrape` with `render` + `premium` | 25 |

The `scrape` costs are for an ordinary site. Sites behind bot protection cost more, so treat the estimate as a floor.

A module typically takes 6–12 calls, mostly Google searches, so 150–300 credits. A full four-module report is about 600–1,000 credits. Give the user that estimate before running more than one module.

Limits per run:

- **At most 12 tool calls per module and 40 for a full report.** Past that, the extra data rarely changes the findings, and the cost keeps growing.
- **Stop a module early** when its high-priority questions are answered, when the last 4 calls added nothing new, or when 3 calls in a row returned errors, empty results or gated content.
- **Stop at once** if the user asks you to move to synthesis.

When you stop, say why in one line ("Stopping data collection: the last four searches repeated what I already have. Moving to synthesis.") and list what's missing in the gap report.

### Errors

| Response | What to do |
|----------|-----------|
| Empty result | Try one alternative query or URL. If it's still empty, log the gap and move on. |
| Gated or paywalled page | Note it, and look for the same facts in search snippets or a cached review. Don't reconstruct gated content from memory. |
| 429 (too many requests) | Retry once later in the run, after other calls. If it fails again, stop collecting and note it. |
| 500 | The page blocked the request (not charged). Retry once with `render`, then with `premium`; skip after that. |
| 403 | The account is out of credits. Stop and tell the user; they can add credits at https://dashboard.scraperapi.com/billing. |
| Irrelevant result | Skip it and query a different source. It doesn't count toward the stop conditions. |

---

## Modules

| The user wants to know... | Module | Instructions | Report template |
|---------------------------|--------|--------------|-----------------|
| What customers want, need or complain about in a category | Consumer Sentiment | [references/consumer-sentiment.md](references/consumer-sentiment.md) | [assets/consumer-sentiment-report.md](assets/consumer-sentiment-report.md) |
| Whether a market is growing, and where demand is shifting | Demand & Trend Signals | [references/demand-trends.md](references/demand-trends.md) | [assets/demand-trends-report.md](assets/demand-trends-report.md) |
| How a category is priced and what buyers get at each tier | Price & Category | [references/price-category.md](references/price-category.md) | [assets/price-category-report.md](assets/price-category-report.md) |
| Who the players are and how they're positioned | Competitive Landscape | [references/competitive-landscape.md](references/competitive-landscape.md) | [assets/competitive-landscape-report.md](assets/competitive-landscape-report.md) |
| All of the above | Full report | All four, in the order below | [assets/full-report.md](assets/full-report.md) |

Read only the reference files for the modules you run.

**Full report order:** Consumer Sentiment, Demand & Trends, Price & Category, then Competitive Landscape. Sentiment comes first because the positioning map in Competitive Landscape needs to know which dimensions the market actually competes on. Finish each module, including its gap report, before starting the next. Open with the Research Brief and close with Strategic Recommendations from the full-report template.

---

## Report quality

1. **Research question first.** State what decision the research informs before presenting data.
2. **Separate data from inference.** "The data shows X" and "this suggests Y" are different claims; use that language.
3. **Cite every data point** with its source URL, and give every sentiment finding at least one direct quote with source and approximate date.
4. **Mark confidence.** Say when a finding is well supported and when it's directional. A single data point is not a pattern.
5. **Name the sampling limits.** Reviews skew toward strong opinions, Reddit skews technical, and published prices aren't negotiated enterprise prices.
6. **Report only what the tools returned.** When a gap remains, say so in the gap report rather than filling it from memory: the user can't tell a remembered figure from a live one.
7. **Tiered recommendations:** Immediate, Near-term, Strategic. Each one specific to the findings.
8. **Date-stamp the report:** "Data collected on [YYYY-MM-DD]".
