# ScraperAPI Skills

[ScraperAPI](https://www.scraperapi.com)'s Agent Skills.

[Learn more about agent skills](https://agentskills.io/).

## Table of Contents

- [Available Skills](#available-skills)
- [Installation](#installation)
- [License](#license)

## Available Skills

### `scraperapi-mcp`

Teaches Claude how to optimally use ScraperAPI's MCP tools — selecting the right tool, setting optimal parameters, minimizing credit costs, and handling large outputs. Includes ready-to-use recipes for common web data workflows.

## Installation

### From GitHub

```bash
npx skills add scraperapi/scraperapi-skills
```

### From [ClawhHub](https://clawhub.ai)

```bash
npx clawhub install scraperapi-mcp
```


## Tool References

| Guide | Description |
|-------|-------------|
| [MCP Server Setup](skills/scraperapi-mcp/references/setup.md) | Server variants, installation, configuration, variant detection |
| [Scraping](skills/scraperapi-mcp/references/scraping.md) | `scrape` tool parameters, escalation strategy, error patterns |
| [Google Search Tools](skills/scraperapi-mcp/references/google.md) | All 5 Google tools, response structures, pagination, time filtering |
| [Amazon SDE Tools](skills/scraperapi-mcp/references/amazon.md) | Product details by ASIN, search, and seller offers/pricing |
| [Walmart SDE Tools](skills/scraperapi-mcp/references/walmart.md) | Search, product details, category browsing, and product reviews |
| [eBay SDE Tools](skills/scraperapi-mcp/references/ebay.md) | Search with filters and product details |
| [Redfin SDE Tools](skills/scraperapi-mcp/references/redfin.md) | For-sale/for-rent property listings, search results, agent profiles |
| [Crawler Tools](skills/scraperapi-mcp/references/crawler.md) | URL regex patterns, depth vs budget, scheduling, webhooks |

## Recipes

| Recipe | Description |
|--------|-------------|
| [SERP & News Monitor](skills/scraperapi-mcp/recipes/serp-news-monitor.md) | Monitor Google SERP & News, extract structured JSON, change reports |

## License

MIT
