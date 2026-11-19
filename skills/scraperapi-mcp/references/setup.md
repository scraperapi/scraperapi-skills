# MCP Server Setup

## Server Variants

ScraperAPI offers two MCP server variants. **Both expose the same full set of 28 tools** — they differ only in transport and configuration, not in which tools are available.

All 28 tools: `scrape`, `google_search`, `google_news`, `google_jobs`, `google_shopping`, `google_maps_search`, `amazon_product`, `amazon_search`, `amazon_offers`, `walmart_search`, `walmart_product`, `walmart_category`, `walmart_review`, `ebay_search`, `ebay_product`, `redfin_for_sale`, `redfin_for_rent`, `redfin_search`, `redfin_agent`, `crawler_job_start`, `crawler_job_status`, `crawler_job_delete`, `ai_parser_create`, `ai_parser_get_details`, `ai_parser_parse_url`, `ai_parser_list`, `ai_parser_update`, `ai_parser_delete`.

### Remote (Hosted) — recommended

The ScraperAPI plugin configures this for you and asks for the API key when the plugin is enabled. To add it by hand:

```json
{
  "mcpServers": {
    "ScraperAPI": {
      "type": "http",
      "url": "https://mcp.scraperapi.com/mcp",
      "headers": {
        "Authorization": "Bearer <YOUR_SCRAPERAPI_API_KEY>"
      }
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

- **Remote** — an HTTP server at `https://mcp.scraperapi.com/mcp` with the API key passed as an `Authorization: Bearer` header.
- **Local** — run as `python -m scraperapi_mcp_server` with the API key in the `API_KEY` env var.

Either way, select tools from the full list above — no variant-specific fallbacks are needed.

### Where the API key goes

| Variant | Setting | Description |
|---------|---------|-------------|
| Remote | `Authorization: Bearer` header | Entered in the plugin's settings when it is enabled, or written into the header by hand |
| Local | `API_KEY` env var | API key passed to the local Python MCP server |

Both hold the same ScraperAPI key value. Direct users to https://dashboard.scraperapi.com to obtain or manage their key.
