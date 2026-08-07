# MCP Server Setup

## Server Variants

ScraperAPI offers two MCP server variants. **Both expose the same full set of 28 tools** — they differ only in transport and configuration, not in which tools are available.

All 28 tools: `scrape`, `google_search`, `google_news`, `google_jobs`, `google_shopping`, `google_maps_search`, `amazon_product`, `amazon_search`, `amazon_offers`, `walmart_search`, `walmart_product`, `walmart_category`, `walmart_review`, `ebay_search`, `ebay_product`, `redfin_for_sale`, `redfin_for_rent`, `redfin_search`, `redfin_agent`, `crawler_job_start`, `crawler_job_status`, `crawler_job_delete`, `ai_parser_create`, `ai_parser_get_details`, `ai_parser_parse_url`, `ai_parser_list`, `ai_parser_update`, `ai_parser_delete`.

### Remote (Hosted) — recommended

```json
{
  "mcpServers": {
    "ScraperAPI": {
      "command": "npx",
      "args": ["mcp-remote", "https://mcp.scraperapi.com/mcp", "--header", "Authorization: Bearer ${SCRAPERAPI_API_KEY}"]
    }
  }
}
```

### Local (Self-Hosted)

Python-based, installed via PyPI (`pip install scraperapi-mcp-server`). Exposes the **same full set of 28 tools** as the remote variant (scrape, all SDEs, crawler, and AI Parser). Requires Python 3.11+. The AI Parser tools call the AI Parser service at `AIPARSER_URL` (defaults to `https://aiparser.scraperapi.com`).

```json
{
  "mcpServers": {
    "ScraperAPI": {
      "command": "python",
      "args": ["-m", "scraperapi_mcp_server"],
      "env": {
        "API_KEY": "<YOUR_SCRAPERAPI_API_KEY>"
      }
    }
  }
}
```

## Variant Detection

Both variants expose the same 28 tools, so tool presence no longer distinguishes them. Tell them apart by transport and configuration instead:

- **Remote** — connected via `npx mcp-remote https://mcp.scraperapi.com/mcp` with the API key passed as a `Authorization: Bearer` header (env `SCRAPERAPI_API_KEY`).
- **Local** — run as `python -m scraperapi_mcp_server` with the API key in the `API_KEY` env var.

Either way, select tools from the full list above — no variant-specific fallbacks are needed.

### Environment Variables

| Variant | Env Var | Description |
|---------|---------|-------------|
| Remote | `SCRAPERAPI_API_KEY` | API key passed as Bearer token to the hosted MCP server |
| Local | `API_KEY` | API key passed to the local Python MCP server |

Both variables hold the same ScraperAPI key value — the different names reflect what each server variant expects. Direct users to https://www.scraperapi.com/dashboard to obtain or manage their key.
