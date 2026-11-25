---
name: scraperapi-cli
description: >
  Reference for the official ScraperAPI command-line tool (`sapi`, package `scraperapi-cli`). Use
  this skill whenever the user wants to scrape, run async jobs, fetch structured data or check
  credits from a terminal or shell script. Trigger on: "scrape this URL from the terminal", "use
  sapi to fetch X", "ScraperAPI CLI", "ScraperAPI from bash", "sapi scrape", "sapi jobs", "pipe
  ScraperAPI into jq", "scrape from a Makefile / cron / CI", "check my ScraperAPI credits from the
  command line", "scrape 10000 URLs from a file". Covers install, auth, commands, JSON and piping
  behaviour, cost checks and shell recipes. Not for writing scraper code: for a new scraper script use scraperapi-scraper-builder,
  and for code that calls ScraperAPI use the language's SDK skill.

  Note: Transmits user-supplied queries, URLs, and content to ScraperAPI.
metadata:
  openclaw:
    requires:
      env:
        - SCRAPERAPI_API_KEY
    emoji: "⌨️"
    homepage: https://www.scraperapi.com/
---

# ScraperAPI CLI (`sapi`)

`sapi` is the official ScraperAPI command-line tool. It is the right choice when:

- The user is already in a terminal and wants a result *now* without writing a script.
- A scrape is part of a shell pipeline (`sapi … | jq …`, `xargs`, `make`, GitHub Actions).
- A one-off scheduled task (cron, launchd, systemd timer) needs to hit ScraperAPI without a project setup.
- The user is exploring — testing whether `--render` or `--premium` unblocks a target before committing the choice to code.

If the user is writing application code in Python, Node, PHP, Ruby, or Java, point them at the matching SDK skill instead — the CLI is for shells, not application logic.

## Install and authenticate

```bash
npm install -g scraperapi-cli    # requires Node.js 18+
sapi init                        # interactive: prompts for the key and validates it
```

Non-interactive setup (for CI / Dockerfiles):

```bash
sapi init --api-key YOUR_API_KEY
```

### Key resolution order

`sapi` looks for the API key in this order, stopping at the first hit:

1. `--api-key <key>` flag on the command
2. `SCRAPERAPI_API_KEY` environment variable
3. `~/.config/scraperapi/config.json` (written by `sapi init`)

In CI, prefer the env var — it keeps the key out of shell history and config files.

## Output contract — important for piping

| Stream | What goes there |
|--------|-----------------|
| stdout | The data (page body, JSON, table rows) |
| stderr | Spinners, warnings, errors |

When stdout is not a TTY (a pipe or redirect), `sapi` automatically switches to JSON mode:

```bash
sapi scrape https://example.com | jq .body            # auto-JSON
sapi scrape https://example.com > out.json            # auto-JSON
sapi scrape https://example.com                       # human output (page body to stdout)
```

Force JSON mode in a TTY with `--json`. Force a specific body format with `--output html|markdown|text|json|csv` — `--output` overrides the non-TTY auto-JSON rule, so you can pipe raw HTML or markdown into another tool without it being wrapped in JSON.

`sapi` does not prompt for confirmation on expensive calls. If you want to check what a request will cost before running it, use [`sapi cost`](#sapi-cost-url--preview-credit-cost).

## `sapi scrape <url>` — synchronous fetch

The workhorse command. Returns the page body to stdout.

```bash
sapi scrape https://example.com
sapi scrape https://example.com --render                       # 10 credits, JS rendering
sapi scrape https://example.com --render --wait-for "#main"    # wait for selector
sapi scrape https://example.com --country uk                   # geotarget UK
sapi scrape https://example.com --premium                      # 10 credits, residential proxies
sapi scrape https://example.com --ultra-premium                # 30 credits, hardest sites
sapi scrape https://example.com --autoparse --json             # structured JSON for supported sites
sapi scrape https://example.com --screenshot --json            # PNG (10 credits) returned as base64 in JSON
sapi scrape https://example.com --async                        # submit as async job, prints jobId
```

### Escalation ladder

Always start at the cheapest option and only escalate when a request fails:

1. Default (1 credit on a normal site; 5 on Amazon/Walmart/eBay, 25 on Google, 30 on LinkedIn, 10 behind Cloudflare and similar). If 200 → done.
2. `--render` if the page loads content via JavaScript (SPA, hydrated tables, infinite scroll). 10 credits.
3. `--premium` if the request fails with 500 or returns a block page from datacenter IPs. 10 credits.
4. `--ultra-premium` only for the hardest anti-bot sites (Cloudflare Turnstile, sophisticated fingerprinting). 30 credits.

Combining flags costs more than either alone: `--render --premium` is 25 credits, `--render --ultra-premium` is 75. `200` and `404` responses are charged; failed (`500`) ones are not.

### Flag reference

| Flag | Description | Credits |
|------|-------------|---------|
| `--render` | JavaScript rendering | 10 |
| `--premium` | Residential proxies | 10 (25 with `--render`) |
| `--ultra-premium` | Advanced anti-bot bypass | 30 (75 with `--render`) |
| `--screenshot` | PNG screenshot (base64 in JSON mode) | 10 |
| `--autoparse` | Auto-parsed JSON for supported sites | — |
| `--country <cc>` | Country code (e.g. `us`, `uk`, `de`). The UK is `uk`, not `gb` | — |
| `--device <type>` | `mobile` or `desktop` user-agent | — |
| `--output <fmt>` | `html`, `markdown`, `text`, `json`, `csv` | — |
| `--session <n>` | Sticky session number (reuse IP) | — |
| `--wait-for <sel>` | CSS selector to wait for (needs `--render`) | — |
| `--timeout <sec>` | Request timeout (default 70). Only `scrape` has this flag | — |
| `--async` | Submit as async job, print job ID, exit | — |
| `--json` | Force JSON output mode | — |
| `--api-key <key>` | Override the configured API key for one call | — |

There is no interactive confirmation step — `sapi scrape` runs the request immediately. Use `sapi cost <url>` (below) to preview credit cost without spending credits.

## `sapi cost <url>` — preview credit cost

```bash
sapi cost https://example.com --json                      # {"credits":1}
sapi cost https://example.com --render --premium --json   # {"credits":25}
sapi cost https://example.com --render --json | jq .credits
```

Use `--json` (or pipe the output) and read `.credits`. Accepts the same parameter flags as `sapi scrape` except `--async` and `--timeout`. Useful as a pre-flight check before running an expensive batch — wire it into a script when you want to fail fast if a chosen flag combination is more expensive than expected.

## `sapi jobs` — async scraping

Use async for batches, slow targets, or fire-and-forget work.

```bash
sapi scrape https://example.com --async              # returns a jobId on stdout
sapi jobs list                                        # list all jobs
sapi jobs get <jobId>                                 # poll every 2s until done, no time limit
sapi jobs get <jobId> --no-poll                       # one-shot status check
sapi jobs cancel <jobId>                              # cancel a running job
```

`sapi jobs get` without `--no-poll` waits until the job is `finished` or `failed`, with no timeout. A job that keeps retrying (async jobs can retry for up to 24 hours) makes a script hang. In scripts, poll with `--no-poll` in your own loop and give up after a deadline. Async jobs (the batch pipeline below) are the right tool whenever there are more than ~100 URLs — sync `sapi scrape` calls in a loop will burn through your concurrent request budget and hit 429s.

### Batch pipeline

Submit one async job per URL, then poll each one. Any `sapi scrape` flag works here (`--render`, `--premium`, `--autoparse`, …), and ScraperAPI handles the retries:

```bash
# 1. Submit one async job per URL (urls.txt: one URL per line), keep the job ids.
while read -r URL; do
  sapi scrape "$URL" --async --json < /dev/null | jq -r .id
done < urls.txt > job-ids.txt

# 2. Poll each one with a deadline (plain `jobs get` would wait forever on a stuck job).
while read -r ID; do
  for _ in $(seq 1 60); do                       # ~10 minutes per job
    JOB=$(sapi jobs get "$ID" --no-poll --json < /dev/null)
    STATUS=$(echo "$JOB" | jq -r .status)
    [ "$STATUS" = finished ] || [ "$STATUS" = failed ] && break
    sleep 10
  done
  echo "$JOB" >> batch-results.ndjson
done < job-ids.txt

# 3. Extract bodies that succeeded.
jq -s '[.[] | select(.status=="finished") | .response.body]' batch-results.ndjson
```

## `sapi structured` — pre-parsed JSON for supported sites

For Amazon and Google searches, prefer `structured` over scraping and parsing HTML. The endpoints return clean, schema-stable JSON.

```bash
sapi structured amazon search "wireless headphones"
sapi structured google serp "best espresso machine"
sapi structured google news "artificial intelligence"
sapi structured google jobs "software engineer remote"
sapi structured google shopping "standing desk"
```

All structured commands accept `--json` for raw output (auto-on when piped). See [`docs.scraperapi.com`](https://docs.scraperapi.com/) for the schema of each vertical.

### Product pages, Google Maps and Redfin: `scrape --autoparse`

For an Amazon, Walmart or eBay product, Google Maps, or Redfin, use `sapi scrape` with `--autoparse` and the page URL. ScraperAPI parses supported pages into JSON:

```bash
sapi scrape "https://www.amazon.com/dp/B07FTKQ97Q" --autoparse --json | jq '.body | fromjson'
sapi scrape "https://www.walmart.com/ip/5253396052" --autoparse --json | jq '.body | fromjson'
sapi scrape "https://www.ebay.com/itm/166619046796" --autoparse --json | jq '.body | fromjson'
```

In JSON mode, `sapi scrape` wraps the response as `{"body": "<string>", "statusCode": 200}`, so the parsed data is a JSON string inside `.body`, and `fromjson` turns it back into an object. Autoparse covers Amazon (product, search, offers, reviews), eBay (product, search), Walmart (product, category), Google (search, news, shopping, jobs, maps) and Redfin (for sale, for rent, search, agent pages). It costs the site's normal rate (5 credits for Amazon, Walmart and eBay).

## `sapi account` — usage and credits

```bash
sapi account --json                 # includes creditsLeft, requestCount, requestLimit, concurrencyLimit
sapi account --json | jq .creditsLeft
```

Use `--json` whenever you need the credit balance: `creditsLeft` is in the JSON output.

Run `sapi account --json` before anything large: it's the cheapest way to confirm the key works and that there are enough credits.

## `sapi config` — stored API key

```bash
sapi config list
sapi config get api_key
sapi config set api_key YOUR_API_KEY
```

Pass `--country`, `--output` and `--timeout` on each call. On a shared machine, use the `SCRAPERAPI_API_KEY` env var rather than storing the key in the config file.

## Shell recipes

### Scrape, extract a title with `pup`, append to CSV

```bash
URL="https://example.com"
TITLE=$(sapi scrape "$URL" --json | jq -r .body | pup 'title text{}')
echo "$URL,$TITLE" >> titles.csv
```

### Daily price check via cron

```bash
# crontab: 7am every day
0 7 * * * /usr/local/bin/sapi scrape "https://www.amazon.com/dp/B07FTKQ97Q" --autoparse --json \
  | jq -c '.body | fromjson | {ts: now, price: .pricing}' >> ~/prices.ndjson
```

### Parallel scrape with `xargs`

```bash
# Run 5 in parallel; --json puts one record per line (no prompts to worry about).
cat urls.txt | xargs -n1 -P5 sapi scrape --render --json > out.ndjson
```

Above ~100 URLs prefer the async [batch pipeline](#batch-pipeline) over this — it sidesteps the concurrent request limit.

### Fail-loud in CI

`sapi` exits non-zero on any error (2 when credits are exhausted, 1 otherwise), so `set -e` works as expected:

```bash
set -euo pipefail
sapi account --json | jq -e '.creditsLeft > 1000'         # fails the build if low
sapi scrape "$URL" --output markdown > page.md
```

### Escalate on block

```bash
sapi scrape "$URL" --output html > out.html \
  || sapi scrape "$URL" --render --output html > out.html \
  || sapi scrape "$URL" --premium --output html > out.html
```

Cheap → expensive, only paying for the next tier when the previous one fails. `--output html` keeps raw HTML in each redirect — without it the non-TTY rule would wrap the body in JSON.

## When NOT to reach for `sapi`

- **Inside application code** (Python, Node, PHP, Ruby, Java) — use the matching SDK skill; shelling out to a CLI is brittle.
- **Tight loops over thousands of URLs in a shell script** — use async jobs (the [batch pipeline](#batch-pipeline), or the Async API's batch endpoint via the scraperapi-async skill) so ScraperAPI handles concurrency and retries.
- **Long-running scheduled projects with webhook delivery** — DataPipeline, configured in the dashboard (scraperapi-datapipeline skill).
- **Whole-site crawls** — the Crawler API (scraperapi-crawler skill), or build the URL list from a sitemap or a Google `site:` search and scrape it with the [batch pipeline](#batch-pipeline).
- **Managing API keys** — the [ScraperAPI dashboard](https://dashboard.scraperapi.com/).

## Troubleshooting

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| `command not found: sapi` | Not installed, or `npm` global bin not on `$PATH` | `npm install -g scraperapi-cli`; check `npm config get prefix` |
| `401 Invalid API key` | Key missing or wrong | Run `sapi account` to confirm; re-run `sapi init` |
| `500` on a public page | ScraperAPI retried for 70s and still got blocked (not charged) | Retry with `--premium`, then `--ultra-premium` |
| `403`, exit code 2 | Credits exhausted for this billing cycle | Check `sapi account --json`; upgrade or enable overages at https://dashboard.scraperapi.com/billing |
| `429` from `sapi` | Hit concurrent request limit | Switch to the async [batch pipeline](#batch-pipeline) or lower the `xargs -P` cap |
| Script hangs forever | `sapi jobs get` polls with no time limit | Use `--no-poll` in a loop with a deadline (see the batch pipeline above) |
| `sapi scrape` times out | Target is slow; default timeout is 70s | `--timeout 120`, or use `--async`. Other commands have no `--timeout` |
| Output is JSON instead of HTML | stdout is piped → auto-JSON | Force with `--output html` if you want raw HTML in a pipe |

See [`docs.scraperapi.com`](https://docs.scraperapi.com/) for the full parameter reference and [current credit costs](https://docs.scraperapi.com/getting-started/quick-start/credits-and-requests-costs).
