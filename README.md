# ScraperAPI Plugin for Claude Code

## Overview

ScraperAPI's official Claude Code plugin — a growing library of agent skills that cover the full ScraperAPI surface: onboarding, product primitives, SDK best practices, command-line and workflow tooling, and end-to-end data pipelines.

- **Onboarding & product routing** — `scraperapi-agent-onboarding` decides which ScraperAPI product fits the job.
- **Product primitives** — first-class skills for the [Standard API](https://docs.scraperapi.com/), [Async Jobs](https://docs.scraperapi.com/making-async-requests), [Crawler](https://docs.scraperapi.com/crawler), [DataPipeline](https://docs.scraperapi.com/data-pipeline), and [MCP](skills/scraperapi-mcp/SKILL.md) tools.
- **SDK best practices** — language-specific guides for Python, Node.js, Ruby, PHP, and Java.
- **Tools** — official ScraperAPI [CLI](skills/scraperapi-cli/SKILL.md) and [n8n](https://n8n.io/integrations/scraperapi/) community node.
- **Workflows** — opinionated, ready-to-run recipes: scraper building, lead enrichment, autonomous research, market research, SEO audits, SERP intelligence, and price monitoring.

[Learn more about agent skills](https://agentskills.io/).

---

## Quick Start

### 1. Get an API key

Sign up at the [ScraperAPI dashboard](https://dashboard.scraperapi.com) and copy your key.

### 2. Set your environment

```bash
export SCRAPERAPI_API_KEY="your-key-here"
```

### 3. Install the plugin

The plugin ships with ScraperAPI's [hosted MCP server](https://mcp.scraperapi.com/mcp) so all 22 MCP tools become available to the agent, authed via your `SCRAPERAPI_API_KEY`.

**Claude Code community marketplace:**

```
/plugin marketplace add scraperapi/scraperapi-skills
/plugin install scraperapi@scraperapi-skills
```


**Claude Plugin Hub:**

```bash
export SCRAPERAPI_API_KEY="your-key-here"
npx claudepluginhub scraperapi/scraperapi-skills --plugin scraperapi
```

**OpenClaw (from [ClawHub](https://clawhub.ai)):**

```bash
export SCRAPERAPI_API_KEY="your-key-here"
openclaw plugins install clawhub:@scraperapitech/scraperapi-skills
```


Once installed, the skills load automatically — start an agent session and try one of the [Usage](#usage) prompts below.

---

## Skills

| Skill | Description |
|-------|-------------|
| **[`scraperapi-agent-onboarding`](skills/scraperapi-agent-onboarding/SKILL.md)** | Entry point for agents and developers new to ScraperAPI. Read first when integrating ScraperAPI into an agent or app, picking the right product, or obtaining an API key. |
| **[`scraperapi-mcp`](skills/scraperapi-mcp/SKILL.md)** | Teaches Claude how to optimally use the ScraperAPI MCP tools — tool selection, parameter tuning, credit minimization, large-output handling. Includes ready-to-use recipes. |
| **[`scraperapi-async`](skills/scraperapi-async/SKILL.md)** | Async Jobs API: submit scraping jobs in the background, retrieve via polling or webhook, including batch jobs up to 50,000 URLs. |
| **[`scraperapi-crawler`](skills/scraperapi-crawler/SKILL.md)** | Crawler product: crawl an entire site or section by following links automatically when target URLs aren't known up front. |
| **[`scraperapi-datapipeline`](skills/scraperapi-datapipeline/SKILL.md)** | DataPipeline: managed, scheduled scraping projects that run automatically and deliver results via webhook or dashboard download. |
| **[`scraperapi-python-sdk`](skills/scraperapi-python-sdk/SKILL.md)** | Best-practices reference for the Python SDK (`scraperapi-sdk`): client setup, the standard → render → premium → ultra_premium escalation ladder, async batches, structured data, error handling. |
| **[`scraperapi-nodejs-sdk`](skills/scraperapi-nodejs-sdk/SKILL.md)** | Best-practices reference for the Node.js / TypeScript SDK (`scraperapi-sdk`): CommonJS and ESM, async/await, POST/PUT, account info, error handling. |
| **[`scraperapi-ruby-sdk`](skills/scraperapi-ruby-sdk/SKILL.md)** | Best-practices reference for the Ruby SDK (`scraperapi` gem): gem setup, all parameters, escalation ladder, sessions, status-code error handling. |
| **[`scraperapi-php-sdk`](skills/scraperapi-php-sdk/SKILL.md)** | Best-practices reference for the PHP SDK (`scraperapi/sdk` Composer package): setup, all parameters, POST requests, error handling. |
| **[`scraperapi-java-sdk`](skills/scraperapi-java-sdk/SKILL.md)** | Best-practices reference for the Java SDK (`com.scraperapi:sdk`): fluent builder API, escalation ladder, async jobs, structured data. |
| **[`scraperapi-cli`](skills/scraperapi-cli/SKILL.md)** | Use the official `sapi` command-line tool from shells, pipelines, cron, and CI. Covers install, auth, every command, JSON / piping behaviour, and common recipes. |
| **[`scraperapi-n8n`](skills/scraperapi-n8n/SKILL.md)** | Generate n8n workflows that use the official ScraperAPI community node (`n8n-nodes-scraperapi-official`). Produces importable workflow JSON or step-by-step build guides. |
| **[`scraperapi-scraper-builder`](skills/scraperapi-scraper-builder/SKILL.md)** | Build a runnable scraper end-to-end: picks structured endpoint vs raw HTML, JS rendering, proxy tier, sync vs async, then generates a complete Python or Node.js script with retries, pagination, and credit estimation. |
| **[`scraperapi-lead-enrichment`](skills/scraperapi-lead-enrichment/SKILL.md)** | Enrich a contact or company from any seed (name, domain, LinkedIn URL, email) and produce a structured contact card with person and company fields. |
| **[`scraperapi-research-agent`](skills/scraperapi-research-agent/SKILL.md)** | Autonomous research agent: takes a question, uses ScraperAPI to search and scrape relevant pages, uploads content as file artifacts to the Anthropic Files API, then feeds everything to Claude for a cited markdown report. |
| **[`scraperapi-market-research`](skills/scraperapi-market-research/SKILL.md)** | Market research from live web data: consumer sentiment, demand and trend signals, price and category structure, and competitive landscape — for product, pricing, positioning, or investment decisions. |
| **[`scraperapi-seo-audit`](skills/scraperapi-seo-audit/SKILL.md)** | Comprehensive SEO audit using live SERPs and on-page scraping: keyword rankings, titles/meta/headings/schema, content gaps, indexation, and prioritized recommendations. Runs out of the box via the bundled MCP tools. |
| **[`scraperapi-serp-intelligence`](skills/scraperapi-serp-intelligence/SKILL.md)** | SERP landscape analysis for SEO strategy: AI Overview presence and attribution, SERP feature composition, query intent, competitor visibility, and where rankings translate to traffic. |
| **[`scraperapi-price-monitoring`](skills/scraperapi-price-monitoring/SKILL.md)** | Track Amazon, Walmart, and Google Shopping prices from ASINs, URLs, or product queries. Detects increases, decreases, restocks, and out-of-stock transitions against an optional baseline. |


## Usage

Each skill triggers from natural-language prompts in Claude Code. A few starting points:

```markdown
# Onboarding
"I'm building an agent that needs web data — how do I use ScraperAPI?"

# Product primitives
"Submit these 5,000 URLs as a ScraperAPI async batch and call me back at https://my-app/cb"
"Crawl example.com and list every product URL"
"Set up a DataPipeline project that scrapes these ASINs every morning"

# CLI
"Use sapi to scrape https://news.ycombinator.com and pipe titles into a CSV"
"Write a cron line that runs sapi structured amazon product on this ASIN daily"

# SDKs
"Write a Python script that scrapes this URL with premium proxies and retries on 429"
"Show me the Node.js SDK call to autoparse Amazon product pages"

# Workflows
"Build me a scraper for this e-commerce site that extracts title, price, and stock"
"Enrich this contact: jane@example.com — find her LinkedIn, company, and role"
"Research the top Python web scraping frameworks and give me a cited report"
"Investigate how competitors are pricing their SaaS plans — scrape real pages and summarise"
```

Each skill's `SKILL.md` documents its full set of trigger phrases and behaviour.


## Project Structure

```
.
├── .claude-plugin/
│   ├── plugin.json              # Claude Code plugin manifest
│   └── marketplace.json         # Claude Code marketplace listing
├── openclaw.plugin.json         # OpenClaw / ClawHub plugin manifest
├── package.json                 # scoped package name for ClawHub publishing
├── .mcp.json                    # hosted MCP server config (bundled for Claude Code)
├── skills/
│   ├── scraperapi-agent-onboarding/
│   ├── scraperapi-async/
│   ├── scraperapi-cli/
│   ├── scraperapi-crawler/
│   ├── scraperapi-datapipeline/
│   ├── scraperapi-java-sdk/
│   ├── scraperapi-lead-enrichment/
│   ├── scraperapi-market-research/
│   ├── scraperapi-mcp/
│   │   ├── SKILL.md
│   │   ├── references/          # per-tool deep dives
│   │   └── recipes/             # end-to-end workflows
│   ├── scraperapi-n8n/
│   ├── scraperapi-nodejs-sdk/
│   ├── scraperapi-php-sdk/
│   ├── scraperapi-price-monitoring/
│   ├── scraperapi-python-sdk/
│   ├── scraperapi-research-agent/
│   │   ├── SKILL.md
│   │   ├── scripts/             # research_agent.py — runnable CLI
│   │   └── assets/              # report_template.md
│   ├── scraperapi-ruby-sdk/
│   ├── scraperapi-scraper-builder/
│   ├── scraperapi-seo-audit/
│   └── scraperapi-serp-intelligence/
├── dist/                        # packaged .skill artifacts (for distribution)
├── LICENSE
└── README.md
```

Each `skills/<name>/SKILL.md` is self-contained — frontmatter declares trigger conditions and required environment, the body teaches Claude how to do the job.

---