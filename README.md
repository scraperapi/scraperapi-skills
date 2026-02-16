# ScraperAPI Skills

[ScraperAPI](https://www.scraperapi.com)'s Agent Skills.

## Available Skills

### `scraperapi-mcp`

Teaches Claude how to optimally use ScraperAPI's MCP tools — selecting the right tool, setting optimal parameters, minimizing credit costs, and handling large outputs.

**Install:**
```bash
npx skills add scraperapi/scraperapi-skills
```

## ScraperAPI MCP Server Setup

Before using the skill, you need the [ScraperAPI MCP server](https://docs.scraperapi.com/integrations/llm-integrations/mcp-server/hosted-remote) connected to Claude Code.

1. Get your API key from the [ScraperAPI Dashboard](https://dashboard.scraperapi.com/)
2. Add the MCP server to your Claude Code configuration:

```json
{
  "mcpServers": {
    "ScraperAPI": {
      "command": "npx",
      "args": ["mcp-remote", "https://mcp.scraperapi.com/mcp", "--header", "Authorization: Bearer YOUR_API_KEY"]
    }
  }
}
```

## MCP Tools

The ScraperAPI MCP server provides 9 tools:

| Tool | Purpose |
|------|---------|
| `scrape` | Scrape any URL and return its content (markdown, text, HTML, JSON, CSV) |
| `google_search` | Structured Google web search results |
| `google_news` | Google News search results |
| `google_jobs` | Google Jobs search results |
| `google_shopping` | Google Shopping product and price results |
| `google_maps_search` | Google Maps local business results |
| `crawler_job_start` | Start a site-wide crawl job |
| `crawler_job_status` | Check crawl job progress |
| `crawler_job_delete` | Cancel and delete a crawl job |

## What the Skill Provides

- **Tool selection guidance** — decision tree for picking the right tool for any task
- **Parameter optimization** — when to use `render`, `premium`, `timePeriod`, `countryCode`, etc.
- **Credit cost awareness** — escalation strategy (standard → render → premium → ultraPremium) to minimize spend
- **Output handling** — strategies for processing large scrape results without flooding the context window
- **Crawler configuration** — URL regex patterns, depth vs budget, scheduling, webhooks

## License

MIT
